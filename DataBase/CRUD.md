# mysql
- 添加字段  
alter table 表名 add 字段名 numeric(1,0) comment '是否影响价格';
- 修改字段类型  
alter table 表名 modify column column_name  LONGTEXT;
- 添加默认值  
ater table 表名 alter column 字段名 drop default; (若本身存在默认值，则先删除)  
alter table 表名 alter column 字段名 set default 默认值;(若本身不存在则可以直接设定)

- 修改字段名  
ALTER TABLE 表名 CHANGE 字段名 新字段名 新字段类型;

DATE_TIME和TIMESTAMP的默认值不能是SYSDATE,需要改为CURRENT_TIMESTAMP.