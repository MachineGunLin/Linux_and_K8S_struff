## centos服务器安装搭建KVM虚拟机环境

作者：XYC

时间：2020年5月10日

参考：

- [https://github.com/FDU-YSP/YSP_doc/blob/master/kvm%E8%99%9A%E6%8B%9F%E6%9C%BA/centos%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%89%E8%A3%85%E6%90%AD%E5%BB%BAKVM%E8%99%9A%E6%8B%9F%E6%9C%BA%E7%8E%AF%E5%A2%83.md](https://github.com/FDU-YSP/YSP_doc/blob/master/kvm虚拟机/centos服务器安装搭建KVM虚拟机环境.md)

- https://blog.csdn.net/weixin_36820871/article/details/80595855

### 1. 安装虚拟化（在宿主机上的操作）

```
yum -y install qemu-kvm libvirt virt-manager virt-viewer
```

### 2. 支持Xshell GUI界面(装好后重新登录一下)

```
yum -y install xorg-x11-xauth mesa-dri-drivers ghostscript-chinese-zh_CN
```

注意：在上面两条命令全部完全正确的执行以后，我们的虚拟机软件环境安装好了。

### 3. 关闭宿主机的防火墙和selinux

```bash
# 查看防火墙状态
systemctl status firewalld.service

# 如果防火墙的状态处在active(running)的状态
systemctl stop firewalld
# 如果想彻底关闭防火墙 即使重启服务器也不会再开启防火墙 则执行（使某服务不自动启动）
systemctl disable firewalld.service

# 临时关闭selinux的方法 执行下面的命令
setenforce 0
# 永久关闭则需要修改配置文件
vi /etc/selinux/config
# 编辑这个文件 添加如下新的一行 然后重启
SELINUX=disabled

reboot
```

### 4. 进入图形化的虚拟机安装和管理界面

```bash
virt-manager
```

### 5. 使用virt-manager图形化安装kvm虚拟机

​    将centos镜像通过xftp上传到宿主机上，在安装kvm虚拟机时选择这个上传到的iso镜像。内存和cpu核心数一般选择8/16GB，cpu核心数根据宿主机的实际硬件配置进行选择。

### 6. 虚拟机的网络配置

```bash
cd /etc/sysconfig/network-scripts/
ls 
vi ifcfg-eth0
# 修改或者添加后面加注的那几行
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static # 设置为静态ip  对应则是dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
UUID=478cbfaa-9368-499a-8b25-1006fedb5f82
DEVICE=eth0 # 表示针对eth0网卡
ONBOOT=yes # 表示已经启用
# 这个里面的IP地址和网关网段都与虚拟机的网卡有关，默认的网卡网段是192.168.122.x
IPADDR=192.168.122.X # (2---254) 具体的网段请根据virtual network的地址范围
GATEWAY=192.168.122.1
NETMASK=255.255.255.0
DNS1=8.8.8.8
DNS2=202.120.224.6 # 根据自己实际情况编辑
```

重启网络服务使配置生效（下面两种都可以）

```bash
service network restart
systemctl restart network.service
```

### 7. 开启宿主机的内核转发功能

​    如果在实际的物理机环境下，重装完系统后，配置完网卡的配置文件ping测试一下外网地址，一般就可以访问了。但安装在服务器上虚拟机则是不同的。Linux系统缺省并没有打开IP转发功能，要确认IP转发功能的状态，可以查看/proc文件系统，使用下面命令：

```bash
cat /proc/sys/net/ipv4/ip_forward
```

**次如果上述文件中的值为0,说明禁止进行IP转发；如果是1,则说明IP转发功能已经打开。**

如果是关闭的，则虚拟机没法连接到外网，只能虚拟机和宿主机之间是可以ping通的。

动态修改内核参数的方法，但这种方法重启失效：

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

若要永久修改，则需要修改`/etc/sysctl.conf` 文件，

```bash
# 增加一行
net.ipv4.ip_forward=1
```

可以重启系统来使修改生效，也可以执行下面的命令来使修改生效

```bash
sysctl -p /etc/sysctl.conf
```

### 8. 重启libvirtd服务

```bash
systemctl restart libvirtd
```

### 9. ssh连接虚拟机

```bash
ssh 192.168.122.X
```

### 10.补充

```bash
# 查看正在运行的虚拟机列表
cirsh list 
```

### 11.修改网关

1. 修改网段

```bash
# 不想使用已有网关网段想自行修改网段的话
# NAT方式
# 此命令定义一个虚拟网络
virsh net-define /usr/share/libvirt/networks/default.xml
```

### default.xml中的内容（可以自行修改网关和ip地址范围）

```bash
<network>
  <name>default</name>
  <bridge name="virbr0"/>
  <forward/>
  <ip address="10.0.0.1" netmask="255.255.255.0">
    <dhcp>
      <range start="10.0.0.2" end="10.0.0.254"/>
    </dhcp>
  </ip>
</network>
```

2. 标记为自动启动

```bash
virsh net-autostart defaulte
```

3. 启动网络

```bash
virsh net-start default
```

### 12.删除已有的虚拟机网络

1. 删除网络定义

```bash
virsh net-undefine default
```

2. 删除网络

```bash
virsh net-destroy default
```

3. 查看网络

```bash
virsh net-list
```

## 注意：

>虚拟网络网关修改之后，需要：
>
>```bash
>systemctl restart libvirtd
>```
>
>并且检查一下步骤7是否ok