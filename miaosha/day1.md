## 1 使用idea创建一个maven项目

使用archetype-quickstart创建一个maven项目

maven项目的初始结构大致如下：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210302154902377.png" alt="image-20210302154902377" style="zoom:67%;" />

main下的java标记为sources root，test标记为test sources root。还需要在main目录下创建resources文件夹（放置资源的配置文件）并标记为resources root。

## 2 构建springboot基础框架

1 在xml文件中进行配置

首先引用springboot的父级依赖，表示这个项目是springboot项目。spring-boot-starter-parent是一个特殊的starter，用来提供相关的maven默认依赖。使用后常用的包依赖可以省去version标签。

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.2</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
```

接着添加依赖。

```

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

```

配置完成后在APP类中加入EnableAutoConfiguration注解（将App的启动类当成可以自动化可以支持配置的bean，并且能够开启基于springboot的整个自动化的一个配置（spring的自动化配置：会将对数据库或者spring本身的一些依赖自动化加入到项目工程中），使用SpringApplication.run(App.class,args)语句，启动后，默认启动了一个tomcat容器并且在8080端口被监听。

使用注解@RestController，

添加以下代码：

```
@RequestMapping("/")
public String home(){
	return "hello world";
}
```

启动后可以在浏览器显示8080.

## 3 修改springboot的配置

在对应classpath的resuorces目录下创建application.properties，根据keyvalue对加载一些配置，在其中可以设置端口等，如

```
server.port=8090
```

## 4 mybatis接入springboot项目







