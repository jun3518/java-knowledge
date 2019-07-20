## Java序列化和反序列化概述

序列化：把对象转换为字节序列的过程称为对象的序列化。

反序列化：把字节序列恢复为对象的过程称为对象的反序列化。

## 序列化的使用场景

1、把的内存中的对象持久化。如存到数据库或文件中。

2、进行PRC调用时传输对象。即可以在进程之间进行传递和接收。

3、深度克隆对象。通过对对象进行序列化和反序列化可以深度克隆对象。

## 在Java如何对对象进行序列化

在Java中，对象想要进行序列化，在类中必须实现java.io.Serializable标识接口或java.io.Externalizable接口，并建议生成一个private static final long serialVersionUID值。为什么建议生成这个值，下文会详细说明原因。

```java
class User implements Serializable {
	// 生成序列化标识值
	private static final long serialVersionUID = 4906905641657588803L;

	private String username;

	private String password;

	public User(String username, String password) {
		this.username = username;
		this.password = password;
	}
	
	public User() {
	}

	public String getUsername() {
		return username;
	}

	public void setUsername(String username) {
		this.username = username;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	@Override
	public String toString() {
		return "User [username=" + username + ", password=" + password + "]";
	}
}
```

以下是封装了Java序列化的工具类：

```java
public class SerializeUtil {
    
	private static final String PATH = "f:/temp/";

    // 序列化
	public static void serialize(Object obj, String fileName) {

		File file = new File(PATH, fileName);
		try (ObjectOutputStream oos = new ObjectOutputStream(
            new FileOutputStream(file));) {
			oos.writeObject(obj);
		} catch (IOException e) {
			throw new RuntimeException(e);
		}
	}

    // 反序列化
	@SuppressWarnings("unchecked")
	public static <T> T deserialize(String fileName) {
		File file = new File(PATH, fileName);
		try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));) {
			return (T) ois.readObject();
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
	}

}
```

Client：

```java
public static void main(String[] args) {
    User user = new User("zhangsan", "123456");
    SerializeUtil.serialize(user, User.class.getSimpleName());
    User user2 = SerializeUtil.deserialize(User.class.getSimpleName());
    System.out.println(user2);
    // 打印结果：User [username=zhangsan, password=123456]
}
```



## private static final long serialVersionUID

为什么在类中实现java.io.Serializable这个标识接口，会建议生成一个private static final long serialVersionUID值呢？

serialVersionUID适用于Java序列化机制。简单来说，JAVA序列化的机制是通过判断类的serialVersionUID来验证类的版本是否一致的。在进行反序列化时，JVM会把传来的字节流中的serialVersionUID于本地相应实体类的serialVersionUID进行比较。如果相同说明是一致的，可以进行反序列化，否则会出现反序列化版本一致的异常，即是InvalidCastException。

当实现java.io.Serializable接口中没有显示的定义serialVersionUID变量的时候，Java序列化机制会根据Class自动生成一个serialVersionUID作序列化版本比较用，这种情况下，如果Class文件(类名，方法明等)没有发生变化(增加空格，换行，增加注释等等不算)，就算再编译多次，serialVersionUID也不会变化的。

如果不希望通过编译来强制划分软件版本，即实现序列化接口的实体能够兼容先前版本，就需要显示的定义一个serialVersionUID，类型为long的变量。不修改这个变量值的序列化实体，都可以相互进行序列化和反序列化。

### 不添加private static final long serialVersionUID的问题

```java
class User implements Serializable {
    
	private String username;

	private String password;

	public User(String username, String password) {
		this.username = username;
		this.password = password;
	}
	
	public User() {
	}

	// getter/setter/toString
}
```

Client：

```java
public static void main(String[] args) {
    User user = new User("zhangsan", "123456");
    SerializeUtil.serialize(user, User.class.getSimpleName());
    User user2 = SerializeUtil.deserialize(User.class.getSimpleName());
    System.out.println(user2);
    // 打印结果：User [username=zhangsan, password=123456]
}
```

从上面的代码和打印结果来看，似乎不添加serialVersionUID也没有问题。

保留序列化源文件，接着修改User类：

```java
class User implements Serializable {

	private String username;

	private String password;
    // 新增age字段
    private Integer age;

	public User(String username, String password) {
		this.username = username;
		this.password = password;
	}
    // 新增相应的构造函数
	public User(String username, String password, Integer age) {
		this.username = username;
		this.password = password;
        this.age = age;
	}
	
	public User() {
	}

	// getter/setter/toString
}
```

Client：

```java
public static void main(String[] args) {
    User user2 = SerializeUtil.deserialize(User.class.getSimpleName());
    System.out.println(user2);
}
```

程序运行结果：

```java
Exception in thread "main" java.lang.RuntimeException: java.io.InvalidClassException: serialize.User; local class incompatible: stream classdesc serialVersionUID = -249687256863967106, local class serialVersionUID = -8414432157578280708
	at serialize.SerializeUtil.deserialize(SerializeUtil.java:41)
	at serialize.SerializeTest.main(SerializeTest.java:13)
Caused by: java.io.InvalidClassException: serialize.User; local class incompatible: stream classdesc serialVersionUID = -249687256863967106, local class serialVersionUID = -8414432157578280708
	at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:687)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1880)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1746)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2037)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1568)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:428)
	at serialize.SerializeUtil.deserialize(SerializeUtil.java:39)
	... 1 more
```

从上面的报错信息可以知道：因为新增了age字段，导致反序列化时serialVersionUID不一致抛异常。

为了避免这样的问题，需要显示的定义一个serialVersionUID的值，类型为long的变量。只要不修改serialVersionUID的值，即使类新增了字段，也不会影响反序列化。

```java
class User implements Serializable {
	
	// 生成序列化标识值
	private static final long serialVersionUID = 4906905641657588803L;

	private String username;

	private String password;

	public User(String username, String password) {
		this.username = username;
		this.password = password;
	}
	
	public User() {
	}

	// getter/setter/toString
}
```

Client：

```java
public static void main(String[] args) {
    User user = new User("zhangsan", "123456");
    SerializeUtil.serialize(user, User.class.getSimpleName());
    User user2 = SerializeUtil.deserialize(User.class.getSimpleName());
    System.out.println(user2);
    // 打印结果：User [username=zhangsan, password=123456]
}
```

保留序列化源文件，接着修改User类：

```java
class User implements Serializable {
	
	// 生成序列化标识值
	private static final long serialVersionUID = 4906905641657588803L;

	private String username;

	private String password;
    // 新增age字段
    private Integer age;

	public User(String username, String password) {
		this.username = username;
		this.password = password;
	}
    // 新增相应的构造函数
	public User(String username, String password, Integer age) {
		this.username = username;
		this.password = password;
        this.age = age;
	}
	
	public User() {
	}

	// getter/setter/toString
}
```

Client：

```java
public static void main(String[] args) {
    User user2 = SerializeUtil.deserialize(User.class.getSimpleName());
    System.out.println(user2);
    // 打印结果：User [username=zhangsan, password=123456, age=null]
}
```

接着修改User类的serialVersionUID为4906905641657588804L：

```java
class User implements Serializable {
	
	// 生成序列化标识值
	private static final long serialVersionUID = 4906905641657588804L;

	private String username;

	private String password;
    // 新增age字段
    private Integer age;

	public User(String username, String password) {
		this.username = username;
		this.password = password;
	}
    // 新增相应的构造函数
	public User(String username, String password, Integer age) {
		this.username = username;
		this.password = password;
        this.age = age;
	}
	
	public User() {
	}

	// getter/setter/toString
}
```

Client：

```java
public static void main(String[] args) {
    User user2 = SerializeUtil.deserialize(User.class.getSimpleName());
    System.out.println(user2);
}
```

打印结果：

```java
Exception in thread "main" java.lang.RuntimeException: java.io.InvalidClassException: serialize.User; local class incompatible: stream classdesc serialVersionUID = 4906905641657588803, local class serialVersionUID = 4906905641657588804
	at serialize.SerializeUtil.deserialize(SerializeUtil.java:41)
	at serialize.SerializeTest.main(SerializeTest.java:13)
Caused by: java.io.InvalidClassException: serialize.User; local class incompatible: stream classdesc serialVersionUID = 4906905641657588803, local class serialVersionUID = 4906905641657588804
	at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:687)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1880)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1746)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2037)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1568)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:428)
	at serialize.SerializeUtil.deserialize(SerializeUtil.java:39)
	... 1 more
```

从结果中可以看到，即使新增了字段age，只要serialVersionUID的值不变，那么反序列化的时候，新增的age会赋数据类型的默认值（Integer默认值为null）。









