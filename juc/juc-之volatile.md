## 问题演示

在将volatile关键字之前，先演示一下没有volatile关键字时会有什么现象：


```java
class ThreadeDemo implements Runnable {

	private boolean flag = false;

	@Override
	public void run() {
		try {
			Thread.sleep(200);
		} catch (InterruptedException e) {
			throw new RuntimeException(e);
		}
		flag = true;
		System.out.println("flag = " + isFlag());
	}
	public boolean isFlag() {
		return flag;
	}
	public void setFlag(boolean flag) {
		this.flag = flag;
	}
}

```

Client：

```java
public static void main(String[] args) {

    ThreadeDemo demo = new ThreadeDemo();
    new Thread(demo).start();
    while (true) {
        if (demo.isFlag()) {
            System.out.println("end...");
            break;
        }
    }
} 
```

执行上面的代码后，会发现程序会阻塞停留在 flag = true 的步骤，不会打印 end... 

在Java中，所有的变量都存储在主内存中。JVM程序在运行的时候，JVM会为每一个线程分配独立的缓存用于提高效率（每个线程都有自己独立的工作内存，里面保存该线程使用到的变量的副本，该副本是主内存中该变量的一份拷贝）。

线程对共享变量的所有操作都必须在自己的工作内存，不能直接从相互内存中读写也不能从主内存中操作。线程间变量值得传递需要通过主内存来完成。

![](https://img2018.cnblogs.com/blog/1130084/201901/1130084-20190121143952505-1865471348.png)



| 步骤 | 主线程                   | demo线程                                                   | main线程                                                     |
| ---- | ------------------------ | ---------------------------------------------------------- | ------------------------------------------------------------ |
| 1    | 主线程会加载flag = false |                                                            |                                                              |
| 2    |                          | demo线程会去主线程中读取flag = false加载到demo线程的缓存中 |                                                              |
| 3    |                          |                                                            | main线程会去主线程中读取flag = false加载到demo线程的缓存中   |
| 4    |                          | 修改demo线程的flag = true，并将值回写到主线程中            |                                                              |
| 5    | 主线程的flag = true      |                                                            |                                                              |
| 6    |                          |                                                            | main线程在while(true)循环中一直读取main线程中的flag = false，所以导致main线程一直无法中断执行 |

在上述表格的第6步中，main线程一直在循环中读取main线程缓存中的flag = false，导致执行无法中断，即main线程无法读取到demo线程对主线程的更新操作，这就是内存可见性问题。



## 内存可见性

内存可见性：一个线程对共享变量的修改，更够及时的被其他线程看到。

内存可见性问题是：当多个线程操作共享数据时，彼此不可见。即线程demo更新主存的共享变量没有更新到main线程的缓存中，导致main线程一直读取本线程缓存的变量。



## 保证内存可见性的方式

要实现共享变量的的可见性，必须保证两点：

（1）线程修改后的共享变量值能够及时从工作内存刷新到主内存中。

（2）其他线程能够及时把共享变量的最新值从主内存更新到自己的工作内存中。

### 保证内存可见性的方式1：使用锁synchronized

Java内存模型（JVM）关于synchronized的两条规定：

（1）线程解锁前，必须把共享变量的最新值刷新到主内存中。

（2）线程加锁时，将清空该线程内存中的共享变量的值，从而使用共享变量时需要从主内存中重新读取最新的值。

注：加锁与解锁需要是同一把锁。

注：线程解锁前对共享变量的修改在下次加锁时对其他线程可见。 

synchronized线程执行互斥代码的过程：

（1）获得互斥锁。

（2）清空工作内存。

（3）从主内存拷贝变量的最新副本到工作内存。

（4）执行代码。

（5）将更改后的共享变量的值刷新到主内存。

（6）释放互斥锁。

### 保证内存可见性的方式2：使用volatile关键字修饰共享变量

volatile关键字的特点：

（1）能够保证volatile变量的可见性。

（2）不能保证volatile变量复合操作的原子性。

volatile关键字如何实现内存可见性呢？是通过加入内存屏障和禁止重排序优化来实现的：

（1）对volatile变量执行写操作时，会在写操作后加入一条store屏障指令。

（2）对volatile变量执行读操作时，会在读操作后加入一条load屏障指令。

通俗地讲：volatile变量在每次被线程访问时，都强迫从主内存中重读该变量的值，而当该变量发生变化时，又会强迫线程将最新的值刷新到主内存，这样任何时刻，不同的线程总能看到该变量的最新值。

线程写volatile变量的过程：

（1）改变线程工作内存中volatile变量副本的值。

（2）将改变后的副本的值从工作内存刷新到主内存。

线程读volatile变量的过程：

（1）从主内存中读取volatile变量的最新值到线程的工作内存中。

（2）从工作内存中读取volatile变量的副本。

注：volatile不能保证volatile变量复合操作的原子性。



## volatile不能保证volatile变量复合操作的原子性

volatile关键字不能保证原子性演示：

```java
class VolatileAtomic implements Runnable {

	private volatile int value = 0;
	@Override
	public void run() {
		try {
			// 放大多线程的问题
			Thread.sleep(200);
		} catch (InterruptedException e) {
			throw new RuntimeException(e);
		}
		System.out.println(Thread.currentThread().getName() + " : " + getValue());
	}
	public int getValue() {
		return value++;
	}
}
```

Client：

```java
public static void main(String[] args) {
    VolatileAtomic volatileAtomic = new VolatileAtomic();
    Thread[] threads = new Thread[3];
    for (int i = 0; i < threads.length; i++) {
        threads[i] = new Thread(volatileAtomic);
    }
    for (Thread thread : threads) {
        thread.start();
    }
}
```

打印结果：

```java
Thread-2 : 0
Thread-0 : 1
Thread-1 : 0
```

从上面代码可以看到，数值0重复出现了，说明上述代码有线程安全问题。

| 步骤 | 主线程                  | 线程1                                                        | 线程2                                                        |
| ---- | ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | 加载value = 0到主内存中 |                                                              |                                                              |
| 2    |                         | 线程1读取主线程的value = 0                                   |                                                              |
| 3    |                         |                                                              | 线程2读取主线程的value = 0                                   |
| 4    |                         | 线程1执行自增 i++ 操作：int temp = value;  value = value + 1; return temp;// 此时temp=0， value=1 |                                                              |
| 5    |                         |                                                              | 线程2执行自增 i++ 操作：int temp = value;  value = value + 1; return temp; // 此时temp=0， value=1 |
| 6    |                         | 将value值为1的变量会写到主线程中                             |                                                              |
| 7    | 主线程中的value值为1    |                                                              |                                                              |
| 8    |                         |                                                              | 将value值为1的变量会写到主线程中                             |
| 9    | 主线程中的value值为1    |                                                              |                                                              |

从上面流程可以看出，虽然执行了两次i++操作，但是主线程中的value值还是为1。

注：可以使用java.util.concurrent.atomic包下相关基本类型的Atomic类来实现线程安全。