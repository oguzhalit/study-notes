# 10 Sqoop

Sqoop 的导入和导出是相对于 Hadoop 而言，导入到 Hadoop 称为 Sqoop 导入，导出 Hadoop 称为 Sqoop 导出

## 10.1 Sqoop 导入
### 10.1.1 导入 MySQL 表数据到 HDFS

```shell
bin/sqoop import --connect jdbc:mysql://192.168.46.34:3306/test --username root --password '' --target-dir /sqoopresult --table user --m 1
```
### 10.1.2 导入 MySQL 表数据到 Hive 表

```shell
bin/sqoop create-hive-table --connect jdbc:mysql://192.168.52.1:3306/test --table user --username root --password '' --hive-table kkk.abcd
```
### 10.1.3 导入表数据子集

```shell
bin/sqoop import --connect jdbc:mysql://192.168.52.1:3306/test --username root --password '' --where "age > 22" --target-dir /wherequery --table user --m 1

bin/sqoop import --connect jdbc:mysql://192.168.52.1:3306/test --username root --password '' --target-dir /wherequery12 --query 'select * from user where age < 23 and $CONDITIONS' --fields-terminated-by '\t' --m 1
```
### 10.1.4 增量导入

```shell
bin/sqoop import --connect jdbc:mysql://172.16.1.177:3306/test --username root --password '' --table user --m 1 --target-dir /wherequery13 --incremental append --check-column age --last-value 22
```
## 10.2 Sqoop 导出
### 10.2.1 导出 HDFS 数据到 MySQL

```shell
sqoop export \ 
--connect jdbc:mysql://hdp-node-01:3306/webdb 
--username root 
--password root  \ 
--table click_stream_visit  \ 
--export-dir /user/hive/warehouse/dw_click.db/click_stream_visit/datestr=2013-09-18 \ 
--input-fields-terminated-by '\001' 
```