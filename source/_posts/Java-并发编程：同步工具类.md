---
layout: posts
title: Java 并发编程：同步工具类
date: 2019-04-05 11:06:44
categories: Java 多线程
tags: [Java,多线程]
---

# 前言

最近在项目中看有用到CyclicBarrier类，为了明白这个类的具体用法，查了一下相关的资料。在JDK 1.5中提供了几个辅助类帮助我们进行多线程并发编程，学习了一下CountDownLatch、CyclicBarrier、Semaphore。

# CountDownLatch

> 一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。

用给定的计数实例化 `CountDownLatch`。在当前计数到达零之前，`await()` 方法会一直受阻塞，调用`countDown()`方法可以减少计数。当计数到达零之后，会释放所有等待的线程，`await()` 的所有后续调用都将立即返回。这种现象只出现一次——计数无法被重置。如果需要重置计数，请考虑使用 `CyclicBarrier`。

根据JDK API写的测试代码及结果：

```java
public class MyTest {
	
	public static void main(String[] args) throws InterruptedException {
		CountDownLatch startCountDownLatch = new CountDownLatch(1);
		CountDownLatch doneCountDownLatch = new CountDownLatch(5);
		ExecutorService threadPool = Executors.newFixedThreadPool(5);
		
		
		for (int i = 0; i < 5; i++) {
			threadPool.execute(new Worker(startCountDownLatch, doneCountDownLatch));
		}
		
		Thread.sleep(2000);
		startCountDownLatch.countDown();
		doneCountDownLatch.await();		//等待doneCountDownLatch计数到达零
		System.out.println("Main " + "aftter doneCountDownLatch await" );
	}
	
	
	static class Worker implements Runnable{
		private CountDownLatch startCountDownLatch;
		private CountDownLatch doneCountDownLatch;
		
		public Worker(CountDownLatch startCountDownLatch,CountDownLatch doneCountDownLatch) {
			this.startCountDownLatch = startCountDownLatch;
			this.doneCountDownLatch = doneCountDownLatch;
		}
		
		public void run() {
			try {
				System.out.println("Worker " + Thread.currentThread() + " before startCountDownLatch await" );
				startCountDownLatch.await();	//等待startCountDownLatch计数到达零
				System.out.println("Worker " + Thread.currentThread() + " after startCountDownLatch await" );
				doneCountDownLatch.countDown();
			} catch (Exception e) {
				e.printStackTrace();
			}	
		}
	}
}
```

```java
Worker Thread[pool-1-thread-2,5,main] before startCountDownLatch await
Worker Thread[pool-1-thread-5,5,main] before startCountDownLatch await
Worker Thread[pool-1-thread-1,5,main] before startCountDownLatch await
Worker Thread[pool-1-thread-3,5,main] before startCountDownLatch await
Worker Thread[pool-1-thread-4,5,main] before startCountDownLatch await
Worker Thread[pool-1-thread-2,5,main] after startCountDownLatch await
Worker Thread[pool-1-thread-3,5,main] after startCountDownLatch await
Worker Thread[pool-1-thread-5,5,main] after startCountDownLatch await
Worker Thread[pool-1-thread-1,5,main] after startCountDownLatch await
Worker Thread[pool-1-thread-4,5,main] after startCountDownLatch await
Main aftter doneCountDownLatch await
```

方法说明：
```java
CountDownLatch(int count)  //构造一个用给定计数初始化的 `CountDownLatch`。
void await() //使当前线程在锁存器倒计数至零之前一直等待，除非当前线程被中断。
boolean await(long timeout, TimeUnit unit) //使当前线程在锁存器倒计数至零之前一直等待，除非当前线程被或超出了指定的等待时间。
void countDown() //递减锁存器的计数，如果计数到达零，则释放所有等待的线程。
long getCount() //返回当前计数。
String toString() //返回标识此锁存器及其状态的字符串。
```


# CyclicBarrier

> 一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为*循环* 的 barrier。
>
> `CyclicBarrier` 支持一个可选的 `Runnable`命令，在一组线程中的最后一个线程到达之后（但在释放所有线程之前），该命令只在每个屏障点运行一次。若在继续所有参与线程之前更新共享状态，此*屏障操作* 很有用。

代码示例及结果：

```java
public class CyclicBarrierTest {
	private static int[][] data;
	private static CyclicBarrier cyclicBarrier;
	private static int[] rawResult;
	public static void main(String[] args) {
		data = new int[][]{{1,2,3},{4,5,6},{7,8,9},{10,11,12,13},{100,200,300,400,500}};
		rawResult = new int[data.length];
		cyclicBarrier = new CyclicBarrier(data.length, new Runnable() {
			
			@Override
			public void run() {
				int sum = 0;
				for (int i = 0; i < rawResult.length; i++) {
					sum += rawResult[i];
				}
				System.out.println("当前线程 " + Thread.currentThread() + "数据处理结果：" + sum);
			}
		});
		ExecutorService threadPool = Executors.newFixedThreadPool(5);
		CyclicBarrierTest cyclicBarrierTest = new CyclicBarrierTest();
		for (int i = 0; i < data.length; i++) {
			threadPool.execute(cyclicBarrierTest.new Worker(i));
		}
	}
	
	class Worker implements Runnable{
		private int i;
		
		public Worker(int i) {
			this.i = i;
		}
		
		public void run() {
			try {
				System.out.println("Worker " + i + "开始处理第" + i + "行数据 " + Thread.currentThread());
				int sum = processRow();
				System.out.println("Worker " + i + "处理完" + i + "行数据,结果：" + sum);
				cyclicBarrier.await();	//当所有线程到达此处时，屏障打开，才能继续执行下面代码
				System.out.println("Worker " + i + "执行其他任务");
			} catch (InterruptedException e) {
				e.printStackTrace();
			} catch (BrokenBarrierException e) {
				e.printStackTrace();
			}
		}
		
		/**
		 * 模拟耗时处理行数据
		 */
		private int processRow() {	
			try {
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			int[] raw = data[i];
			int sum = 0;
			for (int i = 0; i < raw.length; i++) {
				sum += raw[i];
			}
			rawResult[i] = sum;
			return sum;
		}
	}
}
```

```java
Worker 1开始处理第1行数据 Thread[pool-1-thread-2,5,main]
Worker 3开始处理第3行数据 Thread[pool-1-thread-4,5,main]
Worker 2开始处理第2行数据 Thread[pool-1-thread-3,5,main]
Worker 0开始处理第0行数据 Thread[pool-1-thread-1,5,main]
Worker 4开始处理第4行数据 Thread[pool-1-thread-5,5,main]
Worker 3处理完3行数据,结果：46
Worker 1处理完1行数据,结果：15
Worker 0处理完0行数据,结果：6
Worker 2处理完2行数据,结果：24
Worker 4处理完4行数据,结果：1500
当前线程 Thread[pool-1-thread-5,5,main]数据处理结果：1591
Worker 4执行其他任务
Worker 3执行其他任务
Worker 0执行其他任务
Worker 1执行其他任务
Worker 2执行其他任务
```

方法说明：

```java 
CyclicBarrier(int parties) //创建一个新的 `CyclicBarrier`，它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在启动 barrier 时执行预定义的操作。
CyclicBarrier(int parties, Runnable barrierAction)//创建一个新的 `CyclicBarrier`，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动 barrier 时执行给定的屏障操作，该操作由最后一个进入 barrier 的线程执行。
int await()//在所有参与者都已经在此 barrier 上调用 `await` 方法之前，将一直等待。
int await(long timeout,TimeUnit unit)throws InterruptedException,
                 BrokenBarrierException,
                 TimeoutException//在所有参与者都已经在此屏障上调用 await 方法之前将一直等待,或者超出了指定的等待时间。
int getNumberWaiting()// 返回当前在屏障处等待的参与者数目。
int getParties()//返回要求启动此 barrier 的参与者数目。
boolean isBroken()//查询此屏障是否处于损坏状态。
void reset()//将屏障重置为其初始状态。将屏障重置为其初始状态。如果所有参与者目前都在屏障处等待，则它们将返回，同时抛出 BrokenBarrierException。
```

注意：CyclicBarrier 可以重复使用(不调用`reset()`)也可以重复使用，而CountDownLatch只能使用一次。

# Semaphore 

> 一个计数信号量。从概念上讲，信号量维护了一个许可集。如有必要，在许可可用前会阻塞每一个 `acquire()`，然后再获取该许可。每个 `release()`添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，`Semaphore` 只对可用许可的号码进行计数，并采取相应的行动。

下面代码模拟了这样一个场景，一个厕所有3个坑位，有10个人排队蹲坑：

```java
public class SemaphoreTest {
	public static void main(String[] args) {
		Semaphore semaphore = new Semaphore(3);		//3个坑位
		SemaphoreTest semaphoreTest = new SemaphoreTest();
		for (int i = 0; i < 10; i++) {
			new Thread(semaphoreTest.new Person(semaphore, i)).start();
		}	
	}
	class Person implements Runnable{
		private Semaphore semaphore;
		private int num;
		public Person(Semaphore semaphore,int num) {
			this.semaphore = semaphore;
			this.num = num;
		}
		@Override
		public void run() {
			try {
				semaphore.acquire();	//等待坑位
				System.out.println("第"+ num +"个人抢到一个坑");
				Thread.sleep(1000 * num);
				System.out.println("第"+ num +"个人让出一个坑");
				semaphore.release();	//让出坑位
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}
```

运行结果：

```java
第1个人抢到一个坑
第2个人抢到一个坑
第0个人抢到一个坑
第0个人让出一个坑
第8个人抢到一个坑
第1个人让出一个坑
第3个人抢到一个坑
第2个人让出一个坑
第4个人抢到一个坑
第3个人让出一个坑
第5个人抢到一个坑
第4个人让出一个坑
第6个人抢到一个坑
第8个人让出一个坑
第7个人抢到一个坑
第5个人让出一个坑
第9个人抢到一个坑
第6个人让出一个坑
第7个人让出一个坑
第9个人让出一个坑
```

方法说明：

```java
public Semaphore(int permits)//参数permits表示许可数目，即同时可以允许多少线程进行访问
public Semaphore(int permits, boolean fair)//这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
public void acquire() throws InterruptedException //获取一个许可（阻塞）
public void acquire(int permits) throws InterruptedException//获取permits个许可（阻塞）
public void release()//释放一个许可(阻塞)
public void release(int permits)//释放permits个许可(阻塞)
public boolean tryAcquire()//尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException//尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(int permits)//尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException//尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public int availablePermits() //返回此信号量中当前可用的许可数。
  
```

# 总结

CountDownLatch 用于程等待其他所有线程执行完一些任务之后，继续执行其他代码；

CyclicBarrier 用于一组线程到达某个点之后同时继续执行后续代码；

```
public Semaphore(int permits)//参数permits表示许可数目，即同时可以允许多少线程进行访问
public Semaphore(int permits, boolean fair)//这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
public void acquire() throws InterruptedException //获取一个许可（阻塞）
public void acquire(int permits) throws InterruptedException//获取permits个许可（阻塞）
public void release()//释放一个许可(阻塞)
public void release(int permits)//释放permits个许可(阻塞)
public boolean tryAcquire()//尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException//尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(int permits)//尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException//尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public int availablePermits() //返回此信号量中当前可用的许可数。
  
```

# 总结

CountDownLatch 用于程等待其他所有线程执行完一些任务之后，继续执行其他代码；

CyclicBarrier 用于一组线程到达某个点之后同时继续执行后续代码；

```
public Semaphore(int permits)//参数permits表示许可数目，即同时可以允许多少线程进行访问
public Semaphore(int permits, boolean fair)//这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
public void acquire() throws InterruptedException //获取一个许可（阻塞）
public void acquire(int permits) throws InterruptedException//获取permits个许可（阻塞）
public void release()//释放一个许可(阻塞)
public void release(int permits)//释放permits个许可(阻塞)
public boolean tryAcquire()//尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException//尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(int permits)//尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException//尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public int availablePermits() //返回此信号量中当前可用的许可数。
  
```

# 总结

CountDownLatch 用于程等待其他所有线程执行完一些任务之后，继续执行其他代码；

CyclicBarrier 用于一组线程到达某个点之后同时继续执行后续代码；

Semaphore 用于一组线程争抢某些有限资源。