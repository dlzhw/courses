CDH生产环境安装
====

CDH 与 Cloudera Manager版本问题
-----
Cloudera 使用如下命名方式：major.minor.maintenace.如果一个集群运行Cloudera Manager 5.14.0 版本，则主版本是5,次版本是15，修订版本是0.

Cloudera Manger的次版本必须大于等于CDH的次版本，否则Cloudera Manager 将不支持CDH高版本中的新特性。


Cloudera Manager 资源需求
-------
* 磁盘空间
	1. Cloudera Manager Server
	
		*  /var   5GB  
		*  /usr 500MB  
		*  对于parcels,需要的空间取决于你下载的parcels数量
	2. Cloudera Management Service  
	 主要的主机监控和服务监控数据库安装在/var目录下，确保至少20GB可用。
	3. Agents 
		在Agent主机，每一个parcel解压后需要大约3倍的磁盘空间。默认解压后的存储目录为/opt/cloudera/parcels。

* 内存  
	推荐 4G
* Python  
	Cloudera Manager 需要Python 2.4+版本，但是不兼容Python .0+。Hue 需要Python 2.6或2.7。
* Perl  
	Cloudera Manager 需要 perl.
* python-psycopg2包  
	Cloudera Manager 5.8及更高版本依赖python-psycopg2包。这个包没有在SLES 11和SLES 12仓库。需要在运行Cloudera Manager Agent的主机上手动安装。  
	如果Cloudera Manager Server 和 Agent 运行在同一台主机上，首先安装 Cloudera Manager Server ,然后安装 python-psycopg2包，最后安装  Cloudera Manager Agent。  
	python-psyconpg2下载地址： [python-psyconpg2](https://software.opensuse.org/package/python-psycopg2) 
* iproute 包  
	Cloudera Manager 5.12 及更高版本需要iproute包。  
	所有运行 Cloudera Manager Agent的主机都要安装这个包，不同操作系统和版本如下：
	CentOS/RHEL 6.x ---> iproute-2.6  
	CentOS/RHEL 7.x ---> iproute-3.10 

CDH和Cloudera Manager网络和安全需求
--------
* 网络协议支持（仅IP4支持）
* 主机的解析
/ect/hosts文件必须：  
	- 包含集群中所有主机的IP地址，且内容要一致。
	- 不要包含大写的主机名
	- 不要包含重复的IP地址
集群中的主机不能使用别名，不管是在/etc/hosts还是DNS中配置。  
	>正确的/etc/hosts文件应该象下面这样：
	>
		127.0.0.1	localhost.localdomain localhost  
		192.168.1.1	cluster-01.example.com cluster-01  
		192.168.1.2 cluster-02.example.com cluster-02  
		192.168.1.3 cluster-03.example.com cluster-03 
* Cloudera Manager 服务器到集群中主机的无密码远程登录 
* 关闭SELinux
* 关闭防火墙
* Cloudera Manager 和CDH使用多个账号和组来执行它们的任务，这些账号和组是依据所选择的组件来创建的。不要删除和修改这些账号及它们的权限。

CDH和Cloudera Manager数据库支持
--------
Cloudera Manager 和 CDH自带了内嵌的PostgreSQL数据库以简化在非产品环境下的安装部署。在产品环境下，必须配置集群使用外部数据库。大部分情况下，Cloudera 支持各Linux发行版自带的MariaDB,MySQL和PostgreSQL版本。  
数据库安装完成后，要更新最新补丁。  
提示：
* 使用UTF8字符编码
* 对于MySQL 5.6 和 5.7 ，必须安装  MySQL-shared-compat 或 MySQL-shared 包，这是 Cloudera Manager Agent 包安装所需要的。
* 不支持 MySQL 基于全局事务的副本。
*  
JDK 版本
--------
1. 仅支持 Oracle 的64位JDK. JDK 7 在所有Cloudera Manager 5 和CDH 5版本都支持。C5.3及更高版本支持JDK8.
2. 安装Java Cryptography Extension(JCE) 无限强度支持  
 * JDK7之前需要下载并更新JCE.  
 * JDK 1.8.0_151修改java.security文件来启用无限强度支持
 * JDK 1.8.0.161及以后版本已默认启用，无须修改。


版本和下载
----
* 组件版本  
	完整的组件列表，查看选择的发行版本目录下的manifest.json文件。  
	[发行版本地址](https://archive.cloudera.com/cdh5/parcels/)
* Cloudera Manager 5.15.1  
	文件下载地址 [打开](https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.15.1/ "文件下载地址")  
	repo 文件 [下载](https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo "Cloudera Manager Centos 7 yum 仓库文件")  
	tarball [下载](https://archive.cloudera.com/cm5/cm/5/cloudera-manager-centos7-cm5.15.1_x86_64.tar.gz "Cloudera Manager Centos 7 tar包")  

* CDH 5.15.1  
文件下载地址 [打开](https://archive.cloudera.com/cdh5/redhat/7/x86_64/cdh/5.15.1/ "文件下载地址")  
	repo 文件 [下载](https://archive.cloudera.com/cdh5/redhat/7/x86_64/cdh/cloudera-cdh5.repo "CDH Centos 7 yum 仓库文件")  
	




环境准备
-------
1. 安装virtual box
2. 安装centos 7
3. 创建集群安装及管理用户
	>创建用户  
	>$ sudo useradd -m cloudera  
	>设置密码  
	>$ sudo passwd cloudera  
	>添加sudo权限  
	>$ sudo nano /etc/sudoers  
	>root    ALL=(ALL)	ALL  (找到这行)  
	>cloudera        ALL=(ALL)	ALL  
4. 安装常用软件
	>$sudo yum install wget -y  
	>$sudo yum install nano -y
5. 添加 yum 阿里源
	>$ cd /etc/yum.repos.d/  
	>$ sudo mv CentOS-Base.repo CentOS-Base.repo.backup  
	>$ sudo wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo  
	>$ sudo yum clean all && yum makecache 
6. 配置IP
	>$ sudo nano /etc/sysconfig/network-scripts/ifcfg-eno16777736  
	>TYPE=Ethernet  
	>BOOTPROTO=static  
	>NAME=eno16777736  
	>DEVICE=eno16777736  
	>ONBOOT=yes  
	>IPADDR=192.168.37.90  
	>NETMASK=255.255.255.0  
	>GATEWAY=192.168.37.2  
	>DNS1=192.168.37.2  
7. 配置主机名
	>查看主机名  
	>hostname  
	>设置主机名  
	>$sudo hostnamectl set-hostname cloudera-01.zhw.com
8. 配置DNS解析
	>$ sudo nano /etc/hosts  
	>127.0.0.1       localhost.localdomain localhost  
	>192.168.37.90   cloudera-01.zhw.com   cloudera-01  
	>192.168.37.92   cloudera-02.zhw.com   cloudera-02  
	>192.168.37.93   cloudera-03.zhw.com   cloudera-03  
	>192.168.37.94   cloudera-04.zhw.com   cloudera-04  
9. 关闭SELinux  
	>永久生效，需要重启  
	>$ sudo nano /etc/selinux/config  
	> SELINUX=disabled
	> 
	> 即时生效，下次启动自动恢复  
	> sudo setenforce 0    

10. 关闭防火墙
	>永久生效   
	>开启： sudo systemctl enable firewalld  
	>关闭： sudo systemctl disable firewalld  
	>即时生效  
	>开启： sudo service firewalld start  
	>关闭： sudo service firewalld stop  
11. 配置免密登录  
	>Master 节点（cloudera-01）  
	>>  $ ssh-keygen -t rsa   
	在~/.ssh目录下生成私钥文件id\_rsa和公钥文件id\_rsa.pub.  
	复制公钥文件到所有的Agent节点  
	$ chmod 700 ~/.ssh  
	$ scp .ssh/id\_rsa.pub cloudera@cloudera-02:~/id\_rsa\_from\_cloudera-01.pub  
	$ scp .ssh/id\_rsa.pub cloudera@cloudera-03:~/id\_rsa\_from\_cloudera-01.pub  
	$ scp .ssh/id\_rsa.pub cloudera@cloudera-04:~/id\_rsa\_from\_cloudera-01.pub  
	>
	>Agent  节点  
	>>cloudera帐号登录Agent节点  
	$ mkdir ~/.ssh  
	$ cd ~/.ssh  
	$ cat ../id\_rsa\_from\_cloudera-01.pub >> authorized\_keys  
	$ chmod 700 ~/.ssh  
	$ chmod 600 ~/.ssh/authorized\_keys  
	>
	>测试免密登录
	>>在Master节点上运行以下命令  
	$ ssh cloudera@cloudera-02  
	如果能登录则表示配置成功。
12. 

安装步骤 
-------- 
>[官方文档](https://www.cloudera.com/documentation.html)   
>以下操作都以 cloudera 帐号登录  

1. 配置Cloudera Manager 仓库
	>1. 下载cloudera-manager 仓库  
	>$ sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d/  
	>2. 导入仓库签名GPG key
	>sudo rpm --import https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera  
2. 安装JDK
	>JDK必须是Oracle 且 64位的。在所有节点上安装同一版本。  
	>下载[JDK](http://download.oracle.com/otn-pub/java/jdk/8u181-b13/96a7b8442fe848ef90c96a2fad6ed6d1/jdk-8u181-linux-x64.tar.gz?AuthParam=1539445189_898f44f1886bf775eeba84a626ed9f4d)  
	>必须解压到/usr/java/目录  
	>$ sudo mkdir /usr/java  
	>$sudo tar vxf ~/jdk-8u181-linux-x64.tar.gz -C /usr/java/  
3. 安装Cloudera Manager Server
	>1. 下载 [Cloudera Manager Daemons RPM 包](http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5/RPMS/x86_64/cloudera-manager-daemons-5.15.1-1.cm5151.p0.3.el7.x86_64.rpm)  
	>2. 下载 [Cloudera Manager Server RPM 包](http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5/RPMS/x86_64/cloudera-manager-server-5.15.1-1.cm5151.p0.3.el7.x86_64.rpm)  
	>3. 下载[Cloudera Manager Server DB RPM 包](http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5/RPMS/x86_64/cloudera-manager-server-db-2-5.15.1-1.cm5151.p0.3.el7.x86_64.rpm)  
	>4. sudo yum --nogpgcheck localinstall cloudera-manager-daemons-*.rpm  
	>5. sudo yum --nogpgcheck localinstall cloudera-manager-server-*.rpm  
	>
4. 安装数据库
	>1. 安装 MySQL Server  
	>$ wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm  
	>$ sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm  
	>$ sudo yum update  
	>$ sudo yum install mysql-server  
	>$ sudo systemctl start mysqld  
	>$ systemctl status mysqld  
	>2. 配置和启动 MySQL Server   
		 1. 停止 MySql Server    
		 	$ sudo systemctl stop mysqld
		 2. 将 /var/lib/mysql/ib_logfile0 和 /var/lib/mysql/ib_logfile1 移除 /var/lib/mysql目录到一个备份目录。  
		 	$ sudo mkdir /var/mysqlbackup
		 3. 找到 mysql 配置文件的位置（默认为 /etc/my.cnf)。
		 4. 更新配置文件，确保满足下列需求：  
			 1. 为了阻止死锁，修改隔离级别为 READ-COMMITTED  
			 2. 配置InnoDB引擎，否则 Cloudera Manager 不能启动。可以通过下列命令检查当前表使用哪种引擎。
			 3. mysql> show table status;
			 4. 大多数情况下 MySQL 安装时默认设置为比较保守的缓冲大小和内存使用。Cloudera 推荐设置 innodb_flush_method 属性为 O_DIRECT.
			 5. 根据集群的大小设置 max_connections属性。    
			 下面是一个推荐的配置
			 6.  			
					[mysqld]
					datadir=/var/lib/mysql
					socket=/var/lib/mysql/mysql.sock
					transaction-isolation = READ-COMMITTED
					# Disabling symbolic-links is recommended to prevent assorted security risks;
					# to do so, uncomment this line:
					symbolic-links = 0
					#
					key_buffer_size = 32M
					max_allowed_packet = 32M
					thread_stack = 256K
					thread_cache_size = 64
					query_cache_limit = 8M
					query_cache_size = 64M
					query_cache_type = 1
					#
					max_connections = 550
					#expire_logs_days = 10
					#max_binlog_size = 100M
					#log_bin should be on a disk with enough free space.
					#Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
					#system and chown the specified folder to the mysql user.
					log_bin=/var/lib/mysql/mysql_binary_log
					#In later versions of MySQL, if you enable the binary log and do not set
					#a server_id, MySQL will not start. The server_id must be unique within
					#the replicating group.
					server_id=1
					#
					binlog_format = mixed
					#
					read_buffer_size = 2M
					read_rnd_buffer_size = 16M
					sort_buffer_size = 8M
					join_buffer_size = 8M
					# InnoDB settings
					innodb_file_per_table = 1
					innodb_flush_log_at_trx_commit  = 2
					innodb_log_buffer_size = 64M
					innodb_buffer_pool_size = 4G
					innodb_thread_concurrency = 8
					innodb_flush_method = O_DIRECT
					innodb_log_file_size = 512M
					#
					[mysqld_safe]
					log-error=/var/log/mysqld.log
					pid-file=/var/run/mysqld/mysqld.pid
					#
					sql_mode=STRICT_ALL_TABLES  
		5. 确保 MySql Server 自启动  
			$ sudo systemctl enable mysqld  
		6. 启动 MySql Server  
			$ sudo systemctl start mysqld  
		7. 运行 /usr/bin/mysql_secure_installation 设置 MySql root 密码和其它安全相关的属性。初始的 root 密码为空。  
			$ sudo /usr/bin/mysql\_secure\_installation  
	>3. 安装 MySql JDBC Driver  
		1. 不要使用yum install 命令安装 MySql 驱动包。  
		2. 根据安装的组件所在的主机上都要安装 MySql 驱动包。
		3. 下载 [MySql 驱动](http://www.mysql.com/downloads/connector/j/5.1.html)  
		4. $ tar zxvf mysql-connector-java-5.1.46.tar.gz   
		5. 复制 JDBC 驱动到/usr/share/java 目录。  
			$ sudo mkdir -p /usr/share/java/  
			$ cd mysql-connector-java-5.1.46  
			$ sudo cp mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar  
	>4. 为 Cloudera 软件创建数据库  
		* Cloudera Manager Server  
		* Cloudera Manager Service 角色  
		* 每一个 Hive metastore  
		* Sentry Server   
		* Cloudera Navigator Audit Server  
		* Cloudera Navigater Metadata Server  
	    这些数据库必须配置支持Mysql utf8 字符集。
		记录你填写的数据库名，用户名及密码，Cloudera Manager 安装向导需要这些信息连接到数据库。  
		#
		1. 以root用户登录MySql数据库  
			$ mysql -u root -p
		2. 为所使用的每一个服务创建数据库  
			mysql> CREATE DATABASE &lt;database> DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8\_general\_ci;  
			mysql> GRANT ALL ON &lt;database>.* TO '&lt;user>'@'%' IDENTIFIED BY '&lt;password>';  
			下面是Cloudera 软件默认的服务及数据库信息：  
			|服务------------------------------------------------- |数据库----------|	用户名 ------------ |  
			|--------------------------------------------------------|-------------------|-----------------------|  
			|Cloudera Manager Server-----------------------|scm -------------|	scm----------------- |  
			|Activity Monitor-------------------------------------|	amon-----------|	amon---------------|  
			|Reports Manager----------------------------------|	rman------------|	rman----------------|  
			|Hue	---------------------------------------------------|hue-------------	|hue------------------|  
			|Hive Metastore Server----------------------------|	metastore------|	hive-----------------|  
			|Sentry Server	--------------------------------------|sentry	-----------|sentry---------------|  
			|Cloudera Navigator Audit Server---------------|	nav	-------------|nav------------------|  
			|Cloudera Navigator Metadata Server----------|	navms----------|	navms--------------|  
			|Oozie	------------------------------------------------|oozie	------------|oozie-----------------|  
				mysql>  
				CREATE DATABASE scm  DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;  
				GRANT ALL ON scm  .* TO 'cloudera'@'%' IDENTIFIED BY 'cloudera';  
				CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;  
				GRANT ALL ON amon .* TO 'cloudera'@'%' IDENTIFIED BY 'cloudera';  
				CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;  
				GRANT ALL ON rman .* TO 'cloudera'@'%' IDENTIFIED BY 'cloudera';  
				CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;  
				GRANT ALL ON hue .* TO 'cloudera'@'%' IDENTIFIED BY 'cloudera';  
				CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;  
				GRANT ALL ON metastore .* TO 'cloudera'@'%' IDENTIFIED BY 'cloudera';  
				CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;  
				GRANT ALL ON sentry .* TO 'cloudera'@'%' IDENTIFIED BY 'cloudera';  
				CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;  
				GRANT ALL ON nav .* TO 'cloudera'@'%' IDENTIFIED BY 'cloudera';  
				CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;  
				GRANT ALL ON navms .* TO 'cloudera'@'%' IDENTIFIED BY 'cloudera';  
				CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;  
				GRANT ALL ON oozie .* TO 'cloudera'@'%' IDENTIFIED BY 'cloudera';  
		3. 确认你创建了所有的库  
		mysql> SHOW DATABASES;  
		mysql> SHOW GRANTS FOR 'cloudera'@'%';

5. 配置Cloudera Manager 数据库
	1. Cloudera Manager Server 包含了脚本来创建数据库。 
	2. 脚本语法如下：
	3. 
6. 安装 CDH 及 其它软件
	1. $ sudo systemctl start cloudera-scm-server  
	2. sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log  
	3. http://<server_host>:7180
	4. 用户名，密码admin
7. 搭建集群
