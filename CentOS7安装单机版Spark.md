# CentOS7安装单机版Spark

由于Spark依赖于scala，所以安装spark之前需要先安装scala。

我们先安装scala-2.12.2

到[Scala官网](https://www.scala-lang.org/download/2.12.2.html)上下载一下scala-2.12.2.tgz

![img](https://ask.qcloudimg.com/http-save/yehe-2302547/4minrr266d.png?imageView2/2/w/1620)

把压缩包放到虚拟机的/opt/scala目录下，随后进行解压:

> cd /opt/scala
>
> tar -xvf scala-2.12.2.tgz

查看一下是否解压成功:

> ls

![image-20200604224520694](C:\Users\Lin\AppData\Roaming\Typora\typora-user-images\image-20200604224520694.png)

修改环境变量:

> vim /etc/profile

> export JAVA_HOME=/software/jdk1.8.0_201
> export JRE_HOME=${JAVA_HOME}/jre
> export PATH=$JAVA_HOME/bin:$PATH
> export HIVE_HOME=/opt/apache-hive-2.3.7-bin
> export SCALA_HOME=/opt/scala/scala-2.12.2
> export PATH=$PATH:${HIVE_HOME}/bin:${SCALA_HOME}/bin:

文件的结尾如上所示，这一步我们加入了SCALA_HOME，并且在PATH中加入了**${SCALA_HOME}/bin:**

验证一下scala版本是否正确:

> scala -version

![image-20200604224840671](C:\Users\Lin\AppData\Roaming\Typora\typora-user-images\image-20200604224840671.png)

到此scala安装完成，下面安装Spark。

Spark我下载的压缩包是spark-3.0.0-preview2-bin-hadoop3.2.tgz，放到了/opt/spark目录下并解压：

> cd /opt/spark
>
> tar -xvf spark-3.0.0-preview2-bin-hadoop3.2.tgz
>
> ls

![image-20200604225152518](C:\Users\Lin\AppData\Roaming\Typora\typora-user-images\image-20200604225152518.png)

在/etc/profile中添加环境变量:

> export SPARK_HOME=/opt/spark/spark-3.0.0-preview2-bin-hadoop3.2
>
> export PATH=$PATH:${HIVE_HOME}/bin:${SCALA_HOME}/bin:${SPARK_HOME}/bin:

配置spark：

> cd /opt/spark/spark-3.0.0-preview2-bin-hadoop3.2/conf/

ls一下，可以发现有一个模板文件，copy一份:

> cp spark-env.sh.template spark-env.sh

对拷贝的文件添加一些内容:

> vim spark-env.sh

> export JAVA_HOME=/software/jdk1.8.0_201
> export SCALA_HOME=/opt/scala/scala-2.12.2
> export SPARK_HOME=/opt/spark/spark-3.0.0-preview2-bin-hadoop3.2
> export SPARK_MASTER_IP=learn
> export SPARK_EXECUTOR_MEMORY=1G

同时拷贝一份slaves:

> cp slaves.template slaves

编辑slaves,内容为localhost:

> vim slaves
>
> localhost

做个测试:

> ./bin/run-example SparkPi 10

![image-20200604230019863](C:\Users\Lin\AppData\Roaming\Typora\typora-user-images\image-20200604230019863.png)

## 启动Spark Shell

> cd /opt/spark/spark-3.0.0-preview2-bin-hadoop3.2
>
> ./bin/spark-shell

![image-20200604230350673](C:\Users\Lin\AppData\Roaming\Typora\typora-user-images\image-20200604230350673.png)

单机版的Spark安装完成！