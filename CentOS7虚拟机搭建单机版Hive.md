# CentOS7虚拟机搭建单机版Hive

## 1、安装JDK1.8、Hadoop3.2.0和Mysql5.7.30

jdk和Hadoop的配置可以参考[链接](https://github.com/MachineGunLin/Linux_and_K8S_stuff/blob/master/CentOS%E8%99%9A%E6%8B%9F%E6%9C%BA%E9%83%A8%E7%BD%B2Hadoop-3.2.0(%E5%8D%95%E8%8A%82%E7%82%B9%E3%80%81%E4%BC%AA%E5%88%86%E5%B8%83%E5%BC%8F.md](https://github.com/MachineGunLin/Linux_and_K8S_stuff/blob/master/CentOS虚拟机部署Hadoop-3.2.0(单节点、伪分布式).md)

mysql的配置可以参考[这个链接](https://blog.csdn.net/cool_summer_moon/article/details/106090136)



## 2、下载Hive

下载地址是：http://mirror.bit.edu.cn/apache/hive/，我这里下载的是apache-hive-2.3.7-bin.tar.gz

> wget http://mirror.bit.edu.cn/apache/hive/hive-2.3.7/apache-hive-2.3.7-bin.tar.gz

用windows下载下来再xftp上传到虚拟机也是可以的



## 3、解压Hive

这里选择解压到/opt目录下：

> tar -zxvf apache-hive-2.3.7-bin.tar.gz -c /opt/

如果报错，可以试着去掉-z参数



## 4、配置Hive环境变量

修改配置文件：

> vim /etc/profile

在文件末尾追加两句话：

> ```
> export HIVE_HOME=/opt/apache-hive-2.3.7-bin
> export PATH=$PATH:$HIVE_HOME/bin
> ```

修改文件之后source一下：

> source /etc/profile



## 5、配置Hive

> cd /opt/apache-hive-2.3.7-bin/conf/

注意下面的ConnectionUserName和ConnectionPassword是Mysql的用户名和密码。

>  vim hive-site.xml

 ![image-20200604204819659](C:\Users\Lin\AppData\Roaming\Typora\typora-user-images\image-20200604204819659.png)

> cp hive-env.sh.template hive-env.sh

> vim hive-env.sh

在文件末尾追加两句：

> HADOOP_HOME=/software/hadoop-3.2.0
>
> export HIVE_CONF_DIR=/opt/apache-2.3.7-bin/conf

注意HADOOP_HOME填写的是自己环境上hadoop的安装目录。



## 6、加载mysql驱动（要和自己安装的版本一致）

下载地址：https://dev.mysql.com/downloads/connector/j/

下载和mysql-5.7.30匹配的驱动，解压jar包放到/opt/apache-hive-2.3.7-bin/lib目录下。



## 7、初始化数据库

> schematool -initSchema -dbType mysql



## 8、启动Hive

启动hive之前先启动hadoop，可以用命令jps查看hadoop是否启动成功。如何启动hadoop在前面的hadoop安装链接里有详细步骤。

启动hive：

> hive

测试一下：

> show databases;

显示如下说明启动成功：



![image-20200604205745901](C:\Users\Lin\AppData\Roaming\Typora\typora-user-images\image-20200604205745901.png)



建表：

> ```
> CREATE TABLE IF NOT EXISTS test (id INT,name STRING)ROW FORMAT DELIMITED FIELDS TERMINATED BY " " LINES TERMINATED BY "\n";
> ```

插入数据：

> ```
> insert into test values(1,'张三');
> ```

查询:

> select * from test;



![image-20200604205935732](C:\Users\Lin\AppData\Roaming\Typora\typora-user-images\image-20200604205935732.png)
