# kettle简单使用
### [点击下载kettle](https://sourceforge.net/projects/pentaho/files/Data%20Integration/7.1/pdi-ce-7.1.0.0-12.zip/download) </br>
下载完成是一个zip压缩包:pdi-ce-7.1.0.0-12.zip
#### 如果在windows下使用,则在解压后需要先做一下步骤再启动：
	1. 下载 [infobright_jni_64bit.dll或者infobright_jni_32bit.dll](https://github.com/EidosMedia/infobright-core/tree/master/lib)
	 放到Windows system path (for example %WINDIR%/System32/)
	2. 将mysql连接的jar包mysql-connector-java-5.1.46-bin.jar放到data-integration\lib和data-integration\libswt\win64
#### 然后就可以启动sqoon.bat
	注意：打开之后最好把配置文件改一下，防止自动把空字符创换为null：
	在"编辑"栏下选择"编辑kettle.properties文件"中KETTLE_EMPTY_STRING_DIFFERS_FROM_NULL=Y，注意修改完要重启一下才能生效，要不然还是会报错!!!
#### 点击转换，然后点击核心对象配置导入导出表
#### 优化：
连接数据库时配置
```
useServerPrepStmts=false
rewriteBatchedStatements=true
useCompression=true
```
可以十分明显提高读取和插入速度，将日志可以设为错误日志级别
