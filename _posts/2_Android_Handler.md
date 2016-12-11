---
title: Android中的Handler消息处理机制
date: 2016-12-10 19:13:00
categories:
	- Android
tags:
	- 学习笔记
	- 多线程
---

![Handler消息处理机制](/assets/img/cover/handler.jpg)

关键字：Handler Looper 多线程
<!-- more --> 
封面图出自黑马的一位老师
<br/>   
## 概述
<br/>

> Handler的消息处理机制是为了解决Android多个线程操作UI带来的混乱和卡顿(看上去它确实是为此而存在的)

为什么这么说呢？虽然现在Android会禁止非主线程去更新UI，但是为什么要这样？

不从代码来看，试想一哈，学校的食堂窗口，如果学生们一起去挤去抢，那样会有多混乱，超低的效率，超随机的处理方式，可能有人等好久都吃不上饭了。

排队打饭，按顺序来就可以避免这种乱七八糟不可控制的情况。

这个就是规则的好处，而Android也是这样，为了避免多个线程去抢着更新UI，就直接规定非主线程不能更新UI，接着放出一个规则，非主线程要通过Handler来排队更新UI。

<br/>
**总结一下：**Android中的Handler消息处理机制，就类似排队打饭，只不过不是自己去排队了，而且变成了电话预约，打电话先排上队，然后等通知。就是这样，而放到代码中，这个排队的人就是叫 "Handler"，打出去的排队电话就是 "Message"，排队通道就是 "MessageQueue", 食堂大妈就是 "Looper"。

> 额，写完才发现，好像去医院挂号更加像这个，但鉴于敲代码的身体都很好，基本不去医院，那就打饭的例子好了。

<br/>
## 简略流程
<br/>

从以上图片和例子，先大体过一下简略的流程，然后再走较详细的。

一般在使用Handler发送处理消息的时候，就是通过Handler对象发送一个消息，然后它重写消息处理的方法，处理消息。

其实知道这些已经够了，内部的具体实现在用的时候一般是不用关心的，但它的想法是真的很好，通过规则避免问题，咱可以学习瞻仰下大神的想法。

> 重点来了，首先Handler对象调用**发消息**的方法(具体先不管)，接着消息跑到了消息**队列** MessageQueue，然后通过Looper不停的轮询，取出消息队列中的消息**丢还**给**发消息的Handler**去处理，Handler通过一些判断，使用**对应的方法**处理消息。

其实就是这样一个大概的流程，结合上边的例子可以看出这样做的好处，各个线程的需要呈现在用户眼前的任务不再混乱甚至卡顿，而是变成了单线处理方式，有规律有规则。就用这样一个简单的流程去解决，再次佩服！

<br/>
## 详细流程
<br/>

这里从创建Handler Looper MessageQueue开始，然后在较详细的走一圈发送消息的过程。

<br/>
### 创建和初始化

1. 使用Handler时，是由我们自己new出的Handler对象。
2. 在Handler的构造方法中，会有`mLooper = Looper.myLooper();`,这里会得到Looper。而这个方法中是通过`(Looper)sThreadLocal.get();`得到的Looper，是从当前线程获取的，然后去找`sThreadLocal`，发现是`Looper`类中的静态成员，而且是在加载时就已经初始化，返回去再看`(Looper)sThreadLocal.get();`，既然是`get()`得到的，那应该是有个`set()`或者类似的方法，继续找，发现它的`set()`方法，然后里边直接就`new Looper()；`。
3. 在Looper的构造方法中，MessageQueue初始化`mQueue = new MessageQueue()`。

由上可以看出，Handler是自己初始化的，而Looper和MessageQueue都是已经创建好的。

> 应用启动时，主线程启动，`ActivityThread`中的`main()`方法执行，接着 `Looper.prepareMainLooper()`被调用，然后调用`prepare()`，调用了`上边2`那里那个`set()`，初试化了Looper和MessageQueue，在这之后，会在`main()`中继续调用`Looper.loop()`，去直接开始**轮询**MessageQueue中的消息，在这里，它会取出消息`Message msg = queue.next(); // might block`(可能会阻塞，后边再说)
，取出之后再解析分发消息，**消息由哪个Handler来的，就把消息给它分发处理**`msg.target.dispatchMessage(msg);`,这里这个`target`来自Message对象的`target`属性，用于记录是哪个Handler所属的消息。

<br/>
### 发送消息

1. 调用Handler的`obtain()`，得到Message对象。
2. 设置消息内容。
3. 调用Handler的`sendMessage()`发送消息，经过`sendMessageDelayed()`到`sendMessageAtTime()`，在这里，MessageQueue的对象给这个消息排上队`queue.enqueueMessage(msg, uptimeMillis)`,给Message的对象target赋值`msg.target = this` 使 target代表当前的 Handler对象。
4. `enqueueMessage()`中，通过`when`(上边的"uptimeMillis"指的是消息的时间)把消息放到相应的位置(消息队列是一个单链表)。接着`if (needWake) {nativeWake(mPtr);}`(一个进程间通信机制：管道（pipe）。主线程Looper从消息队列读取消息，当读完所有消息时，进入睡眠，主线程阻塞。子线程往消息队列发送消息，并且往管道文件写数据，主线程即被唤醒，从管道文件读取数据。)保证轮询状态，接着`loop()`中调用Message
的target(发送消息的Handler对象)的`dispatchMessage()`，让发出这个消息的Handler去分发消息处理。

5. 
```java
	if (msg.callback != null) {
		handleCallback(msg);
	} else {
		if (mCallback != null) {
			if (mCallback.handleMessage(msg)) {
				return;
			}
		}
		handleMessage(msg);
	}
```
	判断 msg是否设置了callback(Runnable对象，在得到msg时设置，重写run()方法)。

	设置了 就调用handleCallback()调用run方法。

	没有设置 就判断 当前Handler是否设置了mCallback(Callback对象，在创建Handler时设置，重写handlerMessage方法，常用).

	设置了就调用这个对象的 handlerMessage方法

	都没有 就调用 Handler 类的 handlerMessage方法。
	
6. 处理完继续轮询。

<br/>
*完*
<br/>

> 总结:
> 多么厉害的想法，简单有效的规程处理规避了多线程带来的问题，强无敌！作为一个渣渣，表示一下对大神的膜拜！ **b(￣▽￣)d**