## CountDownLatch

CountDownLatch是一个计数器闭锁，通过它可以完成类似于阻塞当前线程的功能，即：一个线程或多个线程一直等待，直到其他线程执行的操作完成。CountDownLatch用一个给定的计数器来初始化，该计数器的操作是原子操作，即同时只能有一个线程去操作该计数器。调用该类await方法的线程会一直处于阻塞状态，直到其他线程调用countDown方法使当前计数器的值变为零，每次调用countDown计数器的值减1。当计数器值减至零时，所有因调用await()方法而处于等待状态的线程就会继续往下执行。这种现象只会出现一次，因为计数器不能被重置，如果业务上需要一个可以重置计数次数的版本，可以考虑使用CycliBarrier。

在某些业务场景中，程序执行需要等待某个条件完成后才能继续执行后续的操作；典型的应用如并行计算，当某个处理的运算量很大时，可以将该运算任务拆分成多个子任务，等待所有的子任务都完成之后，父任务再拿到所有子任务的运算结果进行汇总。

![](https://upload-images.jianshu.io/upload_images/11464886-8d35a1901ff4e80c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/414/format/webp)

```java
class CountDownLatchDemo implements Runnable {

	private CountDownLatch latch;

	public CountDownLatchDemo(CountDownLatch latch) {
		this.latch = latch;
	}

	@Override
	public void run() {
		try {
			int num = 0;
			for (int i = 0; i < 50000; i++) {
				num += i;
			}
			
			System.out.println(Thread.currentThread().getName() + " = " + num);
		} finally {
            // 需要在try...finally中进行释放，因为如果抛异常，那么可能无法释放次数，导致latch.await();一直等待。
			latch.countDown();
		}
	}
}
```

Client：

```java
public static void main(String[] args) throws Exception {
		
		CountDownLatch latch = new CountDownLatch(5);
		
		CountDownLatchDemo demo = new CountDownLatchDemo(latch);
		
		long startTime = System.currentTimeMillis();
		
		for (int i = 0; i < 5; i++) {
			new Thread(demo).start();
		}
		
		latch.await();
		
		long endTime = System.currentTimeMillis();
		
		System.out.println("耗时:" + endTime - startTime);
		
	}
```

打印结果：

```java
Thread-2 = 1249975000
Thread-3 = 1249975000
Thread-1 = 1249975000
Thread-4 = 1249975000
Thread-0 = 1249975000
耗时:2
```



## Semaphore

