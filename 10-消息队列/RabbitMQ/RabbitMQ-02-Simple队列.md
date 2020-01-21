## 一 Simple简单队列

### 1.0 RabbitMQ提供的队列模式

为了便于开发，依据不同的场景给予最合适的队列，RabbitMQ提供了多种队列的实现方式：
- Simple queue 简单队列
- Work queues 工作队列
- Publish/Subscribe 发布订阅
- Routing
- Topics
- RPC

简单队列如图所示：  

![](../../images/arch/rabbitmq-14)  

队列作为中介进行消息的发送饿接收：
- producter（生产者）是指用于发送消息的用户程序；
- queue（队列）是用来存储消息的缓冲区；
- consumer（消费者）是用来接收消息的用户程序；

### 1.1 生产端代码

```go
package main

import (
	"log"
	"github.com/streadway/amqp"
)

func main()  {

	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	errorHandle("ramqp dial err", err)
	defer conn.Close()

	ch, err := conn.Channel()
	errorHandle("get channel err", err)

    // 声明一个队列
	q, err := ch.QueueDeclare(
		"myqueue01", // name
		false,   	// durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	errorHandle("declare a queue err", err)

	// 发送消息
	body := "hello "
	fmt.Println("body=", body)
	err = ch.Publish(
		"",     // exchange 默认传入空字符串时，使用默认的exchange / ，在该exchange中查找对应的queue
		q.Name, 			// routing key
		false,  // mandatory
		false,  // immediate
		amqp.Publishing {
			DeliveryMode: amqp.Persistent,
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

### 1.2 消费端代码

```go
package main

import (
	"log"
	"github.com/streadway/amqp"
)

func main() {

	// 建立连接
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	errorHandle("connect rabbitmq err", err)
	defer conn.Close()

	// 从连接获取channel
	ch, err := conn.Channel()
	errorHandle("open channel err", err)
	defer ch.Close()

	// 声明一个队列
	q, err := ch.QueueDeclare(
		"myqueue01", // name
		false,   // 是否持久化，设置为true后，服务器即使重启，队列也不会消失
		false,   // delete when unused
		false,   // 是否独占：该队列是否只能被一个channel监听
		false,   // no-wait
		nil,     // arguments
	)
	errorHandle("declare a queue err", err)

	// 消费消息
	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto-ack
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	errorHandle("register a consumer err", err)

	forever := make(chan bool)
	go func() {
		for d := range msgs {
			fmt.Printf("Received a message: %s \n", d.Body)
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