Linux-CentOS 常用命令
----------

网络部分
----------
1.  **配置静态IP**  
	CentOS 6  
	> 
		xxddddd  
		cccccc

	CentOS 7  
	> 
	$ sudo vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
	>> 
		TYPE=Ethernet  
		NAME=.....
		#追加或修改下列项目
		BOOTPROTO=static  
		IPADDR=10.0.2.100  
		NETMASK=255.255.255.0  
		GATEWAY=10.0.2.1
		DNS1=114.114.114.114
		DNS2=8.8.8.8  
	> 
	> $ sudo systemctl restart network  
2. **防火墙**  
	CentOS 6
	> 
		xxx   
		ddd  
# 查看防火墙状态
service iptables status
 
# 停止防火墙
service iptables stop
 
# 启动防火墙
service iptables start
 
# 重启防火墙
service iptables restart
 
# 永久关闭防火墙
chkconfig iptables off
 
# 永久关闭后重启
chkconfig iptables on

	CentOS 7  
	> 
	防火墙服务控制  
	>>
		$ systemctl status firewalld				# 查看服务状态  
		$ sudo systemctl start firewalld			# 启动  
		$ sudo systemctl stop firewalld				# 停止  
		$ sudo systemctl restart firewalld			# 重新启动  
	>
	防火墙规则控制
	>>
		$ firewall-cmd --list-all									# 查询当前防火墙规则  
		$ firewall-cmd --query-port=8080/tcp						# 查询端口是否开放  
		$ sudo firewall-cmd --permanent --add-port=80/tcp			# 开放80端口  
		$ sudo firewall-cmd --permanent --remove-port=8080/tcp		# 移除端口  
		$ sudo firewall-cmd --reload								# 重启防火墙(修改配置后要重启防火墙)  
		# 参数解释
		1. firwall-cmd	: 是Linux提供的操作firewall的一个工具；
		2. --permanent	: 表示设置为持久；
		3. --add-port	: 标识添加的端口；
3. 