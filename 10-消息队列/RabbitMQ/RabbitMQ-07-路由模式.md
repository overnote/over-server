## 一 引入路由模式 RoutingKey

在发布订阅模式中，生产者广播消息给了多个接收者，这些消息可以通过路由进行过滤，比如在日志系统中，想要扩展成为根据消息的严重程度来过滤分发，如只将严重的error级别的日志写入磁盘，而不写入info和warn类型的日志消息以节省磁盘空间。 

`ch.QueueBind()`方法负责绑定交换机与队列的关系。
```go
    err = ch.QueueBind(
        q.Name,     //queue name
        "black",    //routing key：这里可以设定绑定的路由
        "logs",     //exchange
        false,
        nil)
```

`fanout`类型的交换机只能广播转发，所以这里也需要使用`direct`类型。Direct exchange的路由算法很简单：就是将exchange的binding_key和消息的routing_key进行比较，如果完全匹配这说明是需要分发的队列。  

![](../images/mq/mq-17.png)  

在图中，`direct`交换机 X 有两个队列与之绑定：
- 队列Q1：由1个路由`binding_key`orange路由到
- 队列Q2：由2个路由`binding_key`: black,green路由到

当然也可以实现多重绑定：  
![](../images/mq/mq-18.png)   

此时交换机会将消息广播至所有匹配的绑定队列，以此实现对同一个binding_key需要分发到多个队列的情况。

## 二 完整代码

### 2.1 生产端

```go
package main

import (
	"flag"
	"github.com/streadway/amqp"
	"log"
)

func main() {

	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	errorHandle("ramqp dial err", err)
	defer conn.Close()

	ch, err := conn.Channel()
	errorHandle("get channel err", err)

	// 声明一个交换机
	err = ch.ExchangeDeclare(
		"logs_direct",
		"direct",
		true,
		false,
		false,
		false,
		nil,
		)
	errorHandle("declare exchange err", err)

	// 发送消息:go run -str=test...
	arg := flag.String("str", "0", "args name...")
	flag.Parse()
	body := *arg
	err = ch.Publish(
		"logs_direct",
		"wrong",				// 这里可以依据不同的日志级别进行隔离
		false,
		false,
		amqp.Publishing {
			ContentType: "text/plain",
			Body:        []byte(body),
		})
	errorHandle("publish a message err", err)

}

func errorHandle(msg string, err error) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}
```

### 2.2 消费端

```go
package main

import (
	"bytes"
	"fmt"
	"github.com/streadway/amqp"
	"log"
	"time"
)

func main()  {

	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	errorHandle("connect rabbitmq err", err)
	defer conn.Close()

	ch, err := conn.Channel()
	errorHandle("open channel err", err)
	defer ch.Close()

	// 声明一个交换机
	err = ch.ExchangeDeclare(
		"logs_direct",
		"direct",
		true,
		false,
		false,
		false,
		nil,
	)
	errorHandle("declare exchange err", err)

	// 声明一个队列：这个队列专门负责处理 wrong 级别日志
	q, err := ch.QueueDeclare(
		"",
		false,
		false,
		true,
		false,
		nil,
	)
	errorHandle("declare queue err", err)

	// 绑定队列：这时候就可以将不同的队列绑定到不同的交换机
	err = ch.QueueBind(
		q.Name, 			//queue name
		"wrong",       //routing key，该key必须和生产端的key一致
		"logs_direct",    //exchange
		false,
		nil,
	)

	// 消费消息
	msgs, err := ch.Consume(
		q.Name,
		"",
		true,
		false,
		false,
		false,
		nil,
	)
	errorHandle("register a consumer err", err)

	forever := make(chan bool)
	go func() {
		for d := range msgs{
			log.Printf("Received a message: %s", d.Body)
			dot_count := bytes.Count(d.Body, []byte("."))
			t := time.Duration(dot_count)		// 利用参数中 . 的数目不同制作间隔
			time.Sleep(t* time.Second)
			log.Printf("Done")
		}
	}()

	fmt.Printf(" [*] Waiting for messages. To exit press CTRL+C \n")
	<-forever
}

func errorHandle(msg string, err error) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}
```
