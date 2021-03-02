------

# LVS

丹丹姐的文档：[LVS知识整理](http://km.vivo.xyz/pages/viewpage.action?pageId=55443681)

博客：https://www.jianshu.com/p/8a61de3f8be9

## 简介：

LVS，全称Linux Virtual Server，是国人章文嵩发起的一个开源项目。
在社区具有很大的热度，是一个基于四层、具有强大性能的反向代理服务器。

## 术语：

LVS的模型中有两个角色：
调度器:Director，又称为Dispatcher，Balancer
`调度器主要用于接受用户请求。`
真实主机:Real Server，简称为RS。
`用于真正处理用户的请求。`

而为了更好地理解，我们将所在角色的IP地址分为以下三种：
Director Virtual IP:调度器用于与客户端通信的IP地址，简称为VIP
Director IP:调度器用于与RealServer通信的IP地址，简称为DIP。
Real Server : 后端主机的用于与调度器通信的IP地址，简称为RIP。

用户IP：CIP

![胡硕 > LVS与Nginx > image2019-9-26_20-52-30.png](http://km.vivo.xyz/download/attachments/107939714/image2019-9-26_20-52-30.png?version=1&modificationDate=1569502351000&api=v2)

## LVS调度模式

### LVS-NAT`Network Address Transform`


![胡硕 > LVS与Nginx > image2019-9-26_20-52-56.png](http://km.vivo.xyz/download/attachments/107939714/image2019-9-26_20-52-56.png?version=1&modificationDate=1569502377000&api=v2)

原理：

基于ip伪装`MASQUERADES`，原理是多目标DNAT。
所以请求和响应都经由Director调度器。

LVS-NAT的优点与缺点

优点：

- 支持端口映射

- RS可以使用任意操作系统

- 节省公有IP地址。

  `RIP和DIP都应该使用同一网段私有地址，而且RS的网关要指向DIP。`

  `使用nat另外一个好处就是后端的主机相对比较安全。`

缺点：

- 请求和响应报文都要经过Director转发;极高负载时，Director可能成为系统瓶颈。

### LVS-TUN`IP Tuneling`

![胡硕 > LVS与Nginx > image2019-9-26_21-10-1.png](http://km.vivo.xyz/download/attachments/107939714/image2019-9-26_21-10-1.png?version=1&modificationDate=1569503402000&api=v2)

原理：

基于隧道封装技术。在IP报文的外面再包一层IP报文。
当Director接收到请求的时候，选举出调度的RealServer
当接受到从Director而来的请求时，RealServer则会使用lo接口上的VIP直接响应CIP。（这句话是什么意思？lo接口是什么意思）
这样CIP请求VIP的资源，收到的也是VIP响应。（这句话是什么意思？）

LVS-TUN的优点与缺点

优点：

- RIP,VIP,DIP都应该使用公网地址，且RS网关不指向DIP;

  `只接受进站请求，解决了LVS-NAT时的问题，减少负载。`

  `请求报文经由Director调度，但是响应报文不需经由Director。`

缺点：

- 不指向Director，所以不支持端口映射。
- RS的OS必须支持隧道功能。
- 隧道技术会额外花费性能，增大开销。

### LVS-DR`Direct Routing`

![胡硕 > LVS与Nginx > image2019-9-26_21-10-50.png](http://km.vivo.xyz/download/attachments/107939714/image2019-9-26_21-10-50.png?version=1&modificationDate=1569503450000&api=v2)



原理

当Director接收到请求之后，通过调度方法选举出RealServer。
讲目标地址的MAC地址改为RealServer的MAC地址。
RealServer接受到转发而来的请求，发现目标地址是VIP。`RealServer配置在lo接口上。`
处理请求之后则使用lo接口上的VIP响应CIP。

LVS-DR的优点与缺点

优点：

- RIP可以使用私有地址，也可以使用公网地址。

  `只要求DIP和RIP的地址在同一个网段内`

- 请求报文经由Director调度，但是响应报文不经由Director。

- RS可以使用大多数OS

缺点：

- 不支持端口映射。
- 不能跨局域网。因为使用了MAC地址
- 

总结：

三种模型虽然各有利弊，但是由于追求性能和便捷，DR是目前用得最多的LVS模型。



## LVS的八种调度方法

### 静态方法:仅依据算法本身进行轮询调度

- RR:Round Robin,轮调
  `一个接一个，自上而下`
- WRR:Weighted RR，加权论调
  `加权，手动让能者多劳。`
- SH:SourceIP Hash（源地址Hash）
  `来自同一个IP地址的请求都将调度到同一个RealServer`
- DH:Destination Hash （内容Hash）
  `不管IP，请求特定的东西，都定义到同一个RS上。`

### 动态方法:根据算法及RS的当前负载状态进行调度

- LC:least connections(最小链接数)
  `链接最少，也就是Overhead最小就调度给谁。`
  `假如都一样，就根据配置的RS自上而下调度。`
- WLC:Weighted Least Connection (加权最小连接数)
  `这个是LVS的默认算法。`
- SED:Shortest Expection Delay(最小期望延迟)
  `WLC算法的改进。`
- NQ:Never Queue
  `SED算法的改进。`
- LBLC:Locality-Based Least-Connection,基于局部的的LC算法
  正向代理缓存机制。访问缓存服务器，调高缓存的命中率。
  和传统DH算法比较，考虑缓存服务器负载。可以看做是DH+LC
  如果有两个缓存服务器
  1.只要调度到其中的一个缓存服务器，那缓存服务器内就会记录下来。下一次访问同一个资源的时候也就是这个服务器了。 (DH)
  2.有一个用户从来没有访问过这两个缓存服务器，那就分配到负载较小的服务器。`LC`
- LBLCR:Locality-Based Least-Connection with Replication(带复制的lblc算法)
  缓存服务器中的缓存可以互相复制。因为即使没有，也能立即从另外一个服务器内复制一份，并且均衡负载

## 其他

### LVS是四层负载均衡

①所谓四层就是基于IP+端口的负载均衡；七层就是基于URL等应用层信息的负载均衡；还有基于MAC地址的二层负载均衡和基于IP地址的三层负载均衡。 换句换说，二层负载均衡会通过一个虚拟MAC地址接收请求，然后再分配到真实的MAC地址；三层负载均衡会通过一个虚拟IP地址接收请求，然后再分配到真实的IP地址；四层通过虚拟IP+端口接收请求，然后再分配到真实的服务器；七层通过虚拟的URL或主机名接收请求，然后再分配到真实的服务器。

②按照OSI模型，IP协议映射到3层网络层协议，TCP和UDP协议映射到4层传输层协议。要实现一套负载均衡系统，必须基于OSI模型4层以上。以一个例子来做说明原因：假设我们要设计一套支持HTTP，以轮询为分发策略的负载均衡系统，后端有两台Real Server。如果我们的负载均衡系统是基于3层（网络层），要发起HTTP请求，首先需要进行TCP三次握手以建立可靠的传输连接。三次握手会发出若干个数据包，由于基于3层的负载均衡器没有能力知道这些数据包是为了建立连接，只能将数据包以轮询的方式，分别发送到RS-1和RS-2。这样TCP的三次握手根本就无法成功。负载均衡系统必须建立在面对网络连接的基础上，而不是面对数据包的基础上。这套系统需要能够理解传输层网络连接，保证一次连接之内的所有数据包都转发到同一后端真实服务器上去。OSI模型4层（传输层）才能提供可靠的数据传输服务，因此它必须基于OSI模型4层之上。

# nginx

## `应用场景`

### 静态资源服务

### 反向代理服务

负载均衡

缓存

性能

### API服务



## 原理

## 2. 反向代理

### 2.1 正向代理

**Nginx** 不仅可以做反向代理，实现负载均衡，还能用做正向代理来进行上网等功能。

![img](https://pic4.zhimg.com/80/v2-b2c357e187a1259829f0d08e1de16737_1440w.jpg)

### 2.2 反向代理

客户端对代理服务器是无感知的，客户端不需要做任何配置，用户只请求反向代理服务器，反向代理服务器选择目标服务器，获取数据后再返回给客户端。反向代理服务器和目标服务器对外而言就是一个服务器，只是暴露的是代理服务器地址，而隐藏了真实服务器的IP地址。

![img](https://pic2.zhimg.com/80/v2-7a3cf7885df57a322cab8c9d8dc25cc5_1440w.jpg)



## 3. 负载均衡

将原先请求集中到单个服务器上的情况改为增加服务器的数量，然后将请求分发到各个服务器上，将负载分发到不同的服务器，即负载均衡。

![img](https://pic2.zhimg.com/80/v2-744d9c94b3fcdebceab2ae1b7c7798e9_1440w.jpg)

## 5. 高可用

为了提高系统的可用性和容错能力，可以增加nginx服务器的数量，当主服务器发生故障或宕机，备份服务器可以立即充当主服务器进行不间断工作。

![img](https://pic1.zhimg.com/80/v2-d26d65f53f88cec4d7553637ca56cb00_1440w.jpg)

## nginx配置文件

- **第一部分 全局块**
  主要设置一些影响 nginx 服务器整体运行的配置指令。
  比如： worker_processes 1; ， worker_processes 值越大，可以支持的并发处理量就越多。

- **第二部分 events块**
  events 块涉及的指令主要影响Nginx服务器与用户的网络连接。
  比如： worker_connections 1024; ，支持的最大连接数。

- **第三部分 http块**
  http 块又包括 http 全局块和 server 块，是服务器配置中最频繁的部分，包括配置代理、缓存、日志定义等绝大多数功能。

- - **server块**：配置虚拟主机的相关参数。
  - **location块**：配置请求路由，以及各种页面的处理情况。



```bash
########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
}
```



https://zhuanlan.zhihu.com/p/285381634





正向代理配置

```java
server {
     listen  443;
    
     # dns resolver used by forward proxying
     resolver  114.114.114.114;

     # forward proxy for CONNECT request
     proxy_connect;
     proxy_connect_allow            443;
     proxy_connect_connect_timeout  10s;
     proxy_connect_read_timeout     10s;
     proxy_connect_send_timeout     10s;

     # forward proxy for non-CONNECT request
     location / {
         proxy_pass http://$host;
         proxy_set_header Host $host;
     }
 }
```



访问方式 访问+解析或者加代理