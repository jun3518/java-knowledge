## ArrayList简介

ArrayList类继承关系如下：

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

从上面ArrayList的继承关系可知，ArrayList实现了List接口，同时ArrayList也实现了RandomAccess接口，其中RandomAccess是一个标识接口，只要List集合实现这个接口，就能支持快速随机访问。

ArrayList类的成员变量关系如下：

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    
    // 默认初始容量
    private static final int DEFAULT_CAPACITY = 10;
	// 空数组，如果数组初始化长度指定为0时，将此值赋值给elementData
    private static final Object[] EMPTY_ELEMENTDATA = {};
	// 如果数组没有指定长度，那么初始化时DEFAULTCAPACITY_EMPTY_ELEMENTDATA赋值给elementData
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
	// 存储数据的数组
    transient Object[] elementData; 
	// 当前存储的数据量
    private int size;
    // 修改次数，用于并发修改校验
    protected transient int modCount = 0;
}
```

## ArrayList初始化

```java
// 无参构造函数
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
// 有参构造函数
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // 判断数组类型是否一样
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // 赋值空数组
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
// 指定初始化时数组长度
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    }
}
```

ArrayList提供了三个构造函数，如果初始化时没有指定数组长度，那么默认为空数组，逻辑比较简单。

## ArrayList新增元素

### add(E)

```java
public boolean add(E e) {
    // 校验或扩容
    ensureCapacityInternal(size + 1); 
    elementData[size++] = e;
    return true;
}
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 如果数组为空，那么对返回数组的Capacity=10
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    // 返回minCapacity，其中minCapacity=size+1
    return minCapacity;
}
// 校验是否需要扩容
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // 如果当前需要的数组容量大于数组长度，那么需要进行扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
/*
	数组扩容，1.5倍扩容，即旧的数组长度是oldCapacity=10，则10+(10/2)=15,
	但是如果旧的数组长度是1（意味着minCapacity=2），
	那么newCapacity=1+(1/2)=1，这显然不符合扩容的目的，
	所以会进一步做判断：如果(newCapacity - minCapacity < 0)，那么newCapacity=2
*/
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

从上面的代码分析，时间复杂度为O(1)。

### add(int, E)

```java
public void add(int index, E element) {
    // 校验index
    rangeCheckForAdd(index);
	// 校验是否需要扩容
    ensureCapacityInternal(size + 1); 
    // 将index位的数据都向后移动一位，留出index位置给index插入
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    // 在index位插入元素E
    elementData[index] = element;
    size++;
}
```

从上面的代码分析，平均移动的元素为n/2个，所以时间复杂度为O(n)。

### addAll(Collection)

```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    // 校验是否需要扩容
    ensureCapacityInternal(size + numNew); 
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
```

addAll(Collection)方法是在数组末尾追加元素。从上面的代码分析，时间复杂度为Collection的长度，所以时间复杂度为O(n)，n为Collection的长度。

### addAll(int, Collection)

```java
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);
    int numMoved = size - index;
	// 如果index的位置不是数组的末尾，则index之后的元素需要后移Collection数组长度个位置
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);
	// 将Collection元素插入数组中
    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

addAll(Collection)方法是在数组末尾追加元素。从上面的代码分析，需要移动index位之后的元素向后移动Collection长度个位置，所以时间复杂度为Collection的长度O(n) + 移动的元素个数O(m)，所以时间复杂度为O(n+m)，所以可以简化为O(n)。

## ArrayList删除元素

### remove(int)

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // 将移除元素后的资源回收
    elementData[--size] = null;
    return oldValue;
}
```

从上面的代码分析，需要移动index位之后的元素移动一位，所以平均移动的元素个数为O(n/2)，所以时间复杂度为O(n)。

### remove(Object)

```java
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; 
}
```

remove(Object)首先会判断数组中的元素的equals是否相等，如果相等，则获取该元素的index位置，接着移除该位置的元素。

首先获取该元素的index位置遍历元素的个数平均为n/2，即为O(n)；接着移除该元素，根据移除元素的分析，会平均移动的元素个数m/2，即耗时为O(m/2)，所以总时间复杂度为O(n)。

### removeAll(Collection)

```java
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, false);
}
/*
代码逻辑：遍历数组，将不存在于Collection中的元素，放在数组的w++位置上。
结束后，将w位置之后的元素回收（置为null）
*/
private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;
    boolean modified = false;
    try {
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {
        // 如果存在并发修改，那么可能会导致r!=size，这是将r位置之后的元素从w位置开始赋值size-r个元素
        if (r != size) {
            System.arraycopy(elementData, r, elementData, w, size - r);
            w += size - r;
        }
        // 将w位置之后的元素回收（置为null）
        if (w != size) {
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
    }
    return modified;
}
```

假设Collection的元素个数为n，而数组的元素个数为m，那么Collection.contains方法耗时为O(n)，所以总的耗时为O(m*n)，所以removeAll(Collection)总时间复杂度为O(n^2)。

### retainAll(Collection)

```java
public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, true);
}
```

从上面的代码分析，retainAll方法和removeAll方法逻辑一样，所以时间复杂度为O(n^2)。

## ArrayList修改元素

### set(int, E)

```java
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

从上面的代码分析，set(int, E)方法的时间复杂度为O(1)。

## ArrayList查询元素

### get(int)

```java
public E get(int index) {
    rangeCheck(index);
    return elementData(index);
}
E elementData(int index) {
    return (E) elementData[index];
}
```

从上面的代码分析，所以get(int)方法的时间复杂度为O(1)。

### iterator()

```java
// java.util.ArrayList#iterator
public Iterator<E> iterator() {
    return new Itr();
}
private class Itr implements Iterator<E> {
    //游标，记录当前指针指向的元素位置
    int cursor;
    // 记录返回元素的位置
    int lastRet = -1; 
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

### listIterator()

```java
// java.util.ArrayList#listIterator()
public ListIterator<E> listIterator() {
    return new ListItr(0);
}
// java.util.ArrayList.ListItr
private class ListItr extends Itr implements ListIterator<E> {
    ListItr(int index) {
        super();
        cursor = index;
    }

    public boolean hasPrevious() {
        return cursor != 0;
    }

    public int nextIndex() {
        return cursor;
    }

    public int previousIndex() {
        return cursor - 1;
    }

    @SuppressWarnings("unchecked")
    public E previous() {
        checkForComodification();
        int i = cursor - 1;
        if (i < 0)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i;
        return (E) elementData[lastRet = i];
    }

    public void set(E e) {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.set(lastRet, e);
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    public void add(E e) {
        checkForComodification();

        try {
            int i = cursor;
            ArrayList.this.add(i, e);
            cursor = i + 1;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
}
```

listIterator()方法不仅有iterator()的所有api功能，还提供了remove()、set(E e)、add(E e)的api，可以在遍历ArrayList的时候，对ArrayList进行删、改、增的操作。

遍历ArrayList的操作时遍历整个ArrayList的元素，所有时间复杂度为O(n)。

## ArrayList的for循环（或增强for循环）和iterator()比较

```java
public static void main(String[] args) {

    List<Integer> dataList = new ArrayList<Integer>(10000000);

    for (int i = 0; i < 10000000; i++) {
        dataList.add(i);
    }


    long start = System.nanoTime();
    for (int i = 0; i < dataList.size(); i++) {
        dataList.get(i);
    }
    long end = System.nanoTime();
    System.out.println("for耗时：" + (end - start));
    start = System.nanoTime();
    Iterator<Integer> it = dataList.iterator();
    while(it.hasNext()) {
        it.next();
    }
    end = System.nanoTime();
    System.out.println("iterator耗时：" + (end - start));

}
```

打印结果：

```java
for耗时：3158201
iterator耗时：5815200
```

从上面的iterator()循环的分析看来，ArrayList的for循环和iterator循环耗时有一些差距，原因在于iterator在遍历的时候，还会做一些校验等操作，所以相对来说耗时较多。

很多时候，不知道接口返回的List是什么类型。对于LinkedList，for循环和iterator遍历性能上iterator优于for循环。那么如果在不知道List是什么类型的情况下，优先考虑使用iterator遍历循环。或者对List进行list instanceof RandomAccess判断，如果为true，则使用for循环，否则使用iterator遍历循环。

## ArrayList运用场景

在上述ArrayList增删改查的时间复杂度分析中发现，对于查（下标index查找）、改的时候，时间复杂度为O(1)，而在增（指定位置）、删（指定位置）的时候，时间复杂度为O(n)。所以对于需要频繁查询和修改的列表，建议使用ArrayList；所以对于需要频繁增加和删除的列表，不建议使用ArrayList，而是使用LinkedList。