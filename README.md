# mysql-sqoop-hbase
### HBASE特点：
hbase是bigtable的开源山寨版本。是建立的hdfs之上，提供高可靠性、高性能、列存储、可伸缩、实时读写的数据库系统。<br>
它介于nosql和RDBMS之间，仅能通过主键(row key)和主键的range来检索数据，仅支持单行事务(可通过hive支持来实现多表join等复杂操作)。主要用来存储非结构化和半结构化的松散数据。
与hadoop一样，Hbase目标主要依靠横向扩展，通过不断增加廉价的商用服务器，来增加计算和存储能力。<br>
HBase中的表一般有这样的特点：<br>
1 大：一个表可以有上亿行，上百万列<br>
2 面向列:面向列(族)的存储和权限控制，列(族)独立检索。<br>
3 稀疏:对于为空(null)的列，并不占用存储空间，因此，表可以设计的非常稀疏。<br>

#### sqoop导入mysql数据到hbase
```
./sqoop import --connect jdbc:mysql://yarn001:3306/experiment_datas --username root --password root --table u_sample --hbase-table experiment_datas --hbase-row-key sample_id,sample_type --column-family u_sample
```

#### hive读取hbase的数据步骤：
##### 创建外部表：
```
CREATE EXTERNAL TABLE experiment_datas.u_sample_copy1 (key struct<sample_id:string,sample_type:string>,sample_current_quantity double,
sample_unit String,sample_empirical_method string,sample_current_level string,sample_parent_level string,samplegrade String,
sample_notes string,patient_id string,cancer_type string,clinicaldiagnosis string,pathologicdiagnosis string,is_project_sample string,
app_order_id string,app_sample_id string,receiveddt TIMESTAMP,u_deadline TIMESTAMP,collectiondt TIMESTAMP,
record_create_date TIMESTAMP,record_update_date TIMESTAMP)
ROW FORMAT DELIMITED 
COLLECTION ITEMS TERMINATED BY '_'
   STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
   WITH SERDEPROPERTIES (
    "hbase.columns.mapping" =
     ":key,u_sample:sample_current_quantity,u_sample:sample_unit,u_sample:sample_empirical_method,
     u_sample:sample_current_level,u_sample:sample_parent_level,u_sample:samplegrade,u_sample:sample_notes,u_sample:patient_id,
     u_sample:cancer_type,u_sample:clinicaldiagnosis,u_sample:pathologicdiagnosis,u_sample:is_project_sample,u_sample:app_order_id,
     u_sample:app_sample_id,u_sample:receiveddt,u_sample:u_deadline,u_sample:collectiondt,u_sample:record_create_date,u_sample:record_update_date"
   )
   TBLPROPERTIES( "hbase.table.name" = "experiment_datas",
    "hbase.mapred.output.outputtable" = "experiment_datas");
```    
##### 创建内部表：
```
CREATE TABLE experiment_datas.u_sample_copy2(sample_id string,sample_type string,sample_current_quantity double,
sample_unit String,sample_empirical_method string,sample_current_level string,sample_parent_level string,samplegrade String,
sample_notes string,patient_id string,cancer_type string,clinicaldiagnosis string,pathologicdiagnosis string,is_project_sample string,
app_order_id string,app_sample_id string,receiveddt TIMESTAMP,u_deadline TIMESTAMP,collectiondt TIMESTAMP,
record_create_date TIMESTAMP,record_update_date TIMESTAMP);
```
##### 将外部表数据导入内部表
```
insert overwrite table experiment_datas.u_sample_copy2 select key.sample_id,key.sample_type,sample_current_quantity,sample_unit,
sample_empirical_method,sample_current_level,sample_parent_level,samplegrade,sample_notes,patient_id,cancer_type,clinicaldiagnosis,
pathologicdiagnosis,is_project_sample,app_order_id,app_sample_id,receiveddt,u_deadline,collectiondt,record_create_date,record_update_date from experiment_datas.u_sample_copy1;
```
#### sqoop导出到mysql
```
./sqoop export --connect jdbc:mysql://yarn001:3306/experiment_datas --username root --password root --table u_sample_copy1 --export-dir /user/hive/warehouse/experiment_datas.db/u_sample_copy2 --input-fields-terminated-by '\001' --input-null-string '\\N' --input-null-non-string '\\N';
```
