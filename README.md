SaltStack 常用模块介绍 一、APIAPI的原理是通过调用master client模块，实例化一个LocalClient对象，再调用cmd()方法来实现例如：API实现test.ping#vim /root/test_ping.pyimport salt.clientclient = salt.client.LocalClient()ret = client.cmd('*','test.ping')print ret执行结果返回一个字典{'minion02': True, 'minion01': True}
二、常用模块介绍1.Archive模块功能：实现系统层面的压缩包调用，支持 gunzip、gzip、rar、tar、unzip等1.1)实例#gunzip解压install.log.gz文件#salt '*' archive.gunzip /root/install.log.gz#用gzip压缩install.log文件#salt '*' archive.gzip /root/install.log1.2)API调用#!/usr/bin/env python import salt.client client = salt.client.LocalClient() ret = client.cmd('*','archive.gzip',['/root/install.log'])
2.cmd模块功能：实现远程命令行调用执行（默认具备root权限）2.1)实例通过free -m查看内存运行情况## salt '*' cmd.run 'free -m'2.2)API调用#!/usr/bin/env python import salt.client client = salt.client.LocalClient() ret = client.cmd('*','cmd.run',['free -m'])
3.cp模块功能：实现远程文件、目录的复制，以及下载URL文件等操作3.1）实例将制定被控主机的/root/install.log文件复制到被控主机本地的/var/cache/salt/minion/localfiles/（salt cache目录）下#salt '*' cp.cache_local_file /root/install.log
将master主机的1.txt文件，复制到被控主机minion的/var/下注意：需要先查看/etc/salt/master配置文件下的 file_roots （源仓库）是在什么位置，默认是在/srv/salt下，如果/srv下没有salt目录，则自己新建，当然，也可以修改目录#salt '*' cp.get_file salt://1.txt /var/1.txt
将master主机的test目录复制到被控主机minion的/var/下注意：需要先查看/etc/salt/master配置文件下的 file_roots （源仓库）是在什么位置，默认是在/srv/salt下，如果/srv下没有salt目录，则自己新建，当然，也可以修改目录#salt '*' cp.get_dir salt://test/ var/
master上下载脚本到被控主机的/root/目录下#salt '*' cp.get_url http://mylinuxer.com/down/lamp.sh /root/lamp.sh3.2)API调用#!/usr/bin/env python import salt.client client = salt.client.LocalClient() ret = client.cmd('*','cp.get_file',['salt://2.txt','/root/2.txt'])
4.cron模块功能：实现被控主机的crontab操作4.1)实例查看指定被控主机、root用户的crontab 清单#salt '*' cron.raw_cron root为指定的被控主机、root用户添加/root/shell/monitor.sh任务#salt '*' cron.set_job root '*' '*' '*' '*' '*' /root/shell/monitor.sh删除指定的被控主机、root用户crontab的/root/shell/monitor.sh任务#salt '*' cron.rm_job root /root/shell/monitor.sh4.2)API调用#!/usr/bin/env python import salt.client client = salt.client.LocalClient() ret = client.cmd('*','cron.set_job',['root','*','*','*','*','*','/root/shell/monitor.sh']) print ret
5.dnsutil模块功能：实现被控主机通过DNS相关操作5.1）实例添加指定被控主机hosts的主机配置项#salt '*' dnsutil.hosts_append /etc/hosts 127.0.0.1 monitor.test.com,monitor2.test.com删除指定被控主机hosts的主机配置项#salt '*' dnsutil.hosts_remove /etc/hosts monitor.test.com5.2)API调用#!/usr/bin/env python import salt.client client = salt.client.LocalClient() ret = client.cmd('*','dnsutil.hosts_append',['/etc/hosts','127.0.0.1','monitor.test.com']) print ret
6.file模块功能：被控主机文件常见操作，包括文件读写、权限、查找、校验等。6.1)实例：修改所有被控主机/root/1.txt文件数属组、用户权限，等价于 chown www:www /root/1.txt#salt '*' file.chown /root/1.txt www www
复制所有被控主机本地 /root/1.txt文件到本地的/var/下#salt '*' file.copy /root/1.txt /var/1.txt
检查所有被控主机 /etc模块是否存在，存在则返回True,检查文件是否存在 使用file.file_exists方法#salt '*' file.directory_exists /etc
获取被控主机/etc/passwd的stats信息#salt '*' file.stats /etc/passwd
获取所有被控主机/etc/passwd的权限mode ,如 775 、644#salt '*' file.get_mode /etc/passwd
修改所有被控主机/root/1.txt的权限mode ,如 775、644#salt '*' file.set_mode /root/1.txt 777
在所控主机创建/opt/test目录#salt '*' file.mkdir /opt/test
将所有被控主机/etc/httpd/httpd.conf文件的LogLevel参数的warn值修改成info#salt '*' file.sed /etc/httpd/httpd.conf 'LogLevel warn' 'LogLevel info'
给所有被控主机的/root/1.txt文件追加内容 "hello world..."#salt '*' file.append /root/1.txt "hello world..."
删除所有被控主机的/root/1.txt文件#salt '*' file.remove /root/1.txt
6.2)API调用#!/usr/bin/env python import salt.client client = salt.client.LocalClient() ret = client.cmd('*','file.remove',['/root/2.txt'])
7.iptables模块功能：被控主机Iptables支持1）实例：为所有的被控主机，追加（-A）80端口的规则---------iptables.append#salt '*' iptables.append filter INPUT rule='-p tcp --dport 80 -j ACCEPT'为所有的被控主机，插入(-I) 3306端口的规则--------iptables.insert#salt '*' iptables.insert filter INPUT position=3 rule='-p tcp --dport 3306 -j ACCEPT'在所有被控主机，删除指定链编号为3（position=3）或这顶存在的规则---------iptables.delete# salt '*' iptables.delete filter INPUT position=3#salt '*' iptables.delete filter INPUT rule='-p tcp --dport 80 -j ACCEPT'2)API调用#!/usr/bin/env python import salt.client client = salt.client.LocalClient() ret = client.cmd('*','iptables.append',['filter','INPUT','rule=\'-p tcp --dport 80 -j ACCEPT\'']) print ret
8.network模块功能：返回被控主机网络信息1）实例：获取被控主机dig、ping、traceroute目录域名信息#salt 'minion01' network.dig www.baidu.com#salt 'minion01' network.ping www.baidu.com#salt 'minion01' network.traceroute www.baidu.com获取指定主机的MAC地址#salt '*' network.hwaddr eth0检测指定被控主机是否输入192.168.1.0/24网段，属于则返回True#salt '*' network.in_subnet 192.168.1.0/24获取指定主机的IP地址配置信息#salt '*' network.ip_addrs获取被控主机网卡配置信息#salt '*' network.interfaces获取被控主机子网信息#salt '*' network.subnets2）API调用#!/usr/bin/env python import salt.client client = salt.client.LocalClient() ret = client.cmd('*','network.ip_addrs') print ret
9.pkg包管理功能：被控主机程序包管理，如 yum 、apt-get等1）实例：为被控主机安装php包，如redhat的Yum环境，等价于 yum -y install php#salt '*' pkg.install php卸载被控主机的php包#salt '*' pkg.remove php升级被控主机的软件包#salt '*' pkg.upgrade2)API调用#!/usr/bin/env python import salt.client client = salt.client.LocalClient() ret = client.cmd('*','pkg.install',['php']) print ret
10.service服务模块功能：被控主机程序包管理服务1)实例：开启(enable)、禁用(disable)nginx开机启动服务#salt '*' service.enable nginx#salt '*' service.disable nginxnginx的start、stop、reload、status、reload操作#salt '*' service.reload nginx#salt '*' service.start nginx#salt '*' service.restart nginx#salt '*' service.stop nginx#salt '*' service.status nginx2)API调用#!/usr/bin/env python import salt.client client = salt.client.LocalClient() ret = client.cmd('*','service.start',['iptables']) print ret



Saltstack returner
默认情况下,发送给salt minion的命令执行结果将返回给salt master.
Saltstack Returner的接口允许将结果发送给任意系统
 1、安装 MySQL-python包#yum -y install MySQL-python mysql-server
2、新建数据库，表CREATE DATABASE `salt`      DEFAULT CHARACTER SET utf8      DEFAULT COLLATE utf8_general_ci;    USE `salt`;    --    -- Table structure for table `jids`    --    DROP TABLE IF EXISTS `jids`;    CREATE TABLE `jids` (      `jid` varchar(255) NOT NULL,      `load` mediumtext NOT NULL,      UNIQUE KEY `jid` (`jid`)    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;    --    -- Table structure for table `salt_returns`    --    DROP TABLE IF EXISTS `salt_returns`;    CREATE TABLE `salt_returns` (      `fun` varchar(50) NOT NULL,      `jid` varchar(255) NOT NULL,      `return` mediumtext NOT NULL,      `id` varchar(255) NOT NULL,      `success` varchar(10) NOT NULL,      `full_ret` mediumtext NOT NULL,      KEY `id` (`id`),      KEY `jid` (`jid`),      KEY `fun` (`fun`)    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
3、授权GRANT ALL ON salt.* to 'salt'@'%' identified by 'salt';
4、修改master 配置文件#vim /etc/salt/master 添加以下内容mysql.host: '127.0.0.1'mysql.user: 'salt'mysql.pass: 'salt'mysql.db: 'salt'mysql.port: 3306master_job_cache: mysql
#/etc/init.d/salt-master restart
5、master端执行测试# salt '*' test.ping
查看数据库已增加了内容
