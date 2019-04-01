+++
date = 2019-04-01T10:55:56+08:00
title = "oracle 记录"
description = ""
slug = "" 
tags = ["oracle", "oracle 临时表空间", "oracle表空间", "oracle 导入导出", "oracle 常用操作"]
categories = []
externalLink = ""
series = []
+++


### 01. dmp文件导入步骤
01) 用管理员登录:
```
# 方法一
system as sysdba
# 方法二
sys as sysdba
```
02) 创建用户
```
CREATE USER <USERNAME> IDENTIFIED BY <PASSWORD>;
```
03) 赋权限给新用户
```
GRANT CREATE USER, DROP USER, ALTER USER, CREATE ANY VIEW, DROP ANY VIEW, EXP_FULL_DATABASE, IMP_FULL_DATABASE, DBA, CONNECT, RESOURCE, CREATE SESSION TO <用户名>;
```
04) 打开cmd窗口
```
IMP <USERNAME>/<PASSWORD>@<SID> FILE=D:\\XXX.DMP FULL=Y;
```
05) 导出命令:
```
# 导出某些指定的表:
exp <username>/<password>@<SID> file=d:\\xxx.dmp tables=(table1,table2,table3)
# 导出某个用户的表:
exp <username>/<password>@<SID> file=d:\\xxx.dmp owner=<username>
```





### 02. 常用操作
01) 切换用户
```
# 方式一
CONNECT SYS/123456 AS SYSDBA;
# 方式二
CONN SCOTT/TIGER;
```
02) 退出sqlplus:
```
EXIT;
```
03) 免登录进入 sqlplus:
```
# 方式一
sqlplus /nolog
# 方式二
sqlplus / as sysdba
```
04) 查询表信息:
```
DESC <TABLENAME>;
```
05) 表结构修改字段:
```
ALTER TABLE <TABLE_NAME> MODIFY (<FIELD_NAME> <FIELD_TYPE>);
```
06) 表结构增加字段:
```
ALTER TABLE <TABLE_NAME> ADD <FIELD_NAME> <FIELD_TYPE>;
```
07) 表结构增加注释:
```
COMMENT ON COLUMN <TABLE_NAME>.<FIELD_NAME> IS '<COMMENT_CONTENT>';
```
08) 显示当前用户:
```
SHOW USER;
```
09) 解锁用户:
```
ALTER USER <USERNAME> ACCOUNT UNLOCK;
```
10) 修改用户密码:
```
ALTER USER <USERNAME> IDENTIFIED BY <NEW_PASSWORD>;
```
11) 显示当前数据库实例:
```
# 方式一
SHOW PARAMETER INTANCE;
# 方式二
SELECT INSTANCE_NAME FROM V$INSTANCE;
```
12) sql语句case when 的用法:

```
case when <condition> then <what to do when condition is true> else <what to do when condition is false> end
```
13) 表结构字段名称修改:
```
ALTER TABLE <TABLE_NAME> RENAME COLUMN <OLD_FIELD_NAME> TO <NEW_FIELD_NAME>;
```

### 03.GRANT 赋于权限
常用的系统权限集合有以下三个:  
CONNECT(基本的连接), RESOURCE(程序开发), DBA(数据库管理)  
常用的数据对象权限有以下五个:  
ALL ON 数据对象名, SELECT ON 数据对象名, UPDATE ON 数据对象名, DELETE ON 数据对象名, INSERT ON 数据对象名, ALTER ON 数据对象名
```
GRANT CONNECT, RESOURCE TO <TABLE_NAME>;
GRANT SELECT ON <TABLE_NAME> TO <USER_NAME>;
GRANT SELECT, INSERT, DELETE ON <TABLE_NAME> TO <USER_NAME>, <USER_NAME>;
```

### 04.REVOKE 回收权限
```
REVOKE CONNECT, RESOURCE FROM <USER_NAME>;
REVOKE SELECT ON <TABLE_NAME> FROM <USER_NAME>;
REVOKE SELECT, INSERT, DELETE ON <TABLE_NAME> FROM <USER_NAME>, <USER_NAME>;
```

### 05. 重建并修改默认临时表空间办法:
01) 查询当前数据库默认临时表空间名和查询临时表空间数据文件
```
select * from database_properties where property_name='DEFAULT_TEMP_TABLESPACE';
select tablespace_name,file_name,bytes/1024/1024 file_size,autoextensible from dba_temp_files;
```
02) 创建新的临时表空间
```
create temporary tablespace temp02 tempfile 'E:\oracle\oradata\lims\TEMP02.DBF' size 1024M autoextend on;
```
03) 修改默认表空间为刚刚建立的临时表空间
```
alter database default temporary tablespace temp02;
```
04) 查看用户所用临时表空间的情况
```
SELECT USERNAME,TEMPORARY_TABLESPACE FROM DBA_USERS;
```
05) 删除原来的临时表空间
```
drop tablespace temp including contents and datafiles;
```
06) 查看所有表空间名确认临时表空间是否已删除
```
select tablespace_name from dba_tablespaces;
```

### 06. 临时表空间相关处理
01) 查询默认临时表空间
```
select * from database_properties where property_name='DEFAULT_TEMP_TABLESPACE';
```
02) 查询临时表空间数据文件状态:
```
select tablespace_name,file_name,bytes/1024/1024 file_size,autoextensible from dba_temp_files;
```
03) 创建新的临时表空间
```
create temporary tablespace temp_01 tempfile '/u01/app/oracle/oradata/XE/temp_01.dbf' size 1024M autoextend on;
```
04) 修改默认表空间为新的表空间
```
alter database default temporary tablespace temp_01;
```
05) 修改临时表空间大小
```
alter database tempfile '/u01/app/oracle/oradata/XE/temp.dbf' resize 512m;
```
06) 设置临时表空间自动增长
```
alter database tempfile '/u01/app/oracle/oradata/XE/temp.dbf' autoextend on next 8m maxsize unlimited;
```
07) 查看临时表空间使用情况:
```
SELECT tbs 表空间名, SUM(totalM) 总共大小M, SUM(usedM) 已使用空间M, SUM(remainedM) 剩余空间M, SUM(usedM) / SUM(totalM) * 100 已使用百分比, SUM(remainedM) / SUM(totalM) * 100 剩余百分比
FROM (
  SELECT tb.file_id ID, tb.tablespace_name tbs, tb.file_name NAME, tb.bytes / 1024 / 1024 totalM, (tb.bytes - SUM(NVL(ta.free_space, 0))) / 1024 / 1024 usedM,
    SUM(NVL(ta.free_space, 0) / 1024 / 1024) remainedM, SUM(NVL(ta.free_space, 0) /(tb.bytes) * 100), (100 - (SUM(NVL(ta.free_space, 0)) /(tb.bytes) * 100))
  FROM dba_temp_free_space ta, dba_temp_files tb
  WHERE ta.tablespace_name = tb.tablespace_name
  GROUP BY tb.tablespace_name, tb.file_name, tb.file_id, tb.bytes
  ORDER BY tb.tablespace_name
) GROUP BY tbs;
```

### 07. 表空间相关处理
01) 扩展表空间
```
alter database datafile '/u01/app/oracle/oradata/XE/system.dbf' resize 1024m;
```
02) 表空间自动增长
```
alter system datafile '/u01/app/oracle/oradata/XE/system_01.dbf' autoextend on next 50m maxsize 500m;
```
03) 新增表空间数据文件
```
alter tablespace system add datafile '/u01/app/oracle/oradata/XE/system_01.dbf' size 1024m;
```
04) 查询表空间的所有数据文件
```
select * from dba_data_files where tablespace_name='SYSTEM';
```
05) 查询表属于哪个表空间
```
select tablespace_name,table_name from user_talbes where table_name='employ';
```
06) 查看表空间使用情况
```
SELECT tbs 表空间名, SUM(totalM) 总共大小M, SUM(usedM) 已使用空间M, SUM(remainedM) 剩余空间M, SUM(usedM) / SUM(totalM) * 100 已使用百分比, SUM(remainedM) / SUM(totalM) * 100 剩余百分比
FROM (
  SELECT tb.file_id ID, tb.tablespace_name tbs, tb.file_name NAME, tb.bytes / 1024 / 1024 totalM, (tb.bytes - SUM(NVL(ta.bytes, 0))) / 1024 / 1024 usedM,
    SUM(NVL(ta.bytes, 0) / 1024 / 1024) remainedM, SUM(NVL(ta.bytes, 0) /(tb.bytes) * 100), (100 - (SUM(NVL(ta.bytes, 0)) /(tb.bytes) * 100))
  FROM dba_free_space ta, dba_data_files tb
  WHERE ta.file_id = tb.file_id
  GROUP BY tb.tablespace_name, tb.file_name, tb.file_id, tb.bytes
  ORDER BY tb.tablespace_name
) GROUP BY tbs;
```

### 00. 其它备忘
01) sql语句和sql命令虽然大小写都可以, 但是建议大写, 大写执行速度快, 同时有些命令只能大写!  
02) sql语句和sql命令必须用分号结尾, cmd中的命令不用分号!  
03) sqlplus 登录密码区分大小写
04) 查询 DIRECTORY 操作需要的权限
```
SELECT DISTINCT PRIVILEGE FROM DBA_SYS_PRIVS WHERE PRIVILEGE LIKE '%DIRECTORY%';
```
05) 查询当前设定的目录配置
```
SELECT * FROM DBA_DIRECTORIES;
```
06) 赋予SCOTT用户删除和创建目录配置的权限
```
GRANT DROP ANY DIRECTORY, CREATE ANY DIRECTORY TO SCOTT;
```
07) 回收SCOTT用户删除和创建目录配置的权限
```
REVOKE DROP ANY DIRECTORY, CREATE ANY DIRECTORY FROM SCOTT;
```
08) 创建某个目录配置
```
CREATE OR REPLACE DIRECTORY <dir_name> AS <dir_path>;
```
09) 删除某个目录配置
```
DROP DIRECTORY <dir_name>;
```
10) 赋予和回收某个目录的读写权限:
```
GRANT READ ON DIRECTORY <dir_name> TO SCOTT;
GRANT WRITE ON DIRECTORY <dir_name> TO SCOTT;
REVOKE READ ON DIRECTORY <dir_name> FROM SCOTT;
REVOKE WRITE ON DIRECTORY <dir_name> FROM SCOTT;
```
11) 查询当前schma下的所有表: SELECT T.* FROM USER_TABLES T;  
12) 查出所有表和对应schma以及表的字段
```
SELECT DISTINCT DCC.OWNER AS "用户", DCC.TABLE_NAME AS "表名", DCC.COLUMN_NAME AS "字段名", DTC.DATA_TYPE AS "数据类型", DCC.COMMENTS AS "字段描述",
       (CASE WHEN CONSTS.COLUMN_NAME IS NOT NULL THEN '是' ELSE '' END) AS "是否主键"
FROM DBA_TAB_COLS DTC
LEFT JOIN DBA_COL_COMMENTS DCC ON (DTC.COLUMN_NAME = DCC.COLUMN_NAME AND DTC.TABLE_NAME = DCC.TABLE_NAME AND DTC.OWNER = DCC.OWNER)
LEFT JOIN DBA_TAB_COMMENTS UTC ON (DTC. OWNER = UTC. OWNER AND DTC.TABLE_NAME = UTC.TABLE_NAME)
LEFT JOIN (SELECT CU.* FROM DBA_CONS_COLUMNS CU, DBA_CONSTRAINTS AU WHERE CU.CONSTRAINT_NAME = AU.CONSTRAINT_NAME AND AU.CONSTRAINT_TYPE = 'P' ) CONSTS
  ON (DTC.OWNER = CONSTS.OWNER AND DTC.TABLE_NAME = CONSTS.TABLE_NAME AND DTC.COLUMN_NAME = CONSTS.COLUMN_NAME)
WHERE DTC.OWNER IN ('BOMMGMT', 'CFGMGMT', 'CHGMGMT', 'CORE', 'CUST', 'MSTDATA', 'INTEGRATION')
ORDER BY DCC.OWNER, DCC.TABLE_NAME;
```
13) 数据库死锁查询
```
select s.username,l.object_id, l.session_id,s.serial#, s.lockwait,s.status,s.machine,s.program from v$session s,v$locked_object l where s.sid = l.session_id;
select sql_text from v$sql where hash_value in (select sql_hash_value from v$session where sid in  (select session_id from v$locked_object));
select distinct  'alter system kill session ''' ||sid||','||serial#||''''||';' as "kill sql" from V$locked_object t1 left join V$session t2 on t1.session_id = t2.sid;
```
14) 约束查询: SELECT UC.TABLE_NAME, UC.CONSTRAINT_NAME, UC.* FROM USER_CONSTRAINTS UC;  
15) 查询所有列名: SELECT * FROM ALL_TAB_COLS WHERE TABLE_NAME = 'XXX';  
16) 查找alert日志, 需要通过 sqlplus / as sysdba 登陆, 然后输入: show parameter dump;  

### 附件
<a href="/files/oracle_readme/oracle_readme.txt" target="_blank">oracle记录txt版本</a>








