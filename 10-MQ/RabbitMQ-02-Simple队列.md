## 一 RabbitMQ快速入门

### 1.1 接收端代码

```go
package main

import (
	"fmt"
	"github.com/streadway/amqp"
)

func main() {

	// 建立连接
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	if err != nil {
		fmt.Println("connect rabbitmq err: ", err)
		return
	}
	defer conn.Close()

	// 从连接获取channel
	ch, err := conn.Channel()
	if err != nil {
		fmt.Println("open channel err: ", err)
		return
	}
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
	if err != nil {
		fmt.Println("declare a queue err: ", err)
		return
	}

	// 获取消息
	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto-ack
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	if err != nil {
		fmt.Println("register a consumer err: ", err)
		return
	}


	forever := make(chan bool)
	go func() {
		for d := range msgs {
			fmt.Printf("Received a message: %s \n", d.Body)
		}
	}()

	fmt.Printf(" [*] Waiting for messages. To exit press CTRL+C \n")
	<-forever

}
```

### 1.2 发送代码

```go
package main

import (
	"fmt"
	"github.com/streadway/amqp"
	"time"
)

func main()  {

	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	if err != nil {
		fmt.Println("amqp dial err: ", err)
		return
	}
	defer conn.Close()

	ch, err := conn.Channel()
	if err != nil {
		fmt.Println("get a channel err: ", err)
		return
	}

    // 声明一个队列
	q, err := ch.QueueDeclare(
		"myqueue01", // name
		false,   	// durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	if err != nil {
		fmt.Println("declare a queue err: ", err)
		return
	}

	for i := 1; i <= 5; i++ {

		body := "hello "
		fmt.Println("body=", body)
		err = ch.Publish(
			"",     // exchange 默认传入空字符串时，使用默认的exchange / ，在该exchange中查找对应的queue
			 q.Name, 			// routing key
			false,  // mandatory
			false,  // immediate
			amqp.Publishing {
				ContentType: "text/plain",
				Body:        []byte(body),
			})
		if err != nil {
			fmt.Println("publish a message err: ", err)
			return
		}
		fmt.Println("发送消息：", body)
	}
}
```