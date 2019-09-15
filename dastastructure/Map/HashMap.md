<https://www.jianshu.com/p/4aa3bb16f36c>

## 概述

HashMap是基于哈希表(散列表)，数据结构是“链表散列”，也就是数组+链表 ，key唯一的value可以重复，允许存储null 键null 值，元素无序。

### 哈希表

数组：一段连续控件存储数据，指定下标的查找，时间复杂度O(1),通过给定值查找，需要遍历数组，自已对比复杂度为O（n） 二分查找插值查找，复杂度为O(logn)。

线性链表：新增和删除结点时，时间复杂度O(1)，查找需要遍历也就是O(n)。

二叉树：对一颗相对平衡的有序二叉树，对其进行插入，查找，删除，平均复杂度O(logn)。

哈希表：哈希表中进行添加，删除，查找等操作，性能十分之高，不考虑哈希冲突的情况下，仅需一次定位即可完成，时间复杂度为O(1)哈希表的主干就是数组。

### 哈希冲突

如果两个不同的元素，通过哈希函数得出的实际存储地址相同。即对某个元素进行哈希运算，得到一个存储地址，然后要进行插入的时候，发现已经被其他元素占用了，其实这就是所谓的哈希冲突，也叫哈希碰撞。哈希冲突的解决方案有多种:开放定址法（发生冲突，继续寻找下一块未被占用的存储地址），再散列函数法，链地址法，而HashMap即是采用了链地址法，也就是数组+链表的方式。

## 运算符基础

### &（按位与）

按位与运算符“&”是双目运算符。其功能是参与运算的两数各对应的二进位相与。只有对应的两个二进位都为1时，结果位才为1。

```java
int num1 = 7; // 对应的二进制：00000000 00000000 00000000 00000111
int num2 = 6; // 对应的二进制：00000000 00000000 00000000 00000110

	00000000 00000000 00000000 00000111
&   00000000 00000000 00000000 00000110
----------------------------------------------
    00000000 00000000 00000000 00000110    
```

```java
System.out.println(7 & 6); // 6
```

### &&（逻辑与）

逻辑与运算符“&&”是双目运算符。只有两个操作数都是真，结果才是真。 逻辑与操作属于短路操作，既如果第一个操作数能够决定结果，那么就不会对第二个操作数求值。对于逻辑与操作而言，如果第一个操作数是假，则无论第二个操作数是什么值，结果都不可能是真，相当于短路了右边。

```java
int num = 1;
if (num == 1 && (++num) == 2) {
    System.out.println("true, num = " + num);
} else {
    System.out.println("false, num = " + num);
}
// 打印结果：true, num = 2
```

从结果来看，++num执行了。

```java
int num = 1;
if (num == 2 && (++num) == 2) {
    System.out.println("true, num = " + num);
} else {
    System.out.println("false, num = " + num);
}
// 打印结果：false, num = 1
```

从结果来看，++num没有执行。

### |（按位或）

按位或运算符“|”是双目运算符。其功能是参与运算的两数各对应的二进位相或。只要对应的二个二进位有一个为1时，结果位就为1。

```java
int num1 = 7; // 对应的二进制：00000000 00000000 00000000 00000111
int num2 = 6; // 对应的二进制：00000000 00000000 00000000 00000110

	00000000 00000000 00000000 00000111
|   00000000 00000000 00000000 00000110
----------------------------------------------
    00000000 00000000 00000000 00000111 
```

```java
System.out.println(7 | 6); // 7
```

### ||（逻辑或）

逻辑或运算符“|”是双目运算符。如果一个操作数或多个操作数为 true，则逻辑或运算符返回布尔值 true；只有全部操作数为false，结果才是 false。

```java

```



## HashMap的数据结构

### HashMap各个属性的含义

```java
// 默认初始容量（数组大小）
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

//1073741824最大的容量范围
static final int MAXIMUM_CAPACITY = 1 << 30;

//加载因子默认0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//加载因子默认0.75
static final int TREEIFY_THRESHOLD = 8;

//当进行resize()时，假如树中节点小于这个值时，用链表代替树，默认值为6
static final int UNTREEIFY_THRESHOLD = 6;

//当整个hashMap中元素数量大于64时，也会进行转为红黑树结构。
static final int MIN_TREEIFY_CAPACITY = 64;

//存储数据的entry数组
transient Node<K,V>[] table;

// Entry元素的Set集合，用于iterator迭代时获取Entry的Set集合
transient Set<Map.Entry<K,V>> entrySet;

//数组的大小
transient int size;

//修改的次数
transient int modCount;

//临界值=加载因子*初始容量（当size大于临界值就会出现数组扩充到原来2 倍）
int threshold;

//加载因子，不指定时默认0.75
final float loadFactor;
```

### Node节点

```java
static class Node<K,V> implements Map.Entry<K,V> {
    // key对应的hash值
    final int hash;
    // key值
    final K key;
    // value值
    V value;
    // 指向下一个Node节点的指针
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

## HashMap初始化

```java
// 无参构造函数，加载因子为默认值0.75
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; 
}
// 指定数组初始容量构造函数，加载因子为默认值0.75
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
// 指定数组初始容量和加载因子构造函数
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("...");
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("...");
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

## 新增元素

### put(K, V)

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
// 计算Key的hash值
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```









<https://blog.csdn.net/m0_37914588/article/details/82287191>