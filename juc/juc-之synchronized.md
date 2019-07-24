## synchronized概述

Synchronized是Java中解决并发问题的一种最常用的方法，也是最简单的一种方法。

Synchronized主要有三个作用：

（1）确保线程互斥的访问同步代码。

（2）保证共享变量的修改能够及时可见。

（3）有效解决重排序问题。

## synchronized使用场景

### 普通同步方法，锁是当前实例对象

```java
public class SynchronizedTest {
    
	public synchronized void method1() {
    	System.out.println("Method 1 start");
        try {
            System.out.println("Method 1 execute");
            Thread.sleep(3000);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
         System.out.println("Method 1 end");
 	}
    
    public synchronized void method2() {
		System.out.println("Method 2 start");
		try {
			System.out.println("Method 2 execute");
			Thread.sleep(3000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("Method 2 end");
	}
}
```

Client：

```java
public static void main(String[] args) {
    
    SynchronizedTest test = new SynchronizedTest();
    new Thread(new Runnable() {
        @Override
        public void run() {
            test.method1();
        }
    }).start();
    
    new Thread(new Runnable() {
        @Override
        public void run() {
            test.method2();
        }
    }).start();
}
```

打印结果：

```java
Method 1 start
Method 1 execute
Method 1 end
Method 2 start
Method 2 execute
Method 2 end
```

从打印结果上看，test.method1();方法执行完之后，才能轮到test.method2();执行。

Client：

```java
public static void main(String[] args) {
    
    SynchronizedTest test01 = new SynchronizedTest();
    SynchronizedTest test02 = new SynchronizedTest();
    new Thread(new Runnable() {
        @Override
        public void run() {
            test01.method1();
        }
    }).start();
    
    new Thread(new Runnable() {
        @Override
        public void run() {
            test02.method2();
        }
    }).start();
}
```

打印结果：

```java
Method 1 start
Method 1 execute
Method 2 start
Method 2 execute
Method 2 end
Method 1 end
```

从打印结果上看，test01.method1();方法在执行过程中test02.method2();同时也在执行。



### 静态同步方法，锁是当前类的class对象

```java
public class SynchronizedTest {
    
    public static synchronized void method1() {
    	System.out.println("Method 1 start");
        try {
            System.out.println("Method 1 execute");
            Thread.sleep(3000);
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
         System.out.println("Method 1 end");
 	}
	
	public static synchronized void method2() {
		System.out.println("Method 2 start");
		try {
			System.out.println("Method 2 execute");
			Thread.sleep(3000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("Method 2 end");
	}  
}
```

Client：

```java
public static void main(String[] args) {
		
    SynchronizedTest test01 = new SynchronizedTest();
    SynchronizedTest test02 = new SynchronizedTest();
    new Thread(new Runnable() {
        @Override
        public void run() {
            test01.method1();
        }
    }).start();
    
    new Thread(new Runnable() {

        @Override
        public void run() {
            test02.method2();
        }
    }).start();
}
```

打印结果：

```java
Method 1 start
Method 1 execute
Method 1 end
Method 2 start
Method 2 execute
Method 2 end
```

从打印结果上看，test01.method1();方法执行完之后，才能轮到test02.method2();执行。



### 同步方法块，锁是括号里面的对象

```java
public class SynchronizedTest {
    
    public void method1() {
    	System.out.println("Method 1 start");
        try {
        	synchronized (this) {
        		System.out.println("Method 1 execute");
        		Thread.sleep(3000);
			}
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
         System.out.println("Method 1 end");
 	}
	
	public void method2() {
		System.out.println("Method 2 start");
		try {
			synchronized (this) {
				System.out.println("Method 2 execute");
				Thread.sleep(3000);
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("Method 2 end");
	}
}
```

Client：

```java
public static void main(String[] args) {
		
    SynchronizedTest test01 = new SynchronizedTest();
    new Thread(new Runnable() {
        @Override
        public void run() {
            test01.method1();
        }
    }).start();

    new Thread(new Runnable() {	
        @Override
        public void run() {
            test01.method2();
        }
    }).start();
}
```

打印结果：

```java
Method 1 start
Method 1 execute
Method 2 start
Method 1 end
Method 2 execute
Method 2 end
```

从打印结果上看，test.method1();方法执行完之后，才能轮到test.method2();执行。

Client：

```java
public static void main(String[] args) {
		
    SynchronizedTest test01 = new SynchronizedTest();
    SynchronizedTest test02 = new SynchronizedTest();
    new Thread(new Runnable() {
        @Override
        public void run() {
            test01.method1();
        }
    }).start();

    new Thread(new Runnable() {	
        @Override
        public void run() {
            test02.method2();
        }
    }).start();
}
```

打印结果：

```java
Method 1 start
Method 1 execute
Method 2 start
Method 2 execute
Method 2 end
Method 1 end
```

从打印结果上看，test.method1();方法执行完之后，才能轮到test.method2();执行。



## synchronize底层原理

Java 虚拟机中的同步(Synchronization)基于进入和退出Monitor对象实现， 无论是显式同步(有明确的 monitorenter 和 monitorexit 指令,即同步代码块)还是隐式同步都是如此。在 Java 语言中，同步用的最多的地方可能是被 synchronized 修饰的同步方法。同步方法 并不是由 monitorenter 和 monitorexit 指令来实现同步的，而是由方法调用指令读取运行时常量池中方法表结构的 ACC_SYNCHRONIZED 标志来隐式实现的。

同步代码块：monitorenter指令插入到同步代码块的开始位置，monitorexit指令插入到同步代码块的结束位置，JVM需要保证每一个monitorenter都有一个monitorexit与之相对应。任何对象都有一个monitor与之相关联，当且一个monitor被持有之后，他将处于锁定状态。线程执行到monitorenter指令时，将会尝试获取对象所对应的monitor所有权，即尝试获取对象的锁。

在JVM中，对象在内存中的布局分为三块区域：对象头、实例变量和填充数据。如下：

![](https://images2017.cnblogs.com/blog/918656/201708/918656-20170824180707777-1312147323.png)

实例变量：存放类的属性数据信息，包括父类的属性信息，如果是数组的实例部分还包括数组的长度，这部分内存按4字节对齐。

填充数据：由于虚拟机要求对象起始地址必须是8字节的整数倍。填充数据不是必须存在的，仅仅是为了字节对齐，这点了解即可。

对象头：Hotspot虚拟机的对象头主要包括两部分数据：Mark Word（标记字段）、Klass Pointer（类型指针）。其中Klass Point是是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例，Mark Word用于存储对象自身的运行时数据，它是实现轻量级锁和偏向锁的关键。

Mark Word：用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等等。Java对象头一般占有两个机器码（在32位虚拟机中，1个机器码等于4字节，也就是32bit），但是如果对象是数组类型，则需要三个机器码，因为JVM虚拟机可以通过Java对象的元数据信息确定Java对象的大小，但是无法从数组的元数据来确认数组的大小，所以用一块来记录数组长度。

Monior：我们可以把它理解为一个同步工具，也可以描述为一种同步机制，它通常被描述为一个对象。与一切皆对象一样，所有的Java对象是天生的Monitor，每一个Java对象都有成为Monitor的潜质，因为在Java的设计中 ，每一个Java对象自打娘胎里出来就带了一把看不见的锁，它叫做内部锁或者Monitor锁。Monitor 是线程私有的数据结构，每一个线程都有一个可用monitor record列表，同时还有一个全局的可用列表。每一个被锁住的对象都会和一个monitor关联（对象头的MarkWord中的LockWord指向monitor的起始地址），同时monitor中有一个Owner字段存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。其结构如下：

![](https://images2017.cnblogs.com/blog/918656/201708/918656-20170824181742996-1450027779.png)

Owner：初始时为NULL表示当前没有任何线程拥有该monitor record，当线程成功拥有该锁后保存线程唯一标识，当锁被释放时又设置为NULL；
EntryQ:关联一个系统互斥锁（semaphore），阻塞所有试图锁住monitor record失败的线程。
RcThis:表示blocked或waiting在该monitor record上的所有线程的个数。
Nest:用来实现重入锁的计数。
HashCode:保存从对象头拷贝过来的HashCode值（可能还包含GC age）。
Candidate:用来避免不必要的阻塞或等待线程唤醒，因为每一次只有一个线程能够成功拥有锁，如果每次前一个释放锁的线程唤醒所有正在阻塞或等待的线程，会引起不必要的上下文切换（从阻塞到就绪然后因为竞争锁失败又被阻塞）从而导致性能严重下降。Candidate只有两种可能的值0表示没有需要唤醒的线程1表示要唤醒一个继任线程来竞争锁。

## Java虚拟机对synchronize的优化 

锁的状态总共有四种，无锁状态、偏向锁、轻量级锁和重量级锁。随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁，但是锁的升级是单向的，也就是说只能从低到高升级，不会出现锁的降级，关于重量级锁，前面我们已详细分析过，下面我们将介绍偏向锁和轻量级锁以及JVM的其他优化手段。

### 偏向锁

偏向锁是Java 6之后加入的新锁，它是一种针对加锁操作的优化手段，经过研究发现，在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，因此为了减少同一线程获取锁(会涉及到一些CAS操作,耗时)的代价而引入偏向锁。偏向锁的核心思想是，如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word 的结构也变为偏向锁结构，当这个线程再次请求锁时，无需再做任何同步操作，即获取锁的过程，这样就省去了大量有关锁申请的操作，从而也就提供程序的性能。所以，对于没有锁竞争的场合，偏向锁有很好的优化效果，毕竟极有可能连续多次是同一个线程申请相同的锁。但是对于锁竞争比较激烈的场合，偏向锁就失效了，因为这样场合极有可能每次申请锁的线程都是不相同的，因此这种场合下不应该使用偏向锁，否则会得不偿失，需要注意的是，偏向锁失败后，并不会立即膨胀为重量级锁，而是先升级为轻量级锁。

### 轻量级锁

倘若偏向锁失败，虚拟机并不会立即升级为重量级锁，它还会尝试使用一种称为轻量级锁的优化手段(1.6之后加入的)，此时Mark Word 的结构也变为轻量级锁的结构。轻量级锁能够提升程序性能的依据是“对绝大部分的锁，在整个同步周期内都不存在竞争”，注意这是经验数据。需要了解的是，轻量级锁所适应的场景是线程交替执行同步块的场合，如果存在同一时间访问同一锁的场合，就会导致轻量级锁膨胀为重量级锁。

### 自旋锁

轻量级锁失败后，虚拟机为了避免线程真实地在操作系统层面挂起，还会进行一项称为自旋锁的优化手段。这是基于在大多数情况下，线程持有锁的时间都不会太长，如果直接挂起操作系统层面的线程可能会得不偿失，毕竟操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，因此自旋锁会假设在不久将来，当前的线程可以获得锁，因此虚拟机会让当前想要获取锁的线程做几个空循环(这也是称为自旋的原因)，一般不会太久，可能是50个循环或100循环，在经过若干次循环后，如果得到锁，就顺利进入临界区。如果还不能获得锁，那就会将线程在操作系统层面挂起，这就是自旋锁的优化方式，这种方式确实也是可以提升效率的。最后没办法也就只能升级为重量级锁了。

### 锁消除

消除锁是虚拟机另外一种锁的优化，这种优化更彻底，Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译)，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间，如下StringBuffer的append是一个同步方法，但是在add方法中的StringBuffer属于一个局部变量，并且不会被其他线程所使用，因此StringBuffer不可能存在共享资源竞争的情景，JVM会自动将其锁消除。



## synchronize的可重入性

从互斥锁的设计上来说，当一个线程试图操作一个由其他线程持有的对象锁的临界资源时，将会处于阻塞状态，但当一个线程再次请求自己持有对象锁的临界资源时，这种情况属于重入锁，请求将会成功。在java中，synchronized是基于原子性的内部锁机制，是可重入的，因此在一个线程调用synchronized方法的同时在其方法体内部调用该对象另一个synchronized方法，也就是说一个线程得到一个对象锁后再次请求该对象锁，是允许的，这就是synchronized的可重入性。

```java
public class SynchronizedTest4 {

	public synchronized void method1() {
    	System.out.println("Method 1 start");
        try {
            System.out.println("Method 1 execute");
            Thread.sleep(3000);
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
         System.out.println("Method 1 end");
        
         // 在method1中调用method3
         method3();
 	}
	
	public synchronized void method2() {
		System.out.println("Method 2 start");
		try {
			System.out.println("Method 2 execute");
			Thread.sleep(3000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("Method 2 end");
	}
	
	public synchronized void method3() {
		System.out.println("Method 3 start");
		try {
			System.out.println("Method 3 execute");
			Thread.sleep(3000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("Method 3 end");
	}
}
```

Client：

```java
public static void main(String[] args) {
		
    SynchronizedTest4 test01 = new SynchronizedTest4();

    new Thread(new Runnable() {

        @Override
        public void run() {
            test01.method1();
        }
    }).start();

    new Thread(new Runnable() {

        @Override
        public void run() {
            test01.method2();
        }
    }).start();

}
```

打印结果：

```java
Method 1 start
Method 1 execute
Method 1 end
Method 3 start
Method 3 execute
Method 3 end
Method 2 start
Method 2 execute
Method 2 end
```

从打印结果中可以看出，method1()执行完之后，接着执行method3()，最后才执行method2()。

注：当子类继承父类时，子类也是可以通过可重入锁调用父类的同步方法。注意由于synchronized是基于monitor实现的，因此每次重入，monitor中的计数器仍会加1。



## 等待唤醒机制与synchronize

所谓等待唤醒机制本篇主要指的是notify/notifyAll和wait方法，在使用这3个方法时，必须处于synchronized代码块或者synchronized方法中，否则就会抛出IllegalMonitorStateException异常，这是因为调用这几个方法前必须拿到当前对象的监视器monitor对象，也就是说notify/notifyAll和wait方法依赖于monitor对象，在前面的分析中，我们知道monitor 存在于对象头的Mark Word 中(存储monitor引用指针)，而synchronized关键字可以获取 monitor ，这也就是为什么notify/notifyAll和wait方法必须在synchronized代码块或者synchronized方法调用的原因。