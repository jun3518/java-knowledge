## 饿汉式（立即加载）

1、将自身实例化对象设置为一个属性，并用static、final修饰。

2、构造方法私有化。

3、静态方法返回该实例。

```java
public class Singleton {
	// 将自身实例化对象设置为一个属性，并用static、final修饰
	private static Singleton singleton = new Singleton();
	// 构造方法私有化
	private Singleton() {
	}
	// 静态方法返回该实例
	public static Singleton getInstance() {
		return singleton;
	}
}
```

### 饿汉式优缺点

优点：实现起来简单，没有多线程同步问题。

缺点：当类Singleton被加载的时候，会初始化static的instance，静态变量被创建并分配内存空间，从这以后，这个static的instance对象便一直占着这段内存（即便你还没有用到这个实例），当类被卸载时，静态变量被摧毁，并释放所占有的内存，因此在某些特定条件下会耗费内存。



## 懒汉式（延迟加载）

延迟加载就是调用getInstance()方法时实例才被创建（故又称为“懒汉模式”），常见的实现方法就是在getInstance()方法中进行new实例化。

1、声明一个Singleton静态成员变量（不实例化）。

2、构造方法私有化。

3、静态方法获取该实例，在获取的时候进行初始化。

```java
public class Singleton {
	// 将自身实例化对象设置为一个属性，并用static修饰
	private static Singleton instance;
	// 构造方法私有化
	private Singleton() {
	}
	// 静态方法返回该实例
	public static Singleton getInstance() {
		if (instance == null) {
			instance = new Singleton();
		}
		return instance;
	}
}
```

### 懒汉式优缺点

优点：实现起来比较简单，当类Singleton被加载的时候，静态变量static的instance未被创建并分配内存空间，当getInstance方法第一次被调用时，初始化instance变量，并分配内存，因此在某些特定条件下会节约了内存。

缺点：在多线程环境中，这种实现方法是完全错误的，根本不能保证单例的状态。



## 懒汉式线程不安全示例

```java
public class Singleton {
	// 将自身实例化对象设置为一个属性，并用static修饰
	private static Singleton instance;
	// 构造方法私有化
	private Singleton() {
	}
	// 静态方法返回该实例
	public static Singleton getInstance() {
		if (instance == null) {
			try {
                // 睡眠1秒，放大多线程问题
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				throw new RuntimeException(e);
			}
			instance = new Singleton();
		}
		return instance;
	}
}
```

```java

public class TestSingleton {
	public static void main(String[] args) {	
		Thread[] threads = new Thread[10];	
		MyRunnable runnable = new MyRunnable();
		for (int i = 0; i < threads.length; i++) {
			threads[i] = new Thread(runnable);
		}
		for (Thread thread : threads) {
			thread.start();
		}
	}
}

class MyRunnable implements Runnable{
	@Override
	public void run() {
		Singleton singleton = Singleton.getInstance();
		System.out.println(singleton);
	}
}
```

打印结果：

```java
singleton.demo02.Singleton@251ff751
singleton.demo02.Singleton@2f1eeb2f
singleton.demo02.Singleton@10c69a60
singleton.demo02.Singleton@251ff751
singleton.demo02.Singleton@25bbaebf
singleton.demo02.Singleton@eb6c0dd
singleton.demo02.Singleton@9ac7010
singleton.demo02.Singleton@2db33043
singleton.demo02.Singleton@6226284a
singleton.demo02.Singleton@25bbaebf
```

从上面的代码中可以看出，此时的Singleton是线程不安全的。



## 实现线程安全的懒汉式1（在getInstance方法级别加锁）

```java
public class Singleton {
    // 将自身实例化对象设置为一个属性，并用static修饰
    private static Singleton instance;
    // 构造方法私有化
    private Singleton() {
    }
    // 静态方法返回该实例，加synchronized关键字实现同步
    public static synchronized Singleton getInstance() {
        if(instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

再次测试代码：

```java
public class TestSingleton {
	public static void main(String[] args) {	
		Thread[] threads = new Thread[10];	
		MyRunnable runnable = new MyRunnable();
		for (int i = 0; i < threads.length; i++) {
			threads[i] = new Thread(runnable);
		}
		for (Thread thread : threads) {
			thread.start();
		}
	}
}

class MyRunnable implements Runnable{
	@Override
	public void run() {
		Singleton singleton = Singleton.getInstance();
		System.out.println(singleton);
	}
}
```

打印结果：

```java
singleton.demo03.Singleton@6d6019f0
singleton.demo03.Singleton@6d6019f0
singleton.demo03.Singleton@6d6019f0
singleton.demo03.Singleton@6d6019f0
singleton.demo03.Singleton@6d6019f0
singleton.demo03.Singleton@6d6019f0
singleton.demo03.Singleton@6d6019f0
singleton.demo03.Singleton@6d6019f0
singleton.demo03.Singleton@6d6019f0
singleton.demo03.Singleton@6d6019f0
```

### 优缺点

优点：在多线程情形下，保证了“懒汉模式”的线程安全。

缺点：在多线程情形下，synchronized方法通常效率低，显然这不是最佳的实现方案。



## 实现线程安全的懒汉式2-DCL双检查锁机制（DCL：double checked locking）

```java
public class Singleton {
	// 将自身实例化对象设置为一个属性，并用static修饰
	private static Singleton instance;
	// 构造方法私有化
	private Singleton() {
	}
	// 静态方法返回该实例
	public static Singleton getInstance() {
        // 第一次检查instance是否被实例化出来，如果没有进入if块
		if (instance == null) {
			synchronized (Singleton.class) {
                // 某个线程取得了类锁，实例化对象前第二次检查instance是否已经被实例化出来，如果没有，才最终实例出对象
				if(instance == null) {
					instance = new Singleton();
				}
			}
		}
		return instance;
	}
}
```

再次测试代码：

```java
public class TestSingleton {
	public static void main(String[] args) {	
		Thread[] threads = new Thread[10];	
		MyRunnable runnable = new MyRunnable();
		for (int i = 0; i < threads.length; i++) {
			threads[i] = new Thread(runnable);
		}
		for (Thread thread : threads) {
			thread.start();
		}
	}
}

class MyRunnable implements Runnable{
	@Override
	public void run() {
		Singleton singleton = Singleton.getInstance();
		System.out.println(singleton);
	}
}
```

打印结果：

```java
singleton.demo04.Singleton@eb6c0dd
singleton.demo04.Singleton@eb6c0dd
singleton.demo04.Singleton@eb6c0dd
singleton.demo04.Singleton@eb6c0dd
singleton.demo04.Singleton@eb6c0dd
singleton.demo04.Singleton@eb6c0dd
singleton.demo04.Singleton@eb6c0dd
singleton.demo04.Singleton@eb6c0dd
singleton.demo04.Singleton@eb6c0dd
singleton.demo04.Singleton@eb6c0dd
```



## 实现线程安全的懒汉式3-静态内部类

首先需要清楚静态内部类的特点：

（1）创建静态内部类的时候是不需要将静态内部类的实例对象绑定到外部类的实例对象上。

（2）静态内部类属于外部类，而不是属于外部类的对象（所以静态内部类不会随外部类的实例化而实例化）。

（3）静态内部类如果要访问外部的成员变量或者成员方法，那么必须是静态的。

（4）会生成两个.class文件，一个是外部的类Outer.class ， 另一个是 Outer$Inner.class。

```java
public class Singleton {
	// 构造方法私有化
	private Singleton() {
	}
	// 创建静态内部类，该类中有一个静态属性Singleton
	private static class StaticSingletonInstance {
		private static final Singleton instance = new Singleton();
	}
	// 静态方法返回该实例
	public static Singleton getInstance() {
		return StaticSingletonInstance.instance;
	}
}
```

测试代码：

```java
public class TestSingleton {
	public static void main(String[] args) {	
		Thread[] threads = new Thread[10];
		MyRunnable runnable = new MyRunnable();
		for (int i = 0; i < threads.length; i++) {
			threads[i] = new Thread(runnable);
		}
		for (Thread thread : threads) {
			thread.start();
		}
	}
}

class MyRunnable implements Runnable{
	@Override
	public void run() {
		Singleton singleton = Singleton.getInstance();
		System.out.println(singleton);
	}
}
```

打印结果：

```java
singleton.demo05.Singleton@2db33043
singleton.demo05.Singleton@2db33043
singleton.demo05.Singleton@2db33043
singleton.demo05.Singleton@2db33043
singleton.demo05.Singleton@2db33043
singleton.demo05.Singleton@2db33043
singleton.demo05.Singleton@2db33043
singleton.demo05.Singleton@2db33043
singleton.demo05.Singleton@2db33043
singleton.demo05.Singleton@2db33043
```

### 优缺点

优点：

（1）这种方式采用了类装载的机制来保证初始化实例时只有一个线程。

（2）静态内部类方式在Singleton类被装载时并不会立即实例化，而是在需要实例化时，调用getInstance方法，才会装载StaticSingletonInstance类，从而完成Singleton的实例化。

（3）类的静态属性只会在第一次加载类的时候初始化，所以在这里，JVM帮助我们保证了线程的安全性，在类进行初始化时，别的线程是无法进入的。

（4）避免了 线程不安全，利用静态内部类特点实现延迟加载，效率高。



## 实现线程安全的懒汉式4-枚举

首先需要清楚枚举的特点：

枚举是天然的单例。

借助枚举来实现单例模式，不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象。

```java
public enum Singleton {
	INSTANCE;
	private static Map<String, String> map = new HashMap<>();
	static {
		map.put("k1", "v1");
		map.put("k2", "v2");
		map.put("k3", "v3");
	}
	public String getValue(String key) {
		return map.get(key);
	}
}
```

测试代码：

```java
public class TestSingleton {
	public static void main(String[] args) {
		Singleton singleton = Singleton.INSTANCE;
		System.out.println(singleton.getValue("k1"));
	}
}
```



## 单例模式注意事项和细节说明

1、单例模式保证了 系统内存中该类只存在一个对象，节省了系统资源，对于一些需要频繁创建销毁的对象，使用单例模式可以提高系统性能。

2、当想实例化一个单例类的时候，必须要记住使用相应的获取对象的方法，而不是使用new的方式创建对象。

3、单例模式使用的场景：需要频繁的进行创建和销毁的对象、创建对象时耗时过多或耗费资源过多（即：重量级对象)，但又经常用到的对象、工具类对象、频繁访问数据库或文件的对象(比如数据源、session工厂等）。

