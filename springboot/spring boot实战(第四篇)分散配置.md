#spring boot实战(第四篇)分散配置

##前言 
分散配置是系统必不可少的一部分，将配置参数抽离出来为后期维护提供很大的便利。spring boot 默认支持两个格式的配置文件:*.properties  *.yml。

##*.properties与*.yml
*.properties属性文件；属于最常见的一种；
*.yml是yaml格式的文件，yaml是一种非常简洁的标记语言。

在*.properties中定义`user.address.stree=hangzhou`等价与yaml文件中的

```
   user:
   		 address: 
   		         stree:hangzhou
```
从上可以发现yaml层次感更强，具体在项目中选择那种资源文件是没有什么规定的。 

## spring boot配置

###简单案例
首先在类路径下创建`application.properties`文件并定义
`name=liaokailin`

创建一个bean`User.java`

```
@Component
public class User {

    private @Value("${name:lkl}") String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
```

在  `HelloWorldController.java`调用对应bean

```
package com.lkl.springboot.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import com.lkl.springboot.config.User;
import com.lkl.springboot.event.CallEventDemo;

@RestController
@RequestMapping("/springboot")
public class HelloWorldController {

    @Autowired
    CallEventDemo callEventDemo;

    @Autowired
    User          user;

    @RequestMapping(value = "/{name}", method = RequestMethod.GET)
    @ResponseBody
    public String sayWorld(@PathVariable("name") String name) {
        System.out.println("userName:" + user.getName()); 
        return "Hello " + name;
    }

}

```

启动该spring boot工程，在命令行执行

`curl  http://localhost:8080/springboot/liaokailin`
控制台成功输出name对应值

###解析
* 在spring boot中默认会加载 `classpath:/,classpath:/config/,file:./,file:./config/` 路径下以`application`命名的property或yaml文件；
* 参数`spring.config.location`设置配置文件存放位置
* 参数`spring.config.name`设置配置文件名称

###配置文件获取随机数

在spring boot配置文件中可调用`Random`中方法

在`application.properties`中为user增加age参数 `age=${random.int}`

```
name=liaokailin
age=${random.int}
```
bean中同时增加参数

```
@Component
public class User {

    private @Value("${name:lkl}") String name;
    private @Value("${age}") Integer     age;
    //getter and setter and toString()
```
在启动工程时会为age随机生成一个值 

```
${random.int(100)} : 限制生成的数字小于10
${random.int[0,100]} : 指定范围的数字
```

###在配置文件调用占位符

修改配置文件：

```
userName=liaokailin
age=${random.int[0,100]}
remark=hello,my name is ${userName},age is ${age}
```
修改bean:

```
@Component
public class User {

    private @Value("${userName:lkl}") String name;
    private @Value("${age}") Integer         age;
    private @Value("${remark}") String       remark;
```
执行发现remark答应出:
`remark=hello,my name is liaokailin,age is 25`。

可以发现将`name`修改为`userName`,在配置文件中调用`${name}`是工程名。

###去掉@Value

大家可以发现前面在bean中调用配置参数使用的为注解`@Value`,在spring boot中是可以省去该注解。

配置文件：

```
userName=liaokailin
age=${random.int[0,100]}
remark=hello,my name is ${userName},age is ${age}
user.address=china,hangzhou
```
增加`user.address=china,hangzhou`，为了调用该参数来使用**`@ConfigurationProperties`**。

`User.java`

```
@Component
@ConfigurationProperties(prefix = "user")
public class User {

    private @Value("${userName:lkl}") String name;
    private @Value("${age}") Integer         age;
    private @Value("${remark}") String       remark;
    private String                           address;

```
使用`@ConfigurationProperties`需要指定`prefix`,同时bean中的属性和配置参数名保持一致。

###实体嵌套配置
在User中定义一个Address实体同样可以快捷配置

`User.java`

```
@Component
@ConfigurationProperties(prefix = "user")
public class User {

    private @Value("${userName:lkl}") String name;
    private @Value("${age}") Integer         age;
    private @Value("${remark}") String       remark;
    private String                           address;
    private Address                          detailAddress;
```

 `Address.java`
 
 ```
 public class Address {

    private String country;
    private String province;
    private String city;
    ...
}
 ```
`application.properties`

```
userName=liaokailin
age=${random.int[0,100]}
remark=hello,my name is ${userName},age is ${age}
user.address=china,hangzhou
user.detailAddress.country=china
user.detailAddress.province=zhejiang
user.detailAddress.city=hangzhou
``` 
运行得到

```
userUser [name=liaokailin, age=57, remark=hello,my name is liaokailin,age is 0, address=china,hangzhou, detailAddress=Address [country=china, province=zhejiang, city=hangzhou]]
```
这种嵌套关系如果通过yaml文件展示出来层次感会更强。

```
user:
	detailAddress:
		country:china
		province:zhejiang
		city:hangzhou

```
注意在yaml中缩进不要使用`TAB`

###配置集合
一个人可能有多个联系地址，那么地址为集合

 `User.java`
 
 ```
 @Component
@ConfigurationProperties(prefix = "user")
public class User {

    private @Value("${userName:lkl}") String name;
    private @Value("${age}") Integer         age;
    private @Value("${remark}") String       remark;
    private String                           address;
    private Address                          detailAddress;
    private List<Address>                    allAddress = new ArrayList<Address>();

 ```
 
 `application.properties`
 
 ```
user.allAddress[0].country=china
user.allAddress[0].province=zhejiang
user.allAddress[0].city=hangzhou

user.allAddress[1].country=china
user.allAddress[1].province=anhui
user.allAddress[1].city=anqing

 ```
 通过`下标`表明对应记录为集合中第几条数据，得到结果：
 
```
userUser [name=liaokailin, age=64, remark=hello,my name is liaokailin,age is 82, address=china,hangzhou, detailAddress=Address [country=china, province=zhejiang, city=hangzhou], allAddress=[Address [country=china, province=zhejiang, city=hangzhou], Address [country=china, province=anhui, city=anqing]]]
```
如果用yaml文件表示为:

`application.yml`

```
user:
	-allAddress:
		country:china
		province:zhejiang
		city:hangzhou
	 -allAddress:
		country:china
		province:anhui
		city:anqing

```

##多配置文件

spring boot设置多配置文件很简单，可以在bean上使用注解`@Profile("development")`即调用`application-development.properties|yml`文件,也可以调用`SpringApplication`中的`etAdditionalProfiles()`方法。

例如：

```
package com.lkl.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Profile;

import com.lkl.springboot.listener.MyApplicationStartedEventListener;

@Profile("development")
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(Application.class);
        //   app.setAdditionalProfiles("development");
        app.addListeners(new MyApplicationStartedEventListener());
        app.run(args);
    }
}

```
也可以通过启动时指定参数`spring.profiles.active`。


##题外话

在实际项目中最好是将配置参数抽离出来集中管理，比如利用淘宝的super-diamond ,consul,zk 等。

下一篇讲述spring boot内嵌容器tomcat以及ssl配置。








