#spring boot实战(第七篇)内嵌容器tomcat配置

##默认容器

spring boot默认web程序启用tomcat内嵌容器tomcat，监听8080端口,servletPath默认为 `/` 通过需要用到的就是端口、上下文路径的修改，在spring boot中其修改方法及其简单；
	
	在资源文件中配置：	
	server.port=9090 
	server.contextPath=/lkl

启动spring boot 

```
2015-10-04 00:06:55.768  INFO 609 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2015-10-04 00:06:55.844  INFO 609 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2015-10-04 00:06:55.928  INFO 609 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 9090 (http)
2015-10-04 00:06:55.930  INFO 609 --- [           main] com.lkl.springboot.Application       : Started Application in 3.906 seconds (JVM running for 4.184)
```

可以看出其监听端口9090，执行
<a>http://localhost:9090/lkl/springboot/liaokailin</a> 成功访问



##自定义tomcat

在实际的项目中简单的配置tomcat端口肯定无法满足大家的需求，因此需要自定义tomcat配置信息来灵活的控制tomcat。

###以定义默认编码为例
	
```
package com.lkl.springboot.container.tomcat;

import org.springframework.boot.context.embedded.EmbeddedServletContainerFactory;
import org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * tomcat 配置
 * @author liaokailin
 * @version $Id: TomcatConfig.java, v 0.1 2015年10月4日 上午12:11:47 liaokailin Exp $
 */
@Configuration
public class TomcatConfig {

    @Bean
    public EmbeddedServletContainerFactory servletContainer() {
        TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory();
        tomcat.setUriEncoding("UTF-8");
        return tomcat;
    }

}

```

构建`EmbeddedServletContainerFactory`的bean，获取到`TomcatEmbeddedServletContainerFactory`实例以后可以对tomcat进行设置，例如这里设置编码为`UTF-8`


##SSL配置

####生成证书
	keytool -genkey -alias springboot -keyalg RSA -keystore /Users/liaokailin/software/ca1/keystore
	设置密码123456
![生成证书](http://my.csdn.net/my/album/detail/1814999)

####tomcat中验证证书是否正确

修改tomcat/conf/server.xml文件
	
```
<Connector
              protocol="org.apache.coyote.http11.Http11NioProtocol"
              port="8443" maxThreads="200"
              scheme="https" secure="true" SSLEnabled="true"
              keystoreFile="/Users/liaokailin/software/ca1/keystore" keystorePass="123456"
              clientAuth="false" sslProtocol="TLS"/>
              
```

启动tomcat ，访问  <a>http://localhost:8443</a>

![https访问](http://my.csdn.net/my/album/detail/1815001)


##spring boot 内嵌tomcat ssl

配置资源文件

```
server.port=8443
server.ssl.enabled=true
server.ssl.keyAlias=springboot
server.ssl.keyPassword=123456
server.ssl.keyStore=/Users/liaokailin/software/ca1/keystore
```

* `server.ssl.enabled ` 启动tomcat ssl配置
* `server.ssl.keyAlias` 别名
* `server.ssl.keyPassword` 密码
* `server.ssl.keyStore` 位置

启动 `spring boot`

![启动spring boot](http://my.csdn.net/my/album/detail/1815003)

访问<a>https://localhost:8443/springboot/helloworld</a>

![访问https](http://my.csdn.net/my/album/detail/1815007)


##多端口监听配置

前面启动ssl后只能走https,不能通过http进行访问，如果要监听多端口，可采用编码形式实现。

1.注销前面ssl配置，设置配置 `server.port=9090 `

2.修改`TomcatConfig.java`

```

package com.lkl.springboot.container.tomcat;

import java.io.File;

import org.apache.catalina.connector.Connector;
import org.apache.coyote.http11.Http11NioProtocol;
import org.springframework.boot.context.embedded.EmbeddedServletContainerFactory;
import org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * tomcat 配置
 * @author liaokailin
 * @version $Id: TomcatConfig.java, v 0.1 2015年10月4日 上午12:11:47 liaokailin Exp $
 */
@Configuration
public class TomcatConfig {

    @Bean
    public EmbeddedServletContainerFactory servletContainer() {
        TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory();
        tomcat.setUriEncoding("UTF-8");
        tomcat.addAdditionalTomcatConnectors(createSslConnector());
        return tomcat;
    }

    private Connector createSslConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();
        try {
            File truststore = new File("/Users/liaokailin/software/ca1/keystore");
            connector.setScheme("https");
            protocol.setSSLEnabled(true);
            connector.setSecure(true);
            connector.setPort(8443);
            protocol.setKeystoreFile(truststore.getAbsolutePath());
            protocol.setKeystorePass("123456");
            protocol.setKeyAlias("springboot");
            return connector;
        } catch (Exception ex) {
            throw new IllegalStateException("cant access keystore: [" + "keystore" + "]  ", ex);
        }
    }
}


```

通过`addAdditionalTomcatConnectors`方法添加多个监听连接;此时可以通过http 9090端口，https 8443端口。








