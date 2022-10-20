# MYSQL

## 课堂笔记

### 一 MySQL概述

```mysql
/*
1.MySQL作用：①持久化保存数据  ②可以方便的对数据进行增，删，改，查的操作
	
2.概念
	DB : 数据库
	DBMS ：数据库管理系统
	SQL：结构化查询语言
	
3.常见的数据库管理系统 ：MySQL，SQLServer,Oracle......

4.SQL的优点：①简单易学  ②所有数据库支持 ③虽然简单但是一种强有力的语言
	
5.SQL(Structural query language)语句分为以下三种类型：
	DML: Data Manipulation Language 数据操纵语言
	DDL:  Data Definition Language 数据定义语言
	DCL:  Data Control Language 数据控制语言

*/
```

常用的操作：

```mysql
#查看所有的库：
show databases;
#查看所有的表
show tables;
#选库
use 库名;
#输出1+1
select 1+1 from dual;#dual：伪表，可以省略不写
select 1+1;
#任何数据类型和null做运算结果为null
select 1+null;
```

##### 1.1 库的DDL操作

```mysql
#创建库 
create database [if not exists] 库名 [character set '编码集'];
#if not exists :有该字段库存在则不会报错否则会报错.
create database if not exists 库名;

#指定库的编码集
create database 库名 character set '编码集';
 
#修改库的编码集
alter database 库名 character set '编码集';

#删除库
#if exists :有该字段库不存在不报错，没有该字段库不存在则报错。
drop database [if exists] 库名;

#选库
use 库名;

#查看库的信息
show create database 库名;

#查看所有的库
show databases;
```

##### 1.2 表的DDL操作

```mysql
#查看所有的表
show tables;

#查看表结构
desc 表名：

#查看表的信息
show create table 表名;

#删除表
#if exists :有该字段表不存在不报错，没有该字段表不存在则报错。
drop table [if exists] 表名;

#清空表
truncate table 表名;

#创建表
#方式一：白手起家
#如果没有指定编码集那么表的默认编码集和库的编码集一致
create table [if exists] 表名(
字段名1 类型，
字段名2 类型，
......
字段名n 类型
)character set '编码集';

#方式二 ：基于现有的表结构创建新表（没有内容）
create table 表名 like 表名;

#方式三 ：将查询结果创建成一张新表
create table 表名
as
select 查询语句;
```

```mysql
####### alter table
#添加字段
alter table 表名 add [column] 字段名 类型;

#删除字段
alter table 表名 drop [column] 字段名;

#修改字段的名字
alter table 表名 change [column] 原字段名 新字段名 字段的类型;

#修改字段的类型
alter table 表名 modify [column] 字段名 字段的类型;

#改表名
alter table 原表名 rename to 新表名;
```

##### 1.3 表的增删改操作

```mysql
/*
insert into 表名(字段名1，字段名2，......) values(值1，值2,.....),(值1，值2,.....),......
*/
#插入单行数据
insert into 表名(字段名1，字段名2，......) values(值1，值2,.....);
#如果是插入的是全字段那么不用在表名后再加字段名
insert into 表名 values(值1，值2,.....);
#插入多行数据
insert into 表名(字段名1，字段名2，......) values(值1，值2,.....),(值1，值2,.....),......
#将查询的结果插入到表中
insert into 表名(字段名1，字段名2，......)
select 字段名1，字段名2，......
xxxx
#注意：查询的字段的个数和类型要和被插入的字段的个数和类型保持一致
```

```mysql
 #删除表中所有的数据
 delete from 表名;
 #指定删除表中哪些数据
 delete from 表名 where 过滤条件;
```

```mysql
#修改表中的所有内容
update 表名 set 字段名1=值1，字段名2=值2，......;
#指定修改表中哪些数据
update 表名 set 字段名1=值1，字段名2=值2，...... WHERE 过滤条件;
```

##### 1.4 其他说明

```mysql
#1.truncate table不能进行事务的回滚。delete from可以进行事务回滚
#2.如果要删除的数据确实不需要回滚那么使用truncate table效率更高
```

```mysql
#使用 ：
limit 索引的位置,数据的条数 
#注意 ： 索引是从0开始
#分页公式 ：
limit （页数-1）*每页条数,每页条数
```



### 二 函数

##### 2.1单行函数

> 返回一个结果，参数一列或一个值

###### 1-基本函数

| 函数                 | 用法                                                         |
| -------------------- | ------------------------------------------------------------ |
| **ABS(x)**           | 返回x的绝对值                                                |
| SIGN(X)              | 返回X的符号。正数返回1，负数返回-1， 0返回0                  |
| PI()                 | 返回圆周率的值                                               |
| CEIL(x)， CEILING(x) | **返回大于或等于某个值的最小整数**                           |
| FLOOR(x)             | **返回小于或等于某个值的最大整数**                           |
| LEAST(e1,e2,e3  …)   | **返回列表中的最小值**                                       |
| GREATEST(e1,e2,e3 …) | 返回列表中的最大值                                           |
| MOD(x,y)             | 返回X除以Y后的余数                                           |
| RAND()               | **返回0~1的随机值**                                          |
| RAND(x)              | 返回0~1的随机值，其中x的值用作种子值，相同的X值会产生相同的随机 数 |
| ROUND(x)             | 返回一个对x的值进行四舍五入后，最接近于X的整数               |
| ROUND(x,y)           | **返回一个对x的值进行四舍五入后最接近X的值，并保留到小数点后面Y位** |
| TRUNCATE(x,y)        | 返回数字x截断为y位小数的结果                                 |
| SQRT(x)              | 返回x的平方根。当X的值为负数时，返回NULL                     |



###### 2-数学相关

+ 角度与弧度互换函数

| 函数       | 用法                                  |
| ---------- | ------------------------------------- |
| RADIANS(x) | 将角度转化为弧度，其中，参数x为角度值 |
| DEGREES(x) | 将弧度转化为角度，其中，参数x为弧度值 |

+++

+ 三角函数

| 函数       | 用法                                                         |
| ---------- | ------------------------------------------------------------ |
| SIN(x)     | 返回x的正弦值，其中，参数x为弧度值                           |
| ASIN(x)    | 返回x的反正弦值，即获取正弦为x的值。如果x的值不在-1到1之间，则返回NULL |
| COS(x)     | 返回x的余弦值，其中，参数x为弧度值                           |
| ACOS(x)    | 返回x的反余弦值，即获取余弦为x的值。如果x的值不在-1到1之间，则返回NULL |
| TAN(x)     | 返回x的正切值，其中，参数x为弧度值                           |
| ATAN(x)    | 返回x的反正切值，即返回正切值为x的值                         |
| ATAN2(m,n) | 返回两个参数的反正切值                                       |
| COT(x)     | 返回x的余切值，其中， X为弧度值                              |

+++

+ 指数与对数

| 函数                  | 用法                                                |
| --------------------- | --------------------------------------------------- |
| POW(x,y)， POWER(X,Y) | 返回x的y次方                                        |
| EXP(X)                | 返回e的X次方，其中e是一个常数， 2.718281828459045   |
| LN(X)， LOG(X)        | 返回以e为底的X的对数，当X<= 0 时，返回的结果为NULL  |
| LOG10(X)              | 返回以10为底的X的对数，当X<= 0 时，返回的结果为NULL |
| LOG2(X)               | 返回以2为底的X的对数，当X<= 0 时，返回NULL          |

+++

+ 进制转换

| 函数          | 用法                     |
| ------------- | ------------------------ |
| BIN(x)        | 返回x的二进制编码        |
| HEX(x)        | 返回x的十六进制编码      |
| OCT(x)        | 返回x的八进制编码        |
| CONV(x,f1,f2) | 返回f1进制数变成f2进制数 |



###### 3-字符串函数

+ mysql中，字符串的位置从1开始

| ![img](file:///C:/Users/YBLLCO~1/AppData/Local/Temp/msohtmlclip1/01/clip_image001.gif)函数 | 用法                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ASCII(S)                                                     | 返回字符串S中的第一个字符的ASCII码值                         |
| CHAR_LENGTH(s)                                               | 返回字符串s的字符数。作用与CHARACTER_LENGTH(s)相同           |
| LENGTH(s)                                                    | **返回字符串s的字节数，和字符集有关**                        |
| CONCAT(s1,s2,......,sn)                                      | **连接s1,s2,......,sn为一个字符串**                          |
| CONCAT_WS(x,  s1,s2,......,sn)                               | **同CONCAT(s1,s2,...)函数，但是每个字符串之间要加上x**       |
| INSERT(str, idx, len,  replacestr)                           | 将字符串str从第idx位置开始， len个字符长的子串替换为字符串replacestr |
| REPLACE(str, a, b)                                           | 用字符串b替换字符串str中所有出现的字符串a                    |
| UPPER(s) 或 UCASE(s)                                         | 将字符串s的所有字母转成大写字母                              |
| LOWER(s) 或LCASE(s)                                          | 将字符串s的所有字母转成小写字母                              |
| LEFT(str,n)                                                  | 返回字符串str最左边的n个字符                                 |
| RIGHT(str,n)                                                 | 返回字符串str最右边的n个字符                                 |
| LPAD(str, len, pad)                                          | 用字符串pad对str最左边进行填充，直到str的长度为len个字符     |
| RPAD(str  ,len, pad)                                         | 用字符串pad对str最右边进行填充，直到str的长度为len个字符     |
| LTRIM(s)                                                     | 去掉字符串s左侧的空格  ltirm(s)                              |
| RTRIM(s)                                                     | 去掉字符串s右侧的空格  rtrim(s)                              |
| TRIM(s)                                                      | 去掉字符串s开始与结尾的空格                                  |
| TRIM(s1 FROM s)                                              | 去掉字符串s开始与结尾的s1                                    |
| TRIM(LEADING  s1  FROM s)                                    | 去掉字符串s开始处的s1  leading                               |
| TRIM(TRAILING  s1  FROM s)                                   | 去掉字符串s结尾处的s1  trailing                              |
| REPEAT(str, n)                                               | 返回str重复n次的结果                                         |
| SPACE(n)                                                     | 返回n个空格                                                  |
| STRCMP(s1,s2)                                                | 比较字符串s1,s2的ASCII码值的大小                             |
| SUBSTR(s,index,len)                                          | 返回从字符串s的index位置其len个字符，作用与SUBSTRING(s,n,len)、 MID(s,n,len)相同 |
| LOCATE(substr,str)                                           | 返回字符串substr在字符串str中首次出现的位置，作用于POSITION(substr IN str)、 INSTR(str,substr)相同。未找到，返回0 |
| ELT(m,s1,s2,  …,sn)                                          | 返回指定位置的字符串，如果m=1，则返回s1，如果m=2，则返回s2，如 果m=n，则返回sn |
| FIELD(s,s1,s2, …,sn)                                         | 返回字符串s在字符串列表中第一次出现的位置                    |
| FIND_IN_SET(s1,s2)                                           | 返回字符串s1在字符串s2中出现的位置。其中，字符串s2是一个以逗号分 隔的字符串 |
| REVERSE(s)                                                   | 返回s反转后的字符串                                          |
| **NULLIF(value1,value2)**                                    | **比较两个字符串，如果value1与value2相等，则返回NULL，否则返回 value1** |



###### 4-日期和时间函数

+ 获取日期、时间

| 函数                                                         | 用法                                                      |
| ------------------------------------------------------------ | --------------------------------------------------------- |
| **CURDATE()**， CURRENT_DATE()                               | 返回当前日期，只包含**年、 月、日**  【2022-09-08】       |
| **CURTIME()**，  CURRENT_TIME()                              | 返回当前时间，只包含**时、 分、秒**  【13:01:20】         |
| **NOW()** /  SYSDATE() / CURRENT_TIMESTAMP() / LOCALTIME() / LOCALTIMESTAMP() | **返回当前系统日期和时间**  【now():2022-09-08 13:01:20】 |
| UTC_DATE()                                                   | 返回UTC (世界标准时间) 日期                               |
| UTC_TIME()                                                   | 返回UTC (世界标准时间) 时间                               |

+++

+ 日期与时间戳的转换

| 函数                     | 用法                                                         |
| ------------------------ | ------------------------------------------------------------ |
| UNIX_TIMESTAMP()         | 以UNIX时间戳的形式返回当前时间。  SELECT  UNIX_TIMESTAMP() - >1634348884 |
| UNIX_TIMESTAMP(date)     | **将时间date以UNIX时间戳的形式返回。**                       |
| FROM_UNIXTIME(timestamp) | ***将UNIX时间戳的时间转换为普通格式的时间***                 |

+++

+ 获取月份、星期、星期数、天数 ...

| 函数                                      | 用法                                             |
| ----------------------------------------- | ------------------------------------------------ |
| YEAR(date) / MONTH(date) / DAY(date)      | **返回具体的日期值  【年/月/日】**               |
| HOUR(time) / MINUTE(time) /  SECOND(time) | 返回具体的时间值   【时/分/秒】                  |
| MONTHNAME(date)                           | 返回月份： January， ...                         |
| DAYNAME(date)                             | 返回星期几： MONDAY， TUESDAY.....SUNDAY         |
| WEEKDAY(date)                             | 返回周几，注意，周1是0，周2是1，。。。周日是6    |
| QUARTER(date)                             | 返回日期对应的季度，范围为1 ~ 4                  |
| WEEK(date)，  WEEKOFYEAR(date)            | 返回一年中的第几周                               |
| DAYOFYEAR(date)                           | 返回日期是一年中的第几天                         |
| DAYOFMONTH(date)                          | 返回日期位于所在月份的第几天                     |
| DAYOFWEEK(date)                           | 返回周几，注意：周日是1，周一是2，。。。周六是 7 |

+++

+ 日期操作函数

| 函数                    | 用法                                        |
| ----------------------- | ------------------------------------------- |
| EXTRACT(type FROM date) | 返回指定日期中特定的部分， type指定返回的值 |

```mysql
SELECT
    extract(minute from now()),
    extract(week FROM NOW()),
    extract(quarter from now()),
    extract(minute_second from now())
FROM  DUAL ;

```



![image-20220908131002143](http://ybll.vip/md-imgs/202209081310267.png)

![image-20220908131014945](http://ybll.vip/md-imgs/202209081310046.png)

+++

+ 时间 和 秒钟 转换的函数

| 函数                 | 用法                                                         |
| -------------------- | ------------------------------------------------------------ |
| TIME_TO_SEC(time)    | 将 time 转化为秒并返回结果值。转化的公式为：  小时*3600+分钟 *60+秒 |
| SEC_TO_TIME(seconds) | 将 seconds 描述转化为包含小时、分钟和秒的时间                |

+++



###### 5-计算日期和时间的函数

| 函数                                                         | 用法                                                |
| ------------------------------------------------------------ | --------------------------------------------------- |
| **DATE_ADD(datetime, INTERVAL  expr type)，  ADDDATE(date,INTERVAL  expr type)** | **返回与给定日期时间相差INTERVAL时 间段的日期时间** |
| **DATE_SUB(date,INTERVAL  expr type)，  SUBDATE(date,INTERVAL expr type)** | **返回与date相差INTERVAL时间间隔的 日期**           |

```mysql
select date_add(now(), interval 1 day) as col from dual;
```

![image-20220908132503381](http://ybll.vip/md-imgs/202209081325524.png)

+++

| 函数                         | 用法                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| ADDTIME(time1,time2)         | 返回time1加上time2的时间。当time2为一个数字时，代表的是 秒  ，可以为负数 |
| SUBTIME(time1,time2)         | 返回time1减去time2后的时间。当time2为一个数字时，代表的 是 秒 ，可以为负数 |
| **DATEDIFF(date1,date2)**    | **返回date1 - date2的日期间隔天数**                          |
| TIMEDIFF(time1,  time2)      | 返回time1 -  time2的时间间隔                                 |
| FROM_DAYS(N)                 | 返回从0000年1月1日起， N天以后的日期                         |
| TO_DAYS(date)                | 返回日期date距离0000年1月1日的天数                           |
| LAST_DAY(date)               | 返回date所在月份的最后一天的日期                             |
| MAKEDATE(year,n)             | 针对给定年份与所在年份中的天数返回一个日期                   |
| MAKETIME(hour,minute,second) | 将给定的小时、分钟和秒组合成时间并返回                       |
| PERIOD_ADD(time,n)           | 返回time加上n后的时间                                        |

+++



###### 6-日期格式化与解析

| 函数                              | 用法                                       |
| --------------------------------- | ------------------------------------------ |
| **DATE_FORMAT(date,fmt)**         | **按照字符串fmt格式化日期date值**          |
| TIME_FORMAT(time,fmt)             | 按照字符串fmt格式化时间time值              |
| GET_FORMAT(date_type,format_type) | 返回日期字符串的显示格式                   |
| STR_TO_DATE(str,  fmt)            | 按照字符串fmt对str进行解析，解析为一个日期 |

+ 非 `get_format()` 中 fmt参数 常用的格式符

  | **格** **式** **符** | **说明**                                                     | **格式符** | 说明                                                         |
  | -------------------- | ------------------------------------------------------------ | ---------- | ------------------------------------------------------------ |
  | %Y                   | 4位数字表示年份                                              | %y         | 表示两位数字表示年份 【20,21,22】                            |
  | %M                   | 月名表示月份(January,....)                                   | %m         | 两位数字表示月份   ( 01,02,03。。。)                         |
  | %b                   | 缩写的月名(Jan.， Feb.， ....)                               | %c         | 数字表示月份(1,2,3,...)                                      |
  | %D                   | 英文后缀表示月中的天数   ( 1st,2nd,3rd,...)                  | %d         | 两位数字表示月中的天数(01,02...)                             |
  | %e                   | 数字形式表示月中的天数   ( 1,2,3,4,5.....)                   |            |                                                              |
  | %H                   | 两位数字表示小数， 24小时制   ( 01,02..)                     | %h  和%I   | 两位数字表示小时， 12小时制   ( 01,02..)                     |
  | %k                   | 数字形式的小时， 24小时制(1,2,3)                             | %l         | 数字形式表示小时， 12小时制   ( 1,2,3,4....)                 |
  | %i                   | 两位数字表示分钟(00,01,02)                                   | %S  和%s   | 两位数字表示秒(00,01,02...)                                  |
  | %W                   | 一周中的星期名称(Sunday...)                                  | %a         | 一周中的星期缩写(Sun.，  Mon.,Tues.， ..)                    |
  | %w                   | 以数字表示周中的天数  (0=Sunday,1=Monday....)                |            |                                                              |
  | %j                   | 以3位数字表示年中的天数(001,002...)                          | %U         | 以数字表示年中的第几周，         ( 1,2,3。。)其中Sunday为周中第一 天 |
  | %u                   | 以数字表示年中的第几周，         ( 1,2,3。。)其中Monday为周中第一 天 |            |                                                              |
  | %T                   | 24小时制                                                     | %r         | 12小时制                                                     |
  | %p                   | AM或PM                                                       | %%         | 表示%                                                        |

+ `get_format()` 的参数【该函数不常用】

  ![image-20220908133118678](http://ybll.vip/md-imgs/202209081331772.png)

+++



###### 7-流程控制函数

| 函数                                                         | 用法                                                 |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| **IF(value,value1,value2)**                                  | **如果value的值为TRUE，返回value1， 否则返回value2** |
| **IFNULL(value1, value2)**                                   | **如果value1不为NULL，返回value1，否 则返回value2**  |
| **CASE WHEN 条件1 THEN 结果1 WHEN 条件2 THEN 结果2 .... [ELSE resultn] END** | **相当于Java的if...else  if...else...**              |
| **CASE expr WHEN 常量值1 THEN 值1 WHEN 常量值1 THEN 值1 ....  [ELSE 值n] END** | **相当于Java的switch...case...**                     |



###### 8-其他

+ 加密解密

| 函数                        | 用法                                                         |
| --------------------------- | ------------------------------------------------------------ |
| **PASSWORD(str)**           | **返回字符串str的加密版本， 41位长的字符串。 加密结果 不可  逆 ，常用于用户的密码加密** |
| MD5(str)                    | 返回字符串str的md5加密后的值，也是一种加密方式。若参数为 NULL，则会返回NULL |
| SHA(str)                    | 从原明文密码str计算并返回加密后的密码字符串，当参数为 NULL时，返回NULL。  SHA加密算法比MD5更加安全 。 |
| ENCODE(value,password_seed) | 返回使用password_seed作为加密密码加密value                   |
| DECODE(value,password_seed) | 返回使用password_seed作为加密密码解密value                   |

+++

+ mysql 信息函数【帮助数据库开发人员更好地维护】

| 函数                                                    | 用法                                                      |
| ------------------------------------------------------- | --------------------------------------------------------- |
| VERSION()                                               | 返回当前MySQL的版本号                                     |
| CONNECTION_ID()                                         | 返回当前MySQL服务器的连接数                               |
| DATABASE()， SCHEMA()                                   | 返回MySQL命令行当前所在的数据库                           |
| USER()， CURRENT_USER()、SYSTEM_USER()， SESSION_USER() | 返回当前连接MySQL的用户名，返回结果格式为 “主机名@用户名” |
| CHARSET(value)                                          | 返回字符串value自变量的字符集                             |
| COLLATION(value)                                        | 返回字符串value的比较规则                                 |

+++

+ other

| 函数                             | 用法                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| FORMAT(value,n)                  | 返回对数字value进行格式化后的结果数据。 n表示 四舍五入 后保留 到小数点后n位 |
| CONV(value,from,to)              | 将value的值进行不同进制之间的转换                            |
| INET_ATON(ipvalue)               | 将以点分隔的IP地址转化为一个数字                             |
| INET_NTOA(value)                 | 将数字形式的IP地址转化为以点分隔的IP地址                     |
| BENCHMARK(n,expr)                | 将表达式expr重复执行n次。用于测试MySQL处理expr表达式所耗费 的时间 |
| CONVERT(value  USING  char_code) | 将value所使用的字符编码修改为char_code                       |

+++

+++

###### 9-部分函数sql举例

+ 课堂举例

```mysql
/*
LOWER('SQL Course') ：将字符串内容全部变成小写
UPPER('SQL Course') ：将字符串内容全部变成大写
*/
SELECT LOWER('abcDeFgKl'),UPPER('abcDeFgKl');

SELECT LOWER(first_name),UPPER(last_name)
FROM employees;

/*
CONCAT('Hello', 'World') : 字符串拼接
SUBSTR('HelloWorld',1,5) : 截取子串（索引位置从1开始）
	1 ：开始的位置
	5 ： 长度（偏移量）
LENGTH('HelloWorld') : 内容的长度
INSTR('HelloWorld', 'W') : W在当前字符串中首次出现的位置

LPAD(salary,10,'*') : 向右对齐
	如果内容长度不够10用 *补
	
RPAD(salary, 10, '*') ：向左对齐
	如果内容长度不够10用 *补

TRIM('H' FROM 'HelloWorld') : 去除字符串两端指定的字符
REPLACE('abcd','b','m') : 将字符串中所有的b替换成m
*/
SELECT CONCAT(first_name,'-',last_name)
FROM employees;

SELECT SUBSTR('abcdef',2,3);

SELECT first_name,LENGTH(first_name)
FROM employees;

SELECT INSTR('abcdc','c');

SELECT LPAD(salary,10,' '),RPAD(salary,10,' ')
FROM employees;

SELECT TRIM('H' FROM 'HHHHHAHHHBHHHH');

SELECT REPLACE('abcccdba','c','C');


/*
ROUND: 四舍五入
ROUND(45.926, 2)			45.93

TRUNCATE: 截断
TRUNCATE(45.926,0)      		45

MOD: 求余
MOD(1600, 300)		          100

*/
SELECT ROUND(45.926, 2),ROUND(45.926, 1),ROUND(45.926, 0),ROUND(45.926, -1);
SELECT TRUNCATE(45.926, 2),TRUNCATE(45.926, 1),TRUNCATE(45.926, 0),TRUNCATE(45.926, -1);

#结果的正负和被模数的正负有关（第一个参数的正负有关）
SELECT MOD(3,2),MOD(-3,2),MOD(3,-2),MOD(-3,-2);

#日期时间 now()
SELECT NOW();

#版本 version()
SELECT VERSION();
```

+ 流程控制举例：

```mysql
#ifnull(字段名,默认值) :如果字段的内容为null用默认值替换
#需求：查询所有员工的工资（工资+奖金）
SELECT salary+IFNULL(commission_pct,0)*salary 工资
FROM employees;


/*
case表达式

case 字段名
when 值1 then 返回值1
when 值2 then 返回值2
when 值3 then 返回值3
else 返回值n
end

case 
when 表达式1 then 返回值1
when 表达式2 then 返回值2
when 表达式3 then 返回值3
else 返回值n
end

*/
#练习：查询部门号为 10, 20, 30 的员工信息, 若部门号为 10, 则打印其工资的 1.1 倍,
# 20 号部门, 则打印其工资的 1.2 倍, 30 号部门打印其工资的 1.3 倍数
SELECT department_id,salary,CASE department_id
			    WHEN 10 THEN salary*1.1
			    WHEN 20 THEN salary*1.2
			    WHEN 30 THEN salary*1.3
			    ELSE salary
			    END AS "new_salary"
FROM employees
WHERE department_id IN(10,20,30);


#案例：查询所有员工的薪水如果大于10000显示会所嫩模,小于10000显示下海干活.
# 	等于10000再接再厉
SELECT salary,CASE 
	      WHEN salary>10000 THEN "会所嫩模"
	      WHEN salary<10000 THEN "下海干海"
	      ELSE "再接再厉"
	      END AS des
FROM employees;
```



##### 2.2多行函数(组函数-聚合函数)

```mysql
/*
AVG() : 求平均值  
SUM() ：求和
注意：上面的函数只能对数值类型做运算

MAX() ：求最大值
MIN() ：求最小值
 
 
COUNT()：统计结果的数量 


*/
#求所有员工薪水的最大值，最小值，平均值，总和
SELECT MAX(salary),MIN(salary),AVG(salary),SUM(salary)
FROM employees;

#注意：下面的写法不对。select后面出现组函数后将不能再出现其它字段，除非该字段出现在
#group by的后面。
SELECT first_name,AVG(salary)
FROM employees;

/*
COUNT(字段名)：统计查询的结果中该字段内容不为null的有多少条
count(*) : 统计查询的结果有多少条数据
count(数值) ：和count(*)的作用一样。count(数值)效率高一些。
*/
SELECT COUNT(commission_pct),COUNT(*),COUNT(1)
FROM employees;

SELECT COUNT(*)
FROM employees
WHERE commission_pct IS NOT NULL;


#求平均值时是否包含null? 不包含null
SELECT SUM(commission_pct)/107,SUM(commission_pct)/35,AVG(commission_pct)
FROM employees;
```



### 三 DCL 数据库事务

##### 1 说明：

**事务**：将一组逻辑操作单元从一种状态变换到另外一种状态

```java
try{
    开启事务
    AA账户 -count
	System.out.println(1/0);
	BB账户 +count
    事务提交
}catch(Exception e){
    事务回滚;
}
```

##### 2 具体操作

```mysql
开启事务---禁止自动提交 : set autocommit=false;
事务回滚 : rollback;
事务提交(一旦提交将不能再回滚) : commit;
事务结束后要恢复自动提交 : set autocommit=true;
```



### 四 约束

#### 1 有哪些约束？

```mysql
	NOT NULL 非空约束，规定某个字段不能为空
	UNIQUE  唯一约束，规定某个字段在整个表中是唯一的
	PRIMARY KEY  主键(非空且唯一)
	FOREIGN KEY  外键
	CHECK  检查约束（MySQL不支持）
	DEFAULT  默认值
```

#### 2 约束分类 ：列级约束 vs 表级约束

```mysql
#列级约束 ：同时只能约一列
#表级约束 ：同时可以约束多列
```

#### 3 创建表时添加约束

3.1方式一（列级约束） ：

```mysql
CREATE TABLE emp(
id INT PRIMARY KEY,#主键=非空+唯一
NAME VARCHAR(20) NOT NULL,#非空
sid INT UNIQUE,#唯一
age INT DEFAULT 18 #默认值
);
```

3.2方式二（表级约束） ：

```mysql
CREATE TABLE emp2(
id INT,
sid INT,
NAME VARCHAR(20),
#表级约束(注意：not null,default只有列级约束)
#CONSTRAINT 索引名 PRIMARY KEY(字段名1,字段名2,....)
CONSTRAINT emp2_id_sid PRIMARY KEY(id,sid)
);


#唯一
CREATE TABLE emp3(
id INT,
sid INT,
NAME VARCHAR(20),
#表级约束(注意：not null,default只有列级约束)
#当添加唯一约束，主键约束，外键约束时会自动给约束的字段创建索引
#CONSTRAINT 索引名 unique(字段名1,字段名2,....)
CONSTRAINT emp3_id_sid UNIQUE(id,sid)
);
```

#### 4.创建表后添加约束（了解）

```mysql
/*
primary key
添加约束 : alter table 表名 add primary key (字段名)
修改约束 : alter table 表名 modify 字段名 类型 primary key
删除约束 : alter table 表名 drop primary key
*/


/*
unique:
添加约束 : alter table 表名  add unique(字段名)
添加约束 : alter table 表名 add constraint  索引名 unique(字段名)
修改约束 ：alter table 表名 modify 字段名 类型  unique
删除约束 ：alter table 表名 drop index 索引名
*/

/*
not null
添加约束：alter table 表名 modify 字段名 字段类型 not null
删除约束 ：alter table 表名 modify 字段名 字段类型 null

default:
设置默认约束 : alter table tb_name alter 字段名 set default value;
删除默认约束 : alter table tb_name alter 字段名 drop default;

foreign key:
添加约束 ：  ALTER TABLE 表名  ADD  [CONSTRAINT emp_dept_id_fk]   FOREIGN KEY(dept_id) 
			REFERENCES dept(dept_id);
删除约束 ：alter table 表名 drop foreign key  索引名
*/
```

#### 5.外键约束

```mysql
#主表
CREATE TABLE dept(
dept_id INT AUTO_INCREMENT PRIMARY KEY,
dept_name VARCHAR(20)
);

#从表
CREATE TABLE emp(
emp_id INT AUTO_INCREMENT PRIMARY KEY,
last_name VARCHAR(15),
dept_id INT,
#外键约束：CONSTRAINT 索引名 FOREIGN KEY(本表的字段名) REFERENCES 主表表名(主表字段名)
CONSTRAINT emp_dept_id_fk FOREIGN KEY(dept_id) REFERENCES dept(dept_id)
);
```

```mysql
#创建表时先创建主表还是从表？	先创建主表再创建从表
#添加数据时先往主表还是从表添加？主表
#删除数据时先删除主表还是从表？从表
```

级联删除(删除部门时会把该部门所有的员工全部删除)

```mysql
#主表
CREATE TABLE dept2(
dept_id INT AUTO_INCREMENT PRIMARY KEY,
dept_name VARCHAR(20)
);

#从表
CREATE TABLE emp2(
emp_id INT AUTO_INCREMENT PRIMARY KEY,
last_name VARCHAR(15),
dept_id INT,
#ON DELETE CASCADE :级联删除
CONSTRAINT emp2_dept_id_fk FOREIGN KEY(dept_id) REFERENCES dept2(dept_id) ON DELETE CASCADE
);
```

***

***

***

***

***



## 摘要汇总|myself

### mysql安装(linux)

```bash
#检查是否存在自带Mysql
rpm -qa | grep -i mysql; 
rpm -qa | grep -i -E mysql\|mariadb;

#卸载mariadb
sudo rpm -e --nodes mysql.tar.gz/others;
rpm -qa | grep -i -E mysql\|mariadb | xargs -n1 sudo rpm -e --nodeps

#正式安装
tar -xvf mysql-5.7.tar.gz -C dir/ #解压压缩包

#目录列表，且依次顺序安装
mysql-community-common-xx.rpm; libs-5.7.28; libs-compat; client; server.rpm; 
#安装命令如下
rpm -ivh mysql-community-server-xx.rpm;

#删除my.cnf文件中，配置项datadir所指目录下的所有文件
#这是原来mysql数据所存储的目录，需要全部删除，保证卸载干净
/etc/my.cnf; datadir=/var/lib/mysql; rm -rf ./*;

#mysql安装后的初始化操作，创建mysql内部数据库和表
sudo mysqld --initialize --user=mysql
cat /var/log/mysqld.log | grep password  #查看临时密码

# erd5#kfzt<9W

#启动mysql, 并登录
sudo systemctl start mysqld
mysql -uroot -p

set password = password("newpw"); #修改root密码
#修改mysql库下user表中root用户允许任意ip连接
use mysql; select host,user from user;
update mysql.user set host='%' where user='root';
flush privileges;
```

++++

### 知识点小记

#### 1. 基本规则与规范

> + 每条命令以 `;` 或者 `\g` 、`\G` 结尾
>
> + 所有`() '' ""` 都需要成对存在
>
> + 字符串型和日期时间类型的数据可以用 `单引号''` 表示
>
> + 列的别名尽量使用`双引号""`，且不建议省略as
>
> + Mysql在windows环境下大小写不敏感，Linux环境下是大小写敏感的
>
>   ![image-20220907195003299](http://ybll.vip/md-imgs/202209071950373.png)
>
> +++

#### 2. sql语句执行顺序：

**from - on - join - where - group by - `with` - having - select - distinct - order by - limit**

```mysql
6) SELECT [DISTINCT]...,....,...
1) FROM ... JOIN ...
2) ON 多表的连接条件
   JOIN ...
   ON ...
3) WHERE 不包含组函数的过滤条件
   AND/OR 不包含组函数的过滤条件
4) GROUP BY ...,...
5) HAVING 包含组函数的过滤条件
7) ORDER BY ... ASC/DESC
8) LIMIT ...,...
```

![image-20220403112534266](http://ybll.vip/md-imgs/202204031126573.png)

+++

#### 3. 列的别名，表的别名

+ `as "别名"`(as 和 " "  可省略，空格不能省略 )    
+ 表别名：`as`(可省略，不能加 `""`)

+++

#### 4. 空值运算

+ 所有运算符或者列值遇到null值，运算的结果都为null。

+++

#### 5.空值与空字符串

+ 一个空串的长度是 0，而空值的长度是空
+ mysql中，空值是占用空间的

#### 6. 运算符

> 不等于：`<>  !=`    安全等于运算符：`<=>`    `IN `  `NOT IN `
>
> `IS NULL` `ISNULL` `IS NOT NULL `  `BETWEEN AND (等价于 >= v1 and <= v2)`
>
>  `LIKE  :( _匹配一个   %匹配0个或多个)` 
>
> `REGEXP :正则表达式匹配 A regexp B`  `RLIKE  : 正则表达式匹配 Ａ rlike B`

> 运算符优先级
>
> ![image-20220411094021600](http://ybll.vip/md-imgs/202204110940478.png)

+++

#### 7. 排序与分页

> `order by` : 子句排序，asc(升序)  desc(降序) order by(子句在select语句结尾) 
>
> `limit` : LIMITE实现分页，`LIMIT [位置偏移量] 行数`
>
> mysql 8.0 中 可使用 ` limit 3 offset 4 ` 相当于 `limit 4,3`
>
> LIMIT子句必须放在整个select语句的最后

+++

#### 8. SQL Joins

![image-20220411085954860](http://ybll.vip/md-imgs/202204110900152.png)

#### 9. SQL99语法新特性

> 自然连接 `natural join` :可以理解为等值连接，查询两张连接表中**所有相同的字段**，然后进行**等值连接**
>
> USING连接 ：指定数据表里的同名字段进行等值连接，但是只能配合JOIN一起使用
>
> ![image-20220411100223729](http://ybll.vip/md-imgs/202204111002985.png)

+++

#### 10. 子查询

> + 单行子查询 : `=` `>` `>=` `<` `<=` `<>`
>
> + 多行子查询 : `IN` `ANY` `ALL` `SOME(是any别名，作用相同，一般用any)`
>
> ***
>
> + 相关子查询 : 子查询条件依赖于外部查询，通常情况下是子查询中的表用到了外部的表并进行了条件关联，因此每执行一次外部查询，子查询都要重新计算一次，这样的子查询就是`关联子查询`
>
> ```mysql
> #查询员工中工资大于本部门平均工资的员工的last_name,salary，和其department_id
> 
> select last_name,salary,department_id
> from employees outer
> where salary > (
> 	select avg(salary)
> 	from employees
> 	where department_id = outer.department_id
> );
> ```
>
> `EXISTS` : 子查询不满足条件，返回false,继续子查询；满足条件，返回true,跳出子查询。
>
> `NOT EXISTS`
>
> +++

#### 11. DDL

```mysql
#数据库命名：A-Z,a-z,0-9,_ 共63字符，长度不超过30个字符
create database if not exists 数据库名 character set 字符集;

show databases; use 数据库名;

select database()#查看当前使用的数据库，全局函数

show tables from 数据库名 #查看指定数据库中的表

show create database 数据库名; #查看数据库的创建信息(\G \g结尾也可以)

alter database 数据库名 character set 字符集; #修改数据库字符集

drop database 数据库名;  drop database if exists 数据库名;


#################### 表DDL
create table if not exists 表名(
    列名1 int,
    列名2 varchar(20)
    primary key(列名1) #主键
);
create table emp as (子查询...);
auto_increment #自增
primary key(..)#主键


desc employees;  
show create table employees #查看表信息 

#追加列：alter table 表名 add 字段1 字段类型 [first|after 字段2]
alter table emp add job_id varchar(15);

#修改列：alter table 表名 modify 字段1 新类型 [first|after 字段2]
alter table emp modify last_name varchar(30) default '123';

#重命名一个列：alter table 表名 change 列名 新列名 新类型;
alter table emp change last_name name varchar(10);

#删除一个列：alter table 表名 drop 字段名
alter table emp drop job_id;

#重命名
rename table emp to myemp;
alter table myemp rename to emp;

#删除表(drop删除不能回滚): drop table if exists 数据表1,数据表2...
drop table if exists emp;

#清空表(truncate不能回滚):truncate table 表名
truncate table emp;

#drop、truncate 操作不能回滚，delete语句操作表可以回滚
```

#### 12. 数据类型

```mysql
#部分数据类型
int float double 
decimal 
date datetime timestamp 
varchar text 
blob binary 
json 
set 
enum

#常见数据类型的属性
null (可包含null) not null(不可包含null) default(默认值)
primary key(主键)  auto_increment(自动递增，适用于整数类型)
unsigned(无符号)  character set name(指定字符集)
```

> + 在定义数据类型时，
>   + 如果是 **整数** ，就用 `INT` ；
>   + 如果是 **小数** ，一定用定点数类型 `DECIMAL(M,D)` ； 
>   + 如果是**日期与时间**，就用 `DATETIME` 。 
> + 好处:首先确保你的系统不会因为数据类型定义出错。
> + 不过，凡事都是有两面的，可靠性 好，并不意味着高效。比如，TEXT 虽然使用方便，但是效率不如 CHAR(M) 和 VARCHAR(M)。

![image-20220414145807192](http://ybll.vip/md-imgs/202204141458580.png)

+++

#### 13. 主键相同则更新

```mysql
insert into 'table' values(19,'电子产品')
on duplicate key update;
```

#### 14. 自我总结：

> + select 查询时，可以先判断内连接还是外连接，然后写查询字段，中途可以执行子查询判断是否正确
>
> + 多表查询（自连接...等等）效率比子查询效率更高
> 
> ***
> 
> + ![image-20220411100516036](http://ybll.vip/md-imgs/202204111005974.png)

+++

## 代码题

```sql
//效率更高
select gpa
from user_profile
where university='复旦大学'
order by gpa desc limit 1  

select max(gpa) gpa from user_profile where university='复旦大学';

//牛客sql在线编程17题，
//现在运营想要看一下男性用户有多少人以及他们的平均gpa是多少。
//round(avg(gpa),1) 保留一位小数，四舍五入
select 
	count(gender) as male_num,
	round(avg(gpa),1) as avg_gpa 
from user_profile
where gender='male';

//牛客sql在线编程18题，
//计算出每个学校每种性别的用户数、30天内平均活跃天数和平均发帖数量。 
select 
    gender,
    university,
    count(device_id) as user_num,
    round(avg(active_days_within_30),1) as avg_active_day,
    round(avg(question_cnt),1) as avg_question_cnt
from user_profile
group by university,gender;

//牛客sql在线编程19题
//取出平均发贴数低于5的学校或平均回帖数小于20的学校
//group by后面不能加聚合函数
select
    university,
    round(avg(question_cnt),3) as avg_question_cnt,
    round(avg(answer_cnt),3) as avg_answer_cnt
from user_profile
group by university
having avg_question_cnt<5 or avg_answer_cnt<20;

//牛客sql在线编程20题
//查看不同大学的用户平均发帖情况，并期望结果按照平均发帖情况进行升序排列 
// round(avg(question_cnt),3)  为什么错？
select 
    university,
    avg(question_cnt) avg_question_cnt
from user_profile
group by university
order by avg_question_cnt;

//牛客sql在线编程21题
select q.device_id,q.question_id,q.result
from 
    question_practice_detail q
where
q.device_id in
    (select
        device_id
    from 
        user_profile u
    where
        u.university='浙江大学')
```



# HIVE SQL

1. Hive SQL执行顺序

![image-20220706095205192](http://ybll.vip/md-imgs/202207060952340.png)

2. 为什么 `having` 后面可以使用 `列的别名` ? 

   > 

# 连续登录问题

```sql
-- SQL2 连续登录至少三天的用户   标签：分组、开窗、时间函数、连续登录问题
-- 查询用户活跃记录表中连续登录大于等于三天的用户，如果存在两次大于等于三天的连续登录，取最大值。
DROP TABLE IF EXISTS `user_active`;
CREATE TABLE user_active
(
    `user_id`     string COMMENT '用户id',
    `active_date` string COMMENT '登录日期时间',
    `test_case`   int
) COMMENT '用户活跃表'
    ROW FORMAT DELIMITED
        FIELDS TERMINATED BY '\t'
        NULL DEFINED AS ''
    LOCATION '/user/hive/warehouse/user_active';

INSERT INTO user_active
VALUES
('1','2022-04-25',NULL),('1','2022-04-26',NULL),('1','2022-04-27',NULL),
('1','2022-04-28',NULL),('1','2022-04-29',NULL),('2','2022-04-12',NULL),
('2','2022-04-13',NULL),('3','2022-04-05',NULL),('3','2022-04-06',NULL),
('3','2022-04-08',NULL),('4','2022-04-04',NULL),('4','2022-04-05',NULL),
('4','2022-04-06',NULL),('4','2022-04-06',NULL),('4','2022-04-06',NULL),
('5','2022-04-12',NULL),('5','2022-04-08',NULL),('5','2022-04-26',NULL),
('5','2022-04-26',NULL),('5','2022-04-27',NULL),('5','2022-04-28',NULL);

select * from user_active;



select
    t3.user_id,
    max(t3.num_days) count
from (
    select
        t2.user_id,
        t2.flag_date,
        count(t2.flag_date) num_days
    from (
        select
            t1.user_id,
            date_sub(t1.active_date, rank() over (partition by t1.user_id order by t1.active_date)) flag_date
        from (
            select distinct
                user_id,
                active_date
            from user_active  -- 顾虑同一天重复登录的
        ) t1
    ) t2
    group by user_id,flag_date
) t3
-- where num_days >=3 -- 可行
group by user_id
-- having count >=3; --可行
-- having t3.num_days >=3; --不可行
```

```sql
--求间断的连续登录天数
--思路：同一个用户，存在 非连续登录的可能(间断时间大一1天)
--1. 求出 当前登录日期 与 上次登录日期 的 日期差
--2. sum()操作获取分组标记flag： 若 日期差 小于2，则表示属于连续登录，取值0；否则表示已经间断了，取值1；
    --每出现一次间断登录，则sum()结果+1，对该结果开窗
--3，由 flag(间断次数) 分组统计，diff(max_date,min_date)求出当前区间，连续登录的天数
select
    user_id,
    max(lx) max_day_count
from (
    select 
        user_id,
        flag,
        datediff(max(login_date), min(login_date))+1 lx
    from (
        select 
            user_id,
            login_date,
            --以此作为分组依据，间断1次的求连续天数，间断两次的求连续天数，然后取最大的连续天数
            sum(if(flag > 2, 1, 0)) over(partition by user_id order by login_date) flag --到当前日期为止的间断次数
        from (
            select 
                user_id,
                login_date,
                datediff(login_date, lag(login_date, 1, '1970-01-01') over(partition by user_id order by login_date)) flag
            from (
                select distinct 
                    user_id,
                    date_format(login_datetime, 'yyyy-MM-dd') login_date
                from login_events
            ) t1
        ) t2
    ) t3
    group by user_id, flag
) t4
group by user_id;
```





# MYSQL字符集问题



MySQL 乱码的根源是的 MySQL 字符集设置不当的问题，本文汇总了有关查看 MySQL 字符集的命令。包括查看 MySQL 数据库服务器字符集、查看 MySQL 数据库字符集，以及数据表和字段的字符集、当前安装的 MySQL 所支持的字符集等。

一、查看 MySQL 数据库服务器和数据库字符集。

mysql> show variables like '%char%';
+--------------------------+-------------------------------------+------
| Variable_name | Value |......
+--------------------------+-------------------------------------+------
| character_set_client | utf8 |...... -- 客户端字符集
| character_set_connection | utf8 |......
| character_set_database | utf8 |...... -- 数据库字符集
| character_set_filesystem | binary |......
| character_set_results | utf8 |......
| character_set_server | utf8 |...... -- 服务器字符集
| character_set_system | utf8 |......
| character_sets_dir | D:\MySQL Server 5.0\share\charsets\ |......
+--------------------------+-------------------------------------+------

二、查看 MySQL 数据表（table） 的字符集。

mysql> show table status from sqlstudy_db like '%countries%';
+-----------+--------+---------+------------+------+-----------------+------
| Name | Engine | Version | Row_format | Rows | Collation |......
+-----------+--------+---------+------------+------+-----------------+------
| countries | InnoDB | 10 | Compact | 11 | utf8_general_ci |......
+-----------+--------+---------+------------+------+-----------------+------

三、查看 MySQL 数据列（column）的字符集。

mysql> show full columns from countries;
+----------------------+-------------+-----------------+--------
| Field | Type | Collation | .......
+----------------------+-------------+-----------------+--------
| countries_id | int(11) | NULL | .......
| countries_name | varchar(64) | utf8_general_ci | .......
| countries_iso_code_2 | char(2) | utf8_general_ci | .......
| countries_iso_code_3 | char(3) | utf8_general_ci | .......
| address_format_id | int(11) | NULL | .......
+----------------------+-------------+-----------------+--------

四、查看当前安装的 MySQL 所支持的字符集。

mysql> show charset;
mysql> show char set;
+----------+-----------------------------+---------------------+--------+
| Charset | Description | Default collation | Maxlen |
+----------+-----------------------------+---------------------+--------+
| big5 | Big5 Traditional Chinese | big5_chinese_ci | 2 |
| dec8 | DEC West European | dec8_swedish_ci | 1 |
| cp850 | DOS West European | cp850_general_ci | 1 |
| hp8 | HP West European | hp8_english_ci | 1 |
| koi8r | KOI8-R Relcom Russian | koi8r_general_ci | 1 |
| latin1 | cp1252 West European | latin1_swedish_ci | 1 |
| latin2 | ISO 8859-2 Central European | latin2_general_ci | 1 |
| swe7 | 7bit Swedish | swe7_swedish_ci | 1 |
| ascii | US ASCII | ascii_general_ci | 1 |
| ujis | EUC-JP Japanese | ujis_japanese_ci | 3 |
| sjis | Shift-JIS Japanese | sjis_japanese_ci | 2 |
| hebrew | ISO 8859-8 Hebrew | hebrew_general_ci | 1 |
| tis620 | TIS620 Thai | tis620_thai_ci | 1 |
| euckr | EUC-KR Korean | euckr_korean_ci | 2 |
| koi8u | KOI8-U Ukrainian | koi8u_general_ci | 1 |
| gb2312 | GB2312 Simplified Chinese | gb2312_chinese_ci | 2 |
| greek | ISO 8859-7 Greek | greek_general_ci | 1 |
| cp1250 | Windows Central European | cp1250_general_ci | 1 |
| gbk | GBK Simplified Chinese | gbk_chinese_ci | 2 |
| latin5 | ISO 8859-9 Turkish | latin5_turkish_ci | 1 |
| armscii8 | ARMSCII-8 Armenian | armscii8_general_ci | 1 |
| utf8 | UTF-8 Unicode | utf8_general_ci | 3 |
| ucs2 | UCS-2 Unicode | ucs2_general_ci | 2 |
| cp866 | DOS Russian | cp866_general_ci | 1 |
| keybcs2 | DOS Kamenicky Czech-Slovak | keybcs2_general_ci | 1 |
| macce | Mac Central European | macce_general_ci | 1 |
| macroman | Mac West European | macroman_general_ci | 1 |
| cp852 | DOS Central European | cp852_general_ci | 1 |
| latin7 | ISO 8859-13 Baltic | latin7_general_ci | 1 |
| cp1251 | Windows Cyrillic | cp1251_general_ci | 1 |
| cp1256 | Windows Arabic | cp1256_general_ci | 1 |
| cp1257 | Windows Baltic | cp1257_general_ci | 1 |
| binary | Binary pseudo charset | binary | 1 |
| geostd8 | GEOSTD8 Georgian | geostd8_general_ci | 1 |
| cp932 | SJIS for Windows Japanese | cp932_japanese_ci | 2 |
| eucjpms | UJIS for Windows Japanese | eucjpms_japanese_ci | 3 |
+----------+-----------------------------+---------------------+--------+

以上查看 MySQL 字符集命令，适用于 Windows & Linux。

四.修改表和字段的字符集
//修改数据库
mysql> alter database name character set utf8;
//修改表
alter table 表名 convert to character set gbk;
//修改字段
alter table 表名 modify column '字段名' varchar(30) character set gbk not null;
//添加表字段
alter table 表名 add column '字段名' varchar (20) character set gbk;
注:执行命令过程中字段名不加引号

http://www.cnblogs.com/dongweihang/p/4126004.html

**Liunx下修改MySQL字符集：**
1.查找MySQL的cnf文件的位置
find / -iname '*.cnf' -print

/usr/share/mysql/my-innodb-heavy-4G.cnf
/usr/share/mysql/my-large.cnf
/usr/share/mysql/my-small.cnf
/usr/share/mysql/my-medium.cnf
/usr/share/mysql/my-huge.cnf
/usr/share/texmf/web2c/texmf.cnf
/usr/share/texmf/web2c/mktex.cnf
/usr/share/texmf/web2c/fmtutil.cnf
/usr/share/texmf/tex/xmltex/xmltexfmtutil.cnf
/usr/share/texmf/tex/jadetex/jadefmtutil.cnf
/usr/share/doc/MySQL-server-community-5.1.22/my-innodb-heavy-4G.cnf
/usr/share/doc/MySQL-server-community-5.1.22/my-large.cnf
/usr/share/doc/MySQL-server-community-5.1.22/my-small.cnf
/usr/share/doc/MySQL-server-community-5.1.22/my-medium.cnf
/usr/share/doc/MySQL-server-community-5.1.22/my-huge.cnf

\2. 拷贝 small.cnf、my-medium.cnf、my-huge.cnf、my-innodb-heavy-4G.cnf其中的一个到/etc下，命名为my.cnf
cp /usr/share/mysql/my-medium.cnf /etc/my.cnf

\3. 修改my.cnf
vi /etc/my.cnf
在[client]下添加
default-character-set=utf8
在[mysqld]下添加
default-character-set=utf8

4.重新启动MySQL
[root@bogon ~]# /etc/rc.d/init.d/mysql restart
Shutting down MySQL                           [ 确定 ]
Starting MySQL.                              [ 确定 ]
[root@bogon ~]# mysql -u root -p
Enter password:
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.1.22-rc-community-log MySQL Community Edition (GPL)
Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

5.查看字符集设置
mysql> show variables like 'collation_%';
+----------------------+-----------------+
| Variable_name      | Value        |
+----------------------+-----------------+
| collation_connection | utf8_general_ci |
| collation_database  | utf8_general_ci |
| collation_server    | utf8_general_ci |
+----------------------+-----------------+
3 rows in set (0.02 sec)
mysql> show variables like 'character_set_%';
+--------------------------+----------------------------+
| Variable_name        | Value               |
+--------------------------+----------------------------+
| character_set_client    | utf8                |
| character_set_connection | utf8                |
| character_set_database  | utf8                |
| character_set_filesystem | binary              |
| character_set_results   | utf8                |
| character_set_server    | utf8                |
| character_set_system    | utf8                |
| character_sets_dir     | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.02 sec)
mysql>


**其他的一些设置方法：**

**修改数据库的字符集**
  mysql>use mydb
  mysql>alter database mydb character set utf-8;
**创建数据库指定数据库的字符集**
  mysql>create database mydb character set utf-8;

**通过配置文件修改:**
修改/var/lib/mysql/mydb/db.opt
default-character-set=latin1
default-collation=latin1_swedish_ci
为
default-character-set=utf8
default-collation=utf8_general_ci
重起MySQL:
[root@bogon ~]# /etc/rc.d/init.d/mysql restart

**通过MySQL命令行修改:**
mysql> set character_set_client=utf8;
Query OK, 0 rows affected (0.00 sec)
mysql> set character_set_connection=utf8;
Query OK, 0 rows affected (0.00 sec)
mysql> set character_set_database=utf8;
Query OK, 0 rows affected (0.00 sec)
mysql> set character_set_results=utf8;
Query OK, 0 rows affected (0.00 sec)
mysql> set character_set_server=utf8;
Query OK, 0 rows affected (0.00 sec)
mysql> set character_set_system=utf8;
Query OK, 0 rows affected (0.01 sec)
mysql> set collation_connection=utf8;
Query OK, 0 rows affected (0.01 sec)
mysql> set collation_database=utf8;
Query OK, 0 rows affected (0.01 sec)
mysql> set collation_server=utf8;
Query OK, 0 rows affected (0.01 sec)
**查看:
**mysql> show variables like 'character_set_%';
+--------------------------+----------------------------+
| Variable_name        | Value               |
+--------------------------+----------------------------+
| character_set_client    | utf8                |
| character_set_connection | utf8                |
| character_set_database  | utf8                |
| character_set_filesystem | binary              |
| character_set_results   | utf8                |
| character_set_server    | utf8                |
| character_set_system    | utf8                |
| character_sets_dir     | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.03 sec)
mysql> show variables like 'collation_%';
+----------------------+-----------------+
| Variable_name      | Value        |
+----------------------+-----------------+
| collation_connection | utf8_general_ci |
| collation_database  | utf8_general_ci |
| collation_server    | utf8_general_ci |
+----------------------+-----------------+
3 rows in set (0.04 sec)



\-------------------------------------------------------------------------
【知识性文章转载】
MYSQL 字符集问题


MySQL的字符集支持(Character Set Support)有两个方面：
    字符集(Character set)和排序方式(Collation)。
对于字符集的支持细化到四个层次:
    服务器(server)，数据库(database)，数据表(table)和连接(connection)。
1.MySQL默认字符集
MySQL对于字符集的指定可以细化到一个数据库，一张表，一列，应该用什么字符集。
但是，传统的程序在创建数据库和数据表时并没有使用那么复杂的配置，它们用的是默认的配置，那么，默认的配置从何而来呢？   (1)编译MySQL 时，指定了一个默认的字符集，这个字符集是 latin1；
   (2)安装MySQL 时，可以在配置文件 (my.ini) 中指定一个默认的的字符集，如果没指定，这个值继承自编译时指定的；
   (3)启动mysqld 时，可以在命令行参数中指定一个默认的的字符集，如果没指定，这个值继承自配置文件中的配置,此时 character_set_server 被设定为这个默认的字符集；
   (4)当创建一个新的数据库时，除非明确指定，这个数据库的字符集被缺省设定为character_set_server；
   (5)当选定了一个数据库时，character_set_database 被设定为这个数据库默认的字符集；
   (6)在这个数据库里创建一张表时，表默认的字符集被设定为 character_set_database，也就是这个数据库默认的字符集；
   (7)当在表内设置一栏时，除非明确指定，否则此栏缺省的字符集就是表默认的字符集；
简单的总结一下，如果什么地方都不修改，那么所有的数据库的所有表的所有栏位的都用
latin1 存储，不过我们如果安装 MySQL，一般都会选择多语言支持，也就是说，安装程序会自动在配置文件中把
default_character_set 设置为 UTF-8，这保证了缺省情况下，所有的数据库的所有表的所有栏位的都用 UTF-8 存储。
2.查看默认字符集(默认情况下，mysql的字符集是latin1(ISO_8859_1)
通常，查看系统的字符集和排序方式的设定可以通过下面的两条命令：
    mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+---------------------------------+
| Variable_name        | Value                  |
+--------------------------+---------------------------------+
| character_set_client    | latin1                  |
| character_set_connection | latin1                  |
| character_set_database  | latin1                  |
| character_set_filesystem | binary              |
| character_set_results   | latin1                  |
| character_set_server    | latin1                  |
| character_set_system   | utf8                   |
| character_sets_dir     | D:"mysql-5.0.37"share"charsets" |
+--------------------------+---------------------------------+
mysql> SHOW VARIABLES LIKE 'collation_%';
+----------------------+-----------------+
| Variable_name      | Value        |
+----------------------+-----------------+
| collation_connection | utf8_general_ci |
| collation_database  | utf8_general_ci |
| collation_server    | utf8_general_ci |
+----------------------+-----------------+
3.修改默认字符集
(1) 最简单的修改方法，就是修改mysql的my.ini文件中的字符集键值，
如   default-character-set = utf8
    character_set_server = utf8
  修改完后，重启mysql的服务，service mysql restart
  使用 mysql> SHOW VARIABLES LIKE 'character%';查看，发现数据库编码均已改成utf8
+--------------------------+---------------------------------+
| Variable_name        | Value                  |
+--------------------------+---------------------------------+
| character_set_client    | utf8                   |
| character_set_connection | utf8                   |
| character_set_database  | utf8                   |
| character_set_filesystem | binary                  |
| character_set_results   | utf8                   |
| character_set_server    | utf8                   |
| character_set_system    | utf8                   |
| character_sets_dir     | D:"mysql-5.0.37"share"charsets" |
+--------------------------+---------------------------------+
  (2) 还有一种修改字符集的方法，就是使用mysql的命令
  mysql> SET character_set_client = utf8 ;

**MySQL中涉及的几个字符集**

character-set-server/default-character-set：服务器字符集，默认情况下所采用的。
character-set-database：数据库字符集。
character-set-table：数据库表字符集。
优先级依次增加。所以一般情况下只需要设置character-set-server，而在创建数据库和表时不特别指定字符集，这样统一采用character-set-server字符集。
character-set-client：客户端的字符集。客户端默认字符集。当客户端向服务器发送请求时，请求以该字符集进行编码。
character-set-results：结果字符集。服务器向客户端返回结果或者信息时，结果以该字符集进行编码。
在客户端，如果没有定义character-set-results，则采用character-set-client字符集作为默认的字符集。所以只需要设置character-set-client字符集。

要处理中文，则可以将character-set-server和character-set-client均设置为GB2312，如果要同时处理多国语言，则设置为UTF8。

**关于MySQL的中文问题**

解决乱码的方法是，在执行SQL语句之前，将MySQL以下三个系统参数设置为与服务器字符集character-set-server相同的字符集。
character_set_client：客户端的字符集。
character_set_results：结果字符集。
character_set_connection：连接字符集。
设置这三个系统参数通过向MySQL发送语句：set names gb2312

**关于GBK、GB2312、UTF8**

UTF- 8：Unicode Transformation Format-8bit，允许含BOM，但通常不含BOM。是用以解决国际上字符的一种多字节编码，它对英文使用8位（即一个字节），中文使用24为（三个字节）来编码。UTF-8包含全世界所有国家需要用到的字符，是国际编码，通用性强。UTF-8编码的文字可以在各国支持UTF8字符集的浏览器上显示。如，如果是UTF8编码，则在外国人的英文IE上也能显示中文，他们无需下载IE的中文语言支持包。

GBK是国家标准GB2312基础上扩容后兼容GB2312的标准。GBK的文字编码是用双字节来表示的，即不论中、英文字符均使用双字节来表示，为了区分中文，将其最高位都设定成1。GBK包含全部中文字符，是国家编码，通用性比UTF8差，不过UTF8占用的数据库比GBD大。

GBK、GB2312等与UTF8之间都必须通过Unicode编码才能相互转换：
GBK、GB2312－－Unicode－－UTF8
UTF8－－Unicode－－GBK、GB2312

对于一个网站、论坛来说，如果英文字符较多，则建议使用UTF－8节省空间。不过现在很多论坛的插件一般只支持GBK。

GB2312是GBK的子集，GBK是GB18030的子集
GBK是包括中日韩字符的大字符集合
如果是中文的网站 推荐GB2312 GBK有时还是有点问题
为了避免所有乱码问题，应该采用UTF-8，将来要支持国际化也非常方便
UTF-8可以看作是大字符集，它包含了大部分文字的编码。
使用UTF-8的一个好处是其他地区的用户（如香港台湾）无需安装简体中文支持就能正常观看你的文字而不会出现乱码。

gb2312是简体中文的码
gbk支持简体中文及繁体中文
big5支持繁体中文
utf-8支持几乎所有字符

首先分析乱码的情况
1.写入数据库时作为乱码写入
2.查询结果以乱码返回
究竟在发生乱码时是哪一种情况呢？
我们先在mysql 命令行下输入
show variables like '%char%';
查看mysql 字符集设置情况:

mysql> show variables like '%char%';
+--------------------------+----------------------------------------+
| Variable_name      | Value                 |
+--------------------------+----------------------------------------+
| character_set_client   | gbk                  | 
| character_set_connection | gbk                  | 
| character_set_database  | gbk                  | 
| character_set_filesystem | binary                 | 
| character_set_results  | gbk                  | 
| character_set_server   | gbk                  | 
| character_set_system   | utf8                  | 
| character_sets_dir    | /usr/local/mysql/share/mysql/charsets/ | 
+--------------------------+----------------------------------------+

在查询结果中可以看到mysql 数据库系统中客户端、数据库连接、数据库、文件系统、查询
结果、服务器、系统的字符集设置
在这里，文件系统字符集是固定的，系统、服务器的字符集在安装时确定，与乱码问题无关
乱码的问题与客户端、数据库连接、数据库、查询结果的字符集设置有关
*注：客户端是看访问mysql 数据库的方式，通过命令行访问，命令行窗口就是客户端，通
过JDBC 等连接访问，程序就是客户端
我们在向mysql 写入中文数据时，在客户端、数据库连接、写入数据库时分别要进行编码转
换
在执行查询时，在返回结果、数据库连接、客户端分别进行编码转换
现在我们应该清楚，乱码发生在数据库、客户端、查询结果以及数据库连接这其中一个或多
个环节
接下来我们来解决这个问题
在登录数据库时，我们用mysql --default-character-set=字符集-u root -p 进行连接，这时我们
再用show variables like '%char%';命令查看字符集设置情况，可以发现客户端、数据库连接、
查询结果的字符集已经设置成登录时选择的字符集了
如果是已经登录了，可以使用set names 字符集;命令来实现上述效果，等同于下面的命令：
set character_set_client = 字符集
set character_set_connection = 字符集
set character_set_results = 字符集

如果碰到上述命令无效时，也可采用一种最简单最彻底的方法：

一、Windows

1、中止MySQL服务
2、在MySQL的安装目录下找到my.ini，如果没有就把my-medium.ini复制为一个my.ini即可
3、打开my.ini以后，在[client]和[mysqld]下面均加上default-character-set=utf8，保存并关闭
4、启动MySQL服务

要彻底解决编码问题，必须使

| character_set_client   | gbk                  | 
| character_set_connection | gbk                  | 
| character_set_database  | gbk                  | 
| character_set_results  | gbk                  | 
| character_set_server   | gbk                  | 
| character_set_system   | utf8   

这些编码相一致，都统一。


如果是通过JDBC 连接数据库，可以这样写URL：
URL=jdbc:mysql://localhost:3306/abs?useUnicode=true&characterEncoding=字符集
JSP 页面等终端也要设置相应的字符集
数据库的字符集可以修改mysql 的启动配置来指定字符集，也可以在create database 时加上
default character set 字符集来强制设置database 的字符集
通过这样的设置，整个数据写入读出流程中都统一了字符集，就不会出现乱码了
为什么从命令行直接写入中文不设置也不会出现乱码？
可以明确的是从命令行下，客户端、数据库连接、查询结果的字符集设置没有变化
输入的中文经过一系列转码又转回初始的字符集，我们查看到的当然不是乱码
但这并不代表中文在数据库里被正确作为中文字符存储
举例来说，现在有一个utf8 编码数据库，客户端连接使用GBK 编码，connection 使用默认
的ISO8859-1（也就是mysql 中的latin1），我们在客户端发送“中文”这个字符串，客户端
将发送一串GBK 格式的二进制码给connection 层，connection 层以ISO8859-1 格式将这段
二进制码发送给数据库，数据库将这段编码以utf8 格式存储下来，我们将这个字段以utf8
格式读取出来，肯定是得到乱码，也就是说中文数据在写入数据库时是以乱码形式存储的，
在同一个客户端进行查询操作时，做了一套和写入时相反的操作，错误的utf8 格式二进制
码又被转换成正确的GBK 码并正确显示出来。

