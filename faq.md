# FAQ



## 加速配置和加速线路的关系

1、带宽共享功能：一个加速线路可以被多个加速配置绑定，这些加速配置共享加速线路的带宽；  
2、一个加速配置可以绑定多个加速线路。  
3、删除加速配置不会影响加速线路，加速线路仍存在；  
4、若加速线路上绑定了加速配置，该加速线路不能删除；若加速线路上无任何加速配置，该加速线路可以删除。  

## 为什么看到大量固定的IP地址在访问源站服务器？

这些地址是加速集群出口EIP，起到两个作用：  
1、传输业务流量（可通过toa模块获取访问者真实IP）  
2、对源站服务器做健康检查  
对源站服务器做健康检查是通过TCP三次握手的原理来探测源站可用性，为短连接。

注意：源站相关的云服务商或IDC安全策略，可能对短时间内大量的tcp短连接比较敏感，例如友商的安骑士、云盾会在您不知情的情况下误杀这类IP。安全起见，建议在加速线路详情页获得出口EIP列表后 提前加入到云盾白名单内（包括友商的CDN厂商信任列表，WAF产品白名单）。

## 如何获取访问者真实IP？

由于经过加速，在日志中看到的访问者IP全部变为PathX的出口IP。 如果需要获取真实的客户端IP,
可以在您的源站服务器上加载UCloud专有的内核模块，让应用直接获取到源IP，这时候，再去查看日志，就是访问者的真实IP了。  
**Linux系统**  
64位的linux系统可运行"modprobe toa"尝试加载模块，成功后无需其他操作。  
如提示未找到该模块，可按如下步骤进行手工编译与加载：

1.查看当前内核版本号，确认依赖"kernel -devel、kernel -headers"是否安装以及版本号是否与内核一致('uname
-r && rpm -qa |egrep 'kernel -devel|kernel -headers')：  
若一致，跳过步骤2，进行toa模块的编译安装  
若不一致，如下图：  
![](/images/toa_201810301429.png) 需要卸载后进行步骤2操作(rpm -e
--nodeps kernel-devel kernel-headers)  
若未安装依赖，如下图： ![](/images/toa_201810301432.png)
  
2.yum搜索是否有与当前内核版本对应的kernel-devel、kernel-headers
若有，则安装对用版本（yum install pkgname\-version\.x86\_64）  
若无，如下图  
![](/images/toa_201810301443.png)  
则打开网站 http://rpm.pbone.net 点击左侧SEARCH标签，填入包名+版本号（如：kernel-devel-3.10.0-693.11.6.el7.x86_64），选择对应的系统发行版本（此处为CentOS7），点击搜索  
![](/images/toa_201810301447.png) 搜索结果：
![](/images/toa_201810301449.png) 或使用谷歌用关键字“rpm.pbone.net
kernel-devel-3.10.0-693.11.6.el7.x86_64”搜索
![](/images/toa_201810301450.png) 下载后rpm方式安装，kernel-headers的安装同理
![](/images/toa_201810301452.png) 确认安装结果（'uname -r && rpm -qa
|egrep 'kernel-devel|kernel-headers'），如下图：
![](/images/toa_201810301453.png)

3.下载linux通用版的源码包，该版本支持Centos 6.9和Centos 7、ubuntu
14.04等绝大多数的linux发行版：  

国内：  
```wget http://pathx.ufile.ucloud.com.cn/linux_toa.tar.gz```

国外：  
```wget http://toa.ufile.ucloud.com.cn/linux_toa.tar.gz```

  
4.编译加载  
```
yum install -y gcc
tar -zxvf linux_toa.tar.gz
cd linux_toa
make
mv toa.ko /lib/modules/`uname -r`/kernel/net/netfilter/ipvs/toa.ko
insmod /lib/modules/`uname -r`/kernel/net/netfilter/ipvs/toa.ko

```

toa模块安装验证如下（lsmod |grep toa）：

![](/images/toa_201810301534.png)

  
5.添加开机模块自动加载  

```

echo "insmod /lib/modules/`uname -r`/kernel/net/netfilter/ipvs/toa.ko"
>> /etc/rc.local

```

**nginx 环境下**，直接在nginx 日志中查看真实访问者地址 日志路径： /var/log/nginx/access.log

![](/images/nginx_真实地址.png)

**apache环境下**，直接在apache日志中查看真实访问者地址  

日志路径：/etc/httpd/logs/access_log 

![](/images/apache获取真实地址.png)

  - 其他web配置环境， 采用同样方法在相关web 日志文件中检查即可  

## 安装了toa 仍然无法查看真实客户端IP

toa原理是从tcp包中取出option字段，解析出真实客户端IP，最后通过内核钩子函数完成替换。如果转发过程中出现了tcp连接截断的情况，如在rs前使用七层负载均衡产品，就会导致安装toa 也获取不到真实客户端IP：

1）client -------> pathx 4层转发 --------- tcp packet (option字段包含：客户端IP ) --------> 7层负载均衡 ----------tcp packet (option字段不再包含 客户端IP) --------> 源站RS（安装toa 不能获取客户端IP）

2）client -------> pathx 4层转发 --------- tcp packet (option字段包含：客户端IP ) --------> 源站RS（安装toa 可以获取客户端IP）

3）client--------> pathx 4层转发 --------- tcp packet (option字段包含：客户端IP ) --------> 4层负载均衡（ulb4开启报文转发 或 AWS NLB开启保留源IP） ----------tcp packet (option字段包含 客户端IP) --------> 源站RS（安装toa 可以获取客户端IP）


如果使用http协议场景，七层转发 不需要安装toa模块就可以获取真实客户端IP：

4）client--------> pathx 7层转发 ------X-Forwarded-For---------> 各类LB --------> 源站RS（从http header获取客户端IP）

5）client--------> pathx 7层转发 ------X-Forwarded-For---------> 源站RS（从http header获取客户端IP）

## 如何查看PathX的出口IP？

请在console线路资源详情页的基本信息中查看

## 非UCloud服务器是否可以使用全球动态加速？

可以，只要是公网路由可达的服务器即可。

## 网站是否需要备案？

1）若加速区域在中国大陆地区，当您的业务域名CName到加速域名后，需要在中华人民共和国工业和信息化部完成相关备案；客户端直接使用加速域名的情况是不需要备案的。
2）加速区域在中国大陆地区之外，不需要备案。

## 全球动态加速和CDN加速有区别吗？

CDN在边缘节点对资源缓存实现访问提速，缓存的对象为静态媒体资源。全链路在公网上，跨国回源的线路不太稳定。国外领先的CDN厂商对回源网络做了深度优化，受政策影响，在某些地区节点数量有限，需要其他产品辅助。

全球应用加速优化的是从客户端到源站的跨国(洲)网络质量，依托UCloud的专线调度能力，控制丢包和延迟，不支持应用数据缓存，每次请求都会访问源站获取资源数据。适合支付、登陆、聊天、长连接等场景

## HTTP(s)网站或API场景是否可以使用？

1）可以使用，全球动态加速支持tcp透传回源。在控制台配置 TCP 80或443端口，证书仍然部署在您的业务服务器上，不需要其他设置。
2）如果您的业务场景需要就近Offloading SSL, 用HTTP协议回源，可以使用PathX 7层端口转发中HTTPS-HTTP方式。
3）如果您使用的HTTP(s)请求，源站不方便安装TOA模块 又想获得真实客户端IP，可以使用PathX  HTTPS-HTTPS或HTTP-HTTP转发方式。

## 什么是多地接入？

以中国(多地)到香港的加速为例，创建加速后，华北、华东、华南的用户访问同一个加速域名，但解析出的IP不同，这些IP分别对应pathx在三个地区的入口，也即，不同区域的用户访问pathx加速域名会解析出离用户最近的加速入口IP。

## 资源使用一段时间后，PathX或GlobalSSH的加速域名+端口突然无法正常访问，而源站+端口可以正常访问

用curl测试一般会报"curl: (56) Failure when receiving data from the peer"
用telnet测试 提示连接成功后，立即收到"Reset By Remote Peer"
重点！重点！重点！用nc测试，则提示连接成功建立。

1. 检查源站是否有安全策略设置，如阿里云的安骑士（云盾会在用户不做任何配置情况下自动封堵，需要开启白名单IP保护 CDN信任厂商），fail2ban等，若有，可以从console资源详情页获取pathx或globalssh 出口ip加入白名单。
2. 以上不能解决您的问题，请先提工单咨询您的源站服务器供应商，是否有自动封堵不明来源IP的安全策略。

2. 检查系统参数设置

    net.ipv4.tcp_timestamps = 1
    net.ipv4.tcp_tw_recycle = 0
    net.ipv4.tcp_tw_reuse = 1

开启tcp_tw_resuse足够进行TCP连接的回收，tcp_tw_recycle由于设计的时间较为早期，并没有考虑NAT技术在如今公网已经普及，会导致经过NAT（诸如网吧、4G、WIFI）用户部分的连接失败。当前这个参数已经基本废弃。提升tcp回收速度，也可以通过减小tcp等待timeout的时间来实现。

## 源站是否可以修改？

源站不支持修改，可以新建一个加速配置，添加新的源站IP或域名，获得新的加速域名。在DNS解析服务商处，修改您的业务域名cname到新的加速域名，删除旧的加速配置。

## pathX欠费回收后是否可以找回？

回收后无法找回。

## pathX是否可配置带宽告警？

可在加速线路详情页"告警模板"处配置

## pathX的限流措施

pathX按购买带宽限流，当实时带宽（出向带宽与入向带宽的最大值）超过购买带宽时会触发限流，限流会引起丢包率上升，连接中断等现象发生，当实时带宽低于购买带宽时限流会自动解除。请在线路详情页配置带宽使用告警。

## 海外SD-WAN是否会支持从海外访问中国大陆地区或从中国大陆地区访问海外的线路？

目前仅支持加速区域和源站均在海外的线路。

