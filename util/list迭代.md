### 问题抛出

对于Java开发中的List实现类：ArrayList和LinkedList，可能你知道在频繁查询的时候使用ArrayList；在频繁增删的时候用LinkedList；不知道什么场景时用ArrayList。

不过你真的会使用ArrayList和LinkedList吗？

#### 问题1：循环依次读取LinkedList和ArrayList会发生什么情况？

1、LinkedList：

```java
@Test
public void linkedListTest() {

    List<Integer> linkedList = new LinkedList<>();
    //构造10数据的LinkedList
    for (int i = 0; i < 100000; i++) {
        linkedList.add(i);
    }

    //用for循环遍历LinkedList
    long start = System.currentTimeMillis();
    for (int i = 0; i < linkedList.size(); i++) {
        linkedList.get(i);
    }
    long end = System.currentTimeMillis();
    // 打印for循环遍历耗时
    System.out.println("for循环遍历100000条数据的LinkedList耗时：" + (end - start));

    //用迭代器while遍历LinkedList
    Iterator<Integer> it = linkedList.iterator();
    start = System.currentTimeMillis();
    while(it.hasNext()) {
        it.next();
    }
    end = System.currentTimeMillis();
    // 打印迭代器while循环耗时
    System.out.println("迭代器while循环遍历100000条数据的LinkedList耗时：" + (end - start));

}
```

打印结果：

```java
for循环遍历100000条数据的LinkedList耗时：4167
迭代器while循环遍历100000条数据的LinkedList耗时：2
```

2、ArrayList：

```java
@Test
public void arrayListTest() {

    List<Integer> arrayList = new ArrayList<>();
    //构造10数据的ArrayList
    for (int i = 0; i < 100000; i++) {
        arrayList.add(i);
    }

    long start = System.currentTimeMillis();
    //用for循环遍历ArrayList
    for (int i = 0; i < arrayList.size(); i++) {
        arrayList.get(i);
    }
    long end = System.currentTimeMillis();
    System.out.println("for循环遍历100000条数据的ArrayList耗时：" + (end - start));
    
    //用迭代器while遍历ArrayList
    Iterator<Integer> it = arrayList.iterator();
    start = System.currentTimeMillis();
    while(it.hasNext()) {
        it.next();
    }
    end = System.currentTimeMillis();
    System.out.println("迭代器while循环遍历100000条数据的ArrayList耗时：" + (end - start));
}
```

打印结果：

```java
for循环遍历100000条数据的ArrayList耗时：1
迭代器while循环遍历100000条数据的ArrayList耗时：2
```

鉴于ArrayList的for循环和迭代器while循环耗时不明显，对ArrayList操作数据再加大10倍，1000000（一百万）条数据：

```java
@Test
public void arrayListTest() {

    List<Integer> arrayList = new ArrayList<>();
    //构造10数据的ArrayList
    for (int i = 0; i < 1000000; i++) {
        arrayList.add(i);
    }

    long start = System.currentTimeMillis();
    //用for循环遍历ArrayList
    for (int i = 0; i < arrayList.size(); i++) {
        arrayList.get(i);
    }
    long end = System.currentTimeMillis();
    System.out.println("for循环遍历100000条数据的ArrayList耗时：" + (end - start));
    
    //用迭代器while遍历ArrayList
    Iterator<Integer> it = arrayList.iterator();
    start = System.currentTimeMillis();
    while(it.hasNext()) {
        it.next();
    }
    end = System.currentTimeMillis();
    System.out.println("迭代器while循环遍历100000条数据的ArrayList耗时：" + (end - start));
}
```

打印结果（测试了几次，都是相差不大，为了说明问题，取相差较大的数据）：

```java
for循环遍历100000条数据的ArrayList耗时：3
迭代器while循环遍历100000条数据的ArrayList耗时：5
```

从上面的结果中可以明显看到，循环依次读取ArrayList和LinkedList时，可以看到：

（1）使用for循环遍历LinkedList和使用迭代器循环遍历LinkedList时（100000的数据），耗时相差巨大。

（2）使用for循环遍历ArrayList和使用迭代器循环遍历ArrayList时（100000的数据），耗时相差不大。

#### 问题2：对LinkedList和ArrayList进行增删操作会发生什么情况？

1、LinkedList：



### 列表数据结构概述

列表是一种数据项构成的有限序列，即按照一定的线性顺序，排列而成的数据项的集合，在这种数据结构上进行的基本操作包括对元素的的查找，插入，和删除。

列表的两种主要表现是数组和链表，在Java中对应的是java.util.ArrayList和java.util.LinkedList。

注：栈和队列是两种特殊类型的列表，底层实现也是数组或链表。

### ArrayList

![](..\images\array.png)

所谓数组，是连续有序的元素序列。 若将有限个类型相同的变量的集合命名，那么这个名称为数组名。组成数组的各个变量称为数组的分量，也称为数组的元素。用于区分数组的各个元素的数字编号称为下标（默认下标从0开始）。即：连续有序、类型相同。