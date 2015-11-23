title: 并发策略
date: 2015-11-07 09:47:13
tags: [并发]
---
这里说的并发策略是指并发时的线程数和任务数。
我们将需要并发的任务分为io密集型和计算密集型，对于两种情况要分别对待

一、线程数
IO密集型
当一个线程执行io操作时，处理器核心会进行上下文切换，将该线程阻塞，执行其他就绪的线程，所以需要比处理器核心数更多的线程数，否则当线程阻塞时没有其他线程可以调用，从而浪费cpu时间。

计算密集型
当多个线程就绪时，处理器核心会频繁切换上下文，这种切换对性能损耗较大，所以将任务拆分后所有任务都是计算密集型的，则创建处理器数相同的线程数就可以了。

如果任务的阻塞时间大于执行时间（即阻塞时间大于50%）可认为是io密集型，否则是计算密集型，对于线程数有以下计算公式：
>线程数=CPU核心数/(1-阻塞系数)<br>
其中，阻塞系数=阻塞时间/执行时间

二、任务数
理想的状态是子任务数等于线程数就好了，并且我们希望任务拆分后各个子任务工作量是相等的。
但在实际开发中，将工作量均等分配到各个任务往往十分困难，所以一种简单粗暴的方式是使任务数比线程数更多。当任务足够多时，一个任务完成后立即切换下一个任务使所以线程一直繁忙，同时可以使用线程池来减少任务切换时线程创建和销毁的开销。

三、Demo
{% codeblock ConcurrentDemo.java %}
package com.climbran.base;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

/**
 * Created by swe on 15/11/6.
 */
public class IOConcurrentDemo {

    public void test(final int partitionsNum)
        throws InterruptedException,ExecutionException{
        //获取处理器核心数
        final int numberOfCores = Runtime.getRuntime().availableProcessors();

        final double blockingCoefficient = 0.99; //阻塞率
        final int poolSize = (int)(numberOfCores/(1-blockingCoefficient));

        System.out.println("Number of Cores avaliable is " + numberOfCores);
        System.out.println("Pool size is " +  poolSize);

        final long start = System.nanoTime();
        final List<Callable<Integer>> partitions = new ArrayList<Callable<Integer>>();
        for(int i = 1; i<= partitionsNum;i++){
            final int tmp = i;
            partitions.add(new Callable<Integer>() {
                public Integer call() throws Exception {
                    //并发任务,这里用Thread.sleep(2000)代替
                    Thread.sleep(2000);
                    return Integer.valueOf(tmp);
                }
            });
        }
        final ExecutorService executorPool = Executors.newFixedThreadPool(poolSize);
        final List<Future<Integer>> valueOfStocks = executorPool.invokeAll(partitions, 10000, TimeUnit.SECONDS);
        final long end = System.nanoTime();
        int sumValue = 0;
        for(final Future<Integer> valueOfStock : valueOfStocks)
            sumValue +=valueOfStock.get();
        executorPool.shutdown();
        System.out.println("value:"+sumValue);
        System.out.println("Time (seconds) taken is " + (end - start)/1.0e9);
    }

    public static void main(final String[] args)
            throws ExecutionException, InterruptedException, IOException {
        new IOConcurrentDemo().test(1000);
    }
}
{% endcodeblock %}

这是一个简单的io密集型任务，运行后输出：

	Number of Cores avaliable is 4
	Pool size is 399
	value:500500
	Time (seconds) taken is 6.057576742
输出的500500说明所有线程都有正确执行到，可以将其中的延时函数替换为网页访问、文件读取等操作，有兴趣的朋友也可以测试以下顺序访问结果...
对于计算密集型可能还需要一个拆分任务数作为参数，在实现拆分策略时使用，具体就不在这展示了。