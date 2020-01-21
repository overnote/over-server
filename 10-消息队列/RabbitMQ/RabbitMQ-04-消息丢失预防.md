## 一 确保消息不丢失

防止消息丢失有两个策略（必须都利用）：
- 消息确认：可以预防消费端宕机引起的消息丢失
- 消息持久化：可以预防RabbitMQ本身宕机造成的消息丢失

## 二 消息确认

RabbitMQ将消息发送到消费者后就会将其标记删除，而不会关心消费者程序是否执行完毕，如果消费端在处理任务时因为一些意外宕机（如处理耗时操作断网），这个消息就彻底丢失了！

如果我们不希望这种场景下的消息丢失，可以使用RabbitMQ提供的消息确认机制来确保每个消息都不会丢失。  

代码操作是：设置Consumer的ch属性`auto-ack`为false。 

```go
	msgs, err := ch.Consume(
		q.Name,
		"",
		false,		// 不再自动ack
		false,
		false,
		false,
		nil,
	)
```

执行原理是：
- RabbitMQ收到消费者发出的表明任务处理完毕的确认包（**ack**）后才会将改消息从队列中删除
- 若消费端未发出ack，则RabbitMQ会认为该消息没有被完全处理，会将其重新加入队列
- 此时如果其他消费者在线，就会将该消息消费处理掉

这时候上述工作队列的代码操作时，即使在执行时Ctrl+C关闭Consumer，也不会丢失消息。  

注意：
- 消息确认没有超时机制，RabbitMQ只会在消费者Down掉之后才会重新分发。
- 消息确认包的目的地必须是当前消息的接收通道，如果将确认包发送到其他通道时会引发异常。
- 消息成功消费后，必须调用`Ack()`方法告诉RabbitMQ消息已经消费，否则消费者退出后消息会重发，却永远没有确认删除的包，因此RabbitMQ消息越积越多就会吃掉越来越多的内存，最后可能导致崩溃。

消费端消费后删除ack方法：
```go
	go func() {
		for d := range msgs{
			log.Printf("Received a message: %s", d.Body)
			dot_count := bytes.Count(d.Body, []byte("."))
			t := time.Duration(dot_count)
			time.Sleep(t* time.Second)
			log.Printf("Done")
			d.Ack(false)			// 不要忘记对消息的确认
		}
	}()
```

## 三 消息持久化

消息确认机制可以让消息不会在消费端丢失，如果RabbitMQ本身崩溃了呢？此时除非明确指定，队列和消息也都会丢失。如果要做到这种情况下的不丢失，必须满足两个条件：
- 队列持久化：`q, err := ch.QueueDeclare("myqueue02", true....)`
  - 贴士：不能持久化多个相同名字的队列
  - 贴士：durable参数在生产者和消费者程序中都要指定为True。
- 消息持久化：只需要在Publishing函数中设置 `amqp.Publishing{ DeliveryMode: amqp.Persistent,....}`

## 四 无法保证消息的绝对不丢失

将消息设置为Persistent并不能百分百地完全保证消息不会丢失。虽然RabbitMQ知道要将消息写到磁盘，但在RabbitMQ接收到消息和写入磁盘前还是有个时间空档。  

因为RabbitMQ并不会对每一个消息都执行fsync(2),因此消息可能只是写入缓存而不是磁盘。所以Persistent选项并不是完全强一致性的，但在应付我们的简单场景已经足够。如需对消息完全持久化，可参考 [publisher confirms](https://www.rabbitmq.com/confirms.html)。  