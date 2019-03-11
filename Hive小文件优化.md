#### 一、为什么小文件多会导致Hive查询变慢
Hive查询的map任务数由文件个数决定，小文件过多会导致资源浪费（ps: 小文件太多对于namenode的性能也有影响）
#### 二、优化方法
- map输入的小文件合并：
```sql
set mapred.max.split.size=256000000;  -- 每个Map最大输入大小(这个值决定了合并后文件的数量)
set mapred.min.split.size.per.node=100000000; -- 节点上split的最小大小(这个值决定了多个DataNode上的文件是否需要合并)
set mapred.min.split.size.per.rack=100000000; -- 交换机下split的最小大小(这个值决定了多个交换机上的文件是否需要合并)  
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat; -- 执行Map前进行小文件合并
```
- 输出结果合并：
```sql
set hive.merge.mapfiles=true -- 在map-only job后合并文件
set hive.merge.mapredfiles=true -- 在map-reduce job后合并文件
set hive.merge.size.per.task=1024000000 -- 合并后每个文件的大小，1G
set hive.merge.smallfiles.avgsize=1024000000 -- 输出文件的平均文件大小，是决定是否执行合并操作的阈值，1G
```
ps: 如果通过SparkSQL访问Hive，可以直接通过reparation后persist来减少输出的小文件

- 插入分区的操作加上distribute by rand()
可以使用distribute by rand() 将数据随机分配给Reduce，避免出现有的文件特别大, 有的文件特别小。
```sql
insert overwrite table table_name partition(dt)
select * from other_table_name distribute by rand();
```

- 修改表存储格式
使用Sequencefile作为表存储格式，不要用textfile，可以在一定程度上减少小文件。
