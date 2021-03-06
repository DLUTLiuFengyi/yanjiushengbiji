#### 标准容器方案

两个docker有不同的ip地址，不能用localhost的方式彼此访问，除非把它们封装成一个容器，或者这容器采用那容器的网络栈，但安全性会有隐患。

#### 自主式pod
定义一个pod，会先启动一个容器pause，只要有pod这个容器就要被启动。然后这个pod里面会有其他容器，其他两个容器AB会公用pause的存储卷、网络栈。

AB没有自己的独立ip地址，有的只是pod的地址，AB之间进程不隔离，就是说A的nginx可以用localhost:9000访问B的php-fpm，不需要写ip地址加映射、乱七八糟的一堆，因为共享pause的网络栈。所以AB的端口不能冲突。

pause挂载一个存储，AB共享这个存储卷。

#### 控制器管理的pod
ReplicationController在有容器异常退出时会自动创建新的pod来替代，异常多出来的容器也会自动回收。新版本使用ReplicaSet。

建议使用Deployment来自动管理ReplicaSet。滚动更新和回滚（老旧版本的RS不被删除而是停用，用来回滚）。

HPA水平pod自动扩展，通过RS监控pod当前的cpu利用率，创建pod来提高cpu利用率。

docker主要面对的是**无状态服务**（没有对应的存储需要实时去保留，apache、负载均衡调度器）
**有状态服务**（实时进行数据更新及存储，mysql、rdms、manggoDB）

#### StatefulSet解决有状态服务问题（Deployment、RS解决无状态服务）

* 稳定的持久化存储 pod重新调度后能访问到相同的持久化数据，PVC
* 稳定的网络标志 pod重新调度后podname、hostname不变，Headless Service
* 有序部署，有序扩展 下一个pod运行之前所有之前的pod都必须是running或ready，init containers
* 有序收缩，有序删除

客户端去访问一组pod（同一组标签），通过访问service的ip和端口间接访问，轮询

一个service绑定php-fpm的标签，squid写的是service的php-ip地址
一个serviec绑定squid的标签
mysql封装在一个pod
创建一个deployment，指定副本数为三，这样有三个不同的php-fpm的pod存在（到底是一个pod还是三个pod？）
再往上有三个Squid的pod