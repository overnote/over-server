## 一 Topic模式的引入

direct类型的交换机无法配置多重路由。在日志系统中，如果想不仅能依据日志级别来定义，也能根据发送日志的源信息来订阅：
- 根据日志级别：`info/warn/crit`
- 根据来源不同：`auth/cron/kern`（即来自设备kern）

topic类型的交换机与binding key没有太大区别，逻辑与direct一样，接收到的消息会分发到所有与其routing key匹配的绑定队列。  

交换机名称定义规范如下：
- 单词之间以 `.` 隔开，如："stock.usd.nyse"
- `*` 表示一个单词
- `#` 表示0或多个单词

如图所示：  

![](../../images/mq/rabbitmq-19.png)  

上图中，消息的routing key用三个单词来表示，依次表示速度、颜色、种类，其形式如："<speed>.<color>.<species>”。  

图中有三个绑定：Q1的绑定键是"*.orange.*", Q2的绑定键是"*.*.rabbit"和"lazy.#"。  

```
Q1对orange颜色的动物感兴趣；
Q2则对所有物种是rabbit的、速度是lazy的所有动物感兴趣
```

举例来说明带有不同routing key的消息会被路由到哪个队列：
```
"quick.orange.rabbit":      分发到Q1和Q2；
"lazy.orange.elephant":     分发到Q1和Q2;
"quick.orange.fox":         分发到Q1；
"lazy.brown.fox":           分发到Q2;
"lazy.pink.rabbit":         分发到Q2，且只会分发一次，虽然它匹配了Q2的两个绑定；
"quick.brown.fox":          Q1、Q2都不分发，因为没有与之匹配的绑定规则，将会被忽略；
"orange":                   无匹配规则，丢失
"quick.orange.male.rabbit": 无匹配规则，丢失
"lazy.orange.male.rabbit":  分发到Q2，虽然有4个单词，但符合"lazy.#"规则的通配符
```

有图Topic类型的交换机相当灵活，其实也可以变相实现其他功能，如发布订阅，即将"#"指定为绑定键，那么就会接收所有的消息，相当于fanout类型的广播；

## 二 Topic模式完整代码

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
		"logs_topic",
		"topic",				// 修改为 topic 类型
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
		"logs_topic",
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
		"logs_topic",
		"topic",
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