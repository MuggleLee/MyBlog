搭建Redis集群的过程中，执行到cluster create <ip>:<port> ... 的时候，发现程序在阻塞，显示：Waiting for the cluster to join 的字样，然后就无休无尽的等待...

![没有开放集群总线端口](https://raw.githubusercontent.com/MuggleLee/PicGo/master/Redis图/集群/没有开放集群总线端口.jpg)


根据字样的提示，在等待集群的创建。嗯？这是什么原因？大部分情况下这是因为集群通信的端口没有开放！

先说下解决方案：
开放Redis服务的两个TCP端口。譬如Redis客户端连接端口为6379，而Redis服务在集群中还有一个叫集群总线端口，其端口为客户端连接端口加上10000，即 6379 + 10000 = 16379。所以开放每个集群节点的客户端端口和集群总线端口才能成功创建集群！

![成功搭建集群](https://raw.githubusercontent.com/MuggleLee/PicGo/master/Redis%E5%9B%BE/%E9%9B%86%E7%BE%A4/%E6%88%90%E5%8A%9F%E6%90%AD%E5%BB%BA%E9%9B%86%E7%BE%A4.jpg)

问题解决了，则反思一下，客户端端口和集群总线端口有什么区别呢？

>客户端端口：客户端访问Redis服务器的端口
集群总线端口：用二进制协议的点对点集群通信的端口。用于节点的失败侦测、配置更新、故障转移授权，等等。

总而言之，客户端端口提供的是外部客户端访问服务的端口；而集群总线端口是提供集群内部各个Redis服务之间的通信。




参考资料：
[https://www.cnblogs.com/cjsblog/p/9048545.html](https://www.cnblogs.com/cjsblog/p/9048545.html)