# 设计模式之外观模式
## What:
外观模式（Facade），也被称为"门面模式"。定义了一个高层、统一的接口，外部通过这个统一的接口对子系统中的一群接口进行访问。

![](https://raw.githubusercontent.com/MuggleLee/PicGo/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E5%A4%96%E8%A7%82%E6%A8%A1%E5%BC%8F/Pattern-facede.png)

## Why:
优点：
1.简化调用流程，客户端不需要知道子系统的实现，提高了安全性，符合“迪特米原则”。
2.提高灵活性，降低用户类和子系统类的耦合度，实现了松耦合。
3.更好的划分访问层次。

缺点：
1.如果新增子系统，需要修改外观类，违背了“开闭原则”。

## Where:
1.三层模型
2.为一个复杂的子系统对外提供一个简单的接口


## How:
外观（Facade）模式包含以下主要角色：

1. **外观（Facade）角色**：为多个子系统对外提供一个共同的接口。
2. **子系统（Sub System）角色**：实现系统的部分功能，客户可以通过外观角色访问它。
3. **客户（Client）角色**：通过一个外观角色访问各个子系统的功能。

![](https://raw.githubusercontent.com/MuggleLee/PicGo/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E5%A4%96%E8%A7%82%E6%A8%A1%E5%BC%8F/Pattern-Facade.png)

示例：

模拟用户下订单的流程。用户需要进入订单服务后进入支付服务，最后进入物流服务。(订单服务->支付服务->物流服务)

外观角色：对外提供服务的接口
```java
public class BuyService{

    //订单服务
    private OrderService orderService = new OrderService();

    //物流服务
    private LogisticsService logisticsService = new LogisticsService();

    //支付服务
    private PayService payService = new PayService();

    public void service() {
        orderService.service();
        payService.service();
        logisticsService.service();
    }
}
```
子系统：
```java
public class OrderService {
    public void service(){
        System.out.println("正在下订单，准备跳转到支付流程...");
    }
}
```
```java
public class PayService {
    public void service(){
        System.out.println("正在支付...支付成功！");
    }
}
```
```java
public class LogisticsService {
    public void service(){
        System.out.println("进入物流系统，准备出库...");
    }
}
```

客户端
```java
public class Client {
    public static void main(String[] args) {
        BuyService client = new BuyService();
        client.service();
    }
}
```

输出结果：
```java
正在下订单，准备跳转到支付流程...
正在支付...支付成功！
进入物流系统，准备出库...
```


# 总结

通过合理使用外观模式，可以帮助我们更好地划分访问的层次。我们可以把需要暴露给外部的功能集中到外观类中，这样既方便客户端使用，也很好地隐藏了内部的细节。





