## 一 安装三台虚拟机环境

需要搭建三台虚拟机环境，以实现集群功能，镜像选择 Minimal Install，勾选 Compatib、Development两个选项卡即可。   

使用VMWare安装三台CentOS7虚拟机环境。  
- 输入win 命令`services.msc`后，启动所有vm开头的服务
- 选择VMWare菜单->编辑->虚拟网络编辑器->VMnet8（NAT模式）-> 点击下方NAT设置->记录出现的网关IP，如192.168.120.2
- 打开控制面板中的网络连接适配器，选择vmnet8连接->右键属性->选择协议版本IPV4->这里取消自动获得IP手动填写一个
  - IP：192.168.120.5（只要120网段与上一步一致即可）
  - 子网：255.255.255.0
  - 网关：即上面记录的网关192.168.122.2
- 将安装好的虚拟机复制三份，复制文件即可。重命名三个文件夹后，点击文件内的CentOS.vmx文件即可打开虚拟机。
  - 每台虚拟机分配的内存建议为：(总内存-n)/3，8g内存n为2，16g内存n为4

安装完毕后，可以直接复制安装的第一个镜像，制作三台虚拟机。

## 二 三台虚拟机分别配置网络

修改mac地址：右键虚拟机名->设置->网络适配器->高级 可以获得mac，必须生成一个新的mac地址
```
# 删除列出的Mac地址配置，只保留一个mac配置，且mac地址address必须刚才生成的mac地址，并将Name值改为 eth0
vim /etc/udev/rules.d/70-persistent-net.rules
```

修改ip地址：
```
vim /etc/sysconfig/network-scripts/ifcfg-eth0

DEVICE=eth0
HWADDR=00:0C:29:85:56:8B    # 值为本虚拟机的mac地址
ONBOOT=yes                  # 是否启动时激活网卡
BOOTPROTO=static
IPADDR=192.168.120.131      # 三台机器依次进行上述配置，IP依次为131，132，133
GATEWAY=192.168.120.2       # 此处是虚拟机本身的网关
NETMASK=255.255.255.0
DNS1=8.8.8.8
```

修改域名配置：
```
vim /ect/sysconfig/network
HOSTNAME=node01             # 注意这里每台机器分别对应node01，node02，node03
``` 

配置域名映射：每台机器添加相同的如下hosts文件
```
vim /etc/hosts  

192.168.120.131 node01.hadoop.com node01
192.168.120.132 node02.hadoop.com node02
192.168.120.133 node03.hadoop.com node03
```

## 三 三台机器分别关闭防火墙与selinux

关闭防火墙：
```
systemctl stop firewalld.service		# 关闭防火墙，centos6命令为：service iptables stop
systemctl disable firewalld.service     # 进制防火墙开机启动，centOS6命令我为：chkconfig iptables off
```

关闭selinux：
```
vim /etc/selinux/config		            新的：
SELINUX=disabled                        # 注释 SELINUX=enforcing，添加                  
```

二与三步骤设置完毕后重启:`reboot  -h  now`，ping www.baidu.com  能ping通则上述配置通过

## 四 三台机器配置ssh免密登录

每台机器都依次执行以下操作：
```
ssh-keygen -t rsa
ssh-copy-id node01
ssh-copy-id node02
ssh-copy-id node03
```

每台机器测试：
```
ssh node01
ssh node02
ssh node03

ping node01
ping node02
ping node03
```

## 四 三台机器配置时钟同步

### 4.1 网络方式配置时钟同步

三台机器的时钟同步，这里使用网络同步方式：
```
# 使用阿里云时钟同步
yum install -y ntp
ntpdate ntp4.aliyun.com        

# 定时执行时钟同步
crontab  -e
*/1 * * * * /usr/sbin/ntpdate ntp4.aliyun.com;      # 粘贴该命令
```

### 4.2 本地方式配置时钟同步

如果一些大数据环境不能联网，则可以使用某台机器进行同步的方式，如下所示：  

以192.168.120.131这台服务器的时间为准进行时钟同步。  
```
# 第一台机器安装ntp
yum -y install ntpd         
service  ntpd  start
chkconfig ntpd o

# 编辑第一台机器
vim /etc/ntp.conf
# 添加内容：
restrict 192.168.120..131  mask  255.255.255.0  nomodify  notrap
# 注释下列内容：
#server  0.centos.pool.ntp.org
#server  1.centos.pool.ntp.org
#server  2.centos.pool.ntp.org
#server  3.centos.pool.ntp.org
# 恢复如下内容：
server   127.127.1.0  #  local  clock
fudge    127.127.1.0  stratum  10

# 配置第一台机器bios与系统时间同步
vim  /etc/sysconfig/ntpd
# 添加内容
SYNC_HWLOCK=yes


# 配置其他机器同步：
crontab  -e
*/1 * * * * /usr/sbin/ntpdate 192.168.120..131
```

## 五 三台机器安装jdk8

卸载默认java
```
rpm -qa | grep jdk              # 查看列表
yum remove *openjdk* -y
yum remove copy-jdk* -y
```

安装jdk8
```
tar -zxvf jdk-8u141-linux-x64.tar.gz -C /usr/local

vim /etc/profile
export JAVA_HOME=/usr/local/jdk1.8.0_141
export PATH=:$JAVA_HOME/bin:$PATH

source /etc/profile
```

## 六 选择一台机器安装mysql

选择一台机器安装mysql，为了方便学习，这里直接选择yum在线安装方式：
```
# 安装mysql
yum install mysql mysql-server mysql-devel -y

# 启动mysql
/etc/init.d/mysqld start

# 设置密码 会提示输入root密码，此时没有直接回车，然后会提示设置root密码。还会提示是否不允许root用户远程登录，一定要填写no，其他选择Y即可
/usr/bin/mysql_secure_installation   

# 设置可以root登录，开放权限
mysql -uroot -p
grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
flush privileges;
exit;
```

## 七

关机注意事项：由于hadoop每次都要重新启动，虚拟机的关机最好使用挂起方式，下次开机只用重启恢复即可。
