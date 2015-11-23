title: 线程诱发型死锁
date: 2015-11-09 21:49:28
tags: [并发]
---
先看下面代码，是通过并发来计算文件大小：
{% codeblock lang:java %}
package com.climbran.totalFileSize;

import java.io.File;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

/**
 * Created by ywq on 15/11/7.
 */
public class NaivelyConcurrentTotalFileSIze {
    private long getTotalSizeOfFileInDir(ExecutorService service,final File file)
        throws Exception{
        if(file.isFile()) return file.length();

        final File[] children = file.listFiles();
        long total = 0;
        if(children!=null) {
            final List<Future<Long>> partialTotalFutures = new ArrayList<Future<Long>>();
            for (final File child : children)
                partialTotalFutures.add(service.submit(new Callable<Long>() {
                    @Override
                    public Long call() throws Exception {
                        return getTotalSizeOfFileInDir(service, child);
                    }
                }));

            for(final Future<Long> partoalTotalFuture : partialTotalFutures)
                total+=partoalTotalFuture.get(100, TimeUnit.SECONDS);
        }
        return total;
    }

    public long getTotalSizeOfFile(final String fileName)
        throws Exception    {
        final ExecutorService service = Executors.newFixedThreadPool(200);
        try {
            return getTotalSizeOfFileInDir(service,new File(fileName));
        }finally {
            service.shutdown();
        }
    }

    public static void main(final String[] args)
        throws Exception{
        final long start = System.nanoTime();
        final long totalSize = new NaivelyConcurrentTotalFileSIze().getTotalSizeOfFile("/Users");
        final long end = System.nanoTime();
        System.out.println("Total Size:" + totalSize);
        System.out.println("Cost Time(Second):"+(end-start)/1.0e9);

    }


}
{% endcodeblock %}

看起来似乎没什么问题，但当扫描目录下文件层次较深时可能出来

	Exception in thread "main" java.util.concurrent.TimeoutException
	
这是由于getTotalSiizeOfFilesInDir()中有阻塞线程的操作，当线程扫描其子目录时，改线程就会阻塞起来，当子目录数量不是很多时不会有什么问题，但是当目录层次很深且目录数大于线程数时，可能较外层的线程都被阻塞了，而子目录分配不到新的线程，从而进入死锁状态。

解决这个问题的一个方式是使深度优先遍历变为广度优先遍历，使每个任务返回当前目录下的文件(file)大小和该目录下的子目录(dir)集合，另一个方式是使用Fork-join API来管理线程，在Fork-join中任务被阻塞时，执行该任务的线程可以将任务挂起执行其他任务。