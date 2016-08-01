#auto-dao


`auto-dao` is a simple, scalable database access framework。

##Quick Start

1.application.properties config datasouce ;

2.create test table , <a href='https://github.com/liaokailin/auto-dao/blob/master/src/test/resources/test.sql'>detail</a>;

3.run `Application.java` startup sample project;

4.visit <a href='http://localhost:8080/auto-dao/test'>http://localhost:8080/auto-dao/test</a> ;



##  Configuration

`auto-dao` dependent on spring framework

1. basic spring config  <a href='https://github.com/liaokailin/auto-dao/blob/master/src/main/java/com/enniu/cloud/common/po/config/ContextConfig.java'>detail</a> ;
 
 `DataSource` 、`JdbcTemplate` 、`DataSourceTransactionManager`


 
 2.auto-dao config
 	
 * The parse of entity-table-relation,default is <a href='https://github.com/liaokailin/auto-dao/blob/master/src/main/java/com/enniu/cloud/common/po/parse/HumpUnderLineParse.java'> HumpUnderLineParse</a> ;
 
 * The detail operator of db,default is <a href='https://github.com/liaokailin/auto-dao/blob/master/src/main/java/com/enniu/cloud/common/po/operator/MySqlJdbcTemplateOperator.java'>MySqlJdbcTemplateOperator</a> ;
 
 * The type of db,default is <a href='https://github.com/liaokailin/auto-dao/blob/master/src/main/java/com/enniu/cloud/common/po/mysql/Mysql.java'>Mysql</a> ;
  
`auto-dao `create auto-config by spring boot ;  <a href='https://github.com/liaokailin/auto-dao/blob/master/src/main/java/com/enniu/cloud/common/po/auto/POFactoryAutoConfig.java'>detail</a>

 

##Expand

default `auto-dao` support access mysql by wrapper JdbcTemplate ，entity-table-relation is  hump-underline(class CusUser ;table T_Cus_User) ;


Expand need to complete two steps：
1. implements `Operator`
2. implements `Parse`





 

