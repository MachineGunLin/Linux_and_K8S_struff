# Virtual-kubelet部署文档

1.  首先连接上搭建好的openstack的controller：

   ![image-20200701112731868](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701112731.png)

   2. 查看正在运行的容器：`docker ps`

      ![image-20200701113448978](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701113449.png)

      找到mariadb容器，进入容器：`docker exec -it mariadb bash`

      ![image-20200701113707338](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701113707.png)

   3. 尝试登录mysql: `mysql -uroot -p`，发现需要密码

      ![image-20200701113939844](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701113939.png)

   4. 新建一个openstack controller的会话，切换到/etc/kolla/目录下，发现有一个passwords.yml存放着密码

![image-20200701114117526](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701114117.png)

5. `vim passwords.yml`

   进入文件之后查找database_password：

   ![image-20200701114258792](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701114258.png)

   将database_password的值复制到刚才要登录mysql时需要填入密码的地方：

   ![image-20200701114355681](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701114355.png)

6. 查看当前已有的数据库：`show databases；`

7. 现在数据库中还没有virtual-kubelet和hadoop数据库，我们需要创建这两个数据库：

   ```
   create database virtual-kubelet;
   create database hadoop;
   ```

   ![image-20200701120531628](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701120736.png)

8. 使用hadoop数据库：`use hadoop;`，然后source一下创建数据库的sql文件: `source /root/hadoop.sql`

   在这一步之前需要先在/root下创建hadoop.sql文件，内容如下：

   ```
   /*
    Navicat Premium Data Transfer
   
    Source Server         : 62服务器
    Source Server Type    : MySQL
    Source Server Version : 80016
    Source Host           : 10.10.87.62:3306
    Source Schema         : hadoop
   
    Target Server Type    : MySQL
    Target Server Version : 80016
    File Encoding         : 65001
   
    Date: 09/07/2019 14:59:31
   */
   
   SET NAMES utf8mb4;
   SET FOREIGN_KEY_CHECKS = 0;
   
   -- ----------------------------
   -- Table structure for hadoop_cluster
   -- ----------------------------
   DROP TABLE IF EXISTS `hadoop_cluster`;
   CREATE TABLE `hadoop_cluster`  (
     `cluster_name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
     `number_of_nodes` int(11) NULL DEFAULT NULL,
     `available_number` int(11) NULL DEFAULT NULL,
     `create_time` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
     `hadoop_master_id` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
     `hadoop_slave_id` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
     `cluster_status` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
     `namespace` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
     PRIMARY KEY (`namespace`, `cluster_name`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   
   -- ----------------------------
   -- Table structure for hadoop_nodes
   -- ----------------------------
   DROP TABLE IF EXISTS `hadoop_nodes`;
   
   ```

   ![image-20200701135825438](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701144928.png)

9.  再换到数据库virtual_kubelet：`use virtual_kubelet;`，source创建表的sql文件: `source /root/virtual_kubelet.sql`，同样地，需要先在/root下创建好virtual_kubelet.sql文件，内容如下：

   ```
   /*
    Navicat Premium Data Transfer
   
    Source Server         : 62服务器
    Source Server Type    : MySQL
    Source Server Version : 80016
    Source Host           : 10.10.87.62:3306
    Source Schema         : virtual_kubelet
   
    Target Server Type    : MySQL
    Target Server Version : 80016
    File Encoding         : 65001
   
    Date: 09/07/2019 14:59:40
   */
   
   SET NAMES utf8mb4;
   SET FOREIGN_KEY_CHECKS = 0;
   
   -- ----------------------------
   -- Table structure for Pod
   -- ----------------------------
   DROP TABLE IF EXISTS `Pod`;
   CREATE TABLE `Pod`  (
     `NameSpace` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
     `Name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
     `Namespace_Name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
     `container_id` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
     `pod_kind` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
     `pod_uid` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
     `pod_createtime` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
     `pod_clustername` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
     `node_name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
     PRIMARY KEY (`Namespace_Name`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   
   SET FOREIGN_KEY_CHECKS = 1;
   
   ```

   ![image-20200701140133626](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701144939.png)

10. 在openstack controller所在机器上切换目录到.kube: `cd /.kube`，并将K8S所在机器上的/etc/kubernetes/admin.conf  文件  scp到.kube目录下, 讲admin.conf文件改名为config: `mv admin.conf config`，并修改文件权限为777：`chmod 777 config`

    ![image-20200701141637088](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701141637.png)

11. 切换为K8S所在的机器，创建一个目录用来存放virtual-kubelet:  `mkdir -p $GOPATH/src/github.com/virtual-kubelet/`（这一步执行前需要安装golang并配置好相应的环境变量（$GOPATH等）

12. 在刚创建的目录下将virtual-kubelet项目clone下来并编译：

    ```
    cd $GOPATH/src/github.com/virtual-kubelet/
    git clone https://github.com/virtual-kubelet/openstack-zun.git
    cd $GOPATH/src/github.com/virtual-kubelet/virtual-kubelet/openstack-zun
    make
    ```

    ![image-20200701142428374](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701142428.png)

13.  编译成功之后会产生一个可执行文件**virtual-kubelet**, 在openstack-zun目录下的bin目录里。

    ![image-20200701142650686](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701142650.png)

14. 在/root下创建一个文件夹vk_related_files，用来存放virtual_kubelet可执行文件以及openstack controller的认证文件，还有virtual-kubelet的启动脚本。创建之后把需要的文件 (virtual-kubelet可执行文件, 认证文件admin-openrc.sh, 启动脚本vk.sh) 放到目录下。

    (1) 可执行文件virtual-kubelet直接由刚才拉取项目的目录中的vk复制过来: `cp $GOPATH/src/github.com/virtual-kubelet/openstack-zun/bin    .     (当前在目录/root/vk_related_files里)

    (2) admin-openrc.sh文件在openstack dashboard界面生成，直接放到vk_related_files目录下。脚本内容如下：

    ```
    
    #!/usr/bin/env bash
    # To use an OpenStack cloud you need to authenticate against the Identity
    # service named keystone, which returns a **Token** and **Service Catalog**.
    # The catalog contains the endpoints for all services the user/tenant has
    # access to - such as Compute, Image Service, Identity, Object Storage, Block
    # Storage, and Networking (code-named nova, glance, keystone, swift,
    # cinder, and neutron).
    #
    # *NOTE*: Using the 3 *Identity API* does not necessarily mean any other
    # OpenStack API is version 3. For example, your cloud provider may implement
    # Image API v1.1, Block Storage API v2, and Compute API v2.0. OS_AUTH_URL is
    # only for the Identity API served through keystone.
    export OS_AUTH_URL=http://10.190.88.200:5000
    # With the addition of Keystone we have standardized on the term **project**
    # as the entity that owns the resources.
    export OS_PROJECT_ID=fe6700c8d5ed49c492dcd0247e58772a
    export OS_PROJECT_NAME="admin"
    export OS_USER_DOMAIN_NAME="Default"
    if [ -z "$OS_USER_DOMAIN_NAME" ]; then unset OS_USER_DOMAIN_NAME; fi
    export OS_PROJECT_DOMAIN_ID="default"
    if [ -z "$OS_PROJECT_DOMAIN_ID" ]; then unset OS_PROJECT_DOMAIN_ID; fi
    # unset v2.0 items in case set
    unset OS_TENANT_ID
    unset OS_TENANT_NAME
    # In addition to the owning entity (tenant), OpenStack stores the entity
    # performing the action as the **user**.
    export OS_USERNAME="admin"
    # With Keystone you pass the keystone password.
    echo "Please enter your OpenStack Password for project $OS_PROJECT_NAME as user $OS_USERNAME: "
    read -sr OS_PASSWORD_INPUT
    export OS_PASSWORD=$OS_PASSWORD_INPUT
    # If your configuration has multiple regions, we set that information here.
    # OS_REGION_NAME is optional and only valid in certain environments.
    export OS_REGION_NAME="RegionOne"
    # Don't leave a blank variable, unset it if it was empty
    if [ -z "$OS_REGION_NAME" ]; then unset OS_REGION_NAME; fi
    export OS_INTERFACE=public
    export OS_IDENTITY_API_VERSION=3
    
    ```

    ![image-20200701143516452](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701143516.png)

    (3) vk.sh是virtual-kubelet的启动脚本，脚本里需要source一下openstack的认证文件admin-openrc.sh，还需要导入启动vk所需要的环境变量，特别注意**database_ip**的ip地址填写的是openstack controller的IP地址，**database_password**字段填写的就是之前在openstack controller机器上的/etc/kolla/passwords.yml文件里的database_password字段的值（mysql的root用户密码），**KUBERNETES_SERVICE_HOST**填写的ip地址就是本机（K8S所在的机器）的IP地址，端口KUBERNETES_SERVICE_PORT默认为6443。最后一句话是启动virtual-kubelet，指定provider为openstack, 节点名称为virtual-kubelet66。

    vk.sh脚本内容如下：

    ```
    source ./admin-openrc.sh
    export OS_DOMAIN_NAME="Default"
    export database_username=root
    export database_password=iQvSyas0uD8aBaaVU3k19tWyyBhFFbumEtSDwcKy
    export database_ip=10.190.88.21
    export KUBERNETES_SERVICE_HOST=10.190.80.30
    export KUBERNETES_SERVICE_PORT=6443
    export KUBELET_PORT=10390
    export OPENSTACK_NETWORK_NAME=container-network
    export APISERVER_CERT_LOCATION=/etc/kubernetes/pki/apiserver.crt
    export APISERVER_KEY_LOCATION=/etc/kubernetes/pki/apiserver.key
    ./virtual-kubelet --provider openstack --nodename virtual-kubelet30
    ```

    ![image-20200701144734412](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701144734.png)

15. 启动vk: `sh vk.sh`，输入一下openstack的密码，就可以看到系统开始打log了。

    ![image-20200701144442702](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701144442.png)

16. 再开一个本机的会话，查看一下vk是否正常启动：`kubectl get nodes`，如果NAME显示为virtual-kubelet30就说明启动成功（NAME可以在启动脚本里修改）

    ![image-20200701144855568](https://raw.githubusercontent.com/MachineGunLin/markdown_pics/master/img/20200701144855.png)

