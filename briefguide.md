{{indexmenu_n>5}}

# 使用指南

以一个部署在中国，需要针对北美、亚太地域用户加速的场景为例。

1、选择线路管理，点击创建——源站所在地选择中国，加速区域选择亚太及北美，带宽数值建议按需输入。
![](/images/pathx_20180718101024.png) 购买后，线路管理列表页会显示对应的数据
![](/images/pathx_20180718101226.png)

2、选择加速管理，点击创建——源站所在地选择中国，根据线上业务填写源站ip或域名、协议及端口及加速名称，加速线路选择步骤1创建的2条线路  
![](/images/pathx_20180718102054.png) 注： 多个加速配置可以绑定同一个加速线路  

3、创建成功后，PathX会返回了一条加速域名，用此加速域名在DNS服务商中添加cname解析即完成业务加速  
业务域名：business.example.com  
源站：106.75.148.3(可填写多个，用英文半角逗号隔开)
或域名source.example.com（域名不能和要使用的业务域名冲突）  
加速线路：美国\\-\\-\\\>中国  
生成的加速域名：8cf315.pathx.ucloudgda.com  
![](/images/pathx_20180718102438.png)

\- 场景一：业务域名支持按地域智能解析
业务域名原先在中国地区有一条A记录指向106.75.148.3，保留该条记录；创建加速后，新增一条记录，地域选择美国，类型选择cname，值填写8cf315.pathx.ucloudgda.com，配置完成后，等待dns生效，这样在美国测试访问business.example.com即可解析到pathx加速域名，在中国测试依然解析到源站106.75.148.3

\- 场景二：业务域名不支持按地域智能解析
由于同一线路不支持A记录和CName记录共存，根据业务需要，如果只需要覆盖美国地区，则删除原先的A记录解析，新增CName解析，值填写生成的加速域名；如果需要同时覆盖中国和美国，那只能新增一个新的用于覆盖北美地区的域名，如business-us.example.com，添加CName解析，值填写生成的加速域名

注：可以用dig命令查看是否对加速域名解析成功。