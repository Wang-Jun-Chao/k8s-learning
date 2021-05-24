# K8S安装

安装机器ubuntu20

## 各台机器规划
```
hdss7-11.host.com 192.168.241.11
hdss7-12.host.com 192.168.241.12
hdss7-21.host.com 192.168.241.21
hdss7-22.host.com 192.168.241.22
hdss7-200.host.com 192.168.241.200
```
在不同的机器是设置主机名
```shell
set-hostname hdss7-11.host.com
set-hostname hdss7-12.host.com
set-hostname hdss7-21.host.com
set-hostname hdss7-22.host.com
set-hostname hdss7-200.host.com
```


启用root登录
https://www.cnblogs.com/zepc007/p/10765314.html

编辑文件设置静态IP
```shell
vim /etc/netplan/00-installer-config.yaml
```

文件内容信息
```yaml
# This is the network config written by 'subiquity'
network:
    ethernets:
      ens33:
        dhcp4: false
        dhcp6: false
        addresses: [192.168.241.101/24]
        optional: true
        gateway4: 192.168.241.2
        nameservers:
          addresses: [192.168.241.2, 1.1.1.1, 8.8.8.8, 114.114.114.114]
    version: 2
```


安装network-manager
```shell
apt-get update && apt-get install network-manager

netplan apply
```

安装通用软件
```shell
# 注意安装的是bind9
apt-get update && apt-get install net-tools telnet tree nmap sysstat lrzsz dos2unix bind9 bind9-utils  dnsutils
```
编辑 named.conf.options文件
```shell
~]# vim /etc/bind/named.conf.options  # 确保以下配置正确
```

文件内容
```shell
options {
listen-on port 53 { 192.168.241.11; };
directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        allow-query { any; };

        forwarders { 192.168.241.254; };

        recursion yes; // 采用递归方法查询IP

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        // dnssec-validation auto;

        dnssec-validation no;

        // listen-on-v6 { any; };
};
```
检查配置
```shell
~]# named-checkconf # 检查配置  没有信息即为正确
```

在 192.168.241.11 配置区域文件
```shell
# 增加两个zone配置，od.com为业务域，host.com.zone为主机域
~]# vim /etc/bind/named.conf.default-zones

zone "host.com" IN {
    type  master;
    file  "/etc/bind/host.com.zone";
    allow-update { 192.168.241.11; };
};

zone "od.com" IN {
    type  master;
    file  "/etc/bind/od.com.zone";
    allow-update { 192.168.241.11; };
};
```

在 hdss7-11.host.com 配置主机域文件。
```shell
# vim /etc/bind/host.com.zone

$ORIGIN host.com.
$TTL 600  ; 10 minutes # 过期时间十分钟 这里的分号是注释
@       IN SOA  dns.host.com. dnsadmin.host.com. (
        2021051601 ; serial
        10800      ; refresh (3 hours) # soa参数
        900        ; retry (15 minutes)
        604800     ; expire (1 week)
        86400      ; minimum (1 day)
        )
NS   dns.host.com.
$TTL 60 ; 1 minute
dns                A    192.168.241.11
HDSS7-11           A    192.168.241.11
HDSS7-12           A    192.168.241.12
HDSS7-21           A    192.168.241.21
HDSS7-22           A    192.168.241.22
HDSS7-200          A    192.168.241.200
```

在 hdss7-11.host.com 配置业务域文件
```shell
~]# vim /etc/bind/od.com.zone
$ORIGIN od.com.
$TTL 600  ; 10 minutes
@       IN SOA  dns.od.com. dnsadmin.od.com. (
        2021051601 ; serial
        10800      ; refresh (3 hours)
        900        ; retry (15 minutes)
        604800     ; expire (1 week)
        86400      ; minimum (1 day)
        )
NS   dns.od.com.
$TTL 60 ; 1 minute
dns                A    192.168.241.11
```

在 hdss7-11.host.com 启动bind服务，并测试
```shell
[root@hdss7-11 ~]# named-checkconf  # 检查配置文件
[root@hdss7-11 ~]# systemctl start named ; systemctl enable named
[root@hdss7-11 ~]# dig -t A hdss7-11.host.com @192.168.241.11 +short #检查是否可以解析到
192.168.241.11
[root@hdss7-11 ~]# dig -t A hdss7-12.host.com @192.168.241.11 +short #检查是否可以解析到
192.168.241.12
[root@hdss7-11 ~]# dig -t A hdss7-21.host.com @192.168.241.11 +short #检查是否可以解析到
192.168.241.21
[root@hdss7-11 ~]# dig -t A hdss7-22.host.com @192.168.241.11 +short #检查是否可以解析到
192.168.241.22
[root@hdss7-11 ~]# dig -t A hdss7-200.host.com @192.168.241.11 +short #检查是否可以解析到
192.168.241.200
```

修改DNS
```shell
root@hdss7-11:~# vim /etc/resolv.conf

# 内容
search host.com
nameserver 192.168.241.11
options edns0
```

```shell
vim /etc/sysconfig/network-scripts/ifcfg-eth0

# 内容
DNS1=192.168.241.11
```

```shell
systemctl restart network
```
