title:  内存栅栏(Memory Barrier)和volatile
date: 2015-11-06 23:32:06
tags: [java,并发]
---
内存栅栏是指本地或者工作内存到主存间的拷贝动作。在程序运行过程中。所有变更会先在线程的寄存器或本地cache中完成，然后才会拷贝到主存以跨越内存栅栏。理解内存栅栏先看下面代码：
{% codeblock lang:java %}
 public class RaceCondition{
 	private static boolean done;
 	
 	public static void main(final String[] args) throws InterruptedException{
 		new Thread(
 			public void run(){
 				int i = 0;
 				while(!done){ i++ ; }
 				System.out.println("Done!");
 			}
 		).start();
 		System.out.println("OS:" + System.getProperty("os.name"));
 		Thread.sleep(2000);
 		done = true;
 		System.out.println("flag done set to true");
 	}
}
{% endcodeblock %}
如果在win7上运行
	
	java RaceCondition
可能输出以下结果：
>OS: Windows 7<br>
flag done set to true<br>
Done!

而在Mac上运行同样命令，则可能是：
> OS: Mac OS X<br>
flag done set to true<br>

这是由于windows平台执行java 命令默认是以server模式，而Mac下默认是以client模式运行。
在server模式下，JIT编译器会对代码新线程里的while循环进行优化，新线程会从寄存器或本地cache中读取done的拷贝值，而不会去速度相对较慢的内存里读取。由于以上原因，主线程中对于done的变更在新线程中不可见。

为解决这个问题，可以使用**volatile**关键字，其作用是告知JIT编译器改变量可能被某个线程更改，不要对其进行优化。volatile可使对变量的每次读写都忽略本地cache直接对内存操作，但正是这样会使得每次访问变量都要跨越内存栅栏导致程序性能下降。

由于每个线程对volatile字段的访问都是独立处理的，所以volatile关键字无法保证读写操作的原子性，为解决这个问题，可以使用**synchronized**关键字标注的setter和getter方法，synchronized可使同步区块内跨越内存栅栏。
