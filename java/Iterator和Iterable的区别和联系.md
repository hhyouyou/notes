## Iterator是什么

```java
package java.util;

import java.util.function.Consumer;

public interface Iterator<E> {
    boolean hasNext();
    E next();
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
}
```

`Iterator`是util包下的一个接口,是个迭代器类

**Iterator模式是用于遍历集合类的标准访问方法。它可以把访问逻辑从不同类型的集合种抽取出来，从而避免向使用者暴露集合内部结构。**

使用者只需要获取这个集合对象的迭代器，使用这个迭代器来对这组集合进行操作，而不需要知道内部实现。

ps：`Array`可能返回`ArrayIterator`，`Set`可能返回 `SetIterator`，`Tree`可能返回`TreeIterator`，但是它们都实现了`Iterator`接口。



## Iterable又是什么？

```java
package java.lang;

import java.util.Iterator;
import java.util.Objects;
import java.util.Spliterator;
import java.util.Spliterators;
import java.util.function.Consumer;

public interface Iterable<T> {

    Iterator<T> iterator();

    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
    
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```



所有集合类都实现了 `Collection `接口，而 `Collection `继承了 `Iterable` 接口。

可以对同一个集合获取多个迭代器。每个实例化迭代器都有新的位置，互不干扰。







## 参考文档

https://blog.csdn.net/zq602316498/article/details/39337899

https://zhuanlan.zhihu.com/p/52366312