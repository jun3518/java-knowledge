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





<https://blog.csdn.net/m0_37914588/article/details/82287191>