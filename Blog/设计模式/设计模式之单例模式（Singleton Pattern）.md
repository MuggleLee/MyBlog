


<a href="#distinction">懒汉模式与饿汉模式的区别及优缺点</a>
单例模式的实现：
- <a href="#lazyAndHungry">懒汉模式和饿汉模式</a>——<a href="#debugOfMultithreading">多线程debug</a>
- <a href="#doubleLock">双重校验加锁机制</a>
- <a href="#innerClass">内部类实现单例模式（延迟加载、线程安全）</a>
- <a href="#serializationAttack">序列化对单例模式的破坏</a>
- <a href="#reflectAttack">反射对单例类的攻击</a>
- <a href="#enum">枚举</a>
- <a href="#noLock">无锁实现单例模式</a>


## What:
> 保证一个类只有一个实例，并提供全局访问点。

## Where:
1. 要求生产唯一序列号。
2. WEB 中的计数器，不用每次刷新都在数据库里加一次，用单例先缓存起来。
3. 创建的一个对象需要消耗的资源过多，比如 I/O 与数据库的连接等。

 <p id="distinction"></p>
 
#### 懒汉模式 vs 饿汉模式
懒汉模式：很懒。在调用的时候才会创建单例。(延迟加载)
饿汉模式：很饿。在系统加载的时候就会创建单例。(预加载)

||懒汉模式|饿汉模式|
|-|-|-|
|优点|第一次调用的时候才初始化，避免内存浪费|因为不用加锁就可以保证线程安全，所以执行效率高|
|缺点|必须加锁才能保证线程安全，加锁则会影响性能|类加载的时候就会初始化，造成内存浪费|
## How:

<p id="lazyAndHungry">懒汉模式和饿汉模式示例：</p>

```java
/**
 * 懒汉模式（延迟加载，非线程安全）
 */
1. public class LazySingleton {
2. 
3.     private static LazySingleton lazySingleton = null;
4. 
5.     private LazySingleton() {
6.     }
7. 
8.     public static LazySingleton getInstance() {
9.         if (lazySingleton == null) {
10.             lazySingleton = new LazySingleton();
11.         }
12.         return lazySingleton;
13.     }
14. }
```

```java
/**
 * 饿汉模式（预加载，线程安全）
 */
1. public class HungrySingleton implements Serializable {
2. 
3.     private final static HungrySingleton hungrySingleton = new HungrySingleton();
4. 
5.     private HungrySingleton() {
6.     }
7. 
8.     public static HungrySingleton getInstance() {
9.         return hungrySingleton;
10.     }
11. }
```
以上两种代码在单线程的情况下是没问题的，但懒汉模式在多线程的情况就会有可能出现问题。
 <p id="debugOfMultithreading"></p>
 
##### *在重现问题之前，首先要学会多线程debug。可以参考[ idea 多线程debug](https://blog.csdn.net/kevindai007/article/details/71412324)


先写一个简单的线程类：
```java
1. public class ThreadDemo implements Runnable{
2.     @Override
3.     public void run() {
4.         LazySingleton lazySingleton = LazySingleton.getInstance();
5.         System.out.println(Thread.currentThread().getName() + "  " + lazySingleton);
6.     }
7. }

```
```java
1. public class Main {
2.     public static void main(String[] args) {
3.         new Thread(new ThreadDemo()).start();
4.         new Thread(new ThreadDemo()).start();
5.         System.out.println(Thread.currentThread().getName() + "  " + "is Done!" );
6.     }
7. }
```
首先在类LazySingleton中的第9行设置断点。分别让Thread0和Thread1到达断点。效果如下：

![单例模式-Thread-0进入断点](http://upload-images.jianshu.io/upload_images/17362740-dbfd29d10413e9fb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![单例模式-Thread-1进入断点](http://upload-images.jianshu.io/upload_images/17362740-fc0c181b67ef4671.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![单例模式-Thread-0获取单例对象](http://upload-images.jianshu.io/upload_images/17362740-55b1c579dd5788a9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![单例模式-Thread-0输出单例对象](http://upload-images.jianshu.io/upload_images/17362740-32b55d7af6cb078a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![单例模式-Thread-1获取单例对象](http://upload-images.jianshu.io/upload_images/17362740-255164d675fdca86.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![单例模式-Thread-1输出单例对象](http://upload-images.jianshu.io/upload_images/17362740-3ac137a35f975443.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由输出结果可以看出，多线程的情况下出现单例对象不一致的情况。
如何写出一个线程安全的单例模式呢？其实很简单，使用synchronized加锁和使用volatile防止重排序。
LazySingleton类修改如下：
```java
public class LazySingleton {

    private volatile static LazySingleton lazySingleton = null;

    private LazySingleton() {
    }

    public static LazySingleton getInstance() {
        synchronized (LazySingleton.class) {
            if (lazySingleton == null) {
                lazySingleton = new LazySingleton();
            }
        }
        return lazySingleton;
    }
}
```
> synchronized关键字可以修饰方法，也可以在方法内部作为synchronized块。如果用synchronized修饰方法对于程序性能是有较大影响的，因为每次进入方法都会加锁。而在方法内部特定的逻辑使用synchronized块，灵活性较高，没有直接用synchronized修饰方法性能的损耗大。

<p id="doubleLock">修改后的单例模式是否还能优化呢？接下来介绍双重校验加锁机制。废话不多说，上代码！</p>

```java
/**
  * 双重校验加锁（延迟加载，线程安全）
  */
public class LazyDoubleCheckSingleton {

    //由于会发生重排序的情况，所以使用volatile保证创建LazyDoubleCheckSingleton实例不会发生重排序。
    private volatile static LazyDoubleCheckSingleton lazyDoubleCheckSingleton = null;

    private LazyDoubleCheckSingleton() {
    }

    public static LazyDoubleCheckSingleton getInstance() {
        if (lazyDoubleCheckSingleton == null) {
            synchronized (LazyDoubleCheckSingleton.class){
                if(lazyDoubleCheckSingleton == null){
                    lazyDoubleCheckSingleton = new LazyDoubleCheckSingleton();
                }
            }
        }
        return lazyDoubleCheckSingleton;
    }
}
```

###### 双重校验加锁机制采用双锁机制，安全且在多线程情况下能保持高性能。

<p id="innerClass">那有没有一种方法，既可以保证线程安全，又能延迟加载呢？</p>

使用内部类实现单例模式(**推荐使用**)：
```java
public class StaticInnerClassSingleton{
    private StaticInnerClassSingleton() {
    }

    private static class InnerClass {
        private static StaticInnerClassSingleton instance = new StaticInnerClassSingleton();
    }

    public static StaticInnerClassSingleton getInstance(){
        return InnerClass.instance;
    }
}

```

这种方式也是比较推荐使用的，因为实现的难度低。通过内部类实现的单例模式既可以延迟加载，不造成内存浪费，又可以不用加锁保证线程安全，提高执行效率。

<p id="serializationAttack"></p>

### 序列化对单例模式的破坏

到此为止，是否觉得写的单例模式完美无缺了呢？接下来看一下的示例代码：


```java
public class SerializableSingleton {
    private final static SerializableSingleton serializableSingleton = new SerializableSingleton();

    private SerializableSingleton() {
  }

    public static SerializableSingleton getInstance() {
        return serializableSingleton;
    }
}

```
```java
public class SerializableSingletonTest {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        SerializableSingleton instance = SerializableSingleton.getInstance();

        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("SingletonTest"));
        oos.writeObject(instance);

        File file = new File("SingletonTest");
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
	//反序列化获取SerializableSingleton对象
        SerializableSingleton newInstance = (SerializableSingleton) ois.readObject();

        System.out.println(instance);
        System.out.println(newInstance);
        System.out.println(instance == newInstance);
    }
}
```
输出结果：
```java
SingletonPattern.HungrySingleton@7f31245a
SingletonPattern.HungrySingleton@568db2f2
false
```
由输出结果可以看出，通过序列化和反序列化之后两个的对象是不一样的，这也违背了单例模式的初衷。但为什么会这样呢？我猜测是反序列化的时候执行了什么代码重新创建对象导致的。接下来通过查看源码去验证我的猜测。

反序列化是通过ObjectInputStream类的readObject方法实现的，那就直接打开readObject方法撸源码吧！
```java
public final Object readObject() throws IOException, ClassNotFoundException{

	//为省篇幅，省略部分代码
        try {
            Object obj = readObject0(false);
            //为省篇幅，省略部分代码
            return obj;
        } 
	//为省篇幅，省略部分代码
    }
```
readObject方法返回Object对象，那Object obj = readObject0(false);这一行代码返回的就是反序列化对象咯，那打开readObject0()方法查看里面有什么乾坤...
```java
    /**
     * Underlying readObject implementation.
     */
    private Object readObject0(boolean unshared) throws IOException {

        //为省篇幅，省略部分代码

        try {
            switch (tc) {

                //为省篇幅，省略部分代码

                case TC_OBJECT:
                    return checkResolve(readOrdinaryObject(unshared));

                //为省篇幅，省略部分代码
            }
        } 
	//为省篇幅，省略部分代码	
    }
```
由方法的注释就知道这个方法是反序列化的实现方法。因为我传进去的是Object对象，所以重点看return checkResolve(readOrdinaryObject(unshared))这一行。
```java
private Object readOrdinaryObject(boolean unshared) throws IOException{
        
	//为省篇幅，省略部分代码
	
        Object obj;
        try {
            obj = desc.isInstantiable() ? desc.newInstance() : null;
        } catch (Exception ex) {
            throw (IOException) new InvalidClassException(
                desc.forClass().getName(),
                "unable to create instance").initCause(ex);
        }

        //为省篇幅，省略部分代码

        if (obj != null &&
            handles.lookupException(passHandle) == null &&
            desc.hasReadResolveMethod())
        {
            Object rep = desc.invokeReadResolve(obj);
            if (unshared && rep.getClass().isArray()) {
                rep = cloneArray(rep);
            }
            if (rep != obj) {
                // Filter the replacement object
                if (rep != null) {
                    if (rep.getClass().isArray()) {
                        filterCheck(rep.getClass(), Array.getLength(rep));
                    } else {
                        filterCheck(rep.getClass(), -1);
                    }
                }
                handles.setObject(passHandle, obj = rep);
            }
        }

        return obj;
    }

```

由于readOrdinaryObject方法返回的是Object对象，所以根据源码，Object对象的声明就在obj = desc.isInstantiable() ? desc.newInstance() : null这一行，我是不是离真相越来越近了呢？接下来继续看isInstantiable源码。
```java
    /**
     * Returns true if represented class is serializable/externalizable and can
     * be instantiated by the serialization runtime--i.e., if it is
     * externalizable and defines a public no-arg constructor, or if it is
     * non-externalizable and its first non-serializable superclass defines an
     * accessible no-arg constructor.  Otherwise, returns false.
     */
    boolean isInstantiable() {
        requireInitialized();
        return (cons != null);
    }
```
根据方法的注释可知，如果一个实现了序列化的类在运行的时候被实例化，那就返回true，所以就会执行desc.newInstance()，这个方法就会通过反射调用无参的构造方法创建一个新的对象。所以问题就是在这里，通过反射返回一个新对象。那有什么办法解决呢？
继续看readOrdinaryObject方法的源码，其中obj != null && handles.lookupException(passHandle) == null && desc.hasReadResolveMethod()就是解决问题的所在。
```java
    /**
     * Returns true if represented class is serializable or externalizable and
     * defines a conformant readResolve method.  Otherwise, returns false.
     */
    boolean hasReadResolveMethod() {
        requireInitialized();
        return (readResolveMethod != null);
    }
```
根据方法注释，我可以知道如果一个类是可序列化或可反序列化的，并且定义了一个方法名为readResolve就会返回true。如果obj != null && handles.lookupException(passHandle) == null && desc.hasReadResolveMethod()判断返回true就会往下执行Object rep = desc.invokeReadResolve(obj)

invokeReadResolve方法通过反射的方式调用反序列化的类中定义的readResolve方法，并且赋值给返回的变量obj。

因此可以猜测，只要在序列化、反序列化中的类定义了readResolve方法就可以解决单例模式序列化与反序列化对象不一致的问题。

改进版SerializableSingleton类：
```java
public class SerializableSingleton implements Serializable {
    private final static SerializableSingleton serializableSingleton = new SerializableSingleton();

    private SerializableSingleton() {

    }

    public static SerializableSingleton getInstance() {
        return serializableSingleton;
    }

    private Object readResolve(){
        return serializableSingleton;
    }
}
```
```java
public class SerializableSingletonTest {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        SerializableSingleton instance = SerializableSingleton.getInstance();

        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("SingletonTest"));
        oos.writeObject(instance);

        File file = new File("SingletonTest");
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));

        SerializableSingleton newInstance = (SerializableSingleton) ois.readObject();

        System.out.println(instance);
        System.out.println(newInstance);
        System.out.println(instance == newInstance);

    }
}

```
运行结果：
```java
SingletonPattern.SerializableSingleton@7f31245a
SingletonPattern.SerializableSingleton@7f31245a
true
```
小结：
> 在设计单例模式的时候，一定要考虑类是否需要序列化，如果需要序列化则需要添加readResolve方法返回单例对象。

<p id="reflectAttack"></p>

### 反射对单例类的攻击


虽然上面解决了线程安全，序列化破坏单例模式的问题，但还有一个情况下，单例模式会被破坏，那就是通过反射攻击。

在讲解之前先看一下的示例：
```java
public class ReflectTest {

    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {

        //饿汉模式
        HungrySingleton instance = HungrySingleton.getInstance();

        Class reflectObject = HungrySingleton.class;
        //获取类的构造器
        Constructor constructor = reflectObject.getDeclaredConstructor();
        //设置在使用构造器的时候不执行权限检查
        constructor.setAccessible(true);
        //通过调用无参构造函数创建对象
        HungrySingleton newInstance = (HungrySingleton) constructor.newInstance();

        System.out.println(instance);
        System.out.println(newInstance);
        System.out.println(instance == newInstance);
    }

}

```
输出结果：
```java
SingletonPattern.HungrySingleton@4554617c
SingletonPattern.HungrySingleton@74a14482
false
```
由输出结果可以看出，通过反射可以破坏单例模式。那有什么解决办法呢？

如果是通过内部类实现单例模式或者是饿汉模式的话，在其私有构造器上添加判断就行。

譬如：
```java
public class HungrySingleton implements Serializable {

    private final static HungrySingleton hungrySingleton = new HungrySingleton();

    private HungrySingleton() {
        if(hungrySingleton != null){
            throw new RuntimeException("单例模式禁止反射调用！");
        }
    }

    public static HungrySingleton getInstance() {
        return hungrySingleton;
    }
}
```
但如果是懒汉模式，这就无法解决反射攻击单例模式了。

<p id="enum"></p>

### 枚举

事实上，还有一种更加优雅的方式设计单例模式。就是使用枚举！
用枚举实现的单例模式，更简洁，其能够自动支持序列化机制，绝对防止多次实例化，还能保证线程安全。
示例：

```java
public enum EnumSingleton {
    INSTANCE;

    private EnumSingleton() {
    }

    public static EnumSingleton getInstance() {
        return INSTANCE;
    }
}
```

```java
1. public class Test {
2.     public static void main(String[] args) throws IOException, ClassNotFoundException {
3. 
4.         EnumSingleton instance = EnumSingleton.getInstance();
5. 
6.         Class reflectClass = EnumSingleton.class;
7. 
8.         //例子1：测试序列化、反序列化是否能破坏枚举单例模式
9.         String fileName = "SingletonTest";
10.         //写文件
11.         ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(fileName));
12.         oos.writeObject(instance);
13.         File file = new File(fileName);
14.         //读文件
15.         ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
16.         EnumSingleton newInstance = (EnumSingleton) ois.readObject();
17.         System.out.println(instance == newInstance);
18. 
19.         //例子2：通过反射调用EnumSingleton无参构造器,测试是否能破坏枚举单例模式
20.         Constructor constructor = null;
21.         try {
22.             constructor = reflectClass.getDeclaredConstructor();
23.             constructor.setAccessible(true);
24.             EnumSingleton newInstance2 = (EnumSingleton) constructor.newInstance();
25.             System.out.println(instance == newInstance2);
26.         } catch (Exception e) {
27.             e.printStackTrace();
28.         }
29. 
30.         //例子3：通过反射调用EnumSingleton有参构造器,测试是否能破坏枚举单例模式
31.         Constructor constructor2 = null;
32.         try {
33.             constructor2 = reflectClass.getDeclaredConstructor(String.class, int.class);
34.             constructor2.setAccessible(true);
35.             EnumSingleton newInstance3 = (EnumSingleton) constructor2.newInstance("MuggleLee", 22);
36.             System.out.println(instance == newInstance3);
37.         } catch (Exception e) {
38.             e.printStackTrace();
39.         }
40.     }
41. }
```
输出结果：
```java
true
java.lang.NoSuchMethodException: SingletonPattern.EnumSingleton.<init>()
	at java.lang.Class.getConstructor0(Class.java:3082)
	at java.lang.Class.getDeclaredConstructor(Class.java:2178)
	at SingletonPattern.SerializableSingletonTest.main(SerializableSingletonTest.java:22)
java.lang.IllegalArgumentException: Cannot reflectively create enum objects
	at java.lang.reflect.Constructor.newInstance(Constructor.java:417)
	at SingletonPattern.SerializableSingletonTest.main(SerializableSingletonTest.java:35)
```
首先看下例子1，输出结果是true，可以证明，序列化反序列化对枚举单例没影响。

接下来看例子2，输出结果报错显示java.lang.NoSuchMethodException。这是因为，枚举类实际上是会继承抽象类Enum，而Enum类只有一个有参构造器 **protected Enum(String name, int ordinal)**，所以通过constructor.newInstance()调用无参构造器是错误的。

那例子3是调用有参构造器，为什么还会报错呢？看报错信息已经很清晰了，Cannot reflectively create enum objects，不能通过反射创建枚举对象。




通过例子可以证明，使用枚举类可以保证不会被序列化破坏，还能保证不会受到反射攻击的影响。那为什么还能保证线程安全呢？

>这和JVM类加载机制有关，static类型的属性会在类被加载之后被初始化，当一个Java类第一次被真正使用到的时候静态资源被初始化、由于虚拟机在加载枚举的类的时候，会使用ClassLoader的loadClass方法，而这个方法使用关键字synchronized同步代码块保证了线程安全，所以Java类的加载和初始化过程都是线程安全的。

<p id="noLock"></p>

### 无锁实现单例模式

以上的几种创建单例模式都是通过synchronized锁实现的，那有没有其他方法，不使用锁又能保证线程安全呢？

我们可以使用CAS实现单例模式！

```java
public class SingletonByCAS {

    private static AtomicReference<SingletonByCAS> INSTANCE = new AtomicReference();

    public SingletonByCAS() {
    }


    public static SingletonByCAS getInstance() {
        SingletonByCAS singletonByCAS = INSTANCE.get();
        for (; ; ) {
            if (singletonByCAS != null) {
                return singletonByCAS;
            }
            singletonByCAS = new SingletonByCAS();
            if (INSTANCE.compareAndSet(null, singletonByCAS)) {
                return singletonByCAS;
            }
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                SingletonByCAS singletonByCAS = SingletonByCAS.getInstance();
                System.out.println(singletonByCAS);
            }).start();
        }
    }
}

```
输出结果：
```java
SingletonPattern.SingletonByCAS@2fd04fd1
SingletonPattern.SingletonByCAS@2fd04fd1
...
...
...
SingletonPattern.SingletonByCAS@2fd04fd1
SingletonPattern.SingletonByCAS@2fd04fd1
```
通过使用AtomicXXX包装类调用compareAndSet方法可以保证线程安全，但通过这种方式实现的单例模式真的好吗？

>使用CAS的好处是不需要通过锁的方式保证线程安全，但是缺点也很显然，因为是通过自旋的方式执行，如果一直循环或者执行速度很慢的话，CPU的开销会非常大。

虽然不推荐这种方式实现单例模式，但也可以更加了解CAS的实现和运用。


# 结论：
> 设计单例模式尽量选择用枚举的方式，代码量不大而且能保证线程安全、不会遭到序列化、反射的破坏。