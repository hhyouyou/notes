[TOC]



## 第一章   工厂模式



### 简单工厂

一个工厂类，创建多个不同产品



### 工厂方法模式

一个产品对应一个工厂



定义一个创建对象的接口,让子类来决定实例化哪个类,

工厂方法是一个类的实例化延迟到其子类

![工厂方法](http://image-djx.test.upcdn.net/md/notes/%E5%B7%A5%E5%8E%82%E6%96%B9%E6%B3%95.png)

```java
public interface Reader {// 抽象的reader
    void reader();
}
// 一个具体的reader, 还可以有JpgReader,PngReader
public class GifReader implements Reader {
    @Override
    public void reader() {
        System.out.println(Thread.currentThread().getStackTrace()[0].getClassName());
        System.out.println(Thread.currentThread().getStackTrace()[1].getClassName());
    }
}
// 抽象工厂
public interface ReaderFactory {
    Reader getReader();
}
// 具体的工厂产出对应的产品
public class GifReaderFactory implements ReaderFactory {
    @Override
    public Reader getReader() {
        return new GifReader();
    }
}
public class Test {
    public static void main(String[] args) throws Exception {
        ReaderFactory readerFactory = new GifReaderFactory();
        Reader reader = readerFactory.getReader();
        reader.reader();
    }
}
```

工厂方法+反射

多个产品对应多个工厂,还可以继续做简化

```java
// 抽象工厂
public abstract class AbstractReaderFactory<T> {
    abstract Reader getReader(Class<T> c) throws Exception;
}

// 具体的工厂产出对应的产品
public class AllReaderFactory extends AbstractReaderFactory {
    @Override
    Reader getReader(Class c) throws Exception {
        return (Reader) Class.forName(c.getName()).newInstance();
    }
}
public class Test {
    public static void main(String[] args) throws Exception {
        AllReaderFactory allReaderFactory = new AllReaderFactory();
        Reader reader1 = allReaderFactory.getReader(GifReader.class);
        reader1.reader();

        Reader reader2 = allReaderFactory.getReader(JpgReader.class);
        reader2.reader();
    }
}
```

#### 抽象工厂

一个工厂对应一个产品族







## 第二章   策略模式

封装算法，把算法从业务中抽象出来。

### 类图

* strategy（InStockStrategy）接口定义一个算法接口， 让其实现类都实现具体的算法内容（inStock）。
* InStockOrder 是具体使用到算法的类，其中的`inStock()`会调用策略中的实际`inStock()`。
* 实际的业务（InStockService）中就可以根据, 入库类型来选择不同策略



![策略模式](http://image-djx.test.upcdn.net/md/notes/%E7%AD%96%E7%95%A5%E6%A8%A1%E5%BC%8F.png)



### 实现

```java
public interface InStockStrategy {
    /**
     *  入库
     * @param inStockNumber 入库单号
     * @param quantity 数量
     */
    void inStock(String inStockNumber, Integer quantity);
}

/**
 * 采购入库
 **/
public class PurchaseInStock implements InStockStrategy {

    @Override
    public void inStock(String inStockNumber, Integer quantity) {
        System.out.println(this.getClass().getName());
    }
}

/**
 * 退货入库
 **/
public class ReturnInStock implements InStockStrategy {
    
    @Override
    public void inStock(String inStockNumber, Integer quantity) {
        System.out.println(this.getClass().getName());
    }
}
```



```java
public class InStockOrder {

    private String stockNumber;
    private Integer number;

    private InStockStrategy inStockStrategy;

    public void inStock() {
        if (Objects.nonNull(inStockStrategy)) {
            inStockStrategy.inStock(stockNumber, number);
        }
    }
    
    public void setInStockStrategy(InStockStrategy inStockStrategy) {
        this.inStockStrategy = inStockStrategy;
    }
}
```



```java
public class InStockService {

    public static void main(String[] args) {
        // 选取策略
        PurchaseInStock purchaseInStock = new PurchaseInStock();

        InStockOrder inStockOrder = new InStockOrder();
        inStockOrder.setInStockStrategy(purchaseInStock);

        // 入库
        inStockOrder.inStock();
    }
}
```





## 第三章   单一指责原则



## 第四章   开放-封闭原则



## 第五章   依赖倒转原则



## 第六章   装饰模式



* 装饰模式是用来：**动态的给对象加上额外的职责**



#### 使用到的地方

1.  Spring中`BeanDefinitionDecorator`: 
2.  commons-collections包中`ListUtils`， 将list装饰成不同功能的list

```java
public static List synchronizedList(List list) {
    return SynchronizedList.decorate(list);
}

public static List unmodifiableList(List list) {
    return UnmodifiableList.decorate(list);
}

public static List predicatedList(List list, Predicate predicate) {
    return PredicatedList.decorate(list, predicate);
}

public static List typedList(List list, Class type) {
    return TypedList.decorate(list, type);
}

public static List transformedList(List list, Transformer transformer) {
    return TransformedList.decorate(list, transformer);
}

public static List lazyList(List list, Factory factory) {
    return LazyList.decorate(list, factory);
}
public static List fixedSizeList(List list) {
    return FixedSizeList.decorate(list);
}
```

3. `scheduledThreadPool `装饰任务



#### 装饰模式和继承的区别

* 装饰者模式强调的是**动态**的扩展, 而继承关系是**静态的**.
* **组合优于继承**



#### 装饰和建造者

* Builder模式是一种**创建型**的设计模式. **旨在解决对象的差异化构建的问题**.
* 装饰者模式是一种**结构型**的设计模式. **旨在处理对象和类的组合关系**.



> Builder模式的**差异化构建**是**可预见**的, 而装饰者模式实际上提供了一种不可预见的扩展组合关系.



对象，ProductInfoVo， 本来只有商品的 id、 name

但是我还需要他的 库存信息？ 加上 stock信息

还需要供应信息？加上供应商信息

还需要操作人信息？加上操作人信息

> 基础对象，Product 接口
>
> Product getProductInfo();
>
> Product getProductSimpleInfo();
>
> Product getProductAndUserInfo();
>
> Product getProdctMoreInfo()

根据不同需求，给商品这个接口，装载上不同的信息，防止信息



#### 使用场景

* 在不改变原有类结构基础上，新增或者限制或者改造功能时候。





## 第七章 代理模式



**代理模式**（proxy）， 为其他对象提供一种代理以控制对这个对象的访问。



 #### 应用场景

1. 远程代理 ：为一个对象在不同的地址空间提供局部代表。这样可以隐藏一个对象存在于不同地址空间的事实
2. 虚拟代理 ：根据需要创建开销很大的对象。通过它来存放实例化需要很长时间的真实对象
3. 安全代理 ：安全代理，用来控制真实对象访问时的权限
4. 智能指引 ： 是指当调用真实对象时，代理处理另外一些事



#### 静态代理

讲代理比较经典的就是，Java中访问数据库，使用代理service进行，事务的开启关闭回滚

静态代理，就是新建一个代理类，在执行目标类方法前后进行其他处理。

```java
public interface UserService {
    void addUser();
    void updateUser(String username);
    void deleteUser(String username);
}

public class UserServiceImpl implements UserService {
    @Override
    public void addUser() {
        System.out.println("新增一个用户");
    }
    @Override
    public void updateUser(String username) {
        System.out.println("更新用户：" + username + " 信息");
    }
    @Override
    public void deleteUser(String username) {
        System.out.println("删除用户：" + username);
    }
}

public class UserServiceTransactionImpl implements UserService {
    private final UserServiceImpl userServiceImpl;
    private final TransactionalImpl transactional = new TransactionalImpl();
    public UserServiceTransactionImpl(UserServiceImpl userServiceImpl) {
        this.userServiceImpl = userServiceImpl;
    }
    @Override
    public void addUser() {
        transactional.open();
        try {
            userServiceImpl.addUser();
            transactional.commit();
        } catch (Exception e) {
            transactional.rollback();
        }
    }
    @Override
    public void updateUser(String username) {
        transactional.open();
        try {
            userServiceImpl.updateUser(username);
            transactional.commit();
        } catch (Exception e) {
            transactional.rollback();
        }
    }

    @Override
    public void deleteUser(String username) {
        transactional.open();
        try {
            userServiceImpl.deleteUser(username);
            transactional.commit();
        } catch (Exception e) {
            transactional.rollback();
        }

    }
public class Test {
    public static void main(String[] args) {
        UserServiceImpl userService = new UserServiceImpl();
        UserServiceTransactionImpl userServiceProxy = new UserServiceTransactionImpl(userService);
        userServiceProxy.addUser();
        userServiceProxy.updateUser("user1");
        userServiceProxy.deleteUser("user1");
    }
    
}
```



#### 动态代理

动态的新建一个类，然后再目标类方法前后进行其他处理。

1. 基于`cglib`
2. 基于`jdk`

```java
public class CglibDemo {

    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        TransactionalImpl transactional = new TransactionalImpl();
        enhancer.setSuperclass(UserServiceImpl.class);
        MethodInterceptor methodInterceptor = (o, method, objects, methodProxy) -> {
            System.out.println("原方法名是 ： " + method.getName());
            System.out.println("原方法声明的类为 " + method.getDeclaringClass());

            // TODO如果注解中有 @Transactional 的就进行，干嘛干嘛。。。。。
            Annotation[] annotations = method.getAnnotations();
            transactional.open();
            Object o1 = methodProxy.invokeSuper(o, objects);
            transactional.commit();
            return o1;
        };
        enhancer.setCallback(methodInterceptor);
        UserServiceImpl userService = (UserServiceImpl) enhancer.create();
        userService.addUser();
        userService.updateUser("user1");
        userService.deleteUser("user1");
    }
}

public class JDKProxyDemo {
    public static void main(String[] args) {
        TransactionalImpl transactional = new TransactionalImpl();
        UserServiceImpl userService = new UserServiceImpl();
        UserService userServiceProxy = (UserService) Proxy.newProxyInstance(UserService.class.getClassLoader(), new Class[]{UserService.class}, (proxy, method, args1) -> {
            // 注解中有 @Transactional 的进行开启
            Annotation[] annotations = method.getAnnotations();
            transactional.open();
            Object result = method.invoke(userService, args1);
            transactional.commit();
            return result;
        });
        userServiceProxy.addUser();
        userServiceProxy.updateUser("user1");
        userServiceProxy.deleteUser("user1");
    }
}
```





## 第八章 适配器模式



> 对已有的方法进行兼容
>
> 在方法外再封装一层，让新的接口，可以复用老的代码



在spring中的使用， 实现AdvisorAdapter, 的三个适配器





## 第九章 备忘录模式

参考https://juejin.im/post/5bd12141f265da0ad82c4f71





**Originator（原发器）**：它是一个普通类，可以创建一个备忘录，并存储它的当前内部状态，也可以使用备忘录来恢复其内部状态，一般将需要保存内部状态的类设计为原发器。

**Memento（备忘录)**：存储原发器的内部状态，根据原发器来决定保存哪些内部状态。备忘录的设计一般可以参考原发器的设计，根据实际需要确定备忘录类中的属性。需要注意的是，除了原发器本身与负责人类之外，备忘录对象不能直接供其他类使用，原发器的设计在不同的编程语言中实现机制会有所不同。

**Caretaker（负责人）**：负责人又称为管理者，它负责保存备忘录，但是不能对备忘录的内容进行操作或检查。在负责人类中可以存储一个或多个备忘录对象，它只负责存储对象，而不能修改对象，也无须知道对象的实现细节。

备忘录模式的核心是备忘录类以及用于管理备忘录的负责人类的设计。





### 第十章 备忘录模式总结

备忘录模式的**主要优点**如下：

- 它提供了一种状态恢复的实现机制，使得用户可以方便地回到一个特定的历史步骤，当新的状态无效或者存在问题时，可以使用暂时存储起来的备忘录将状态复原。
- 备忘录实现了对信息的封装，一个备忘录对象是一种原发器对象状态的表示，不会被其他代码所改动。备忘录保存了原发器的状态，采用列表、堆栈等集合来存储备忘录对象可以实现多次撤销操作。

备忘录模式的**主要缺点**如下：

- 资源消耗过大，如果需要保存的原发器类的成员变量太多，就不可避免需要占用大量的存储空间，每保存一次对象的状态都需要消耗一定的系统资源。

**适用场景**：

- 保存一个对象在某一个时刻的全部状态或部分状态，这样以后需要时它能够恢复到先前的状态，实现撤销操作。
- 防止外界对象破坏一个对象历史状态的封装性，避免将对象历史状态的实现细节暴露给外界对象。





可以用在前端？？？

我觉得前端更合适一点，后端更适合数据的持久化，应该说是后台

前端页面上，可以对某些东西，编辑然后撤销，最后决定好了，在持久化到后台就可以了

后端没啥用







## 第十一章 组合模式





## 第十二章 迭代器模式



参考: https://juejin.im/post/5bbf67e86fb9a05d3447e054





### 迭代器模式

**迭代器模式(Iterator Pattern)**：提供一种方法来访问聚合对象，而不用暴露这个对象的内部表示，其别名为游标(Cursor)。迭代器模式是一种对象行为型模式



#### 角色

**Iterator（抽象迭代器）**：它定义了访问和遍历元素的接口，声明了用于遍历数据元素的方法，例如：用于获取第一个元素的first()方法，用于访问下一个元素的next()方法，用于判断是否还有下一个元素的hasNext()方法，用于获取当前元素的currentItem()方法等，在具体迭代器中将实现这些方法。

**ConcreteIterator（具体迭代器）**：它实现了抽象迭代器接口，完成对聚合对象的遍历，同时在具体迭代器中通过游标来记录在聚合对象中所处的当前位置，在具体实现时，游标通常是一个表示位置的非负整数。

**Aggregate（抽象聚合类）**：它用于存储和管理元素对象，声明一个createIterator()方法用于创建一个迭代器对象，充当抽象迭代器工厂角色。

**ConcreteAggregate（具体聚合类）**：它实现了在抽象聚合类中声明的createIterator()方法，该方法返回一个与该具体聚合类对应的具体迭代器ConcreteIterator实例。

在迭代器模式中，提供了一个外部的迭代器来对聚合对象进行访问和遍历，迭代器定义了一个访问该聚合元素的接口，并且可以跟踪当前遍历的元素，了解哪些元素已经遍历过而哪些没有。迭代器的引入，将使得对一个复杂聚合对象的操作变得简单。

在迭代器模式中应用了工厂方法模式，抽象迭代器对应于抽象产品角色，具体迭代器对应于具体产品角色，抽象聚合类对应于抽象工厂角色，具体聚合类对应于具体工厂角色。



## 第十三章 中介者模式

[http://laijianfeng.org/2018/10/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F-%E4%B8%AD%E4%BB%8B%E8%80%85%E6%A8%A1%E5%BC%8F%E5%8F%8A%E5%85%B8%E5%9E%8B%E5%BA%94%E7%94%A8/](http://laijianfeng.org/2018/10/设计模式-中介者模式及典型应用/)





同一层级，多个服务互相调用，造成混乱。

由一个中间人，统一调度



java.util.Timer



## 参考文章

1. [美团技术团队-设计模式在外卖营销业务中的实践](https://tech.meituan.com/2020/03/19/design-pattern-practice-in-marketing.html)

