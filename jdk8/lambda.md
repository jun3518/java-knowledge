# Lambda

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



# 方法引用

## 说明

方法引用：若 Lambda 体中的功能，已经有方法提供了实现，可以使用方法引用（可以将方法引用理解为 Lambda 表达式的另外一种表现形式）。

## 语法格式

### 1、对象的引用 :: 实例方法名

Systemout::println;

```java
@Test
public void test01() {
    Consumer<String> con = System.out::println;
    con.accept("zhangsan");
}

@Test
public void test02() {
    Random random = new Random();	
    Supplier<Integer> su = random::nextInt;	
    Integer num = su.get();	
    System.out.println(num);	
}
```

### 2、类名 :: 静态方法名

Integer::compare;

```java
@Test
public void test03(){
    Comparator<Integer> com = Integer::compare;
    int result = com.compare(10, 20);
    System.out.println(result);
}
```

### 3、类名加实例名方法

String::equals;

```java
@Test
public void test04(){
	BiPredicate<String, String> bp = (x, y) -> x.equals(y);
    System.out.println(bp.test("abcde", "abcde"));
    System.out.println("-----------------------------------------");
    BiPredicate<String, String> bp2 = String::equals;
    System.out.println(bp2.test("abc", "abc"));
}
```

### 注意：

#### 1、方法引用所引用的方法的参数列表与返回值类型，需要与函数式接口中抽象方法的参数列表和返回值类型保持一致！

#### 2、若Lambda 的参数列表的第一个参数，是实例方法的调用者，第二个参数(或无参)是实例方法的参数时，格式： ClassName::MethodName。

## 构造器引用

构造器引用 :构造器的参数列表，需要与函数式接口中参数列表保持一致！

### 1、类名::new

```java
@Test
public void test04() {
    Function<Integer, List<String>> func = ArrayList::new; 
    List<String> list = func.apply(10);
}
```

### 2、数组引用

```java
@Test
public void test04() {
    Function<Integer, int[]> func = int[]::new;
    int[] arrys = func.apply(10);
}
```

# Stream

## 创建Stream流的步骤

### 1、创建 Stream

#### 1.1 Collection 提供了两个方法  stream() 与 parallelStream()

```java
List<String> list = new ArrayList<>();
Stream<String> stream = list.stream(); //获取一个顺序流
Stream<String> parallelStream = list.parallelStream(); //获取一个并行流
```

#### 1.2 通过 Arrays 中的 stream() 获取一个数组流

```java
Integer[] nums = new Integer[10];
Stream<Integer> stream1 = Arrays.stream(nums);
```

#### 1.3 通过 Stream 类中静态方法 of()

```java
Stream<Integer> stream2 = Stream.of(1,2,3,4,5,6);
```

#### 1.4 创建无限流

```java
//迭代
Stream<Integer> stream3 = Stream.iterate(0, (x) -> x + 2).limit(10);
stream3.forEach(System.out::println);
//生成
Stream<Double> stream4 = Stream.generate(Math::random).limit(2);
stream4.forEach(System.out::println);
```

## Stream API操作

```java
List<Employee> emps = Arrays.asList(
    new Employee(102, "李四", 59, 6666.66),
    new Employee(101, "张三", 18, 9999.99),
    new Employee(103, "王五", 28, 3333.33),
    new Employee(104, "赵六", 8, 7777.77),
    new Employee(104, "赵六", 8, 7777.77),
    new Employee(104, "赵六", 8, 7777.77),
    new Employee(105, "田七", 38, 5555.55)
);
```

### 筛选与切片

#### 1、filter——接收 Lambda ， 从流中排除某些元素。

```java
@Test
public void test01(){
    //所有的中间操作不会做任何的处理
    Stream<Employee> stream = emps.stream()
        .filter((e) -> {
            System.out.println("测试中间操作");
            return e.getAge() <= 35;
        });

    //只有当做终止操作时，所有的中间操作会一次性的全部执行，称为“惰性求值”
    stream.forEach(System.out::println);
}
```

#### 2、limit——截断流，使其元素不超过给定数量。

```java
@Test
public void test02(){
    emps.stream()
        .filter((e) -> {
            System.out.println("短路！"); // &&  ||
            return e.getSalary() >= 5000;
        }).limit(3)
        .forEach(System.out::println);
}
```

#### 3、skip(n) —— 跳过元素，返回一个扔掉了前 n 个元素的流。若流中元素不足 n 个，则返回一个空流。与 limit(n) 互补

```java
@Test
public void test03(){
    emps.parallelStream()
        .filter((e) -> e.getSalary() >= 5000)
        .skip(2)
        .forEach(System.out::println);
}
```

#### 4、distinct——筛选，通过流所生成元素的 hashCode() 和 equals() 去除重复元素

```java
@Test
public void test04(){
    emps.stream()
        .distinct()
        .forEach(System.out::println);
}
```



#### 5、映射map

map——接收 Lambda ， 将元素转换成其他形式或提取信息。接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。

flatMap——接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流。

```java
public static Stream<Character> filterCharacter(String str){
    List<Character> list = new ArrayList<>();
    for (Character ch : str.toCharArray()) {
        list.add(ch);
    }
    return list.stream();
}
```

map：

```java
List<String> strList = Arrays.asList("aaa", "bbb", "ccc", "ddd", "eee");	
Stream<String> stream = strList.stream()
    .map(String::toUpperCase);
stream.forEach(System.out::println);
```

flatMap：

```java
Stream<Character> stream3 = strList.stream()
			   .flatMap(TestStreamAPI1::filterCharacter);	
stream3.forEach(System.out::println);
```

#### 6、 排序sort

sorted()——自然排序。

sorted(Comparator com)——定制排序。

sorted()：

```java
emps.stream()
    .map(Employee::getName)
    .sorted()
    .forEach(System.out::println);
```

sorted(Comparator com)：

```java
emps.stream()
    .sorted((x, y) -> {
        if(x.getAge() == y.getAge()){
            return x.getName().compareTo(y.getName());
        }else{
            return Integer.compare(x.getAge(), y.getAge());
        }
    }).forEach(System.out::println);
```

### 终止操作

allMatch——检查是否匹配所有元素
anyMatch——检查是否至少匹配一个元素
noneMatch——检查是否没有匹配的元素
findFirst——返回第一个元素
findAny——返回当前流中的任意元素
count——返回流中元素的总个数
max——返回流中最大值
min——返回流中最小值

```java
boolean bl = emps.stream()
				.allMatch((e) -> e.getStatus().equals(Status.BUSY));		
System.out.println(bl);
boolean bl1 = emps.stream()
    .anyMatch((e) -> e.getStatus().equals(Status.BUSY));
System.out.println(bl1);
boolean bl2 = emps.stream()
    .noneMatch((e) -> e.getStatus().equals(Status.BUSY));
System.out.println(bl2);
```

```java
Optional<Employee> op = emps.stream()
			.sorted((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()))
			.findFirst();		
System.out.println(op.get());
System.out.println("--------------------------------");
Optional<Employee> op2 = emps.parallelStream()
    .filter((e) -> e.getStatus().equals(Status.FREE))
    .findAny();
System.out.println(op2.get());
```

```java
long count = emps.stream()
						 .filter((e) -> e.getStatus().equals(Status.FREE))
						 .count();	
System.out.println(count);
Optional<Double> op = emps.stream()
    .map(Employee::getSalary)
    .max(Double::compare);
System.out.println(op.get());
Optional<Employee> op2 = emps.stream()
    .min((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
System.out.println(op2.get());
```

//注意：流进行了终止操作后，不能再次使用：java.lang.IllegalStateException: stream has already been operated upon or closed

```java
@Test
public void test4(){
    Stream<Employee> stream = emps.stream()
        .filter((e) -> e.getStatus().equals(Status.FREE));
    long count = stream.count();
    stream.map(Employee::getSalary)
        .max(Double::compare);
}
```



