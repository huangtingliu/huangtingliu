---
layout: default
title: 使用mysqldump进行mysql数据库备份
tags: [数据库]
imgPath: 2018-08-10-MYSQL-BACKUP
---

几个月没写文章了，最近有个mysql数据库备份的任务，整理了一份教程。  

在软件开发过程中，保障数据的安全一定是最重要的部分之一了。而数据都是保存在数据库中，如何合理的备份数据库也是至关重要。


# 一、几种备份方案

简单介绍下几种备份的方案

### 按是否影响数据库
- 热备份：备份时不影响数据库读写
- 温备份：备份时，数据库可读，但不可写
- 冷备份：备份时，数据库不可读写

### 按数据量
- 完全备份
- 增量备份

### 按备份工具
- 直接拷贝数据库的数据文件：物理备份，属于冷备份，速度较快。适合数据量比较小的场景
- 使用mysqldump命令备份：逻辑备份，InnoDB存储引擎支持热备份，MyISAM存储引擎只能温备份。适合数据量还行的场景，先完全备份，再定期bin-log增量备份
- lvm2 snapshot：几乎热备份，借助文件系统管理工具。适合数据量一般，不过分影响业务运行，先完全备份，再定期bin-log增量备份
- mysqlhotcopy：几乎热备份，仅支持MyISAM存储引擎
- xtrabackup：强大的备份工具，热备份，支持完全备份和增量备份，适合数据量大，不过分影响业务运行，先使用xtrabackup完全备份，再定期xtrabackup增量备份


# 二、使用mysqldump进行备份

使用原理：假如，我们会每周执行一次全量备份，然后根据log-bin日志文件进行每天增量备份，定期操作可以使用window的任务计划执行我们的bat文件

## 1. 首先开启日志功能、执行全量备份的命令，会保存备份sql文件与新旧日志文件
***注意：如果mysql服务启动不起来可能是log-bin设置的目录（截图中的H:/mysqlbak/log-bin/）不存在***

![]({{ site.url }}{{site.post-img-dir}}/{{page.imgPath}}/log-bin-add.png)

## 2. 执行完全备份命令（我们用脚本写成bat文件，将下面的完全备份命令写入其中，windows任务计划每周执行该文件，bat文件请看文章结尾）
```
mysqldump --no-defaults -hlocalhost -uroot -proot --single-transaction --flush-logs --master-data=2 -R -E --databases test > H:\mysqlbak\BAK_ALL_test.sql
```
其中  
--no-defaults： 代表不使用默认配置  
-hlocalhost： mysql服务ip地址  
-uroot： mysql账号  
-proot： mysql密码  
--databases： test 代表备份的数据库为test  
--single-transaction： 只使用于INNODB存储引擎，  
--flush-logs： 代表结束当前日志，生成新的日志文件  
-R： 代表备份存储过程和函数  
-E： 代表备份事件
--master-data=2： 代表在输出的备份sql文件中会记录下新日志的文件名称和位置，用于日后恢复，在sql文件中会以注释的方式记录该内容如：  
```  
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=120; 
```

***注意：如果没有在mysql的安装目录下的my.ini文件设置log-bin参数开启日志功能的话则会报错***  
``` 
mysqldump: Error: Binlogging on server not active 
```
## 3. mysql自动日志，用于增量备份

因为我们在mysql的安装目录下的my.ini文件设置了log-bin参数，即设置了日志保存路径，执行完全备份时的--flush-logs参数会生成新的日志文件，所以在该路径上会出现mysql-bin.000001、mysql-bin.000002、mysql-bin.index..等日志文件
![]({{ site.url }}{{site.post-img-dir}}/{{page.imgPath}}/mysql-bin.png)  
mysql-bin.000001为旧的日志文件
mysql-bin.000002为新的日志文件
后面的mysql所有操作都会实时记录到mysql-bin.000002文件中，直到下次我们再进行备份为止

# 三、然后我们进行恢复（假如哪天mysql数据库数据丢了）
## 1.关闭日志功能，重启mysql
***注意：因为备份的语句也会记录到日志，利用日志恢复的语句也会记录到日志、并且日志恢复语句还执行不成功，所以备份前先关闭日志功能
打开mysql的安装目录下的my.ini文件找到log-bin参数，加上#号注释掉 log-bin=H:/mysqlbak/log-bin/mysql-bin*** 

![]({{ site.url }}{{site.post-img-dir}}/{{page.imgPath}}/log-bin-annotation.png)  

## 2.先恢复完成备份
***注意：需要设置mysql安装目录下的bin目录为环境变量
此时的备份sql也会记录到日志文件中，所以最好应该先关闭日志，不然的话后面增量恢复又重新执行了一次全量备份的sql***
```
mysql.exe -hlocalhost -uroot -proot --default-character-set=utf8 < H:\mysqlbak\BAK_ALL_test.sql
```

## 3.恢复增量备份
利用日志文件恢复增量备份（mysqlbinlog --no-defaults 日志文件地址 | mysql  -uroot -proot db_name）,只需要备份最后一个日志文件即可 


```
mysqlbinlog --no-defaults H:\mysqlbak\log-bin\mysql-bin.000002 | mysql  -uroot -proot test
```

可以查看日志内容
```
mysqlbinlog --no-defaults H:\mysqlbak\log-bin\mysql-bin.000002
```
可以导出为sql查看
```
mysqlbinlog --no-defaults H:\mysqlbak\log-bin\mysql-bin.000002 -r test.sql
```

# 结束  
操作到这里就完成了，下面的内容可用可不用

# 以下是bat文件的操作流程，附bat文件脚本内容  
0. 修改备份huaantest_all_bak.bat文件（忽略则默认使用该配置）  
--修改备份文件保存目录 set BACKUP_PATH=H:\mysqlbak\  
--修改mysql bin目录：找到set MYSQL=D:\dev\mysql\mysql\MySQL Server 5.6\bin\ 修改为自己的目录  
--修改数据库 set DATABASES=huaantest 为自己的数据库  
--修改数据库 set HOST=192.168.1.137  
--修改用户名 set USERNAME=root  
--修改密码 set PASSWORD=123456  
1. 设置环境变量
--设置mysql安装目录下的bin目录为环境变量  
2. 配置Mysql日志文件以便后期增量备份还原  
--创建目录H:/mysqlbak/log-bin，修改mysql安装目录下my.ini文件添加log-bin=H:/mysqlbak/log-bin/mysql-bin  
（详细：https://dev.mysql.com/doc/refman/5.6/en/replication-options-binary-log.html#option_mysqld_log-bin）  
--重启mysql  
3. 执行备份（保存在“H:\mysqlbak\”目录下，可修改，会生成备份sql文件）
--huaantest_all_bak.bat文件为执行备份文件，可手动双击启动备份，也可以设置windows定时计划定期执行  
--文件内可配置备份数据库名，mysql地址账号密码以及备份的保存路径
5. 还原执行
--注释mysql安装目录下my.ini文件添加log-bin=H:/mysqlbak/log-bin/mysql-bin,重启mysql  
--完全备份还原命令：mysql.exe -hlocalhost -uroot -p123456 --default-character-set=utf8 < H:\mysqlbak\xxxx.sql  
--执行增量还原命令：mysqlbinlog --no-defaults H:\mysqlbak\log-bin\mysql-bin.xxxxx | mysql  -uroot -p123456 huaantest  


# bat文件内容
```
@echo off & setlocal ENABLEEXTENSIONS

:: ---------- 配置项 ----------

:: 备份放置的路径，加 \
set BACKUP_PATH=H:\mysqlbak\

:: 要备份的数据库名称，多个用空格分隔
set DATABASES=test

:: MySQL IP地址
set HOST=localhost

:: MySQL 用户名
set USERNAME=root

:: MySQL 密码
set PASSWORD=root

:: MySQL Bin 目录，加 \
:: 如果可以直接使用 mysqldump（安装时添加 MySQL Bin 目录到了环境变量），此处留空即可
set MYSQL=D:\dev\mysql\mysql\MySQL Server 5.6\bin\

:: WinRAR 自带命令行工具的可执行文件路径，长文件名注意用 Dos 长文件名书写方式
set WINRAR=C:\Progra~1\WinRAR\Rar.exe

:: ---------- 以下请勿修改 ----------

set YEAR=%date:~0,4%
set MONTH=%date:~5,2%
set DAY=%date:~8,2%
:: 如果在 dos 下输入 time 返回的不是 24 小时制（没有 0 填充），请自行修改此处
set HOUR=%time:~0,2%
set MINUTE=%time:~3,2%
set SECOND=%time:~6,2%

:: set DIR=%BACKUP_PATH%%YEAR%\%MONTH%\%DAY%\
set DIR=%BACKUP_PATH%%YEAR%\%MONTH%\%DAY%\
set ADDON=%YEAR%%MONTH%%DAY%%HOUR%%MINUTE%%SECOND%

:: create dir
if not exist %DIR% (
mkdir %DIR% 2>nul
)
if not exist %DIR% (
echo Backup path: %DIR% not exists, create dir failed.
goto exit
)

:: backup
echo Start dump databases...
for %%D in (%DATABASES%) do (
echo Dumping database %%D ...
:: 备份包括存储过程和函数和事件，-R存储过程和函数  -E事件
mysqldump --no-defaults -h%HOST% -u%USERNAME% -p%PASSWORD% --single-transaction --flush-logs --master-data=2 -R -E --databases %%D > %DIR%BAK_ALL_%%D_%ADDON%.sql
:: winrar
if exist %WINRAR% (
%WINRAR% a -k -r -s -m1 -ep1 %%D.%ADDON%.rar %%D.%ADDON%.sql
del /F /S /Q %%D.%ADDON%.sql 2>nul
)
)
echo Done

:exit
```