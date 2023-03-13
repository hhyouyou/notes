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
* Vector： 和ArrayList 类似，但他是线程安全的。

### Queue

* LinkedList: 可以用它来实现双向对列
* PriorityQueue: 基于堆结构实现，可以用来实现优先队列

## Map
* TreeMap：基于红黑树实现
* HashMap：基于哈希表实现
* Hashtable：和hashMap类似，但它是线程安全的。但是更应该是用ConcurrenthashMap, 效率更高，实现了分段锁
* LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者是做小使用顺序

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

将数组转为List, 让数组可以适配List中的方法。





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



## Vector

### 1. 同步

vector的实现和ArrayList类似，但是使用了synchronized进行同步。

```java
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}
public synchronized E get(int index) {
    if (index >= elementCount)throw new ArrayIndexOutOfBoundsException(index);
    return elementData(index);
}
```





### 2. 扩容

Vector 的构造函数可以传入 capacityIncrement 参数，它的作用是在扩容时使容量capacity增长 capacityIncrement。如果这个参数的值小于等于0，扩容时每次都令capacity为原来的两倍。

​	

### 3.与ArrayList的比较

* Vector是同步的，因此开销就比ArrayList要大，访问速度更慢。最好使用ArrayList而不是Vector，因为同步操作完全可以由外层控制
* Vector每次扩容默认是2倍，而ArrayList是1.5倍。



## HashMap

JDK 1.8

### 1. 存储结构

内部包含了一个Node数组。Node实现了Map.Entry<k,V>接口。存着hash值、键值对和下一个节点。从next可以看出map中的每个元素都可以是链表。数组+链表

```java
transient Node<K,V>[] table;

 static class Node<K,V> implements Map.Entry<K,V> {
     final int hash;
     final K key;
     V value;
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



### 2. 添加元素 put操作



```java
  /**
     * Implements Map.put and related methods.
     *
     * @param hash         hash for key
     * @param key          the key
     * @param value        the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict        if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K, V>[] tab;
        Node<K, V> tabNode;
        int tabLength, index;
        tab = table;
        tabLength = tab.length;
        if (tab == null || tabLength == 0) {
            tab = resize();
            tabLength = tab.length;
        }
        // 根据key的hash码, 和hashMap的长度，计算这个键值对放在数组的哪个下标
        index = (tabLength - 1) & hash;
        tabNode = tab[index];
        if (tabNode == null) {
            // 如果这个下标的node还是空的， 那就直接插入完事儿
            tab[index] = newNode(hash, key, value, null);
        } else {
            //  要插入的这个下标已经有元素了，这个时候，就是hash冲突，需要扩展链表了
            Node<K, V> newNode;
            K k;
            k = tabNode.key;
            // if (tabNode.hash == hash && (k == key || (key != null && key.equals(k)))) {
            if (tabNode.hash == hash && (Objects.equals(key, k))) {
                // 如果两个key的hash一样，值也一样，那么就替换value呗
                newNode = tabNode;
            } else if (tabNode instanceof TreeNode) {
                // 如果是个treeNode(双向),直接加了，这里边是给树加元素
                newNode = ((TreeNode<K, V>) tabNode).putTreeVal(this, tab, hash, key, value);
            } else {
                // 这边用来处理链表，遍历节点
                for (int binCount = 0; ; ++binCount) {
                    if ((newNode = tabNode.next) == null) {
                        tabNode.next = newNode(hash, key, value, null);
                        // -1 for 1st  这里面就是大于 TREEIFY_THRESHOLD = 8 转换为树了
                        if (binCount >= TREEIFY_THRESHOLD - 1) {
                            // 见下文链表树化
                            treeifyBin(tab, hash);
                        }
                        break;
                    }
                    // 有相等元素，就直接出去了
                    if (newNode.hash == hash &&
                            ((k = newNode.key) == key || (key != null && key.equals(k)))) {
                        break;
                    }
                    tabNode = newNode;
                }
            }
            // existing mapping for key
            if (newNode != null) {
                V oldValue = newNode.value;
                if (!onlyIfAbsent || oldValue == null) {
                    newNode.value = value;
                }
                // 将节点移动到最后，jdk8用的尾插法
                afterNodeAccess(newNode);
                return oldValue;
            }
        }
        // 修改次数
        ++modCount;
        // 插入新元素后，元素的数量是否大于 容量*负载因子
        if (++size > threshold) {
            // 扩容
            resize();
        }
        // 是否需要把最后一个元素删掉
        afterNodeInsertion(evict);
        return null;
    }
```





### 3. 扩容

```java
final Node<K, V>[] resize() {
        Node<K, V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                // 容量已经最大了，只能把阈值也调整到最大，不管什么负载因子了
                threshold = Integer.MAX_VALUE;
                return oldTab;
            } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY) {
                // 左移1位 double threshold
                newThr = oldThr << 1;
            }
        } else if (oldThr > 0) {
            // 原来的容量<= 0 阈值>0
            // initial capacity was placed in threshold
            newCap = oldThr;
        } else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float) newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ? (int) ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"unchecked"})
        Node<K, V>[] newTab = (Node<K, V>[]) new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            // 遍历老数组下标
            for (int j = 0; j < oldCap; ++j) {
                Node<K, V> node = oldTab[j];
                if (node != null) {
                    oldTab[j] = null;
                    // 元素不为空，扩容时需要处理
                    if (node.next != null) {
                        if (node instanceof TreeNode) {
                            // 如果是树节点，由这个split来处理
                            ((TreeNode<K, V>) node).split(this, newTab, j, oldCap);
                        } else { // preserve order
                            // 是链表这边处理
                            // 老索引处链表，头、尾
                            Node<K, V> loHead = null, loTail = null;
                            // 新索引处链表，头、尾
                            Node<K, V> hiHead = null, hiTail = null;
                            Node<K, V> next;
                            // 循环处理数组索引j位置上哈希冲突的链表中每个元素
                            do {
                                next = node.next;
                          // 判断key的hash值与老数组长度与操作后结果决定元素是放在原索引处还是新索引
                                if ((node.hash & oldCap) == 0) {
                                    // 放在原索引处 建立新链表
                                    if (loTail == null) {
                                        loHead = node;
                                    } else {
                                        loTail.next = node;
                                    }
                                    loTail = node;
                                } else {
                                    // 在新索引处，建立新链表
                                    if (hiTail == null) {
                                        hiHead = node;
                                    } else {
                                        hiTail.next = node;
                                    }
                                    hiTail = node;
                                }
                            } while ((node = next) != null);
                            if (loTail != null) {
                                loTail.next = null;
                                newTab[j] = loHead;
                            }
                            if (hiTail != null) {
                                hiTail.next = null;
                                newTab[j + oldCap] = hiHead;
                            }
                        }
                    } else {
                        // 不是链表，就把hash和新长度与一下，算出新下标
                        newTab[node.hash & (newCap - 1)] = node;
                    }
                }
            }
        }
        return newTab;
    }
```



### 4. 链表树化

在上面put操作中，如果链表添加节点时，节点数大于8，那么就会进入链表树化的方法。

> 但是不是说，进入这个方法就一定会进行红黑树的转换。 此处还需要判断，桶的大小<64? 扩容：树化



```java
final void treeifyBin(Node<K, V>[] tab, int hash) {
    int tabLength, index;
    Node<K, V> e;
    // 进行树化之前，需要判断，桶的大小是否小于 64。先扩容到64，之后才
    if (tab == null || (tabLength = tab.length) < MIN_TREEIFY_CAPACITY) {
        resize();
    } else if ((e = tab[index = (tabLength - 1) & hash]) != null) {
        TreeNode<K, V> head = null, tail = null;
        do {
            // 将节点转换为树节点，但还不是红黑树
            TreeNode<K, V> p = replacementTreeNode(e, null);
            if (tail == null) {
                head = p;
            } else {
                p.prev = tail;
                tail.next = p;
            }
            tail = p;
        } while ((e = e.next) != null);
        if ((tab[index] = head) != null) {
            // 转红黑树
            head.treeify(tab);
        }
    }
}
```

上面这个方法比较简单，只是把普通过的`Node`节点转换成 `TreeNode`节点。真正的红黑树转换还在下面：



```java



```













## ConcurrentHashMap

1.7 中使用的是 分段锁Segment 继承自 重入锁ReentrantLock。

1.8 中使用的是CAS操作来支持更高的并发度，在 CAS 操作失败时使用内置锁 synchronized。



CAS(compare and swap )， 比较并交换， 故名思意。在进行修改值的操作时，会比较旧值。

比如**”线程1“**将key` a =1` 修改成 `a=2`, 那么会进行原子操作比较，是不1改成2，如果是，则修改成功。

如果此时有**“线程2”**率先修改成功让`a=3`, 那么**”线程1“**将无法让`a=2`



## LinkedHashMap

### 存储结构

继承自HashMap，所以可以根据hash快速查找

```
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>{
```

自己维护了一个双向链表

```java
   /**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;
    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail;
 static class Entry<K,V> extends HashMap.Node<K,V> {
     Entry<K,V> before, after;
     Entry(int hash, K key, V value, Node<K,V> next) {
         super(hash, key, value, next);
     }
 }
```

accessOrder 决定了顺序，默认为 false，此时维护的是插入顺序。true对应访问顺序，插入顺序为false

```java
final boolean accessOrder;
```

LinkedHashMap 最重要的是以下用于维护顺序的函数，它们会在 put、get 等方法中调用。

```java
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
```



### afterNodeAccess

用来维护访问顺序，每次都把最后一次访问的节点，移动到最后

当一个节点被访问时，如果 accessOrder 为 true，则会将该节点移到链表尾部。也就是说指定为 LRU 顺序之后，在每次访问一个节点时，会将这个节点移到链表尾部，保证链表尾部是最近访问的节点，那么链表首部就是最近最久未使用的节点。

```java

    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
```



### afterNodeInsertion()

在 put 等操作之后执行，当 removeEldestEntry() 方法返回 true 时会移除最晚的节点，也就是链表首部节点 first。

evict 只有在构建 Map 的时候才为 false，在这里为 true。

```
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```

removeEldestEntry() 默认为 false，如果需要让它为 true，需要继承 LinkedHashMap 并且覆盖这个方法的实现，这在实现 LRU 的缓存中特别有用，通过移除最近最久未使用的节点，从而保证缓存空间足够，并且缓存的数据都是热点数据。

```
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```



#### LRU 缓存

以下是使用 LinkedHashMap 实现的一个 LRU 缓存：

- 设定最大缓存空间 MAX_ENTRIES 为 3；
- 使用 LinkedHashMap 的构造函数将 accessOrder 设置为 true，开启 LRU 顺序；
- 覆盖 removeEldestEntry() 方法实现，在节点多于 MAX_ENTRIES 就会将最近最久未使用的数据移除。

```
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private static final int MAX_ENTRIES = 3;

    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > MAX_ENTRIES;
    }

    LRUCache() {
        super(MAX_ENTRIES, 0.75f, true);
    }
}
public static void main(String[] args) {
    LRUCache<Integer, String> cache = new LRUCache<>();
    cache.put(1, "a");
    cache.put(2, "b");
    cache.put(3, "c");
    cache.get(1);
    cache.put(4, "d");
    System.out.println(cache.keySet());
}
[3, 1, 4]
```









































 