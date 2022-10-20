1. 什么是数据仓库（数仓）？

   ​		数据仓库是一个为数据分析而设计的企业级数据管理系统;

2. 数据仓库模型：维度建模

   > 数据模型就是数据组织和存储方法，它强调从业务、数据存取和使用角度合理存储数据。只有将数据有序的组织和存储起来之后，数据才能得到高性能、低成本、高效率、高质量的使用。
   >
   > 高性能：良好的数据模型能够帮助我们快速查询所需要的数据。
   >
   > 低成本：良好的数据模型能减少重复计算，实现计算结果的复用，降低计算成本。
   >
   > 高效率：良好的数据模型能极大的改善用户使用数据的体验，提高使用数据的效率。
   >
   > 高质量：良好的数据模型能改善数据统计口径的混乱，减少计算错误的可能性。
   >

   

5. 数仓架构

![12](http://ybll.vip/md-imgs/202207241855422.png)



## 数仓项目采集

### 1-模拟数据

+ 创建用户行为日志模拟的脚本 `lg.sh`

```shell
#!/bin/bash
for host in hadoop102 hadoop103; do
    echo "========== $host =========="
    ssh $host "cd /opt/module/applog/; java -jar gmall2020-mock-log-2021-01-22.jar >/dev/null 2>&1 &"
done
```

```txt
在 hadoop102 hadoop103 上会产生日志数据，
目录：/opt/module/applog/log/
log日志文件格式：app.yyyy-mm-dd.log
随后通过flume 的 tailDir Source 实现监听，并存入kafa中
```

+ 模拟业务数据

  > ![image-20220722113341042](http://ybll.vip/md-imgs/202207221133156.png)

### 2- 环境准备

#### 2.1 Hadoop安装

##### 2.1.1 编写分发脚本

> 0. 集群的每台机器都需要安装Hadoop
>
> 1. 编写集群分发脚本 
>
>    ```bash
>    #scp 实现服务器之间的数据拷贝 语法：scp -r $pdir/$fname $user@$host:$pdir/$fname
>    scp -r /opt/module/jdk root@hadoop103:/opt/module
>    scp -r root@hadoop102:/opt/module/* root@hadoop104:/opt/module
>    #rsync 远程同步工具 主要用于备份和镜像，具有速度快、避免复制相同内容和支持符号链接的优点
>    #语法：rsync -av $pdir/fname $user@$host:$pdir/$fname
>    #-a:归档拷贝  -v:显示复制过程
>    rsync -av hadoop-3.1.3/ root@hadoop103:/opt/module/hadoop-3.1.3/
>    ```
>
> 2. shell脚本代码
>
>    + 创建 `xsync` 脚本文件
>
>    ```bash
>    #!/bin/bash
>    if [ $# -lt 1 ]
>    then
>    	echo Not Enough Arguement!
>    	exit;
>    fi
>    #遍历所有机器
>    for host in hadoop102 hadoop103 hadoop104
>    do
>    	echo === $host ===
>    	#当前主机，遍历所有文件
>    	for file in $@
>    	do
>    		if [ -e $file ]
>    		then
>    			#获取父目录
>    			pdir=$(cd -P $(dirname $file), pwd)
>    			#获取当前文件的名称
>    			fname=$(basename $file)
>    			ssh $host "mkdir -p $pdir"
>    			rsync -av $pdir/$fname $host:$pdir
>    		else
>    			echo $file does not exists!
>    		fi
>    	done
>    done
>    ```
>
>    ```bash
>    #后续使用
>    chmod +x xsync #赋予可执行权限
>    xsync /home/atguigu/bin	#分发该脚本
>    sudo cp xsync /bin/ #复制脚本到全局目录，以便全局调用
>    sudo ./bin/xsync etc/profile.d/my_env.sh	#同步环境变量配置
>    ```
>
> 3. 

##### 2.1.2 配置免密登录

> + 免密登录原理
>
>   ![](http://ybll.vip/md-imgs202203291214444.png)
>
> + linux命令操作
>
>   ```bash
>   ##注意：不同用户之间的ssh公钥/私钥不一样
>   #下面操作在root[/root/.ssh]和atguigu两个用户都要执行 
>   
>   #生成公钥和私钥
>   cd /home/atguigu/.ssh
>   ssh-keygen -t rsa #连敲三个回车，会生成两个文件：id_rsa(私钥) id_rsa.pub(公钥)
>   #将公钥拷贝到目标机器 [自己本身这台机器也要拷贝]
>   ssh-copy-id hadoop102
>   ssh-copy-id hadoop103
>   ssh-copy-id hadoop104
>   #.ssh目录下文件解释
>   #known_hosts : 记录ssh访问过计算机的公钥(public key)
>   #authorized_keys : 存放授权过的无密登录服务器公钥
>   ```
>
> + ![image-20220424180946666](http://ybll.vip/md-imgs/202204241809765.png)

##### 2.1.3 Hadoop配置文件修改

> ```xml
> <!--  #####################   1.配置core-site.xml  ########################-->
> <configuration>
> 	<!-- 指定NameNode的地址 -->
>     <property>
>         <name>fs.defaultFS</name>
>         <value>hdfs://hadoop102:8020</value>
> 	</property>
>     
> 	<!-- 指定hadoop数据的存储目录 -->
>     <property>
>         <name>hadoop.tmp.dir</name>
>         <value>/opt/module/hadoop-3.1.3/data</value>
> 	</property>
>     
> 	<!-- 配置HDFS网页登录使用的静态用户为atguigu -->
>     <property>
>         <name>hadoop.http.staticuser.user</name>
>         <value>atguigu</value>
> 	</property>
>     
>     <!-- 解决hive无法启动 beeline -->
> 	<!-- 配置该atguigu(superUser)允许通过代理访问的主机节点 -->
>     <property>
>         <name>hadoop.proxyuser.atguigu.hosts</name>
>         <value>*</value>
> 	</property>
> 	<!-- 配置该atguigu(superUser)允许通过代理用户所属组 -->
>     <property>
>         <name>hadoop.proxyuser.atguigu.groups</name>
>         <value>*</value>
> 	</property>
> </configuration>
> 
> <!-- ##################### 2.配置hdfs-site.xml ########################-->
> <configuration>
> 	<!-- nn web端访问地址-->
> 	<property>
>         <name>dfs.namenode.http-address</name>
>         <value>hadoop102:9870</value>
>     </property>
>     
> 	<!-- 2nn web端访问地址-->
>     <property>
>         <name>dfs.namenode.secondary.http-address</name>
>         <value>hadoop104:9868</value>
>     </property>
> </configuration>
> 
> <!-- #####################  3.配置yarn-site.xml  ########################-->
> <configuration>
> 	<!-- 指定MR走shuffle -->
>     <property>
>         <name>yarn.nodemanager.aux-services</name>
>         <value>mapreduce_shuffle</value>
> 	</property>
>     
> 	<!-- 指定ResourceManager的地址-->
>     <property>
>         <name>yarn.resourcemanager.hostname</name>
>         <value>hadoop103</value>
> 	</property>
>     
> 	<!-- 环境变量的继承 -->
>     <property>
>         <name>yarn.nodemanager.env-whitelist</name>
>         <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
> 	</property>
>     
> 	<!-- yarn容器允许分配的最大最小内存 -->
>     <property>
>         <name>yarn.scheduler.minimum-allocation-mb</name>
>         <value>512</value>
>     </property>
>     <property>
>         <name>yarn.scheduler.maximum-allocation-mb</name>
>         <value>4096</value>
> 	</property>
>     
> 	<!-- yarn容器允许管理的物理内存大小 -->
>     <property>
>         <name>yarn.nodemanager.resource.memory-mb</name>
>         <value>4096</value>
> 	</property>
>     
> 	<!-- 关闭yarn对物理内存和虚拟内存的限制检查 -->
>     <property>
>         <name>yarn.nodemanager.pmem-check-enabled</name>
>         <value>false</value>
>     </property>
>     <property>
>         <name>yarn.nodemanager.vmem-check-enabled</name>
>         <value>false</value>
>     </property>
>     
>     <!-- 开启日志聚集功能 -->
> 	<property>
> 	   	<name>yarn.log-aggregation-enable</name>
> 	    <value>true</value>
> 	</property>
> 	<!-- 设置日志聚集服务器地址 -->
> 	<property>  
> 	    <name>yarn.log.server.url</name>  
> 	    <value>http://hadoop102:19888/jobhistory/logs</value>
> 	</property>
> 	<!-- 设置日志保留时间为7天 -->
> 	<property>
> 	    <name>yarn.log-aggregation.retain-seconds</name>
> 	    <value>604800</value>
> 	</property>
>     
> </configuration>
> 
> <!-- #####################  4.配置mapred-site.xml ########################-->
> <configuration>
> 	<!-- 指定MapReduce程序运行在Yarn上 -->
>     <property>
>         <name>mapreduce.framework.name</name>
>         <value>yarn</value>
>     </property>
>     
>     <!-- 历史服务器端地址 -->
> 	<property>
>     	<name>mapreduce.jobhistory.address</name>
>     	<value>hadoop102:10020</value>
> 	</property>
> <!-- 历史服务器web端地址 -->
> 	<property>
> 	    <name>mapreduce.jobhistory.webapp.address</name>
> 	    <value>hadoop102:19888</value>
> 	</property>
>     
> </configuration>
> ```
>
> ```bash
> vim /opt/module/hadoop-3.1.3/etc/hadoop/workers
> 	#新增内容
> 	#不允许有多余空格，包括结尾和空行
> 	hadoop102
> 	hadoop103
> 	hadoop104
> xsync /opt/module/hadoop-3.1.3/etc/  #同步置集群其他机器
> ```
>
> +++

##### 2.1.4 启动Hadoop集群

> ```bash
> #1.第一次启动，需要在hadoop102节点上格式化 NameNode 
> #注意：格式化NameNode会产生新的集群Id,会导致NameNode和DataNode的集群id不一致而报错
> #解决：停止namenode和datanode进程，删除集群所有机器的data和logs目录，再重新格式化
> hdfs namenode -format
> #2.启动HDFS (hadoop102上)
> sbin/start-dfs.sh
> #3.启动YARN (在ResourceManager的节点上启动-hadoop103)
> sbin/start-yarn.sh
> #4.hadoop102上启动历史服务器
> mapred --daemon start historyserver
> #4.查看浏览器是否能连接
> http://hadoop102:9870 #HDFS上存储的数据信息 (HDFS-NameNode)
> http://hadoop103:8088 #YARN上运行的Job信息 (YARN-ResourceManager)
> http://hadoop102:19888/jobhistory #历史服务器
> http://hadoop104:9868
> ```
>
> + 命令小结
>
>   ```bash
>   #HDFS
>   start-dfs.sh  / stop-dfs.sh
>   #YARN
>   start-yarn.sh / stop-yarn.sh
>   #HDFS各个组件
>   hdfs --daemon start/stop namenode/datanode/secondarynamenode
>   #YARN各个组件
>   yarn --daemon start/stop resourcemanager/nodemanager
>   #启动/停止 历史服务器(日志聚集是在历史服务器中查看运行日志)
>   mapred --daemon start/stop historyserver
>   
>   #刷新namenode
>   hdfs dfsadmin -refreshNodes
>   ```
>
> + 端口号总结
>
>   | 端口名称                   | Hadoop2.x | Hadoop3.x          |
>   | -------------------------- | --------- | ------------------ |
>   | NameNode内部通信(集群内部) | 8020/9000 | **8020**/9000/9800 |
>   | NameNode(http外部访问)     | **50070** | **9870**           |
>   | MapReduce 查看执行任务端口 | 8088      | 8088               |
>   | 历史服务器通信端口         | 19888     | **19888**          |
>
> +++

#### 2.2 ZooKeeper安装

```bash
#解压zookeeper安装包
tar -zxvf zookeeper-3.5.7.tar.gz -C /opt/module
xsync zookeeper-3.5.7.tar.gz/ #同步到集群的其他服务器
#配置服务器编号
mkdir -p zkData
touch zkData/myid   #文件名必须是myid，源码中可见
vim myid#hadoop102,103,104 的myid文件填写不同myid(如：2,3,4)
#修改zoo.cfg
mv zoo_sample.cfg zoo.cfg
vim zoo.cfg
	dataDir=/opt/module/zookeeper-3.5.7/zkData #修改该行
	server.2=hadoop102:2888:3888
	server.A=B:C:D #增加该行，集群多少台机器，就添加多少行
	A:就是myid的值,一个数字
    B:服务器的地址，host或者ip
	C:Follower与Leader交换信息的端口；2888
	D:执行选举新Leader时服务器互相通信的端口；3888
#分发配置到集群的其他机器 xsync

bin/zkServer.sh start #启动集群；每台服务器均需输入该命令启动，可写脚本
```

#### 2.3 Kafka安装

```bash
#1.解压安装包
tar -zxvf kafka_2.12-3.0.0.tgz -C /opt/module/
mv kafka_2.12-3.0.0/ kafka

#2.修改配置文件
vim config/server.properties
#输入一下内容
#broker的全局唯一编号，不能重复
broker.id=0  ###########每台节点都需要改，且id不能一样
#删除topic功能使能,当前版本此配置默认为true，已从配置文件移除
delete.topic.enable=true
#处理网络请求的线程数量
num.network.threads=3
#用来处理磁盘IO的线程数量
num.io.threads=8
#发送套接字的缓冲区大小
socket.send.buffer.bytes=102400
#接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400
#请求套接字的缓冲区大小
socket.request.max.bytes=104857600
#kafka运行日志存放的路径 ###########该项必须改
log.dirs=/opt/module/kafka/datas
#topic在当前broker上的分区个数
num.partitions=1
#用来恢复和清理data下数据的线程数量
num.recovery.threads.per.data.dir=1
#segment文件保留的最长时间，超时将被删除
log.retention.hours=168
#配置连接Zookeeper集群地址 ################该项必须改
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka

#3.配置环境变量
vim /etc/profile.d/my_env.sh
source /etc/profile
xsync kafka/
#4.修改hadoop103,hadoop104中的broker.id=1,2

#5.启动集群,先启动zookeeper,再启动kafka
myzookeeper.sh start #自己写的脚本

#5.hadoop102,103,104节点均启动
bin/kafka-server-start.sh -daemon config/server.properties
#6.关闭集群
bin/kafka-server-stop.sh stop
```

+ kafka群起脚本

  ```shell
  #Kafka群起脚本
  #!/bin/bash
  if [ $# -lt 1 ]
  then
  	echo "参数数量有误！(start stop)"
  	exit
  fi
  for i in hadoop102 hadoop103 hadoop104
  do
  case $1 in
  start)
  	echo "===== 启动 $i KAFKA ====="
  	ssh $i /opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties
  ;;
  stop)
  	echo "===== 停止 $i KAFKA ====="
          ssh $i /opt/module/kafka/bin/kafka-server-stop.sh stop
  ;;
  *)
  	echo "参数输入有误！(start stop)"
  	exit
  ;;
  esac
  done
  ```

#### 2.4 Flume安装

```bash
#解压安装包
tar -zxf /opt/software/apache-flume-1.9.0-bin.tar.gz -C /opt/module
#将lib文件夹下的guava-11.0.2.jar重命名以兼容Hadoop3.1.3
mv /opt/module/flume/lib/guava-11.0.2.jar guava-11.0.2.jar.bak
#若没有启动hadoop直接使用flume，需要有该jar包
```

> ### 堆内存调整
>
> 生产环境中，flume的堆内存通常设置为4G或者更高，配置方式如下（虚拟机环境暂不配置）：
>
> ```sql
> vim conf/flume-env.sh
> 
> export JAVA_OPTS="-Xms4096m -Xmx4096m -Dcom.sun.management.jmxremote"
> ```
>
> 注：
>
> ① -Xms表示JVM Heap（堆内存）最小尺寸，初始分配；
>
> ② -Xmx 表示JVM Heap（堆内存）最大允许的尺寸，按需分配。

+++

#### 2.5 Mysql安装

```bash
#检查是否存在自带Mysql
rpm -qa | grep -i mysql; 
rpm -qa | grep -i -E mysql\|mariadb;

#卸载mariadb
sudo rpm -e --nodes mysql.tar.gz/others;
rpm -qa | grep -i -E mysql\|mariadb | xargs -n1 sudo rpm -e --nodeps

#正式安装
tar -xvf mysql-5.7.tar -C dir/ #解压压缩包

#目录列表，且依次顺序安装
mysql-community-common-xx.rpm; libs-5.7.28; libs-compat; client; server.rpm; 
#安装命令如下
rpm -ivh mysql-community-server-xx.rpm;

sudo rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm

#删除my.cnf文件中，配置项datadir所指目录下的所有文件
#这是原来mysql数据所存储的目录，需要全部删除，保证卸载干净
/etc/my.cnf; datadir=/var/lib/mysql; rm -rf ./*;

#mysql安装后的初始化操作，创建mysql内部数据库和表
sudo mysqld --initialize --user=mysql
cat /var/log/mysqld.log | grep password  #查看临时密码

# erd5#kfzt<9W
# s_3V_Ka8,=vh

#rREHywlRN0-;

#启动mysql, 并登录
sudo systemctl start mysqld
mysql -uroot -p

set password = password("newpw"); #修改root密码
#修改mysql库下user表中root用户允许任意ip连接
use mysql; select host,user from user;
update mysql.user set host='%' where user='root';
flush privileges;
```

> ![image-20220824095728504](http://ybll.vip/md-imgs/202208240958503.png)

#### 2.6 Maxwell 安装

> 0. 安装好 `mysql` 和 `MaxWell`
>
> 1. `Mysql` 中创建 Maxwell所需数据库和用户
>
>    > 且需要启动 MySQL Binlog
>
>    ```mysql
>    create database maxwell;
>    
>    create user 'maxwell'@'%' identified by 'maxwell';
>    
>    grant all on maxwell.* to 'maxwell'@'%'; #访问maxwell数据库权限
>    
>    grant select,replication client,replication slave on *.* to 'maxwell'@'%';
>    ```
>
> 2. 修改 `MaxWell` 配置文件 `config.properties`
>
>    ```properties
>    # tl;dr config
>    log_level=info
>    
>    #Maxwell数据发送目的地，可选配置有stdout|file|kafka|kinesis|pubsub|sqs|rabbitmq|redis
>    producer=kafka
>    #目标Kafka集群地址
>    kafka.bootstrap.servers=hadoop102:9092,hadoop103:9092,hadoop104:9092
>    #目标Kafka topic，可静态配置，例如:maxwell，也可动态配置，例如：%{database}_%{table}
>    kafka_topic=topic_db
>    
>    #MySQL相关配置 （user 和 password 不要修改）
>    host=hadoop102
>    user=maxwell
>    password=maxwell
>    jdbc_options=useSSL=false&serverTimezone=Asia/Shanghai
>    ```
>
>    + 若Maxwell发送数据的目的地是kafka集群，需要首先将kafka集群启动
>
> 3. Maxwell启动与停止
>
>    ```bash
>    #启动命令
>    bin/maxwell --config config.properties --daemon
>    
>    #停止命令
>    ps -ef | grep maxwell | grep -v grep | awk '{print $2}' | xargs kill -9
>    
>    #历史数据全量同步
>    bin/maxwell-bootstrap --database gmall --table user_info --config config.properties
>    ```
>
> 4. 启停脚本 maxwell.sh
>
>    ```shell
>    #!/bin/bash
>    MAXWELL_HOME=/opt/module/maxwell
>    status_maxwell(){
>        result=`ps -ef | grep com.zendesk.maxwell.Maxwell | grep -v grep | wc -l`
>        return $result
>    }
>    
>    start_maxwell(){
>        status_maxwell
>        if [[ $? -lt 1 ]]; then
>            echo "启动Maxwell"
>            $MAXWELL_HOME/bin/maxwell --config $MAXWELL_HOME/config.properties --daemon
>        else
>            echo "Maxwell正在运行"
>        fi
>    }
>    
>    stop_maxwell(){
>        status_maxwell
>        if [[ $? -gt 0 ]]; then
>            echo "停止Maxwell"
>            ps -ef | grep com.zendesk.maxwell.Maxwell | grep -v grep | awk '{print $2}' | xargs kill -9
>        else
>            echo "Maxwell未在运行"
>        fi
>    }
>    
>    case $1 in
>        start )
>            start_maxwell
>        ;;
>        stop )
>            stop_maxwell
>        ;;
>        restart )
>           stop_maxwell
>           start_maxwell
>        ;;
>    esac
>    ```
>
>+++



### 3-日志采集flume (f1)

![image-20220719141302129](http://ybll.vip/md-imgs/202207201843895.png)

+++

> + 用户行为日志文件分布在hadoop102,hadoop103上，需要配置日志采集flume
>
> + 日志采集flume需求
>
>   > 1. 采集日志文件内容
>   > 2. 对日志格式(json)进行检验，检验通过后才能发送到Kafka - 拦截器
>
> + 使用 flume 组件
>
>   > + Source：TailDir Source（断电续传，监控多目录，多个可追加文件）
>   > + Channel：Kafka Channel（source with no sink，直接将日志存入kafka的Topic中）
>   >   + 使用此类型channel，就省去了sink，提高了效率。KafkaChannel数据存储在kafka中，即数据存储在了磁盘中，这里使用了kafka存储数据即获得了kafka的高效率，又利用了kafka数据落盘的安全性。

+++

#### 3.1 自定义拦截器

+ pom.xml

  ```xml
  <dependencies>
      <dependency>
          <groupId>org.apache.flume</groupId>
          <artifactId>flume-ng-core</artifactId>
          <version>1.9.0</version>
          <scope>provided</scope>
      </dependency>
  
      <dependency>
          <groupId>com.alibaba</groupId>
          <artifactId>fastjson</artifactId>
          <version>1.2.62</version>
      </dependency>
  </dependencies>
  
  <build>
      <plugins>
          <plugin>
              <artifactId>maven-compiler-plugin</artifactId>
              <version>2.3.2</version>
              <configuration>
                  <source>1.8</source>
                  <target>1.8</target>
              </configuration>
          </plugin>
          <plugin>
              <artifactId>maven-assembly-plugin</artifactId>
              <configuration>
                  <descriptorRefs>
                      <descriptorRef>jar-with-dependencies</descriptorRef>
                  </descriptorRefs>
              </configuration>
              <executions>
                  <execution>
                      <id>make-assembly</id>
                      <phase>package</phase>
                      <goals>
                          <goal>single</goal>
                      </goals>
                  </execution>
              </executions>
          </plugin>
      </plugins>
  </build>
  ```

+ 代码

  ```java
  public class JSONUtils {
      public static boolean isJSONValidate(String log){
          try {
              // 1. 解析JSON字符串
              JSON.parse(log);
              return true;
          } catch (Exception e) {
              // 2. 失败了，证明不是JSON字符串
              return false;
          }
      }
  }
  ```

  ```java
  public class ETLInterceptor implements Interceptor {
      @Override
      public void initialize() { }
      @Override
      public Event intercept(Event event) {
          String message = new String(event.getBody(), StandardCharsets.UTF_8);
          Map<String, String> headers = event.getHeaders();
          //判断是否是Json字符串
          if(JSONUtils.isJSONValidate(message)){
              headers.put("isJson","true");
          }else {
              headers.put("isJson","false");
          }
          return event;
      }
  
      @Override
      public List<Event> intercept(List<Event> list) {
          Iterator<Event> iterator = list.iterator();
          while (iterator.hasNext()){
              //获取事件
              Event event = iterator.next();
              Event new_event = intercept(event);
              //判断是否符合json格式
              if(new_event.getHeaders().get("isJson").equals("false")){
                  iterator.remove();
              }
          }
          return list;
      }
  
      @Override
      public void close() { }
  
      public static class Builder implements Interceptor.Builder{
  
          @Override
          public Interceptor build() {
              return new ETLInterceptor();
          }
          
          @Override
          public void configure(Context context) {
  
          }
      }
  }
  ```

+ 打包到集群

  ```bash
  #flume-interceptor-1.0-SNAPSHOT-jar-with-dependencies.jar
  collection-1.0-SNAPSHOT-jar-with-dependencies.jar
  #分发jar包
  xsync flume/lib
  ```

#### 3.2 日志采集flume的配置文件

+ flume-tailDir-kafka.conf

```yam
#agent
a1.sources = r1
a1.channels = c1

#source
a1.sources.r1.type = TAILDIR
a1.sources.r1.positionFile = /opt/module/flume/taildir_postion.json
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /opt/module/applog/log/app.*
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = com.atguigu.coll.interceptor.ETLInterceptor$Builder

#channel
a1.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel
a1.channels.c1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092,hadoop104:9092
a1.channels.c1.kafka.topic = topic_log
# 手动指定 消费者组id,不要和其他业务重复
a1.channels.c1.kafka.consumer.group.id = atguigu
# 设置不解析Flume的Event头部信息，及存储数据不需要头部信息
a1.channels.c1.kafka.parseAsFlumeEvent = false

#bind
a1.sources.r1.channels = c1
```

+ hadoop102 hadoop103 上日志采集flume 群起脚本 `f1.sh`

```shell
#!/bin/bash
# 1. 判断是否存在参数
if [ $# == 0 ];then
  echo -e "请输入参数：\nstart   启动日志采集flume；\nstop   关闭日志采集flume；"&&exit
fi

FLUME_HOME=/opt/module/flume

# 2. 根据传入的参数执行命令
case $1 in
  "start"){
      # 3. 分别在hadoop102 hadoop103 上启动日志采集flume
      for host in hadoop102 hadoop103
        do
          echo "---------- 启动 $host 上的 日志采集flume ----------"
          ssh $host " nohup $FLUME_HOME/bin/flume-ng agent -n a1 -c $FLUME_HOME/conf/ -f $FLUME_HOME/job/flume-tailDir-kafka.conf -Dflume.root.logger=INFO,LOGFILE >$FLUME_HOME/logs/flume.log 2>&1 &"
        done
  };;
"stop"){
      # 4. 分别在hadoop102 hadoop103 上启动日志采集flume
      for host in hadoop102 hadoop103
        do
          echo "---------- 停止 $host 上的 日志采集flume ----------"
          flume_count=$(xcall jps -ml | grep flume-tailDir-kafka|wc -l);
          if [ $flume_count != 0 ];then
              ssh $host "ps -ef | grep flume-tailDir-kafka | grep -v grep | awk '{print \$2}' | xargs -n1 kill -9"
          else
              echo "$host 当前没有日志采集flume在运行"
          fi
        done
  };;
esac
```

+ 说明

  > nohup：表示不挂起，不挂断地运行命令
  >
  > awk：使用默认分隔符（空格）切分
  >
  > xargs：表示取前面命令运行的结果，作为后面命令的输入参数

#### 3.3 测试

```bash
# 启动日志采集flume
f1.sh start

#模拟日志产生
lg.sh

# 查看是否有创建 topic_log 
kafka-topics.sh --bootstrap-server hadoop102:9092 --list

#启动一个消费者，从头消费 topic_log主题中的数据
kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic topic_log --from-beginning

#能够看到消费者消费数据
#echo >>  非json格式的数据，则消费者端无法看到追加的日志信息

#关闭日志采集flume集群
f1.sh stop
```

### 4-业务数据采集

> ![image-20220722113341042](http://ybll.vip/md-imgs/202207221133156.png)

#### 4.1 MaxWell

> ​		Maxwell是由美国Zendesk公司开源，使用`Java编写`的**`MySQL变更数据抓取软件`**。他会实时监控Mysql数据库的数据变更操作（包括insert、update、delete），并将变更数据以JSON的格式发送给Kafka、Kinesi等流数据处理平台。

+++

##### 4.1.1 Mysql主从复制

+ Mysql的二进制日志 -- `Binlog`

  > Mysql服务器端的一种日志，会保存Mysql数据库的所有数据变更记录
  >
  > Binlog的主要作用包括：
  >
  > + 主从复制：用来建立一个和主数据库完全一样的数据库环境，称为从数据库
  >
  >   > ```txt
  >   > 热备：主数据库服务器故障后，可切换从数据库继续工作
  >   > 读写分离：主数据库负责数据写入操作，从数据库负责数据查询操作
  >   > ```
  >   >
  >   > + 原理示意图
  >   >
  >   >   ![image-20220722145021020](http://ybll.vip/md-imgs/202207221450128.png)
  >   >
  >   >   ```txt
  >   >   ① Master主库接收到数据变更请求，完成数据变更，并将其写到二进制日志（binary log）中。
  >   >   ② Slave从库向Mysql master发送dump协议，将Master主库的binary log events拷贝到从库的中继日志（relay log）中。
  >   >   ③ Slave从库读取并回放中继日志中的事件，将更新的数据同步到自己的数据库中。
  >   >   ```
  >
  >   +++
  >
  > + 数据恢复

  +++

+ 启动 MySQL Binlog

  > Mysql服务器的Binlog默认是未开启的，如需进行同步，需要将MySQL的Binlog开启
  >
  > + 修改配置文件 `/etc/my.cnf`
  >
  >   ```properties
  >   #数据库id
  >   server-id=1
  >   #启动Binlog，该参数的值会作为binlog的文件名前缀
  >   log-bin=mysql-bin
  >   #binlog类型，maxwell要求为row类型
  >   binlog_format=row
  >   #启动binlog的数据库，需根据实际情况修改配置
  >   binlog-do-db=gmall
  >   ```
  >
  > + MysqlBinlog模式介绍
  >
  >   1. Statement：基于语句，记录所有写操作的SQL语句，节省空间但是可能造成数据不一致，如insert中出现now()
  >   2. row ：基于行，记录每次写操作的被操作行记录的变化，保持数据的绝对一致性，但占用了较大
  >   3. mixed ：
  >
  > + 修改后重启 `sudo systemctl restart mysqld`

+++

##### 4.1.2 MaxWell部署

0. 安装好 `mysql` 和 `MaxWell`

1.  `Mysql` 中创建 Maxwell所需数据库和用户

   > 且需要启动 MySQL Binlog

   ```mysql
   create database maxwell;
   
   create user 'maxwell'@'%' identified by 'maxwell';
   
   grant all on maxwell.* to 'maxwell'@'%'; #访问maxwell数据库权限
   
   grant select,replication client,replication slave on *.* to 'maxwell'@'%';
   ```

2. 修改 `MaxWell` 配置文件

   ```properties
   #Maxwell数据发送目的地，可选配置有stdout|file|kafka|kinesis|pubsub|sqs|rabbitmq|redis
   producer=kafka
   #目标Kafka集群地址
   kafka.bootstrap.servers=hadoop102:9092,hadoop103:9092,hadoop104:9092
   #目标Kafka topic，可静态配置，例如:maxwell，也可动态配置，例如：%{database}_%{table}
   kafka_topic=topic_db
   
   #MySQL相关配置 （user 和 password 不要修改）
   host=hadoop102
   user=maxwell
   password=maxwell
   jdbc_options=useSSL=false&serverTimezone=Asia/Shanghai
   ```

   + 若Maxwell发送数据的目的地是kafka集群，需要首先将kafka集群启动

3. Maxwell启动与停止

   ```bash
   #启动命令
   bin/maxwell --config config.properties --daemon
   
   #停止命令
   ps -ef | grep maxwell | grep -v grep | awk '{print $2}' | xargs kill -9
   
   #历史数据全量同步
   bin/maxwell-bootstrap --database gmall --table user_info --config config.properties
   ```

4. 启停脚本 maxwell.sh

   ```shell
   #!/bin/bash
   
   MAXWELL_HOME=/opt/module/maxwell
   
   status_maxwell(){
       result=`ps -ef | grep com.zendesk.maxwell.Maxwell | grep -v grep | wc -l`
       return $result
   }
   
   
   start_maxwell(){
       status_maxwell
       if [[ $? -lt 1 ]]; then
           echo "启动Maxwell"
           $MAXWELL_HOME/bin/maxwell --config $MAXWELL_HOME/config.properties --daemon
       else
           echo "Maxwell正在运行"
       fi
   }
   
   
   stop_maxwell(){
       status_maxwell
       if [[ $? -gt 0 ]]; then
           echo "停止Maxwell"
           ps -ef | grep com.zendesk.maxwell.Maxwell | grep -v grep | awk '{print $2}' | xargs kill -9
       else
           echo "Maxwell未在运行"
       fi
   }
   
   
   case $1 in
       start )
           start_maxwell
       ;;
       stop )
           stop_maxwell
       ;;
       restart )
          stop_maxwell
          start_maxwell
       ;;
   esac
   ```

   

json格式存入Kafka的topic中

```json
#增量数据同步json格式 type:insert
{
  "database": "gmall",
  "table": "comment_info",
  "type": "insert",
  "ts": 1658590980,
  "xid": 9570,
  "commit": true,
  "data": {
    "id": 1550869093469126668,
    "user_id": 790,
    "nick_name": null,
    "head_img": null,
    "sku_id": 12,
    "spu_id": 3,
    "order_id": 4937,
    "appraise": "1203",
    "comment_txt": "评论内容：81783265177429535173737239368268136427443336237595",
    "create_time": "2020-06-14 23:43:00",
    "operate_time": null
  }
}

# 历史数据全年同步的json格式 type:bootstrap-insert
{
  "database": "gmall",
  "table": "user_info",
  "type": "bootstrap-insert",
  "ts": 1658590616,
  "data": {
    "id": 389,
    "login_name": "cvz5dsr9xd",
    "nick_name": "天达",
    "passwd": null,
    "name": "臧天达",
    "phone_num": "13781829335",
    "email": "cvz5dsr9xd@263.net",
    "head_img": null,
    "user_level": "2",
    "birthday": "1994-07-14",
    "gender": "M",
    "create_time": "2020-06-14 23:21:00",
    "operate_time": null,
    "status": null
  }
}

```

+ 增量数据输出格式

  ![image-20220723234557987](http://ybll.vip/md-imgs/202207232345128.png)

+ 字段说明

  ![image-20220723234711174](http://ybll.vip/md-imgs/202207232347294.png)

4. 说明

   > `MaxWell` 同步mysql增量数据，存入Kafka
   >
   > + Mysql：安装好mysql,启动 MySQL Binlog ; 创建 `maxwell` 的数据库和用户名
   > + MaxWell : 修改配置文件
   >   + 指定同步目的地，Kafka集群地址，以及topic
   >   + 指定Mysql信息



#### 4.2 采集业务数据

```bash
kafka-topics.sh --bootstrap-server hadoop102:9092 --list

kafka-topics.sh --bootstrap-server hadoop102:9092 --create --partitions 1 --replication-factor 3 --topic topic_db
```



### 5-日志消费flume(f2)

#### 5.1 采集步骤说明

> + 消费Kafka集群中的 `topic_log` 的数据，写入到 `HDFS` 中
>
> + flume选择： `kafkaSource`  `FileChannel`  `HDFSSink`
>   + ![image-20220724190119424](http://ybll.vip/md-imgs/202207241901521.png)
> + flume配置文件 `flume-kafka-hdfs.conf`
>
> ```properties
> # agent
> a1.sources=r1
> a1.sinks=k1
> a1.channels=c1
> 
> #sources
> a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
> a1.sources.r1.batchSize = 5000
> a1.sources.r1.batchDurationMillis = 2000
> a1.sources.r1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092,hadoop104:9092
> a1.sources.r1.kafka.topics = topic_log
> a1.sources.r1.kafka.consumer.group.id = flume2_log
> #拦截器
> #a1.sources.r1.interceptors = i1
> #a1.sources.r1.interceptors.i1.type = com.atguigu.collection.interceptor.TimeStampInterceptor$Builder
> 
> #channels
> a1.channels.c1.type = file
> a1.channels.c1.checkpointDir =/opt/module/flume/checkpoint/behavior1
> a1.channels.c1.dataDirs = /opt/module/flume/data/behavior1/
> 
> a1.channels.maxFileSize=2146435071
> a1.channels.capacity=1000000
> a1.channels.keep-alive=6
> 
> #sinks
> a1.sinks.k1.type=hdfs
> #数据飘移问题演示，hdfs生成目录精确到分钟
> a1.sinks.k1.hdfs.path=hdfs://hadoop102/origin_data/gmall/log/topic_log/%Y-%m-%d/%H-%M
> #解决数据飘移问题后，可设置hdfs目录每天创建一个，每个目录中会创建多个文件
> #a1.sinks.k1.hdfs.path=hdfs://hadoop102/origin_data/gmall/log/topic_log/%Y-%m-%d
> #文件格式：log-1592131470866.gz
> a1.sinks.k1.hdfs.filePrefix = log-
> #不使用本地时间戳,自定义拦截器实现hdfs目录创建的时间戳和日志信息中的时间戳同步，解决时间飘移问题
> a1.sinks.k1.hdfs.useLocalTimeStamp = false
> 
> a1.sinks.k1.hdfs.round = true
> a1.sinks.k1.hdfs.roundValue = 1
> a1.sinks.k1.hdfs.roundUnit = minute
> 
> #解决出现大量小文件问题
> a1.sinks.k1.hdfs.rollInterval = 10
> a1.sinks.k1.hdfs.rollSize = 134217728
> a1.sinks.k1.hdfs.rollCount = 0
> a1.sinks.k1.hdfs.fileType = CompressedStream
> a1.sinks.k1.hdfs.codeC = gzip
> 
> #bind
> a1.sources.r1.channels = c1
> a1.sinks.k1.channel = c1
> ```
>
> + 测试
>
> ```bash
> #启动flume1,将产生的日志数据存入Kafka的topic_log中;并启动消费者消费该Topic
> f1.sh start
> bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic topic_log
> 
> #启动flume2,将topic_log中的数据，存入HDFS中
> bin/flume-ng agent -n a1 -c conf/ -f jobs/flume-kafka-hdfs.conf -Dflume.root.logger=INFO,console
> 
> #模拟生成日志数据
> lg.sh
> ```
>
>  	**HDFS对应目录生成文件**



#### 5.2 数据时间飘移问题

> ​		**数据时间飘移问题 : **当数据产生时间是今天的23：59：50时(T1)，在经过日志采集flume1到达kafka，然后再由日志消费flume2，消费出来，最终写入到HDFS时，这里的 `HDFSSink` 是需要使用事件头部中的时间戳属性提取时间的(如创建基于时间的文件夹等)，因为我们设置了HDFS sink的属性值hdfs.useLocalTimeStamp = true 所以，会将数据进入到flume2的时间T2作为当前事件的头部属性中时间戳属性的值。而此时T2是会大于T1的。此时T2可能是明天的00：00：10;所以就会造成当前的数据会进入到明天的文件夹中。
>
> +++
>
> 解决：
>
> 0. 配置分析
>
>    ![image-20220724193111092](http://ybll.vip/md-imgs/202207241931195.png)
>
> 1. 自定义source拦截器，修改头部信息，使得hdfs目录创建时间和日志信息时间戳一致
>
>    ```java
>    public class TimeStampInterceptor implements Interceptor {
>        @Override
>        public void initialize() { }
>    
>        @Override
>        public Event intercept(Event event) {
>            //1.获取Body
>            byte[] body = event.getBody();
>            //2.解析成 Json对象
>            JSONObject json = JSONObject.parseObject(new String(body));
>            //3.获取Json对象中时间戳的值 ts
>            String ts = json.getString("ts");
>    
>            //4.封装头部信息，timestamp是HDFS内部解析时指定的key
>            event.getHeaders().put("timestamp",ts);
>            return event;
>        }
>    
>        @Override
>        public List<Event> intercept(List<Event> list) {
>            for (Event event : list) {
>                intercept(event);
>            }
>            return list;
>        }
>    
>        @Override
>        public void close() { }
>    
>        public static class Builder implements Interceptor.Builder{
>            @Override
>            public Interceptor build() {
>                return new TimeStampInterceptor();
>            }
>            @Override
>            public void configure(Context context) {}
>        }
>    }
>    ```
>
> 2. 修改 flume2 配置文件
>
>    ```properties
>    #配置拦截器
>    a1.sources.r1.interceptors = i1
>    a1.sources.r1.interceptors.i1.type = com.atguigu.collection.interceptor.TimeStampInterceptor$Builder
>    
>    #每分钟创建一个目录
>    a1.sinks.k1.hdfs.round = true
>    a1.sinks.k1.hdfs.roundValue = 1
>    a1.sinks.k1.hdfs.roundUnit = minute
>    ```
>
> 3. 测试同上
>
>    ```bash
>    #启动flume1,将产生的日志数据存入Kafka的topic_log中;并启动消费者消费该Topic
>    f1.sh start
>    bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic topic_log
>    
>    #启动flume2,将topic_log中的数据，存入HDFS中
>    bin/flume-ng agent -n a1 -c conf/ -f jobs/flume-kafka-hdfs.conf -Dflume.root.logger=INFO,console
>    
>    #模拟生成日志数据
>    lg.sh
>    ```

+++

#### 5.3 启动脚本及优化参数

1. 配置优化

   > + FileChannel优化
   >
   >   + 通过配置dataDirs指向多个路径，每个路径对应不同的硬盘，增大Flume吞吐量。
   >
   >     Comma  separated list of directories for storing log files. Using multiple  directories on separate disks can improve file channel peformance  
   >
   >   + checkpointDir和backupCheckpointDir也尽量配置在不同的硬盘对应的目录中，保证checkpoint坏掉后，可以快速使用backupCheckpointDir恢复数据。
   >
   > + HDFS Sink优化 ：HDFS小文件处理：配置三个参数
   >
   >   + hdfs.rollInterval=3600
   >   + hdfs.rollSize=134217728
   >   + hdfs.rollCount =0
   >
   >   效果：当文件达到128M时会产生新的文件；当创建超过3600秒时会滚动产生新的文件

2. 启动脚本

   ```shell
   #! /bin/bash
   # 1. 判断是否存在参数
   if [ $# == 0 ];then
     echo -e "请输入参数：\nstart   启动日志消费flume；\nstop   关闭日志消费flume；"&&exit
   fi
   case $1 in
   "start"){
         echo " --------启动 hadoop104 消费flume-------"
         ssh hadoop104 "nohup /opt/module/flume/bin/flume-ng agent --conf-file /opt/module/flume/jobs/flume-kafka-hdfs.conf --name a1 -Dflume.root.logger=INFO,LOGFILE >/opt/module/flume/logs/flume.log  2>&1 &"
   };;
   
   "stop"){
         echo "---------- 停止 hadoop104 上的 日志消费flume ----------"
         flume_count=$(xcall jps -ml | grep flume-kafka-hdfs|wc -l);
         if [ $flume_count != 0 ];then
             ssh hadoop104 "ps -ef | grep flume-kafka-hdfs | grep -v grep | awk '{print \$2}' | xargs -n1 kill -9"
         else
             echo " hadoop104 当前没有日志采集flume在运行"
         fi
     };;
   esac
   ```

3. 测试

   ```bash
   #启动flume1,flume2,再模拟日志生成，查看HDFS数据变化
   f1.sh start
   f2.sh start
   lg.sh
   ```
   
   ![image-20220726105237138](http://ybll.vip/md-imgs/202207261052282.png)

### 6-业务数据同步

> 业务数据是数据仓库的重要数据来源，我们需要每日定时从业务数据库中抽取数据，传输到数据仓库中，之后再对数据进行分析统计。为保证统计结果的正确性，需要保证数据仓库中的数据与业务数据库是同步的，离线数仓的计算周期通常为天，所以数据同步周期也通常为天，即每天同步一次即可。
>
> **在同步业务数据时有两种同步策略：** **全量同步** 和 **增量同步**
>
> + 全量同步策略
>
>   ![image-20220725110634224](http://ybll.vip/md-imgs/202207251106351.png)
>
> + 增量同步策略
>
>   ![image-20220725110702702](http://ybll.vip/md-imgs/202207251107790.png)
>
> + 如何选择?
>
>   + **若业务表数据量比较大，且每天的数据变化比例还比较低，应采用增量同步，否则采用全量同步。**
>
>   | 同步策略 | 优点 | 缺点                                                         |
>   | :------: | -------------------------------- | ------------------------------------------------------------ |
>   | 全量同步     | 逻辑简单                     | 某些情况下效率低下                                           |
>   | 增量同步 | 效率高，无需同步和存储重复的数据 | 逻辑复杂，需要将每日的新增及变化数据同原来的数据进行整合，才能使用 |
>
> + 本次采集项目
>
>   > 全部表都实现 **增量同步** ，保存至 Kafka，供后续实时项目;
>   >
>   > 全量同步部分表只 Mysql
>   >
>   > <img src="http://ybll.vip/md-imgs/202207251111858.png" alt="image-20220725111106781" style="zoom: 67%;" />

+++

#### 6.1 全量数据同步

##### 6.1.1 DataX

> + 同步工具概述
>
>   ![image-20220725111654309](http://ybll.vip/md-imgs/202207251116414.png)
>
> 

##### 6.1.2 业务数据同步步骤

![image-20220725162800421](http://ybll.vip/md-imgs/202207251628695.png)

1. 编写DataX配置文件 `activity_info.json`

   ```json
   {
     "job": {
       "content": [
         {
           "reader": {
             "name": "mysqlreader",
             "parameter": {
               "column": [
                 "id",
                 "activity_name",
                 "activity_type",
                 "activity_desc",
                 "start_time",
                 "end_time",
                 "create_time"
               ],
               "connection": [
                 {
                   "jdbcUrl": [
                     "jdbc:mysql://hadoop102:3306/gmall"
                   ],
                   "table": [
                     "activity_info"
                   ]
                 }
               ],
               "username": "root",
               "password": "1234",
               "splitPk": ""
             }
           },
           "writer": {
             "name": "hdfswriter",
             "parameter": {
               "column": [
                 {
                   "name": "id",
                   "type": "bigint"
                 },
                 {
                   "name": "activity_name",
                   "type": "string"
                 },
                 {
                   "name": "activity_type",
                   "type": "string"
                 },
                 {
                   "name": "activity_desc",
                   "type": "string"
                 },
                 {
                   "name": "start_time",
                   "type": "string"
                 },
                 {
                   "name": "end_time",
                   "type": "string"
                 },
                 {
                   "name": "create_time",
                   "type": "string"
                 }
               ],
               "compress": "gzip",
               "defaultFS": "hdfs://hadoop102:8020",
               "fieldDelimiter": "\t",
               "fileName": "activity_info",
               "fileType": "text",
               "path": "${targetdir}",
               "writeMode": "append"
             }
           }
         }
       ],
       "setting": {
         "speed": {
           "channel": 1
         }
       }
     }
   }
   ```

2. 测试配置文件

   ```bash
   #需要先创建目录，否则报错
   hadoop fs -mkdir -p /origin_data/gmall/db/activity_info_full/2020-06-14
   
   #执行dataX
   python bin/datax.py job/activity_info.json -p"-Dtargetdir=/origin_data/gmall/db/activity_info_full/2020-06-14"
   ```

3. DataX配置文件生成的python脚本 `gen_import_config.py`

   ```python
   # coding=utf-8
   import json
   import getopt
   import os
   import sys
   import MySQLdb
   
   #MySQL相关配置，需根据实际情况作出修改
   mysql_host = "hadoop102"
   mysql_port = "3306"
   mysql_user = "root"
   mysql_passwd = "1234"
   
   #HDFS NameNode相关配置，需根据实际情况作出修改
   hdfs_nn_host = "hadoop102"
   hdfs_nn_port = "8020"
   
   #生成配置文件的目标路径，可根据实际情况作出修改
   output_path = "/opt/module/datax/job/import"
   
   def get_connection():
       return MySQLdb.connect(host=mysql_host, port=int(mysql_port), user=mysql_user, passwd=mysql_passwd)
   
   def get_mysql_meta(database, table):
       connection = get_connection()
       cursor = connection.cursor()
       sql = "SELECT COLUMN_NAME,DATA_TYPE from information_schema.COLUMNS WHERE TABLE_SCHEMA=%s AND TABLE_NAME=%s ORDER BY ORDINAL_POSITION"
       cursor.execute(sql, [database, table])
       fetchall = cursor.fetchall()
       cursor.close()
       connection.close()
       return fetchall
   
   def get_mysql_columns(database, table):
       return map(lambda x: x[0], get_mysql_meta(database, table))
   
   def get_hive_columns(database, table):
       def type_mapping(mysql_type):
           mappings = {
               "bigint": "bigint",
               "int": "bigint",
               "smallint": "bigint",
               "tinyint": "bigint",
               "decimal": "string",
               "double": "double",
               "float": "float",
               "binary": "string",
               "char": "string",
               "varchar": "string",
               "datetime": "string",
               "time": "string",
               "timestamp": "string",
               "date": "string",
               "text": "string"
           }
           return mappings[mysql_type]
       meta = get_mysql_meta(database, table)
       return map(lambda x: {"name": x[0], "type": type_mapping(x[1].lower())}, meta)
   
   def generate_json(source_database, source_table):
       job = {
           "job": {
               "setting": {
                   "speed": {
                       "channel": 3
                   },
                   "errorLimit": {
                       "record": 0,
                       "percentage": 0.02
                   }
               },
               "content": [{
                   "reader": {
                       "name": "mysqlreader",
                       "parameter": {
                           "username": mysql_user,
                           "password": mysql_passwd,
                           "column": get_mysql_columns(source_database, source_table),
                           "splitPk": "",
                           "connection": [{
                               "table": [source_table],
                               "jdbcUrl": ["jdbc:mysql://" + mysql_host + ":" + mysql_port + "/" + source_database]
                           }]
                       }
                   },
                   "writer": {
                       "name": "hdfswriter",
                       "parameter": {
                           "defaultFS": "hdfs://" + hdfs_nn_host + ":" + hdfs_nn_port,
                           "fileType": "text",
                           "path": "${targetdir}",
                           "fileName": source_table,
                           "column": get_hive_columns(source_database, source_table),
                           "writeMode": "append",
                           "fieldDelimiter": "\t",
                           "compress": "gzip"
                       }
                   }
               }]
           }
       }
       if not os.path.exists(output_path):
           os.makedirs(output_path)
       with open(os.path.join(output_path, ".".join([source_database, source_table, "json"])), "w") as f:
           json.dump(job, f)
   
   def main(args):
       source_database = ""
       source_table = ""
       options, arguments = getopt.getopt(args, '-d:-t:', ['sourcedb=', 'sourcetbl='])
       for opt_name, opt_value in options:
           if opt_name in ('-d', '--sourcedb'):
               source_database = opt_value
           if opt_name in ('-t', '--sourcetbl'):
               source_table = opt_value
       generate_json(source_database, source_table)
   
   if __name__ == '__main__':
       main(sys.argv[1:])
   ```

4. 执行python脚本

   ```bash
   #需要先安装MySQL-python驱动
   sudo yum install -y MySQL-python
   
   #使用脚本 python gen_import_config.py -d db_name -t tbl_name
   python ~/bin/gen_import_config.py -d gmall -t activity_rule
   
   #对应目录生成了同步 activity_rule表 的json配置文件
   ```

5. 编写shell脚本，统一生成json配置文件  `gen_import_config.sh`

   ```shell
   #!/bin/bash
   
   python ~/bin/gen_import_config.py -d gmall -t activity_info
   python ~/bin/gen_import_config.py -d gmall -t activity_rule
   python ~/bin/gen_import_config.py -d gmall -t base_category1
   python ~/bin/gen_import_config.py -d gmall -t base_category2
   python ~/bin/gen_import_config.py -d gmall -t base_category3
   python ~/bin/gen_import_config.py -d gmall -t base_dic
   python ~/bin/gen_import_config.py -d gmall -t base_province
   python ~/bin/gen_import_config.py -d gmall -t base_region
   python ~/bin/gen_import_config.py -d gmall -t base_trademark
   python ~/bin/gen_import_config.py -d gmall -t cart_info
   python ~/bin/gen_import_config.py -d gmall -t coupon_info
   python ~/bin/gen_import_config.py -d gmall -t sku_attr_value
   python ~/bin/gen_import_config.py -d gmall -t sku_info
   python ~/bin/gen_import_config.py -d gmall -t sku_sale_attr_value
   python ~/bin/gen_import_config.py -d gmall -t spu_info
   ```

6. 测试python生成的json配置文件

   ```bash
   #需要先创建目录，否则报错
   hadoop fs -mkdir -p /origin_data/gmall/db/activity_info_full/2020-06-14
   
   #执行dataX
   python bin/datax.py job/import/gmall.activity_info.json -p"-Dtargetdir=/origin_data/gmall/db/activity_info_full/2020-06-14"
   
   #查看同步结果
   hadoop fs -cat /origin_data/gmall/db/activity_info_full/2020-06-14/* | zcat
   ```

   ![image-20220725173106829](http://ybll.vip/md-imgs/202207251731901.png)

7. 编写全表数据同步的shell脚本 `mysql_to_hdfs_full.sh`

   ```shell
   #!/bin/bash
   # 定义datax的根目录
   DATAX_HOME=/opt/module/datax
   # 如果传入日期则do_date等于传入的日期，否则等于前一天日期
   if [ -n "$2" ] ;then
       do_date=$2
   else
       do_date=`date -d "-1 day" +%F`
   fi
   #处理目标路径，此处的处理逻辑是，如果目标路径不存在，则创建；若存在，则清空，目的是保证同步任务可重复执行
   handle_targetdir() {
     hadoop fs -test -e $1
     if [[ $? -eq 1 ]]; then
       echo "路径$1不存在，正在创建......"
       hadoop fs -mkdir -p $1
     else
       echo "路径$1已经存在"
       fs_count=$(hadoop fs -count $1)
       content_size=$(echo $fs_count | awk '{print $3}')
       if [[ $content_size -eq 0 ]]; then
         echo "路径$1为空"
       else
         echo "路径$1不为空，正在清空......"
         hadoop fs -rm -r -f $1/*
       fi
     fi
   }
   #数据同步
   import_data() {
     datax_config=$1
     target_dir=$2
     handle_targetdir $target_dir
     python $DATAX_HOME/bin/datax.py -p"-Dtargetdir=$target_dir" $datax_config
   }
   # 根据传入的表名，处理不同的表
   case $1 in
   "activity_info")
     import_data /opt/module/datax/job/import/gmall.activity_info.json /origin_data/gmall/db/activity_info_full/$do_date
     ;;
   "activity_rule")
     import_data /opt/module/datax/job/import/gmall.activity_rule.json /origin_data/gmall/db/activity_rule_full/$do_date
     ;;
   "base_category1")
     import_data /opt/module/datax/job/import/gmall.base_category1.json /origin_data/gmall/db/base_category1_full/$do_date
     ;;
   "base_category2")
     import_data /opt/module/datax/job/import/gmall.base_category2.json /origin_data/gmall/db/base_category2_full/$do_date
     ;;
   "base_category3")
     import_data /opt/module/datax/job/import/gmall.base_category3.json /origin_data/gmall/db/base_category3_full/$do_date
     ;;
   "base_dic")
     import_data /opt/module/datax/job/import/gmall.base_dic.json /origin_data/gmall/db/base_dic_full/$do_date
     ;;
   "base_province")
     import_data /opt/module/datax/job/import/gmall.base_province.json /origin_data/gmall/db/base_province_full/$do_date
     ;;
   "base_region")
     import_data /opt/module/datax/job/import/gmall.base_region.json /origin_data/gmall/db/base_region_full/$do_date
     ;;
   "base_trademark")
     import_data /opt/module/datax/job/import/gmall.base_trademark.json /origin_data/gmall/db/base_trademark_full/$do_date
     ;;
   "cart_info")
     import_data /opt/module/datax/job/import/gmall.cart_info.json /origin_data/gmall/db/cart_info_full/$do_date
     ;;
   "coupon_info")
     import_data /opt/module/datax/job/import/gmall.coupon_info.json /origin_data/gmall/db/coupon_info_full/$do_date
     ;;
   "sku_attr_value")
     import_data /opt/module/datax/job/import/gmall.sku_attr_value.json /origin_data/gmall/db/sku_attr_value_full/$do_date
     ;;
   "sku_info")
     import_data /opt/module/datax/job/import/gmall.sku_info.json /origin_data/gmall/db/sku_info_full/$do_date
     ;;
   "sku_sale_attr_value")
     import_data /opt/module/datax/job/import/gmall.sku_sale_attr_value.json /origin_data/gmall/db/sku_sale_attr_value_full/$do_date
     ;;
   "spu_info")
     import_data /opt/module/datax/job/import/gmall.spu_info.json /origin_data/gmall/db/spu_info_full/$do_date
     ;;
   "all")
     import_data /opt/module/datax/job/import/gmall.activity_info.json /origin_data/gmall/db/activity_info_full/$do_date
     import_data /opt/module/datax/job/import/gmall.activity_rule.json /origin_data/gmall/db/activity_rule_full/$do_date
     import_data /opt/module/datax/job/import/gmall.base_category1.json /origin_data/gmall/db/base_category1_full/$do_date
     import_data /opt/module/datax/job/import/gmall.base_category2.json /origin_data/gmall/db/base_category2_full/$do_date
     import_data /opt/module/datax/job/import/gmall.base_category3.json /origin_data/gmall/db/base_category3_full/$do_date
     import_data /opt/module/datax/job/import/gmall.base_dic.json /origin_data/gmall/db/base_dic_full/$do_date
     import_data /opt/module/datax/job/import/gmall.base_province.json /origin_data/gmall/db/base_province_full/$do_date
     import_data /opt/module/datax/job/import/gmall.base_region.json /origin_data/gmall/db/base_region_full/$do_date
     import_data /opt/module/datax/job/import/gmall.base_trademark.json /origin_data/gmall/db/base_trademark_full/$do_date
     import_data /opt/module/datax/job/import/gmall.cart_info.json /origin_data/gmall/db/cart_info_full/$do_date
     import_data /opt/module/datax/job/import/gmall.coupon_info.json /origin_data/gmall/db/coupon_info_full/$do_date
     import_data /opt/module/datax/job/import/gmall.sku_attr_value.json /origin_data/gmall/db/sku_attr_value_full/$do_date
     import_data /opt/module/datax/job/import/gmall.sku_info.json /origin_data/gmall/db/sku_info_full/$do_date
     import_data /opt/module/datax/job/import/gmall.sku_sale_attr_value.json /origin_data/gmall/db/sku_sale_attr_value_full/$do_date
     import_data /opt/module/datax/job/import/gmall.spu_info.json /origin_data/gmall/db/spu_info_full/$do_date
     ;;
   esac
   ```

8. 使用及说明

   > ```bash
   > mysql_to_hdfs_full.sh all 2020-06-14
   > mysql_to_hdfs_full.sh activity_info  #全量同步当前时间前一天的数据
   > ```
   >
   > + 没有对应目录则会创建目录，再进行数据全量同步
   > + 有目录则会清空数据，再进行全量同步

   +++
   
   效果展示
   
   > ![image-20220726105533312](http://ybll.vip/md-imgs/202207261055460.png)
   >
   > +++
   >
   > ![image-20220726105634950](http://ybll.vip/md-imgs/202207261056049.png)
   >
   > +++
   >
   > 
   
   
   
   +++

#### 6.2  增量数据同步

![image-20220726070940306](http://ybll.vip/md-imgs/202207260710342.png)

##### 6.2.1 业务数据消费flume(f3)

> 需求：将MaxWell采集到Kafka中topic_db的业务变更数据传输到HDFS中
>
> Flume组件：KafkaSource、HDFSSink、FileChannel
>
> ![image-20220726071614195](http://ybll.vip/md-imgs/202207260716347.png)
>
> > + flume配置
> >
> >   ```properties
> >   # agent
> >   a1.sources = r1
> >   a1.channels = c1
> >   a1.sinks = k1
> >   
> >   # sources
> >   a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
> >   a1.sources.r1.batchSize = 5000
> >   a1.sources.r1.batchDurationMillis = 2000
> >   a1.sources.r1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092,hadoop104
> >   a1.sources.r1.kafka.topics = topic_db
> >   a1.sources.r1.kafka.consumer.group.id = flume3_db
> >   a1.sources.r1.setTopicHeader = true
> >   a1.sources.r1.topicHeader = topic
> >   a1.sources.r1.interceptors = i1
> >   a1.sources.r1.interceptors.i1.type =com.atguigu.collection.interceptor.TableNameInterceptor$Builder
> >   
> >   
> >   a1.channels.c1.type = file
> >   a1.channels.c1.checkpointDir = /opt/module/flume/checkpoint/behavior2
> >   a1.channels.c1.dataDirs = /opt/module/flume/data/behavior2/
> >   a1.channels.c1.maxFileSize = 2146435071
> >   a1.channels.c1.capacity = 1000000
> >   a1.channels.c1.keep-alive = 6
> >   
> >   ## sink1
> >   a1.sinks.k1.type = hdfs
> >   # %{tableName}:头部信息中获取tableName
> >   a1.sinks.k1.hdfs.path = /origin_data/gmall/db/%{tableName}_inc/%Y-%m-%d
> >   a1.sinks.k1.hdfs.filePrefix = db
> >   a1.sinks.k1.hdfs.round = false
> >   
> >   
> >   a1.sinks.k1.hdfs.rollInterval = 10
> >   a1.sinks.k1.hdfs.rollSize = 134217728
> >   a1.sinks.k1.hdfs.rollCount = 0
> >   
> >   
> >   a1.sinks.k1.hdfs.fileType = CompressedStream
> >   a1.sinks.k1.hdfs.codeC = gzip
> >   
> >   ## bind
> >   a1.sources.r1.channels = c1
> >   a1.sinks.k1.channel= c1
> >   ```
> >
> > + 拦截器编写
> >
> >   ```java
> >   public class TableNameInterceptor implements Interceptor {
> >       @Override
> >       public void initialize() { }
> >   
> >       @Override
> >       public Event intercept(Event event) {
> >           //1.获取header头部信息,和body
> >           Map<String, String> header = event.getHeaders();
> >           JSONObject json = JSONObject.parseObject(
> >               new String(event.getBody(), 
> >               StandardCharsets.UTF_8)
> >           );
> >           //2.获取时间戳对象
> >           String ts = json.getString("ts");
> >           ts = ts + "000";
> >           //3.获取表名
> >           String table_name = json.getString("table");
> >           //4.将时间戳和表名放入header
> >           header.put("tableName",table_name);
> >           header.put("timestamp",ts);
> >           return event;
> >       }
> >   
> >       @Override
> >       public List<Event> intercept(List<Event> list) {
> >           for (Event event : list) {
> >               intercept(event);
> >           }
> >           return list;
> >       }
> >   
> >       @Override
> >       public void close() { }
> >       
> >       public static class Builder implements Interceptor.Builder {
> >   
> >           @Override
> >           public Interceptor build() {
> >               return new TableNameInterceptor();
> >           }
> >           @Override
> >           public void configure(Context context) { }
> >       }
> >   }
> >   ```
> >
> > + 测试
> >
> >   ```bash
> >   # 在hadoop104上启动flume3,开启mysql增量数据从Kafka采集到 HDFS
> >   bin/flume-ng agent -n a1 -c conf/ -f jobs/flume-kafka-hdfs-db.conf -Dflume.root.logger=INFO,console
> >
> >   #启动maxwell,对mysql进行监听
> >   maxwell.sh start
> >   
> >   #可开启一个kafka消费者监听 topic_log，检查是否f1有存入数据到Kafka
> >   kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic topic_db
> >   
> >   #hadoop102上模拟log日志信息生成
> >   java -jar gmall2020-mock-log-2021-01-22.jar
> >   
> >   #hdfs上查看目录的变化 /origin_data/gmall/db/
> >   ```
> >   
> >   ![image-20220726111116502](http://ybll.vip/md-imgs/202207261111634.png)
> >   
> >   +++
> >   
> >   ![image-20220726111201985](http://ybll.vip/md-imgs/202207261112071.png)
> >   
> > + HDFS创建`日期-目录`说明
> >
> >   > maxwell输出的到Kafka中的Json字符串中的ts字段值是Mysql数据变动的日期，此时与模拟数据的日期对应不上，因此需要修改
> >   >
> >   > ![image-20220726111633077](http://ybll.vip/md-imgs/202207261116162.png)
> >   >
> >   > 此项目为了模拟真实环境，对Maxwell源码进行了改动，增加了一个参数mock_date，该参数的作用就是指定Maxwell输出JSON字符串的ts时间戳的日期，接下来进行测试。
> >   >
> >   > + 修改Maxwell配置文件 `config.properties` ，增加 `mock_date` 参数，
> >   >
> >   >   > 和/opt/module/db_log/application.properties中的mock.date参数保持一致；
> >   >   >
> >   >   > mock_date=2020-06-14
> >   >
> >   >   注：该参数仅供当前项目学习使用，修改该参数后重启Maxwell才可生效。
> >   >
> >   > + 重启Maxwell  `maxwell.sh restart`
> >   >
> >   > + 重新生成模拟数据（同上一部分测试）
> >   > + 此时生成的`日期目录`与模拟数据变化的日期能够对应
> >
> >   +++
> >

+++

##### 6.2.2 相关脚本说明

+ 业务数据消费flume脚本: f3.sh

  > ```shell
  > #!/bin/bash
  > 
  > case $1 in
  > "start")
  >         echo " --------启动 hadoop104 业务数据flume-------"
  >         ssh hadoop104 "nohup /opt/module/flume/bin/flume-ng agent -n a1 -c /opt/module/flume/conf -f /opt/module/flume/jobs/flume-kafka-hdfs-db.conf >/dev/null 2>&1 &"
  > ;;
  > "stop")
  > 
  >         echo " --------停止 hadoop104 业务数据flume-------"
  >         ssh hadoop104 "ps -ef | grep flume-kafka-hdfs-db.conf | grep -v grep |awk '{print \$2}' | xargs -n1 kill -9"
  > ;;
  > esac
  > ```
  >
  > + 测试
  >
  >   ```bash
  >   # 在hadoop104上启动flume3,开启mysql增量数据从Kafka采集到 HDFS
  >   f3.sh start
  >   #f3.sh stop 关闭flume进程
  >   #启动maxwell,对mysql进行监听
  >   maxwell.sh start
  >   
  >   #可开启一个kafka消费者监听 topic_log，检查是否f1有存入数据到Kafka
  >   kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic topic_db
  >   
  >   #hadoop102上模拟log日志信息生成
  >   java -jar gmall2020-mock-log-2021-01-22.jar
  >   
  >   #hdfs上查看目录的变化 /origin_data/gmall/db/
  >   ```

+ `增量表`首日进行`全量同步`

  > 通常情况下，增量表需要在首日进行一次全量同步，后续每日再进行增量同步，首日全量同步可以使用Maxwell的bootstrap功能，方便起见，下面编写一个增量表首日全量同步脚本。
  >
  > + mysql_to_kafka_inc_init.sh
  >
  >   ```shell
  >   #!/bin/bash
  >   
  >   # 该脚本的作用是初始化所有的增量表，只需执行一次
  >   
  >   MAXWELL_HOME=/opt/module/maxwell
  >   
  >   import_data() {
  >       $MAXWELL_HOME/bin/maxwell-bootstrap --database gmall --table $1 --config $MAXWELL_HOME/config.properties
  >   }
  >   
  >   case $1 in
  >   "cart_info")
  >     import_data cart_info
  >     ;;
  >   "comment_info")
  >     import_data comment_info
  >     ;;
  >   "coupon_use")
  >     import_data coupon_use
  >     ;;
  >   "favor_info")
  >     import_data favor_info
  >     ;;
  >   "order_detail")
  >     import_data order_detail
  >     ;;
  >   "order_detail_activity")
  >     import_data order_detail_activity
  >     ;;
  >   "order_detail_coupon")
  >     import_data order_detail_coupon
  >     ;;
  >   "order_info")
  >     import_data order_info
  >     ;;
  >   "order_refund_info")
  >     import_data order_refund_info
  >     ;;
  >   "order_status_log")
  >     import_data order_status_log
  >     ;;
  >   "payment_info")
  >     import_data payment_info
  >     ;;
  >   "refund_payment")
  >     import_data refund_payment
  >     ;;
  >   "user_info")
  >     import_data user_info
  >     ;;
  >   "all")
  >     import_data cart_info
  >     import_data comment_info
  >     import_data coupon_use
  >     import_data favor_info
  >     import_data order_detail
  >     import_data order_detail_activity
  >     import_data order_detail_coupon
  >     import_data order_info
  >     import_data order_refund_info
  >     import_data order_status_log
  >     import_data payment_info
  >     import_data refund_payment
  >     import_data user_info
  >     ;;
  >   esac
  >   ```
  >
  > + 执行脚本
  >
  > ```bash
  > #先清空HDFS上之前同步的增量表数据
  > 
  > #执行上述脚本
  > mysql_to_kafka_inc_init.sh all
  > 
  > #查看同步结果
  > ```
  >
  > + 结果：所有增量表数据全部到达HDFS
  >
  >   ![image-20220726123101637](http://ybll.vip/md-imgs/202207261231746.png)
  >
  >   +++
  >
  >   ![image-20220726122940457](C:\Users\YBLLcodes\AppData\Roaming\Typora\typora-user-images\image-20220726122940457.png)

  +++

+ 采集通道`启动/停止`脚本 : cluster.sh

  ```shell
  #!/bin/bash
  
  case $1 in
  "start"){
          echo ================== 启动 集群 ==================
  
          #启动 Zookeeper集群
          myzookeeper.sh start
  
          #启动 Hadoop集群
          myhadoop.sh start
  
          #启动 Kafka采集集群
          mykafka.sh start
  
          #启动采集 Flume
          f1.sh start
  
          #启动日志消费 Flume
          f2.sh start
  
          #启动业务消费 Flume
          f3.sh start
  
          #启动 maxwell
          maxwell.sh start
  
          };;
  "stop"){
          echo ================== 停止 集群 ==================
  
          #停止 Maxwell
          maxwell.sh stop
  
          #停止 业务消费Flume
          f3.sh stop
  
          #停止 日志消费Flume
          f2.sh stop
  
          #停止 日志采集Flume
          f1.sh stop
  
          #停止 Kafka采集集群
          mykafka.sh stop
  
          #停止 Hadoop集群
          myhadoop.sh stop
  
          #停止 Zookeeper集群
          myzookeeper.sh stop
  
  };;
  esac
  ```





## 数仓项目

### 0-总结

> 1、数仓概述
> 	数仓就是数据仓库,主要负责数据的存储、管理和分析
> 2、数仓架构
> 	日志数据->日志服务器本地磁盘->Flume -> Kafka ->Flume->HDFS->HIVE[ODS/DWD/DIM/DWS/ADS] ->datax -> mysql ->可视化
> 						->全量导入: -> datax -> HDFS
> 	业务数据->mysql->													->HIVE[ODS/DWD/DIM/DWS/ADS] ->datax -> mysql ->可视化
> 						->增量导入: -> maxwell -> kafka ->Flume ->HDFS
> 3、日志数据格式
> 	页面数据: 普通文本、采用gzip压缩
> 		数据格式: { "common":{....},"page":{...},"actions":[ {....},{...} ...],"displays":[{...},{...},...] ,"ts":...}
> 	启动数据
> 		数据格式: { "common":{....},"start":{...},"ts":...}
> 	1、同步策略
> 		全量同步: 每天都将表中所有数据同步到HDFS一个目录中
> 			适用场景: 表的数据量很小或者表数据量很大但是每天新增和修改的数据特别多
> 			优点: 
> 				1、导入简单: select * from 表名
> 				2、适用简单:
> 					1、查询最新数据,只需要查看HDFS最新目录即可
> 					2、查询某天的历史数据,只需要查看HDFS某天的目录即可
> 			缺点: 因为每天都是将表所有数据同步,所以如果数据变动比较小,表数据量比较大,那么此时HDFS多个目录中就存在大量的重复数据,占用过多的磁盘空间。
> 		增量同步: 首日是将表中所有数据同步到HDFS一个目录中,后续每日只需要同步当天新增和修改的数据。
> 			适用场景: 表数据量很大但是每天新增和修改的数据不是特别多
> 			优点: HDFS多个目录中不存在重复的数据
> 			缺点:
> 				1、导入稍微复杂,需要找到当天新增和修改的数据
> 				2、使用稍微复杂,如果需要查询最新的数据,需要从多个目录获取
> 	2、业务数据格式: 文本,gzip压缩
> 		全量数据格式: mysql列之间的数据以\t分割
> 		增量数据格式: json文本
> 			首日数据格式: 
> 				{ "databases":...,"table":"..","ts":....,type:"bootstrap-start","datas":{} }
> 				{ "databases":...,"table":"..","ts":....,type:"bootstrap-insert","datas":{....} }
> 				.....
> 				{ "databases":...,"table":"..","ts":....,type:"bootstrap-complete","datas":{} }
> 			每日数据格式:
> 				{ "databases":...,"table":"..","ts":....,type:"insert","datas":{...} }
> 				{ "databases":...,"table":"..","ts":....,type:"delete","datas":{...} }
> 				{ "databases":...,"table":"..","ts":....,type:"update","datas":{...},"old":{....} }
> 2、建模的意义
> 	建模: 将数据有序的组织与存储[建表]
> 	模型: 将数据有序的组织与存储的方法[其实就是指导建表的理论]
> 	ER模型
> 		定义: 以实体与关系的方式描述企业业务,以规范化的形式表示,遵循3NF理论
> 			实体: 就是对象
> 			关系: 1对1、N对N、1对N关系
> 			规范化: 设计数据库的时候需要遵循标准[范式]
> 		优点: 减少数据冗余,增量数据的一致性[其实就是将一个表拆成了多个表]
> 		缺点: 表会比较多,查询的时候性能会比较慢
> 		第一范式: 属性不可切分
> 		第二范式: 非主键字段必须完全函数依赖主键字段
> 		第三范式: 非主键字段不能传递函数依赖主键字段
> 		ER模型不适合做数据分析,所以数仓不采用ER模型
> 	维度模型
> 		定义: 以事实与维度描述企业业务
> 			事实: 与业务过程对应
> 			维度: 事实发生的环境描述
> 		优点: 表会比较少,查询性能会相对比较高 [其实就是将多个表合并成一个表]
> 		缺点: 数据冗余,一致性比较差
> 		维度模型适合用于数据分析,数仓一般采用维度建模
> 3、维度建模之事实表
> 	定义: 围绕业务过程设计的表称之为事实表,
> 		事实表中的字段一般包含维度外键+度量值
> 	特点: 细长[列少行多]
> 	分类: 事务型事实表、周期快照型事实表、累积型快照事实表
> 	事务型事实表
> 		定义: 记录每个业务过程最细粒度操作的表
> 		设计流程
> 			1、选择业务过程[确定需要创建多少个事务型事实表]
> 				选择需求需要的或者可能需要的业务过程
> 				一般一个业务过程对应一张事务型事实表
> 			2、确定粒度[确定表的行数据是啥]
> 				粒度: 代表表中每一行存储的数据代表的含义
> 				尽可能选择最细粒度,粒度越细,后续能够满足的需求越多
> 			3、确定维度[确定事实表的维度外键列]
> 				维度尽可能丰富,维度越丰富,后续可供分析的指标越多
> 			4、确认度量[确定事实表的度量值列]
> 				度量值一般是可累加的数字
> 		业务过程设计事务型事实表之后,理论上是可以满足该业务过程所有需求分析的.
> 		但是在某些场景需求指标,使用事务型事实表性能会比较低。
> 		不足之处:
> 			1、存量[现存数据量]型指标 <账号余额等>
> 			2、多事务关联指标: 多个业务过程混合计算的指标
> 		以上两种场景需要多个事务型事实表[大表]join操作,性能比较低。
> 		采用的同步策略: 增量同步
> 	周期快照型事实表
> 		定义: 以固定的时间周期记录业务过程的事实。
> 		适用场景: 存量型指标、状态型指标
> 		好处: 在业务表中一般都有现成的表会直接记录用户账号余额等,所以此时可以直接将该表的数据每天全量同步一份,后续需要的时候直接查询即可,不需要多个事务型事实表join得到结果。
> 		设计流程:
> 			1、确定粒度
> 				粒度由周期与维度决定
> 				周期一般每天一次
> 				维度由需求决定
> 			2、确定度量
> 				度量值也有需求决定
> 		采用的同步策略: 全量同步
> 		事实类型
> 			1、可加事实: 表中的度量值与表中所有维度统计都有意义[事务型事实表的事实就是可加事实]
> 			2、半可加事实: 表中的度量值与表中部分维度统计才有意义[周期快照事实表的事实就是半可加事实]
> 			3、不可加事实: 比如比率不中一般会包含多个关键业务过程对应的时间维度,通过这些时间能直接累加,一般需要将不可加事实转成可加事实
> 	累积型快照事实表
> 		定义: 围绕一个业务流程中的多个关键的业务过程设计的就是累积型快照事实表
> 		累积型快照事实表中一般会包含多个关键业务过程对应的时间字段,通过这些时间字段可以知道该流程进行到了哪个业务过程。
> 		适用场景: 用于多事务关联指标
> 		设计流程
> 			1、选择业务过程
> 				选择需要的多个业务过程
> 				一个累积型快照事实表对应多个业务过程
> 			2、确定粒度
> 				尽可能选择最细粒度
> 			3、确定维度
> 				选择多个业务过程的维度信息
> 				然后给每个业务过程额外添加一个时间维度
> 			4、确定度量
> 				选择多个业务过程的度量值
> 		同步策略: 增量同步
> 4、维度建模之维度表
> 	1、定义: 业务过程发生的环境描述信息
> 	2、维度表中一般包含主键与维度属性两种字段
> 	3、维度表的设计
> 		1、确定维度[确定需要创建多少个维度表]
> 			根据事实表需要的维度来创建维度表,一个维度一般创建一个维度表
> 			如果多个事实表都有共同的维度,此时只需要创建一个公共的维度表
> 			如果某个维度表的维度属性很少,此时可以不用创建该维度表,直接将维度属性冗余到事实表中[维度退化]
> 		2、确认主维表与相关维表[确定维度表粒度,主键列]
> 			主维表与相关维表是指业务系统与维度相关联的表
> 			维度表的粒度与主维表的粒度一致
> 		3、确定维度属性[确定维度属性列]
> 			维度表的维度属性一般可以直接从主维表和相关维表直接获取或者通过加工得到。
> 			维度属性获取原则
> 				1、维度属性要尽可能丰富[维度属性越丰富后续能够分析的指标越多]
> 				2、维度属性尽量不要只有编码,要编码和文字共存
> 				3、需要沉淀出通用的维度[避免后续需求的数据重复处理]
> 	4、维度设计要点
> 		1、规范化与反规范化
> 			规范化: 以范式理论设计表[将一个表拆分为多个表]
> 				优点: 减少数据冗余,增强数据一致性
> 				缺点: 表比较多,查询性能慢
> 			反规范化: 不遵循范式理论设计表[将多个表合成一个表]
> 				优点: 表比较少,查询性能比较高
> 				缺点: 数据冗余
> 			在数仓中设计维度表的时候一般采用反规范化设计
> 			以反规范化设计的维度表称之为星型模型
> 		2、维度变化
> 			维度表的数据是可能改变的
> 			会将维度表分为两类:
> 				全量快照维度表
> 					1、定义: 每天都将mysql表中所有数据同步到hive表一个分区中,hive表每个分区都是当天mysql表中所有数据[快照]
> 					2、优点: 使用简单[如果需要查询最新数据,只需要查询hive表最新分区即可,如果需要查询某天历史数据,只需要历史某天分区即可]
> 					3、缺点: 存在数据冗余[hive表多个分区间可能存在完全一模一样的数据]
> 					4、适用场景: 表数据量不大或者表数据量大但是变化频率比较高
> 					5、同步策略: 全量同步
> 				拉链表
> 					1、定义: 能够记录数据生命周期的表称之为拉链表
> 						拉链表中每条数据都有生效日期和失效日期两个字段
> 						如果某条数据是最新数据,此时失效日期是一个极大值[9999-12-31]
> 						如果某条数据在mysql发生了修改,那么会将该数据的失效日期改成修改时间的前一天日期。
> 					2、适用场景: 表数据量大并且变化频率比较低
> 					3、优点: hive表中多个分区间不存在重复的数据,能够减少磁盘空间的占用
> 					4、缺点: 适用会稍显麻烦
> 						1、查询最新数据: select .. from .. where 失效日期 = '9999-12-31'
> 						2、查询某天历史数据: select .. from .. where 生效日期<= '指定时间' and 失效日期>='指定时间'
> 		3、多值维度
> 			定义: 事实表一行数据能够与维度表多条数据相关联
> 			解决方案:
> 				1、尝试降低事实表的粒度
> 				2、如果事实表粒度不能降低
> 					1、维度外键的个数固定,可以额外添加多个字段,一个字段保存一个维度外键
> 					2、维度外键的个数不固定,此时只需要一个复杂类型字段,保存多个维度外键
> 		4、多值属性
> 			定义: 维度表中某个维度属性有多个值
> 			解决方案:
> 				1、如果维度属性值个数固定,可以添加多个字段,每个字段保存一个维度属性值
> 				2、如果维度属性值个数不固定,可以通过一个特殊类型的字段保存多个维度属性值
> 5、数仓设计
> 	1、数仓的分层
> 		好处: 将复杂的问题简单化
> 		ODS: 原始数据层[保存是采集的原始数据]
> 		DWD: 数据明细层[保存的各个业务对应的事实表]
> 		DIM: 公共维度层[保存的是各个事实表相关联的维度表]
> 		DWS: 公共汇总层[保存是多个需求都需要的公共的汇总结果]
> 		ADS: 数据应用层[保存是需求指标的统计结果]
> 	2、数仓的构建流程
> 														->构建业务总线矩阵->DWD+DIM
> 		数据调研[业务的调研、需求调研] -> 明确作用域 								->开发->调度
> 														->构建指标体系 -> DWS
> 	3、数据调研
> 		业务调研
> 			熟悉业务过程,能够将所有的业务过程列举出来
> 			熟悉业务过程对表数据的影响
> 		需求调研
> 			能够根据需求找到与之对应的业务过程和事实表
> 	4、明确作用域
> 		就是在每一层对数据做分类,便于数据的管理
> 	5、构建总线矩阵
> 		矩阵的每一行就是一个业务过程
> 		矩阵的每一列就是维度
> 		如果业务过程与维度有关系,在交叉处打上一个勾勾
> 		后续矩阵的每一行对应一张事务型事实表
> 		矩阵的每一列对应一张维度表,如果维度属性很少,可以不用创建表直接冗余在事实表中
> 	6、构建指标体系
> 		原子指标 = 业务过程[事实表] + 度量值 + 聚合逻辑
> 		派生指标 = 原子指标 + 统计周期[取数据的日期范围] + 业务限定[过滤条件] + 统计粒度[分组的字段]
> 		衍生指标 = 一个或者多个派生指标通过复杂的计算而来
> 		派生指标与衍生指标一般是对应具体的需求。
> 		后续分析需求之后,需要将需求拆分为一个个的派生指标,后续会发现多个需求可能需要用到同一个派生指标,此时可以将该派生指标的结果直接保存到DWS,后续直接查询结果即可,减少数据重复处理。
> 		dws中的一张表可以保存业务过程相同、统计周期相同、统计粒度相同的多个派生指标。
> 7、ODS层
> 	建表逻辑:
> 		1、建多少张表: 由采集的数据种类决定
> 		2、内部表还是外部表: 外部表
> 		3、表名的定义: ODS_表名_全量[full]/增量[inc]
> 		4、数据的存储类型
> 			普通文本、json、csv: stored as textfile
> 			parquet/orc: stored as parquet/orc
> 		5、表的字段和类型以及序列化与反序列化
> 			1、普通文本[字段以固定分隔符分割的]: 字段自己定义[如果是mysql数据可以用mysql列名], 列的类型与mysql保存一致即可,序列化与反序列化器用默认的即可
> 			2、json数据: 字段名与json一级属性名保持一致,类型与json属性值的类型保持一致,序列化与反序列化器:ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
> 			3、csv数据: 序列化与反序列化器:ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
> 			4、parquet/orc: 列名与文件中的列名保持一致,序列化器与反序列化器保持默认
> 		6、分区: 一般按天分区
> 		7、表数据的保存位置: 与其他表保存数据的父目录保持一致即可
> 		8、是否压缩: 一般都是会压缩
> 		9、压缩格式: ODS层是保存的原始数据,数据量会比较大,所以一般选用压缩比比较高的压缩格式: gzip\lzo
> 		10、数据从哪里来: HDFS保存的采集数据
> 		11、数据到哪里去: 保存到ODS表当天分区中
> 		12、数据加载方式: load data inpath '..' overwrite into table 表名 partition(字段名=字段值)
> 8、DIM层
> 	建表逻辑:
> 		1、建多少张表: 
> 			由事实表的维度决定[由业务总线矩阵的列决定]
> 			一般总线矩阵的一个列会创建一个维度表,如果维度表的维度属性很少,可以直接将维度属性冗余到事实中。
> 		2、内部表还是外部表: 外部表
> 		3、表名的定义: DIM_表名_全量[full]/拉链[zip]
> 		4、数据的存储类型： DIM层查询会比较多,选用列存储格式
> 			parquet/orc: stored as parquet/orc
> 		5、表的字段和类型
> 			表的字段从维度相关的主维表与相关维表直接获取或者通过加工得到[可以直接将主维表和相关维表所有字段全部选择]
> 		6、分区: 一般按天分区
> 		7、表数据的保存位置: 与其他表保存数据的父目录保持一致即可
> 		8、是否压缩: 一般都是会压缩
> 		9、压缩格式: 选用速度快的压缩: snappy
> 		10、数据从哪里来: 
> 			全量: 从ODS相关表当天分区获取
> 			拉链:
> 				首日: 从ODS相关表首日分区获取
> 				每日: 从前一天9999-12-31分区 + 从ODS相关表当天分区获取
> 		11、数据到哪里去: 
> 			全量: 数据加载到DIM表当天分区中
> 			拉链: 
> 				首日: 全部放入9999-12-31分区
> 				每日: 最新数据覆盖写入9999-12-31,当天失效的数据放入前一天日期分区中
> 		12、数据加载方式: 
> 			insert overwrite table ... partition(...) select ... from ods表 where dt='当天'
> 9、DWD层
> 	建表逻辑:
> 		1、建多少张表: 
> 			一个业务过程一张事务型事实表 + 一个业务过程的存量型/状态型指标个数对应一张周期快照事实表[有需求才创建] + 一个业务流程多事务关联指标对应一张累积快照事实表[有需求才创建]
> 		2、内部表还是外部表: 外部表
> 		3、表名的定义: DWD_作用域_表名_事务型[inc]/周期快照型[full]/累积快照型[acc]
> 		4、数据的存储类型： DWD层查询会比较多,选用列存储格式
> 			parquet/orc: stored as parquet/orc
> 		5、表的字段和类型
> 			维度外键 + 冗余的维度 + 度量值
> 		6、分区: 一般按天分区
> 		7、表数据的保存位置: 与其他表保存数据的父目录保持一致即可
> 		8、是否压缩: 一般都是会压缩
> 		9、压缩格式: 选用速度快的压缩: snappy
> 		10、数据从哪里来: 
> 			事务型事实表: 从ODS相关表当天分区获取
> 			周期快照事实表: 从ODS相关表当天分区获取
> 			累积快照事实表: 
> 				首日: 从ODS首日分区获取
> 				每日: 从9999-99-99分区获取 + 从ODS当日分区获取
> 		11、数据到哪里去: 
> 			事务型事实表[每个分区保存的是当天发生的事实]: 
> 				首日: 根据事实发生的时间将数据放入对应分区中
> 				每日: 直接将数据放入当日分区中
> 			周期快照事实表[每个分区保存是mysql表当日全量数据]: 数据写入表当日分区中
> 			累积快照事实表: 完成的流程放入完成时间对应分区中,未完成的流程放入9999-99-99分区
> 		12、数据加载方式: 
> 			insert overwrite table ... partition(...) select ... from ods表 where dt='当天' ..
> 10、DWS层
> 	建表逻辑:
> 		1、建多少张表: 
> 			不固定,一般是一个DWS的表对应多个业务过程相同、统计周期相同、统计粒度相同的多个派生指标
> 			一个业务过程最近一日指标单独对应一张表
> 			一个业务过程最近N日指标单独对应一张表
> 			一个业务过程历史至今指标单独对应一张表
> 		2、内部表还是外部表: 外部表
> 		3、表名的定义: DWS_作用域_粒度_业务过程_1d/nd/td
> 		4、数据的存储类型： DWD层查询会比较多,选用列存储格式
> 			parquet/orc: stored as parquet/orc
> 		5、表的字段和类型
> 			统计粒度、原子指标
> 		6、分区: 一般按天分区
> 		7、表数据的保存位置: 与其他表保存数据的父目录保持一致即可
> 		8、是否压缩: 一般都是会压缩
> 		9、压缩格式: 选用速度快的压缩: snappy
> 		10、数据从哪里来: 
> 			1d:
> 				首日: 取出dwd首日和之前的所有数据
> 				每日: 只需要dwd当日分区数据
> 			nd: 一般从1d表查询n个分区
> 			td: 一般从1d或者dwd获取从开始到当日日期所有数据
> 		11、数据到哪里去: 
> 			1d: 
> 				首日: 每天的统计结果放入当日分区
> 				每日: 将当日统计结果放入当日分区
> 			nd: 放入当日分区
> 			td: 放入当日分区
> 		12、数据加载方式: 
> 			insert overwrite table ... partition(...) select ... from dwd/1d表 where dt=...
> 11、ADS层
> 	建表逻辑:
> 		1、建多少张表: 
> 			一个需求一个表
> 		2、内部表还是外部表: 外部表
> 		3、表名的定义: ADS_作用域_表名
> 		4、数据的存储类型： stored as textfile
> 		5、表的字段和类型
> 			统计日期、统计粒度、周期、指标结果字段
> 		6、分区: 没有分区[ads数据量比较少,分区会产生很多小文件]
> 		7、表数据的保存位置: 与其他表保存数据的父目录保持一致即可
> 		8、是否压缩: 不压缩[ads数据量比较少]
> 		10、数据从哪里来: 
> 			dws/dwd
> 		11、数据到哪里去: 
> 			将结果插入到ads表
> 		12、数据加载方式: 
> 			insert overwrite table ads表
> 			select * from ads表 where 统计日期!='当日'
> 			union all
> 			select ... from dws/dws表 where dt=...

### 1-维度建模

> 维度模型将复杂的业务通过 事实 和 维度 两个概念进行呈现;
>
> 事实通常应对业务过程；【可以概括为一个个不可拆分的行为事件，例如电商交易中的下单，取消订单，付款，退单等，都是业务过程。】
>
> 维度通常应对 业务过程发生时所处的环境
>
> +++

#### **维度建模-事实表**

> +  围绕业务过程创建，是数仓维度建模的核心
> + 【包含与该业务过程有关的维度引用（维度表外键）以及该业务过程的度量（通常是可累加的数字类型字段）。】
> + 特点：事实表通常比较“细长”，即列较少，但行较多，且行的增速快。
> + 三种类型：分别是**事务事实表**、周期快照事实表 和 累积快照事实表
>
> +++
>
> ***事务型事实表***
>
> > 事务事实表用来记录各业务过程，它保存的是各业务过程的**原子操作事件**，即最细粒度的操作事件。**粒度是指事实表中一行数据所表达的业务细节程度**。
> >
> > 事务型事实表可用于分析与各业务过程相关的各项统计指标，由于其保存了最细粒度的记录，可以提供最大限度的灵活性，可以支持无法预期的各种细节层次的统计需求。
> >
> > 设计流程：【四个步骤】
> >
> > + ***选择业务过程*** ：可以概括为一个个不可拆分的行为事件，例如 电商交易中的下单，取消订单，付款，退单等，都是业务过程。通常情况下，一个业务过程对应一张事务型事实表。
> > + ***确定粒度*** ：尽可能选择最细粒度，以此满足各种程度的需求，例如 订单事实表中一行数据表示的是一个订单中的一个商品项。
> > + ***确定维度*** ：确定每张事务型事实表 相关的维度有哪些 【确定维度时应尽量多的选择与业务过程相关的环境信息】
> > + ***确定事实*** ：指每个业务过程的度量值（通常是可累加的数字类型的值，如金额、次数、个数、件数等）
> >
> > +++
> >
> > 不足：对于某些特定类型的需求，其逻辑可能会比较复杂，或者效率会比较低下，例如 存量型指标、多事务关联统计
> >
> > + 存量型指标
> >
> >   > 例如商品库存，账户余额等【各分类商品购物车存量top3 .....】
> >   >
> >   > + 聚合两个大表得到结果，效率低
> >
> >   ![image-20220808104229552](http://ybll.vip/md-imgs/202208081042633.png)
> >
> > + 多事务关联统计
> >
> >   > 【统计最近30天，用户下单到支付的时间间隔的平均值】
> >   >
> >   > + 统计思路应该是找到下单事务事实表和支付事务事实表，过滤出最近30天的记录，然后按照订单id对两张事实表进行关联，之后用支付时间减去下单时间，然后再求平均值。
> >   > + 逻辑上虽然并不复杂，但是其效率较低，应为下单事务事实表和支付事务事实表均为大表，大表join大表的操作应尽量避免
> >   
> >   ![image-20220808141440395](http://ybll.vip/md-imgs/202208081414486.png)
>
> +++
>
> ***周期型快照事实表***
>
> > + 周期快照事实表以具有规律性的、可预见的时间间隔来记录事实，
> >
> > + 主要用于分析一些存量型（例如商品库存，账户余额）或者状态型（空气温度，行驶速度）指标。
> >
> > +++
> >
> > 设计流程：【两个步骤】
> >
> > + 确定粒度
> >
> >   粒度可由 ***`采样周期`*** 和 ***`维度`*** 描述，确定二者后即可确定粒度
> >
> >   【统计每个仓库中每种商品的库存】
> >
> >   + 周期：每日 ；维度：仓库，商品
> >   + 该周期型快照事实表的粒度：每日-仓库-商品
> >
> > + 确定事实
> >
> >   根据统计指标决定
> >
> >   【统计每个仓库中每种商品的库存】
> >
> >   + 事实(度量值）：商品库存
> >
> > +++
> >
> > 周期型快照事实表的事实类型（度量值类型）
> >
> > ![image-20220808164809202](http://ybll.vip/md-imgs/202208081648312.png)
>
> +++
>
> ***累计型快照事实表***
>
> > + ![image-20220808165048301](http://ybll.vip/md-imgs/202208081650369.png)
> > + 基于一个业务流程中的多个关键业务过程联合处理而构建的事实表，如交易流程中的下单、支付、发货、确认收货业务过程。
> > + 累积型快照事实表通常具有多个日期字段，每个日期对应业务流程中的一个关键业务过程（里程碑）
> > + 累积型快照事实表主要用于分析业务过程（里程碑）之间的时间间隔等需求，例如前文提到的用户下单到支付的平均时间间隔，使用累积型快照事实表进行统计，就能避免两个事务事实表的关联操作，从而变得十分简单高效。
> >
> > +++
> >
> > 设计流程 【四个步骤】
> >
> > + ***选择业务过程*** ：选择一个业务需求中需要关联分析的多个关键业务过程，***多个业务过程对应一张累积型快照事实表***。
> > + ***确定粒度*** ：精确定义每行数据表示的是什么，尽量选择最小粒度。
> > + ***确定维度*** ：选择与各业务过程相关的维度，需要注意的是，***每各业务过程均需要一个日期维度。***
> > + ***确定事实*** ：选择各业务过程的度量值。
>
> +++

+++

#### 维度建模-维度表

> + 维度表是维度建模的基础和灵魂
> + 事实表紧紧围绕业务过程进行设计，而维度表则围绕业务过程所处的环境进行设计
> + 维度表主要包含一个主键和各种维度字段，维度字段称为维度属性。
>
> +++
>
> 维度表设计步骤 【三个步骤】
>
> + 确定维度（维度表）
>
>   > + 设计事实表时，已经确定了与每个事实表相关的维度，理论上每个相关维度均需对应一张维度表
>   > + 可能存在多个事实表与同一个维度都相关的情况，这种情况需保证维度的唯一性，即只创建一张维度表。
>   > + 如果某些维度表的维度属性很少，例如只有一个名称，则可不创建该维度表，而把该表的维度属性直接增加到与之相关的事实表中，这个操作称为**维度退化**
>
> + 确定主维表 和 相关维表
>
>   > + 此处的主维表和相关维表均指**业务系统**中与某维度相关的表。
>   > + 例如业务系统中与商品相关的表有sku_info，spu_info，base_trademark，base_category3，base_category2，base_category1等
>   > + 其中sku_info就称为商品维度的主维表，其余表称为商品维度的相关维表
>   > + 维度表的粒度通常与主维表相同。
>
> + 确定维度属性
>
>   > + 确定维度属性即确定维度表字段
>   > + 维度属性主要来自于业务系统中与**该维度对应的主维表和相关维表**
>   > + 维度属性可直接从主维表或相关维表中选择，也可通过进一步加工得到。
>   >
>   > +++
>   >
>   > 确定维度属性时，需要遵循以下要求：
>   >
>   > 1. **尽可能生成丰富的维度属性**
>   >
>   >    维度属性是后续做分析统计时的查询约束条件、分组字段的基本来源，是数据易用性的关键。维度属性的丰富程度直接影响到数据模型能够支持的指标的丰富程度。
>   >
>   > 2. **尽量不使用编码，而使用明确的文字说明，一般可以编码和文字共存。**
>   >
>   > 3. **尽量沉淀出通用的维度属性**
>   >
>   >    有些维度属性的获取需要进行比较复杂的逻辑处理，例如需要通过多个字段拼接得到。为避免后续每次使用时的重复处理，可将这些维度属性沉淀到维度表中。
>
> +++
>
> 设计要点
>
> + 规范化和反规范化
>
>   > + **规范化**是指使用一系列范式设计数据库的过程，其目的是**减少数据冗余，增强数据的一致性**。通常情况下，规范化之后，一张表的字段会拆分到多张表
>   > + **反规范化**是指将多张表的数据冗余到一张表，其目的是**减少join操作，提高查询性能**
>   > + 在设计维度表时，如果对其进行规范化，得到的维度模型称为**雪花模型**，如果对其进行反规范化，得到的模型称为***星型模型***。
>
> + 维度变化
>
>   > + 维度属性通常不是静态的，而是会随时间变化的
>   > + 数据仓库的一个重要特点就是反映历史的变化，所以如何保存维度的历史状态是维度设计的重要工作之一
>   > + 保存维度数据的历史状态，通常有以下两种做法，分别是全量快照表和拉链表
>   >
>   > +++
>   >
>   > **全量快照表**：【全量同步】适合 表数据量不大，或者表数据量很大但是变化频率也很大
>   >
>   > + 离线数据仓库的计算周期通常为每天一次，所以可以每天保存一份全量的维度数据。这种方式的优点和缺点都很明显
>   >
>   > + 优点是简单而有效，开发和维护成本低，且方便理解和使用
>   >
>   > + 缺点是浪费存储空间，尤其是当数据的变化比例比较低时
>   >
>   > **拉链表**：【增量同步】 适合 数据会发生变化，但是变化频率并不高的维度
>   >
>   > + 拉链表的意义就在于能够更加高效的保存维度信息的历史状态
>   > + 拉链表记录每条信息的生命周期（生效开始日期，结束日期）
>   > + 使用：生效开始日期 <= 某个日期 and 生效结束日期 >= 某个日期
>
> + 多值维度
>
>   > + 事实表中，某条记录在维度表中有多条记录与之对应，这称为多值维度（订单表一条记录为一个订单，一个订单可能包含多个sku商品）
>   > + 解决一：降低事实表的粒度，如将一个订单降低为一个订单中的一个商品项
>   > + 解决二：在事实表中采用多字段保存其对应的维度值，每个字段保存一个维度id（只适用于多值维度个数固定的情况）
>   > + 或者使用一个复杂类型作为字段类型
>
> + 多值属性
>
>   > + 维度表中，某个属性同时有多个值，称为多值属性（商品维度的平台属性和销售属性，每个商品都有多个属性值）
>   > + 第一种：将多值属性放到一个字段，该字段内容为key1:value1，key2:value2的形式，例如一个手机商品的平台属性值为“品牌:华为，系统:鸿蒙，CPU:麒麟990”
>   > + 第二种：将多值属性放到多个字段，每个字段对应一个属性。这种方案只适用于多值属性个数固定的情况。



+++



### 2-数据仓库设计

> ***数据仓库分层规划***
>
> + 优秀可靠的数仓体系，需要良好的数据分层结构。合理的分层，能够使数据体系更加清晰，使复杂问题得以简化。以下是该项目的分层规划
> + ![image-20220808155505929](http://ybll.vip/md-imgs/202208081555055.png)

+++

#### 2.1 数据仓库构建流程

![image-20220809112034243](http://ybll.vip/md-imgs/202208091120393.png)

##### 数据调研



##### 明确数据域

+ 数据仓库模型设计除横向的分层外，通常也需要根据业务情况进行纵向划分数据域。划分数据域的意义是**便于数据的管理和应用**。

+ 通常可以根据业务过程或者部门进行划分，本项目根据业务过程进行划分，需要注意的是**一个业务过程只能属于一个数据域。**

+ 本项目所有业务过程及数据域划分如下：

  | **数据域** | **业务过程**                                                 |
  | ---------- | ------------------------------------------------------------ |
  | **交易域** | 加购物车、减购物车、下单、支付完成、确认收货、退单、退款完成 |
  | **流量域** | 页面浏览、启动应用、动作、曝光、错误                         |
  | **用户域** | 注册、登录                                                   |
  | **互动域** | 收藏、评价                                                   |
  | **工具域** | 优惠券领取、优惠券使用（下单）、优惠券使用（支付）           |



##### 构建业务中线矩阵

> + 业务总线矩阵中包含维度模型所需的所有事实（业务过程）以及维度，以及各业务过程与各维度的关系。
> + 矩阵的行是一个个业务过程
> + 矩阵的列是一个个的维度
> + 行列的交点表示业务过程与维度的关系。
> + **总线矩阵中通常只包含事务型事实表，另外两种类型的事实表需单独设计。**
> + 后续的**DWD层**以及**DIM层**的搭建需参考业务总线矩阵

![image-20220809190357388](http://ybll.vip/md-imgs/202208091903503.png)

![image-20220809190625114](http://ybll.vip/md-imgs/202208091906199.png)

![image-20220809190651871](http://ybll.vip/md-imgs/202208091906976.png)

##### 明确统计指标

+ 原子指标

  > + 基于某一**业务过程**的**度量值**，是业务定义中不可再拆解的指标，原子指标的核心功能就是对指标的**聚合逻辑**进行了定义。
  >
  > + 原子指标包含三要素，分别是**业务过程、度量值、聚合逻辑。**
  >
  > + 【**订单总额**就是一个典型的原子指标，其中的业务过程为用户下单、度量值为订单金额，聚合逻辑为sum()求和】
  >
  > + 原子指标只是用来辅助定义指标一个概念，**通常不会有实际统计需求与之对应**。

+ 派生指标

  > + 基于原子指标，通常会对应实际的统计需求
  >
  > ![image-20220809191354944](http://ybll.vip/md-imgs/202208091913046.png)

+ 衍生指标

  > + 一个或多个派生指标的基础上，通过各种逻辑运算复合而成的 【比率、比例等类型的指标】
  > + 衍生指标也会对应实际的统计需求
  >
  > ![image-20220809191535905](http://ybll.vip/md-imgs/202208091915990.png)

+++

+ 指标体系对于数仓建模的意义

  > + 绝大多数的统计需求，都可以使用原子指标、派生指标以及衍生指标这套标准去定义
  > + 同时能够发现这些统计需求都直接的或间接的对应一个或者是多个派生指标
  > + 当统计需求足够多时，必然会出现部分统计需求对应的派生指标相同的情况
  > + 就可以考虑将这些公共的派生指标保存下来，这样做的主要目的就是减少重复计算，提高数据的复用性
  > + 公共的派生指标统一保存在数据仓库的DWS层。因此DWS层设计，就可以参考根据现有的统计需求整理出的派生指标。
  >
  > +++
  >
  > 本项目所抽取出来的派生指标：
  >
  > ![image-20220809192128356](http://ybll.vip/md-imgs/202208091921499.png)

  +++

##### 维度模型设计

+ 维度模型的设计参照上述得到的业务总线矩阵
+ 事实表存储在DWD层
+ 维度表存储在DIM层

##### 汇总模型设计

+ 汇总模型的设计参考上述整理出的指标体系（主要是派生指标）即可
+ 汇总表与派生指标的对应关系是，**一张汇总**表通常包含**业务过程相同、统计周期相同、统计粒度相同的多个派生指标**
+ 汇总表与事实表的对应关系是？

+++

+++

### 3-数据仓库环境准备

#### 3.1 Hive On Spark 搭建

> + Hive引擎包括：默认MR、tez、spark
>
> + Hive on Spark : **(bin/hive sql语句…)   **Hive既作为存储元数据又负责SQL的解析优化，语法是HQL语法，执行引擎是Spark，Spark负责转换成RDD执行
>
> + Spark on Hive : **(bin/spark-sql sql)**     Hive只作为存储元数据，Spark负责SQL解析优化，语法是Spark SQL语法，Spark负责转换成RDD执行。
>
>   +++
>
> + 兼容性说明
>
>   官网下载的Hive3.1.2和Spark3.0.0默认是不兼容的。因为Hive3.1.2支持的Spark版本是2.4.5，所以需要我们重新编译Hive3.1.2版本。
>
>   编译步骤：官网下载Hive3.1.2源码，修改pom文件中引用的Spark版本为3.0.0，如果编译通过，直接打包获取jar包。如果报错，就根据提示，修改相关方法，直到不报错，打包获取jar包。

+++

1.  配置spark 环境变量：hive需要执行 bin/spark-submit

2.  hive中创建 spark配置文件：`spark-defaults.conf`

   ```properties
   #添加如下配置，原spark-yarn安装包下的该配置文件的配置需要注释掉
   spark.master               yarn
   spark.eventLog.enabled     true
   spark.eventLog.dir         hdfs://hadoop102:8020/spark-history
   spark.executor.memory      1g
   spark.driver.memory		   1g
   #spark.eventLog.dir（用于存储历史日志） 配置的目录需要手动创建，且与原spark-yarn对应
   ```

3. 向 HDFS 上传 Spark纯净版 jar包，用于Spark执行任务时，直接使用

   ```bash
   tar -zxvf /opt/software/spark-3.0.0-bin-without-hadoop.tgz
   hadoop fs -mkdir /sparkjars
   hadoop fs -put spark-3.0.0-bin-without-hadoop/jars/* /sparkjars
   #jars包中，移除了hadoop,hive等依赖jar包，防止版本冲突
   ```

4. 修改 hive-site.xml 文件

   ```xml
   <?xml version="1.0"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   <configuration>
       <!-- jdbc连接的URL -->
       <property>
           <name>javax.jdo.option.ConnectionURL</name>
           <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8</value>
       </property>
       <!-- jdbc连接的Driver-->
       <property>
           <name>javax.jdo.option.ConnectionDriverName</name>
           <value>com.mysql.jdbc.Driver</value>
       </property>
       <!-- jdbc连接的username-->
       <property>
           <name>javax.jdo.option.ConnectionUserName</name>
           <value>root</value>
       </property>
       <!-- jdbc连接的password -->
       <property>
           <name>javax.jdo.option.ConnectionPassword</name>
           <value>1234</value>
       </property>
       <!-- Hive默认在HDFS的工作目录 -->
       <property>
           <name>hive.metastore.warehouse.dir</name>
           <value>/user/hive/warehouse</value>
       </property>
       <!-- Hive元数据存储的验证 -->
       <property>
           <name>hive.metastore.schema.verification</name>
           <value>false</value>
       </property>
       <!-- 元数据存储授权  -->
       <property>
           <name>hive.metastore.event.db.notification.api.auth</name>
           <value>false</value>
       </property>
       <property>
           <name>javax.jdo.option.ConnectionURL</name>
           <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8</value>
       </property>
       <property>
           <name>hive.cli.print.header</name>
           <value>true</value>
       </property>
       <property>
           <name>hive.cli.print.current.db</name>
           <value>true</value>
       </property>
       <!-- 指定hiveserver2连接的host -->
       <property>
           <name>hive.server2.thrift.bind.host</name>
           <value>hadoop102</value>
       </property>
       <!-- 指定hiveserver2连接的端口号 -->
       <property>
           <name>hive.server2.thrift.port</name>
           <value>10000</value>
       </property>
   
       <property>
           <name>hive.querylog.location</name>
           <value>/opt/module/hive/log</value>
           <description>Location of Hive run time structured log file</description>
       </property>
       <!--Spark依赖位置（注意：端口号8020必须和namenode的端口号一致）-->
       <property>
           <name>spark.yarn.jars</name>
           <value>hdfs://hadoop102:8020/sparkjars/*</value>
       </property>
   
       <!--Hive执行引擎-->
       <property>
           <name>hive.execution.engine</name>
           <value>spark</value>
       </property>
       <!--指定元数据序列化与反序列化器-->
       <property>
           <name>metastore.storage.schema.reader.impl</name>
           <value>org.apache.hadoop.hive.metastore.SerDeStorageSchemaReader</value>
       </property>
   
       <!-- 使用元数据服务的方式访问Hive时，添加如下配置 -->
       <!-- 指定存储元数据要连接的地址 -->
       <!--    <property>-->
       <!--        <name>hive.metastore.uris</name>-->
       <!--        <value>thrift://hadoop102:9083</value>-->
       <!--    </property>-->
   </configuration>
   ```

5. 测试

   ```bash
   bin/hive #启动hive客户端
   
   hive (default)> create table student(id int, name string); #创建一张表
   hive (default)> insert into table student values(1,'abc'); #插入数据
   ```

6. 调优

   + **增加ApplicationMaster**资源比例

     容量调度器对每个资源队列中同时运行的Application Master占用的资源进行了限制，该限制通过yarn.scheduler.capacity.maximum-am-resource-percent参数实现，其默认值是0.1，表示每个资源队列上Application Master最多可使用的资源为该队列总资源的10%，目的是防止大部分资源都被Application Master占用，而导致Map/Reduce Task无法执行。

     生产环境该参数可使用默认值。但学习环境，集群资源总数很少，如果只分配10%的资源给Application Master，则可能出现，同一时刻只能运行一个Job的情况，因为一个Application Master使用的资源就可能已经达到10%的上限了。故此处可将该值适当调大。

     （1）在hadoop102的/opt/module/hadoop-3.1.3/etc/hadoop/capacity-scheduler.xml文件中**修改**如下参数值

   ```xml
   <property>
       <name>yarn.scheduler.capacity.maximum-am-resource-percent</name>
       <value>0.8</value>
   </property>
   ```

   + `hive-env.sh` 修改配置

     ```shell
     export HADOOP_HEAPSIZE=2048
     ```



+++

#### 3.2 模拟数据准备

> 日志数据
>
> 1. 修改102、103生成数据的配置文件里面的日期: mock.date="2020-06-14"
> 2. 102、103执行jar包模拟生成2020-6-14的日志数据
> 3. 修改102、103生成数据的配置文件里面的日期: mock.date="2020-06-15"
> 4. 102、103执行jar包模拟生成2020-6-15的日志数据
> 5. 修改102、103生成数据的配置文件里面的日期: mock.date="2020-06-16"
> 6. 102、103执行jar包模拟生成2020-6-16的日志数据
>
> +++
>
> 业务数据
> + 模拟生成历史的业务数据[ 2020-6-10、2020-6-11、2020-6-12、2020-6-13、2020-6-14 ]
> 	+ 修改生成数据的配置文件里面的日期: mock.date=2020-06-10, mock.clear=1,mock.clear.user=1	
> 	+ 执行jar包模拟生成2020-06-10的业务数据	
> 	+ 修改生成数据的配置文件里面的日期: mock.date=2020-06-11, mock.clear=0,mock.clear.user=0
> 	+ 执行jar包模拟生成2020-06-11的业务数据
> 	+ 修改生成数据的配置文件里面的日期: mock.date=2020-06-12
> 	+ 执行jar包模拟生成2020-06-12的业务数据
> 	+ 修改生成数据的配置文件里面的日期: mock.date=2020-06-13
> 	+ 执行jar包模拟生成2020-06-13的业务数据
> 	+ 修改生成数据的配置文件里面的日期: mock.date=2020-06-14  
> 	+ 执行jar包模拟生成2020-06-14的业务数据
> 	
> + 2020-06-14采集数据
> 
>     + 全量同步: datax
> 
>     + 增量同步: maxwell首日同步[同步表中所有数据]
> 
>       1. 修改maxwell配置中的时间为mock_date=2020-06-14
> 
>       2. 删除maxwell断点记录
> 
>          ```mysql
>           drop table maxwell.bootstrap;
>           drop table maxwell.columns;
>           drop table maxwell.databases;
>           drop table maxwell.heartbeats;
>           drop table maxwell.positions;
>           drop table maxwell.schemas;
>           drop table maxwell.tables;
>          ```
> 
>       3. 启动maxwell
> 
>       4. 通过脚本采集首日数据
> 
>+ 2020-6-15采集
> 	
> 	+ 停止maxwell
> 	+ 修改生成数据的配置文件里面的日期: mock.date=2020-06-15
> 	+ 执行jar包模拟生成2020-06-15的业务数据
> 	+ 修改maxwell配置中的时间为mock_date=2020-06-15
> 	+ 全量同步: datax
> 	+ 增量同步: 启动maxwell	
> 	
> + 2020-6-16采集	
> 	
> 	+ 停止maxwell
> 	+ 修改生成数据的配置文件里面的日期: mock.date=2020-06-16
> 	+ 执行jar包模拟生成2020-06-16的业务数据
> 	+ 修改maxwell配置中的时间为mock_date=2020-06-16
> 	+ 全量同步: datax
> 	+ 增量同步: 启动maxwell

+++

### MySQL表关联图

![image-20220815165203928](http://ybll.vip/md-imgs/202208151652475.png)

+++

![image-20220815165240933](http://ybll.vip/md-imgs/202208151652028.png)

### 4- ODS层搭建

#### 4.1 sql代码

```sql
--ODS建表逻辑
---1、建多少张表: 数据有多少种,建多少张表【业务数据[28] + 日志数据[1] = 29】
---2、内部表还是外部表: 公司一般采用外部表
---------内部表: 会删除表元数据以及表HDFS数据
---------外部表: 只删除元数据
---3、表名如何定义: ods_表名_全量[full]/增量[inc]
---4、字段以及字段类型以及存储格式
-------采集的数据保存到HDFS的时候是文本
-------------------存储格式: STORED AS TEXTFILE
-------------------如果文本中数据字段之间是固定分隔符分割的:
-----------------------------列名自定义[或者直接用mysql列名],列的类型可以全部string[也可以与mysql的列名保存一致]
-----------------------------row format delimited fields terminated by '分隔符'
-------------------如果文本中数据是json:
----------------------------1、只弄一个字段,字段类型是string,后续可以使用get_json_object取出json中的值。
----------------------------2、需要指定多个列,列名必须是json的一级属性名,列的类型必须与json的值类型保持一致。
---------------------------------ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
-------------------如果文本中数据是csv
----------------------------指定 ROW FORMAT SERDE'org.apache.hadoop.hive.serde2.OpenCSVSerde'
-------采集的数据保存到HDFS的时候是parquet/orc
-------------------存储格式: STORED AS PARQUET/ORC
-------------------列名必须与parquet/orc中的列名保持一致，类型也一致
---5、是否分区: 工作中一般都是分区表
---6、按照什么分区: 工作中一般按天分区
---7、指定表数据的HDFS存储路径: 数据存储的父目录与其他表保持一致即可
---8、是否压缩: 要
---9、压缩格式: ods层是原始数据, 一般采用压缩比比较大的压缩格式: gzip
---10、数据从哪里来: HDFS
---11、数据加载到哪里去: 一个目录的数据一般是加载到一个分区中
---12、数据的加载方式: load data inpath 'HDFS路径' overwrite into table 表名 partition (分区字段名='分区字段值')

---json数据建表案例
--方式1
drop table person;
create table person(
    line string
);

select get_json_object(line, "$.id"),get_json_object(line, "$.name"),get_json_object(line,"$.xingqu[0]") from person;
--get_json_object函数取值:
--$: 代表整个json字符串
--. : 用于获取子属性的值  $.属性名
--[] : 用于获取json中数组的值 $.属性名[角标]
desc function extended get_json_object;
--方式2:
drop table person;
create table person(
    id bigint,
    name string,
    xingqu array<STRING>
)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe';


--hive 复杂数据类型
--array
----类型定义: array<元素类型>
----构造对象: array(元素值,...)
----取值方式: 数组[角标]
--struct
----类型定义: struct<属性名:类型,。。。。>
----构造对象: named_struct("属性名",属性值,"属性名",属性值,...)
----取值方式: struct对象.属性名
--map
----类型定义: map<k的类型,v的类型>
----构建对象: map(k,v,k,v,....)
----取值方式:
--------取出所有的key: map_keys(map对象)
--------取出所有的value值: map_keys(map对象)
--------根据key取出value值: map对象["key"]


--数组对象构建
select array(1,2,3,4,5)[1];
--struct对象构建
---此时属性默认
select struct(20,30,40,50);
---手动指定属性名和属性值
select named_struct("name","lisi","age",20).age;

--map对象构建
select map("name","lisi","age",20);

select map_keys( map("name","lisi","age",20));
select map_values( map("name","lisi","age",20));
select map("name","lisi","age",20)["name"];
```

```sql
--------------------------日志数据ods建表
drop table ods_log_inc;
create external table if not exists ods_log_inc(
    common struct<ar:string,ba:string,ch:string,is_new:string,md:string,mid:string,os:string,uid:string,vc:string>,
    ts bigint,
    actions array<struct<action_id:string,item:string,item_type:string,ts:bigint>>,
    displays array<struct<display_type:string,item:string,item_type:string,`order`:bigint,pos_id:bigint>>,
    `start` struct<entry:string,loading_time:bigint,open_ad_id:bigint,open_ad_ms:bigint,open_ad_skip_ms:bigint>,
    err struct<error_code:string,msg:string>
) partitioned by (dt string)
row format SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
stored as textfile
location '/xx/yy/ods_log_inc';
--加载数据
load data inpath '/origin_data/gmall/log/topic_log/2020-06-14/*' overwrite into table ods_log_inc partition (dt='2020-06-14')


----------------------------全量数据ODS建表
drop table ods_activity_info_full;
create external table if not exists ods_activity_info_full(
    id bigint,
    activity_name string,
    activity_type string,
    activity_desc string,
    start_time string,
    end_time string,
    create_time string
) partitioned by (dt string)
row format delimited fields terminated by '\t'
    NULL DEFINED AS '' --datax导入mysql的数据到HDFS的时候,mysql字段的值如果是null,保存到HDFS的时候是空字符串,hive的字段如果是null值,保存到HDFS的时候以\N保存
stored as textfile
location '/xx/yy/ods_activity_info_full';

load data inpath '/origin_data/gmall/db/activity_info_full/2020-06-14/*' overwrite into table ods_activity_info_full partition (dt='2020-06-14');

-----------------------------------增量表建表
create external table if not exists ods_comment_info_inc(
    `type` string,
    ts bigint,
    data struct<id:bigint,user_id:bigint,nick_name:string,head_img:string,sku_id:bigint,spu_id:bigint,order_id:bigint,
                appraise:string,comment_txt:string,create_time:string,operate_time:string>
)
partitioned by (dt string)
row format SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
stored as textfile
location '/xx/yy/ods_comment_info_inc';

load data inpath '/origin_data/gmall/db/comment_info_inc/2020-06-14/*' overwrite into table ods_comment_info_inc partition (dt='2020-06-14');




DROP TABLE IF EXISTS ods_log_inc;
CREATE EXTERNAL TABLE ods_log_inc
(
    `common`   STRUCT<ar :STRING,ba :STRING,ch :STRING,is_new :STRING,md :STRING,mid :STRING,os :STRING,uid :STRING,vc
                      :STRING> COMMENT '公共信息',
    `page`     STRUCT<during_time :STRING,item :STRING,item_type :STRING,last_page_id :STRING,page_id
                      :STRING,source_type :STRING> COMMENT '页面信息',
    `actions`  ARRAY<STRUCT<action_id:STRING,item:STRING,item_type:STRING,ts:BIGINT>> COMMENT '动作信息',
    `displays` ARRAY<STRUCT<display_type :STRING,item :STRING,item_type :STRING,`order` :STRING,pos_id
                            :STRING>> COMMENT '曝光信息',
    `start`    STRUCT<entry :STRING,loading_time :BIGINT,open_ad_id :BIGINT,open_ad_ms :BIGINT,open_ad_skip_ms
                      :BIGINT> COMMENT '启动信息',
    `err`      STRUCT<error_code:BIGINT,msg:STRING> COMMENT '错误信息',
    `ts`       BIGINT  COMMENT '时间戳'
) COMMENT '活动信息表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_log_inc/';

--7.2 业务表
--7.2.1 活动信息表（全量表）
DROP TABLE IF EXISTS ods_activity_info_full;
CREATE EXTERNAL TABLE ods_activity_info_full
(
    `id`            STRING COMMENT '活动id',
    `activity_name` STRING COMMENT '活动名称',
    `activity_type` STRING COMMENT '活动类型',
    `activity_desc` STRING COMMENT '活动描述',
    `start_time`    STRING COMMENT '开始时间',
    `end_time`      STRING COMMENT '结束时间',
    `create_time`   STRING COMMENT '创建时间'
) COMMENT '活动信息表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_activity_info_full/';
--7.2.2 活动规则表（全量表）
DROP TABLE IF EXISTS ods_activity_rule_full;
CREATE EXTERNAL TABLE ods_activity_rule_full
(
    `id`               STRING COMMENT '编号',
    `activity_id`      STRING COMMENT '类型',
    `activity_type`    STRING COMMENT '活动类型',
    `condition_amount` DECIMAL(16, 2) COMMENT '满减金额',
    `condition_num`    BIGINT COMMENT '满减件数',
    `benefit_amount`   DECIMAL(16, 2) COMMENT '优惠金额',
    `benefit_discount` DECIMAL(16, 2) COMMENT '优惠折扣',
    `benefit_level`    STRING COMMENT '优惠级别'
) COMMENT '活动规则表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_activity_rule_full/';
--7.2.3 一级品类表（全量表）
DROP TABLE IF EXISTS ods_base_category1_full;
CREATE EXTERNAL TABLE ods_base_category1_full
(
    `id`   STRING COMMENT '编号',
    `name` STRING COMMENT '分类名称'
) COMMENT '一级品类表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_base_category1_full/';
--7.2.4 二级品类表（全量表）
DROP TABLE IF EXISTS ods_base_category2_full;
CREATE EXTERNAL TABLE ods_base_category2_full
(
    `id`           STRING COMMENT '编号',
    `name`         STRING COMMENT '二级分类名称',
    `category1_id` STRING COMMENT '一级分类编号'
) COMMENT '二级品类表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_base_category2_full/';
--7.2.5 三级品类表（全量表）
DROP TABLE IF EXISTS ods_base_category3_full;
CREATE EXTERNAL TABLE ods_base_category3_full
(
    `id`           STRING COMMENT '编号',
    `name`         STRING COMMENT '三级分类名称',
    `category2_id` STRING COMMENT '二级分类编号'
) COMMENT '三级品类表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_base_category3_full/';
--7.2.6 编码字典表（全量表）
DROP TABLE IF EXISTS ods_base_dic_full;
CREATE EXTERNAL TABLE ods_base_dic_full
(
    `dic_code`     STRING COMMENT '编号',
    `dic_name`     STRING COMMENT '编码名称',
    `parent_code`  STRING COMMENT '父编号',
    `create_time`  STRING COMMENT '创建日期',
    `operate_time` STRING COMMENT '修改日期'
) COMMENT '编码字典表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_base_dic_full/';
--7.2.--7.省份表（全量表）
DROP TABLE IF EXISTS ods_base_province_full;
CREATE EXTERNAL TABLE ods_base_province_full
(
    `id`         STRING COMMENT '编号',
    `name`       STRING COMMENT '省份名称',
    `region_id`  STRING COMMENT '地区ID',
    `area_code`  STRING COMMENT '地区编码',
    `iso_code`   STRING COMMENT '旧版ISO-3166-2编码，供可视化使用',
    `iso_3166_2` STRING COMMENT '新版IOS-3166-2编码，供可视化使用'
) COMMENT '省份表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_base_province_full/';
--7.2.8 地区表（全量表）
DROP TABLE IF EXISTS ods_base_region_full;
CREATE EXTERNAL TABLE ods_base_region_full
(
    `id`          STRING COMMENT '编号',
    `region_name` STRING COMMENT '地区名称'
) COMMENT '地区表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_base_region_full/';
--7.2.9 品牌表（全量表）
DROP TABLE IF EXISTS ods_base_trademark_full;
CREATE EXTERNAL TABLE ods_base_trademark_full
(
    `id`       STRING COMMENT '编号',
    `tm_name`  STRING COMMENT '品牌名称',
    `logo_url` STRING COMMENT '品牌logo的图片路径'
) COMMENT '品牌表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_base_trademark_full/';
--7.2.10 购物车表（全量表）
DROP TABLE IF EXISTS ods_cart_info_full;
CREATE EXTERNAL TABLE ods_cart_info_full
(
    `id`           STRING COMMENT '编号',
    `user_id`      STRING COMMENT '用户id',
    `sku_id`       STRING COMMENT 'sku_id',
    `cart_price`   DECIMAL(16, 2) COMMENT '放入购物车时价格',
    `sku_num`      BIGINT COMMENT '数量',
    `img_url`      BIGINT COMMENT '商品图片地址',
    `sku_name`     STRING COMMENT 'sku名称 (冗余)',
    `is_checked`   STRING COMMENT '是否被选中',
    `create_time`  STRING COMMENT '创建时间',
    `operate_time` STRING COMMENT '修改时间',
    `is_ordered`   STRING COMMENT '是否已经下单',
    `order_time`   STRING COMMENT '下单时间',
    `source_type`  STRING COMMENT '来源类型',
    `source_id`    STRING COMMENT '来源编号'
) COMMENT '购物车全量表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_cart_info_full/';
--7.2.11 优惠券信息表（全量表）
DROP TABLE IF EXISTS ods_coupon_info_full;
CREATE EXTERNAL TABLE ods_coupon_info_full
(
    `id`               STRING COMMENT '购物券编号',
    `coupon_name`      STRING COMMENT '购物券名称',
    `coupon_type`      STRING COMMENT '购物券类型 1 现金券 2 折扣券 3 满减券 4 满件打折券',
    `condition_amount` DECIMAL(16, 2) COMMENT '满额数',
    `condition_num`    BIGINT COMMENT '满件数',
    `activity_id`      STRING COMMENT '活动编号',
    `benefit_amount`   DECIMAL(16, 2) COMMENT '减金额',
    `benefit_discount` DECIMAL(16, 2) COMMENT '折扣',
    `create_time`      STRING COMMENT '创建时间',
    `range_type`       STRING COMMENT '范围类型 1、商品 2、品类 3、品牌',
    `limit_num`        BIGINT COMMENT '最多领用次数',
    `taken_count`      BIGINT COMMENT '已领用次数',
    `start_time`       STRING COMMENT '开始领取时间',
    `end_time`         STRING COMMENT '结束领取时间',
    `operate_time`     STRING COMMENT '修改时间',
    `expire_time`      STRING COMMENT '过期时间'
) COMMENT '优惠券信息表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_coupon_info_full/';
--7.2.12 商品平台属性表（全量表）
DROP TABLE IF EXISTS ods_sku_attr_value_full;
CREATE EXTERNAL TABLE ods_sku_attr_value_full
(
    `id`         STRING COMMENT '编号',
    `attr_id`    STRING COMMENT '平台属性ID',
    `value_id`   STRING COMMENT '平台属性值ID',
    `sku_id`     STRING COMMENT '商品ID',
    `attr_name`  STRING COMMENT '平台属性名称',
    `value_name` STRING COMMENT '平台属性值名称'
) COMMENT 'sku平台属性表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_sku_attr_value_full/';
--7.2.13 商品表（全量表）
DROP TABLE IF EXISTS ods_sku_info_full;
CREATE EXTERNAL TABLE ods_sku_info_full
(
    `id`              STRING COMMENT 'skuId',
    `spu_id`          STRING COMMENT 'spuid',
    `price`           DECIMAL(16, 2) COMMENT '价格',
    `sku_name`        STRING COMMENT '商品名称',
    `sku_desc`        STRING COMMENT '商品描述',
    `weight`          DECIMAL(16, 2) COMMENT '重量',
    `tm_id`           STRING COMMENT '品牌id',
    `category3_id`    STRING COMMENT '品类id',
    `sku_default_igm` STRING COMMENT '商品图片地址',
    `is_sale`         STRING COMMENT '是否在售',
    `create_time`     STRING COMMENT '创建时间'
) COMMENT 'SKU商品表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_sku_info_full/';
--7.2.14 商品销售属性值表（全量表）
DROP TABLE IF EXISTS ods_sku_sale_attr_value_full;
CREATE EXTERNAL TABLE ods_sku_sale_attr_value_full
(
    `id`                   STRING COMMENT '编号',
    `sku_id`               STRING COMMENT 'sku_id',
    `spu_id`               STRING COMMENT 'spu_id',
    `sale_attr_value_id`   STRING COMMENT '销售属性值id',
    `sale_attr_id`         STRING COMMENT '销售属性id',
    `sale_attr_name`       STRING COMMENT '销售属性名称',
    `sale_attr_value_name` STRING COMMENT '销售属性值名称'
) COMMENT 'sku销售属性名称'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_sku_sale_attr_value_full/';
--7.2.15 SPU表（全量表）
DROP TABLE IF EXISTS ods_spu_info_full;
CREATE EXTERNAL TABLE ods_spu_info_full
(
    `id`           STRING COMMENT 'spu_id',
    `spu_name`     STRING COMMENT 'spu名称',
    `description`  STRING COMMENT '描述信息',
    `category3_id` STRING COMMENT '品类id',
    `tm_id`        STRING COMMENT '品牌id'
) COMMENT 'SPU商品表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_spu_info_full/';
--7.2.16 购物车表（增量表）
DROP TABLE IF EXISTS ods_cart_info_inc;
CREATE EXTERNAL TABLE ods_cart_info_inc
(
    `type` STRING COMMENT '变动类型',
    `ts`   BIGINT COMMENT '变动时间',
    `data` STRUCT<id :STRING,user_id :STRING,sku_id :STRING,cart_price :DECIMAL(16, 2),sku_num :BIGINT,img_url :STRING,sku_name
                  :STRING,is_checked :STRING,create_time :STRING,operate_time :STRING,is_ordered :STRING,order_time
                  :STRING,source_type :STRING,source_id :STRING> COMMENT '数据',
    `old`  MAP<STRING,STRING> COMMENT '旧值'
) COMMENT '购物车增量表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_cart_info_inc/';
--7.2.1--7.评论表（增量表）
DROP TABLE IF EXISTS ods_comment_info_inc;
CREATE EXTERNAL TABLE ods_comment_info_inc
(
    `type` STRING COMMENT '变动类型',
    `ts`   BIGINT COMMENT '变动时间',
    `data` STRUCT<id :STRING,user_id :STRING,nick_name :STRING,head_img :STRING,sku_id :STRING,spu_id :STRING,order_id
                  :STRING,appraise :STRING,comment_txt :STRING,create_time :STRING,operate_time :STRING> COMMENT '数据',
    `old`  MAP<STRING,STRING> COMMENT '旧值'
) COMMENT '评价表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_comment_info_inc/';
--7.2.18 优惠券领用表（增量表）
DROP TABLE IF EXISTS ods_coupon_use_inc;
CREATE EXTERNAL TABLE ods_coupon_use_inc
(
    `type` STRING COMMENT '变动类型',
    `ts`   BIGINT COMMENT '变动时间',
    `data` STRUCT<id :STRING,coupon_id :STRING,user_id :STRING,order_id :STRING,coupon_status :STRING,get_time :STRING,using_time
                  :STRING,used_time :STRING,expire_time :STRING> COMMENT '数据',
    `old`  MAP<STRING,STRING> COMMENT '旧值'
) COMMENT '优惠券领用表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_coupon_use_inc/';
--7.2.19 收藏表（增量表）
DROP TABLE IF EXISTS ods_favor_info_inc;
CREATE EXTERNAL TABLE ods_favor_info_inc
(
    `type` STRING COMMENT '变动类型',
    `ts`   BIGINT COMMENT '变动时间',
    `data` STRUCT<id :STRING,user_id :STRING,sku_id :STRING,spu_id :STRING,is_cancel :STRING,create_time :STRING,cancel_time
                  :STRING> COMMENT '数据',
    `old`  MAP<STRING,STRING> COMMENT '旧值'
) COMMENT '收藏表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_favor_info_inc/';
--7.2.20 订单明细表（增量表）
DROP TABLE IF EXISTS ods_order_detail_inc;
CREATE EXTERNAL TABLE ods_order_detail_inc
(
    `type` STRING COMMENT '变动类型',
    `ts`   BIGINT COMMENT '变动时间',
    `data` STRUCT<id :STRING,order_id :STRING,sku_id :STRING,sku_name :STRING,img_url :STRING,order_price
                  :DECIMAL(16, 2),sku_num :BIGINT,create_time :STRING,source_type :STRING,source_id :STRING,split_total_amount
                  :DECIMAL(16, 2),split_activity_amount :DECIMAL(16, 2),split_coupon_amount
                  :DECIMAL(16, 2)> COMMENT '数据',
    `old`  MAP<STRING,STRING> COMMENT '旧值'
) COMMENT '订单明细表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_order_detail_inc/';
--7.2.21 订单明细活动关联表（增量表）
DROP TABLE IF EXISTS ods_order_detail_activity_inc;
CREATE EXTERNAL TABLE ods_order_detail_activity_inc
(
    `type` STRING COMMENT '变动类型',
    `ts`   BIGINT COMMENT '变动时间',
    `data` STRUCT<id :STRING,order_id :STRING,order_detail_id :STRING,activity_id :STRING,activity_rule_id :STRING,sku_id
                  :STRING,create_time :STRING> COMMENT '数据',
    `old`  MAP<STRING,STRING> COMMENT '旧值'
) COMMENT '订单明细活动关联表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_order_detail_activity_inc/';
--7.2.22 订单明细优惠券关联表（增量表）
DROP TABLE IF EXISTS ods_order_detail_coupon_inc;
CREATE EXTERNAL TABLE ods_order_detail_coupon_inc
(
    `type` STRING COMMENT '变动类型',
    `ts`   BIGINT COMMENT '变动时间',
    `data` STRUCT<id :STRING,order_id :STRING,order_detail_id :STRING,coupon_id :STRING,coupon_use_id :STRING,sku_id
                  :STRING,create_time :STRING> COMMENT '数据',
    `old`  MAP<STRING,STRING> COMMENT '旧值'
) COMMENT '订单明细优惠券关联表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_order_detail_coupon_inc/';
--7.2.23 订单表（增量表）
DROP TABLE IF EXISTS ods_order_info_inc;
CREATE EXTERNAL TABLE ods_order_info_inc
(
    `type` STRING COMMENT '变动类型',
    `ts`   BIGINT COMMENT '变动时间',
    `data` STRUCT<id :STRING,consignee :STRING,consignee_tel :STRING,total_amount :DECIMAL(16, 2),order_status :STRING,user_id
                  :STRING,payment_way :STRING,delivery_address :STRING,order_comment :STRING,out_trade_no :STRING,trade_body
                  :STRING,create_time :STRING,operate_time :STRING,expire_time :STRING,process_status :STRING,tracking_no
                  :STRING,parent_order_id :STRING,img_url :STRING,province_id :STRING,activity_reduce_amount
                  :DECIMAL(16, 2),coupon_reduce_amount :DECIMAL(16, 2),original_total_amount :DECIMAL(16, 2),freight_fee
                  :DECIMAL(16, 2),freight_fee_reduce :DECIMAL(16, 2),refundable_time :DECIMAL(16, 2)> COMMENT '数据',
    `old`  MAP<STRING,STRING> COMMENT '旧值'
) COMMENT '订单表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_order_info_inc/';
--7.2.24 退单表（增量表）
DROP TABLE IF EXISTS ods_order_refund_info_inc;
CREATE EXTERNAL TABLE ods_order_refund_info_inc
(
    `type` STRING COMMENT '变动类型',
    `ts`   BIGINT COMMENT '变动时间',
    `data` STRUCT<id :STRING,user_id :STRING,order_id :STRING,sku_id :STRING,refund_type :STRING,refund_num :BIGINT,refund_amount
                  :DECIMAL(16, 2),refund_reason_type :STRING,refund_reason_txt :STRING,refund_status :STRING,create_time
                  :STRING> COMMENT '数据',
    `old`  MAP<STRING,STRING> COMMENT '旧值'
) COMMENT '退单表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_order_refund_info_inc/';
--7.2.25 订单状态流水表（增量表）
DROP TABLE IF EXISTS ods_order_status_log_inc;
CREATE EXTERNAL TABLE ods_order_status_log_inc
(
    `type` STRING COMMENT '变动类型',
    `ts`   BIGINT COMMENT '变动时间',
    `data` STRUCT<id :STRING,order_id :STRING,order_status :STRING,operate_time :STRING> COMMENT '数据',
    `old`  MAP<STRING,STRING> COMMENT '旧值'
) COMMENT '退单表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_order_status_log_inc/';
--7.2.26 支付表（增量表）
DROP TABLE IF EXISTS ods_payment_info_inc;
CREATE EXTERNAL TABLE ods_payment_info_inc
(
    `type` STRING COMMENT '变动类型',
    `ts`   BIGINT COMMENT '变动时间',
    `data` STRUCT<id :STRING,out_trade_no :STRING,order_id :STRING,user_id :STRING,payment_type :STRING,trade_no
                  :STRING,total_amount :DECIMAL(16, 2),subject :STRING,payment_status :STRING,create_time :STRING,callback_time
                  :STRING,callback_content :STRING> COMMENT '数据',
    `old`  MAP<STRING,STRING> COMMENT '旧值'
) COMMENT '支付表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_payment_info_inc/';
--7.2.2--7.退款表（增量表）
DROP TABLE IF EXISTS ods_refund_payment_inc;
CREATE EXTERNAL TABLE ods_refund_payment_inc
(
    `type` STRING COMMENT '变动类型',
    `ts`   BIGINT COMMENT '变动时间',
    `data` STRUCT<id :STRING,out_trade_no :STRING,order_id :STRING,sku_id :STRING,payment_type :STRING,trade_no :STRING,total_amount
                  :DECIMAL(16, 2),subject :STRING,refund_status :STRING,create_time :STRING,callback_time :STRING,callback_content
                  :STRING> COMMENT '数据',
    `old`  MAP<STRING,STRING> COMMENT '旧值'
) COMMENT '退款表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_refund_payment_inc/';
--7.2.28 用户表（增量表）
DROP TABLE IF EXISTS ods_user_info_inc;
CREATE EXTERNAL TABLE ods_user_info_inc
(
    `type` STRING COMMENT '变动类型',
    `ts`   BIGINT COMMENT '变动时间',
    `data` STRUCT<id :STRING,login_name :STRING,nick_name :STRING,passwd :STRING,name :STRING,phone_num :STRING,email
                  :STRING,head_img :STRING,user_level :STRING,birthday :STRING,gender :STRING,create_time :STRING,operate_time
                  :STRING,status :STRING> COMMENT '数据',
    `old`  MAP<STRING,STRING> COMMENT '旧值'
) COMMENT '用户表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_user_info_inc/';
```

#### 4.2 收集脚本

+ ods 日志数据加载脚本

  ```shell
  #! /bin/bash
  # hdfs_to_ods_log.sh [日期]
  #1、判断日期是否传入,如果日期传入,使用传入的日期,如果没有传入则使用前一天日期
  [ "$1" ] && datestr=$1 || datestr=$(date -d '-1 day' +%F)
  #2、导入数据
  sql="load data inpath '/origin_data/gmall/log/topic_log/${datestr}/*' overwrite into table ods_log_inc partition (dt='${datestr}');"
  #执行导入sql语句
  /opt/module/hive/bin/hive -e "use gmall0509;${sql}"
  ```

+ ods 业务数据加载脚本

  ```shell
  #! /bin/bash
  # hdfs_to_ods_db.sh all/表名 [日期]
  #1、判断参数是否传入
  if [ $# -lt 1 ]
  then
  	echo "必须传入all/表名一个参数..."
  	exit
  fi
  #2、判断日期是否传入,如果传入则使用指定日期,如果没有传入使用前一天的日期
  [ "$2" ] && datestr=$2 || datestr=$(date -d '-1 day' +%F)
  
  function import_data(){
  # $*: 获取所有参数,如果使用""包裹之后,$*当做整体
  # $#: 获取参数个数
  # $@: 获取所有参数,如果使用""包裹之后,把每个参数当做单独的个体
  # $?: 获取上一个指令的结果
  	tableNames=$*
  	sql="use gmall0509;"
  	#遍历所有表,拼接每个表的数据加载sql语句
  	for table in $tableNames
  	do
  		sql="${sql}load data inpath '/origin_data/gmall/db/${table:4}/${datestr}/*' overwrite into table ${table} partition (dt='$datestr');"
  	done
  	#执行sql
  	/opt/module/hive/bin/hive -e "$sql"
  }
  
  #3、根据第一个参数匹配导入数据
  case $1 in
  "all")
  	import_data "ods_activity_info_full" "ods_activity_rule_full" "ods_base_category1_full" "ods_base_category2_full" "ods_base_category3_full" "ods_base_dic_full" "ods_base_province_full" "ods_base_region_full" "ods_base_trademark_full" "ods_cart_info_full" "ods_cart_info_inc" "ods_comment_info_inc" "ods_coupon_info_full" "ods_coupon_use_inc" "ods_favor_info_inc" "ods_order_detail_activity_inc" "ods_order_detail_coupon_inc" "ods_order_detail_inc" "ods_order_info_inc" "ods_order_refund_info_inc" "ods_order_status_log_inc" "ods_payment_info_inc" "ods_refund_payment_inc" "ods_sku_attr_value_full" "ods_sku_info_full" "ods_sku_sale_attr_value_full" "ods_spu_info_full" "ods_user_info_inc"
  ;;
  "ods_activity_info_full")
  	import_data "ods_activity_info_full"
  ;;
  "ods_activity_rule_full")
  	import_data "ods_activity_rule_full"
  ;;
  "ods_base_category1_full")
  	import_data "ods_base_category1_full"
  ;;
  "ods_base_category2_full")
  	import_data "ods_base_category2_full"
  ;;
  "ods_base_category3_full")
  	import_data "ods_base_category3_full"
  ;;
  "ods_base_dic_full")
  	import_data "ods_base_dic_full"
  ;;
  "ods_base_province_full")
  	import_data "ods_base_province_full"
  ;;
  "ods_base_region_full")
  	import_data "ods_base_region_full"
  ;;
  "ods_base_trademark_full")
  	import_data "ods_base_trademark_full"
  ;;
  "ods_cart_info_full")
  	import_data "ods_cart_info_full"
  ;;
  "ods_cart_info_inc")
  	import_data "ods_cart_info_inc"
  ;;
  "ods_comment_info_inc")
  	import_data "ods_comment_info_inc"
  ;;
  "ods_coupon_info_full")
  	import_data "ods_coupon_info_full"
  ;;
  "ods_coupon_use_inc")
  	import_data "ods_coupon_use_inc"
  ;;
  "ods_favor_info_inc")
  	import_data "ods_favor_info_inc"
  ;;
  "ods_order_detail_activity_inc")
  	import_data "ods_order_detail_activity_inc"
  ;;
  "ods_order_detail_coupon_inc")
  	import_data "ods_order_detail_coupon_inc"
  ;;
  "ods_order_detail_inc")
  	import_data "ods_order_detail_inc"
  ;;
  "ods_order_info_inc")
  	import_data "ods_order_info_inc"
  ;;
  "ods_order_refund_info_inc")
  	import_data "ods_order_refund_info_inc"
  ;;
  "ods_order_status_log_inc")
  	import_data "ods_order_status_log_inc"
  ;;
  "ods_payment_info_inc")
  	import_data "ods_payment_info_inc"
  ;;
  "ods_refund_payment_inc")
  	import_data "ods_refund_payment_inc"
  ;;
  "ods_sku_attr_value_full")
  	import_data "ods_sku_attr_value_full"
  ;;
  "ods_sku_info_full")
  	import_data "ods_sku_info_full"
  ;;
  "ods_sku_sale_attr_value_full")
  	import_data "ods_sku_sale_attr_value_full"
  ;;
  "ods_spu_info_full")
  	import_data "ods_spu_info_full"
  ;;
  "ods_user_info_inc")
  	import_data "ods_user_info_inc"
  ;;
  *)
  	echo "表名输入错误..."
  ;;
  esac
  ```

### 5- DIM层搭建

#### 5.1 sql代码

```sql
use gmall0509;
show tables;
--dim层建表
---1、建多少张维度表:
---------业务总线矩阵的每一列应该创建一个维度表,如果该维度表的维度属性很少,此时可以不用创建,直接冗余到事实表中
---2、内部表还是外部表: 还是采用外部表
---3、表名: dim_表名_全量[full]/拉链[zip]
---4、字段
-------维度表的字段从主维表和相关维表直接获取或者通过加工得到【如果不知道直接将主维表和相关维表所有字段直接全部取出作为维度表的维度属性】
---5、分区: 按天分区
---6、数据存储格式: 采用列式存储[orc/parquet] STORED AS ORC
---7、表的存储位置: 与其他表存储父目录保持一致即可
---8、是否压缩: 需要
---9、压缩格式: 选用snappy压缩 （需要频繁进行计算分析）
---10、数据从哪里来:
--  全量: 从ODS相关表当天分区获取
--  拉链:
--    首日: 从ODS相关表首日分区获取
--    每日: 从前一天9999-12-31分区 + 从ODS相关表当天分区获取
---11、数据到哪里去:
--  全量: 数据加载到DIM表当天分区中
--  拉链:
--    首日: 全部放入9999-12-31分区
--    每日: 最新数据覆盖写入9999-12-31,当天失效的数据放入前一天日期分区中
---12、数据加载方式: insert overwrite table dim表 partition(分区字段名=字段值) select * from ods表名 ..
--

-- 商品维度表
-- 建表的相关 业务数据表：
-- sku_info | spu_info | base_category3.2.1 | base_trademark | ods_sku_attr_value_full | ods_sku_sale_attr_value_full |
DROP TABLE IF EXISTS dim_sku_full;
CREATE EXTERNAL TABLE dim_sku_full
(
    `id`                   STRING COMMENT 'sku_id',
    `price`                DECIMAL(16, 2) COMMENT '商品价格',
    `sku_name`             STRING COMMENT '商品名称',
    `sku_desc`             STRING COMMENT '商品描述',
    `weight`               DECIMAL(16, 2) COMMENT '重量',
    `is_sale`              BOOLEAN COMMENT '是否在售',
    `spu_id`               STRING COMMENT 'spu编号',
    `spu_name`             STRING COMMENT 'spu名称',
    `category3_id`         STRING COMMENT '三级分类id',
    `category3_name`       STRING COMMENT '三级分类名称',
    `category2_id`         STRING COMMENT '二级分类id',
    `category2_name`       STRING COMMENT '二级分类名称',
    `category1_id`         STRING COMMENT '一级分类id',
    `category1_name`       STRING COMMENT '一级分类名称',
    `tm_id`                STRING COMMENT '品牌id',
    `tm_name`              STRING COMMENT '品牌名称',
    `sku_attr_values`      ARRAY<STRUCT<attr_id :STRING,value_id :STRING,attr_name :STRING,value_name:STRING>> COMMENT '平台属性',
    `sku_sale_attr_values` ARRAY<STRUCT<sale_attr_id :STRING,sale_attr_value_id :STRING,sale_attr_name :STRING,sale_attr_value_name:STRING>> COMMENT '销售属性',
    `create_time`          STRING COMMENT '创建时间'
) COMMENT '商品维度表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dim/dim_sku_full/'
    TBLPROPERTIES ('orc.compress' = 'snappy');

desc formatted dim_sku_full;

-- 数据导入
with
sku as (
    select id,
           spu_id,
           price,
           sku_name,
           sku_desc,
           weight,
           tm_id,
           category3_id,
           sku_default_igm,
           is_sale,
           create_time,
           dt
    from ods_sku_info_full
    where dt='2020-06-14'
),
spu as (
    select id,
           spu_name,
--            description, --不需要
            category3_id, --从sku获取
           tm_id  --从sku获取
--            dt
    from ods_spu_info_full
    where dt='2020-06-14'
),
c3 as (
    select id,
           name,
           category2_id
--            dt
    from ods_base_category3_full
    where dt='2020-06-14'
),
c2 as (
    select id,
           name,
           category1_id
--            dt
    from ods_base_category2_full
    where dt='2020-06-14'
),
c1 as(
    select id,
           name
--            dt
    from ods_base_category1_full
    where dt='2020-06-14'
),
tm as (
    select id,
           tm_name
--            logo_url,  --需求统计中不需要该列
--            dt
    from ods_base_trademark_full
    where dt='2020-06-14'
),
attr as (
    select
--            id,
--            attr_id,
--            value_id,
           sku_id,
--            attr_name,
--            value_name,
--            dt
           -- 多值属性，每件sku可能对应有多个平台属性
            collect_set(named_struct('attr_id',attr_id,'value_id',value_id,'attr_name',attr_name,'value_name',value_name)) attrs
    from ods_sku_attr_value_full
    where dt='2020-06-14'
    group by sku_id
),
sale_attr as (
    select
--            id,
           sku_id,
--            spu_id,
--            sale_attr_value_id,
--            sale_attr_id,
--            sale_attr_name,
--            sale_attr_value_name,
--            dt
           -- 多值属性，每件sku可能对应有多个销售属性
            collect_set(named_struct('sale_attr_id',sale_attr_id,'sale_attr_value_id',sale_attr_value_id,'sale_attr_name',sale_attr_name,'sale_attr_value_name',sale_attr_value_name)) sale_attrs
    from ods_sku_sale_attr_value_full
    where dt='2020-06-14'
    group by sku_id
)
insert overwrite table dim_sku_full partition (dt='2020-06-14')
select
    sku.id,
    sku.price,
    sku.sku_name,
    sku.sku_desc,
    sku.weight,
    sku.is_sale,
    sku.spu_id,
    spu.spu_name,
    sku.category3_id,
    c3.name,
    c3.category2_id,
    c2.name,
    c2.category1_id,
    c1.name,
    sku.tm_id,
    tm.tm_name,
    attr.attrs,
    sale_attr.sale_attrs,
    sku.create_time
from sku
left join spu on sku.spu_id = spu.id
left join c3 on sku.category3_id = c3.id
left join c2 on c3.category2_id = c2.id
left join c1 on c2.category1_id = c1.id
left join tm on sku.tm_id = tm.id
left join attr on sku.id = attr.sku_id
left join sale_attr on sku.id = sale_attr.sku_id;


----- 优惠券维度表
-- 相关mysql表：coupon_info | base_dic(join两次，一次得购物券类型编码，一次得优惠券使用范围编码)
DROP TABLE IF EXISTS dim_coupon_full;
CREATE EXTERNAL TABLE dim_coupon_full
(
    `id`               STRING COMMENT '购物券编号',
    `coupon_name`      STRING COMMENT '购物券名称',
    `coupon_type_code` STRING COMMENT '购物券类型编码',
    `coupon_type_name` STRING COMMENT '购物券类型名称', -- ods_base_dic
    `condition_amount` DECIMAL(16, 2) COMMENT '满额数',
    `condition_num`    BIGINT COMMENT '满件数',
    `activity_id`      STRING COMMENT '活动编号',
    `benefit_amount`   DECIMAL(16, 2) COMMENT '减金额',
    `benefit_discount` DECIMAL(16, 2) COMMENT '折扣',
    `benefit_rule`     STRING COMMENT '优惠规则:满元*减*元，满*件打*折', --加工
    `create_time`      STRING COMMENT '创建时间',
    `range_type_code`  STRING COMMENT '优惠范围类型编码',
    `range_type_name`  STRING COMMENT '优惠范围类型名称', -- ods_base_dic
    `limit_num`        BIGINT COMMENT '最多领取次数',
    `taken_count`      BIGINT COMMENT '已领取次数',
    `start_time`       STRING COMMENT '可以领取的开始日期',
    `end_time`         STRING COMMENT '可以领取的结束日期',
    `operate_time`     STRING COMMENT '修改时间',
    `expire_time`      STRING COMMENT '过期时间'
) COMMENT '优惠券维度表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dim/dim_coupon_full/'
    TBLPROPERTIES ('orc.compress' = 'snappy');
--
with ci as (
    select id,
           coupon_name,
           coupon_type,
           condition_amount,
           condition_num,
           activity_id,
           benefit_amount,
           benefit_discount,
           create_time,
           range_type,
           limit_num,
           taken_count,
           start_time,
           end_time,
           operate_time,
           expire_time,
           dt
    from ods_coupon_info_full
    where dt = '2020-06-15'
),
     bc1 as (
         select dic_code,
                dic_name
         from ods_base_dic_full
             --查询所有购物券的类型编码
         where dt = '2020-06-15' and parent_code = '32'
     ),
     bc2 as (
         select dic_code,
                dic_name
         from ods_base_dic_full
             --查询优惠券使用范围编码
         where dt = '2020-06-15' and parent_code = '33'
     )
insert overwrite table dim_coupon_full partition (dt='2020-06-15')
select id,
       coupon_name,
       coupon_type,
       bc1.dic_name,
       condition_amount,
       condition_num,
       activity_id,
       benefit_amount,
       benefit_discount,
       case coupon_type
           when '3201' then concat('满', condition_amount, '元减', benefit_amount, '元')
           when '3202' then concat('满', condition_num, '件打', 10 * (1 - benefit_discount), '折')
           when '3203' then concat('减', benefit_amount, '元')
        end benefit_rule,  --沉淀通用维度属性
       create_time,
       range_type,
       bc2.dic_name,
       limit_num,
       taken_count,
       start_time,
       end_time,
       operate_time,
       expire_time
from ci
         left join bc1
                   on ci.coupon_type = bc1.dic_code
         left join bc2
                   on ci.range_type = bc2.dic_code;


-- 活动维度表
--相关mysql表：activity_rule（主维表） | activity_info | base_dic
DROP TABLE IF EXISTS dim_activity_full;
CREATE EXTERNAL TABLE dim_activity_full
(
    `activity_rule_id`   STRING COMMENT '活动规则ID',
    `activity_id`        STRING COMMENT '活动ID',
    `activity_name`      STRING COMMENT '活动名称',
    `activity_type_code` STRING COMMENT '活动类型编码',
    `activity_type_name` STRING COMMENT '活动类型名称',
    `activity_desc`      STRING COMMENT '活动描述',
    `start_time`         STRING COMMENT '开始时间',
    `end_time`           STRING COMMENT '结束时间',
    `create_time`        STRING COMMENT '创建时间',
    `condition_amount`   DECIMAL(16, 2) COMMENT '满减金额',
    `condition_num`      BIGINT COMMENT '满减件数',
    `benefit_amount`     DECIMAL(16, 2) COMMENT '优惠金额',
    `benefit_discount`   DECIMAL(16, 2) COMMENT '优惠折扣',
    `benefit_rule`       STRING COMMENT '优惠规则',
    `benefit_level`      STRING COMMENT '优惠级别'
) COMMENT '活动信息表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dim/dim_activity_full/'
    TBLPROPERTIES ('orc.compress' = 'snappy');

--
with atr as (
    select id,
           activity_id,
           activity_type,
           condition_amount,
           condition_num,
           benefit_amount,
           benefit_discount,
           benefit_level
    from ods_activity_rule_full where dt='2020-06-15'
), at as (
    select id,
           activity_name,
           activity_type,
           activity_desc,
           start_time,
           end_time,
           create_time
    from ods_activity_info_full where dt='2020-06-15'
), bc as (
    select
        dic_code,
        dic_name
    -- 活动类型编码
    from ods_base_dic_full where dt='2020-06-15' and parent_code = '31'
)
insert overwrite table dim_activity_full partition(dt='2020-06-15')
select
    atr.id,
    activity_id,
    activity_name,
    atr.activity_type,
    bc.dic_name,
    activity_desc,
    start_time,
    end_time,
    create_time,
    condition_amount,
    condition_num,
    benefit_amount,
    benefit_discount,
    case atr.activity_type
       when '3101' then concat('满', condition_amount, '元减', benefit_amount, '元')
       when '3102' then concat('满', condition_num, '件打', 10 * (1 - benefit_discount), '折')
       when '3103' then concat('打', 10 * (benefit_amount), '折')
    end benefit_rule,
    benefit_level
from atr left join at
    on atr.activity_id = at.id
left join bc
    on atr.activity_type = bc.dic_code;


-- 地区维度表
-- base_province（主维表） | base_region（地区表）
DROP TABLE IF EXISTS dim_province_full;
CREATE EXTERNAL TABLE dim_province_full
(
    `id`            STRING COMMENT 'id',
    `province_name` STRING COMMENT '省市名称',
    `area_code`     STRING COMMENT '地区编码',
    `iso_code`      STRING COMMENT '旧版ISO-3166-2编码，供可视化使用',
    `iso_3166_2`    STRING COMMENT '新版IOS-3166-2编码，供可视化使用',
    `region_id`     STRING COMMENT '地区id',
    `region_name`   STRING COMMENT '地区名称'
) COMMENT '地区维度表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dim/dim_province_full/'
    TBLPROPERTIES ('orc.compress' = 'snappy');

--
insert overwrite table dim_province_full partition(dt='2020-06-15')
select
    province.id,
    province.name,
    province.area_code,
    province.iso_code,
    province.iso_3166_2,
    region_id,
    region_name
from
(
    select
        id,
        name,
        region_id,
        area_code,
        iso_code,
        iso_3166_2
    from ods_base_province_full
    where dt='2020-06-15'
)province
left join
(
    select
        id,
        region_name
    from ods_base_region_full
    where dt='2020-06-15'
)region
on province.region_id=region.id;


--日期维度表
--当日日期维度的表存储格式是orc,自带的InputFormat是orc
--我们当前日期的数据是普通文本,普通文本如果直接加载到orc的表中,后续读取数据的时候,orcInputFormat不能正常读取普通文本,只能读取orc文件
--所以需要创建一个临时表,先将日期数据加载到存储格式为textFile的临时表中,然后查询临时表的数据写入orc格式表中,此时实现数据的中转
DROP TABLE IF EXISTS dim_date;
CREATE EXTERNAL TABLE dim_date
(
    `date_id`    STRING COMMENT '日期ID',
    `week_id`    STRING COMMENT '周ID,一年中的第几周',
    `week_day`   STRING COMMENT '周几',
    `day`        STRING COMMENT '每月的第几天',
    `month`      STRING COMMENT '一年中的第几月',
    `quarter`    STRING COMMENT '一年中的第几季度',
    `year`       STRING COMMENT '年份',
    `is_workday` STRING COMMENT '是否是工作日',
    `holiday_id` STRING COMMENT '节假日'
) COMMENT '时间维度表'
    STORED AS ORC
    LOCATION '/warehouse/gmall/dim/dim_date/'
    TBLPROPERTIES ('orc.compress' = 'snappy');

--临时表读取本地文件
DROP TABLE IF EXISTS tem_date;
CREATE EXTERNAL TABLE tem_date
(
    `date_id`    STRING COMMENT '日期ID',
    `week_id`    STRING COMMENT '周ID,一年中的第几周',
    `week_day`   STRING COMMENT '周几',
    `day`        STRING COMMENT '每月的第几天',
    `month`      STRING COMMENT '一年中的第几月',
    `quarter`    STRING COMMENT '一年中的第几季度',
    `year`       STRING COMMENT '年份',
    `is_workday` STRING COMMENT '是否是工作日',
    `holiday_id` STRING COMMENT '节假日'
) COMMENT '时间维度表'
    row format delimited fields terminated by '\t'
    LOCATION '/warehouse/gmall/dim/tem_date/'
    TBLPROPERTIES ('orc.compress' = 'snappy');
load data inpath '/date_info.txt' overwrite into table tem_date;
-- 从临时表载入数据
insert overwrite table dim_date select * from tem_date;
show create table tem_date;
show create table dim_date;



-- 用户表
DROP TABLE IF EXISTS dim_user_zip;
CREATE EXTERNAL TABLE dim_user_zip
(
    `id`           STRING COMMENT '用户id',
    `login_name`   STRING COMMENT '用户名称',
    `nick_name`    STRING COMMENT '用户昵称',
    `name`         STRING COMMENT '用户姓名',
    `phone_num`    STRING COMMENT '手机号码',
    `email`        STRING COMMENT '邮箱',
    `user_level`   STRING COMMENT '用户等级',
    `birthday`     STRING COMMENT '生日',
    `gender`       STRING COMMENT '性别',
    `create_time`  STRING COMMENT '创建时间',
    `operate_time` STRING COMMENT '操作时间',
    `start_date`   STRING COMMENT '开始日期',
    `end_date`     STRING COMMENT '结束日期'
) COMMENT '用户表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dim/dim_user_zip/'
    TBLPROPERTIES ('orc.compress' = 'snappy');



--首日导入
-- ods_user_info_inc首日分区中保存是当天mysql表中所有数据。
-- 此时ods_user_info_full首日分区中的数据是直接从mysql表中获取的,肯定是最新数据,不存在过期数据
insert overwrite table dim_user_zip partition (dt='9999-12-31')
select
    data.id,
    data.login_name,
    data.nick_name,
    data.name,
    data.phone_num,
    data.email,
    data.user_level,
    data.birthday,
    data.gender,
    data.create_time,
    data.operate_time,
    date_format(nvl(data.operate_time,data.create_time),'yyyy-MM-dd') start_date, --从开始时间字段
    '9999-12-31' end_date
from ods_user_info_inc where dt='2020-06-14' and type='bootstrap-insert';
--测试时使用
-- order by start_date desc;
-- select * from dim_user_zip order by start_date desc;

--每日更新
--Join
set hive.exec.dynamic.partition.mode=nonstrict;
with old as (
    select id,
           login_name,
           nick_name,
           name,
           phone_num,
           email,
           user_level,
           birthday,
           gender,
           create_time,
           operate_time,
           start_date,
           end_date
    from dim_user_zip where dt='9999-12-31'
),new as (
    -- 一个用户一天内可能修改多次,所以一般只需要最后一次修改的数据
    select
        id,
        login_name,
        nick_name,
        name,
        phone_num,
        email,
        user_level,
        birthday,
        gender,
        create_time,
        operate_time,
        start_date,
        end_date
    from (
             select data.id,
                    data.login_name,
                    data.nick_name,
                    data.name,
                    data.phone_num,
                    data.email,
                    data.user_level,
                    data.birthday,
                    data.gender,
                    data.create_time,
                    data.operate_time,
                    '2020-06-15' start_date,
                    '9999-12-31' end_date,
                    row_number() over (partition by data.id order by ts desc, type desc) rk
             from ods_user_info_inc
             where dt = '2020-06-15'
         ) t1
    where rk=1
),

-- Join 方式
--每日导入
---第一种方案:
-----dim_user_info_zip:9999-12-31分区记录的是当日之前所有用户最新数据
-----ods_user_info_inc: 2020-06-15分区记录是当日的新增和修改的数据
----【dim_user_info_zip 9999-12-21分区】 join  ods_user_info_inc[2020-06-15]
-- user_full as (
--     select old.id old_id,
--            old.login_name old_login_name,
--            old.nick_name old_nick_name,
--            old.name old_name,
--            old.phone_num old_phone_num,
--            old.email old_email,
--            old.user_level old_user_level,
--            old.birthday old_birthday,
--            old.gender old_gender,
--            old.create_time old_create_time,
--            old.operate_time old_operate_time,
--            old.start_date old_start_date,
--            old.end_date old_end_date,
--            new.id new_id,
--            new.login_name new_login_name,
--            new.nick_name new_nick_name,
--            new.name new_name,
--            new.phone_num new_phone_num,
--            new.email new_email,
--            new.user_level new_user_level,
--            new.birthday new_birthday,
--            new.gender new_gender,
--            new.create_time new_create_time,
--            new.operate_time new_operate_time,
--            new.start_date new_start_date,
--            new.end_date new_end_date
--     from old full join new
--     on old.id = new.id
-- )
-- insert overwrite table dim_user_zip partition (dt)
-- select
--     nvl(new_id,old_id) id,
--     if(new_id is null,old_login_name,new_login_name) login_name,
--     if(new_id is null,old_nick_name,new_nick_name) nick_name,
--     if(new_id is null,old_name,new_name) name,
--     if(new_id is null,old_phone_num,new_phone_num) phone_num,
--     if(new_id is null,old_email,new_email) email,
--     if(new_id is null,old_user_level,new_user_level) user_level,
--     if(new_id is null,old_birthday,new_birthday) birthday,
--     if(new_id is null,old_gender,new_gender) gender,
--     if(new_id is null,old_create_time,new_create_time) create_time,
--     if(new_id is null,old_operate_time,new_operate_time) operate_time,
--     if(new_id is null,old_start_date,new_start_date) start_date,
--     if(new_id is null,old_end_date,new_end_date) end_date,
--     if(new_id is null,old_end_date,new_end_date) dt
-- from user_full --最新数据，9999-12-31分区
-- union
-- select
--    old_id,
--    old_login_name,
--    old_nick_name,
--    old_name,
--    old_phone_num,
--    old_email,
--    old_user_level,
--    old_birthday,
--    old_gender,
--    old_create_time,
--    old_operate_time,
--    old_start_date start_date,
--    cast(date_sub(new_start_date,1) as string) end_date,
--     cast(date_sub(new_start_date,1) as string) dt
-- from user_full
-- where new_id is not null and old_id is not null; --前一天最新和当日更新数据的交集，获取过期日期，并放入过期日期的分区中


--Union
    -----第二种方案:
    ---9999-12-31 union all 2020-6-15
    ---然后按照用户分区,按照生效时间降序,每个用户第一条数据是最新的,其他的就是失效的。
user_full as (
    select id,
           login_name,
           nick_name,
           name,
           phone_num,
           email,
           user_level,
           birthday,
           gender,
           create_time,
           operate_time,
           start_date,
           end_date,
           -- 根据日期降序排序，日期最大的表示最新数据
           row_number() over (partition by id order by start_date desc) rn
    from (
             select id,
                    login_name,
                    nick_name,
                    name,
                    phone_num,
                    email,
                    user_level,
                    birthday,
                    gender,
                    create_time,
                    operate_time,
                    start_date,
                    end_date
             from new
             union all
             select id,
                    login_name,
                    nick_name,
                    name,
                    phone_num,
                    email,
                    user_level,
                    birthday,
                    gender,
                    create_time,
                    operate_time,
                    start_date,
                    end_date
             from old
         ) t1
)  --select * from user_full;
-- --查询最新数据[优先取右边字段,右边是最新数据,左表没有连接上也是最新数据]
insert overwrite table dim_user_zip partition (dt)
select
    id,
    login_name,
    nick_name,
    name,
    phone_num,
    email,
    user_level,
    birthday,
    gender,
    create_time,
    operate_time,
    start_date,
    end_date,
    end_date dt
from user_full
where rn=1  -- rn=1 最新数据
union
select
    id,
    login_name,
    nick_name,
    name,
    phone_num,
    email,
    user_level,
    birthday,
    gender,
    create_time,
    operate_time,
    start_date,
    cast(date_sub('2020-06-15',1) as string) end_date,
    cast(date_sub('2020-06-15',1) as string) dt
from user_full
where rn=2; -- rn=2 失效数据
```

#### 5.2 同步脚本

##### 首日同步脚本

```shell
#! /bin/bash
#ods_to_dim_init.sh all/表名
#1、判断参数是否传入
if [ $# -lt 1 ]
then
	echo "必须传入all/表名..."
	exit
fi
#商品维度表首日加载sql语句
dim_sku_full_sql="
...
"
#优惠券维度表
dim_coupon_full_sql="
...
"

#活动维度表
dim_activity_full_sql="
...
"

#地区维度表
dim_province_full_sql="
...
"

#用户维度表(拉链表)
dim_user_zip_sql="
...
"
#2、根据第一个参数匹配加载数据到dim
case $1 in
"all")
	/opt/module/hive/bin/hive -e "use gmall0509;${dim_activity_full_sql}${dim_coupon_full_sql}${dim_province_full_sql}${dim_sku_full_sql}${dim_user_zip_sql}"
;;
"dim_activity_full")
	/opt/module/hive/bin/hive -e "use gmall0509;${dim_activity_full_sql}"
;;
"dim_coupon_full")
	/opt/module/hive/bin/hive -e "use gmall0509;${dim_coupon_full_sql}"
;;
"dim_province_full")
	/opt/module/hive/bin/hive -e "use gmall0509;${dim_province_full_sql}"
;;
"dim_sku_full")
	/opt/module/hive/bin/hive -e "use gmall0509;${dim_sku_full_sql}"
;;
"dim_user_zip")
	/opt/module/hive/bin/hive -e "use gmall0509;${dim_user_zip_sql}"
;;
esac
```



##### 每日同步脚本

```shell
#! /bin/bash
#ods_to_dim_init.sh all/表名 [日期]
#1、判断参数是否传入
if [ $# -lt 1 ]
then
	echo "必须传入all/表名..."
	exit
fi

#2、判断日期是否传入,如果传入则使用指定日期,如果没有传入则使用前一天日期
[ "$2" ] && datestr=$2 || datestr=$(date -d '-1 day' +%F)

dim_sku_full_sql="
with sku as (
        select id,
           spu_id,
           price,
           sku_name,
           sku_desc,
           weight,
           tm_id,
           category3_id,
           is_sale,
           create_time,
           dt
    from ods_sku_info_full where dt='${datestr}'
),spu as (
    select id,
           spu_name,
           category3_id,
           tm_id
    from ods_spu_info_full where dt='${datestr}'
), c3 as (
    select id,
           name,
           category2_id,
           dt
    from ods_base_category3_full where dt='${datestr}'
),c2 as (
    select id,
           name,
           category1_id
    from ods_base_category2_full where dt='${datestr}'
), c1 as (
    select id,
           name
    from ods_base_category1_full where dt='${datestr}'
),tm as (
    select id,
           tm_name
    from ods_base_trademark_full where dt='${datestr}'
),sav as (
    select
           sku_id,
           collect_list(named_struct('attr_id',attr_id,'value_id',value_id,'attr_name',attr_name,'value_name',value_name)) sku_attr_values
    from ods_sku_attr_value_full where dt='${datestr}'
    group by sku_id
),ssav as (
    select
        sku_id,
        collect_list(named_struct('sale_attr_id',sale_attr_id,'sale_attr_value_id',sale_attr_value_id,'sale_attr_name',sale_attr_name,'sale_attr_value_name',sale_attr_value_name)) sku_sale_attr_values
    from ods_sku_sale_attr_value_full where dt='${datestr}'
    group by sku_id
)
insert overwrite table dim_sku_full partition (dt='${datestr}')
select
    sku.id,
    price,
    sku_name,
    sku_desc,
    weight,
    is_sale,
    spu_id,
    spu_name,
    sku.category3_id,
    c3.name,
    category2_id,
    c2.name,
    category1_id,
    c1.name,
    sku.tm_id,
    tm_name,
    sku_attr_values,
    sku_sale_attr_values,
    create_time
from sku left join spu
on sku.spu_id = spu.id
left join tm
on sku.tm_id = tm.id
left join c3
on sku.category3_id = c3.id
left join c2
on c3.category2_id = c2.id
left join c1
on c2.category1_id = c1.id
left join sav
on sku.id = sav.sku_id
left join ssav
on sku.id = ssav.sku_id;
"

dim_coupon_full_sql="
with ci as (
    select id,
           coupon_name,
           coupon_type,
           condition_amount,
           condition_num,
           activity_id,
           benefit_amount,
           benefit_discount,
           create_time,
           range_type,
           limit_num,
           taken_count,
           start_time,
           end_time,
           operate_time,
           expire_time
    from ods_coupon_info_full where dt='${datestr}'
), bc1 as (
    select
        dic_code,
        dic_name
    --查询所有购物券的类型编码
    from ods_base_dic_full where dt='${datestr}' and parent_code='32'
),bc2 as (
    select
        dic_code,
        dic_name
    --查询优惠券使用范围编码
    from ods_base_dic_full where dt='${datestr}' and parent_code='33'
)
insert overwrite table dim_coupon_full partition (dt='${datestr}')
select
    id,
    coupon_name,
    coupon_type,
    bc1.dic_name,
    condition_amount,
    condition_num,
    activity_id,
    benefit_amount,
    benefit_discount,
    case coupon_type
        when '3201' then concat('满',condition_amount, '元减',benefit_amount,'元' )
        when '3202' then concat('满',condition_num,'打',(1-benefit_discount)*10,'折')
        else concat('减',benefit_amount,'元')
    end,
    create_time,
    range_type,
    bc2.dic_name,
    limit_num,
    taken_count,
    start_time,
    end_time,
    operate_time,
    expire_time
from ci left join bc1
on ci.coupon_type = bc1.dic_code
left join bc2
on ci.range_type = bc2.dic_code;
"

dim_activity_full_sql="
with at as (
    select id,
           activity_name,
           activity_type,
           activity_desc,
           start_time,
           end_time,
           create_time
    from ods_activity_info_full where dt='${datestr}'
), atr as (
    select id,
           activity_id,
           activity_type,
           condition_amount,
           condition_num,
           benefit_amount,
           benefit_discount,
           benefit_level
    from ods_activity_rule_full where dt='${datestr}'
),bc as (
    select
        dic_code,
        dic_name
    from ods_base_dic_full where dt='${datestr}' and parent_code='31'
)
insert overwrite table dim_activity_full partition (dt='${datestr}')
select
    atr.id,
    activity_id,
    activity_name,
    atr.activity_type,
    dic_name,
    activity_desc,
    start_time,
    end_time,
    create_time,
    condition_amount,
    condition_num,
    benefit_amount,
    benefit_discount,
    case atr.activity_type
        when '3101' then concat('满',condition_amount, '元减',benefit_amount,'元' )
        when '3102' then concat('满',condition_num,'打',(1-benefit_discount)*10,'折')
        else concat('打',(1-benefit_discount)*10,'折')
    end,
    benefit_level
from atr left join at
on atr.activity_id = at.id
left join bc
on atr.activity_type = bc.dic_code;
"

dim_province_full_sql="
with pv as (
    select id,
           name,
           region_id,
           area_code,
           iso_code,
           iso_3166_2
    from ods_base_province_full where dt='${datestr}'
),rg as (
    select id,
           region_name
    from ods_base_region_full where dt='${datestr}'
)
insert overwrite table dim_province_full partition (dt='${datestr}')
select
    pv.id,
    pv.name,
    area_code,
    iso_code,
    iso_3166_2,
    region_id,
    region_name
from pv left join rg
on pv.region_id = rg.id;
"

dim_user_zip_sql="
set hive.exec.dynamic.partition.mode=nonstrict;
with old as (
    select id,
           login_name,
           nick_name,
           name,
           phone_num,
           email,
           user_level,
           birthday,
           gender,
           create_time,
           operate_time,
           start_date,
           end_date
    from dim_user_zip where dt='9999-12-31'
), new as (
    --一个用户一天内可能修改多次,所以一般只需要最后一次修改的数据
    select id,
           login_name,
           nick_name,
           name,
           phone_num,
           email,
           user_level,
           birthday,
           gender,
           create_time,
           operate_time,
           start_date,
           end_date
    from (
             select data.id,
                    data.login_name,
                    data.nick_name,
                    data.name,
                    data.phone_num,
                    data.email,
                    data.user_level,
                    data.birthday,
                    data.gender,
                    data.create_time,
                    data.operate_time,
                    '${datestr}' start_date,
                    '9999-12-31' end_date,
                    row_number() over (partition by data.id order by ts desc) rn
             from ods_user_info_inc
             where dt = '${datestr}'
         ) t1 where rn=1
),user_full as (
    select id,
           login_name,
           nick_name,
           name,
           phone_num,
           email,
           user_level,
           birthday,
           gender,
           create_time,
           operate_time,
           start_date,
           end_date,
           row_number() over (partition by id order by start_date desc) rn
    from (
             select id,
                    login_name,
                    nick_name,
                    name,
                    phone_num,
                    email,
                    user_level,
                    birthday,
                    gender,
                    create_time,
                    operate_time,
                    start_date,
                    end_date
             from new
             union all
             select id,
                    login_name,
                    nick_name,
                    name,
                    phone_num,
                    email,
                    user_level,
                    birthday,
                    gender,
                    create_time,
                    operate_time,
                    start_date,
                    end_date
             from old
         ) t1
)
insert overwrite table dim_user_zip partition (dt)
---选出最新数据
select id,
       login_name,
       nick_name,
       name,
       phone_num,
       email,
       user_level,
       birthday,
       gender,
       create_time,
       operate_time,
       start_date,
       t1.end_date,
       t1.dt
from (
         select id,
                login_name,
                nick_name,
                name,
                phone_num,
                email,
                user_level,
                birthday,
                gender,
                create_time,
                operate_time,
                start_date,
                end_date,
                end_date dt
         from user_full
         where rn = 1
         union all
---失效数据
         select id,
                login_name,
                nick_name,
                name,
                phone_num,
                email,
                user_level,
                birthday,
                gender,
                create_time,
                operate_time,
                start_date,
                cast(date_sub('${datestr}', 1) as string) end_date,
                cast(date_sub('${datestr}', 1) as string) dt
         from user_full
         where rn = 2
     ) t1
"

#3、根据第一个参数匹配加载数据到dim
case $1 in
"all")
	/opt/module/hive/bin/hive -e "use gmall0509;${dim_activity_full_sql}${dim_coupon_full_sql}${dim_province_full_sql}${dim_sku_full_sql}${dim_user_zip_sql}"
;;
"dim_activity_full")
	/opt/module/hive/bin/hive -e "use gmall0509;${dim_activity_full_sql}"
;;
"dim_coupon_full")
	/opt/module/hive/bin/hive -e "use gmall0509;${dim_coupon_full_sql}"
;;
"dim_province_full")
	/opt/module/hive/bin/hive -e "use gmall0509;${dim_province_full_sql}"
;;
"dim_sku_full")
	/opt/module/hive/bin/hive -e "use gmall0509;${dim_sku_full_sql}"
;;
"dim_user_zip")
	/opt/module/hive/bin/hive -e "use gmall0509;${dim_user_zip_sql}"
;;
esac
```

### 6- DWD层搭建

```sql
交易域
	加购物车	事务事实表
	下单		 事务事实表
	支付成功	事务事实表
	购物车		周期快照事实表
	交易流程	累计快照事实表
	
工具域
	优惠券使用(使用)	事务事实表
	
互动域
	收藏商品	事务事实表

流量域
	页面浏览	事务事实表

用户域
	用户注册	事务事实表
	用户登录    事务事实表
```







### 7- Superset可视化

> + Apache Superset是一个现代的数据探索和可视化平台。它功能强大且十分易用，可对接各种数据源，包括很多现代的大数据分析引擎，拥有丰富的图表展示形式，并且支持自定义仪表盘。
> + Superset是由Python语言编写的Web应用，要求Python3.7的环境
>
> +++

#### 7.1 python环境

安装 Minconda

> + conda是一个开源的包、环境管理器，可以用于在同一个机器上安装不同Python版本的软件包及其依赖，并能够在不同的Python环境之间切换，
> + Anaconda包括Conda、Python以及一大堆安装好的工具包，比如：numpy、pandas等，
> + Miniconda包括Conda、Python。
>
> 此处，我们不需要如此多的工具包，故选择MiniConda。
>
> +++
>
> ```bash
> # 下载地址
> # https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
> 
> bash Miniconda3-latest-Linux-x86_64.sh
> #安装时可以指定安装路径  /opt/module/miniconda3
> 
> source ~/.bashrc #加载环境变量，使其生效
> 
> #Miniconda安装完成后，每次打开终端都会激活其默认的base环境，我们可通过以下命令，禁止激活默认base环境
> conda config --set auto_activate_base false
> ```
>
> + `conda` 环境管理常用命令
>
>   > 创建环境：`conda create -n env_name`
>   >
>   > 查看所有环境：`conda info --envs`
>   >
>   > 删除一个环境：`conda remove -n env_name --all`
>   >
>   > 激活环境：`conda activate env_name`
>   >
>   > 退出环境：`conda deactivate`
>   >
>   > +++
>   
> + 安装其他版本python的环境
>
>   ```bash
>   conda create -n superset python=3.9
>   ```

#### 7.2 Superset部署

+ 进入 superset 环境

  ```bash
  conda activate superset
  ```

+ 安装 Superset

  ```bash
  #安装以下依赖
  sudo yum install -y gcc gcc-c++ libffi-devel python-devel python-pip python-wheel python-setuptools openssl-devel cyrus-sasl-devel openldap-devel
  
  #安装（更新）setuptools和pip
  pip install --upgrade setuptools pip -i https://pypi.douban.com/simple/
#pip是python的包管理工具，可以和centos中的yum类比
  
  #安装superset
  pip install apache-superset==1.4.2 -i https://pypi.douban.com/simple/
  #更换华为云镜像(下载很慢时使用)
  pip install apache-superset --trusted-host https://repo.huaweicloud.com -i https://repo.huaweicloud.com/repository/pypi/simple
  
  #markupsafe依赖的版本回退到 2.0.1
  pip install --force-reinstall MarkupSafe==2.0.1
  
  #初始化数据库
  export FLASK_APP=superset
  superset db upgrade
  
  #创建管理员用户 user:admin - password:admin
  superset fab create-admin
  
  #Superset 初始化
  superset init
  ```
  
  ```bash
  #启动 Superset
  
  #安装 gunicorn
  pip install gunicorn -i https://pypi.douban.com/simple/
  
  #启动 Superset
  gunicorn --workers 5 --timeout 120 --bind hadoop102:8787  "superset.app:create_app()"
  #workers:指定进程个数
  #timeout:worker进程的超时时间
  #bind:绑定本机地址
  
  #登录Superset http://hadoop102:8787
  #安装mysql数据源，安装后需要重启生效
  conda install mysqlclient
  ```

#### 7.3 Superset使用

