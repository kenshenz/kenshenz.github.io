---
layout: post
date : 2016-06-04 17:11
title : ［译］Disruptor入门
category : lessons
tags : [disruptor, 并发框架]
postid : 1465031502280
---
{% include JB/setup %}

##Disruptor入门

Disruptor的jar包可以从Maven Central下载。或者也可以从我们wiki里 的下载页面下载项目包。

###基本的事件生产者和消费者

在开始学习Disruptor之前，我们先考虑一个非常简单的情景，我们要从生产者传输一个长整型数据到消费者，然后消费者把数据打印出来。首先，我们需要订一个带有长整型数据的事件类型。

<!--break-->

```
public class LongEvent
{
    private long value;

    public voidset(long value)
    {
        this.value = value;
    }
}
```

为了让Disruptor帮事件预分配内存空间，我们需要一个EventFactory来构建事件。

```
importcom.lmax.disruptor.EventFactory;

public class LongEventFactory implements EventFactory<LongEvent>
{
    public LongEvent newInstance()
    {
        return new LongEvent();
    }
}
```

定义好事件之后，我们还需要创建一个消费者来处理事件。在我们的例子中，我们希望消费者在控制台上打印数据。

```
importcom.lmax.disruptor.EventHandler;

public class LongEventHandler implements EventHandler<LongEvent>
{
    public void onEvent(LongEvent event, long sequence, boolean endOfBatch)
    {
        System.out.println("Event: "+ event);
    }
}
```

我们还需要一个事件源，这里先假设数据来自某些有序的I/O设备，例如由ByteBuffer构成的网络数据或文件数据。

```
importcom.lmax.disruptor.RingBuffer;

public class Long EventProducer
{
    private final RingBuffer<LongEvent> ringBuffer;

    public LongEvent Producer(RingBuffer<LongEvent> ringBuffer)
    {
        this.ringBuffer = ringBuffer;
    }

    public void onData(ByteBufferbb)
    {
        long sequence = ringBuffer.next();  // Grab the next sequencetry
        {
            LongEvent event = ringBuffer.get(sequence); // Get the entry in the Disruptor// for the sequence
            event.set(bb.getLong(0));  // Fill with data
        }
        finally
        {
            ringBuffer.publish(sequence);
        }
    }
}
```

很明显，使用一个队列，会使得事件发布变得复杂。之所以这么做是为了可以为事件预分配内存空间。这里最起码有两个步骤来发布消息；第一步是请求获取一个RingBuffer序列位，第二步是发送数据。还有，我们需要把发布消息放置在try/finally代码块中。如果获取到一个RingBuffer序列位（调用RingBuffer.next()），那么我们必须把这个序列位发布。如果没有这样子做会导致Disruptor的状态混乱。特别是在多生产者的场景中，会导致消费者停止工作，并且只有重启程序才能恢复。

###使用第三版转换器

Disruptor3.0中，新增了一个功能非常丰富的Lambda风格的api，它帮开发者把复杂的东西都封装在RingBuffer里面，因此在3.0版本中，发布消息的更佳方式是使用Event Publisher或Event Translator API，例如：

```
import com.lmax.disruptor.RingBuffer;
import com.lmax.disruptor.EventTranslatorOneArg;

public class LongEventProducerWithTranslator
{
    private final RingBuffer<LongEvent> ringBuffer;

    public LongEventProducerWithTranslator(RingBuffer<LongEvent> ringBuffer)
    {
        this.ringBuffer = ringBuffer;
    }

    private static final EventTranslatorOneArg<LongEvent, ByteBuffer> TRANSLATOR =
        new EventTranslatorOneArg<LongEvent, ByteBuffer>()
        {
            public void translateTo(LongEvent event, long sequence, ByteBuffer bb)
            {
                event.set(bb.getLong(0));
            }
        };

    public void onData(ByteBuffer bb)
    {
        ringBuffer.publishEvent(TRANSLATOR, bb);
    }
}
```

这种方式的好处是转换器的代码可以放在独立的class文件中，测试起来更容易。Disruptor提供了不同的接口（EventTranslator，EventTranslatorOneArg，EventTranslactorTwoArg等等）。之所以允许转换器定义为静态类或非捕获lambda是因为转换器所需要的参数是通过调用RingBuffer传到转换器的。
最后一步来把所有东西都整合在一起。Disruptor提供了一个DSL来简化构建。

```
import com.lmax.disruptor.dsl.Disruptor;
import com.lmax.disruptor.RingBuffer;
import java.nio.ByteBuffer;
import java.util.concurrent.Executor;
import java.util.concurrent.Executors;

public class LongEventMain
{
    public static void main(String[] args) throws Exception
    {
        // Executor that will be used to construct new threads for consumers
        Executor executor = Executors.newCachedThreadPool();

        // The factory for the event
        LongEventFactory factory = new LongEventFactory();

        // Specify the size of the ring buffer, must be power of 2.
        int bufferSize = 1024;

        // Construct the Disruptor
        Disruptor<LongEvent> disruptor = new Disruptor<>(factory, bufferSize, executor);

        // Connect the handler
        disruptor.handleEventsWith(new LongEventHandler());

        // Start the Disruptor, starts all threads running
        disruptor.start();

        // Get the ring buffer from the Disruptor to be used for publishing.
        RingBuffer<LongEvent> ringBuffer = disruptor.getRingBuffer();

        LongEventProducer producer = new LongEventProducer(ringBuffer);

        ByteBuffer bb = ByteBuffer.allocate(8);
        for (long l = 0; true; l++)
        {
            bb.putLong(0, l);
            producer.onData(bb);
            Thread.sleep(1000);
        }
    }
}
```

###基本的调整选项

使用上面的方式可以再大多数情景下完成工作。然而，如果你可以确认Disruptor将运行在什么样的软件和硬件环境下，那么可以通过调整部分选项来提升性能。这里有两个主要的选项，单生产者抑或多生产者，还有等待策略。

###单生产者 vs 多生产者

并发系统中提升性能的最好方式之一是坚持单独写原则（Single Writer Princple）。如果情景中只会有一个线程来生产事件，那你就可以通过调整选项来获得额外的性能。

```
public class LongEventMain
{
    public static void main(String[] args) throws Exception
    {
        //.....
        // Construct the Disruptor with a SingleProducerSequencer
        Disruptor<LongEvent> disruptor = new Disruptor(factory, 
                                                       bufferSize,
                                                       ProducerType.SINGLE, // Single producer
                                                       new BlockingWaitStrategy(),
                                                       executor);

        //.....
    }
}
```

为了支出通过这样的调整可以获得多少性能的提升，我们来做一下比较，测试运行在i7 Sandy Bridge MacBook Air。

多生产者

```
Run 0, Disruptor=26,553,372 ops/sec
Run 1, Disruptor=28,727,377 ops/sec
Run 2, Disruptor=29,806,259 ops/sec
Run 3, Disruptor=29,717,682 ops/sec
Run 4, Disruptor=28,818,443 ops/sec
Run 5, Disruptor=29,103,608 ops/sec
Run 6, Disruptor=29,239,766 ops/sec
```

单生产者

```
Run 0, Disruptor=89,365,504 ops/sec
Run 1, Disruptor=77,579,519 ops/sec
Run 2, Disruptor=78,678,206 ops/sec
Run 3, Disruptor=80,840,743 ops/sec
Run 4, Disruptor=81,037,277 ops/sec
Run 5, Disruptor=81,168,831 ops/sec
Run 6, Disruptor=81,699,346 ops/sec
```

###可供选择的等待策略

Disruptor使用的默认等待测试时BlockingWaitStrategy。实际上，BlockingWaitStrategy使用了一个标准锁和条件变量来控制唤醒线程。BlockingWaitStrategy是最慢的的一个等待策略，但也是基本的适合大多数配置的一个策略。然而，如果对部署环境足够了解，则可以通过调优来获得更多性能上的提升。

####SleepingWaitStrategy

类似BlockingWaitStrategy，SleepingWaitStrategy试图只使用CPU最基本的功能，他通过使用一个简单的忙等待循环，但在循环中调用了LockSupport.parkNanos(1)。在一台标准的Linux服务器中，每60µs（1ms=1000µs）暂停一次线程，对于生产者线程，它不需要去增加计数器，也不需要为条件变量标记信号。然而这也意味着，在生产者线程和消费者线程之间传输数据的代价会更高。这种策略适合用于不需要低延迟，对不会生产者线程有什么的影响/冲击的情境下。举个例子，异步日志。

####YieldingWaitStrategy

YieldingWaitStrategy是其中一个可以使用在低延迟系统的策略，前提是这些系统要有选项来超频CPU主频，来达到更低的等待时间。YieldingWaitStrategy会不断循环等待sequence去增加变量的值。在循环里，Thread.yield()会被调用，来唤醒其他队列线程。这是一个推荐的等待策略，当需要非常高的性能并且Event Handler线程少于逻辑处理线程，例如超线程（hyper-threading）的可以使用的时候。

####BusySpinWaitStrategy

BusySpinWaitStrategy是拥有最高性能的等待策略，但对部署环境的限制也是最多的。这个等待策略只应该在这种情景下使用，Event Handler线程数少于CPU上的物理核心数，例如，超线程不能使用的时候。
