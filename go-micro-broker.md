## Go Micro中的broker用于代理消息的发布与订阅

### 内容
- 基于go micro 编写一个最基本的发布订阅功能.

- 一个定时的发布者, 每隔一秒给主题发布一个消息. 两个协程的订阅者, 分别订阅发布者发布的内容.

### 发布者
``` golang
// 创建一个发布者, 并每秒钟给主题发送一次信息
func pub(){
	// 创建一个每秒钟执行的定时器
	tick := time.NewTicker(time.Second)
	i := 0
	// 定时器开始执行
	for range tick.C {
		// 创建一个消息
		msg := &broker.Message{
			Header: map[string]string{
				"id":fmt.Sprintf("%d", i),
			},
			Body:[]byte(fmt.Sprintf("%d:%s", i, time.Now().String())),
        }
        // 打印 broker
		log.Info(broker.String())
		// 发布消息
		if err := broker.Publish(topic, msg); err != nil {
			log.Info("[pub] Message publication failed: %v", err)
		} else {
			fmt.Println("[pub] Message published: ", string(msg.Body))
		}
		i++
	}
}
```

#### broker 的 [Message](https://godoc.org/github.com/micro/go-micro/broker#Message) 类型定义. 
- `Header` 消息头
- `Body` 消息体

```
type Message struct {
    Header map[string]string
    Body   []byte
}
```

### 消费者(订阅者)
> 消费者负责接收消息, 并处理消息.

```golang
// 一个订阅
func sub()  {
	// 订阅消息
	_, err := broker.Subscribe(topic, func(p broker.Event) error {
		log.Infof("[sub] Received Body: %s, Header: %s\n", string(p.Message().Body), p.Message().Header)
		return nil
	})

	if err != nil {
		fmt.Println(err)
	}
}
```

```
// 另一个订阅
func sub2()  {
	_, err := broker.Subscribe(topic, func(event broker.Event) error {
		log.Infof("[sub2] Received Body: %s, Header: %s\n", string(event.Message().Body), event.Message().Header)
		return nil
	})

	if err != nil {
		fmt.Println(err)
	}
}

```

### man 函数
> 发布者和订阅者都已经有了, 那么可以编写启动的代码.

``` golang
func main() {
	// cmd.Init() 处理环境变量和命令行变量.

    cmd.Init()
    // broker 初始化
	if err := broker.Init(); err != nil {
		log.Fatalf("broker.Init() error :%v", err)
	}

    // 链接
	if err := broker.Connect(); err != nil {
		log.Fatalf("broker.Connect() error:%v", err)
	}

	go pub() // 协程启动发布者
	go sub() // 协程启动订阅1
	go sub2() // 协程启动订阅2

    // 执行20‘s
	<- time.After(time.Second * 20)
}
```

### 所有代码

``` golang
package main

import (
	"fmt"
	"github.com/micro/go-micro/v2/config/cmd"
	"github.com/micro/go-micro/v2/util/log"
	"time"
	"github.com/micro/go-micro/v2/broker"
)

var (
	topic = "go.micro.learning.topic.log"
	b     broker.Broker
)

func pub(){
	// 创建一个每秒钟执行的定时器
	tick := time.NewTicker(time.Second)
	i := 0
	// 定时器开始执行
	for range tick.C {
		// 创建一个消息
		msg := &broker.Message{
			Header: map[string]string{
				"id":fmt.Sprintf("%d", i),
			},
			Body:[]byte(fmt.Sprintf("%d:%s", i, time.Now().String())),
		}
		log.Info(broker.String())
		// 发布消息
		if err := broker.Publish(topic, msg); err != nil {
			log.Info("[pub] Message publication failed: %v", err)
		} else {
			fmt.Println("[pub] Message published: ", string(msg.Body))
		}
		i++
	}
}

// 一个订阅
func sub()  {
	// 订阅消息
	_, err := broker.Subscribe(topic, func(p broker.Event) error {
		log.Infof("[sub] Received Body: %s, Header: %s\n", string(p.Message().Body), p.Message().Header)
		return nil
	})

	if err != nil {
		fmt.Println(err)
	}
}

// 另一个订阅
func sub2()  {
	_, err := broker.Subscribe(topic, func(event broker.Event) error {
		log.Infof("[sub2] Received Body: %s, Header: %s\n", string(event.Message().Body), event.Message().Header)
		return nil
	})

	if err != nil {
		fmt.Println(err)
	}
}

func main() {
	cmd.Init()
	if err := broker.Init(); err != nil {
		log.Fatalf("broker.Init() error :%v", err)
	}

	if err := broker.Connect(); err != nil {
		log.Fatalf("broker.Connect() error:%v", err)
	}

	go pub()
	go sub()
	go sub2()

	<- time.After(time.Second * 20)
}

```

### 程序执行结果

```
2020-04-02 22:04:53  level=info [eats]
[pub] Message published:  0:2020-04-02 22:04:53.739508 +0800 CST m=+1.146087208
2020-04-02 22:04:53  level=info [sub] Received Body: %s, Header: %s
[0:2020-04-02 22:04:53.739508 +0800 CST m=+1.146087208 map[id:0]]
2020-04-02 22:04:53  level=info [sub2] Received Body: %s, Header: %s
[0:2020-04-02 22:04:53.739508 +0800 CST m=+1.146087208 map[id:0]]
2020-04-02 22:04:54  level=info [eats]
[pub] Message published:  1:2020-04-02 22:04:54.739537 +0800 CST m=+2.146086147
2020-04-02 22:04:54  level=info [sub] Received Body: %s, Header: %s
[1:2020-04-02 22:04:54.739537 +0800 CST m=+2.146086147 map[id:1]]
2020-04-02 22:04:54  level=info [sub2] Received Body: %s, Header: 

.....
```