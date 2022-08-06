 # LTP网络测试
**注意事项**
1. 运行测试前，请检查系统是否有运行业务服务或者其他性能工具测试
2. 测试开始后，尽量不要进行操作，如有必要请确认操作是否会严重占用系统资源

| 序号 | 工具 | 版本 | 说明 |
| --- | --- | --- | --- |
| 1 | Ltp | 20220121 | 测试Linux系统可靠性、健壮性和稳定性 |

## 前置条件
***********************************************
* 准备两台服务器（client和server），具体网络拓扑如下：   
![图片](https://user-images.githubusercontent.com/110716073/183240302-03dd0428-a21f-41bc-8e35-aadfa130e194.png)  

client:  
Test Link ip:1.1.1.1
Control Link ip:2.1.1.1
server:  
Test Link ip:1.1.1.2
Control Link ip:2.1.1.2

Test Link和Control Link是不同网段IP

***********************************************

* 需要在client和server中安装libmnl库  
`yum –y install  libmnl  libmnl-devel  libmnl-static	libnftnl`

* 需要在ltp-20220121源码/ltp-20220121/testcases/network/stress/route/route-change-netlink.c中的291行（send前）添加usleep(1000）

* 需要在server中的~/.bashrc文件最后添加一行命令（否则运行network.sh脚本时出现“bash:: 未找到命令”，客户端ssh远程调用命令失败）
`export PATH=$PATH:/opt/ltp/testcases/bin`

## LTP源码编译
下载地址：https://github.com/linux-test-project/ltp/tags?after=20190517
```
unzip ltp-20220121
cd ltp-20220121
make autotools
./configure
make && make install
```

安装目录（默认/opt/ltp）

Usage: ./network.sh OPTIONS
| 选项 | 选项说明 |
| --- | --- |
| -6 | IPv6 tests | 
| -m | multicast tests |
| -n | NFS tests |
| -r | RPC tests |
| -s | SCTP tests |
| -t | TCP/IP command tests |
| -c | TI-RPC tests |
| -d | TS-RPC tests |
| -a | Application stress tests (HTTP, SSH, DNS) |
| -e | Interface stress tests |
| -b | Stress tests with malformed ICMP packets |
| -i | IPsec ICMP stress tests |
| -T | IPsec TCP stress tests |
| -U | IPsec UDP stress tests |
| -D | IPsec DCCP stress tests |
| -S | IPsec SCTP stress tests |
| -R | route stress tests |
| -M | multicast stress tests |
| -F | network features tests (TFO, vxlan, etc.) |
| -fx | where x is a runtest file |
| -q | quiet mode (this implies not logging start of test in kernel log) |
| -Q | don't log start of test in kernel log |
| -V|v | verbose |
| -h | print this help |

## LTP配置
编译完成后需要对LTP进行相关配置

* 在/opt/ltp/testcases/bin/route-change-gw.sh文件中的31行添加sleep 1
```
     23 test_gw()
     24 {
     25         local gw="$(tst_ipaddr_un -h 2,254 1 $(($1 + 1)))"
     26         local iface="$(tst_iface)"
     27 
     28         tst_res TINFO "testing route over gateway '$gw'"
     29 
     30         tst_add_ipaddr -s -q -a $gw rhost
     31         sleep 1
     32         ROD ip route add $rt dev $iface via $gw
     33         EXPECT_PASS_BRK ping$TST_IPV6 -c1 -I $lhost $rhost \>/dev/null
     34         ROD ip route del $rt dev $iface via $gw
     35         tst_del_ipaddr -s -q -a $gw rhost
     36 }
```

* 在/opt/ltp/testcases/bin/route-change-dst.sh文件中的31行和33行分别添加sleep 1
```
     22 test_dst()
     23 {
     24         local iface="$(tst_iface)"
     25         local rt="$(tst_ipaddr_un -p $1)"
     26         local rhost="$(tst_ipaddr_un $1 1)"
     27 
     28         tst_res TINFO "testing route '$rt'"
     29 
     30         tst_add_ipaddr -s -q -a $rhost rhost
     31         sleep 1
     32         ROD ip route add $rt dev $iface
     33         sleep 1
     34         EXPECT_PASS_BRK ping$TST_IPV6 -c1 -I $(tst_ipaddr) $rhost \>/dev/null
     35         ROD ip route del $rt dev $iface
     36         tst_del_ipaddr -s -q -a $rhost rhost
     37 }
     38 
     39 tst_run
```

## LTP所需服务和工具的安装&配置
__client需要安装的服务&工具__
1. xinetd
2. rshd
3. rsync
4. vsftpd
5. ftp
6. telnet，telnet-server
7. fingerd
8. nfsd,echo
9. dhcpd
10. dhcpd6
11. bind，bind-utils
12. psmisc，
13. expect
14. httpd
15. rstatd
16. dnsmasq
17. nfslock
18. ssh
19. nft（ iptables-1.8.5、iptables-devel-1.8.5、iptables-help-1.8.5、iptables-libs-1.8.5、iptables-nft-1.8.5、libnftnl-1.1.7、libnftnl-devel-1.1.7、nftables-0.9.6、nftables-devel-0.9.6、nftables-help-0.9.6）

__server需要安装的服务&工具__
1. xinetd
2. rshd
3. rsync
4. vsftpd
5. ftp
6. telnet,telnet-server
7. fingerd
8. nfsd
9. echo
10. dhcpd
11. dhcpd6
12. bind，bind-utils
13. psmisc
14. expect
15. rdist
16. httpd
17. rstatd
18. rwho
19. dnsmasq
10. nfslock
21. ssh
22. nft（ iptables-1.8.5、iptables-devel-1.8.5、iptables-help-1.8.5、iptables-libs-1.8.5、iptables-nft-1.8.5、libnftnl-1.1.7、libnftnl-devel-1.1.7、nftables-0.9.6、nftables-devel-0.9.6、nftables-help-0.9.6）

### xinetd
* 安装
```
yum -y install xinetd
systemctl restart xinetd
systemctl enable xinetd

```

### rsh
参考http://t.zoukankan.com/dezai1223-p-5193559.html

* 安装  
`yum -y install rsh  rsh-server`

* 在/etc/xinetd.d/下分别添加rsh，rlogin文件（如果没有，则创建）

rsh：修改/etc/xinetd.d/rsh文件，确保disable = no，内容如下：
```
# default: on
# description: The rshd server is the server for the rcmd(3) routine and,
# consequently, for the rsh(1) program. The server provides
# remote execution facilities with authentication based on
# privileged port numbers from trusted hosts.
service shell
{
socket_type = stream
wait = no
user = root
log_on_success += USERID
log_on_failure += USERID
server = /usr/sbin/in.rshd
disable = no
}
```

rlogin：修改/etc/xinetd.d/rlogin文件，确保disable = no，内容如下：
```
# default: on
# description: rlogind is the server for the rlogin(1) program. The server
# provides a remote login facility with authentication based on
# privileged port numbers from trusted hosts.
service login
{
socket_type = stream
wait = no
user = root
log_on_success += USERID
log_on_failure += USERID
server = /usr/sbin/in.rlogind
disable = no
}
```

* 在/etc/securetty文件最后，添加rexec、rsh、rlogin三行，执行如下命令：
```
echo "rexec" >> /etc/securetty
echo "rsh" >> /etc/securetty
echo "rlogin" >> /etc/securetty
```

* 编辑/etc/hosts，添加client和server的ip、主机名
```
2.1.1.1 client
2.1.1.2 server
2011::15:2 client
2011::15:3 server
```

* 创建~/.rhosts文件 ,添加client和server的主机名、用户名
```
client root
server root
```

* 注销/etc/pam.d/rsh文件中的 auth  required  pam_securetty.so行
```
#%PAM-1.0
# For root login to succeed here with pam_securetty, "rsh" must be
# listed in /etc/securetty.
auth       required     pam_nologin.so
#auth       required     pam_securetty.so
auth       required     pam_env.so
auth       required     pam_rhosts.so
account    include      password-auth
session	   optional     pam_keyinit.so    force revoke
session    required     pam_loginuid.so
session    include      password-auth
```
* 注销/etc/pam.d/rlogin文件中的 auth  required  pam_securetty.so行
```
#%PAM-1.0
# For root login to succeed here with pam_securetty, "rlogin" must be
# listed in /etc/securetty.
auth       required     pam_nologin.so
#auth       required     pam_securetty.so
auth       required     pam_env.so
auth       sufficient   pam_rhosts.so
auth       include      password-auth
account    include      password-auth
password   include      password-auth
session	   optional     pam_keyinit.so    force revoke
session    required     pam_loginuid.so
session    include      password-auth
```

* 重启xinetd服务  
`service xinetd restart`

* 关闭防火墙  
`systemctl stop firewalld`

* 加入开机自启动  
`service enable xinetd`

* 测试验证，免密登录（client和server都安装rsh、xinetd并进行以上配置）：
server：
```
[root@server ~]# rsh client
Last login: Thu Jul 21 12:40:10 from server
```

client：
```
[root@client ~]# rsh server
Last login: Thu Jul 21 12:40:10 from client
```

### rsync
参考https://blog.csdn.net/weixin_52270081/article/details/118196766

* 安装  
`yum -y install rsync`

* 启动服务  
`rsync --daemon`

* 查看进程  
`ps -ef | grep rsync   netstat -anpt | grep 873`

* 拷贝文件测试：  
本地模式，类似于cp命令： 
`rsync /var/tools/* /var/share` 			#将本地的/var/tools/下所有的文件拷贝到本地/var/share目录下

远程模式，类似于scp命令：  
`rsync -r /var/tools 1.1.1.3:/var/`		 	#将本地的/var/tools/下所有的文件拷贝到1.1.1.3的/var目录下

* 加入开机自启动  
`systemctl enable rsync`

### ftp
* 安装  
`yum -y install ftp`

### vsftpd
参考https://blog.csdn.net/qq_48391148/article/details/124081167
* 安装  
`yum -y install vsftpd`

* 启动服务：  
`service vsftpd start`

* 加入开机自启动  
`systemctl enable vsftpd`

* 查看进程
```
ps aux | grep vsftpd        
netstat -anplut | grep vsftpd
```

* 登录验证
`useradd ftp1`			#创建用户ftp1
`echo 123456|passwd ftp1 --stdin`			#给ftp1设置密码
`ftp 1.1.1.2`			#登录验证

### telnet,telnet-server
参考https://blog.51cto.com/u_15475949/4884767
* 安装  
`yum -y install telnet    telnet-server`

* 修改telnet服务配置文件/etc/xinetd.d/telnet，没有则创建，将disable=yes改为disable=no，内容如下：
```
service telnet
{
disable = no
flags = REUSE
socket_type = stream
wait = no
user = root
server = /usr/sbin/in.telnetd
log_on_failure += USERID
}
```

* 需要把server服务器的/usr/lib/systemd/system/telnet.socket文件 修改为/usr/lib/systemd/system/telnet.socket.bak
`mv /usr/lib/systemd/system/telnet.socket /usr/lib/systemd/system/telnet.socket.bak`

* 重启服务  
`systemctl restart xinetd`

* 加入开机自启动  
`systemctl enable telnet`

* 连接测试  
`telnet server`			#在client服务器执行，然后输入用户名密码可以登录成功
`telnet client`			#在server服务器执行，然后输入用户名密码可以登录成功

### finger
参考https://blog.csdn.net/culinqian4296/article/details/108788016
* 安装  
`yum -y install finger`

* 测试验证
```
#finger
Login     Name       Tty      Idle  Login Time   Office     Office Phone   Host
root      root       pts/0          Aug  5 17:01                           (1.1.1.1)
```

### nfs
参考https://blog.csdn.net/qq_42835445/article/details/122130709
* 安装  
`yum -y install nfs-utils`

如果yum安装失败，则需要把nfs-utils和其所有依赖包手动下载并安装，共16个安装包，把安装包放到服务器的同一个目录中，然后执行如下命令：
```
rpm -ivh libnftnl*.rpm
rpm -ivh iptables-*.rpm --force --nodeps
rpm -ivh nftables-*.rpm --force --nodeps
```

* 服务器创建共享目录
```
#mkdir share        
#pwd
/var/share
```

* 服务器配置共享目录  
编辑/etc/exports文件，添加共享命令：
`/var/share 1.1.1.1(rw,sync,no_root_squash,no_all_squash)`

* 启动服务
```
systemctl start rpcbind
systemctl start nfs
```

* 添加开机自启动
```
Systemctl enable  nfs 
Systemctl enable rpcbind 
```

* 启动后查看服务是否共享成功（ 显示刚才配置的共享目录和IP说明共享生效 ）
```
#showmount -e localhost
Export list for localhost:
/var/share 1.1.1.1
```

### dhcpd
参考https://www.jb51.net/article/220118.htm
* 安装  
`yum -y install dhcp`

* dhcp服务是需要先配置一下配置文件才能启动的，刚安装好的配置文件(/etc/dhcp/dhcpd.conf)是空的，所以启动不起来，会报错。配置如下：
需要先ifconfig查看dhcp服务器的本地IP（必须是静态IP，如本地ip是1.1.1.1），然后 vi/etc/dhcp/dhcpd.conf，添加以下内容（配置文件的网段与dhcp服务器网段一致）：
```
option domain-name "1.1.1.254";
     ##域名:参见/etc/resolv.conf
option domain-name-servers 1.1.1.254;
     ##指定dns服务器，多台用逗号隔开。
subnet 1.1.1.0 netmask 255.255.255.0 {
     ##指定子网络及子网掩码
range 1.1.1.10 1.1.1.20;
     ##指定IP范围
option routers 1.1.1.254;
    ##指定默认网关
}
```

* 启动服务  
`systemctl start dhcpd`

* 查看端口正常监听，运行正常，监听67端口  
`netstat -anptu | grep dhcp`

### dhcpd6

编辑/etc/dhcp/dhcpd6.conf		#添加如下内容：
```
default-lease-time 600;
max-lease-time 7200;
log-facility local7;
#subnet6 2008:db8:0:2::/64 {
#range6 2008:db8:0:2::20 2008:db8:0:2:30;
#option dhcp6.name-servers 2008:db8:0:1::200;
#        option dhcp6.domain-search "lab.local";
#}
subnet6 2008:db8:0:3::/64 {
        interface enp5s0;
range6 2008:db8:0:3::201 2008:db8:0:3::999;
option dhcp6.name-servers 2008:db8:0:1::200;
option dhcp6.domain-search "lab.local";
}
```

* 启动服务  
`systemctl restart dhcpd6`

* 加入开机自启动  
`systemctl status dhcpd6`

### bind
* 安装  
`yum -y install bind`

### bind-utils
* 安装  
`yum –y install bind-utils`

### psmisc
* 安装  
`yum -y  install psmisc`

### expect
* 安装  
`yum -y install expect`

### rdist
参考：https://blog.csdn.net/rj042/article/details/4980748
* 安装  
`yum -y install rdist`

* 需要在目的主机上在~/.rhosts加对方的主机名和用户名
```
[root@client testscripts]# cat ~/.rhosts
client root
server root

[root@server .ssh]# cat ~/.rhosts 
server root
client root
```

### http
参考：https://blog.csdn.net/qq_45609914/article/details/122470131
* 安装  
`yum -y install httpd`		#默认已安装

* 查看http服务状态  
`systemctl status httpd`

* 启动服务 
`systemctl start httpd`

* 加入开机自启动  
`systemctl enable httpd`

* 查看端口正常监听  
`netstat -anptu | grep httpd`

### rstatd,rusersd
* 安装  
`yum -y install rstatd rusers rusers-server`

* 启动rstatd服务  
`systemctl start rstatd`

* 启动rusersd服  
`systemctl start rusersd`

* 重启xinetd服务  
`systemctl restart xinetd`

* 加入开机自启动
```
systemctl enable rstatd
systemctl status rusersd
```

### rwho
* 安装  
`yum -y install rwho`

* 加入开机自启动  
`systemctl enable rwhod`

### dnsmasq
* 安装  
`yum -y install dnsmasq`

### nfslock
* 安装  
`yum –y install nfslock`

* 重启服务  
`systemctl restart nfslock`

* 加入开机自启动  
`systemctl enable nfslock`

### nft
* 默认iptables版本是1.4.21，需要重新手动下载新版本并安装，所需软件包/依赖包如下（安装包来源openeuler 20.03 LTS 3）
```
iptables-1.8.5-2.oe1.x86_64.rpm
iptables-devel-1.8.5-2.oe1.x86_64.rpm
iptables-help-1.8.5-2.oe1.noarch.rpm
iptables-libs-1.8.5-2.oe1.x86_64.rpm
iptables-nft-1.8.5-2.oe1.x86_64.rpm
libnftnl-1.1.7-1.oe1.x86_64.rpm
libnftnl-devel-1.1.7-1.oe1.x86_64.rpm
nftables-0.9.6-4.oe1.x86_64.rpm
nftables-devel-0.9.6-4.oe1.x86_64.rpm
nftables-help-0.9.6-4.oe1.noarch.rpm
```

* 安装
```
rpm -ivh libnftnl*.rpm
rpm -ivh iptables-*.rpm --force --nodeps
rpm -ivh nftables-*.rpm --force --nodeps
```
__注：这些包会破坏系统iptables原始版本1.4.21，iptables版本由iptables v1.4.21变为iptables v1.8.5 (legacy)__

### dns
* 在client配置dns，在管理口网卡中添加本机ip为DNS，cat /etc/sysconfig/network-scripts/ifcfg-enp5s0 如下：
```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=no
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
#IPV6ADDR=2003::10:2/64
IPV6ADDR=2008:db8:0:3::202
#IPV6_DEFAULTGW=2003::10:1
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp5s0
UUID=f76353db-f4ca-428a-9da1-75fe7eb8fa6f
DEVICE=enp5s0
ONBOOT=yes
IPADDR=1.1.1.1
NETMASK=255.255.255.0
GATEWAY=1.1.1.254
DNS1=1.1.1.1
DNS2=114.114.114.114
```
* 重启网卡  
`service network restart`

* 查看cat /etc/resolv.conf是否生效
```
nameserver 1.1.1.1
nameserver 114.114.114.114
```

* vi /etc/named.conf，修改下图中第一行和最后一行为“any”
```
options {
        listen-on port 53 { any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        allow-query     { any; };
```

* vi /etc/named.rfc1912.zones，在文件最下方添加client和server解析，如下：
```
zone "client" IN {
        type master;
        file "client.zone";
        allow-update { none; };
};

zone "server" IN {
        type master;
        file "server.zone";
        allow-update { none; };
};
```

* 在/var/named目录下添加两个文件：client.zone和server.zone（文件名称需要与步骤2中的file “client.zone”、file “server.zone”保持一致），两个文件内容分别如下:
以client配置为例： 
__client.zone__
```
$TTL 1D
@	IN SOA	@ rname.invalid. (
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
	NS	@
	A	1.1.1.1
	AAAA	::1
www	A	1.1.1.1
client  A       1.1.1.1
```

__server.zone__
```
$TTL 1D
@	IN SOA	@ rname.invalid. (
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
	NS      server
	A       2.1.1.1	
www	A	2.1.1.1
server  A       2.1.1.1
```
* 重启服务  
`systemctl restart named`

* 测试验证  
在client执行如下：
```
#host client
client has address 1.1.1.1
client has IPv6 address ::1
```

在server执行如下：
```
#host server
server has address 2.1.1.1
```

__注：如果重启设备后发现host用例又失败了，检查named服务是否正常启动，如果服务启动失败，根据systemctl status named的错误提示排查，可能是某个文件没有权限导致。__

### ssh
参考https://blog.csdn.net/Gurad2008/article/details/6270775
SSH访问远程主机建立信任关系，实现免密登录

**********************************************************************************
1. 在client执行ssh-keygen -b 1024 -t rsa,期间有几个选项让输入私钥，直接enter就行了
2. 第一步执行完成后，会在~/.ssh/下生成会生成几个文件，其中一个就是id_dsa.pub，这是生成该主机A的公钥
3. 把id_dsa.pub拷贝到server服务器的~/.ssh下面，并以authorized_keys的名字保存
$ scp id_rsa.pub user@IP: /root/.ssh/authorized_keys
4. 这样server就可以实现对client的无密码访问
5. 在server中修改/etc/hosts文件，加入client的hostname 和IP对应，就可以实现SSH用hostname就可以对client进行访问
**********************************************************************************

## 运行测试用例
__保证以上服务分别在client和server都已安装并正常启动__

* 在client执行如下命令，对系统进行网络基础测试和压力测试：
`./ltp_network.sh >all.txt |tailf all.txt`		#ltp_network.sh脚本需要手动创建，创建方法和添加内容如下：

* 在client服务器的/opt/ltp/testscripts目录下新增脚本ltp_network.sh，并赋予可执行权限，添加以下内容：
```
#!/bin/bash

#yum -y  install bind bind-utils psmisc
export LTPROOT="/opt/ltp"					#ltp安装路径，client和server必须保持一致
export PATH="$LTPROOT/testcases/bin:$PATH"			
export TST_NET_RHOST_RUN_DEBUG=1
export RHOST="server"				#server服务器的主机名
export LHOST_HWADDRS="00:00:00:00:00:00"	#client服务器的test link网卡MAC地址
export RHOST_HWADDRS="00:00:00:00:00:00"		# server服务器的test link网卡MAC地址
export LTP_RSH="server"		#server服务器的主机名
export RUSER="root"			#server服务器用户名
export PASSWD=123456	#server服务器密码
export LHOST_IPV4_HOST="1.1.1.1"	#client服务器的test link网卡IP地址
export RHOST_IPV4_HOST="1.1.1.2"	# server服务器的test link网卡IP地址
export LHOST_IFACES="eth1"		# client服务器的test link网卡名称
export RHOST_IFACES="eth1"		# server服务器的test link网卡名称
export IPV4_LHOST="1.1.1.1"			#client服务器的test link网卡IP地址
export IPV4_RHOST="1.1.1.2"			# server服务器的test link网卡IP地址
export LHOST_IPV6_HOST="2011::10:2"	# client服务器的test link网卡IP地址
export RHOST_IPV6_HOST="2011::10:3"	# server服务器的test link网卡IP地址
export IPV6_LHOST="2011::10:2"	# client服务器的test link网卡IP地址
export IPV6_RHOST="2011::10:3"	# server服务器的test link网卡IP地址
export LTP_TIMEOUT_MUL=6		#设置超时时间n*5
./network.sh -6mnrstcdaebiTUDSRMFV		#运行network.sh脚本的所有测试项中的用例，V是输出详细测试信息
#./network.sh –FV			#运行单个测试项用例
```
__附：其他模块测试__
| 序号 | 模块测试 | 说明 |
| --- | --- | --- |
| 1 | ./runltp -f commands | 系统常规命令 |
| 2 | ./runltp -f syscalls | 系统内核调用 |
| 3 | ./runltp -f fs | 文件系统压力测试 |
| 3 | ./testscripts/network.sh -6mmrstcdaebiTUDSRMF | 压测环境配置network/stress/README |
| 4 | ./runltp -p -l /tmp/resultlog.info -d /tmp -o /tmp/ltpscreen.info | 默认测试 |
