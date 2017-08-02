---
title: path B 安装cloudera cdh
date: 2017-08-02 19:26:55
tags: 大数据 
---

#### 安装Cloudera Manager and CDH的几种方式
- PATH A： 通过Cloudera Manager自动安装 (不适合生产环境)
- PATH B： Installation Using Cloudera Manager Parcels or Packages
- PATH C： Manual Installation Using Cloudera Manager Tarballs

#### 服务器环境准备
- SSH无密码验证配置
  1. 任一台主机(A)执行
  ```
  ssh-keygen
  cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  chmod 600 ~/.ssh/authorized_keys
  ```
  <!--more-->
  2. 使用ssh-copy-id命令将公钥传送到远程主机(B)上
    ```
    ssh-copy-id root@B
    ```
    这样从A就可以无密码登录B
- 关闭selinux
- 配置hosts
- 配置hostname
- 关闭防火墙并设置开机不启动
  1. 重启后永久性生效：
开启：`chkconfig iptables on`
关闭：`chkconfig iptables off`
  2. 即时生效，重启后失效：
开启：`service iptables start`
关闭：`service iptables stop`
- 优化虚拟内存需求率
  1. 检查虚拟内存需求率 `cat /proc/sys/vm/swappiness`
  2. 临时修改 `sysctl vm.swappiness=10`
  3. 永久修改 `vi /etc/sysctl.conf`  `vm.swappiness = 10`
- 解决透明大页面问题
  1. 检查 `cat /sys/kernel/mm/transparent_hugepage/defrag`
  2. 临时修改 
  `echo never > /sys/kernel/mm/transparent_hugepage/defrag`
  `echo never > /sys/kernel/mm/transparent_hugepage/enabled`
  3. 永久修改 `vi /etc/rc.local` 加入上述语句


#### 数据库安装配置：
1. 安装MySQL数据库
2. mysql-connector
    `mkdir -p /usr/share/java/`
    `cd /usr/share/java/`
    `wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.41.tar.gz`
    `tar -zxvf mysql-connector-java-5.1.41.tar.gz`
    `cp mysql-connector-java-5.1.41/mysql-connector-java-5.1.41-bin.jar /usr/share/java/mysql-connector-java.jar`
3. 为Cloudera Manager Server初始化所需的数据库
    `grant all on *.* to 'temp'@'%' identified by 'temp' with grant option;`
    `/usr/share/cmf/schema/scm_prepare_database.sh mysql -h 192.168.100.222 -utemp -ptemp --scm-host 192.168.100.222 scm scm scm`
    `drop user 'temp'@'%';`
    https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_installing_configuring_dbs.html#concept_mff_xjm_hn
4. 创建hue,hive等其他服务所需的数据和账号
    ```
    grant all privileges on *.* to 'scm'@'%' identified by 'scm' WITH GRANT OPTION;
    GRANT ALL PRIVILEGES ON *.* TO 'scm'@'%' IDENTIFIED BY 'scm' WITH GRANT OPTION;
    GRANT ALL PRIVILEGES ON *.* TO 'scm'@'192.168.100.235' IDENTIFIED BY 'scm' WITH GRANT OPTION;

    GRANT ALL ON *.* to 'root'@'%' identified by '123456' WITH GRANT OPTION;

    mkdir -p /usr/share/java/
    wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.41.tar.gz
    tar -zxvf mysql-connector-java-5.1.41.tar.gz
    cp mysql-connector-java-5.1.41/mysql-connector-java-5.1.41.39-bin.jar /usr/share/java/mysql-connector-java.jar


    /usr/share/cmf/schema/scm_prepare_database.sh mysql -h 192.168.100.222 -utemp -ptemp --scm-host 192.168.100.222 scm scm scm

    create database amon DEFAULT CHARACTER SET utf8;
    grant all on amon.* TO 'amon'@'%' IDENTIFIED BY 'amon_password';
    grant all privileges on amon.* TO 'amon'@'192.168.100.222' IDENTIFIED BY 'amon_password';

    create database rman DEFAULT CHARACTER SET utf8;
    grant all on rman.* TO 'rman'@'%' IDENTIFIED BY 'rman_password';

    create database metastore DEFAULT CHARACTER SET utf8;
    grant all on metastore.* TO 'hive'@'%' IDENTIFIED BY 'hive_password';
    grant all on metastore.* TO 'hive'@'master.hadoop.dev.8dol.com' IDENTIFIED BY 'hive_password';

    create database sentry DEFAULT CHARACTER SET utf8;
    grant all on sentry.* TO 'sentry'@'%' IDENTIFIED BY 'sentry_password';

    create database nav DEFAULT CHARACTER SET utf8;
    grant all on nav.* TO 'nav'@'%' IDENTIFIED BY 'nav_password';

    create database navms DEFAULT CHARACTER SET utf8;
    grant all on navms.* TO 'navms'@'%' IDENTIFIED BY 'navms_password';

    create database oozie;
    grant all privileges on oozie.* to 'oozie'@'localhost' identified by 'oozie';
    grant all privileges on oozie.* to 'oozie'@'%' identified by 'oozie';
    grant all privileges on oozie.* TO 'oozie'@'192.168.100.222' IDENTIFIED BY 'oozie';

    create database hue;
    grant all privileges on hue.* to 'hue'@'localhost' identified by 'hue';
    grant all privileges on hue.* to 'hue'@'%' identified by 'hue';
    grant all privileges on hue.* TO 'hue'@'192.168.100.222' IDENTIFIED BY 'hue';
    ```
参考文档：
https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_mysql.html#cmig_topic_5_5_2

#### 手动通过Cloudera Manager安装CDH（PATH B）
1. 安装JDK
2. 安装数据库
3. 安装启动Cloudera Manager Server
4. 安装Cloudera Manager Agents（手动或通过Cloudera Manager Installation wizard）
5. 安装CDH and Managed Service software（手动或通过Cloudera Manager Installation wizard）
6. 创建、配置、启动CDH and Managed Services
参考文档：
https://www.cloudera.com/documentation/enterprise/latest/topics/installation_installation.html
https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_install_path_b.html#cmig_topic_6_6_1

### 错误解决
- 启动hdfs时，报错 Canary 测试无法在目录 /tmp/.cloudera_health_monitoring_canary_files 中创建文件。 
经过查看日志，发现 Name node is in safe mode. 
解决方法：sudo -uhdfs hdfs dfsadmin -safemode leave

参考文档：
http://www.jianshu.com/p/57179e03795f
http://cmdschool.blog.51cto.com/2420395/1775398
http://blog.csdn.net/wendingzhulu/article/details/52423580