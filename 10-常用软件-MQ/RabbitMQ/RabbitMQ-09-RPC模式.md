## 一 RPC模式概述

RPC系统的模式是客户端发送请求给服务器，服务器接收处理后回复一条响应消息。RabbitMQ对此提供了很好的支持，通过在请求中指定callback的回调队列地址来实现接收服务器的响应消息：
```go
q, err := ch.QueueDeclare(
    "",     //name
    false,  //durable
    false,  //delete when usused
    true,   //exclusive
    false,  //nowait
    nil,    //argments
)

err = ch.Publish(
    "",     //exchange
    "rpc_queue" //routing key
    false,  //mandatory
    false,  //immediate
    amqp.Publishing{
        ContentType:    "text/plain",
        CorrelationId:  corrId,
        ReplyTo:        q.Name,     // 回调队列
        Body:           []byte(strconv.Itoa(n)),
    }
)
```

RPC的调用过程：
- Client启动时创建一个匿名的、独有的回调队列；
- 对每一个RPC Request，Client都会设置两个参数：用于标识回调队列的reply_to和用于唯一标识的correlation_id;
- Request被发送到rpc_queue队列。
- RPC服务器等待rpc_queue的消息，一旦消息到达，处理任务后将响应结果消息发送到reply_to指定的队列；
- Client等待callback队列的消息，一旦消息到达，查找与correlation_id匹配的request，然后返回给应用程序。  

![](../../images/mq/rabbitmq-20.png)

RPC server对Client请求的响应同样需要通过消息队列来传递，为了提升效率，每个客户端只创建一个队列（而不是每次请求创建一个队列），回调队列接收到一个response之后，使用correlataion_id来标识。客户端在request中添加一个唯一的correlation_id，在接收到服务器返回的response时，根据该值来确定与之匹配的request并处理。如果未能找到与之匹配的correlation_id，说明该response并不属于当前client的请求，为了安全起见，将其忽略即可。  

我们可能会问：为什么在没有找到与之匹配的correlation_id时是将其忽略而不是失败报错？这是考虑到服务端的竞争条件：假设RPC server在发送response后宕机了，而此时却没能对当前request发出确认消息(ack).如果这种情况出现，该请求还在队列中会被再次派发。因此当前Request会在服务端处理两次，也会给客户端发送两次Response，故而，client要能处理重复的response，而server端对于Request需要实现幂等。  

## 二 完整代码

### 2.1 客户端

```go
package main

import (
	"flag"
	"fmt"
	"github.com/streadway/amqp"
	"log"
	"math/rand"
	"strconv"
	"time"
)

func main(){

	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	errorHandle( "Failed to connect to RabbitMQ", err)
	defer conn.Close()

	ch, err := conn.Channel()
	errorHandle("Failed to open a channel", err)
	defer ch.Close()

	q, err := ch.QueueDeclare(
		"",             //name
		false,            //durables
		false,            //delete when unused
		true,            //exclusive
		false,            //no wait
		nil,            //args
	)
	errorHandle("Failed to declare a queue", err)

	msgs , err := ch.Consume(
		q.Name,        //queue
		"",            //consumer
		true,        //auto-ack
		false,        //exclusive
		false,        //no-lock
		false,        //nowait
		nil,
	)
	errorHandle("Faield to register a consumer", err)

	// 模拟制作参数
	rand.Seed(time.Now().UTC().UnixNano())
	arg := flag.Int("num", 0, "args name...")
	flag.Parse()
	body := *arg
	fmt.Println("body==", body)

	// 调用rpc
	corrId := randomString(32)
	err = ch.Publish(
		"",            //exchange
		"rpc_queue",    //routing key
		false,        //mandatory
		false,        //immediate
		amqp.Publishing{
			ContentType:        "text/plain",
			CorrelationId:      corrId,		// 设置一个CorrelationId
			ReplyTo:            q.Name,
			Body:               []byte(strconv.Itoa(body)),
		})
	errorHandle("Failed to publish a message", err)

	for d:= range msgs {
		if corrId == d.CorrelationId {
			res, err := strconv.Atoi(string(d.Body))
			errorHandle("Failed to convert body to integer", err)
			log.Printf(" [.] Got %d", res)
			break
		}
	}
}

func errorHandle(msg string, err error) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}

func randomString(l int) string {
	bytes := make([]byte, l)
	for i:=0; i<l; i++ {
		bytes[i] = byte(randInt(65, 90))
	}
	return string(bytes)
}

func randInt(min int, max int) int {
	return min + rand.Intn(max - min)
}
```

### 2.2 服务端

```go
package main

import (
	"github.com/streadway/amqp"
	"log"
	"strconv"
	"time"
)

// 模拟一个rpc服务
func rpcFunc(n int) int{
	log.Println("server rpc run...")
	time.Sleep(time.Second * 2)
	return n + 100
}

func main(){

	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	errorHandle("Failed to connect to RabbitMQ", err)
	defer conn.Close()

	ch, err := conn.Channel()
	errorHandle("Failed to open a channel", err)
	defer ch.Close()

	q, err := ch.QueueDeclare(
		"rpc_queue",     //name
		false,            //durables
		false,            //delete when unused
		false,            //exclusive
		false,            //no wait
		nil,            //args
	)
	errorHandle("Failed to declare a queue", err)

	err = ch.Qos(
		1,     // prefetch count
		0,  // prefetch size
		false,    //global
	)
	errorHandle("Failed to set Qos", err)

	msgs, err := ch.Consume(
		q.Name,        //queue
		"",            //exchange
		false,        // auto-ack
		false,        //exclusive
		false,        //no-local
		false,        //no-wait
		nil,        //args
	)
	errorHandle("Failed to register a consumer", err)

	forever := make(chan bool)
	go func() {

		for d := range msgs {

			n, err := strconv.Atoi(string(d.Body))
			errorHandle("Failed to convert body to an integer", err)

			log.Printf(" [.] rpcFunc(%d)", n)
			res := rpcFunc(n)

			err = ch.Publish(
				"",        //exchange
				d.ReplyTo,        //routing key
				false,        //mandatory
				false,        //immediate
				amqp.Publishing{
					ContentType :    "text/plain",
					CorrelationId:    d.CorrelationId,
					Body:            []byte(strconv.Itoa(res)),
				})
			errorHandle("Failed to publish a message", err)

			d.Ack(false)
		}
	}()
	log.Printf(" [*] Awaiting RPC req")
	<-forever
}

func errorHandle(msg string, err error) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}
```

### 2.3 运行

```
# 运行服务端
go run server.go

# 运行客户端
go run client.go -num=11
```