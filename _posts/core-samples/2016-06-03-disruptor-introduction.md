---
layout: post
date : 2016-06-03 15:15
title : ［翻译］Disruptor介绍
category : lessons
tags : [disruptor, 并发框架]
postid : 1464939255275
---
{% include JB/setup %}

理解Disruptor是什么的最好方法就是拿它与我们非常熟悉的功能相似的东西做比较，例如Java的BlockingQueue。类似队列，Disruptor的主要功能是在线程间传输数据（例如消息或事件）。然而，为了与队列区分，disruptor提供了以下核心特性:

* 广播事件给消费者，通过消费者依赖图
* 为事件预配置内存
* 无锁

<!--break-->

##核心概念

在我们了解Disruptor如何运作之前，先明确一些概念的定义，这些概念会贯穿于整个文档和代码。对于那些DDD的夯实者，思考一下这些在Disruptor无所不在的概念。

* Ring Buffer：Ring Buffer作为Disruptor的主要内容，从3.0开始只负责存储或更新数据/事件。并且用户可以完全替代它来实现高级功能
* Sequence：Disruptor使用Sequence来标记哪一个组件是最新的。每一个消费者（EventProcessor）都维持着一个Sequence。大多数并发代码依靠这些Sequence值的移动，因此Sequence支持AtomicLong的很多特性。实际上，两者的区别是Sequence包含了额外的功能来防止Sequence之间的错误数据共享
* Sequencer：Sequencer是Disruptor的核心。两个实现类（Single producer，multi producer）实现了所有并发算法，为了高速、正确地在生产者和消费者之间传输数据
* Sequence Barrier：Sequence Barrier由Sequencer产出，并且包含了指向任意的消费者下的由Sequencer和Sequences发布的主Sequence的引用。它包含了一套逻辑，用于决定是否有事件需要消费者去处理
* Wait Strategy：Wait Strategy决定了一个消费者要等待一个什么样的事件。更多细节在关于自由锁的章节中展开
* Event：生产者和消费者之间传输的数据单位。Event由用户自由定义
* EventProcessor：以循环的方式处理事件，并拥有消费者的Sequence的操作权。还有一种BatchEventProcessor，是更加高效的实现
* EventHandler：一个接口，由用户实现来作为消费者
* Producer：由用户编写，把事件入队



###事件广播

事件广播是Disruptor和队列之间最大的不同。当你有多个消费者监听着同一个Disruptor，事件会被发布到所有监听着的消费者，而对于队列，一个事件只会发布给一个消费者。当你需要对同一数据进行并行的并相互独立的操作，那么Disruptor就是你所需要的。一个来自LMAX的权威例子就是，当我们有3个操作，分别为journalling（写日志文件），replication（发送数据到另一台机器并确保发送成功），business logic（业务处理）。使用WorkerPool来进行并行处理。

###消费者依赖图

为了支持真实业务应用在并行处理上的需求，必须支持消费者之间协调工作。提到上面的例子，直到journalling和replication消费者都完成了他们的任务，business logic才能执行。我们称这概念为gating或者更正确的说法是前置行为。gating发生在2个地方，1个是我们需要保证生产者不能过载消费者，这可以通过调用RingBuffer.addGatingConsumers()来增加相关消费者。2是构建一个包含Sequences的SequenceBarrier，并且必须先完成它们的处理。

###事件预设

Disruptor其中一个目标就是使用在低延迟的环境。在低延迟系统中，必须减少或移除内存的分配。在Java系统中，通过垃圾回收机制来达到减少延迟。（在低延迟C/C++系统中，笨重的内存分配机制）。

为了达到低延迟的目的，我们可以通过Disruptor为有需要的事件预先分配内存空间。我们构建好EventFactory并准备给RingBuffer里面的实体调用时，当发布新的数据到Disruptor时，api允许我们持有构建好的对象，从而可以调用对应的方法或更新对象的属性。Disruptor确保这些操作是并发安全的，只要它们被正确实现。

###无锁（lock-free）

另一个核心实现，对于渴望低延迟的系统尤其推崇，那就是大范围地使用无锁算法来实现Disruptor。 使用memory barries、compare-and-swap操作来确保所有内存数据的可见性、正确性。只有一种情况是真的需要使用锁的，那就是BlockingWaitStrategy。很多低延迟系统会使用忙等待（busy-wait）来避免使用同一个条件而导致的混乱，然而，在一些系统中，忙等待回导致性能退化，尤其是当cpu资源有严格的控制时，例如在虚拟环境下的web服务。

