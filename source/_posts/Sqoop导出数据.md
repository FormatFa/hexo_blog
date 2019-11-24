---
title: Sqoop导出数据
date: 2019-11-22 14:50:20
tags:
- sqoop
- 大数据
typora-root-url: Sqoop导出数据
---

Sqoop - > sql hadoop

### Hive 导出到 MySQL 

测试数据apps.csv ,上传到hdfs中的/sqoop/apps.csv下

```

mt,10000
kpa,100
qq,30
twitter,40


[root@bigdata blog]# hdfs dfs -cat /sqoop/apps.csv
mt,10000
kpa,100
qq,30
twitter,40
```

hive中创建测试数据表,指定数据为上面数据的路径 (hive->mysql其实是hdfs->mysql,下面这步其实可以省略)

```
create table if not exists  apps(name string,download int) row format delimited  fields terminated by ',' lines terminated by '\n' location '/sqoop/apps.csv';

hive> select * from apps;
OK
mt      10000
kpa     100
qq      30
twitter 40
Time taken: 0.054 seconds, Fetched: 4 row(s)
```

在mysql中创建对应的表

```
create table apps(name varchar(100),download int);
```

sqoop 导出语句

- 设置导出的数据路径
- 设置导出到mysql的jdbc连接,账号密码

至少需要指定**--connect**,**--export-dir**, **--table**三个参数

```
sqoop-export --connect 'jdbc:mysql://localhost:3306/train' --export-dir "/sqoop/apps.csv" --table apps --username 'root' --password 'root'  
```

### sqoop 用法帮助

`usage: sqoop export [GENERIC-ARGS] [TOOL-ARGS]`

#### 通用参数

- `--connect <jdbc-uri>`

  指定jdbc的连接,关系型数据库的连接

- `--connection-manager <class-name> `

  Specify connection manager

- `--connection-param-file <properties-file>  `

  connection参数的文件

- `  --driver <class-name>`

  指定jdbc驱动的类

- `--hadoop-home <hdir>`

  override $HADOOP_MAPRED_HOME_ARG

- `--hadoop-mapred-home <dir> `

  override $HADOOP_MAPRED_HOME_ARG

- `--help`

  输出用法

- `--metadata-transaction-isolation-level <isolationlevel>`

   transactionisolation level for metadata queries. For more details check ava.sql.Connection javadoc or the JDBC

- `-oracle-escaping-disabled <boolean>`

  Disable the  escaping mechanism of the Oracle/OraOop connection managers

- `-P` 

  从命令行里读取密码

- `--password <password>`

  设置密码

- `--password-alias <password-alias>`

  Credential provider password alias

- `--relaxed-isolation`

  Use read-uncommitted  isolation for imports

```

                                           
   --skip-dist-cache                                          Skip copying
                                                              jars to
                                                              distributed
                                                              cache
   --temporary-rootdir <rootdir>                              Defines the
                                                              temporary
                                                              root
                                                              directory
                                                              for the
                                                              import
   --throw-on-error                                           Rethrow a
                                                              RuntimeExcep
                                                              tion on
                                                              error
                                                              occurred
                                                              during the
                                                              job
   --username <username>                                      Set
                                                              authenticati
                                                              on username
   --verbose                                                  Print more
                                                              information
                                                              while
                                                              working

Export control arguments:
   --batch                                                    Indicates
                                                              underlying
                                                              statements
                                                              to be
                                                              executed in
                                                              batch mode
   --call <arg>                                               Populate the
                                                              table using
                                                              this stored
                                                              procedure
                                                              (one call
                                                              per row)
   --clear-staging-table                                      Indicates
                                                              that any
                                                              data in
                                                              staging
                                                              table can be
                                                              deleted
   --columns <col,col,col...>                                 Columns to
                                                              export to
                                                              table
   --direct                                                   Use direct
                                                              export fast
                                                              path
   --export-dir <dir>                                         HDFS source
                                                              path for the
                                                              export
-m,--num-mappers <n>                                          Use 'n' map
                                                              tasks to
                                                              export in
                                                              parallel
   --mapreduce-job-name <name>                                Set name for
                                                              generated
                                                              mapreduce
                                                              job
   --staging-table <table-name>                               Intermediate
                                                              staging
                                                              table
   --table <table-name>                                       Table to
                                                              populate
   --update-key <key>                                         Update
                                                              records by
                                                              specified
                                                              key column
   --update-mode <mode>                                       Specifies
                                                              how updates
                                                              are
                                                              performed
                                                              when new
                                                              rows are
                                                              found with
                                                              non-matching
                                                              keys in
                                                              database
   --validate                                                 Validate the
                                                              copy using
                                                              the
                                                              configured
                                                              validator
   --validation-failurehandler <validation-failurehandler>    Fully
                                                              qualified
                                                              class name
                                                              for
                                                              ValidationFa
                                                              ilureHandler
   --validation-threshold <validation-threshold>              Fully
                                                              qualified
                                                              class name
                                                              for
                                                              ValidationTh
                                                              reshold
   --validator <validator>                                    Fully
                                                              qualified
                                                              class name
                                                              for the
                                                              Validator

Input parsing arguments:
   --input-enclosed-by <char>               Sets a required field encloser
   --input-escaped-by <char>                Sets the input escape
                                            character
   --input-fields-terminated-by <char>      Sets the input field separator
   --input-lines-terminated-by <char>       Sets the input end-of-line
                                            char
   --input-optionally-enclosed-by <char>    Sets a field enclosing
                                            character

Output line formatting arguments:
   --enclosed-by <char>               Sets a required field enclosing
                                      character
   --escaped-by <char>                Sets the escape character
   --fields-terminated-by <char>      Sets the field separator character
   --lines-terminated-by <char>       Sets the end-of-line character
   --mysql-delimiters                 Uses MySQL's default delimiter set:
                                      fields: ,  lines: \n  escaped-by: \
                                      optionally-enclosed-by: '
   --optionally-enclosed-by <char>    Sets a field enclosing character

Code generation arguments:
   --bindir <dir>                             Output directory for
                                              compiled objects
   --class-name <name>                        Sets the generated class
                                              name. This overrides
                                              --package-name. When
                                              combined with --jar-file,
                                              sets the input class.
   --escape-mapping-column-names <boolean>    Disable special characters
                                              escaping in column names
   --input-null-non-string <null-str>         Input null non-string
                                              representation
   --input-null-string <null-str>             Input null string
                                              representation
   --jar-file <file>                          Disable code generation; use
                                              specified jar
   --map-column-java <arg>                    Override mapping for
                                              specific columns to java
                                              types
   --null-non-string <null-str>               Null non-string
                                              representation
   --null-string <null-str>                   Null string representation
   --outdir <dir>                             Output directory for
                                              generated code
   --package-name <name>                      Put auto-generated classes
                                              in this package

HCatalog arguments:
   --hcatalog-database <arg>                        HCatalog database name
   --hcatalog-home <hdir>                           Override $HCAT_HOME
   --hcatalog-partition-keys <partition-key>        Sets the partition
                                                    keys to use when
                                                    importing to hive
   --hcatalog-partition-values <partition-value>    Sets the partition
                                                    values to use when
                                                    importing to hive
   --hcatalog-table <arg>                           HCatalog table name
   --hive-home <dir>                                Override $HIVE_HOME
   --hive-partition-key <partition-key>             Sets the partition key
                                                    to use when importing
                                                    to hive
   --hive-partition-value <partition-value>         Sets the partition
                                                    value to use when
                                                    importing to hive
   --map-column-hive <arg>                          Override mapping for
                                                    specific column to
                                                    hive types.

Generic Hadoop command-line arguments:
(must preceed any tool-specific arguments)
Generic options supported are
-conf <configuration file>     specify an application configuration file
-D <property=value>            use value for given property
-fs <local|namenode:port>      specify a namenode
-jt <local|resourcemanager:port>    specify a ResourceManager
-files <comma separated list of files>    specify comma separated files to be co                                                                                                             pied to the map reduce cluster
-libjars <comma separated list of jars>    specify comma separated jar files to                                                                                                              include in the classpath.
-archives <comma separated list of archives>    specify comma separated archives                                                                                                              to be unarchived on the compute machines.

The general command line syntax is
bin/hadoop command [genericOptions] [commandOptions]


At minimum, you must specify --connect, --export-dir, and --table

```



