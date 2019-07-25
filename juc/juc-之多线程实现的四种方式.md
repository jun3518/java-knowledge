## 概述

Java多线程实现方式主要有四种：

（1）继承Thread类创建线程

（2）实现Runnable接口创建线程

（3）实现Callable接口通过FutureTask包装器来创建Thread线程

（4）使用ExecutorService、Callable、Future实现有返回结果的线程



## 继承Thread类创建线程

```java
class ThreadDemo extends Thread {
	
	@Override
	public void run() {
		System.out.println(Thread.currentThread().getName() + "启动..");
	}
}
```

Client：

```java
public static void main(String[] args) {
		
		ThreadDemo threadDemo1 = new ThreadDemo();
		ThreadDemo threadDemo2 = new ThreadDemo();
		threadDemo1.start();
		threadDemo2.start();	
	}
```

打印结果：

```java
Thread-0启动..
Thread-1启动..
```



## 实现Runnable接口创建线程

```java
class ThreadDemo implements Runnable {
	
	@Override
	public void run() {
		System.out.println(Thread.currentThread().getName() + "启动..");
	}
}
```

Client：

```java
ThreadDemo demo01 = new ThreadDemo();
ThreadDemo demo02 = new ThreadDemo();
new Thread(demo01).start();
new Thread(demo02).start();
```

打印结果：

```java
Thread-0启动..
Thread-1启动..
```



## 实现Callable接口通过FutureTask包装器来创建Thread线程

Callable接口：

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

```java
class CallableDemo implements Callable<Integer> {

	@Override
	public Integer call() throws Exception {
		int num = 0;
		for (int i = 0; i < 100; i++) {
			num += i;
		}
		return num;
	}	
}
```

Client：

```java
public static void main(String[] args) throws Exception {

    CallableDemo callableDemo = new CallableDemo();
    // 由Callable<Integer>创建一个FutureTask<Integer>对象：
    FutureTask<Integer> futureTask = new FutureTask<>(callableDemo);
    // 注释：FutureTask<Integer>是一个包装器，它通过接受Callable<Integer>来创建，它同时实现了Future和Runnable接口。
    // 由FutureTask<Integer>创建一个Thread对象：
    Thread thread = new Thread(futureTask);
    thread.start();
    // 至此，一个线程就创建完成了。
    Integer num = futureTask.get();  // futureTask.get()方法会阻塞main线程的往下执行
    System.out.println(num); // 打印结果：4950

}
```

FutureTask实现了Runnable接口，从FutureTask的run方法中可以看到，在run方法中其实调用了Callable的call方法获取值，然后将值传给FutureTask对象的private Object outcome;属性。在通过futureTask.get()获取执行结果值的时候，会将其outcome返回。