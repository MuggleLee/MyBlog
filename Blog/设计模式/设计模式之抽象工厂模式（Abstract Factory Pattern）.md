

## What:

>提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。


## Why:
#### 优点：
1.当增加一个新的产品族时不需要修改原代码，满足开闭原则。
2.可以在类的内部对产品族中相关联的多等级产品共同管理，而不必专门引入多个新的类来进行管理。
3.当一个族中的多个对象被设计成一起工作时， 它能够保证客户端始终只使用同一个族中的对象。

#### 缺点：
1.当产品等级中需要增加一个新的产品时，所有的工厂类都需要进行修改，违背开闭原则。
2.当添加产品族的时候，需要重复创建大量类，增加系统的维护难度。

## Where:
1.系统中有多于一个的族， 而每次只使用其中某一族。
2.需要创建的对象是一系列相互关联或相互依赖的产品族。

## How:

先了解什么是产品等级和产品族
>产品等级：产品等级结构即产品的继承结构
产品族：在抽象工厂模式中，产品族是指由同一个工厂生产的，位于不同产品等级结构中的一组产品

![产品族与产品等级图](https://raw.githubusercontent.com/MuggleLee/PicGo/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E6%8A%BD%E8%B1%A1%E5%B7%A5%E5%8E%82%E6%A8%A1%E5%BC%8F/%E4%BA%A7%E5%93%81%E6%97%8F%E4%B8%8E%E4%BA%A7%E5%93%81%E7%AD%89%E7%BA%A7%E5%9B%BE.png)
抽象工厂有以下几个角色：
抽象工厂（Abstract Factory）：提供了创建产品的接口，它包含多个创建产品的方法 newProduct()，可以创建多个不同等级的产品。
具体工厂（Concrete Factory）：主要是实现抽象工厂中的多个抽象方法，完成具体产品的创建。
抽象产品（Product）：定义了产品的规范，描述了产品的主要特性和功能，抽象工厂模式有多个抽象产品。
具体产品（ConcreteProduct）：实现了抽象产品角色所定义的接口，由具体工厂来创建，它 同具体工厂之间是多对一的关系。

示例：模拟家用电器制造，每种电器都有多种品牌，每个品牌也有多种电器，而客户端不需要知道创建的细节，只需要创建对应的品牌工厂就可以获取对应的电器。

![示例UML](https://raw.githubusercontent.com/MuggleLee/PicGo/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E6%8A%BD%E8%B1%A1%E5%B7%A5%E5%8E%82%E6%A8%A1%E5%BC%8F/AbstarctFactory--Sample.png)
AirConditioner、Refrigerator、WashingMachine（抽象产品）：
```java
public interface AirConditioner {
    void produce();
}
public interface Refrigerator {
    void produce();
}
public interface WashingMachine {
    void produce();
}
```
MediaAirConditioner、MediaRefrigerator、HaierAirConditioner、HaierRefrigerator（具体产品）：
```java
public class MediaAirConditioner implements AirConditioner {
    @Override
    public void produce() {
        System.out.println("制造美的空调...");
    }
}
public class MediaRefrigerator implements Refrigerator {
    @Override
    public void produce() {
        System.out.println("制造美的冰箱...");
    }
}
public class HaierAirConditioner implements AirConditioner {
    @Override
    public void produce() {
        System.out.println("制造海尔空调...");
    }
}
public class HaierRefrigerator implements Refrigerator {
    @Override
    public void produce() {
        System.out.println("制造海尔冰箱...");
    }
}
```
ElectricApplianceFactory（抽象工厂）:
```java
public interface ElectricApplianceFactory {
    AirConditioner getAirConditioner();

    Refrigerator getRefrigerator();
}
```
MediaFactory、HaierFactory（具体工厂）：
```java
public class MediaFactory implements ElectricApplianceFactory {
    @Override
    public AirConditioner getAirConditioner() {
        return new MediaAirConditioner();
    }

    @Override
    public Refrigerator getRefrigerator() {
        return new MediaRefrigerator();
    }
}
public class HaierFactory implements ElectricApplianceFactory {
    @Override
    public AirConditioner getAirConditioner() {
        return new HaierAirConditioner();
    }

    @Override
    public Refrigerator getRefrigerator() {
        return new HaierRefrigerator();
    }
}
```




Test:测试类
```java
public class Test {
    public static void main(String[] args) {
        MediaFactory mediaFactory  = new MediaFactory();
        mediaFactory.getAirConditioner().produce();
        mediaFactory.getRefrigerator().produce();

        HaierFactory haierFactory= new HaierFactory();
        haierFactory.getAirConditioner().produce();
        haierFactory.getRefrigerator().produce();
    }
}
```
输出结果：
```java
制造美的空调...
制造美的冰箱...
制造海尔空调...
制造海尔冰箱...
```


# 总结

抽象工厂模式和工厂模式实现过程类似，但有什么区别或者联系呢？
我个人认为抽象模式和工厂模式的关系是互补的。而用哪种设计模式应该根据实际的开发情况考虑。使用工厂模式是针对一个产品等级设计的，而抽象工厂模式是针对一个产品族设计的。
参考资料:
[http://c.biancheng.net/view/1351.html](http://c.biancheng.net/view/1351.html)