## 一  发布订阅模式

在工作队列中，队列中的每个任务都只会被分发到一个工作者（消费端）进行处理。如果将同一个消息发送给多个消费者进行处理，这就是广为人知的发布/订阅模式。  

发布订阅系统常见于日志系统，包括两个部分：
- 生产者：产生日志消息
- 消费者：两个消费者，一个负责将消息打印在屏幕，一个负责将消息写入磁盘

本质上来说，当系统收到一个日志处理请求时，会把这个消息广播给所有的接收者。而RabbitMQ交换机的fanout类型会将所接收到的消息广播给所有绑定的队列，这很符合日志的场景。  

消费端设置交换机类型：
```go
	// 获取channel后自定义一个交换机
	err = ch.ExchangeDeclare(
		"logs",		// 交换机名称
		"fanout",		// 交换机类型，该类型适合发布订阅
		false,
		false,
		false,
		false,
		nil,
		)
	errorHandle("get exchange err", err)
```

此时的队列设置：若使用命名队列，那么必须让生产者和消费者同时使用一个名称的队列。在日志系统中红，队列的名字并不是重点，因为日志系统需要记录所有的消息，而不是某个队列中的消息，日志系统关系的是消费者程序接受和处理的消息都应该是未处理过的。  

首先，无论何时当消费者连接到Rabbit时我们需要一个新的、空的队列，因此就不会存在之前的消息。我们可以通过创建一个随机名字的队列来实现，而更好的方法是：让服务器自己选择一个随机队列给我们。  

再者，当我们的消费者程序断开连接时，这个队列要能自动的删除。  

消费端队列设置：
```go
    // 声明一个队列
	q, err := ch.QueueDeclare(
		"",				// 无名字
		false,
		false,
		true,			// 不再独占该队列
		false,
		nil,
    )
    // 绑定队列
	err = ch.QueueBind(
		q.Name,
		"",
		"logs",
		false,
		nil,
		)
```


生产端只需要创建对应的交换机，在Publish的时候往该交换机Publish即可！  

此时的结构：  

![](../images/mq/mq-16.png)  

## 二 完整代码实现

### 2.1 生产端
```go
package main

import (
	"flag"
	"fmt"
	"github.com/streadway/amqp"
	"log"
)

func main()  {

	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	errorHandle("ramqp dial err", err)
	defer conn.Close()

	ch, err := conn.Channel()
	errorHandle("get channel err", err)

	// 获取channel后自定义一个交换机
	err = ch.ExchangeDeclare(
		"logs",		// 交换机名称
		"fanout",		// 交换机类型，该类型适合发布订阅
		true,
		false,
		false,
		false,
		nil,
		)
	errorHandle("get exchange err", err)

	// 发送消息:go run producer.go -str=test...
	arg := flag.String("str", "0", "args name...")
	flag.Parse()
	body := *arg
	fmt.Println("body==", body)
	err = ch.Publish(
		"logs",			// publish的时候设置exchange
		"",
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
	"fmt"
	"github.com/streadway/amqp"
	"log"
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

	err = ch.ExchangeDeclare(
		"logs",   // name
		"fanout", // type
		true,     // durable
		false,    // auto-deleted
		false,    // internal
		false,    // no-wait
		nil,      // arguments
	)

	q, err := ch.QueueDeclare(
		"",				// 无名字
		false,
		false,
		true,			// 不再独占该队列
		false,
		nil,
	)
	errorHandle("declare a queue err", err)

	// 绑定队列
	err = ch.QueueBind(
		q.Name,
		"",
		"logs",	// exchange 名称
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
		for d := range msgs {
			log.Printf(" [x] %s", d.Body)
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