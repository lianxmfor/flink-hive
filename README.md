# flink-hive

1. clone project

``` shell
$ git clone git@github.com:lianxmfor/flink-hive.git
$ cd flink-hive && git lfs pull
```

2. start up docker compose:

``` shell
$ docker compose up -d
[+] Running 13/13
 ⠿ Network flink-hive_default                        Created
 ⠿ Volume "flink-hive_namenode"                      Created
 ⠿ Volume "flink-hive_datanode"                      Created
 ⠿ Container flink-hive_namenode_1                   Started
 ⠿ Container hive-server                             Started
 ⠿ Container flink-hive_presto-coordinator_1         Started
 ⠿ Container flink-hive_datanode_1                   Started
 ⠿ Container flink                                   Started
 ⠿ Container flink-hive_zookeeper_1                  Started
 ⠿ Container flink-hive_hive-metastore_1             Started
 ⠿ Container flink-hive_redis_1                      Started
 ⠿ Container flink-hive_hive-metastore-postgresql_1  Started
 ⠿ Container flink-hive_kafka_1                      Started
```

3. start in hive RETL, and create table

``` shell
$ docker exec -it hive-server /opt/hive/bin/beeline -u jdbc:hive2://localhost:10000
ls: cannot access /opt/hive/lib/hive-jdbc-*-standalone.jar: No such file or directory
Connecting to jdbc:hive2://localhost:10000
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/hive/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/hadoop-2.7.1/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Connected to: Apache Hive (version 2.1.0)
Driver: Hive JDBC (version 2.1.0)
21/08/24 08:18:32 [main]: WARN jdbc.HiveConnection: Request to set autoCommit to false; Hive does not support autoCommit=false.
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 2.1.0 by Apache Hive
0: jdbc:hive2://localhost:10000> 
0: jdbc:hive2://localhost:10000> create database testdb ;
No rows affected (2.048 seconds)
0: jdbc:hive2://localhost:10000> use testdb ;
No rows affected (0.106 seconds)
0: jdbc:hive2://localhost:10000> create table source (a bigint, b bigint) ;
No rows affected (1.026 seconds)
0: jdbc:hive2://localhost:10000> show tables ;
+-----------+--+
| tab_name  |
+-----------+--+
| source    |
+-----------+--+
1 row selected (0.877 seconds)
0: jdbc:hive2://localhost:10000> insert into source values (1, 1) ;
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. tez, spark) or using Hive 1.X releases.
No rows affected (7.498 seconds)
0: jdbc:hive2://localhost:10000> select a, b from source ;
+----+----+--+
| a  | b  |
+----+----+--+
| 1  | 1  |
+----+----+--+
1 row selected (0.238 seconds)
0: jdbc:hive2://localhost:10000> 
```

4. flink

``` shell
$ docker exec -ti flink bash
root@9a9985190163:/opt/flink# cp lib/sql-client-defaults.yaml conf/
root@9a9985190163:/opt/flink# cat conf/sql-client-defaults.yaml 
execution:
  planner: blink
  type: batch
  current-catalog: myhive
catalogs:
  - name: myhive
    type: hive
    hive-conf-dir: /opt/hive/conf
```

5. start up sql client

``` shell
root@9a9985190163:/opt/flink# start-cluster.sh 
Starting cluster.
Starting standalonesession daemon on host 9a9985190163.
Starting taskexecutor daemon on host 9a9985190163.
root@9a9985190163:/opt/flink
#root@9a9985190163:/opt/flink# sql-client.sh 
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/flink/lib/log4j-slf4j-impl-2.12.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/flink/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
No default environment specified.
Searching for '/opt/flink/conf/sql-client-defaults.yaml'...found.
Reading default environment from: file:/opt/flink/conf/sql-client-defaults.yaml
Command history file path: /root/.flink-sql-history
Flink SQL> show catalogs ;
+-----------------+
|    catalog name |
+-----------------+
| default_catalog |
|          myhive |
+-----------------+
2 rows in set

Flink SQL> use catalog myhive ;
[INFO] Execute statement succeed.

Flink SQL> show databases ;
+---------------+
| database name |
+---------------+
|       default |
|        testdb |
+---------------+
2 rows in set

Flink SQL> use testdb ;
[INFO] Execute statement succeed.

Flink SQL> show tables ;
+------------+
| table name |
+------------+
|     source |
+------------+
1 row in set

Flink SQL> SET sql-client.execution.result-mode=tableau;
[INFO] Session property has been set.

Flink SQL> select a, b from source ;
Empty set
```
