#spring boot实战(第一篇)第一个案例

## 前言
###写在前面的话
	一直想将spring boot相关内容写成一个系列的博客，今天终于有时间开始了第一篇文章
	以后有时间就会继续写下去。
	
### spring boot 博客内容规划

* spring boot 基本用法
* 自动配置
* 技术集成
* 性能监控
* 源码解析

` spring boot` 功能强大，后面会细细道来。

## 第一个案例

###  工程的构建	

构建spring boot工程一般采用两种方式 gradle 、maven;相对于maven的pom配置gradle 更加简单，有兴趣的同学可以去学习下gradle，这里采用`maven`。

创建一个maven工程，对应的`pom.xml`文件：

```

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.2.4.RELEASE</version>
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>

	<dependencies>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

```
版本采用`1.2.4`,非官网最新版，但属于稳定版本

创建 `Application.java`

```
package com.u51.lkl.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
运行main方法

```

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.2.4.RELEASE)

2015-08-25 22:53:35.484  INFO 554 --- [           main] com.u51.lkl.springboot.Application       : Starting Application on mac.local with PID 554 (/Users/liaokailin/code/github/blog-springboot/target/classes started by lkl in /Users/liaokailin/code/github/blog-springboot)
2015-08-25 22:53:35.561  INFO 554 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@328908cd: startup date [Tue Aug 25 22:53:35 CST 2015]; root of context hierarchy
2015-08-25 22:53:36.395  INFO 554 --- [           main] o.s.b.f.s.DefaultListableBeanFactory     : Overriding bean definition for bean 'beanNameViewResolver': replacing [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration; factoryMethodName=beanNameViewResolver; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [org/springframework/boot/autoconfigure/web/ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration.class]] with [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter; factoryMethodName=beanNameViewResolver; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter.class]]
2015-08-25 22:53:37.404  INFO 554 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 8080 (http)
2015-08-25 22:53:37.796  INFO 554 --- [           main] o.apache.catalina.core.StandardService   : Starting service Tomcat
2015-08-25 22:53:37.798  INFO 554 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.0.23
2015-08-25 22:53:37.955  INFO 554 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2015-08-25 22:53:37.955  INFO 554 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 2397 ms
2015-08-25 22:53:38.813  INFO 554 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2015-08-25 22:53:38.818  INFO 554 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'characterEncodingFilter' to: [/*]
2015-08-25 22:53:38.819  INFO 554 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2015-08-25 22:53:39.075  INFO 554 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@328908cd: startup date [Tue Aug 25 22:53:35 CST 2015]; root of context hierarchy
2015-08-25 22:53:39.151  INFO 554 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],methods=[],params=[],headers=[],consumes=[],produces=[],custom=[]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2015-08-25 22:53:39.151  INFO 554 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],methods=[],params=[],headers=[],consumes=[],produces=[text/html],custom=[]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest)
2015-08-25 22:53:39.180  INFO 554 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2015-08-25 22:53:39.180  INFO 554 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2015-08-25 22:53:39.232  INFO 554 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2015-08-25 22:53:39.326  INFO 554 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2015-08-25 22:53:39.430  INFO 554 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2015-08-25 22:53:39.433  INFO 554 --- [           main] com.u51.lkl.springboot.Application       : Started Application in 4.829 seconds (JVM running for 5.238)

```

spring boot已经启动，内嵌`tomcat`容器，监听为`8080`端口

就是这么简单，一个spring boot 的程序就创建了。


### @SpringBootApplication 注解

SpringBootApplication注解源码如下：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Configuration
@EnableAutoConfiguration
@ComponentScan
public @interface SpringBootApplication {

	/**
	 * Exclude specific auto-configuration classes such that they will never be applied.
	 * @return the classes to exclude
	 */
	Class<?>[] exclude() default {};

}
```
> `@Configuration` : 表示`Application`作为sprig配置文件存在
> `@EnableAutoConfiguration`: 启动spring boot内置的自动配置
> `@ComponentScan` : 扫描bean，路径为`Application`类所在`package`以及`package`下的子路径，这里为 `com.u51.lkl.springboot`，在`spring boot`中bean都放置在该路径已经子路径下。

### 构建REST工程

 上面的操作连个HelloWorld都没有出来，远远满不足我们的需求。
 
 创建一个package：`com.u51.lkl.springboot.controller` 保存`controller`
 
 构建 `HelloWorldController.java`
 
 ```
 package com.u51.lkl.springboot.controller;

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/springboot")
public class HelloWorldController {

    @RequestMapping(value = "/{name}", method = RequestMethod.GET)
    @ResponseBody
    public String sayWorld(@PathVariable("name") String name) {
        return "Hello " + name;
    }
}

 ```
 
再次执行 `Application` 然后访问[http://localhost:8080/springboot/Liaokailin](http://localhost:8080/springboot/Liaokailin)

得到结果：
	`Hello Liaokailin`
	
方法中涉及的注解就不解释了，比较简单～


##第一个案例就到这里。



	
	


	