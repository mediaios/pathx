{{indexmenu_n>4}}

# 名词解释

## 源站IP，源站域名

即需要加速的IP或者域名，二者选一即可，为了保证业务的高可用，建议填写域名。

## 业务所在地

源站服务器所在的地区，如源站在洛杉矶，业务所在地则为美国。

## 加速区域

业务需要覆盖的地区（与源站所在地距离较远），如源站在美国，需要覆盖中国用户，则加速区域为中国。

## cname域名

全球动态加速提供的就近接入域名。您需要通过DNS智能解析，将源站域名的cname记录修改为此域名，即可实现各地用户的就近接入。

## 加速线路

加速线路指从待加速区到业务所在地之间的加速。  
若源站所在地为中国，待加速区为美国。那么加速线路是 美国到中国访问的加速。

## 加速配置

配置待加速的源站服务器（IP/域名），业务端口，指定业务所在地，选择加速线路