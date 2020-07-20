[TOC]


# 概述

本章准备记录java的集合框架，

Collection：存储对象的集合

Map：存储键值对映射表



## Collection

### Collection<?>

```java
public interface Collection<E> extends Iterable<E> 
```

这个接口是集合的根结构，代表了一组元素。但是Collection<?>并不关心这组元素的内容。他只提供操作者组元素的基本操作方法，怎么添加，怎么删除，怎么循环。

```java
int size(); 
boolean isEmpty();
boolean contains(Object o);
Iterator<E> iterator();//Returns an iterator over the elements in this collection.
Object[] toArray();//Returns an array containing all of the elements in this collection.
<T> T[] toArray(T[] a);// the runtime type of the returned array is that of the specified array.
boolean add(E e);
boolean remove(Object o);
void clear();
```





### Set

* TreeSet : 基于红黑树实现，支持有序操作。查找的效率是 O(logN)
* HashSet：基于哈希表实现，支持快速查找，无序。查找效率是O(1)
* LinkedHashSet：具有HashSet的查找效率，内部使用双向链表维护元素的插入顺序。


### List
* ArrayList : 基于动态数组实现，支持随机访问。
* LinkedList：基于双向链表实现，只能顺序访问，但是可以快速的在链表中间插入和删除元素。LinkedList还可以用作栈、队列、和双向队列。
* Vector： 和ArrayList 类似，单他是线程安全的。

### Queue

* LinkedList: 可以用它来实现双向对列
* PriorityQueue: 基于堆结构实现，可以用来实现优先队列

## Map
* TreeMap：基于红黑树实现
* HashMap：基于哈希表实现
* Hashtable：和hashMap类似，但它是线程安全的。但是更应该是用ConcurrenthashMap, 效率更高，实现了分段锁
* LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者是做小使用顺序
* LinkedHashMap

# 二、容器中的设计模式

## 迭代器模式

### iterator()

```java
Iterator<E> iterator();
```

iterator方法返回一个实现了Iterator接口的对象。获取一个迭代器。

Iterator接口包换三个方法:

```java
public interface Iterator<E> {
    boolean hasNext();
    E next();
    default void remove() {throw new UnsupportedOperationException("remove");}
}
```

使用这个Iterator对象来迭代集合内的元素。foreach



## 适配器模式

java.util.Arrays#asList() 可以把数组类型转换为 List 类型。

```java 
@SafeVarargs
public static <T> List<T> asList(T... a)
```



# 三、源码分析



## ArrayList

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

1. **概览**

   因为ArrayList是基于数组实现的，所以支持快速随机访问。RandomAccess接口标识着可以支持随机访问。

   数组的默认大小为10。

   ```java
   private static final int DEFAULT_CAPACITY = 10;
   ```

2. **扩容**

   添加元素的时候使用 `ensureCapacityInternal()`方法来保证容量足够，如果不够时需要使用`grow()`来扩容，新的容量为 ` oldCapacity + (oldCapacity >> 1)` ，也就是旧容量的1.5倍。

   扩容操作，需要调用 `Arrays.copyof()`把原数组整个复制到新的数组中，这个代价比较高，最好在创建的时候就指定容量，减少扩容次数。

   ```java
   public boolean add(E e) {
       ensureCapacityInternal(size + 1);  // Increments modCount!!
       elementData[size++] = e;
       return true;
   }
   private static int calculateCapacity(Object[] elementData, int minCapacity) {
       if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
           return Math.max(DEFAULT_CAPACITY, minCapacity);
       }
       return minCapacity;
   }
   private void ensureCapacityInternal(int minCapacity) {
       ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
   }
   private void ensureExplicitCapacity(int minCapacity) {
       modCount++;
       // overflow-conscious code
       if (minCapacity - elementData.length > 0)
           grow(minCapacity);
   }
   private void grow(int minCapacity) {
       // overflow-conscious code
       int oldCapacity = elementData.length;
       int newCapacity = oldCapacity + (oldCapacity >> 1);
       if (newCapacity - minCapacity < 0)
           newCapacity = minCapacity;
       if (newCapacity - MAX_ARRAY_SIZE > 0)
           newCapacity = hugeCapacity(minCapacity);
       // minCapacity is usually close to size, so this is a win:
       elementData = Arrays.copyOf(elementData, newCapacity);
   }
   ```

3. **删除元素**

   需要调用 `System.arraycopy()`将 index+1后面的元素都复制到index位上，该操作时间复杂度为O(N)。

   ```java
   public E remove(int index) {
       rangeCheck(index);
       modCount++;
       E oldValue = elementData(index);
       int numMoved = size - index - 1;
       if (numMoved > 0)
           System.arraycopy(elementData, index+1, elementData, index,numMoved);
       elementData[--size] = null; // clear to let GC do its work
       return oldValue;
   }
   ```

4. 序列化

   ArrayList基于数组实现，并且具有动态扩容的特性，因此保存的元素不一定都会被使用，所以没必要全部序列化。

   保存元素的数组 elementData使用 transient修饰该关键字声明数组默认不会序列化。

   ```java
   transient Object[] elementData;
   ```

   ArrayList 实现了 writeObject()和readObject()来控制只序列化数组中有元素的部分。

   ```java 
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;
        // Read in size, and any hidden stuff
        s.defaultReadObject();
   	 // Read in capacity
        s.readInt(); // ignored
        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
            ensureCapacityInternal(size);
            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
   
   private void writeObject(java.io.ObjectOutputStream s)
       throws java.io.IOException{
       // Write out element count, and any hidden stuff
       int expectedModCount = modCount;
       s.defaultWriteObject();
       // Write out size as capacity for behavioural compatibility with clone()
       s.writeInt(size);
       // Write out all elements in the proper order.
       for (int i=0; i<size; i++) {
           s.writeObject(elementData[i]);
       }
       if (modCount != expectedModCount) {
           throw new ConcurrentModificationException();
       }
   }
   ```

5. Fail-Fast

   modCount 用来记录ArrayList结构发生变化的次数。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。

   在进行序列化或者迭代操作的时候，需要比较操作前后modCount是否改变，如果变了需要抛出ConcurrentModificationException。代码参考上节序列化中的 writeObject()方法。

















 