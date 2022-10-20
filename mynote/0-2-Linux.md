## 一 Linux命令

### 1 笔记常用汇总

> ```shell
> service network restart #重启网络服务
> systemctl status network #查看网络状态 start | stop | restart
> systemctl list-unit-files #查看所有服务的开机启动状态
> systemctl disable ${service_name} #关掉服务的自动启动 firewalld.service
> systemctl enable ${service_name}#开启服务的自动启动
> 
> less filename 	#和more类似，比其更强大；空格键|[pagedown](向下翻页) [pageup](向上翻页) /字串(向下搜索字串,n向下查找,N向上查找) ?字串(向上搜索字串,n向上查找,N向下查找) q(退出)
> echo [选项 -e(支持\控制的字符转换] # \\(输出\) \n(换行) \t(制表符)
> head [-n 行数 (头部指定行数内容)] filename #显示文件头部内容，默认10行，可指定行数
> tail [-n 行数 (尾部指定行数内容) -f(显示文件最近追加的内容，监听文件变化)] filename
> ln -s [原文件或目录] [软链接名]
> find xiyou/ -name "*.txt" #按文件名，根据名称查找xiyou/目录下的.txt结尾文件
> 
> chmod u-x,o+x chm.txt
> chmod 777 chm.txt
> chmod -R 777 test/ #整个文件夹里所有文件的权限均修改为777
> 
> tar -zcvf test.tar.gz test.txt #压缩文件成 test.tar.gz
> tar -zxvf 1.tar -C /opt #解压缩 1.tar 到目录/opt
> 
> 
> ps -aux | grep xxx  #查看系统进程
> ps -ef | grep xxx # 查看子父进程之间的关系
> kill -9 5102
> killall firefox
> netstat -anp | grep 进程号 #查看该进程网络信息
> netstat -lnp | grep 端口号 #查看网络端口号占用情况
> 
> rpm -qa #显示列表
> rpm -ivh firefox-45.0.1-1.el6.centos.x86_64.rpm  #安装
> rpm -e firefox #卸载
> yum -y install firefox
> ```
>
> **快捷键**
>
> ![image-20220919210319542](http://ybll.vip/md-imgs/202209192103766.png)



### 2 平时收集

> ```shell
> date +%s #获取时间戳  1650624172
> ```
>
> ```bash
> #实践 datanode 和 namenode 之间通信时使用(掉线时限参数的测试)
> jps #类似ps,只获取java应用程序的进程
> kill -9 [PID] #杀死进程
> hdfs --daemon start datanode #单独开启节点的 datanode
> ```
>
> ```bash
> systemctl restart network #重启网络服务
> systemctl status mysqld
> 
> jps -ml #查看jar程序更多信息(进程的全类名等)
> 
> netstat -anp | grep 10000
> 
> yarn app -list #查看yarn运行的AppcationMaster列表
> yarn app -kill [Application-Id]
> 
> hdfs fsck /  #从hdfs根目录下，检查block块是否有损坏
> #hdfs修改文件夹权限
> hadoop fs -chmod -R 777 / #根目录，777权限
> 
> #hive命令行执行，修复表
> msck repair table ods_user_info_inc;
> 
> #查看系统配置（open file ...)
> ulimit -a
> 
> #hdfs集群迁移
> distcp hdfs://hadoop102:8020/${hdfs对应的目录[文件]} hdfs://${其他集群}/${对应存放目录[文件]}
> 
> #查看内存分配情况
> jmap -heap ${进程号} #NameNode的进程号...
> ```
>
> + CentOS7 使用yum安装中文包
>
>  > 1. 查看系统版本  `cat /etc/redhat-release`
>   >
>   > ![image-20220423174816077](http://ybll.vip/md-imgs/202204231748143.png)
>   >
>   > 2. 安装语言包
>   >
>   > > ```shell
>   > > yum groupinstall chinese-support #本人亲自使用
>   > > yum groupinstall "fonts"	#博客上的其他
>   > > 
>   > > locale -a | grep zh  #查看是否有中文语言包
>   > > ```
>   > >
>   > > ![image-20220423175140073](http://ybll.vip/md-imgs/202204231751133.png)
>   >
>   > ***
>   >
>   > 3. 正式修改
>   >
>   > > ```shell
>   > > #查看当前系统语言
>   > > echo $LANG
>   > > 
>   > > ###方式一  #腾讯云CentOS7.6 使用该方式有效
>   > > cp /etc/locale.conf /etc/locale.conf_bak #先备份，然后修改该文件
>   > > 	LANG="zh_CN.UTF-8"  #将内容修改成这句
>   > > #重启若无效，进行第二步
>   > > vim /etc/profile.d/lang.sh #修改该文件，只需改一行
>   > > #修改内容，修改后可再重启试试
>   > > /zh*) LANG   #命令模式下，用该命令定位修改行
>   > > en_US.UTF-8  修改为 zh_CN.UTF-8
>   > > ```
>   > >
>   > > + ```shell
>   > >   if [ -n "$LANG" ];then
>   > >   case $LANG in
>   > >     *.utf8*|*.UTF-8*)
>   > >     if [ "$TERM" = "linux" ]; then
>   > >         if [ "$consoletype" = "vt" ]; then
>   > >             case $LANG in
>   > >                     ja*) LANG=en_US.UTF-8 ;;
>   > >                     ko*) LANG=en_US.UTF-8 ;;
>   > >                     si*) LANG=en_US.UTF-8 ;;
>   > >                     zh*) LANG=en_US.UTF-8 ;;　　//这里修改为zh_CN.UTF_8
>   > >                     ar*) LANG=en_US.UTF-8 ;;
>   > >                     fa*) LANG=en_US.UTF-8 ;;
>   > >                     he*) LANG=en_US.UTF-8 ;;
>   > >                     en_IN*) ;;
>   > >                     *_IN*) LANG=en_US.UTF-8 ;;
>   > >             esac
>   > >         fi
>   > >     fi
>   > >     ;;
>   > >   ```
>   > >
>   > > +++
>   > >
>   > > ```shell
>   > > ###方式二  #博客上出现，本人使用后无效果
>   > > localectl set-locale LANG=zh_CN.UTF-8
>   > > reboot 
>   > > ```
>
>  
>
>  
>
>  



***

***

***



## 二 Linux笔记

### Linux基础

1. Linux内核由 林纳斯·托瓦兹(Linus Torvald)编写 ; Linux是一套免费的类Unix操作系统，是一个多用户，多任务，支持多线程和多CPU的操作系统

2. Linux目录结构

   > 

### vim编辑器

![image-20220415164326745](http://ybll.vip/md-imgs/202204151643739.png)

> 一般模式：（vim打开后就直接进入该模式）
>
> `shift + 6  (^)` 回到行首；`shift + 4  ($)` 回到行尾
>
> `gg / (1 + G)` 光标到到第一行；`G` 光标到最后一行；`数字gg 或者 数字G` 光标跳到指定行
>
> `yy` 复制一行 ;  `y数字y` 复制多行  ；`dd` 删除当前行；`d数字d` 删除多行；
>
> `数字x` 往后剪切多个字母(包括当前)；`数字X` 往前剪切多个字母(不包括当前)
>
> `yw` 复制一个词；`数字yw` 复制多个词；`dw` 删除一个词 ；`数字dw` 删除多个词
>
> `p` 粘贴  ；`数字p` 粘贴多次
>
> `u` 撤销一次，回到上一步  ； `.`  撤销u
>
> ***
>
> ![img](http://ybll.vip/md-imgs/202204151713431.gif)
>
> ***
>
> 编辑模式
>
> `i` : 当前光标前  ；`I` : 当前行首
>
> `a` :  当前光标后；`A` : 当前行末
>
> `o` : 下一行；`O` : 上一行
>
> ***
>
> 指令模式
>
> `:w` 保存；`:q` 退出；`:!`  强制执行；`组合 :wq!` : 保存并强制退出
>
> `/word` : 查找单词；`n`:查找下一个；`N`:查找上一个
>
> `:set nu` : 显示行号；`set nonu` : 关闭行号
>
> `:%s/old/new/g` : 替换内容 ；`/g` 替换匹配的所有内容

***

### Linux命令

#### 1  网络相关

```sh
ifconfig #查看网络ip
ping 目的主机 #测试网络是否接通
vim /etc/sysconfig/network-scripts/ifconfig-ens33 #ip配置文件
	BOOTPROTO="static"  #ip配置方法 none/static/bootp/dhcp
	ONBOOT="yes" #系统启动时自动开启
	IPADDR=192.168.1.2 #ip地址
	GATEWAY=192.168.1.2 # 网关
	DNS1=192.168.1.2 #域名解析器
service network restart #重启网络服务
systemctl status network
#出问题解决：(可能的解决方式)
systemctl stop NetworkManager
systemctl disable NetworkManager

hostname #查看主机名
vim /etc/hostname #修改主机名
vim /etc/hosts #修改hosts映射文件;windows中host文件c:\Windows\System32\drivers\etc
```

#### 2 系统管理

```shell
#一个正在执行的程序或命令：进程(process); 启动后一直存在，常驻内存的进程：服务(service)
#服务管理的基本语法： service 服务名 start|stop|restart|status
ls /etc/init.d  #查看服务目录，netconsole|network
service network status|stop|start|restart

#chkconfig(CentOS 6版本)设置后台服务的开机自启配置
chkconfig 服务名 off|on|--list #关闭自启|打开自启|查看开机自启状态
chkconfig network on|off #开启|关闭network自动启动
chkconfig --level ${指定级别} network on|off #指定级别的自动启动

#systemctl(CentOS 7版本，重点掌握)
systemctl start|stop|restart|status ${服务名} #启停操作
#查看服务目录 /usr/lib/systemd/system
systemctl status firewalld #案例
#systemctl设置后台服务的自启配置
systemctl list-unit-files #查看所有服务的开机启动状态
systemctl disable ${service_name} #关掉服务的自动启动
systemctl enable ${service_name}#开启服务的自动启动
#案例
systemctl status firewall.d
systemctl stop firewall.d
systemctl enable firewalld.service
systemctl disable firewalld.service

#系统运行级别
#CentOS6:0-6,7种
#CentOS7:2种
	multi-user.target #多用户有网，无图形界面
	graphical.target #多用户有网，有图形界面
systemctl get-default #查看当前运行级别
systemctl set-default ${运行级别} #两种运行级别

#关机重启命令
sync #关机，将数据由内存同步到硬盘
halt #关机，关闭系统，不断电
poweroff #关机断电
reboot #重启，相当于shutdown -r now
shutdown # -H(halt) -r(reboot) new(立刻关机) ${时间}(等待多久后关机，单位是min)
shutdown -h 1 "This server will shutdown after 1 mins" #1分钟后关机，且显示在屏幕中
```

#### 3 常用基本命令

##### 3.1 帮助命令

> ```shell
> man ${命令或配置文件} #获取帮助信息
> #man 获取内置命令时会出现bash的帮助信息，或者加上-f
> man -f cd
> 
> help ${命令} #获得shell内置命令的帮助信息
> help cd
> #内置命令是内嵌在shell中的，常驻系统内存，如 cd exit history ...
> #ls 等不是内置命令
> ls --help # 可以用这种方式使用help查看帮助信息
> 
> #用type命令判断是哪种命令
> reset #彻底清屏
> 
> #常用快捷键
> ctrl + c #停止进程
> ctrl + l #清屏,等同于clear;
> ```



##### 3.2 文件目录类

> ```shell
> pwd
> ls -a(全不文件) -l(长数据串)
> 
> cd [参数 ~(回到家目录) -(回到下一次目录) ..(回到上一级) -P(跳转到实际物理路径,而非快捷方式路径)]
> 
> mkdir -p(创建多层目录)	rmdir (删除空目录)	touch (创建空文件)
> 
> cp [选项 -r(递归复制整个文件夹)] source dest #复制source到dest
> 
> rm [选项 -r(递归删除) -f(强制删除) -v(显示详细执行信息)] deleteFile
> 
> mv oldF newF #重命名或移动
> 
> cat [-n (显示行号)] filename #查看文件
> 
> 
> #分屏查看文件，空格键(向下翻页) 回车键(下一行) q(退出more) Ctrl+f(下一页) Ctrl+b(上一页) =(当前行号) :f(输出当前文件名和行号)
> more filename
> 
> #和more类似，比其更强大；空格键|[pagedown](向下翻页) [pageup](向上翻页) /字串(向下搜索字串,n向下查找,N向上查找) ?字串(向上搜索字串,n向上查找,N向下查找) q(退出)
> less filename
> 
> echo [选项 -e(支持\控制的字符转换]  \\(输出\) \n(换行) \t(制表符)
> 
> head [-n 行数 (头指定行数内容)] filename #显示文件头部内容，默认10行，可指定行数
> tail [-n 行数 (尾部指定行数内容) -f(显示文件最近追加的内容，监听文件变化)] filename
> 
> #输出重定向
> ls -l > ${文件} 	#列表内容写入文件中,覆盖写入 
> ls -l >> ${文件}  #列表内容写入文件中，追加写入
> #举例
> cat profile > test
> echo 'test' >> test
> 
> 
> #ln 软链接  列表属性是l(ll查看时的第一位)
> ln -s [原文件或目录] [软链接名]
> #删除时，软连接后不要加/
> rm -rf lntest/ #不删除软链接，会把原目录里的文件删除
> rm -rf lntest #单纯地删除软链接，不删除原目录文件
> 
> history #查看已经执行过的历史命令
> ```



##### 3.3 时间日期类

> ```shell
> date [option] [+format]
> option : -d<时间字符串> (显示指定的"时间字符串",表示时间，而非当前时间) -s<日期时间> (设置系统日期时间)
> +format : +日期时间格式 (指定显示时使用的日期时间格式)
> 
> date #查看系统当前时间
> #举例 +format
> date +%Y #显示当前年份(一定要有 + ) +%Y|%m|%d|%H|%M|%S
> date "+%Y-%m-%d %H:%M:%S"  # 2022-04-16 15:40:11
> #举例 option
> date -d '1 days ago' #显示前一天: Fri Apr 15 15:41:45 CST 2022
> date -s '2017-06-19 20:59:50' #设置系统当前时间
> 
> #cal查看日历
> cal [选项 (具体某一年 2022|2021|2020...)] #不加选项，这是显示本月的日历 
> cal #显示当月的日历
> cal 2022 #显示2022年的日历
> ```



##### 3.4 用户管理命令

###### 用户管理

> ```shell
> useradd ${用户名}
> useradd -g ${组名} ${用户名}
> passwd ${用户名}
> id ${用户名} #查看用户名是否存在
> cat /etc/passwd #查看创建了哪些用户
> su ${用户名} #切换用户，只能获得执行权限，不获取环境变量
> su - ${用户名} #切换用户，获取执行权限和环境变量
> userdel ${用户名} #删除用户，保留用户主目录
> userdel -r ${用户名} #删除用户和用户主目录，都删除
> whoami  #显示自身用户名称
> who am i #显示当前登录用户的用户名以及登录时间
> usermod -g ${用户组} ${用户名}  #修改用户所属的组
> ```
>
> ```bash
> #sudo设置普通用户具有root权限
> useradd atguigu ; passwd atguigu #新建用户并设置密码
> vim /etc/sudoers #修改配置文件
> 	root 		ALL=(ALL) 	ALL  #在该句后面添加
> 	${用户名} 	  ALL=(ALL)	  ALL #使用sudo时，需要密码
> 	${用户名}	  ALL=(ALL)   NOPASSWD:ALL #使用sudo时不用密码
> #修改后，登录其他账号，使用sudo命令，即可获得root权限进行操作
> ```
>
> +++

+++

###### 用户组管理命令

> + 每个用户都有一个用户组，系统可以对一个用户组中的所有用户进行集中管理
>+ 用户属于与他同名的用户组，这个用户组在创建用户时同时创建
> + 组的添加和修改和删除实际上是对 `/etc/group` 文件的更新
> 
> ```shell
>groupadd ${组名}
> groupdel ${组名}
> groupmod -n ${新组名} ${旧组名}
> cat /etc/group #查看所有的用户组
> cat /etc/sudoers #查看权限修改文件
> cat /etc/passwd #查看创建的用户
> ```

***

##### 3.5 文件权限类

![image-20220416170323047](http://ybll.vip/md-imgs/202204161703454.png)

`-(文件)	d(目录)		l(软链接) `

`rwxrwxrwx (文件属主权限，文件属主同组用户权限，其他用户)  -(表示没有 读|写|执行 其中一个的权限)` 

> 作用到文件:
>
> + r ：可读 ；w ：可修改，不能删除，需要对文件所属目录有w限权才能删除文件；x：可执行
>
> 作用到目录：
>
> + r ：可读取，查看；w：可修改，目录内创建|删除|重命名目录 ；x：可以直接进入目录
>
> ![image-20220416172137061](http://ybll.vip/md-imgs/202204161721293.png)
>
> 链接数说明：
>
> + 文件：链接数指 硬连接个数
> + 目录：子文件夹个数
>
> > ```shell
> > #chmod 改变权限
> > chmod [{ugoa} {+-=} {rwx}] 文件或目录
> > chmod [mode=421] 文件或目录
> > u:所有者 g:所有组 o:其他人 a:所有人(ugo的总和)
> > r=4 w=2 x=1 rwx=4+2+1=7
> > #案例
> > chmod u+x chm.txt
> > chmod g+x chm.txt
> > chmod u-x,o+x chm.txt
> > chmod 777 chm.txt
> > chmod -R 777 test/ #整个文件夹里所有文件的权限均修改为777
> > ```
> >
> > ```shell
> > #chown 改变所有者
> > chown [选项 -R(递归操作)][最终用户][文件或目录]
> > chown -R ybllcodes:ybll test/ #递归改变文件的 所有者:所有组
> > 
> > #chgrp 改变所有组
> > chgrp [最终用户组][文件或目录]
> > chgrp root test/
> > ```

***



##### 3.6 搜索查找类

###### find 查找文件或目录

> ```shell
>find [搜索范围][选项]
> -name <查询方式>  :指定文件名查找
> -user <用户名称>  :查找指定用户名的所有文件
> -size <文件大小>  :按照指定文件大小查找文件【单位:b(块，512字节) c(字节) w(字,2字节) k(千字节) M(兆字节) G(吉字节)】
> #案例
> find xiyou/ -name "*.txt" #按文件名，根据名称查找xiyou/目录下的.txt结尾文件
> find xiyou/ -user ybllcodes #按拥有者，查找xiyou/目录下，用户名为-user的文件
> find /home -size +204800 #按文件大小，在/home目录下查找大于200m的文件 (+num 大于; -num 小于; num 等于)
> ```
> 

+++

###### locate 快速定位文件路径

> ```shell
>locate 搜索文件
> ```
> 

###### grep 过滤查找及 `|` 管道符

> `|` : 表示将前一个命令的处理结果输出传递到后面的命令处理
>
> ```shell
> grep [选项 -n(显示匹配行及行号)] 查找内容 原文件
> ls | grep -n test
> grep -n t /root/profile
> ```

***

##### 3.7 压缩解压类

> gzip | gunzip 压缩
>
> ```shell
> gzip 文件 		#压缩文件成.gz格式(不能压缩目录)
> gunzip 文件.gz 	#解压缩 .gz文件
> #只能压缩文件，不能压缩目录
> #不保留原来的文件
> #同时压缩多个文件会产生多个压缩包
> gzip g.txt
> gunzip g.txt.gz
> ```
>
> zip | unzip
>
> ```shell
> zip [选项 -r(压缩目录)] 目标文件.zip 源文件 #源文件 -> 目标文件.zip
> unzip [选项 -d<指定目录> (指定解压后的文件存放的目录)] 文件.zip  #解压缩文件
> #zip压缩命令在windows/linux都通用，支持压缩目录，且保留源文件
> #案例
> zip test.zip text1.txt text2.txt  #将两个文件压缩成test.zip，保留源文件
> unzip test.zip #解压缩
> unzip test.zip -d /opt #指定解压缩到/opt 目录下
> ```
>
> tar 打包
>
> ![image-20220417115404384](http://ybll.vip/md-imgs/202204171227058.png)
>
> ```shell
> tar [选项] [目录/文件.tar.gz] [将要打包的内容]
> 	-z 打包同时压缩 (尽量加上，测试不加出错)
> 	-c 压缩,产生打包文件
> 	-v 显示详细信息
> 	-f 指定压缩后的文件名
> 	-x 解压缩
> 	-C 解压到指定目录(只能是已经存在的目录)	
> #案例
> #可压缩到指定目录的指定文件名
> tar -zcvf test.tar.gz test.txt
> tar -zcvf 1.tar test.txt
> tar -zcvf tar.gz test.txt #以上三种测试都正确
> #可解压缩到指定目录(只能是已经存在的目录)
> tar -zxvf tar.gz #解压所到当前位置
> tar -zxvf 1.tar -C /opt #解压缩 1.tar 到目录/opt
> 
> #总结
> #压缩时就使用   -zcvf  
> #解压缩就使用   -zxvf    -C <指定解压缩到目录>
> ```

+++

##### 3.8 磁盘分区类

###### du [ disk usage ] 

+ 查看文件和目录占用的磁盘空间(显示文件|目录占用空间大小)

> ```shell
>du [选项] 目录/文件
> 	-h 以易于阅读的格式自行显示，即在磁盘空间大小后加上单位
> 	-a 默认只显示目录，加上 -a 还包括显示文件
> 	-c 在最后显示总大小
> 	-s 只显示总大小 搭配 -h 不能搭配-a -c
> 	--max-depth=n 指定统计子目录的深度为第n层
> du -sh #查看当前用户住目录占用的磁盘空间总大小
> 
> ```
> 
> ![image-20220417122617376](http://ybll.vip/md-imgs/202204171227795.png)
>
> ***
>



###### df [ disk free ] 

+  查看磁盘空间使用情况

> ```shell
>df [选项]
> 	-h 以易于阅读的格式自行显示
> df -h
> ```
> 
> ![image-20220417122644289](http://ybll.vip/md-imgs/202204171227922.png)
> 
> ***
> 



###### lsblk 

+ 查看设备挂载情况

> ```shell
>lsblk -f
> 	-f 查看详细的设别挂载情况，显示文件系统信息
> ```
> 
> ![image-20220417123005695](http://ybll.vip/md-imgs/202204171230282.png)
> 
> ***
> 



###### fdisk 

+ 查看分区，(必须在 root 用户下才能使用)

> ```shell
>fdisk -l (查看磁盘分区详情)
> fdisk 硬盘设别名 (对新增硬盘进行分区操作)
> 
> ```
> 
> ![image-20220417123301838](http://ybll.vip/md-imgs/202204171233661.png)
> 
> +++
> 



###### mount | umount

+ 挂载|卸载

> ```shell
>
> ```
> 
> 

+++

##### 3.9 进程管理类

###### ps

> **ps (process status)查看当前系统进程状态**
>
> ```shell
> d.service #守护进程，可以看做系统服务(systemctl控制)
> sshd.service
> 
> -参数 #Unix风格
> 不加-直接写参数 #BSD风格
> 
> ps aux | grep xxx  #查看系统进程
> 	a 列出带有终端的所有进程
> 	u 面向用户友好的显示风格
> 	x 显示当前用户的所有进程，包括没有终端的进程
> 	-u 列出某个用户关联的所有进程
> 	-e 列出所有进程
> 	-f 显示完整格式的进程列表
> 	
> USER：该进程是由哪个用户产生的
> PID：进程的ID号
> %CPU：该进程占用CPU资源的百分比，占用越高，进程越耗费资源；
> %MEM：该进程占用物理内存的百分比，占用越高，进程越耗费资源；
> VSZ：该进程占用虚拟内存的大小，单位KB；
> RSS：该进程占用实际物理内存的大小，单位KB；
> TTY：该进程是在哪个终端中运行的。其中tty1-tty7代表本地控制台终端，tty1-tty6是本地的字符界面终端，		tty7是图形终端。pts/0-255代表虚拟终端。
> STAT：进程状态。常见的状态有：R：运行、S：睡眠、T：停止状态、s：包含子进程、+：位于后台
> START：该进程的启动时间
> TIME：该进程占用CPU的运算时间，注意不是系统时间
> COMMAND：产生此进程的linux命令
> ```
>
> ![image-20220419121834992](http://ybll.vip/md-imgs/202204191222064.png)
>
> 注意：`ps 命令 `也会产生一个进程，如图`ps -aux | grep mysql`
>
> ![image-20220419122215158](http://ybll.vip/md-imgs/202204191222230.png)
>
> +++
>
> 
>
> ```shell
> ps -ef | grep xxx # 查看子父进程之间的关系
> 
> UID：用户ID 
> PID：进程ID 
> PPID：父进程ID 
> C：CPU用于计算执行优先级的因子。数值越大，表明进程是CPU密集型运算，执行优先级会降低；数值越小，表明进程是I/O密集型运算，执行优先级会提高 
> STIME：进程启动的时间 
> TTY：完整的终端名称 
> TIME：CPU时间 
> CMD：启动进程所用的命令和参数
> ```
>
> ![image-20220419172802225](http://ybll.vip/md-imgs/202204191728336.png)
>
> +++
>
> ![image-20220419120918958](http://ybll.vip/md-imgs/202204191222084.png)
>
> +++
>
> 经验技巧
>
> > 需要查看父进程的ID，可以使用 -ef
> >
> > 否则使用 -aux

+++

###### kill

> kill 终止进程
>
> ```shell
> kill [选项] 进程号 #通过进程号杀死进程
> 	-9 表示强迫进程立即停止
> killall 进程名称 #通过进程名称杀死进程，也支持通配符(若非系统负载太大且变得太慢时使用)
> 
> kill -9 5102
> killall firefox
> ```
>

+++

###### pstree

> pstree 查看进程树
>
> ```shell
> pstree [选项]
> 	-p 显示进程的PID
> 	-u 显示进程所属的用户
> pstree -p
> pstree -u
> ```
> 

+++

###### top

> top 实时监控系统进程状态
>
> ```shell
> top [选项]
> 	-d 秒数  #指定top命令每隔几秒更新，默认是3s
> 	-i 使top不显示任何闲置或者僵死进程
> 	-p 通过指定监控进程ID来仅仅监控某个进程的状态
> #后续操作(键盘键入，都是大写，只有 q 小写，退出top)
> P:默认该项排序，cpu使用率排序
> M:以内存使用率来排序
>N:以PID排序
> q:退出
>
> #显示结果各字段解释:后续更新,或使用时再查
> ```
> 
> > 案例
> >
> > `top -d 1` ：间隔1s更新一次
> >
>> ![image-20220419211211645](http://ybll.vip/md-imgs/202204192112743.png)
> >
>> `top -i` ：不显示闲置或僵死进程![image-20220419211326433](http://ybll.vip/md-imgs/202204192113495.png)
> >
> > `top -p 2575` ：监控 2575端口
> >
> > ![image-20220419211428289](http://ybll.vip/md-imgs/202204192114351.png)
> 
> ***
> 

###### netstat

> netstat 显示网络状态和端口占用信息
>
> netstat | grep [端口号，服务名，进程号]
>
> ```shell
> netstat -anp | grep 进程号 #查看该进程网络信息
> netstat -lnp | grep 端口号 #查看网络端口号占用情况
> 	-a 显示所有正在监听和未监听的套接字(sockekt)
> 	-l 仅列出监听的服务状态
> 	-n 拒绝显示别名，能显示数字的全部转换成数字
> 	-p 表示显示哪个进程在调用
> ```
>
> 案例
>
> > `netstat -anp | grep sshd`：通过进程号查看sshd进程的网络信息
> >
> > ![image-20220419212420562](http://ybll.vip/md-imgs/202204192124628.png)
> >
> > `netstat -nltp | grep 22`: 查看22端口是否被占用（-lnp结果一样）
> >
> > ![image-20220419212523728](http://ybll.vip/md-imgs/202204192125789.png)

##### 3.10 crontab系统定时任务

> ```shell
> crontab [选项]
> 	-e 编辑crontab定时任务
> 	-l 查询crontab任务
> 	-r 删除当前用户所有的crontab任务
> ```
>
> 案例
>
> > `crontab -e`  #进入crontab编辑界面，然后在里面编辑
> >
> > `*/1 * * * * /bin/echo '11' >> /root/ybllcodes.txt` #每隔1分钟向ybllcodes.txt里面追加 11	
> >
> > ![image-20220419214426109](http://ybll.vip/md-imgs/202204192144205.png)

***

#### 4  软件包管理

##### 4.1 RPM

> RedHat软件包管理工具，类似windows中的setup.exe，是Linux系统里的打包安装工具
>
> ```shell
> rpm -qa #查询所有安装的rpm软件包
> ```
>
> ![image-20220419214735885](http://ybll.vip/md-imgs/202204192147949.png)
>
> ```shell
> #卸载
> rpm -e RPM软件包
> rpm -e --nodeps RPM软件包
> 	-e 卸载软件包
> 	--nodeps 卸载软件时，不检查依赖
> #案例
> rpm -e firefox
> ```
>
> ```shell
> #安装
> rpm -ivh RPM安装包
> 	-i Install,安装
> 	-v #--verbose,显示详细信息
> 	-h #--hash,显示进度条
> 	--nodeps #安装时不检查依赖
> #案例
> rpm -ivh firefox-45.0.1-1.el6.centos.x86_64.rpm
> 
> 
> #rpm安装后的目录：
> /etc/${software}/.. #conf目录
> /var/lib/${software}/.. #lib目录
> /usr/bin  /usr/sbin  /usr/local/bin  /usr/local/sbin   #bin/sbin目录
> /var/log #/log目录
> ```

##### 4.2 YUM

> 全称 Yellow dog Updater,Modified ,是shell前端软件包管理器，基于RPM包管理，能够从指定服务器自动下载RPM包并且安装，可以自动处理依赖关系，且一次安装所有依赖的软件包。
>
> ![image-20220419215531843](http://ybll.vip/md-imgs/202204192155916.png)
>
> ```shell
> yum [选项] [参数]
> 	#选项
> 	-y 对所以提问都回答yes
> 	#参数(不需要 - )
> 	install 安装rpm软件包
> 	update 更新rpm软件包
> 	check-update 检查是否有可用的更新rpm软件包
> 	remove 删除指定的rpm包
> 	list 显示软件包信息
> 	clean 清理yum过期缓存
> 	deplist 显示yum 软件包的所有依赖关系
> #案例
> yum -y install firefox
> ```
>
> 修改网络yum源(默认的yum源，需要连接apache网站，网速慢)
>
> ```shell
> #修改网络yum源为国内镜像的网站，如网易163,aliyun等
> yum install wget #安装wget，用来从指定url上下载文件
> #在 /etc/yum.repos.d/目录下，备份默认的repos文件
> cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
> #下载网易163或这是aliyun的repos文件
> wget http://mirrors.aliyum.com/repo/Centos-7.repo #阿里云
> #用下载的repos替换默认的repos文件
> mv CentOS7-Base-163.repo CentOS-Base.repo
> #清理旧缓存数据，缓存新数据
> yum clean all
> yum makecache
> #测试
> yum list | grep firefox
> yum -y install firefox
> ```



***

***

***



## 三 Shell

#### 0 注意点

> ```bash
> #两种替换方式
> `expr 2 \* 3`  或者  $(expr 2 \* 3)
> 
> #双引号"" 里面会识别$变量;单引号'' 里面不会识别$变量,直接当成字符串
> 
> #在双(( )) 中可以使用常规的数学符号:>、<、>=、<=等
> if((12<=20));then echo OK;fi #OK (条件判断中可以没有空格,if后也可以没有)
> if [ 12 -lt 20 ];then echo OK;fi #OK (if后必须有空格; []条件中也必须有空格)
> 
> ```
> 
>

![image-20220419222920282](http://ybll.vip/md-imgs/202204192229375.png)

```shell
cat /etc/shells
	/bin/sh
	/bin/bash
	/usr/bin/sh
	/usr/bin/bash
#bash 和 sh 的关系：sh -> bash
#默认的解析器是 /bin/bash
#脚本以 #!/bin/bash 开头
```

#### 1 Shell入门

```shell
#运行方式 sh | bash | ./ | .
bash test.sh
sh test.sh
#前两种不需要有执行权限，本质是bash帮着去执行
./test.sh  #需要文件有执行权限，本质是自己执行

. test.sh  #脚本前加 . | source ,使脚本在当前shell里执行，无需打开子shell
source test.sh # 和 . test.sh相似
#./test.sh 和 . test.sh 是不同的
#前三种是在当前shell中打开一个子shell来执行，执行完test.sh，子shell关闭，回到父shell
#后两种无需打开子shell,这就是为什么每次修改完/etc/profile后，需要再执行source
#二者区别在于环境变量的继承关系，子shell中设置的当前变量，父shell是不可见的
```

> 变量
>
> ```shell
> set #查看当前定义的变量(包括用户自定义变量)
> 
> #系统预定义变量
> $HOME | $PWD | $SHELL | $USER
> 使用 env | printenv 查看所有
> echo $HOME
> printenv HOME //打印全局变量
> 
> #用户自定义变量 [=前后不能有空格]
> #全局变量，当前shell，子shell中都能用；局部变量，只有当前shell能用
> export [变量名] #将局部变量提升为全局变量
> #更改为全局变量后，在子shell中修改变量，不会改变当前shell的值，退出后，值不会变化，即使子shell中使用了export，也不会更改当前shell的值
> 
> #只读变量
> readonly b=5
> unset b #撤销变量，不能撤销readonly变量
> #变量类型默认是字符串，不能进行数字计算
> #双引号"" 里面会识别$变量;单引号'' 里面不会识别$变量,直接当成字符串
> 
> #特殊变量
> echo $PATH
> $n #n = 1 2 3 4 ..9 {10} {11} 
> 	$0 当前脚本本身
> 	$1 第一个参数  #echo "hello $1"
> 	
> $# #获取参数个数，可用于循环
> 
> $* | $@ #获取所有参数
> $* #所有参数当做一个整体,使用双引号""包裹后，会当成一个整体，不使用则和 $@ 一样
> $@ #把每个参数区分对待，可以看做得到参数集合，可以用于循环遍历
> 
> $? #返回最后一次执行的命令的返回状态
> ```
>
> ***
>
> 运算符
>
> ```shell
> $(( )) | $[ ]  #表达式
> #在双(( )) 中可以使用常规的数学符号:>、<、>=、<=等
> expr 1 + 2 #输出3 ，字符中间必须有空格
> expr 1 \* 2 # *是特殊字符，需要转义
> 
> 
> #数字计算一般使用  $[]
> a=$[(2+3)*4]  # a的值为20
> 
> #计算两数和的脚本 add.sh
> #!/bin/bash
> sum=$[$1+$2]
> echo sum=$sum
> :wq
> chmod -x add.sh
> ./add.sh 25 89
> ```
>
> 条件判断
>
> ```shell
> #test condition
> test $a = hello #判断a是否等于hello
> #[ condition ] ([]前后必须有空格)
> [ $a = hello ] #=两边也必须有空格
> 
> #数值的比较
> #不能使用 < 和 >
> -eq (等于)  | -nq (不等于) | -lt (小于) | -gt (大于) | -le (小于等于) | -ge (大于等于)
> [ 2 -lt 8 ] 
> #文件权限进行判断
> -r (读的权限) | -w (写的权限) | -x (执行的权限)
> [ -r hello.sh ]  |  [ -w hello.sh]
> #文件类型的判断
> -e (文件是否存在) | -f (是否是文件) | -d (是否是目录)
> [ -e /root/test ]  |  [ -f add.sh ]
> 
> #多条件判断(用于后面流程控制)
> #类似 java中的【? c1:c2】 (若前一个判断为true,则执行&&后面，不执行||;若前一个判断为false,则不执行&&后面，直接执行||后面)
> [ 15 -lt 20 ] && echo ok || echo no # ok 
> [ 21 -lt 20 ] && echo ok || echo no # no
> ```
>
> read 读取控制台输入
>
> ```bash
> read [选项] [参数]
> 	#选项
> 	-p 指定读取时的提示符
> 	-t 指定读取时等待的时间(s),不加则一直等待
> 	#参数 指定读取时的变量名
> #案例
> read -t 10 -p "请输入你的芳名：" name
> echo $name
> ```

+++

#### 2 流程控制(重点)

> 分支判断
>
> > ```shell
> > ###单分支 ,if后必须有空格，必须有 then 还有 fi
> > #可以写成 [ condition ];then
> > if [ condition ]
> > then
> > 	程序
> > fi
> > 
> > #写在一行，可以用分号;分割
> > if [ $a -lt 25 ];then echo OK;fi #分号前后空格无所谓
> > 
> > #测试文件 if.sh
> > if [ "$1"x = 'atguigu'x ] #字符串拼接一个x,防止没有输入参数时报错
> > then
> > 	echo 'Welcome to atguigu'
> > fi
> > 
> > #if中条件的逻辑与和逻辑非
> > if [ $a -gt 18 ] && [ $a -lt 35 ]
> > if [ $a -gt 18 -a $a -lt 35 ] #等价于上一句，逻辑与 -a
> > if [ $a -gt 18 -o $a -lt 35 ] #逻辑或 -o ,大于18或者小于35
> > 
> > 
> > ###多分支
> > if [ condition ]
> > then
> > 	程序
> > elif [ condition ]
> > then
> > 	程序
> > else 
> > 	程序
> > fi
> > 
> > 
> > ###case语句
> > case $变量名 in  #case后必须有in
> > "值1")
> > 	代码
> > ;;				#两个分号;;表示退出，相当于break
> > "值2")
> > 	代码
> > ;;
> > *)				#默认模式，相当于default
> > 	代码
> > ;;
> > esac
> > #值1、值2...等不一定需要双引号""
> > ```
>
> +++
>
> for循环
>
> > ```shell
> > #基本语法
> > for (( 初始值;循环控制条件;变量变化 ))
> > do
> > 	程序
> > done
> > #案例 for.sh
> > #!/bin/bash
> > for(( i=1; i <= $1; i++ ))
> > do
> > 	sum=$[$sum+$i]
> > done
> > echo sum=$sum
> > #在双(( )) 中可以使用常规的数学符号:>、<、>=、<=等
> > 
> > #另一种用法
> > for 变量 in 值1 值2 值3...
> > do
> > 	程序
> > done
> > #案例
> > for i in yang bao ybllcode;do echo $i; done; #输出 yang bao ybllcodes (每个输出后都换行)
> > for i in {1..100}; do sum=$[$sum+$i];done;echo $sum #输出5050
> > for para in "$*" #只有一个元素
> > for para in $*  # 多个元素
> > for para in $@  # $@ 始终是多个元素
> > ```
>
> +++
>
> > while循环
> >
> > ```shell
> > while [ condition ]
> > do
> > 	程序
> > done
> > #案例
> > #!/bin/bash
> > a=1
> > while [ $a -le $1 ]
> > do
> > 	#sum2=$[$sum2 + $a ]
> > 	#a=$[ $a + 1 ]
> > 	let sum2+=a 
> > 	let a++    #可以用 let, 替代上面两行
> > done
> > echo sum2=$sum2
> > ```

+++

#### 3 函数

> 系统函数
>
> > ```shell
> > # basename [string/path] [suffix]
> > echo `basename /root/scripts/for.sh` 		# for.sh
> > echo `basename /root/scripts/for.sh .sh`  	# for
> > 
> > #dirname (切取最后一个 / 之前的字符串)
> > ...
> > ```
>
> 自定义函数
>
> > ```shell
> > [function] funname[()]
> > {
> > 	Action
> > 	[return int;]
> > }
> > #调用前必须声明
> > #函数返回值只能通过$?获取，return后跟的数值n(0-255)
> > ```

+++

#### 4 正则表达式
> ![image-20220919210406338](http://ybll.vip/md-imgs/202209192104427.png)
>
> ```shell
> ^ #匹配开头
> ^a   ^abc
> 
> $ #匹配结尾
> a$   abc$
> 
> ^$ #匹配空行
> cat daily_archive.sh | grep -n ^$ #匹配空行，并显示行号(-n)
> 
> . #匹配任意一个字符
> cat daily_archive.sh | grep r..t
> 
> * #不单独使用，和上一个字符连用，表示匹配该字符 0次 或者 多次
> cat daily_archive.sh | grep ro*t  # rt/rot/root/rooot/...  (中间不能出现其他字符)
> 
> .* #任意字符出现任意次
> cat daily_archive.sh | grep ^a.*bash$ #匹配a开头，bash结尾的字符，中间可以出现任意字符
> 
> [] #表示字符区间
> [6,8] #匹配6或8
> [0-9] #匹配数字
> [0-9]* #匹配任意多位数字
> [a-z] #匹配a-z
> [a-c,1-5] #匹配a-c,或者1-5
> 
> \ #转义
> grep daily_archive.sh | grep '\$' #匹配$,必须加单引号；$ \$ "\$" 这些都错,执行后都是出现全部
> 
> #案例:正则匹配手机号 11位
> grep -E ^1[345789][0-9]{9}$  #加上-E，支持扩展匹配正则表达式，默认不支持{}
> ```

+++

#### 5 文本处理工具

> cut
>
> > ```shell
> > cut [选项参数] filename
> > 	-f 指定列号
> > 	-d 指定分隔符，默认是 '\t'
> > 	-c 按字符进行切割，后加n表示取第几列，从1开始，没有0
> > cut -d " " -f 1 cut.txt #按空格分割，取第1列
> > 
> > #结果为ip地址;grep netmask :获取ip地址的那一行;cut:切割，获取第10列
> > ifconfig | grep netmask | cut -d " " -f 10
> > ```
>
> awk(内容多，难点)
>
> > ```shell
> > awk [选项参数] '/pattern/{action}' filename
> > 	#选项参数
> > 	-F 指定分割字符，默认是空格 " "
> > 	-v 赋值一个变量，可以在action中使用
> > 	#pattern写在//之中;action写在{}之中;可以加多个p/a
> > 	写法：'/p1/{a1} /p2/{a2} /p3/{a3}'
> > #pattern ：正则表达式，按行进行匹配
> > #action : 对匹配的行进行操作，内置有函数和变量，和shell函数类似
> > 
> > #案例1 pass文件中以root开头的所有行，并输出该行第7列
> > cp /etc/passwd /root/scripts/pass #文件准备
> > awk -F : '/^root/{print $7}' pass
> > cat pass | grep ^root | cut -d ":" -f 7
> > 
> > #案例2 pass文件以root开头的所有行，并且输出第一列和第七列
> > awk -F : '/^root/{print $1","$7}' pass
> > #cut不好实现-逗号','分割
> > 
> > #案例3 只显示pass中的第1列和第7列，以逗号分割，在所有行前加上列名 user,shell ; 且最后一行添加"end of file"
> > awk -F : 'BEGIN{print "user,shell"} {print $1","$7} END{print "end of file"}' pass
> > 
> > #案例4 将pass文件中用户id($3) 提取出来，且数值加1
> > awk -v i=1 -F : '{print $3+i}' pass
> > 
> > #案例5 awk内置变量--BEGIN()中使用无效，END()中可以使用
> > 	FILENAME : 文件名
> > 	NR : 当前读取的行号(匹配后的行实际在文件中的行数，不是读取的第几行)
> > 	NF : 当前记录的域的个数(当前读取行中，列的个数)
> > #统计pass文件名，每行的行号，每行的列数
> > awk -F : '{print "行号: "NR", col:"NF} END{print "filename: "FILENAME}' /root/scripts/pass
> > 
> > awk -F : '/^a/{print "a开头的行号:"NR", col :"NF } END{print "文件名 "FILENAME}' /root/scripts/pass   #a开头的行
> > 
> > #案例6 查询ifconfg命令输出结果中的空行所在行号
> > ifconfig | awk '/^$/{print NR}'
> > 
> > #案例7 切割ip (ifconfig中)
> > ifconfig | awk '/netmask/{print $2}'  #awk会省略前置空格，直接从非空格开始计数1
> > ifconfig | grep netmask | cut -d " " -f 10 #不省略前置空格
> > ```
> >
> > 案例1,2
> >
> > ![image-20220423124531619](http://ybll.vip/md-imgs/202204231245689.png)
> >
> > 案例3
> >
> > ![image-20220423124824858](http://ybll.vip/md-imgs/202204231248982.png)
> >
> > 案例5
> >
> > ![image-20220423131247113](http://ybll.vip/md-imgs/202204231312216.png)
> >
> > 案例7
> >
> > ![image-20220423131830977](http://ybll.vip/md-imgs/202204231318057.png)

#### 6 综合案例

> 归档文件
>
> ```shell
> #!/bin/bash
> #判断输入个数是否为1
> if [ $# -ne 1 ]
> then
> 	echo "只能有一个参数，作为归档的目录"
> 	exit
> fi
> 
> #从参数中获取目录名称
> if [ -d $1 ] 
> then
> 	echo 
> else
> 	echo "目录不存在！"
> 	exit
> fi
> 
> DIR_NAME=$(basename $1)
> DIR_PATH=$(cd $(dirname $1);pwd)
> 
> #获取当前日期
> DATE=$(date +%y%m%d)
> 
> #定义生成的归档文件名称
> FILE=archive_${DIR_NAME}_${DATE}.tar.gz
> DEST=/root/archive/$FILE
> #开始归档
> echo "开始归档"
> echo
> tar -zcvf $DEST $DIR_PATH/$DIR_NAME
> 
> if [ $? -eq 0 ]
> then
> 	echo
> 	echo
> 	echo "归档成功"
> 	echo "归档文件为: ${DEST}"
> 	echo 
> else
> 	echo "归档出现问题"
> fi
> exit
> ```
>
> 发送消息 ( linux自带的 mesg 和 write 工具，不同用户之间发送消息)
>
> > 