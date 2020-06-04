# CentOS安装单机版HBase

首先看一下Hadoop和HBase的版本关系：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191212185226311.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hBQ0tFUlJPTkdHRQ==,size_16,color_FFFFFF,t_70)

由于我之前安装的Hadoop版本是3.2.0,这里选择安装HBase-2.2.5



## 1、安装Java和Hadoop

参考[链接]([https://github.com/MachineGunLin/Linux_and_K8S_stuff/blob/master/CentOS%E8%99%9A%E6%8B%9F%E6%9C%BA%E9%83%A8%E7%BD%B2Hadoop-3.2.0(%E5%8D%95%E8%8A%82%E7%82%B9%E3%80%81%E4%BC%AA%E5%88%86%E5%B8%83%E5%BC%8F).md](https://github.com/MachineGunLin/Linux_and_K8S_stuff/blob/master/CentOS虚拟机部署Hadoop-3.2.0(单节点、伪分布式).md))



## 2、安装HBase

HBase2.2.5下载地址：https://www.apache.org/dyn/closer.lua/hbase/2.2.5/hbase-2.2.5-bin.tar.gz

下载完成后解压：

> sudo tar -vxzf hbase-2.2.5-bin.tar.gz
>
> sudo mv -f hbase-2.2.5-bin.tar.gz hbase

配置hbase-env.sh:

> cd hbase
>
> vim conf/hbase-env.sh

设置如下参数:

> export JAVA_HOME=/software/jdk1.8.0_201
> export HBASE_MANAGES_ZK=flase

配置hbase-site.xml:

> <configuration>
>   <property>
>     <name>hbase.rootdir</name>
>     <value>hdfs://localhost:9000/hbase</value>
>   </property>
>   <property>
>     <name>hbase.tmp.dir</name>
>     <value>./tmp</value>
>   </property>
>   <property>
>       <name>hbase.cluster.distributed</name>
>       <value>false</value>
>   </property>
>   <property>
>     <name>hbase.unsafe.stream.capability.enforce</name>
>     <value>false</value>
>   </property>
> </configuration>

安装HBase：

进入HBase目录：

> cd /hbase

启动HBase:

> sudo ./bin/start-hbase.sh

查看是否启动成功:

> jps

![image-20200604223342928](C:\Users\Lin\AppData\Roaming\Typora\typora-user-images\image-20200604223342928.png)

能看到HMaster，说明安装成功。