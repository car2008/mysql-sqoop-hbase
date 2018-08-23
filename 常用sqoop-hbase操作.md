
# 测试mysql与hbasse，mysql与hive
## sqoop导入mysql数据到hbase
```
./sqoop import --connect jdbc:mysql://yarn001:3306/experiment_datas --username root --password root --table u_sample --hbase-table experiment_datas --hbase-row-key sample_id,sample_type --column-family u_sample 
```
### 测试1
hive读取hbase的数据：
创建外部表：
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
创建内部表：
```
CREATE TABLE experiment_datas.u_sample_copy2(sample_id string,sample_type string,sample_current_quantity double,
sample_unit String,sample_empirical_method string,sample_current_level string,sample_parent_level string,samplegrade String,
sample_notes string,patient_id string,cancer_type string,clinicaldiagnosis string,pathologicdiagnosis string,is_project_sample string,
app_order_id string,app_sample_id string,receiveddt TIMESTAMP,u_deadline TIMESTAMP,collectiondt TIMESTAMP,
record_create_date TIMESTAMP,record_update_date TIMESTAMP);
```
将外部表数据导入内部表
```
insert overwrite table experiment_datas.u_sample_copy2 select key.sample_id,key.sample_type,sample_current_quantity,sample_unit,
sample_empirical_method,sample_current_level,sample_parent_level,samplegrade,sample_notes,patient_id,cancer_type,clinicaldiagnosis,
pathologicdiagnosis,is_project_sample,app_order_id,app_sample_id,receiveddt,u_deadline,collectiondt,record_create_date,record_update_date from experiment_datas.u_sample_copy1;
```
sqoop导出到mysql
```
./sqoop export --connect jdbc:mysql://yarn001:3306/experiment_datas --username root --password root --table u_sample_copy1 --export-dir /user/hive/warehouse/experiment_datas.db/u_sample_copy2 --input-fields-terminated-by '\001' --input-null-string '\\N' --input-null-non-string '\\N';
```

### 测试2
创建内部表：
```
CREATE TABLE experiment_datas.u_sample_copy3(sample_current_quantity double,
sample_unit String,sample_empirical_method string,sample_current_level string,sample_parent_level string,samplegrade String,
sample_notes string,patient_id string,cancer_type string,clinicaldiagnosis string,pathologicdiagnosis string,is_project_sample string,
app_order_id string,app_sample_id string,receiveddt TIMESTAMP,u_deadline TIMESTAMP,collectiondt TIMESTAMP,
record_create_date TIMESTAMP,record_update_date TIMESTAMP);
```
将外部表数据导入内部表
```
insert overwrite table experiment_datas.u_sample_copy3 select sample_current_quantity,sample_unit,
sample_empirical_method,sample_current_level,sample_parent_level,samplegrade,sample_notes,patient_id,cancer_type,clinicaldiagnosis,
pathologicdiagnosis,is_project_sample,app_order_id,app_sample_id,receiveddt,u_deadline,collectiondt,record_create_date,record_update_date from experiment_datas.u_sample_copy1;
```
sqoop导出到mysql
```
./sqoop export --connect jdbc:mysql://yarn001:3306/experiment_datas --username root --password root --table u_sample_copy3 --columns sample_current_quantity,sample_unit,sample_empirical_method,sample_current_level,sample_parent_level,samplegrade,sample_notes,patient_id,cancer_type,clinicaldiagnosis,pathologicdiagnosis,is_project_sample,app_order_id,app_sample_id,receiveddt,u_deadline,collectiondt,record_create_date,record_update_date --export-dir /user/hive/warehouse/experiment_datas.db/u_sample_copy3 --input-fields-terminated-by '\t' --input-null-string '\\N' --input-null-non-string '\\N';
```
### 测试3
创建内部表：
```
CREATE TABLE experiment_datas.u_sample_copy4(key string,sample_current_quantity double,
sample_unit String,sample_empirical_method string,sample_current_level string,sample_parent_level string,samplegrade String,
sample_notes string,patient_id string,cancer_type string,clinicaldiagnosis string,pathologicdiagnosis string,is_project_sample string,
app_order_id string,app_sample_id string,receiveddt TIMESTAMP,u_deadline TIMESTAMP,collectiondt TIMESTAMP,
record_create_date TIMESTAMP,record_update_date TIMESTAMP);
```
将外部表数据导入内部表
```
insert overwrite table experiment_datas.u_sample_copy4 select * from experiment_datas.u_sample_copy1;
```
sqoop导出到mysql
```
./sqoop export --connect jdbc:mysql://yarn001:3306/experiment_datas --username root --password root --table u_sample_copy4 --export-dir /user/hive/warehouse/experiment_datas.db/u_sample_copy4 --input-fields-terminated-by '\001' --input-null-string '\\N' --input-null-non-string '\\N';
```

### 测试4
hive读取hbase的数据：
创建外部表：
```
CREATE EXTERNAL TABLE experiment_datas.u_sample_copy1 (sample_id string,sample_type string ,sample_current_quantity double,
sample_unit String,sample_empirical_method string,sample_current_level string,sample_parent_level string,samplegrade String,
sample_notes string,patient_id string,cancer_type string,clinicaldiagnosis string,pathologicdiagnosis string,is_project_sample string,
app_order_id string,app_sample_id string,receiveddt TIMESTAMP,u_deadline TIMESTAMP,collectiondt TIMESTAMP,
record_create_date TIMESTAMP,record_update_date TIMESTAMP)
   STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
   WITH SERDEPROPERTIES (
    "hbase.columns.mapping" =
     ":key,u_sample:sample_type,u_sample:sample_current_quantity,u_sample:sample_unit,u_sample:sample_empirical_method,
     u_sample:sample_current_level,u_sample:sample_parent_level,u_sample:samplegrade,u_sample:sample_notes,u_sample:patient_id,
     u_sample:cancer_type,u_sample:clinicaldiagnosis,u_sample:pathologicdiagnosis,u_sample:is_project_sample,u_sample:app_order_id,
     u_sample:app_sample_id,u_sample:receiveddt,u_sample:u_deadline,u_sample:collectiondt,u_sample:record_create_date,u_sample:record_update_date"
   )
   TBLPROPERTIES( "hbase.table.name" = "experiment_datas",
    "hbase.mapred.output.outputtable" = "experiment_datas");
```    
创建内部表：
```
CREATE TABLE experiment_datas.u_sample_copy2(sample_id string,sample_type string,sample_current_quantity double,
sample_unit String,sample_empirical_method string,sample_current_level string,sample_parent_level string,samplegrade String,
sample_notes string,patient_id string,cancer_type string,clinicaldiagnosis string,pathologicdiagnosis string,is_project_sample string,
app_order_id string,app_sample_id string,receiveddt TIMESTAMP,u_deadline TIMESTAMP,collectiondt TIMESTAMP,
record_create_date TIMESTAMP,record_update_date TIMESTAMP);
```
将外部表数据导入内部表
```
insert overwrite table experiment_datas.u_sample_copy2 select * from experiment_datas.u_sample_copy1;
```
sqoop导出到mysql
```
./sqoop export --connect jdbc:mysql://yarn001:3306/experiment_datas --username root --password root --table u_sample_copy2 --export-dir /user/hive/warehouse/experiment_datas.db/u_sample_copy2 --input-fields-terminated-by '\001' --input-null-string '\\N' --input-null-non-string '\\N';
```
```
DROP TABLE IF EXISTS u_sample_copy2;
```
### 测试5
hive读取hbase的数据：
创建外部表：
```
CREATE EXTERNAL TABLE experiment_datas.u_sample_copy1 (sample_id string,sample_type string ,sample_current_quantity double,
sample_unit String,sample_empirical_method string,sample_current_level string,sample_parent_level string,samplegrade String,
sample_notes string,patient_id string,cancer_type string,clinicaldiagnosis string,pathologicdiagnosis string,is_project_sample string,
app_order_id string,app_sample_id string,receiveddt TIMESTAMP,u_deadline TIMESTAMP,collectiondt TIMESTAMP,
record_create_date TIMESTAMP,record_update_date TIMESTAMP)
   STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
   WITH SERDEPROPERTIES (
    "hbase.columns.mapping" =
     ":key,u_sample:sample_type,u_sample:sample_current_quantity,u_sample:sample_unit,u_sample:sample_empirical_method,
     u_sample:sample_current_level,u_sample:sample_parent_level,u_sample:samplegrade,u_sample:sample_notes,u_sample:patient_id,
     u_sample:cancer_type,u_sample:clinicaldiagnosis,u_sample:pathologicdiagnosis,u_sample:is_project_sample,u_sample:app_order_id,
     u_sample:app_sample_id,u_sample:receiveddt,u_sample:u_deadline,u_sample:collectiondt,u_sample:record_create_date,u_sample:record_update_date"
   )
   TBLPROPERTIES( "hbase.table.name" = "experiment_datas",
    "hbase.mapred.output.outputtable" = "experiment_datas");
```    
创建内部表：
```
CREATE TABLE experiment_datas.u_sample_copy2(sample_id string,sample_type string,sample_current_quantity double,
sample_unit String,sample_empirical_method string,sample_current_level string,sample_parent_level string,samplegrade String,
sample_notes string,patient_id string,cancer_type string,clinicaldiagnosis string,pathologicdiagnosis string,is_project_sample string,
app_order_id string,app_sample_id string,receiveddt TIMESTAMP,u_deadline TIMESTAMP,collectiondt TIMESTAMP,
record_create_date TIMESTAMP,record_update_date TIMESTAMP) row format delimited fields terminated by '\t';
```
将外部表数据导入内部表
```
insert overwrite table experiment_datas.u_sample_copy2 select * from experiment_datas.u_sample_copy1;
```
sqoop导出到mysql
```
./sqoop export --connect jdbc:mysql://yarn001:3306/experiment_datas --username root --password root --table u_sample_copy2 --export-dir /user/hive/warehouse/experiment_datas.db/u_sample_copy2 --input-fields-terminated-by '\t' --input-null-string '\\N' --input-null-non-string '\\N';
```
```
DROP TABLE IF EXISTS u_sample_copy2;
```
```
select count(1) from u_sample_copy2;
```
### 测试6 （mysql与hive导入导出）通过测试 采用！！！！！！！！！
```
./sqoop import --connect jdbc:mysql://yarn001:3306/experiment_datas --username root --password root --table u_sample --hive-import --target-dir /user/hive/warehouse/ --hive-table u_sample_copy2 -m 1 --hive-overwrite --hive-delims-replacement "|" --fields-terminated-by "\t" --null-string '\\N' --null-non-string '\\N';
```
参数解读：
* --hive-overwrite ：对这个表的数据进行覆盖
* --hive-delims-replacement "|" ：[自动在导入时把\n和\r换成|，防止因为换行符造成记录条数增加；也可以使用--hive-drop-import-delims把这些去除;前两个不起作用的话，可以用--map-column-java参数来显示指定列为String类型](https://stackoverflow.com/questions/28076200/hive-drop-import-delims-not-removing-newline-while-using-hcatalog-in-sqoop)
* --fields-terminated-by "\t" ： 当mysql中的数据导入到hdfs中，默认使用的分隔符是逗号，可配置
* --null-string '\\N' --null-non-string '\\N' ：Sqoop默认将NULL值导入为字符串NULL，然而，Hive使用\N来表示NULL值
```
./sqoop export --connect jdbc:mysql://yarn001:3306/experiment_datas --username root --password root --table u_sample_copy2 --columns sample_id,sample_type,sample_current_quantity,sample_unit,sample_empirical_method,sample_current_level,sample_parent_level,samplegrade,sample_notes,patient_id,cancer_type,clinicaldiagnosis,pathologicdiagnosis,is_project_sample,app_order_id,app_sample_id,receiveddt,u_deadline,collectiondt,record_create_date,record_update_date --export-dir /user/hive/warehouse/u_sample_copy2 --input-fields-terminated-by '\t' --input-null-string '\\N' --input-null-non-string '\\N';
```
参数解读：
* --columns ：要导出的列名
*  --input-fields-terminated-by '\t' ：与导入时对应的分隔符
* --input-null-string '\\N' --input-null-non-string '\\N' ： 在导入作业时，应该追加参数—NULL字符串和—NULL -non-string;在导出作业时，如果希望正确地保留NULL值，则应该追加参数—input-null-string和—input-null-non-string
### 测试7（从hive到hbase）通过测试 采用！！！！！！！！！
1. 在hbase中创建表及columnfamily
```
create 'experiment_datas','u_sample'
```
2. hive中创建外部表与hbase中的表关联
```
CREATE EXTERNAL TABLE experiment_datas.u_sample_copy3 (sample_id string,sample_type string ,sample_current_quantity double,
sample_unit String,sample_empirical_method string,sample_current_level string,sample_parent_level string,samplegrade String,
sample_notes string,patient_id string,cancer_type string,clinicaldiagnosis string,pathologicdiagnosis string,is_project_sample string,
app_order_id string,app_sample_id string,receiveddt TIMESTAMP,u_deadline TIMESTAMP,collectiondt TIMESTAMP,
record_create_date TIMESTAMP,record_update_date TIMESTAMP)
   STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
   WITH SERDEPROPERTIES (
    "hbase.columns.mapping" =
     ":key,u_sample:sample_type,u_sample:sample_current_quantity,u_sample:sample_unit,u_sample:sample_empirical_method,
     u_sample:sample_current_level,u_sample:sample_parent_level,u_sample:samplegrade,u_sample:sample_notes,u_sample:patient_id,
     u_sample:cancer_type,u_sample:clinicaldiagnosis,u_sample:pathologicdiagnosis,u_sample:is_project_sample,u_sample:app_order_id,
     u_sample:app_sample_id,u_sample:receiveddt,u_sample:u_deadline,u_sample:collectiondt,u_sample:record_create_date,u_sample:record_update_date"
   )
   TBLPROPERTIES( "hbase.table.name" = "experiment_datas",
    "hbase.mapred.output.outputtable" = "experiment_datas");
```
3. 将hive中内部表的数据导入上面创建的外部表，则可在hbase中查看数据
```
insert overwrite table experiment_datas.u_sample_copy3 select * from experiment_datas.u_sample_copy2;
```
