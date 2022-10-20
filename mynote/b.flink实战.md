## 4 Flink

### Flink SQL Join

> 普通 join
>
> + 默认情况下，会一直存在状态中，一般设置 TTL
>
>   ```java
>   tEnv.getConfig().setIdleStateRetention(Duration.ofSeconds(10));
>   ```
>
> + left join
>
>   + 左表先来，会有输出，右表数据用 null补齐；右表接着来了，先删除，再新增【-D | +I】
>   + 右表先来,左表无匹配行，则无输出
>
> +++
>
> Kafka Sink
>
> + 普通的 Kafka连接器无法写入 更新及删除，因此不能和 left join 配合
>
> + upsert Kafka : K-V的形式写入 Kafka，【需要指定主键，主键的值即为 key】
>
>   ```java
>           TableResult tableResult = tEnv.executeSql("create table ab(" +
>                   " id int, " +
>                   " name string, " +
>                   " age int," +
>                   " primary key(id) not enforced " +
>                   ")with(" +
>                   "  'connector' = 'upsert-kafka', " +
>                   "  'topic' = 'ab4', " +
>                   "  'properties.bootstrap.servers' = 'hadoop162:9092', " +
>                   "  'key.format' = 'json', " +
>                   "  'value.format' = 'json'" +
>                   ")");
>   ```
>
>   + create table时，需要指定主键 **not enforced**
>
>   + Key不存在时，则为新增 +I
>
>   + Key 存在，但 Value = null,这表示删除前一个 相同key的数据 -D
>
>   + Key存在，且Value 不是 null ，则表示更新 +U
>
>     ![image-20221012113844544](http://ybll.vip/md-imgs/202210121138624.png)
>
> +++
>
> Kafka Source
>
> + 普通的 Kafka连接器 ：遇到 null【upsert kafka 采用 K-V 形式写入时，-D操作】，会自动忽略
>
>   ![image-20221012114635415](http://ybll.vip/md-imgs/202210121146492.png)
>
> + upsert kafka 连接器
>
>   ```java
>   tEnv.executeSql("create table ab4(" +
>                   " id int, " +
>                   " name string, " +
>                   " age int," +
>                   " primary key(id) not enforced " +
>                   ")with(" +
>                   "  'connector' = 'upsert-kafka', " +
>                   "  'topic' = 'yb1', " +
>                   "  'properties.bootstrap.servers' = 'hadoop162:9092', " +
>                   "  'key.format' = 'json', " +
>                   "  'value.format' = 'json'" +
>                   ")");
>   ```
>
>   + DDL 创建表时，需要指定主键 **not enforced**
>
>   + 每次都会从第一条数据开始构建整个过程【没有 checkpoint时】，
>
>   + **一般写的时候采用 upsert kafka ，读的时候采用普通的 kafka**
>
>     ![image-20221012115243407](http://ybll.vip/md-imgs/202210121152481.png)
>
>   +++
>
> + 测试 ：upsert kafka写入，kafka消费？
>
> + 总结 kafaka Source 消费 Kafka 的规律
>
>   > 使用 upsert-kafka sink 写入到 kafka 时
>   >
>   > + 使用 普通 kafka 消费：无需过多处理，null会自动过滤，所有数据都是 +I
>   >
>   > + 使用 upsert-kafka 消费：null会自动识别为 -D,每次从第一条数据开始输出，构建整个过程[从状态中恢复则不会从头读]
>   >
>   > + 流的方式消费：需要自定义反序列化器 【使用SimpleStringSchema 遇到 null 会报错，空指针异常】
>   >
>   >   ```java
>   >   public class FlinkSourceUtil {
>   >       public static SourceFunction<String> getKafkaSource(String topic, String groupId) {
>   >           Properties props = new Properties();
>   >           props.setProperty("bootstrap.servers", Constant.KAFKA_BROKERS);
>   >           props.setProperty("group.id", groupId);
>   >           FlinkKafkaConsumer<String> source = new FlinkKafkaConsumer<>(
>   >               topic,
>   >               new KafkaDeserializationSchema<String>() {
>   >                   // 是否要结束流
>   >                   @Override
>   >                   public boolean isEndOfStream(String nextElement) {
>   >                       // 从 kafka 消费数据, 流永远不停, 返回 false
>   >                       return false;
>   >                   }
>   >                   
>   >                   // 对传入的数组, 实现反序列化
>   >                   @Override
>   >                   public String deserialize(ConsumerRecord<byte[], byte[]> record) throws Exception {
>   >                       byte[] value = record.value();
>   >                       // 如果 value 是 null, 不需要反序列化
>   >                       if (value != null) {
>   >                           return new String(value, StandardCharsets.UTF_8);
>   >                       }
>   >                       return null;
>   >                   }
>   >                   
>   >                   // 读到数据的类型, 决定了流中的数据类型
>   >                   @Override
>   >                   public TypeInformation<String> getProducedType() {
>   >   //                    return Types.STRING;
>   >   //                    return TypeInformation.of(String.class);
>   >                       return TypeInformation.of(new TypeHint<String>() {});
>   >                   }
>   >               },
>   >               props
>   >           );
>   >           source.setStartFromLatest();
>   >           return source;
>   >       }
>   >   }
>   >   ```
>
> +++
>
> Interval Joins : 写select 语句进行join时，确定 上下 界
>
> +++
>
> Temporal Join : 
>
> + Versioned Tables : 版本表 的 Join，不同的时间【处理时间，时间时间】Join的字段不同
>
>   ```sql
>   -- 语法
>   select [cols_list]
>   from tbl1
>   [left] join tbl2 for system_time as of tbl1.{proctime|rowtime}
>   on tbl1.id = tbl2.id
>   ```
>
>   + 特殊语法 ：**for system_time as of ...**
>   + 左表必须提供时间属性：【处理时间|事件时间】
>
> 
>+++

### Lookup Join

> Temporal Join 的一种特殊情况，左表的 时间属性 必须是 **处理时间【pt as proctime() 】**
>
> 后续专门用来解决(补充维度) **事实表 Join 维度表**
>
> + 左表数据每次 join 右表最新的数据
>
> + 支持 mysql，不支持 phoenix
>
>   ```java
>   tEnv.executeSql("create table person(" +
>                   " id string, " +
>                   " pt as proctime()" +   // 处理时间
>                   ")" + SQLUtil.getKafkaSource("per", "atguigu"));
>   
>   tEnv.executeSql("CREATE  TABLE base_dic ( " +
>                   "  dic_code string, " +
>                   "  dic_name string " +
>                   ") WITH ( " +
>                   "  'connector' = 'jdbc', " +
>                   "  'url' = 'jdbc:mysql://hadoop162:3306/gmall2022?useSSL=false', " +
>                   "  'table-name' = 'base_dic', " +
>                   "  'username' = 'root', " +
>                   "  'password' = 'aaaaaa',  " +
>                   "'lookup.cache.max-rows' = '10'," +
>                   "'lookup.cache.ttl' = '1 minute' " +
>                   ")");
>   tEnv
>       .sqlQuery("select " +
>                 "person.id, " +
>                 "dic.dic_name " +
>                 "from person " +
>                 "join base_dic for system_time as of person.pt as dic " +
>                 "on person.id=dic.dic_code")
>       .execute()
>       .print();
>           
>   ```
>
>   ```
>   默认每次总是去查询数据库.
>   一般是用来事实表和维度表的 join. 维度表来说, 变化比较慢
>   ```
>
> +++

### Flink CDC

1.读取快照时的格式

```json
{
    "before": null,
    "after": {
        "source_table": "coupon_range",
        "source_type": "ALL",
        "sink_table": "dim_coupon_range",
        "sink_type": "dim",
        "sink_columns": "id,coupon_id,range_type,range_id",
        "sink_pk": "id",
        "sink_extend": null
    },
    "source": {
        "version": "1.5.4.Final",
        "connector": "mysql",
        "name": "mysql_binlog_source",
        "ts_ms": 0,
        "snapshot": "false",
        "db": "gmall_config",
        "sequence": null,
        "table": "table_process",
        "server_id": 0,
        "gtid": null,
        "file": "",
        "pos": 0,
        "row": 0,
        "thread": null,
        "query": null
    },
    "op": "r",
    "ts_ms": 1665024261972,
    "transaction": null
}		
```

2. udpate 配置表

   ```json
   {
       "before": {
           "source_table": "user_infoabc",
           "source_type": "ALL",
           "sink_table": "dim_user_info",
           "sink_type": "dim",
           "sink_columns": "id,login_name,name,user_level,birthday,gender,create_time,operate_time",
           "sink_pk": "id",
           "sink_extend": " SALT_BUCKETS = 3"
       },
       "after": {
           "source_table": "user_info",
           "source_type": "ALL",
           "sink_table": "dim_user_info",
           "sink_type": "dim",
           "sink_columns": "id,login_name,name,user_level,birthday,gender,create_time,operate_time",
           "sink_pk": "id",
           "sink_extend": " SALT_BUCKETS = 3"
       },
       "source": {
           "version": "1.5.4.Final",
           "connector": "mysql",
           "name": "mysql_binlog_source",
           "ts_ms": 1665026142000,
           "snapshot": "false",
           "db": "gmall_config",
           "sequence": null,
           "table": "table_process",
           "server_id": 1,
           "gtid": null,
           "file": "mysql-bin.000008",
           "pos": 415,
           "row": 0,
           "thread": null,
           "query": null
       },
       "op": "u",
       "ts_ms": 1665026142815,
       "transaction": null
   }
   ```

   3. delete

   ```json
   {
       "before": {
           "source_table": "user_info",
           "source_type": "ALL",
           "sink_table": "dim_user_info",
           "sink_type": "dim",
           "sink_columns": "id,login_name,name,user_level,birthday,gender,create_time,operate_time",
           "sink_pk": "id",
           "sink_extend": " SALT_BUCKETS = 3"
       },
       "after": null,
       "source": {
           "version": "1.5.4.Final",
           "connector": "mysql",
           "name": "mysql_binlog_source",
           "ts_ms": 1665026284000,
           "snapshot": "false",
           "db": "gmall_config",
           "sequence": null,
           "table": "table_process",
           "server_id": 1,
           "gtid": null,
           "file": "mysql-bin.000008",
           "pos": 998,
           "row": 0,
           "thread": null,
           "query": null
       },
       "op": "d",
       "ts_ms": 1665026284553,
       "transaction": null
   }	
   ```

   4. insert

   ```insert
   {
       "before": null,
       "after": {
           "source_table": "a",
           "source_type": "ALL",
           "sink_table": "b",
           "sink_type": "dim",
           "sink_columns": "a,b,c",
           "sink_pk": "id",
           "sink_extend": null
       },
       "source": {
           "version": "1.5.4.Final",
           "connector": "mysql",
           "name": "mysql_binlog_source",
           "ts_ms": 1665026350000,
           "snapshot": "false",
           "db": "gmall_config",
           "sequence": null,
           "table": "table_process",
           "server_id": 1,
           "gtid": null,
           "file": "mysql-bin.000008",
           "pos": 1445,
           "row": 0,
           "thread": null,
           "query": null
       },
       "op": "c",
       "ts_ms": 1665026350561,
       "transaction": null
   }
   ```

   5. 更新主键

      先删除，再新增



### TVFs Windows

### Top N



### ClickHouse





#### ods

> ods_db: maxwell 同步到 kafka【topic:  ods_db】
>
> ```json
> {
>   "database": "gmall2022",
>   "table": "user_info",
>   "type": "insert",
>   "ts": 1665021530,
>   "xid": 606,
>   "xoffset": 0,
>   "data": {
>     "id": 1,
>     "login_name": "cyp52665",
>     "nick_name": null,
>     "passwd": null,
>     "name": null,
>     "phone_num": "13691126476",
>     "email": "cyp52665@0355.net",
>     "head_img": null,
>     "user_level": "1",
>     "birthday": "1982-10-06",
>     "gender": null,
>     "create_time": "2022-10-06 01:58:50",
>     "operate_time": null,
>     "status": null
>   }
> ```
>
> +++
>
> ods_log: flume 同步到 kafka 【topic: ods_log】
>
> ```json
> {
>   "actions": [
>     {
>       "action_id": "cart_add",
>       "item": "22",
>       "item_type": "sku_id",
>       "ts": 1665021644293
>     },
>     {
>       "action_id": "get_coupon",
>       "item": "2",
>       "item_type": "coupon_id",
>       "ts": 1665021647586
>     }
>   ],
>   "common": {
>     "ar": "110000",
>     "ba": "iPhone",
>     "ch": "Appstore",
>     "is_new": "0",
>     "md": "iPhone X",
>     "mid": "mid_166755",
>     "os": "iOS 13.3.1",
>     "uid": "4",
>     "vc": "v2.1.134"
>   },
>   "displays": [
>     {
>       "display_type": "query",
>       "item": "25",
>       "item_type": "sku_id",
>       "order": 1,
>       "pos_id": 4
>     },
>     {
>       "display_type": "promotion",
>       "item": "26",
>       "item_type": "sku_id",
>       "order": 2,
>       "pos_id": 5
>     },
>     {
>       "display_type": "query",
>       "item": "14",
>       "item_type": "sku_id",
>       "order": 3,
>       "pos_id": 4
>     },
>     {
>       "display_type": "query",
>       "item": "16",
>       "item_type": "sku_id",
>       "order": 4,
>       "pos_id": 4
>     },
>     {
>       "display_type": "query",
>       "item": "19",
>       "item_type": "sku_id",
>       "order": 5,
>       "pos_id": 1
>     },
>     {
>       "display_type": "query",
>       "item": "2",
>       "item_type": "sku_id",
>       "order": 6,
>       "pos_id": 1
>     },
>     {
>       "display_type": "query",
>       "item": "35",
>       "item_type": "sku_id",
>       "order": 7,
>       "pos_id": 1
>     }
>   ],
>   "page": {
>     "during_time": 9880,
>     "item": "22",
>     "item_type": "sku_id",
>     "last_page_id": "home",
>     "page_id": "good_detail",
>     "source_type": "query"
>   },
>   "err": {
>     "error_code": 1803,
>     "msg": " Exception in thread \\  java.net.SocketTimeoutException\\n \\tat com.atgugu.gmall2020.mock.bean.log.AppError.main(AppError.java:xxxxxx)"
>   },
>   "start": {
>     "entry": "notice",
>     "loading_time": 14782,
>     "open_ad_id": 19,
>     "open_ad_ms": 8208,
>     "open_ad_skip_ms": 0
>   },
>   "ts": 1665021641000
> }
> ```
>
> +++

#### dim

> ##### ods_db  =>  flink  =>  dim层(hbase中)【 hbase-Phoenix JDBC】
>
> ```txt
> dim_activity_info
> dim_activity_rule
> dim_activity_sku
> dim_base_category1
> dim_base_category2
> dim_base_category3
> dim_base_province
> dim_base_region
> dim_base_trademark
> dim_coupon_info
> dim_coupon_range
> dim_financial_sku_cost
> dim_sku_info
> dim_spu_info
> dim_user_info
> ```
>
> +++
>
> 1. 过滤脏数据：filter() --> kafka 的 ods_db主题数据
>
> 2.  Flink CDC 读取配置表信息
>
>    ```java
>    // flink cdc ：先读取快照，后续mysql更新时又会基于binlog日志产生Josn数据流入执行
>    MySqlSource<String> tblProcessSource = MySqlSource.<String>builder()
>        .hostname("hadoop162")
>        .port(3306)
>        .databaseList("gmall_config")
>        .tableList("gmall_config.table_process")
>        .username("root")
>        .password("aaaaaa")
>        .deserializer(new JsonDebeziumDeserializationSchema())
>        .startupOptions(StartupOptions.initial()) //启动时都全量读一次
>        .build();
>    
>    SingleOutputStreamOperator<TableProcess> tprocessDS = env.fromSource(tblProcessSource,
>                    WatermarkStrategy.noWatermarks(),
>                    "mysql_tbl_process_source")
>        .map(...)
>    ```
>
> 3. 由配置信息表，在 Phoenix 中 创建/删除表
>
>    > + 创建 Phoenix 连接
>    >
>    > ```java
>    > public static final String PHOENIX_DRIVER = "org.apache.phoenix.jdbc.PhoenixDriver";
>    > public static final String PHOENIX_URL = "jdbc:phoenix:hadoop162,hadoop163,hadoop164:2181";
>    > ```
>    >
>    > + 根据配置信息表，拼接 create / drop 语句
>    >
>    > ```java
>    > StringBuffer sql = new StringBuffer();
>    > sql
>    >     .append("create table if not exists ")
>    >     .append(tp.getSinkTable())
>    >     .append("(")
>    >     .append(tp.getSinkColumns().replaceAll("[^,]+","$0 varchar"))
>    >     .append(", constraint pk primary key (")
>    >     .append(tp.getSinkPk() == null? "id" : tp.getSinkPk())
>    >     .append("))")
>    >     .append(tp.getSinkExtend() == null ? "":tp.getSinkExtend());
>    > System.out.println("维度建表语句：" + sql);
>    > 
>    > String sql = "drop table " + tp.getSinkTable();
>    > ```
>
> 4. 数据流 和 配置流 进行connect
>
>    > + 配置流 广播
>    >
>    > ```java
>    > // 用 mysql 中的 source_table + all 做为 key
>    > // key的类型String:  数据流中的数据根据 key 来获取一个配置信息(TableProcess)
>    > // value的类型: ableProcess
>    > MapStateDescriptor<String, TableProcess> tpStateDesc = new MapStateDescriptor<>("tpState", String.class, TableProcess.class);
>    > // 1. 先把配置流做成广播流
>    > BroadcastStream<TableProcess> tpBcStream = confStream.broadcast(tpStateDesc);
>    > ```
>    >
>    > + dataSource .connect(tblProcessSource).process() ...
>    >
>    >   > + 广播流中 ：拼接 key ,写入 广播状态 `MapStateDescriptor<String, TableProcess> tpStateDesc`
>    >   >
>    >   > + 数据流中 ：拼接 key,与 广播状态中进行匹配，匹配成功则返回 `Tuple2<JSONObject, TableProcess>`
>    >   >
>    >   >   > JSONObject : ods_db中，maxwell同步后，取 data 字段，**添加 op_type 字段** （maxwell 的 type字段）
>    >   >   >
>    >   >   > TableProcess : 根据配置信息，后续插入数据，字段在 data中都有
>
> 5.  从 `Tuple2<JSONObject, TableProcess>` 数据流中，过滤出 插入数据 需要的字段[在 TableProcess 中]
>
>    ```java
>    JSONObject dataObj = value.f0;
>    TableProcess tblProcess = value.f1;
>    List<String> colsList = Arrays.asList(tblProcess.getSinkColumns().split(","));
>    Set<String> keys = dataObj.keySet();
>    keys.removeIf(key -> !colsList.contains(key) && !"op_type".equals(key));
>    return Tuple2.of(dataObj, tblProcess);
>    ```
>
> 6.  插入数据
>
>    ```java
>    DataStreamSink<Tuple2<JSONObject, TableProcess>> ds = dataSource.addSink(FlinkSinkUtils.getPhoenixSink());
>    
>    // 工具类
>    public static SinkFunction<Tuple2<JSONObject, TableProcess>> getPhoenixSink() {
>        return new PhoenixSink();
>    }
>    
>    // 自定义 PhoenixSink
>    public class PhoenixSink extends RichSinkFunction<Tuple2<JSONObject, TableProcess>> {
>    
>        private DruidDataSource dataSource;  // 自定义 Druid连接池
>    
>        @Override
>        public void open(Configuration parameters) throws Exception {
>            dataSource = DruidPhoenixDSUtil.getDataSource();
>        }
>        @Override
>        public void invoke(Tuple2<JSONObject, TableProcess> value, Context context) throws Exception {
>            JSONObject data = value.f0;
>            TableProcess tableProcess = value.f1;
>            DruidPooledConnection conn = dataSource.getConnection();
>            StringBuilder sql = new StringBuilder();
>            sql.append("upsert into ")
>                    .append(tableProcess.getSinkTable())
>                    .append("(")
>                    .append(tableProcess.getSinkColumns())
>                    .append(") values(")
>                    .append(tableProcess.getSinkColumns().replaceAll("[^,]+","?"))
>                    .append(")");
>            System.out.println("Phoenix 插入语句 ： " + sql.toString());
>    
>            //获取预处理对象，填充占位符
>            PreparedStatement ps = conn.prepareStatement(sql.toString());
>            String[] cols = tableProcess.getSinkColumns().split(",");
>            for( int i = 0; i<cols.length;i++ ){
>                String col = cols[i];
>                String v = data.getString(col);
>                ps.setString(i + 1, v);
>            }
>            //执行语句
>            ps.execute();
>            conn.commit(); // 手动提交
>            ps.close(); //关闭预处理对象
>            conn.close(); // 归还连接，连接池中获取，则是归还；DriverManager获取，则是关闭
>        }
>    }
>    ```
>
> +++

+++



#### dwd

> ##### ods_log  =>  flink  => kafka
>
> > 1. ETL 过滤 json 数据 => ods_log
> >
> > 2. 新老用户 纠正
> >
> >    ```json
> >    "common": {
> >        "ar": "110000",
> >        "ba": "iPhone",
> >        "ch": "Appstore",
> >        "is_new": "0",
> >        "md": "iPhone X",
> >        "mid": "mid_166755",
> >        "os": "iOS 13.3.1",
> >        "uid": "4",
> >        "vc": "v2.1.134"
> >    },
> >    "ts": 1665021641000
> >    ```
> >
> >    keyBy() : mid 字段；
> >
> >    ValueState<String> valueState : 单值状态 存每个 mid 第一次登录的日期( ts字段 计算)
> >
> > 3. 分流输出 到 kafka
> >
> >    ```java
> >    OutputTag<JSONObject> errOutputTag = new OutputTag<JSONObject>("errTag") {
> >    }; // 错误信息流 dwd_traffic_err
> >    OutputTag<JSONObject> pageOutputTag = new OutputTag<JSONObject>("pageTag") {
> >    }; // 页面信息流 dwd_traffic_page;后续用的多
> >    OutputTag<JSONObject> displayOutputTag = new OutputTag<JSONObject>("displayTag") {
> >    }; // 曝光信息流 dwd_traffic_display
> >    OutputTag<JSONObject> actionOutputTag = new OutputTag<JSONObject>("actionTag") {
> >    }; // 事件信息流 dwd_traffic_action
> >    // 主流 输出到 dwd_traffic_start
> >    ```
> >
> >    ```java
> >    streamsMap.get(START).map(JSONAware::toJSONString).addSink(FlinkSinkUtils.getKafkaSink(Constant.TOPIC_TRAFFIC_START));
> >    
> >    // kafkaSink; [输出到 kafka; FlinkKafkaProducer]
> >    public static SinkFunction<String> getKafkaSink(String topic) {
> >    
> >        Properties propers = new Properties();
> >        propers.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, Constant.KAFKA_BROKERS);
> >        propers.setProperty(ProducerConfig.TRANSACTION_TIMEOUT_CONFIG, 15 * 60 * 1000 + "");
> >        return new FlinkKafkaProducer<String>(
> >            "default",
> >            new KafkaSerializationSchema<String>() {
> >                @Override
> >                public ProducerRecord<byte[], byte[]> serialize(String s, @Nullable Long aLong) {
> >                    return new ProducerRecord<>(topic,s.getBytes(StandardCharsets.UTF_8));
> >                }
> >            },
> >            propers,
> >            FlinkKafkaProducer.Semantic.EXACTLY_ONCE // 精确一次
> >        );
> >    }
> >    ```
> >
> > +++
>
> +++
>
> 
>
> ##### ods_db  =>  flink  =>  kafka 
>
> > 【kafka Source】使用 flink sql 从 kafka 读取数据 : ods_db 主题
> >
> > ```java
> > // kafka Source: create table ... with(...)
> > tEnv.executeSql(
> >                 "create table ods_db(" +
> >                         "`database` string, " +
> >                         "`table` string, " +
> >                         "`type` string, " +
> >                         "`ts` bigint, " +
> >                         "`data` map<string, string>, " +
> >                         "`old` map<string, string>, " +
> >                         "`pt` as proctime() " +
> >                         ")"
> >                         + SQLUtils.getKafkaSource(Constant.TOPIC_ODS_DB,groupId));
> > 
> > public static String getKafkaSource(String topic, String groupId) {
> >     return "with(" +
> >         " 'connector' = 'kafka', " +
> >         " 'topic' = '" + topic + "', " +
> >         " 'properties.bootstrap.servers' = '" + Constant.KAFKA_BROKERS + "', " +
> >         " 'properties.group.id' = '" + groupId + "', " +
> >         " 'scan.startup.mode' = 'latest-offset', " +
> >         " 'format' = 'json'" +
> >         ")";
> > }
> > ```
> >
> > 【mysql Source】使用 flink sql 从 mysql 读取数据 : base_dic 表
> >
> > ```java
> > public void readBaseDic(StreamTableEnvironment tEnv){
> >     tEnv.executeSql("create table base_dic ( " +
> >             "  dic_code string, " +
> >             "  dic_name string " +
> >             ") WITH ( " +
> >             "  'connector' = 'jdbc', " +
> >             "  'driver' = 'com.mysql.cj.jdbc.Driver', " +
> >             "  'url' = 'jdbc:mysql://hadoop162:3306/gmall2022?useSSL=false', " +
> >             "  'table-name' = 'base_dic', " +
> >             "  'username' = 'root', " +
> >             "  'password' = 'aaaaaa',  " +
> >             "  'lookup.cache.max-rows' = '10'," +
> >             "  'lookup.cache.ttl' = '1 hour' " +
> >             ")");
> > }
> > ```
> >
> > 【kafka Sink / upsert-kafka Sink】: 没有 left join 时，可以使用，普通kafka sink 不能写入 更新及删除
> >
> > ```java
> > tEnv.executeSql("create table dwd_trade_cart_add(" +
> >                 "id string,  " +
> >                 "user_id string,  " +
> >                 "sku_id string,  " +
> >                 "source_id string,  " +
> >                 "source_type_code string,  " +
> >                 "source_type_name string,  " +
> >                 "sku_num string,  " +
> >                 "ts bigint  " +
> >                 ")" + SQLUtils.getKafkaSink(Constant.TOPIC_DWD_TRADE_CART_ADD));
> > result.executeInsert("dwd_trade_cart_add");
> > 
> > public static String getKafkaSink(String topic) {
> >     return "with(" +
> >         " 'connector' = 'kafka', " +
> >         " 'topic' = '" + topic + "', " +
> >         " 'properties.bootstrap.servers' = '" + Constant.KAFKA_BROKERS + "', " +
> >         " 'format' = 'json'" +
> >         ")";
> > }
> > 
> > // UpsertKafkaSink:支持左连接
> > public static String getUpsertKafkaSink(String topic) {
> >     return "with( " +
> >         " 'connector' = 'upsert-kafka'," +
> >         " 'topic' = '"+ topic +"', " +
> >         " 'properties.bootstrap.servers' = '" + Constant.KAFKA_BROKERS + "', " +
> >         " 'key.format' = 'json', " +
> >         " 'value.format' = 'json' " +
> >         ")";
> > }
> > ```
> >
> > 
>
> +++

+++



#### dws

> `dwd_数据域_统计粒度_业务过程_统计周期(window)`
>
> 









