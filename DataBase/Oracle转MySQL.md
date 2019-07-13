## 1. 配置里面切换数据源
 ```properties
#datasource
spring.datasource.url= jdbc:mysql://10.16.1.174:3306/OCC-ALL-0528?useUnicode=true&characterEncoding=utf-8
#JPA
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
spring.jpa.database=MySQL
```

## 2. Oracle特有语法修改
方案一：写两套sql，判断数据库类型来调用不同的方法。
```java
@Autowired
private  JpaProperties  jpaProperties;
if("ORACLE".equalsIgnoreCase(jpaProperties.getDatabase().name())){
	oriFactorId = super.repository.findFactorIdById(modifiedList.get(i).getId());
}else{
	oriFactorId = super.repository.findFactorIdByIdForMySQL(modifiedList.get(i).getId());
}
```
使用Java代码实现。递归查询，start with..connect by 

方案二：改为JPA语句，或者使用标准的sql语法。
- 日期相关  SYSDATE关键字，TO_DATE函数（mysql为str_to_date）。  
  可以将日期作为方法入参传入，使用Java来获取日期（new Date()）。
- nvl 函数。 mysql不支持，使用coalesce函数替换。
- 连接符 ||  ，mysql的含义为“或”。  
mysql 不支持，如果只是两个字符进行拼接可以改为concat函数， concat(a,b)。  
oracle的concat函数只支持两个参数，mysql可以支持多个。可以使用嵌套使用concat。  
select concat(concat(id,code),name) from base_goods;  
标准JPQL语法中concat函数是支持多个参数的。
- where条件中 in 后面跟单个字符串，Oracle支持，mysql不支持。统一改为 “=”。
 ```java
 @Query(value="select XXXX  from XXX where PARENT_GOODS_ID in ?1" ,nativeQuery = true)
 List<GoodsBomChild> getGoodsBomChildByParentGoodIds(String parentGoodId, String bomId);
```
- 日期直接加减的含义不同，oracle代表+1天，mysql结果为时间的数字值+1。mysql可以使用select sysdate() + interval 1 day。  
    > select sysdate  ....;  2019-06-26 15:50:14  
  select sysdate + 1 ...；2019-06-27 15:50:14

    > select sysdate() ;  2019-06-26 15:50:14  
    select sysdate() + 1 ；20190626155015

- current_date 含义不同，oracle中没有current_time关键字。
    oracle中的current_date（作用同sysdate），相当于mysql中的current_date + current_time。
- mysql中每个派生出来的表都需要别名。
    select field1 from (select sysdate() as field1 from table1)  aa
- mysql下update时不能有本身的子查询
```sql
update base_goods  set dr = 0  where id in 
(select distinct b.id from base_goods b where b.dr=1)
--------------------------------------------------------
update base_goods set dr = 0 where id in
(
  select a.id from(
    select distinct b.id from base_goods b where b.dr=1
  ) a
)
```
## 3. 运维相关
- Mysql在V5.1之前默认存储引擎是MyISAM；在此之后默认存储引擎是InnoDB。MyISAM是不支持事务的。现在生产环境一般使用v5.7.x版本。
- 表名，字段名不要加双引号
oracle表和字段是有大小写的区别。oracle默认是大写，如果我们用双引号括起来的就区分大小写，如果没有，系统会自动转成大写。mysql可以使用show Variables like '%table_names'，查看lower_case_table_names的值，0代表区分，1代表不区分，一般设置为不区分（默认不区分）。mysql不支持双引号。
- TO_DATE
- varcahr2  ， varchar
- number   ,  numeric 或 decimal
- varchar建索引遇到长度太长的问题。  
mysql中utf8字符占3个字节，utf8mb4占4个字节，GBK为2个字节。  
InnoDB存储引擎的表索引的前缀长度最长是767字节(bytes)，如果需要建索引，就不能超过 767 bytes。	
utf8编码时，最大长度为：767/3 = 255。  
utf8mb4编码，最大长度为： 767/4 = 191。  
GBK是双字节，最大长度为：768/2=384。  