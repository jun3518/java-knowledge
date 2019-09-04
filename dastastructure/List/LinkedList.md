## LinkedList简介

LinkedList类继承关系如下：

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

从上面LinkedList的继承关系可知，LinkedList实现了List接口，但是LinkedList同时又实现了Deque接口，Deque是一个双端队列的定义接口，这里可以说明一下，LinkedList底层实现是双向队列。

LinkedList类的成员变量关系如下：

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable {
    
    transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;
    
    protected transient int modCount = 0;
}
```

对于成员变量Node，这到底是什么呢？跟踪Node：

```java
private static class Node<E> {
    E item;
    // 下一个Node节点指针
    Node<E> next;
    // 上一个Node节点指针
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

从上面的代码可看出，Node是一个私有静态内部类，其结构表明了Node是链表的一个节点，其中E为要存储的内容，next为指向下一个Node节点的指针，prev为指向上一个Node节点指针。

## LinkedList初始化

```java
// 无参构造函数
public LinkedList() {
}
// 有参构造函数
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

LinkedList提供了两个构造函数，为什么LinkedList不提供像ArrayList(int initialCapacity)初始容量的构造函数呢？这就涉及了列表的两种形式：数组和链表。ArrayList底层实现是数组，可以预先分配连续的内存空间，避免频繁扩容的问题，而LinkedList底层实现是链表（双向链表），是不连续的，需要通过指针进行关联，没有频繁扩容导致的性能问题，而且LinkedList需要通过last指针指定最后一个真实存储元素的位置，LinkedList初始化时没有存储元素，所以初始化时初始容量没有任何意义。

## LinkedList新增元素

### add(E)

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}
/*
默认添加元素：
使用变量l暂存last指向的当前最后一个元素，
接着构建一个Node元素存储新增的元素，
last指针指向新增的元素，
变量l指向未添加元素前，链表的最后一个元素，此时将l指向的节点的next指针指向新增的元素，
此时：last指向了新增的元素，即：last还是指向最后一个元素，
而之前的最后一个元素的next指针指向了新增的元素，从而实现了链表元素的新增
*/
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    // last指针指向链表末位新增的Node元素
    last = newNode;
    // 如果l为null，说链表为空，新增的节点使用first指针指向新增的元素
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

从上面的代码分析，时间复杂度为O(1)。

### addFirst(E)

```java
public void addFirst(E e) {
    linkFirst(e);
}
/*
在首位添加元素：
使用变量f暂存first指向的第一个元素，
接着构建一个Node元素存储新增的元素，
first指针指向新增的元素，
变量f指向未添加元素前，链表的第一个元素，此时将f指向的节点的prev指针指向新增的元素，
此时：first指向了新增的元素，即：first还是指向第一个元素，
而之前的第一个元素的prev指针指向了新增的元素，从而实现了链表元素的新增
*/
private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);
    // first指针指向在链表首位新增的Node元素
    first = newNode;
    // 如果f为null，说链表为空，新增的节点使用last指针指向新增的元素
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}
```

从上面的代码分析，时间复杂度为O(1)。

### addLast(E)

```java
public void addLast(E e) {
    linkLast(e);
}
```

addLast(E)方法和add(E)方法的逻辑一样，都是在链表末尾追加元素。

从上面的代码分析，时间复杂度为O(1)。

## LinkedList删除元素

### remove()

```java
public E remove() {
    return removeFirst();
}
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
private E unlinkFirst(Node<E> f) {
    final E element = f.item;
    // 记录链表首位Node节点的下一个节点
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; // help GC
    // first指针记录下一个节点
    first = next;
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}
```

从上面的代码分析，unlinkFirst的时间复杂度为O(1)，所以remove()方法的时间复杂度为O(1)。

### removeFirst()

```java
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
```

remove()方法其实是调用了removeFirst()方法，所以所以removeFirst()方法的时间复杂度为O(1)。

### removeLast()

```java
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}
private E unlinkLast(Node<E> l) {
    final E element = l.item;
    // 记录最后一个节点的上一个节点
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null; // help GC
    // last指针指向最后一个节点的上一个节点
    last = prev;
    if (prev == null)
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    return element;
}
```

从上面的代码分析，unlinkLast的时间复杂度为O(1)，所以removeLast()方法的时间复杂度为O(1)。

### remove(Object)

```java
public boolean remove(Object o) {
	// 如果移除的元素可能为null，如果为null的情况，那么使用item==null进行比较
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        // 如果移除的元素不为null，如果为null的情况，那么使用o.equals(item)进行比较,
        // 因为item可能为null
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

从上面的代码分析，时间复杂度为O(n)。

### remove(int)

```java
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
/*
	移除index位置的Node元素：
	将要移除的Node的prev指向的元素的next指针，指向将要移除的Node的next指向的元素;
	将要移除的Node的next指向的元素的prev指针，指向将要移除的Node的prev指向的元素;
*/
E unlink(Node<E> x) {
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }
    x.item = null;
    size--;
    modCount++;
    return element;
}
// 获取index位置的Node元素
Node<E> node(int index) {
	// 如果index的位置：0< index < size/2，则从first开始查找；否则从last查找
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

从上面的代码分析，node方法的时间复杂度为O(n/2)，即为O(n)，unlink方法的时间复杂度为O(1)，所以remove(int)方法的时间复杂度为O(n)。

## LinkedList修改元素

### set(int, E)

```java
public E set(int index, E element) {
    checkElementIndex(index);
    // 获取index位置的Node节点
    Node<E> x = node(index);
    E oldVal = x.item;
    // 将index节点的item值替换成新元素E
    x.item = element;
    return oldVal;
}
```

从上面的代码分析，node方法的时间复杂度为O(n/2)，set(int, E)方法的时间复杂度为O(n)。

## LinkedList查询元素

### get(int)

```java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
// 查找index位置的Node节点
Node<E> node(int index) {
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

从上面的代码分析，node方法的时间复杂度为O(n/2)，即为O(n)，所以get(int)方法的时间复杂度为O(n)。

### getFirst()

```java
public E getFirst() {
    final Node<E> f = first;
    // first指针没有指向元素，空链表
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}
```

从上面的代码分析，getFirst()方法的时间复杂度为O(1)。

### getLast()

```java
public E getLast() {
    final Node<E> l = last;
    // last指针没有指向元素，空链表
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}
```

从上面的代码分析，getLast()方法的时间复杂度为O(1)。

### iterator()

```java
// java.util.AbstractSequentialList#iterator
public Iterator<E> iterator() {
    return listIterator();
}
public ListIterator<E> listIterator() {
    return listIterator(0);
}
// java.util.AbstractList#listIterator()
public ListIterator<E> listIterator() {
    return listIterator(0);
}
// java.util.LinkedList#listIterator
public ListIterator<E> listIterator(int index) {
    checkPositionIndex(index);
    return new ListItr(index);
}
// java.util.LinkedList.ListItr
private class ListItr implements ListIterator<E> {
    // 记录当前返回的Node节点的指针
    private Node<E> lastReturned;
    // 记录下一个Node节点的指针
    private Node<E> next;
    // 记录下一个节点的位置
    private int nextIndex;
    // 记录当前修改的次数，用于校验并发修改
    private int expectedModCount = modCount;

    ListItr(int index) {
        // assert isPositionIndex(index);
        next = (index == size) ? null : node(index);
        nextIndex = index;
    }

    public boolean hasNext() {
        return nextIndex < size;
    }
	// 返回下一个节点的值
    public E next() {
        checkForComodification();
        if (!hasNext())
            throw new NoSuchElementException();
		// 要返回的Node节点
        lastReturned = next;
        // 记录下一个节点
        next = next.next;
        // 记录下一个节点的位置
        nextIndex++;
        return lastReturned.item;
    }
	// 判断是否能返回当前位置的上一个节点的元素
    public boolean hasPrevious() {
        return nextIndex > 0;
    }
	// 返回当前位置的上一个节点的元素
    public E previous() {
        checkForComodification();
        if (!hasPrevious())
            throw new NoSuchElementException();

        lastReturned = next = (next == null) ? last : next.prev;
        nextIndex--;
        return lastReturned.item;
    }

    public int nextIndex() {
        return nextIndex;
    }

    public int previousIndex() {
        return nextIndex - 1;
    }

    public void remove() {
        checkForComodification();
        if (lastReturned == null)
            throw new IllegalStateException();

        Node<E> lastNext = lastReturned.next;
        unlink(lastReturned);
        if (next == lastReturned)
            next = lastNext;
        else
            nextIndex--;
        lastReturned = null;
        expectedModCount++;
    }

    public void set(E e) {
        if (lastReturned == null)
            throw new IllegalStateException();
        checkForComodification();
        lastReturned.item = e;
    }

    public void add(E e) {
        checkForComodification();
        lastReturned = null;
        if (next == null)
            linkLast(e);
        else
            linkBefore(e, next);
        nextIndex++;
        expectedModCount++;
    }

    public void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (modCount == expectedModCount && nextIndex < size) {
            action.accept(next.item);
            lastReturned = next;
            next = next.next;
            nextIndex++;
        }
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
// java.util.AbstractList#listIterator()
public ListIterator<E> listIterator() {
    return listIterator(0);
}
// java.util.LinkedList#listIterator
public ListIterator<E> listIterator(int index) {
    checkPositionIndex(index);
    return new ListItr(index);
}
// java.util.LinkedList.ListItr
private class ListItr implements ListIterator<E> {
    private Node<E> lastReturned;
    private Node<E> next;
    private int nextIndex;
    private int expectedModCount = modCount;

    ListItr(int index) {
        // assert isPositionIndex(index);
        next = (index == size) ? null : node(index);
        nextIndex = index;
    }

    public boolean hasNext() {
        return nextIndex < size;
    }

    public E next() {
        checkForComodification();
        if (!hasNext())
            throw new NoSuchElementException();

        lastReturned = next;
        next = next.next;
        nextIndex++;
        return lastReturned.item;
    }

    public boolean hasPrevious() {
        return nextIndex > 0;
    }

    public E previous() {
        checkForComodification();
        if (!hasPrevious())
            throw new NoSuchElementException();

        lastReturned = next = (next == null) ? last : next.prev;
        nextIndex--;
        return lastReturned.item;
    }

    public int nextIndex() {
        return nextIndex;
    }

    public int previousIndex() {
        return nextIndex - 1;
    }

    public void remove() {
        checkForComodification();
        if (lastReturned == null)
            throw new IllegalStateException();

        Node<E> lastNext = lastReturned.next;
        unlink(lastReturned);
        if (next == lastReturned)
            next = lastNext;
        else
            nextIndex--;
        lastReturned = null;
        expectedModCount++;
    }

    public void set(E e) {
        if (lastReturned == null)
            throw new IllegalStateException();
        checkForComodification();
        lastReturned.item = e;
    }

    public void add(E e) {
        checkForComodification();
        lastReturned = null;
        if (next == null)
            linkLast(e);
        else
            linkBefore(e, next);
        nextIndex++;
        expectedModCount++;
    }

    public void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (modCount == expectedModCount && nextIndex < size) {
            action.accept(next.item);
            lastReturned = next;
            next = next.next;
            nextIndex++;
        }
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

listIterator()方法不仅有iterator()的所有api功能，还提供了remove()、set(E e)、add(E e)的api，可以在遍历LinkedList的时候，对LinkedList进行删、改、增的操作。

遍历LinkedList的操作时遍历整个LinkedList的元素，所有时间复杂度为O(n)。

## LinkedList运用场景

在上述LinkedList增删改查的时间复杂度分析中发现，对于增（在首、尾新增）、删（在在首、尾新删）的时候，时间复杂度为O(1)，而在改（指定位置）、查的时候，时间复杂度为O(n)。所以对于需要频繁增删的列表，建议使用LinkedList。