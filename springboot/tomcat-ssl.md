tomcat ssl
http://tomcat.apache.org/tomcat-7.0-doc/ssl-howto.html

通过如下命令 得到证书
keytool -genkey -alias springboot -keyalg RSA  -keystore


```
mac:tomcat lkl$ keytool -genkey -alias springboot -keyalg RSA  -keystore /Users/liaokailin/software/ca/tomcat/.keystore
输入密钥库口令:  
再次输入新口令: 
您的名字与姓氏是什么?
  [Unknown]:  liaokailin
您的组织单位名称是什么?
  [Unknown]:  en     
您的组织名称是什么?
  [Unknown]:  en
您所在的城市或区域名称是什么?
  [Unknown]:  hz
您所在的省/市/自治区名称是什么?
  [Unknown]:  zj
该单位的双字母国家/地区代码是什么?
  [Unknown]:  cn
CN=liaokailin, OU=en, O=en, L=hz, ST=zj, C=cn是否正确?
  [否]:  是

输入 <springboot> 的密钥口令
	(如果和密钥库口令相同, 按回车):  
再次输入新口令: 


```
keytool -selfcert -alias springboot -keystore .keystore

导出证书，可以提供给客户端安装 
keytool -export -alias springboot -keystore .keystore -storepass 123456 -rfc -file springboot.cer


在tomcat中屏蔽8080端口，开启8443端口

```
<Connector
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           port="8443" maxThreads="200"
           scheme="https" secure="true" SSLEnabled="true"
           keystoreFile="/Users/liaokailin/software/ca/tomcat/.keystore" keystorePass="123456"
           clientAuth="false" sslProtocol="TLS"/>
```
           
           



