## hive查询hbase上的json数据：

### 首先创建外部表读取hbase数据
```
CREATE EXTERNAL TABLE experiment_datas.chemical_2 (key struct<sample_id:string,splitTime:string>,datas string)
ROW FORMAT DELIMITED 
COLLECTION ITEMS TERMINATED BY ':'
   STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
   WITH SERDEPROPERTIES (
    "hbase.columns.mapping" =
     ":key,mutation_chemo:datas"
   )
   TBLPROPERTIES( "hbase.table.name" = "sample_summarize_jsonarrsy",
    "hbase.mapred.output.outputtable" = "sample_summarize_jsonarrsy");
``` 


### 再处理string类型的json

 ``` 
select rr.detection_result
from (
select split(regexp_replace(regexp_extract(datas,'^\\[(.+)\\]$',1),'\\}\\,\\{', '\\}\\|\\|\\{'),'\\|\\|') as str
from chemical_5) pp
lateral view explode(pp.str) ss as col 
lateral view json_tuple(ss.col,'sample_id','gene','locus','detection_result','cancer_type','chrom','pos') rr as sample_id,gene,locus,detection_result,cancer_type,chrom,pos where gene='XRCC1' AND chrom='chr19';
``` 
