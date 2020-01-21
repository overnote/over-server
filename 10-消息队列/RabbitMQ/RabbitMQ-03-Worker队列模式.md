## 一 工作队列

### 1.0 工作队列概述

工作队列模型： 一个生产者生产的消息，可以被多个消费者来消。  

在资源密集型任务场景中，可以将该类任务推入队列，由队列程序分发到在线的工作程序执行。  

![](../images/arch/mq-15.png)  

### 1.1 工作队列生产端
```go
	arg := flag.String("str", "0", "args name...")
	flag.Parse()
	body := *arg
	err = ch.Publish(
		"",     
		q.Name, 		
		false,  
		false, 
		amqp.Publishing {
			ContentType: "text/plain",
			Body:        []byte(body),
		})
```

### 1.2 工作队列消费端

```go
	go func() {
		for d := range msgs{
			log.Printf("Received a message: %s", d.Body)
			dot_count := bytes.Count(d.Body, []byte("."))
			t := time.Duration(dot_count)		// 利用参数中 . 的数目不同制作间隔
			time.Sleep(t* time.Second)
			log.Printf("Done")
		}
	}()
```

### 1.3 执行

```
# 启动第一个命令行，运行消费端
go run consumer.go

# 启动第二个命令行，运行消费端
go run consumer.go

# 启动第三个命令行，运行生产端
go run producer.go -str=test..
go run producer.go -str=test.....
```

我们会发现两个消费端，每人消费了一个消息。默认情况下，RabbitMQ会按收到的消息顺序依次发送到每一个消费者中，从总体上来看，每个消费者会收到同样多的消息。这种消息分发方式叫做round-robin（轮询调度）。 

## 二 公平分发

有时候队列的轮询调度并不能满足我们的需求，假设有这么一个场景，存在两个消费者程序，所有的单数序列消息都是长耗时任务而双数序列消息则都是简单任务，那么结果将是一个消费者一直处于繁忙状态而另外一个则几乎没有任务被挂起。  

导致这种情况发生的根本原因是RabbitMQ是根据消息的入队顺序进行派发，而并不关心在线消费者还有多少未确认的消息，它只是简单的将第N条消息分发到第N个消费者。 
```go
	err = ch.Qos(
		1,      // prefetch count
		0,      // prefetch size
		false,  // global
	)
```

关于队列的长度：如果所有的消费者都繁忙，队列可能会被消息填满。你需要注意这种情况，要么通过增加消费者来处理，要么改用其他的策略。  

### 三 完整的工作队列代码

### 2.1 生产端

```go
package main

import (
	"flag"
	"log"
	"github.com/streadway/amqp"
)

func main() {

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

	err = ch.Qos(
		1,      // prefetch count
		0,      // prefetch size
		false,  // global
	)

	// 发送消息:go run producer.go -str=test...
	arg := flag.String("str", "0", "args name...")
	flag.Parse()
	body := *arg
	err = ch.Publish(
		"",
		q.Name,
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
	"log"
	"github.com/streadway/amqp"
	"time"
)

func main()  {

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
