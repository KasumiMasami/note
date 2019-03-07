#### 一、开发调优
-	避免重复创建RDD，类似不要重复创建变量
-	尽可能复用同一个RDD，避免多个RDD存在数据重叠或包含
-	需要多次使用的RDD要持久化，数据量小的缓存在内存，放不下再考虑内存加硬盘
-	尽量避免使用shuffle类算子(groupByKey/join/repartition)
Ps:  join操作可以转换为将较小表广播到每台executor，然后大表map匹配）
-	使用map-side预聚合的shuffle算子，减少需要shuffle的数据
Eg:  reduceByKey替代groupByKey
-	有数据库连接操作的应该使用foreachPartitions替代foreach，在每个partition只用维护一个连接即可
-	使用filter之后进行重分区操作减少分区数
-	使用Kryo优化序列化性能
#### 二、资源调优
-	根据数据量选择合适num-executors、executor-memory、executor-cores和driver-memory
-	修改参数spark.default.parallelism，设置每个stage的task数量。
Ps:  一般num-executors * executor-cores的2~3倍较为合适
-	根据实际需求调节spark.storage.memoryFraction和spark.shuffle.memoryFraction的大小
Ps:  RDD持久化多的调大spark.storage.memoryFraction(默认0.6)，shuffle类操作多的调大spark.shuffle.memoryFraction(默认0.2)，还有0.2是executor自身预留的
