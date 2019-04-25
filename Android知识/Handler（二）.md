# Handler
## 工作原理
### 工作流程解析
Handler机制的工作流程主要包括4个步骤：

1. 异步通信准备
2. 消息发送
3. 消息循环
4. 消息处理

![](https://github.com/jy1314/Android-Knowledge/blob/master/util/picture/handler1.jpg )
### 工作流程图
![](https://github.com/jy1314/Android-Knowledge/blob/master/util/picture/handler2.jpg )
### 示意图
![](https://github.com/jy1314/Android-Knowledge/blob/master/util/picture/handler3.jpg )

### 注意
线程（Thread）、循环器（Looper）、处理者（Handler）之间的对应关系如下：

1. 1个线程（Thread）只能绑定 1个循环器（Looper），但可以有多个处理者（Handler）
2. 1个循环器（Looper） 可绑定多个处理者（Handler）
3. 1个处理者（Handler） 只能绑定1个循环器（Looper）
