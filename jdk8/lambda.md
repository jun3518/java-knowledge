## 说明

Lambda 表达式的基础语法：Java8中引入了一个新的操作符 "->" 该操作符称为箭头操作符或 Lambda 操作符箭头操作符将 Lambda 表达式拆分成两部分：

（1）左侧：Lambda 表达式的参数列表

（2）右侧：Lambda 表达式中所需执行的功能， 即 Lambda 体

## 语法格式

### 1、无参数，无返回值：

​	() -> System.out.println("Hello Lambda!");

```java
@Test
public void test1(){
	Runnable r1 = () -> System.out.println("Hello Lambda!");
	r1.run();
}
```

### 2、有一个参数，并且无返回值：

(x) -> System.out.println(x);

```java
@Test
public void test02() {
    Consumer<String> con = (x) -> System.out.println(x);
    con.accept("zhangsan");
}
```

### 3、若只有一个参数，小括号可以省略不写：

x -> System.out.println(x);

```java
@Test
public void test03() {
    Consumer<String> con = x -> System.out.println(x);
    con.accept("zhangsan");
}
```

### 4、有两个以上的参数，有返回值，并且 Lambda 体中有多条语句：

(x, y) -> {

​	// ...

};

```java
@Test
public void test04() {
    Comparator<Integer> com = (x, y) -> {
        System.out.println("zhangsan");
        return Integer.compare(x, y);
    };
}
```

### 5、若 Lambda 体中只有一条语句， return 和 大括号都可以省略不写：

(x, y) -> x + y;

```java
@Test
public void test05() {
	Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
}
```

### 6、Lambda 表达式的参数列表的数据类型可以省略不写，因为JVM编译器通过上下文推断出，数据类型，即“类型推断”：

(Integer x, Integer y) -> Integer.compare(x, y);

```java
@Test
public void test06() {
	Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
	int result = com.compare(10, 11);
}
```

## Lambda 表达式需要“函数式接口”的支持

函数式接口：接口中只有一个抽象方法的接口，称为函数式接口。 可以使用注解 @FunctionalInterface 修饰可以检查是否是函数式接口。

## Java8 内置的四大核心函数式接口

### 1、Consumer\<T> : 消费型接口

void accept(T t);

```java
//Consumer<T> 消费型接口 :
@Test
public void test1(){
    happy(10000, (m) -> System.out.println("消费：" + m + "元"));
} 

public void happy(double money, Consumer<Double> con){
    con.accept(money);
}
```

### 2、Supplier\<T> : 供给型接口

T get(); 

```java
//Supplier<T> 供给型接口 :
@Test
public void test2(){
    List<Integer> numList = getNumList(10, () -> (int)(Math.random() * 100));
    for (Integer num : numList) {
        System.out.println(num);
    }
}
//需求：产生指定个数的整数，并放入集合中
public List<Integer> getNumList(int num, Supplier<Integer> sup){
    List<Integer> list = new ArrayList<>();	
    for (int i = 0; i < num; i++) {
        Integer n = sup.get();
        list.add(n);
    }
    return list;
}
```

### 3、Function<T, R> : 函数型接口

R apply(T t);

```java
//Function<T, R> 函数型接口：
@Test
public void test3(){
    String newStr = strHandler("\t\t\t HelloWorld   ", (str) -> str.trim());
    System.out.println(newStr);

    String subStr = strHandler("Hello Lambda!", (str) -> str.substring(2, 5));
    System.out.println(subStr);
}
```

### 4、Predicate\<T> : 断言型接口

boolean test(T t);

```java
//Predicate<T> 断言型接口：
@Test
public void test4(){
    List<String> list = Arrays.asList("Hello", "java", "Lambda", "www", "ok");
    List<String> strList = filterStr(list, (s) -> s.length() > 3);
    for (String str : strList) {
        System.out.println(str);
    }
}
//需求：将满足条件的字符串，放入集合中
public List<String> filterStr(List<String> list, Predicate<String> pre){
    List<String> strList = new ArrayList<>();
    for (String str : list) {
        if(pre.test(str)){
            strList.add(str);
        }
    }
    return strList;
}
```

### 5、其他函数式接口

| 函数式接口                                                   | 参数类型                | 返回类型                | 用途                                                         |
| ------------------------------------------------------------ | ----------------------- | ----------------------- | :----------------------------------------------------------- |
| Consumer\<T>消费型接口                                       | T                       | void                    | 对类型为T的对象应用操作，包含方法：void accept(T t)          |
| Supplier\<T>供给型接口                                       | 无                      | T                       | 返回类型为T的对象，包含方法：T get();                        |
| Function<T, R>函数型接口                                     | T                       | R                       | 对类型为T的对象应用操作，并返回结果。结果是R类型的对象。包含方法：R apply(T t); |
| Predicate\<T>断定型接口                                      | T                       | boolean                 | 确定类型为T的对象是否满足某约束，并返回boolean 值。包含方法boolean test(T t); |
| UnaryOperator\<T>(Function 子接口) )                         | T                       | T                       | 对类型为T的对象进行一元运算，并返回T类型的结果。包含方法为T apply(T t); |
| BinaryOperator\<T>( (n BiFunction  子接口) )                 | T                       | T                       | 对类型为T的对象进行二元运算，并返回T类型的结果。包含方法为T apply(T t1, T t2); |
| BiConsumer<T, U>                                             | T, U                    | void                    | 对类型为T, U 参数应用操作。包含方法为void accept(T t, U u)   |
| ToIntFunction\<T><br/>ToLongFunction\<T><br/>ToDoubleFunction\<T> | T                       | int<br/>long<br/>double | 分 别 计 算 int 、 long 、double、值的函数                   |
| IntFunction\<R><br/>LongFunction\<R><br/>DoubleFunction\<R>  | int<br/>long<br/>double | R                       | 参数分别为int、long、double 类型的函数                       |

