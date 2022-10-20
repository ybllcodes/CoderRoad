# Git

## 1-Git简介

1. Git是世界上最先进的 分布式版本控制系统（Version Control System）

2. 安装

   > https://git-scm.com/
   >
   > Use Git from Git Bash only

3. 设置Git账户

   | 命令                               | 含义                         |
   | ---------------------------------- | ---------------------------- |
   | git config --list                  | 查看所有配置                 |
   | git config --list --show-origin    | 查看所有配置以及所在文件位置 |
   | git config --global user.name xxx  | 设置git用户名                |
   | git config --global user.email xxx | 设置git邮箱                  |
   | git config core.autocrlf false     | 取消换行符转换的warning提醒  |

   > Git的三级配置
   >
   > ![image-20220830191322268](http://ybll.vip/md-imgs/202208301913383.png)

## 2-Git使用

```bash
git init #将目录转换为git可以管理的目录
#生成 .git/ 目录，存放git重要的数据结构（暂存区、本地库）
```

工作区（编写文件） —— 暂存区（临时存储） —— 本地库（历史版本）

### 2.1 Git 常用命令

![image-20220830193520812](http://ybll.vip/md-imgs/202208301935891.png)

| 命令                         | 作用                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| git status                   | 查看本地库的状态                                             |
| git add [file]               | 把工作区的修改添加到暂存区                                   |
| git add .                    | 添加当前目录中的所有修改到暂存区                             |
| git rm --cached [file]       | 将一个未tracking文件从暂存区撤销修改  (都可以撤销)           |
| git restore --staged [file]  | 将一个tracking文件从暂存区撤销修改  （都可以撤销）           |
| git restore [file]\|         | 将工作区的修改撤销                                           |
| git checkout -- [file]       | 从本地库检出最新文件覆盖工作区的文件(文件还没有提交到暂存区, 否则无效) |
| **git commit -a -m “说明” ** | 用于将tracked file添加到本地库  （只能提交 tracked file）    |
| git commit –m “说明” [file]  | 将暂存区的修改提交到本地库                                   |

### 2.2 Git 版本切换

| 命令                                                 | 作用                                     |
| ---------------------------------------------------- | ---------------------------------------- |
| git log  【空格：换页；q：退出】                     | 以完整格式查看本地库状态(查看提交历史)   |
| git log --pretty=oneline                             | 以单行模式查看本地库状态                 |
| git reset --hard HEAD^                               | 回退一个版本                             |
| git reset --hard HEAD~n                              | 回退N个版本                              |
| **git reflog**                                       | 查看所有操作的历史记录                   |
| **git reset --hard [具体版本号，例如：1f9a527等]  ** | 回到（回退和前进都行）指定版本号的版本， |

### 2.3 文件比较

| 命令                     | 作用               |
| ------------------------ | ------------------ |
| git diff 文件            | 工作区和暂存区比较 |
| git diff HEAD <file>     | 工作区和本地库比较 |
| git diff --cached <file> | 暂存区和本地库比较 |

![image-20220830195127371](http://ybll.vip/md-imgs/202208301951425.png)

### 2.4 忽略文件

> + 创建 `.gitignore` 文件，列出要忽略的文件的模式
>
> ```bash
> # 忽略所有的 .a 文件 
> *.a
> 
> # 排除忽略lib.a文件
> !lib.a
> 
> # 忽略整个目录
> build/
> 
> ```
>
> + 遇到中文无法在 git bash 中显示的场景，可以尝试执行如下命令
>
>   `git config --global core.quotepath false`

### 2.5 分支操作

> 1. 分支介绍
>
>    ![image-20220830200512301](http://ybll.vip/md-imgs/202208302005385.png)
>
> 2. 常用命令
>
>    | 命令                                       | 描述                                                         |
>    | ------------------------------------------ | ------------------------------------------------------------ |
>    | git branch [分支名]                        | 创建分支                                                     |
>    | git branch -v                              | 查看分支,可以使用-v参数查看详细信息                          |
>    | git checkout [分支名]                      | 切换分支                                                     |
>    | git merge [分支名]                         | 合并分支；  将merge命令中指定的分支合并到当前分支上  例如：如果想将dev分支合并到master分支，那么必须在master分支上执行merge命令 |
>    | git branch –d [分支名]                     | 删除分支                                                     |
>    | git checkout –b [分支名]                   | 新建并切换到新分支                                           |
>    | git log --oneline --decorate --graph --all | 输出你的提交历史、各个分支的指向以及项目的分支分叉情况       |
>
> 3. 说明
>
>    > 编辑冲突的文件，把“>>>>>>>>>”、“<<<<<<”和“========”等这样的行删除，编辑至满意的状态，提交。
>    >
>    > **提交的时候注意**：git commit命令不要带文件名

+++

## 3-远程协作

![image-20220901132157348](http://ybll.vip/md-imgs/202209011321444.png)



### 3.1 本机与远程仓库的SSH连接

```bash
ssh-keygen #生成本机的密钥
#保存在家目录的 .ssh/ 目录中
#将公钥 id_rsa.pub 的内容拷贝至远程仓库　gitee

#测试连通
ssh -T ssh@gitee.com
```

### 3.2 本地库与远程库交互操作

| 命令                              | 作用                                                     |
| --------------------------------- | -------------------------------------------------------- |
| git remote add 变量名 远程仓库url | 为远超仓库的url指定一个变量名，之后可以使用变量名代替url |
| git push -u 变量名 "本地分支名"   | 将本地的指定分支推送到远程仓库                           |
| git remote -v                     | 查看远程仓库列表                                         |
| git pull 变量名 远程仓库分支名    | 拉取远程仓库指定分支到本地                               |
| git clone 变量名                  | 下载整个远程仓库到本地                                   |

### 3.3 处理push冲突

+ 当本地分支的版本落后于远程仓库的版本时，是无法顺利push的
+ 本地顺利push的前提是，本地仓库的版本要比远程仓库新。
+ 解决：可以先拉取远程仓库的最新版本到本地，然后合并处理冲突后，再次push

### 3.4 其他说明

> + 可以将别人的公开仓库fork到自己的仓库中，随意修改
> + 当对其他人仓库的文件进行修改后，此时没有权限取提交修改，可以向仓库所有者，发起一次pull request请求，请求仓库所有者合并我们的修改，当然也可能遭到拒绝
> + 邀请成员：可以生成专属邀请链接，发送链接给指定的用户，用户打开后即可以选择加入团队，加入团队后可以直接向团队的仓库提交修改，无需pull request

+++

+++

+++



# Redus

## 1-NoSQL

+ NoSQL : Not Only SQL , 非关系型的数据库
+ NoSQL 不拘泥于关系型数据库的设计范式，放弃通用的技术标准，为某一领域特定场景而设计，从而使性能、容量、扩展性都达到一定程度的突破

+++

+ 特点
  + 不遵循SQL标准
  + 不支持事务的 ACID
  + 远超于SQL的性能
+ 适用场景
  + 对数据高并发读写
  + 海量数据的读写
  + 对数据 高可扩展性（不拘泥与表形式）
+ 不适用的场景
  + 需要事务支持，对数据安全性要求高（不能容忍丢数据）、
  + 基于sql的结构化查询存储，处理复杂的关系，需要即席查询

+++

+ NoSQL家族

  > **1）**Memcached
  >
  > （1）很早出现的NoSQL数据库
  >
  > （2）数据都在内存中，一般不持久化
  >
  > （3）支持简单的key-value模式，数据类型支持单一
  >
  > （4）一般是作为缓存数据库辅助持久化的数据库
  >
  > **2**）**Redis** 
  >
  > （1）几乎覆盖了Memcached的绝大部分功能
  >
  > （2）数据都在内存中，支持持久化，主要用作备份恢复
  >
  > （3）支持丰富的数据类型，例如 string 、 list 、 set、zset、hash等
  >
  > （4）一般是作为缓存数据库辅助持久化的数据库
  >
  > **3**）mongoDB
  >
  > （1）高性能、开源、模式自由的文档型数据库
  >
  > （2）数据都在内存中，如果内存不足，把不常用的数据保存到硬盘
  >
  > （3）虽然是key-value模式，但是对value(尤其是json)提供了丰富的查询功能
  >
  > （4）支持二进制数据及大型对象
  >
  > （5）可以根据数据的特点替代RDBMS(关系数据库管理系统)，成为独立的数据库。或者配合RDBMS，存储特定的数据
  >
  > **4**）**HBase**
  >
  > （1）Hbase是Hadoop项目的数据库，主要用于对大量数据进行随机、实时的读写操作.
  >
  > （2）Hbase能支持到数十亿行 × 百万列的数据表
  >
  > **5**）Cassandra
  >
  > （1）Cassandra用于管理由大量商用服务器构建起来的庞大集群上的海量数据集(PB级)
  >
  > **6**）Neo4j
  >
  > （1）Neo4j是基于图结构的数据库，一般用于构建社交网络、交通网络、地图等
  >
  > +++

## 2-Redis简介

> 官网：http://Redis.io
>
> 中文翻译（非官方）：http://www.Redis.net.cn
>
> +++
>
> + Redis是什么
>
>   + 一个开源的 K-V 存储系统
>   + 支持的 value 类型很多，如 `string` `list` `set` `zset` `hash`
>   + Redis会周期型地把更新的数据 `写入磁盘` 或者把修改操作 `写入追加的记录文件`
>   + 支持高可用和集群模式
>
> + 应用场景·
>
>   1. 配合关系性数据库，做高速缓存
>
>   ![image-20220831100453686](http://ybll.vip/md-imgs/202208311004796.png)
>
>   +++
>
>   2. 大数据场景：缓存
>
>   ![image-20220831100613665](http://ybll.vip/md-imgs/202208311006792.png)
>
>   +++
>
>   3. 大数据场景：数据库
>
>   ![image-20220831100635673](http://ybll.vip/md-imgs/202208311006765.png)
>
>   +++
>
>   4. 利用多样的数据结构，存储特定的数据
>
>      > + 最新N个数据 ：list 实现按自然事件排序
>      > + 排行榜，topN ：zset,有序集合
>      > + 手机验证码（时效性的数据）：Expire过期
>      > + 计数器，秒杀 ：原子性，自增方法 **incr** **decr**
>      > + 去除大量重复数据 ：set集合
>      > + 构建队列 ：list集合
>      > + 发布订阅消息系统 ：pub/sub模式（一般使用 kafka）

+++



## 3-Redis安装与使用

### 3.1-安装与配置

> ```bash
> #安装新版gcc编译器
> sudo yum -y install gcc-c++
> 
> #进入src/目录，修改Makefile文件
> vim src/Makefile
> 	#修改该行配置，使得安装后的配置文件在家目录的bin/下，可以直接执行
> 	PREFIX?=/home/atguigu
> #执行编译&&安装命令
> make && make install
> ```
>
> + 查看安装目录：/home/atguigu/bin
>
>   > （1）Redis-benchmark:性能测试工具，可以在自己本子运行，看看自己本子性能如何(服务启动起来后执行)
>   >
>   > （2）Redis-check-aof：修复有问题的AOF文件
>   >
>   > （3）Redis-check-dump：修复有问题的RDB文件
>   >
>   > （4）Redis-sentinel：启动Redis哨兵服务
>   >
>   > （5）redis-server：Redis服务器启动命令
>   >
>   > （6）redis-cli：客户端，操作入口
>
> + 启动 Redis :  ***`redis-server 配置文件`*** 
>
>   > 1. 拷贝一份 redis.conf 配置文件到工作目录
>   >
>   >    ```bash
>   >    mkdir /home/atguigu/myredis
>   >    cp /opt/software/redis-6.2.1/redis.conf /home/atguigu/myredis
>   >    ```
>   >
>   > 2. 绑定主机IP,修改 bind 属性
>   >
>   >    ```bash
>   >    vim redis.conf
>   >    	bind 0.0.0.0
>   >    	#0.0.0.0 代表当前机器的所有IP地址
>   >    		#内网地址：192.168.10.102
>   >    		#外网IP:xxx.xxx.xxx.xxx
>   >    		#本机环形网卡：localhost 127.0.0.1
>   >    ```
>   >
>   > 3. 启动
>   >
>   >    ```bash
>   >    redis-server /home/atguigu/myredis/redis.conf
>   >    ```
>
> + 客户端访问
>
>   > ```bash
>   > redis-cli
>   > redis-cli -p 6379
>   > redis-cli -h hadoop102 -p 6379
>   > redis-cli -h 127.0.0.1 -p 6379
>   > 
>   > #通过ping命令测试验证
>   > ping
>   > 	PONG
>   > 	
>   > #关闭服务（关闭服务器redis-server）
>   > redis-cli shutdown #linux命令行操作，关闭Redi服务
>   > shutdown #进入客户端后，执行该命令也是关闭Redis服务
>   > 
>   > 
>   > #Redis默认 16 个库，从 0-15
>   > select 1 #切换到 1号库
>   > ```
>
> + Redis 的相关配置
>
>   > 1. 计量单位说明：大小写不敏感
>   >
>   >    > 1k => 1000 bytes
>   >    >
>   >    > 1kb => 1024 bytes
>   >    >
>   >    > 1m => 1000000 bytes
>   >    >
>   >    > 1mb => 1024 * 1024 bytes
>   >    >
>   >    > 1g => 1000000000 bytes
>   >    >
>   >    > 1gb => 1024 * 1024 * 1024 bytes
>   >
>   > 2. bind
>   >
>   >    > + 默认 `bind=127.0.0.1` ，只接受本机的访问请求；
>   >    >
>   >    > + 不写的情况下，无限制接受任何ip地址的访问，生产环境肯定要写你应用服务器的地址
>   >    >
>   >    > + 如果开启了protected-mode，那么在没有设定bind ip且没有设密码的情况下，Redis只允许接受本机的请求
>   >    >
>   >    >   ```bash
>   >    >   protected-mode no #yes
>   >    >   ```
>   >
>   > 3. port 服务端口号
>   >
>   >    > ```bash
>   >    > port 6379
>   >    > ```
>   >
>   > 4. daemonize
>   >
>   >    > ```bash
>   >    > daemonize yes #是否为后台进程，no
>   >    > ```
>   >
>   > 5. pidfile
>   >
>   >    > ```bash
>   >    > #存放pid 文件的位置，每个实例都会产生一个不同的pid文件
>   >    > pidfile /home/atguigu/myredis/redis_6379.pid
>   >    > ```
>   >
>   > 6. log file
>   >
>   >    > ```bash
>   >    > #日志文件存放位置
>   >    > log file "/home/atguigu/myredis/logs"
>   >    > #目录存放在logs文件中，不是生成logs目录
>   >    > ```
>   >
>   > 7. database
>   >
>   >    > ```bash
>   >    > #设定库的数量 默认16
>   >    > database 16
>   >    > ```
>   >
>   > 8. requirepass
>   >
>   >    > ```bash
>   >    > #设置密码
>   >    > requirepass 123456 #默认没有加该项，即没有密码
>   >    > ```
>   >    >
>   >    > ![image-20220831175909278](http://ybll.vip/md-imgs/202208311759333.png)
>   >
>   > 9. maxmemory
>   >
>   >    > + 设置Redis可以使用的内存量
>   >    > + 一 旦到达内存使用上限，Redis将会试图移除内部数据，移除规则可以通过`maxmemory-policy`来指定
>   >    > + 如果无法根据规则移除内存中的数据，或设置了 不允许移除 ，则会针对需要申请内存的指令返回错误信息，如set 、lpush等
>   >    >
>   >    > ```bash
>   >    > maxmemory <bytes>
>   >    > #1g 1gb ...
>   >    > ```
>   >
>   > 10. maxmemory-policy
>   >
>   >     > 移除策略
>   >     >
>   >     > ```bash
>   >     > # maxmemory-policy noeviction 
>   >     > 
>   >     > #volatile-lru：使用LRU算法移除key，只对设置了过期时间的键运算
>   >     > #allkeys-lru：使用LRU算法移除key
>   >     > #volatile-lfu ：使用LFU策略移除key,只对设置了过期时间的键运算.
>   >     > #allkeys-lfu  :使用LFU策略移除key
>   >     > #volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键运算
>   >     > #allkeys-random：移除随机的key
>   >     > #volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key
>   >     > #noeviction：不进行移除。针对写操作，只是返回错误信息
>   >     > 
>   >     > ```
>   >
>   > 11. maxmemory-samples
>   >
>   >     > + 设置样本数
>   >     > + LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小
>   >     > + 一般设置3到7的数字，数值越小样本越不准确，但是性能消耗也越小
>   >     >
>   >     > ```bash
>   >     > # maxmemory-samples 5
>   >     > ```
>
> +++



### 3.2 数据类型

0. 帮助手册：[Redis 命令参考 — Redis 命令参考 (redisdoc.com)](http://redisdoc.com/)

1. redis键（key）

   ```bash
   keys * #查看当前库的所有键
   keys ?a #?匹配一个字符
   exists <key> #判断键是否存在
   type <key> #查看键对应的value类型 (abc 1 234 都是string)
   del <key> #删除某个键
   
   expire <key> <second> #设置某个键的过期时间
   ttl <key> #查看某个键的过期时间（s）,-1：用不过期，-2：已经过期
   
   dbsize #当前库中 key 的数量
   flushdb #清空当前库
   flushall #清空所有库
   ```

2. String

   > + Redis最基本的类型，适合保存单值类型
   > + 二进制安全（客户端发送什么，服务端就存什么，不会进行二次编码，称为二进制安全），意味着Redis的 string 可以包含任何数据，如：jpg图片，序列化对象等
   > + 一个Redis中字符串value最多可以是 512MB
   >
   > ```bash
   > set key value
   > get key
   > append key value #往key的valUe末尾追加字符串
   > strlen key #得到值的长度
   > setnx key value #key不存在时则设置 key
   > incr key #增1
   > decr key #减1
   > incrby key step
   > decrby key step
   > 
   > mset k1 v2 k2 v2
   > mget k1 k2 k3
   > msetnx k1 v1 k2 v2
   > 
   > getrange key start end #闭区间 [start,end],0开始计数，【0,n-1】
   > setrange key start value #下标为start开始替换，包括start
   > 
   > setex key seconds value #设置值和过期时间
   > 
   > getset key value #设置新值，同时获取旧值
   > ```

3. List

   > + 单键多值
   >
   > + 字符串列表，按照插入顺序排序，可以重复，可以 **从左或从右** 插入
   >
   > + 底层是一个双向链表，对两端操作性能高，通过索引下标 操作中间的节点性能较差
   >
   > + 从左到右：【0，n -1】；从右到左：【-n，-1】；遍历全部【0，-1】
   >
   >   ![image-20220831145111733](http://ybll.vip/md-imgs/202208311451842.png)
   >
   > ```bash
   > lpush key e1 e2 e3...
   > rpush key e1 e2 e3...
   > lrange key start end #获取[start,end]范围的值(闭区间)，[0.-1]:获取全部
   > llen key #获取list长度
   > 
   > lpop key [num] #删除0 或者 num 个值；【值在键在，值光键亡】
   > rpop key [num]
   > 
   > rpoplpush list1 list2 #从list1右侧删除(pop)一个值，插入(push)到list2的左边
   > 
   > lrange key start stop
   > lindex key index #获取index下标的元素值，左、右遍历的下标都可以
   > 
   > linsert key before|after pivot element #在指定元素pivot的 前|后 插入新元素element
   > 
   > lrem key count element #从左边删除count个指定value
   > #lrem list 2 a :list集合中，从左到右，删除两个 a
   > ```

4. Set

   > + 元素无序且不重复，set提供了判断成员是否在set集合内存在的接口、
   > + Redis的Set就是String类型的无序集合，底层就是一个value为null的Hash表，所以添加、删除、查找的复杂度都是 O(1)
   >
   > ```bash
   > sadd key member1 member2...
   > smembers key #取出集合所有的值
   > sismember key member #判断key是否包含member,返回 1|0
   > 
   > scard key #返回元素个数
   > 
   > srem key member1 member2...#删除指定元素
   > spop key #随机删除集合中的一个值
   > 
   > srandmember key count #随机从集合中取出 count 个值
   > 
   > sinter key1 key2 #返回多个集合的交集元素
   > sunion key1 key2 #返回多个集合的并集元素
   > sdiff key1 key2	#返回多个集合的差集元素，key1中出现，其他key中没有的元素
   > ```

5. Hash

   > ![image-20220831154724038](http://ybll.vip/md-imgs/202208311547134.png)
   >
   > + hash适合存储Map 或 对一个对象的某些属性有频繁的写操作场景
   >
   > ```bash
   > hset key f1 v1 f2 v2 f2 v3 ...
   > hsetnx key f1 v1 f2 v2 ...
   > hget key field
   > 
   > hexists key field
   > 
   > hkeys key
   > hvals key
   > hgetall key
   > 
   > hincrby key field step
   > ```

6. zset

   > + 有序集合，没有重复元素；与set的区别就是每个成员都关联了一个评分（score）,用于实现排序
   > + 集合中的成员是唯一的，但评分可以重复
   > + 元素有序，所以可以根据评分（score）或者 次序（position）来获取某一范围的元素
   > + 访问有序集合中间的元素也很快
   >
   > ```bash
   > zadd key score1 member1 score2 member2...
   > 
   > zrange key start stop [withscores] #正序取(默认升序排列)，withscores:评分也列出来，放在member的下一列
   > zrevrange key start stop [withscores] #倒序取（降序排列）
   > 
   > zrangebyscore key min max [withscores] #根据score范围，从小到大
   > zrevrangebyscore key max min [withscores] #根据score范围，从大到小
   > +INF : 无穷大
   > -INF : 无穷小
   > 
   > zincrby key step member #给key集合的member成员的score增加 step
   > 
   > zcount key min max #统计score在 [min,max] 范围内元素的个数
   > 
   > zrank key member #返回member成员的排名，从0开计数（升序排列）
   > zrevrank key member #降序排列
   > ```



### 3.3 Jedis

> Jedis 是Redis的Java客户端，提供了JavaAPI来操作 Redis
>
> ```xml
> <dependency>
>     <groupId>redis.clients</groupId>
>     <artifactId>jedis</artifactId>
>     <version>3.3.0</version>
> </dependency>
> ```
>
> +++
>
> ```java
> //创建一个客户端对象, 连接服务端
> Jedis jedis = new Jedis("hadoop102", 6379);
> 
> //向服务端发送命令
> jedis.ping();
> jedis.set("key","value");
> ...
>     
> //关闭客户端
> //如果该连接是从池中借来的，自动还回池子，如果是new对象，那么就直接关闭
> jedis.close();
> 
> //创建一个存放客户端连接的池子
> JedisPool jedisPool = new JedisPool("hadoop102", 6379);
> //从池子中借一个连接
> Jedis jedis = jedisPool.getResource();
> ```



### 3.4 Redis持久化

> Redis提供两种不同形式的持久化：***RDB*** 和 ***AOF***
>
> RDB:全量快照备份，将内存中所有数据持久化到磁盘的一个文件中
>
> AOF:增量日志备份，将所有写操作命令，不断地追加记录在一个日志文件中
>
> +++
>
> + RDB
>
>   > + RDB备份默认为自动开启，不使用时，将 `dbfilename`后设置成空白
>   > + 在指定时间间隔内，将内存中的数据集快照写入磁盘
>   > + 在Redis服务启动时，将指定位置的快照文件直接读到内存里
>   >
>   > +++
>   >
>   > redis.conf配置
>   >
>   > ```bash
>   > dbfilename dump.rdb #RDB保存的文件名,不写则表明不开启 RDB
>   > 
>   > dir ./ #RDB文件保存路径，默认是启动目录 ./
>   > 	dir /home/atguigu/myredis/
>   > ```
>   >
>   > +++
>   >
>   > 自动备份与手动备份
>   >
>   > + 手动备份
>   >
>   >   > （1）save: 在执行时，server端会阻塞客户端的读写。全力备份。
>   >   >
>   >   > （2）bgsave: 在执行时,server端不阻塞客户端的读写，一边备份一边服务，速度慢。
>   >   >
>   >   > （3）shutdown时服务会立刻执行备份后再关闭
>   >   >
>   >   > （4）flushall时会将清空后的数据备份
>   >
>   > + 自动备份
>   >
>   >   > redis.conf
>   >   >
>   >   > ![image-20220831202810263](http://ybll.vip/md-imgs/202208312028385.png)
>   >   >
>   >   > ```bash
>   >   > save 900 1
>   >   > save 300 10
>   >   > save 60 10000
>   >   > #save <seconds> <changes>
>   >   > ```
>   >
>   > +++
>   >
>   > redis.conf其他配置
>   >
>   > ```bash
>   > rdbcompression yes #rdb保存时，将文件压缩
>   > 
>   > rdbchecksum yes #文件检验
>   > #在存储快照后，还可以让Redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能
>   > ```
>   >
>   > +++
>   >
>   > RDB的优缺点
>   >
>   > + 优点：相比AOF,节省磁盘空间，恢复速度快，适合 **容灾恢复**
>   > + 缺点：数据量大时消耗性能；隔一定时间备份一次，最后一次备份后的修改可能丢失
>
>   +++
>
> + AOF（Append Only File）
>
>   > + 默认不开启，需要手动在配置文件中开启
>   > + 以日志形式来记录每个**写操作**，将Redis执行过程中的所有写指令都记录下来
>   > + 只允许追加文件，不可以改写文件，Redis启动之时会读取该文件重新构建数据
>   > + 换言之，Redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作
>   >
>   > +++
>   >
>   > redis.conf
>   >
>   > ```bash
>   > appendonly yes #开启AOF，默认 no
>   > appendfilename "appendonly.aof" #指定AOF文件名
>   > #文件可读，可以看懂
>   > dir /home/atguigu/myredis #RDB的路径一致，同一个参数项
>   > 
>   > #备份策略
>   > #根操作系统OS有关
>   > # no: don't fsync, just let the OS flush the data when it wants. Faster.
>   > #来一条数据，写一次aof文件，最后一次写操作可能丢失
>   > # always: fsync after every write to the append only log. Slow, Safest.
>   > #每秒从 buff区写入aof文件，1s内数据可能丢失
>   > # everysec: fsync only one time every second. Compromise.
>   > 
>   > #AOF文件损坏恢复
>   > redis-check-aof --fix appendonly.aof
>   > ```
>   >
>   > +++
>   >
>   > Rewrite
>   >
>   > + 当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的重写,只保留可以恢复数据的最小指令集
>   > + 可以手动使用命令 `bgrewriteaof` 手动重写
>   > + 重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担
>   >
>   > ```bash
>   > #Rewrite
>   > #系统载入时或者上次重写完毕时，Redis会记录此时AOF大小，设为base_size,如果：
>   > #Redis的AOF当前大小>= base_size +base_size*100% (默认)
>   > #当前大小>=64mb(默认)
>   > #两种情况，Redis会对AOF进行重写。
>   > auto-aof-rewrite-percentage 100
>   > auto-aof-rewrite-min-size 64mb
>   > ```
>   >
>   > +++
>   >
>   > AOF的优缺点
>   >
>   > + 优点：备份机制更稳健，丢失数据概率更低。默认只丢1秒的数据，极限情况只丢失最后一次操作的数据；可读的日志文本，通过操作AOF文件，可以处理误操作
>   >
>   > + 缺点
>   >
>   >   > （1）比起RDB占用更多的磁盘空间
>   >   >
>   >   > （2）恢复备份速度要慢
>   >   >
>   >   > （3）每次写都同步的话，有一定的性能压力
>   >   >
>   >   > （4）存在个别bug，造成恢复不能
>
> +++
>
> 持久化优先级：
>
> + AOF的优先级大于RDB
> + 同时开启了AOF和RDB，Redis服务启动时恢复数据以AOF为准
>
> +++
>
> RDB和AOF的选择
>
> > （1）官方推荐两个都启用。
> >
> > （2）如果对数据不敏感，可以选单独用RDB
> >
> > （3）不建议单独用 AOF，因为可能会出现Bug。
> >
> > （4）如果只是做纯内存缓存，可以都不用

+++

+++

+++



# HBase

## 1-HBase介绍

> HBase是一种 分布式、可扩展、支持海量数据存储的NoSQL数据库
>
> ![image-20220902103939309](http://ybll.vip/md-imgs/202209021039475.png)
>
> 数据模型：
>
> + 逻辑结构上，和关系型数据库类似，数据存储在一张表中，有行有列
>
> + 底层物理结构：K-V 对，更像一个 multi-dimensional map（多维地图）
>
>   ![hbase3](http://ybll.vip/md-imgs/202209021040202.jpg)
>
> +++
>
> 逻辑结构：
>
> ![image-20220902104245265](http://ybll.vip/md-imgs/202209021042364.png)
>
> +++



### 1.1 物理存储结构

> ![image-20220902104315856](http://ybll.vip/md-imgs/202209021043984.png)
>
> + 数据模型：
>
>   + NameSpace ：命名空间，类似于关系性数据库的 database，自带两个命名空间- `hbase`  `default`
>
>   + Table ：类似于关系型数据库的表，定义表时，只需要声明列族，不需要什么具体的列，写数据时，字段可以 **动态、按需指定** ，可以应对字段变更的场景
>
>   + Row ：每行数据由 一个 **RowKey** 和 多个Column 组成，按照RowKey的字典顺序存储，查询时根据RowKey进行检索
>
>   + Column Family ：列族，多个列的集合，同一个表的不同列族可以有完全不同的属性配置，但是同一个列族的所有列都有相同的属性，**列族存在的意义是HBase会把相同列族的列尽量放在同一台机器上（一个列族，对应一个hdfs上的目录，可能有多个文件）**;列族的定义越少越好
>
>     + info | nstu 两个列族
>
>     ![image-20220904133741626](http://ybll.vip/md-imgs/202209041337822.png)
>
>   + Column Qualifier ：列名的定义；可以随意定义，一行中的列不限名字，不限数量，只限定列族，列名称前必须带着所属的列族 -- **info:name** **info:age**
>
>   + Time Stamp ：用于标识数据的不同版本（version）,每条数据写入时，会自动加上该字段，其值为**写入HBase的时间**
>
>   + Cell ：具体的value值，以字节码的形式存贮，由{rowkey, column Family：column Qualifier, time Stamp} 唯一确定的单元
>
>   + Region ：
>
>     ![image-20220904183238555](http://ybll.vip/md-imgs/202209041832700.png)
>
>     + 一个表的若干行组成，在 Region 中行的排序按照 rowkey 字典排序；
>     + **Region不能跨RegionServer,当数据量大时，会拆分Region**;
>     + Region 由 RegionServer进程管理，HBase在进行负载均衡时，可能会发生移动，交由另一个RegionServer进行管理
>     + Region是基于HDFS的，所有数据的存储操作都是调用了 HDFS客户端接口来实现
>
> +++



### 1.2 HBase基本架构

> ![image-20220904200723333](http://ybll.vip/md-imgs/202209042007471.png)
>
> + Region Server
>
>   + Region的管理者，实现类为HRegionServer
>   + 负责对数据的操作（DML）：`get`  `put`  `delete`
>   + 负责对Region的操作：`splitRegion`  `compactRegion`
>
> + Master
>
>   + 所有 Region Server 的管理者，实现类为 HMaster
>   + 负责对表，库的操作（DDL）：`create`  `delete`  `alter`
>   + 对RegionServer的操作：分配 region 到每个 RegionServer，监控每个RegionServer的状态；负载均衡和故障转移 
>
> + Zookeeper
>
>   + HBase通过 Zookeeper 来实现master的高可用，RegionServer的监控，元数据的入口，集群配置的维度等工作
>
> + HDFS
>
>   + 底层的数据存储服务，使得HBase有高容错性
>
>   +++



### 1.3 RegionServer架构

> ![image-20220904201934026](http://ybll.vip/md-imgs/202209042019137.png)
>
> **数量关系：**
>
> + 一个 ***`Block Cache`*** (读缓存)：缓存的是 HFile中的 `Data Block`（64k）
>
> + 一个***`WAL`***，预写日志，防止RegionServer故障；负责将所有发向RegionServer的命令顺序写到HDFS（`hbase/WALs` 目录），形成日志文件（类似redis的aof文件）
>
> + 多个 ***`Region`*** ：读、写操作的最基本的单位（所有读写命令都需要先找到对应的Region）
>
>   + 多个 ***`store`*** （列族）
>
>     + 一个 ***`MemStore`*** : 写缓存，定期刷写到hdfs（异步刷写）,刷写后，会形成一个新的HFile文件
>
>       ```bash
>       flush 'tblname' #手动刷写
>       #写操作时，还没有从MemStore刷写到hdfs，此时RegionServer挂掉，
>       #此时Master会重新给该Region分配RegionServer,新的RegionServer会根据WAL日志文件，将数据加载到Region,从而使得数据不会丢失
>       ```
>
>     + 多个 ***`StoreFile`***  (保存数据的文件，HFile)
>
>       > + HFile最基本单元：Data Block （64k）
>       >
>       >   ![image-20220904204130588](http://ybll.vip/md-imgs/202209042041672.png)
>
>       ```bash
>       #查看 HFile
>       hbase hfile -p ${StoreFile_Path} #hdfs上具体的文件路径
>       ```
>
> +++
>
> 图解：
>
> ![image-20220904203601020](http://ybll.vip/md-imgs/202209042036183.png)
>
> +++
>
> ![image-20220904203846218](http://ybll.vip/md-imgs/202209042038377.png)

+++



## 2-HBase入门

### 2.1 HBase安装部署

> + 解压，安装
>
>   ```bash
>   tar -zxvf hbase-2.0.5-bin.tar.gz -C /opt/module
>   mv /opt/module/hbase-2.0.5 /opt/module/hbase
>   
>   #环境变量
>   sudo vim /etc/profile.d/my_env.sh
>   	export HBASE_HOME=/opt/module/hbase
>   	export PATH=$PATH:$HBASE_HOME/bin
>   ```
>
> + 修改配置
>
>   ```bash
>   #vim conf/hbase-env.sh
>   export HBASE_MANAGES_ZK=false
>   #HBASE需要使用zookeeper,设置为false,则启动hbase后，放弃内置的zk,而使用外部zk
>   ```
>
>   ```xml
>   # vim conf/hbase-site.xml
>   <configuration>
>       <!-- hdfs存储的根目录 -->
>       <property>
>           <name>hbase.rootdir</name>
>           <value>hdfs://hadoop102:8020/hbase</value>
>       </property>
>   	<!-- 是否搭建hbase集群（分布式）-->
>       <property>
>           <name>hbase.cluster.distributed</name>
>           <value>true</value>
>       </property>
>   	<!-- zk的地址，端口号默认，无需添加 -->
>       <property>
>           <name>hbase.zookeeper.quorum</name>
>           <value>hadoop102,hadoop103,hadoop104</value>
>   	</property>
>       <!-- 解决hadoop版本兼容问题-->
>       <property>
>           <name>hbase.unsafe.stream.capability.enforce</name>
>           <value>false</value>
>       </property>
>       
>       <property>
>           <name>hbase.wal.provider</name>
>           <value>filesystem</value>
>       </property>
>   </configuration>
>   ```
>
>   ```bash
>   # vim conf/regionservers
>   # 设置哪些节点启动 regionservers
>   	hadoop102
>   	hadoop103
>   	hadoop104
>   ```
>
>   + `xsync /opt/module/hbase/*` 分发到其他节点，再配置好其他节点的环境变量
>
> + HBase的启动
>
>   + 集群时间同步问题，必须同步后才能启动，否则报错
>
>     ```bash
>     date #查看时间
>     sudo date -s '2022-09-02 02:02:02' #时间设置
>     sudo ntpdate -u 'ntp1.aliyun.com' #跟阿里云时间服务器保持时间同步
>     
>     #集群时间同步
>     xcall sudo ntpdate -u 'ntp1.aliyun.com'
>     ```
>
>   ```bash
>   #Zookeeper、Hadoop 正常启动
>   #单点启动
>   bin/hbase-daemon.sh start master
>   bin/hbase-daemon.sh start regionserver
>   
>   #群起
>   bin/start-hbase.sh
>   bin/stop-hbase.sh
>   
>   #访问页面 hadoop102:master服务所在节点
>   hadoop102:16010
>   ```
>
> + Hbase高可用（master配置多节点)
>
>   ![image-20220902134646125](http://ybll.vip/md-imgs/202209021346238.png)
>
> +++
>

### 2.2 HBase Shell操作

> ```bash
> start-hbase.sh #群起Hbase
> hbase shell
> 	hbase> help #查看所有命令
> 	hbase> help 'create_namespace' #查看如何使用
> ```
>
> > + namespace
> >
> >   > Commands: alter_namespace, create_namespace, describe_namespace, drop_namespace, list_namespace, list_namespace_tables
> >   >
> >   > ```bash
> >   > list_namespace #查看所有库
> >   > 
> >   > help 'create_namespace'
> >   > create_namespace 'mydb1',{'createtime'=>'2020-02-02'}
> >   > describe_namespace 'mydb'
> >   > 
> >   > help 'alter_namespace'
> >   > alter_namespace 'mydb', {METHOD=> 'set','createtime'=> '2022-09-03'}
> >   > 
> >   > drop_namespace 'mydb1' #只能删空库，有表时，不能删除
> >   > 
> >   > create 'mydb:mytbl' ,'f1'
> >   > list_namespace_tables 'mydb'
> >   > ```
> >
> > + ddl
> >
> >   > alter, alter_async, alter_status, create, describe, disable, disable_all, 
> >   > 			drop, drop_all, enable, enable_all, exists, get_table, is_disabled, is_enabled, 
> >   > 			list, list_regions, locate_region, show_filters
> >   >
> >   > ```bash
> >   > list #列出用户创建的表
> >   > # mydb:mytbl测试
> >   > disable 'mydb:mytbl' #disable 后，就不能查看表了(scan...)
> >   > drop 'mydb:mytbl' #对于表，必须先 disable，才能drop
> >   > 
> >   > #default:student测试
> >   > create 'student', 'info'
> >   > describe 'student'
> >   > ```
> >
> > + dml
> >
> >   > append, count, delete, deleteall, ***get,*** get_counter, get_splits, incr, ***put, scan***, truncate, truncate_preserve
> >   >
> >   > ```bash
> >   > 
> >   > put 'mydb:mytbl','100','f1:age' ,'23'
> >   > 
> >   > flush 'mydb:mytbl'
> >   > scan 'mydb:mytbl' #获取当前表的最新数据
> >   > scan 'mydb:mytbl',{RAW=>TRUE,VERSIONS=> 10}
> >   > 
> >   > delete 'mydb:mytbl', '102','f1:age'
> >   > scan 'mydb:mytbl',{RAW=>TRUE,VERSIONS=> 10} # 102多一行，type=Delete
> >   > ```

+++



## 3-Hbase进阶

### 3.1 写流程

![image-20220904221118769](http://ybll.vip/md-imgs/202209042211901.png)

> 1. Client 向 Zookeeper发送请求，得到HBase系统表 `hbase:meta` 由哪个 Region Server 所管理【meta表的每一行就是一个Region的相关属性信息】
> 2. Client 访问该Region Server,获取 `hbase:meta` 表，根据写请求的 `namespace:table/rowkey` 查询出目标数据位于哪个RegionServer,哪个Region中；并将该表信息写入缓存中（提高效率，避免多次访问）
> 3. 与目标 Region Server 进行通信（发送put请求）
> 4. 将数据顺序写入到 WAL (HLog)
> 5. 将数据写入到对应Region的 MemStore（会在MemStore中进行排序）
> 6. 向客户端发送 ack
> 7. 等待后续MemStore的Flush刷写，将数据写到 HFile



### 3.2 Flush刷写

> MemStore => HDFS（HFile）
>
> +++
>
> + Memstore级别·
>
>   + 当某个memstore 的大小 达到 `hbase.hregion.memstore.flush.size` (默认128M)
>   + 其所在的Region 的所有memstore都会被刷写
>   + 因此，建议不要创建太多列族，否则一个memstore达到128M,会多并发地往HDFS写数据
>
> + Region级别
>
>   + 当某个 Region 中所有memstore的大小 达到 `hbase.hregion.memstore.flush.size` (上述参数，默认128M) * `hbase.hregion.memstore.block.multiplier` (默认值：4) 
>   + 此时，阻止继续往该Region写数据，所有memstore开始进行刷写
>
> + RegionServer级别
>
>   + 一个RegionServer中的阈值大于 `java_heapsize `* `hbase.regionserver.global.memstore.size`（默认值0.4）* `hbase.regionserver.global.memstore.size.lower.limit`（默认值0.95）
>
>   + region会按照其所有memstore的大小顺序（由大到小）依次进行刷写；直到region server中所有memstore的总大小减小到上述值以下
>
>     +++
>
>   + 当RegionServer中memstore的总大小达到 `java_heapsize` * `hbase.regionserver.global.memstore.size`（默认值0.4）时
>
>   + 会阻止继续往所有的memstore写数据；等待memstore刷写
>
> + HLog数量上限
>
>   > + 当WAL文件的数量超过hbase.regionserver.max.logs，
>   >
>   > + region会按照时间顺序依次进行刷写，直到WAL文件数量减小到hbase.regionserver.max.log以下
>   > + （该属性名已经废弃，现无需手动设置，最大值为32）
>
> + 定时刷写
>
>   + 自动刷新的时间间隔配置：`hbase.regionserver.optionalcacheflushinterval`（默认1小时）
>
> + 手动刷写：`flush 表名 或 region名 或 regionserver名`

+++



### 3.3 读流程

> + 布隆过滤器可以判断一个元素是否一定不在一个集合中或者可能(有误判可能)在一个集合中。
>
> <img src="http://ybll.vip/md-imgs/202209042252534.png" alt="image-20220904225231391" style="zoom:150%;" />
>
> +++
>
> ![image-20220904225421030](http://ybll.vip/md-imgs/202209042254191.png)
>
> > 1. 请求 Zookeeper，获取 `hbase:meta` 所在的 RegionServer
> >
> > 2. 向该RegionServer发送请求，获取 `hbase:meta` 表信息，并将其缓存在Client本地，方便下次使用
> >
> > 3. 扫描 `hbase:meta` 表，找到读请求的行，属于哪些Regin，哪些ReginServer
> >
> > 4. 向目标表的 RegionServer 发送读请求，获取数据
> >
> >    > RegionServer收到读请求后：
> >    >
> >    > + 为每个Region 初始化一个 RegionScanner对象
> >    >
> >    > + 每个 RegionScanner 为当前Region的每个列族（store）都初始化一个 StoreScanner对象
> >    >
> >    > + 每个StoreScanner对象
> >    >
> >    >   + 创建一个 MemstoreScanner对象（读取MemberStore缓存）
> >    >   + 为每个StoreFile创建一个 StorefileScanner对象（读取HFile文件）
> >    >
> >    >   > + StorefileScanner优先加载 StoreFile (HFile)的 `Load-on-open-section` 部分的数据，由 BloomFilter(布隆过滤器)判断 所需读取的数据 是否一定不在该 HFile
> >    >   > + 一定不在则跳过读取该 HFile；可能在，则先读取索引部分，根据索引信息，找到 所需读取的行 所在的数据块(Block)的元数据（文件中的起始位置，偏移量，id等）
> >    >   > + 由Block的 id 判断是否在  `Block Cache`(读缓存) 中，若已经缓存，则直接从缓存中读取，否则，从HFile中扫描Block,并将其放入缓存中，方便后续使用
> >
> > 5.  `memstore` `blockchache` `storefile(HFile)`三者查到的所有数据进行合并，挑选指定 ts的数据返回【此处所有数据是指同一条数据的不同版本（time stamp）或者不同的类型（Put/Delete）】
> >
> > 6. 将合并后的最终结果返回给客户端
> >
> > ![image-20220904231508256](http://ybll.vip/md-imgs/202209042315390.png)

+++



### 3.4 Compaction

> + 由于memstore每次刷写都会生成一个新的HFile，且同一个字段的不同版本（timestamp）和不同类型（Put/Delete）有可能会分布在不同的HFile中，因此查询时需要遍历所有的HFile
> + 为了减少HFile的个数，以及清理掉过期和删除的数据，会进行StoreFile Compaction
>
> +++
>
> Compaction分为两种，分别是Minor Compaction和Major Compaction
>
> + Minor Compaction会将临近的若干个较小的HFile合并成一个较大的HFile，并清理掉部分过期和删除的数据
> + Major Compaction会将一个Store下的所有的HFile合并成一个大HFile，并且**会**清理掉所有过期和删除的数据
>
> +++
>
> ![image-20220904234444068](http://ybll.vip/md-imgs/202209042344151.png)

### 3.5 Region切分

> + 默认情况下，每个Table起初只有一个Region，随着数据的不断写入，Region会自动进行拆分
> + 刚拆分时，两个子Region都位于当前的Region Server，但处于负载均衡的考虑，HMaster有可能会将某个Region转移给其他的Region Server
>
> +++
>
> +  0.94版本之前的策略：`ConstantSizeRegionSplitPolicy `
>   + 当一个Store（对应一个列族）的StoreFile大小大于配置 hbase.hregion.max.filesize（默认10G）时就会拆分
> + 0.94版本之后的策略
>   + ...
> + 2.0版本之后的策略 ：【新的split策略：`SteppingSplitPolicy`】
>   + 当前RegionSer ver上该表只有一个Region，按照2 *  `hbase.hregion.memstore.flush.size`(默认128M) 分裂【2*128M】
>   + 否则按照 `hbase.hregion.max.filesize` (默认10G)分裂

## 4-Hbase API

![image-20220905000032705](http://ybll.vip/md-imgs/202209050000787.png)



## 5-Hbase使用设计

### 5.1 预分区

> + 每一个region维护着startRow与endRowKey，如果加入的数据符合某个region维护的rowKey范围，则该数据交给这个region维护
> + 因此，依照这个原则，我们可以将数据所要投放的分区提前大致的规划好，以提高HBase性能
> + 如何实现预分区：建表语句后追加 预分区内容
>
> +++
>
> 1. 手动设定预分区
>
>    ```bash
>    create 'tbl' 'col_f' ,SPLITS => ['1000','2000','3000','4000'];
>    #此时，tbl有 5个region
>    ```
>
> 2. 生成16进制 序列预分区
>
>    ```bash
>    create 'tbl' 'col_f',{NUMREGIONS => 15, SPLITALGO => 'HexStringSplit'}
>    #15个region,00000,11111,22222，...，aaaaa,bbbbb,....,eeeee,...
>    # 15 ,num 可以自己指定
>    ```
>
> 3. 按照文件中设置的规则 预分区
>
>    ```bash
>    # vim mysplits.txt
>    aaaaa
>    bbbbb
>    ccccc
>    ddddd
>    
>    #建表语句：
>    create 'tbl' 'col_f' , SPLITS_FILE => 'mysplits.txt'
>    ```
>
> +++



### 5.2 RowKey设计（重要）

> ![image-20220905181539258](http://ybll.vip/md-imgs/202209051815362.png)
>
> +++
>
> + 设计原则：
>
>   ![image-20220905182056991](http://ybll.vip/md-imgs/202209051820065.png)
>
> + 需求：
>
>   ![image-20220905182034998](http://ybll.vip/md-imgs/202209051820071.png)
>
> ![image-20220905181858421](http://ybll.vip/md-imgs/202209051818556.png)

### 5.3 优化(调参)

> + 内存优化
>
> > + HBase操作过程中需要大量的内存开销，毕竟Table是可以缓存在内存中的
> > + 不建议分配非常大的堆内存，因为GC过程持续太久会导致RegionServer处于长期不可用状态，一般16~36 G
> > + 如果因为框架占用内存过高导致系统内存不足，框架一样会被系统服务拖死
> > + conf/hbase-env.sh 文件
>
> ```properties
> #对master和regionserver都有效
> export HBASE_HEAPSIZE=1G
> 
> #只对master有效
> export HBASE_MASTER_OPTS=自定义的jvm虚拟机参数
> 
> #只对regionserver有效
> export HBASE_REGIONSERVER_OPTS=自定义的jvm虚拟机参数
> ```
>
> +++
>
> +  `conf/hbase-site.xml` 参数配置
>
> 1. **设置RPC监听数量**
>
>    ```bash
>    #属性：hbase.regionserver.handler.count
>    # regionserver处理读写请求的线程数
>    #解释：默认值为30，用于指定RPC监听的数量，可以根据客户端的请求数进行调整，读写请求较多时，增加此值。
>    ```
>
> 2. **手动控制Major Compaction**
>
>    ```bash
>    #属性：hbase.hregion.majorcompaction
>    #解释：默认值：604800000秒（7天）， Major Compaction的周期，若关闭自动Major Compaction，可将其设为0
>    #一般关闭自动 Major Compaction,因为吃内存，可以手动进行Major Compaction
>    ```
>
> 3. **优化HStore文件大小**
>
>    ```bash
>    #属性：hbase.hregion.max.filesize
>    #解释：默认值10737418240（10GB），如果需要运行HBase的MR任务，可以减小此值，因为一个region对应一个map任务，如果单个region过大，会导致map任务执行时间过长。该值的意思就是，如果HFile的大小达到这个数值，则这个region会被切分为两个Hfile。
>    ```
>
> 4. **优化HBase客户端缓存**
>
>    ```bash
>    #属性：hbase.client.write.buffer
>    #解释：默认值2097152bytes（2M）用于指定HBase客户端缓存，增大该值可以减少RPC调用次数，但是会消耗更多内存，反之则反之。一般我们需要设定一定的缓存大小，以达到减少RPC次数的目的。
>    ```
>
> 5. **指定scan.next扫描HBase所获取的行数**
>
>    ```bash
>    #属性：hbase.client.scanner.caching
>    #解释：用于指定scan.next方法获取的默认行数，值越大，消耗内存越大。
>    #客户端Client向 RegionServer一次请求返回的行数
>    ```
>
> 6. **BlockCache（读缓存）占用RegionServer堆内存的比例**
>
>    ```bash
>    #属性：hfile.block.cache.size
>    #解释：默认0.4，读请求比较多的情况下，可适当调大
>    ```
>
> 7. **MemStore（写缓存）占用RegionServer堆内存的比例**
>
>    ```bash
>    #属性：hbase.regionserver.global.memstore.size
>    #解释：默认0.4，写请求较多的情况下，可适当调大
>    ```

+++



## 6-整合Phoenix

> Phoenix 是 HBase的开源SQL皮肤，使用标准的JDBC API 代替 HBase客户端API 来创建表，以及数据的增删改查
>
> + 使用简单，可以直接写sql
> + 效率没有手动设计 rowKey 再使用API高，性能较差
>
> +++
>
> + Phoenix架构
>
>   ![image-20220903164549698](http://ybll.vip/md-imgs/202209031645870.png)
>
> +++



### 6.1 安装

> ```bash
> #官网地址
> ```
>
> 官网地址: http://phoenix.apache.org
>
> ```bash
> #上传解压安装包
> tar -zxvf apache-phoenix-5.0.0-HBase-2.0-bin.tar.gz -C /opt/module/
> mv apache-phoenix-5.0.0-HBase-2.0-bin phoenix
> 
> #sql-hbase转换需要依赖的jar包,将其复制到 hbase/lib 下，再同步
> # phoenix-5.0.0-HBase-2.0-server.jar
> cp /opt/module/phoenix/phoenix-5.0.0-HBase-2.0-server.jar /opt/module/hbase/lib/
> xsync /opt/module/hbase/lib/phoenix-5.0.0-HBase-2.0-server.jar #分发
> ```
>
> + 在hbase-site.xml中添加支持二级索引的参数(如果不需要创建二级索引，不用不加)。之后分发到所有regionserver的节点上【分发到其他节点 xsync】
>
>   ```xml
>   <property>
>       <name>hbase.regionserver.wal.codec</name>
>       <value>org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec</value>
>   </property>
>   
>   <property>
>       <name>hbase.region.server.rpc.scheduler.factory.class</name>
>       <value>org.apache.hadoop.hbase.ipc.PhoenixRpcSchedulerFactory</value>
>   <description>Factory to create the Phoenix RPC Scheduler that uses separate queues for index and metadata updates</description>
>   </property>
>   
>   <property>
>       <name>hbase.rpc.controllerfactory.class</name>
>       <value>org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory</value>
>       <description>Factory to create the Phoenix RPC Scheduler that uses separate queues for index and metadata updates</description>
>   </property>
>   ```
>
> + 重启HBase,连接Phoenix
>
>   ```bash
>   stop-hbase.sh
>   start-hbase.sh
>   
>   #连接Phoenix
>   /opt/module/phoenix/bin/sqlline.py hadoop102,hadoop103,hadoop104:2181
>   #zookeeper为localhost时可以不写
>   /opt/module/phoenix/bin/sqlline.py #直接启动 Phoenix Shell 客户端，进行操作
>   ```
>
> +++
>
> Phoenix 与 HBase 的映射规则
>
> + Phoenix中 表的主键 => HBase 的 rowkey
> + Phoenix中 表的非主键列 => 普通字段 Coll，默认使用 `0` 作为列族，创建表时也可以手动指定：`列族.列名`
> + Phoenix 是多字段联合主键 => 将主键字段顺序拼接在一起，作为 rowkey 
>
> +++
>
> 其他说明：
>
> + Phoenix启动后，会默认在 default 库下创建一些表，不要去动，`list` 可以查看
>
> +++



### 6.2 Phoenix Shell 操作

> ```sql
> -- 显示所有表
> !table
> !tables
> 
> -- 创建表
> -- 注意：Phoenix 创建的表,表名，列名会默认转成大写，在hbase shell查询时，都是大写
> -- Phoenix Shell使用sql语句时，增删该查、创建等操作都无需使用大写
> create table if not exists student(
> -- 不能加　tab 键
> id varchar primary key, -- 单个列作为主键，映射为rowkey
> name varchar,
> addr varchar
> );
> 
> create table if not exists us_population(
> state char(2) not null,
> city varchar not null,
> population bigint,
> constraint my_pk primary key (state,city) -- 指定多个列作为主键
> );
> 
> --增删改查操作
> upsert into student values('1001','ybllcodes','beijing'); --新增、更新操作
> 
> select * from student;
> 
> delete from student where id='101';
> 
> drop table student;
> 
> !quit;
> ```
>
> > 说明：
> >
> > + Phoenix对表的操作，可以在  `scan 'STUDENT',{RAW=>TRUE,VERSIONS => 20}` 中查看操作过程
> > + 在Hbase Shell 上操作 Phoenix的表中数据（增删改查数据），Phoenix Shell的sql查询时无法捕获

### 6.3 Phoenix JDBC操作

> + 胖客户端:  hive cli（启动时加载很多jar包   客户端功能|sql解析功能） =====> sql ======> yarn
> + 瘦客户端:  beeline(启动时加载的jar包少  客户端功能) | datagrip ======> sql =======> hiveserver2(sql解析功能) ========> yarn
>
> +++
>
> 胖客户端：**将Phoenix的所有功能都集成在客户端，导致客户端代码打包后体积过大**
>
> + maven依赖
>
>   ```xml
>   <dependencies>
>       <dependency>
>           <groupId>org.apache.phoenix</groupId>
>           <artifactId>phoenix-core</artifactId>
>           <version>5.0.0-HBase-2.0</version>
>           <exclusions>
>               <exclusion>
>                   <groupId>org.glassfish</groupId>
>                   <artifactId>javax.el</artifactId>
>               </exclusion>
>           </exclusions>
>       </dependency>
>   
>       <dependency>
>           <groupId>org.apache.hadoop</groupId>
>           <artifactId>hadoop-common</artifactId>
>           <version>2.7.2</version>
>       </dependency>
>   </dependencies>
>   ```
>
> + 测试代码
>
>   ```java
>   import java.sql.*;
>   /**
>    * Created by Smexy on 2022/9/4
>    *
>    *      JDBC怎么玩?
>    *              ①获取Connection(url,driver(省略，根据url自动解析驱动),user,password)
>    *              ②获取PrepareStatement
>    *              ③填充占位符
>    *              ④执行sql
>    *              ⑤如果是读，返回一个ResultSet，遍历ResultSet获取结果
>    *              ⑥关闭连接
>    */
>   public class FatClientTest
>   {
>       public static void main(String[] args) throws SQLException {
>   
>           // 1.添加链接   所有的hbase客户端，去连接hbase所使用的zk
>           String url = "jdbc:phoenix:hadoop102:2181";
>           // 2.获取连接
>           Connection connection = DriverManager.getConnection(url);
>           // 3.编译SQL语句
>           PreparedStatement preparedStatement = connection.prepareStatement("select * from STUDENT");
>           // 4.执行语句
>           ResultSet resultSet = preparedStatement.executeQuery();
>           // 5.输出结果
>           while (resultSet.next()){
>               System.out.println(resultSet.getString(1) + ":" + resultSet.getString(2) + ":" + resultSet.getString(3));
>           }
>           // 6.关闭资源
>           connection.close();
>       }
>   }
>   ```
>
> +++
>
> 瘦客户端：**将Phoenix的功能进行拆解，主要功能由服务端提供，只使用轻量级的客户端向服务端发送请求**
>
> + 启动 query server
>
>   ```bash
>   /opt/module/phoenix/bin/queryserver.py start
>   ```
>
> + maven依赖
>
>   ```xml
>       <!--
>           客户端只负责编写sql，将sql发送给 hadoop103:8765端口即可。
>           大部分解析sql的功能，交给服务实现。 在虚拟机上启动服务。
>       -->
>       <dependencies>
>           <dependency>
>               <groupId>org.apache.phoenix</groupId>
>               <artifactId>phoenix-queryserver-client</artifactId>
>               <version>5.0.0-HBase-2.0</version>
>           </dependency>
>       </dependencies>
>   ```
>
> + 测试代码
>
>   ```java
>   public class ThinClientTest
>   {
>       public static void main(String[] args) throws SQLException {
>   
>           // 1.添加链接  去连接 phoenixServer服务，由phoenixServer再去连接hbase
>           String url = ThinClientUtil.getConnectionUrl("hadoop102", 8765);
>           // 2.获取连接
>           Connection connection = DriverManager.getConnection(url);
>           // 3.编译SQL语句
>           PreparedStatement preparedStatement = connection.prepareStatement("select * from student");
>           // 4.执行语句
>           ResultSet resultSet = preparedStatement.executeQuery();
>           // 5.输出结果
>           while (resultSet.next()){
>               System.out.println(resultSet.getString(1) + ":" + resultSet.getString(2) + ":" + resultSet.getString(3));
>           }
>           // 6.关闭资源
>           connection.close();
>       }
>   }
>   ```

+++



### 6.4 Phoenix 二级索引

> ![image-20220905142608385](http://ybll.vip/md-imgs/202209051426470.png)
>
> + 创建了索引表，hdfs上只会有对应的 库-表-region-列族 目录，目录下没有实际的存储文件
>
>   ![image-20220905135623579](http://ybll.vip/md-imgs/202209051409029.png)
>
> + 删除索引
>
>   ```bash
>    drop index index_name_addr_2 on student;
>   ```
>
>   + 删除索引，hdfs上的目录也会删除
>
> + 二级索引分类：单值索引、复合索引、包含索引
>
>   > ![image-20220905142652757](http://ybll.vip/md-imgs/202209051426846.png)
>   >
>   > + 单值索引
>   >
>   >   ```bash
>   >   create index index_stu_name on student(name); #单值，创建了一张表，index_stu_name
>   >   
>   >   explain select id,name from student where name = 'yb'; #查看该语句执行计划
>   >   #索引前 full scan over student
>   >   #索引后 range scan over index_stu_name[ 'yb' ]
>   >   ```
>   >
>   >   ![image-20220905133503061](http://ybll.vip/md-imgs/202209051413891.png)
>   >
>   >   +++
>   >
>   >   + 创建了一张新表，以 **索引字段** 和 **原表rowkey** 为 **新的rowkey**，因此索引字段可以重复
>   >
>   >     ![image-20220905133648396](http://ybll.vip/md-imgs/202209051416682.png)
>   >
>   >     +++
>   >
>   >     ![image-20220905134700019](http://ybll.vip/md-imgs/202209051421023.png)
>   >
>   >   
>   >
>   >   + 原表数据发生变化，索引表也会随即更新
>   >
>   >     ![image-20220905134220126](http://ybll.vip/md-imgs/202209051419095.png)
>   >
>   >     
>   >
>   > + 复合索引
>   >
>   >   ```bash
>   >   create index index_name_addr student(name,addr) #复合索引
>   >   ```
>   >
>   >   ![image-20220905135446924](http://ybll.vip/md-imgs/202209051423293.png)
>   >
>   > + 包含索引
>   >
>   >   ```bash
>   >   create index index_name_addr_2 on student(name) include(addr); #包含索引
>   >   ```
>   >
>   >   ![image-20220905140453933](http://ybll.vip/md-imgs/202209051424760.png)
>   >
>   >   
>   >
>   >   
>
> + 索引的存储位置分：本地索引、全局索引
>
>   > ![image-20220905142732971](http://ybll.vip/md-imgs/202209051427055.png)
>   >
>   > + 本地索引：在数据表中新建一个列族来存储索引数据。避免了在写操作的时候往不同服务器的索引表中写索引带来的额外开销
>   >
>   >   ```bash
>   >   create local index my_index on student(my_column)
>   >   ```
>   >
>   > +++

+++



# ClickHouse

官方文档：https://clickhouse.com/docs/zh/

## 1-ClickHouse简介

> + ClickHouse 是俄罗斯的 Yandex(类似国内百度，最大的搜索引擎公司) 在 2016 年开源的 **列式存储数据库（DBMS）**
> + 使用 c++ 编写，主要用于 **在线分析处理查询（OLAP）**
> + 可以使用SQL 查询实时生成分析数据报告
>
> +++
>
> 特点：
>
> + 列式存储（列式存储适合查询，行式存储适合更新，插入)
>   + 对于列的聚合，计数，求和等统计操作，列式存储优于行式存储
>   + 同一列数据类型一致，针对数据存储更容易进行数据压缩，可以选择更优的数据压缩算法，提高数据的压缩比重
>   + 压缩比重大，一方面节省磁盘空间，另一方面使得缓存cache更好地发挥
> + DBMS的功能：基本覆盖 标准SQL的语法，包括DDL，DML，各种函数，用户管理及权限管理，数据备份和恢复
> + 多样化引擎：和Mysql类似，有更多的引擎（表引擎，库引擎）；包括**合并树**，日志，接口，其他 四大类20多种引擎
> + 高吞吐写入能力
>   + 采用 ***LSM Tree*** 结构（实现顺序写磁盘），数据写入后 **定期在后台合并（Compaction）** 
>   + 数据导入时，全部顺序写磁盘（append写），写入后数据段不可更改，只能追加写
>   + compaction合并操作也是多个段merge sort后，顺序写磁盘
>   + 顺序写的特性：充分利用了磁盘的吞吐能力
>   + 官方测试显示：clickhouse能够达到 50MB-200MB/s的写入吞吐能力，设每行100B,能够达到50w-200w条/s的写入速度
> + 数据分区与**线程级别的并行**
>   + 表数据划分为**多个partition**,每个partition再进一步划分为多个**indexgranularity（索引粒度）**
>   + 通过多个CPU核心，分别处理一部分来实现并行数据处理，因此**单条Query就能利用整机所有CPU**
>   + 极致的并行处理能力，极大地降低了查询延时，但是不适合**高 qps** 的业务查询

## 2-ClickHouse入门

### 2.1 ClickHouse安装

> 1. 取消CentOS打开文件数限制(若集群启动，需要同步)
>
>    > ```properties
>    > sudo vim /etc/security/limits.conf
>    > #
>    > * soft nofile 65536 
>    > * hard nofile 65536 
>    > * soft nproc 131072 
>    > * hard nproc 131072
>    > 
>    > # * => 所有用户所有组，单用户配置：atguigu@atguigu
>    > # soft/hard => 允许实际存活的数量/最大的数量 soft <= hard
>    > # nofile => Number file,文件数量
>    > # nproc => Number proc ，进程数量
>    > ```
>
> 2. 安装依赖
>
>    ```bash
>    sudo yum install -y libtool
>    sudo yum install -y *unixODBC*
>    ```
>
> 3. CentOS 取消 SELINUX （Security Enforce Linux：Linux最高的安全保护模式）
>
>    ```bash
>    sudo vim /etc/selinux/config
>    	SELINUX=disable
>    #修改后，需要重启生效
>    
>    #查看状态
>    getenforce
>    
>    #临时关闭
>    setenforce 0
>    ```
>
> 4. 安装ClickHouse（单机）
>
>    > ```bash
>    > mkdir /opt/software/ck
>    > #复制4个rpm包到该目录（21.7.3版本）
>    > clickhouse-client-21.7.3.14-2.noarch.rpm
>    > clickhouse-common-static-21.7.3.14-2.x86_64.rpm
>    > clickhouse-common-static-dbg-21.7.3.14-2.x86_64.rpm
>    > clickhouse-server-21.7.3.14-2.noarch.rpm
>    > 
>    > #执行安装
>    > sudo rpm -ivh /opt/software/ck/*.rpm
>    > #clickhouse-server安装时，默认 default用户，需要输入密码，可以不设置，直接回车
>    > 
>    > sudo rpm -qa | grep clickhouse #查看安装情况
>    > 
>    > #修改配置文件
>    > sudo vim /etc/clickhouse-server/config.xml
>    > 	#修改改行，使得任意远程节点都能访问该 clickhouse
>    > 	<listen_host>0.0.0.0</listen_host>
>    > 	
>    > #启动Server(启动后，默认开机自启 clickhouse-server)
>    > sudo clickhouse start
>    > #restart 重启
>    > 
>    > #####以下命令可选
>    > #sudo systemctl start clickhouse-server #启动Server
>    > #sudo systemctl disable clickhouse-server #关闭开机自启
>    > 
>    > #click 连接 server
>    > clickhouse-client -h hadoop102 -m
>    > # -h => 指定主机地址
>    > # -m => 可以在命令窗口数据多行命令
>    > clickhouse-client -h hadoop102 --query 'show databases;'
>    > # --query => 类似 hive -e 'hql'
>    > 
>    > ```
>    >
>    > + rpm安装 ClickHouse对应的目录：
>    >
>    >   ![image-20220906115717322](http://ybll.vip/md-imgs/202209061157427.png)
>    >
>    > + 卸载 ClickHouse
>    >
>    >   ```bash
>    >   #查看是否安装
>    >   sudo rpm -qa | grep clickhouse #查看安装情况
>    >   #卸载
>    >   rpm -qa | grep clickhouse | sudo xargs rpm -e 
>    >   
>    >   #删除数据和配置文件
>    >   sudo rm -rf /var/lib/clickhouse/ 
>    >   sudo rm -rf /etc/clickhouse-* s
>    >   sudo rm -rf /var/log/clickhouse-server/
>    >   
>    >   #集群模式
>    >   rm -rf /etc/metrika.xml #linux文件
>    >   rmr /clickhouse #删除zk上元数据
>    >   ```
>    >
>    > + 其他说明
>    >
>    >   > ![image-20220906120023310](http://ybll.vip/md-imgs/202209061200379.png)
>    >   >
>    >   > +++
>    >   >
>    >   > ![image-20220906120935251](http://ybll.vip/md-imgs/202209061209303.png)
>    >   >
>    >   > +++
>    >   >
>    >   > ![image-20220906121030549](http://ybll.vip/md-imgs/202209061210620.png)
>    >
>    >   ```txt
>    >   ClickHouse各文件目录：
>    >       bin/    ===>  /usr/bin/ 
>    >       conf/   ===>  /etc/clickhouse-server/
>    >       lib/    ===>  /var/lib/clickhouse 
>    >       log/    ===>  /var/log/clickhouse-server
>    >   
>    >   
>    >   PartitionId_MinBlockNum_MaxBlockNum_Level
>    >   分区值_最小分区块编号_最大分区块编号_合并层级
>    >       =》PartitionId
>    >           数据分区ID生成规则
>    >           数据分区规则由分区ID决定，分区ID由PARTITION BY分区键决定。根据分区键字段类型，ID生成规则可分为：
>    >               未定义分区键
>    >                   没有定义PARTITION BY，默认生成一个目录名为all的数据分区，所有数据均存放在all目录下。
>    >   
>    >               整型分区键
>    >                   分区键为整型，那么直接用该整型值的字符串形式做为分区ID。
>    >   
>    >               日期类分区键
>    >                   分区键为日期类型，或者可以转化成日期类型。
>    >   
>    >               其他类型分区键
>    >                   String、Float类型等，通过128位的Hash算法取其Hash值作为分区ID。
>    >       =》MinBlockNum
>    >           最小分区块编号，自增类型，从1开始向上递增。每产生一个新的目录分区就向上递增一个数字。
>    >       =》MaxBlockNum
>    >           最大分区块编号，新创建的分区MinBlockNum等于MaxBlockNum的编号。
>    >       =》Level
>    >           合并的层级，被合并的次数。合并次数越多，层级值越大。
>    >           
>    >           
>    >   bin文件：数据文件
>    >   mrk文件：标记文件
>    >       标记文件在 idx索引文件 和 bin数据文件 之间起到了桥梁作用。
>    >       以mrk2结尾的文件，表示该表启用了自适应索引间隔。
>    >   primary.idx文件：主键索引文件，用于加快查询效率。
>    >   minmax_create_time.idx：分区键的最大最小值。
>    >   checksums.txt：校验文件，用于校验各个文件的正确性。存放各个文件的size以及hash值。
>    >   ```
>
>    +++
>
> +++



### 2.2 数据类型(大小写敏感)

1. 整型

   > 固定长度的整型，包括有符号整型或无符号整型。
   >
   > 整型范围（-2n-1~2n-1-1）：
   >
   > Int8 - [-128 : 127]
   >
   > Int16 - [-32768 : 32767]
   >
   > Int32 - [-2147483648 : 2147483647]
   >
   > Int64 - [-9223372036854775808 : 9223372036854775807]
   >
   > 无符号整型范围（0~2n-1）：
   >
   > UInt8 - [0 : 255]
   >
   > UInt16 - [0 : 65535]
   >
   > UInt32 - [0 : 4294967295]
   >
   > UInt64 - [0 : 18446744073709551615]
   >
   > **使用场景：** **个数、数量、也可以存储型id**

2. 浮点型

   > Float32 - float
   >
   > Float64 – double
   >
   > 建议尽可能以整数形式存储数据。例如，将固定精度的数字转换为整数值，如时间用毫秒为单位表示，因为浮点型进行计算时可能引起四舍五入的误差。
   >
   > ![image-20220906121536027](http://ybll.vip/md-imgs/202209061215084.png)
   >
   > **使用场景：一般数据值比较小，不涉及大量的统计计算，精度要求不高的时候。比如保存商品的重量。**

3. 布尔型

   > 没有单独的类型来存储布尔值。可以使用 UInt8 类型，取值限制为 0 或 1

4. Decimal 型

   > + 有符号的浮点数，可在加、减和乘法运算过程中保持精度。对于除法，最低有效数字会被丢弃（不舍入）
   >
   > +++
   >
   > **Decimal32(s)，相当于Decimal(9-s,s)，有效位数为1~9**
   >
   > **Decimal64(s)，相当于Decimal(18-s,s)，有效位数为1~18**
   >
   > **Decimal128(s)，相当于Decimal(38-s,s)，有效位数为1~38**
   >
   > + s表示保留小数点后的位数
   >
   > + 使用场景： 一般金额字段、汇率、利率等字段为了保证小数点精度，都使用Decimal进行存储。

5. 字符串

   > 1. String
   >
   > 字符串可以任意长度的。它可以包含任意的字节集，包含空字节。
   >
   > 2. FixedString(N)
   >
   > 固定长度 N 的字符串，N 必须是严格的正自然数。当服务端读取长度小于 N 的字符串时候，通过在字符串末尾添加空字节来达到 N 字节长度。 当服务端读取长度大于 N 的字符串时候，将返回错误消息
   >
   > +++
   >
   > + 与String相比，极少会使用FixedString，因为使用起来不是很方便。
   >
   > + 使用场景：名称、文字描述、字符型编码。 固定长度的可以保存一些定长的内容，比如一些编码，性别等但是考虑到一定的变化风险，带来收益不够明显，所以定长字符串使用意义有限。

6. 枚举类型

   > + `Enum8` `Enum16`,保存 `'String'=Int8 | Int16` 的对应关系
   >
   >   ```sql
   >   CREATE TABLE t_enum
   >   (
   >       x Enum8('hello' = 1, 'world' = 2)
   >   )
   >   ENGINE = TinyLog;
   >   
   >   SELECT CAST(x, 'Int8') FROM t_enum;
   >   ```
   >
   > + 使用场景：对一些状态、类型的字段算是一种空间优化，也算是一种数据约束。但是实际使用中往往因为一些数据内容的变化增加一定的维护成本，甚至是数据丢失问题。所以谨慎使用。

7. 时间类型（与hive区分，一般不使用 String存储）

   > 目前ClickHouse 有三种时间类型
   >
   > + `Date` 接受 **年-月-日** 的字符串比如 ‘2019-12-16’
   >
   > + `Datetime` 接受 **年-月-日 时:分:秒 **的字符串比如 ‘2019-12-16 20:50:10’
   >
   > + `Datetime64` 接受 **年-月-日 时:分:秒.亚秒 **的字符串比如‘2019-12-16 20:50:10.66’
   >
   > 日期类型，用两个字节存储，表示从 1970-01-01 (无符号) 到当前的日期值。
   
8. 数组

   > **Array(T)**：由 T 类型元素组成的数组。![image-20220906122552305](http://ybll.vip/md-imgs/202209061225385.png)
   >
   > T 可以是任意类型，包含数组类型。 但不推荐使用多维数组，ClickHouse 对多维数组的支持有限。例如，不能在MergeTree表中存储多维数组。
   >
   > +++
   >
   > ```sql
   > SELECT array(1, 2) AS x, toTypeName(x) ;
   > SELECT [1, 2] AS x, toTypeName(x);
   > ```
   >
   > +++

   

## 3-表引擎(大小写敏感)

> + 表引擎是 ClickHouse 的一大特色，决定了如何存储表的数据，包括：
>   + 数据的存储方式和位置，写到哪里以及从哪里读取数据
>   + 支持哪些查询以及如何支持
>   + 并发数据访问
>   + 索引的使用（如果存在）
>   + 是否可以执行多线程请求
>   + 数据复制参数
> + 使用方式：建表时，显式地定义该表使用的引擎，以及引擎使用的相关参数
> + 引擎名称的大小写敏感

+++

***TinyLog***

> 以列文件的形式保存在磁盘上，不支持索引，没有并发控制。一般保存少量数据的小表，生产环境上作用有限。可以用于平时练习测试用
>
> ```sql
> create table t_tinylog ( id String, name String) engine=TinyLog;
> ```
>
> +++

***Memory***

> + 内存引擎，数据以未压缩的原始形式直接保存在内存当中，服务器重启数据就会消失
> + 读写操作不会相互阻塞，不支持索引
> + 简单查询下有非常非常高的性能表现（超过10G/s）
>
> +++
>
> + 一般用到它的地方不多，测试场景
> + 在需要非常高的性能，同时数据量又不太大（上限大概 1 亿行）的场景
>
> +++

### MergeTree

> + ClickHouse中最强大的表引擎：MergeTree（合并树）以及该系列中的其他引擎
> + 支持索引和分区【地位相当于 innodb(支持事务)于 MySQL】
>
> +++
>
> 案例：
>
> ```sql
> create table t_order_mt(
>     id UInt32,
>     sku_id String,
>     total_amount Decimal(16,2),
>     create_time Datetime
>  ) engine =MergeTree
>    partition by toYYYYMMDD(create_time)
>    primary key (id)
>    order by (id,sku_id);
> -- MergeTree其实还有很多参数(绝大多数用默认值即可)，
> -- 这三个参数是更加重要的，也涉及了关于MergeTree的很多概念
> 
> insert into  t_order_mt values
> (101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
> (102,'sku_002',2000.00,'2020-06-01 11:00:00'),
> (102,'sku_004',2500.00,'2020-06-01 12:00:00'),
> (102,'sku_002',2000.00,'2020-06-01 13:00:00'),
> (102,'sku_002',12000.00,'2020-06-01 13:00:00'),
> (102,'sku_002',600.00,'2020-06-02 12:00:00');
> ```
>
> ![image-20220906124324372](http://ybll.vip/md-imgs/202209061243468.png)
>
> +++
>
> **1. partition by 分区【可选】**
>
> + 目的：降低扫描的范围，优化查询速度
>
> + 分目录：MergeTree是以列文件 + 索引文件 + 表定义文件组成；设定了分区，则这些文件就会保存在不同的分区目录中
>
> + 并行：分区后，跨分区的查询统计，ClickHouse会以分区为单位，并行处理
>
> + 数据写入与合并分区
>
>   + 任何批次的数据写入时，都会产生一个临时分区，不会纳入已有分区
>
>   + 等待固定时间（10-15min）,ClickHouse会自动执行合并操作，把临时分区的数据合并到已有分区
>
>     ```bash
>     optimize table tblname final; #手动合并临时分区到已有分区
>     optimize table tblname partition pname final;
>     ```
>
>   + 举例：再次执行上述插入语句，select时会有4个分区（5,1,5,1）;手动合并后，只有两个分区（2,10）
>
> +++
>
> **2. parmary key 主键【可选】**
>
> + 要求：***主键必须是order by字段的前缀字段***
>
> + ClickHouse中的主键，提供了一级索引，但是没有唯一约束，因此可能存在 相同primary key的数据
>
> + 主键设定需要依据 查询语句中的 where 条件；通过二分查找，能够定位到对应的 index granularity（索引粒度）,避免了全表扫描
>
> + index pranularity：在 **稀疏索引** 中两个相邻索引对应数据的间隔，默认是 8192；一般不修改，除非该列存在大量重复值（大于8192）
>
> + 稀疏索引
>
>   ![image-20220906181749010](http://ybll.vip/md-imgs/202209061819046.png)
>
>   + 稀疏索引的好处就是可以用很少的索引数据，定位更多的数据，代价就是只能定位到索引粒度的第一行，然后再进行进行一点扫描
>
> +++
>
> ***3. order by 【必选】***
>
> + order by 设定了分区内的数据按照哪些字段顺序进行有序保存
> + order by是MergeTree中唯一一个必填项，比primary key 还重要
> + 因为当用户不设置主键的情况，很多处理会依照order by的字段进行处理
>
> +++
>
> **4. 二级索引**
>
> + (v20.1.2.4版本前需要手动开启)
>
>   ```bash
>   set allow_experimental_data_skipping_indices=1;
>   ```
>
> + 新版本默认开启，且该参数已经被删除
>
>   +++
>
> + 如何创建
>
>   ```sql
>   create table t_order_mt2(
>       id UInt32,
>       sku_id String,
>       total_amount Decimal(16,2),
>       create_time  Datetime,
>   	INDEX a total_amount TYPE minmax GRANULARITY 5
>    ) engine =MergeTree
>      partition by toYYYYMMDD(create_time)
>      primary key (id)
>      order by (id, sku_id);
>   ```
>
>   > INDEX a total_amount  : 创建索引a，对应 total_amount 字段
>   >
>   > GRANULARITY 5 : 设定二级索引 对于 一级索引 的粒度
>   >
>   > TYPE minmax  : 计算得到二级索引粒度下， total_amount的最大值/最小值，用于比较
>   >
>   > +++
>   >
>   > 每 5 * 8192 行数据计算出 一对 total_amount 的最大值/最小值，当扫描时，会先 比对最大值/最小值，如果不在区间内，直接跳过当前区域的扫描
>   >
>   > ![image-20220906183008887](http://ybll.vip/md-imgs/202209061830045.png)
>
> +++
>
> **5. 数据TTL**
>
> + TTL即Time To Live，MergeTree提供了可以管理数据或者列的生命周期的功能
>
> + **必须靠触发合并操作才能实现数据的时效。**TTL运算的基础字段必须是日期类型，例 Datetime
>
>   +++
>
> + 列级别 TTL
>
>   > ```mysql
>   > create table t_order_mt3(
>   >  id UInt32,
>   >  sku_id String,
>   >  total_amount Decimal(16,2) TTL create_time+interval 10 SECOND,
>   >  create_time Datetime 
>   > ) engine =MergeTree
>   > partition by toYYYYMMDD(create_time)
>   >  primary key (id)
>   >  order by (id, sku_id);
>   >  
>   > insert into t_order_mt3 values
>   > (106,'sku_001',1000.00,'2020-06-12 22:52:30'),
>   > (107,'sku_002',2000.00,'2020-06-12 22:52:30'),
>   > (110,'sku_003',600.00,'2020-06-13 12:00:00');
>   > ```
>   >
>   > + 在合并时，需要启动一个合并任务，如果机器CPU不行，任务无法及时执行，此时重启服务端，然后手动合并
>
> + **表级别 TTL**
>
>   > ```sql
>   > -- 插入数据
>   > insert into  t_order_mt values
>   > (106,'sku_001',1000.00,'2020-06-12 22:52:30'),
>   > (107,'sku_002',2000.00,'2020-06-12 22:52:30'),
>   > (110,'sku_003',600.00,'2021-06-13 12:00:00')
>   > 
>   > -- 修改表信息
>   > alter table t_order_mt MODIFY TTL create_time + INTERVAL 10 SECOND;
>   > -- 注意：
>   > -- 表中的每一列都会用 create_time进行判断，时间到了就删除；时间没到不删除
>   > -- 判断的字段必须是Date或者Datetime类型，推荐使用分区的日期字段,且ttl不能使用primary key
>   > 
>   > --能够使用的时间周期
>   > -- SECOND
>   > -- MINUTE
>   > -- HOUR
>   > -- DAY
>   > -- WEEK
>   > -- MONTH
>   > -- QUARTER
>   > -- YEAR 
>   > 
>   > ```
>
>   +++

+++

### ReplacingMergeTree

> + MergeTree的一个变种，存储特性完全继承MergeTree，只是多了一个去重的功能
> + 去重时机：**同一批插入的数据** 以及 **同一分区的多文件 合并的过程**
> + 去重范围：只会在分区内部去重，不能跨分区（不同分区，仍然会有重复）
> + 去重依据：依据**primary key** 进行去重
>   + primary key相同 ：保留版本字段最大的
>   + primary key相同，版本字段相同：按插入顺序保留最后一个 
> + 有限幂等性
>
> +++
>
> 案例
>
> ```sql
> -- 创建表
> create table t_order_rmt(
>     id UInt32,
>     sku_id String,
>     total_amount Decimal(16,2) ,
>     create_time  Datetime 
>  ) engine =ReplacingMergeTree(create_time)
>    partition by toYYYYMMDD(create_time)
>    primary key (id)
>    order by (id, sku_id);
> -- 版本字段为：create_time
> -- 不设置版本字段，则按照插入顺序保留最后一条
> 
> -- 插入数据
> insert into  t_order_rmt values
> (101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
> (102,'sku_002',2000.00,'2020-06-01 11:00:00'),
> (102,'sku_004',2500.00,'2020-06-01 12:00:00'),
> (102,'sku_002',2000.00,'2020-06-01 13:00:00'),
> (102,'sku_002',12000.00,'2020-06-01 13:00:00'),
> (102,'sku_002',600.00,'2020-06-02 12:00:00');
> 
> -- 手动合并
> OPTIMIZE TABLE t_order_rmt FINAL;
> ```
>
> +++



### SummingMergeTree

