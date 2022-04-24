# Dubbo



# 远程调用

## 协议

http协议是超文本传输协议，传输文本数据

ftp协议是文件的上传和下载

RPC协议：远程过程调用协议，不同服务器之间的调用它的网络交互功能的代码就不需要我们去编写了，它的调试和维护也很麻烦

​					算是一个很大的协议，并且它还有很多小协议

## RPC-远程过程调用协议 

### RPC 是什么？ 

PRC 是 Remote Procedure Call Protocol ，称为：远程过程调用协议。是一种通过网络从 远程计算机程序上请求服务，而不需要了解底层网络技术的协议。该协议允许运行于一台计 算机的程序调用另一台计算机的程序。程序员无需编为网络交互功能编码。 

### RPC 的作用？ 

主要功能是让构建分布式计算（应用）更容易，在提供强大的远程调用能力时不损失本 地调用的语义简洁性。在一台计算的程序使用其他计算机上的功能就是使用自己的功能一样。 RPC 技术提供了透明的访问其他服务的底层实现细节。使用分布式系统中的服务更加方便。 

### 分布式？

 分布式指多台计算机位于网络系统中，多台计算给形成一个整体对外界提供服务。用户 使用系统不知道是多台计算机，使用不同的操作系统，不同的应用程序提供服务。

# Dubbo框架

### 简介

简介：Dubbo 是一个框架 Dubbo 是一个分布式服务框架，致力于提供高性能和透明化的 RPC 远程服务调用方案、服 务治理方案。

## Dubbo服务的实现原理

Dubbo 的底层实现是动态代理，由 Dubbo 框架创建远程服务（接口）对象的代理对象， 通过代理对象调用远程方法。

![image-20211112165411184](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211112165411184.png)

调用端会去创建一个代理对象，调用这个代理对象通过序列化转为二进制或json数据，再按协议组装数据，最后再发送给服务提供者（服务端），服务端接收数据后按协议解析收到的数据，调用服务程序，处理业务，得到结果后再把处理结果发给调用者，调用者解析返回的数据，通过反序列化变为对象。

### Dobbo支持的协议

支持 8 种协议：dubbo , hessian , rmi , http, webservice , thrift , memcached , redis。 dubbu 官方推荐使用 dubbo 协议。

dubbo的默认端口号是：20880

### Dobbo协议

A、Dubbo 协议特点 

Dubbo 协议采用单一长连接和异步通讯，适合于小数据量大并发的服务调用，以及服务消费 者机器数远大于服务提供者机器数的情况。 B、 网络通信

Dubbo 协议底层网络通信默认使用的是 netty，性能非常优秀，官方推荐使用 

C、 不适合的地方

 Dubbo 协议不适合传送大数据量的服务，比如传文件，传视频等，除非请求量很低

**D、使用 Dubbo 协议**



在spring容器中加入

<dubbo:protocol name="dubbo" portl="20880" />

### 长连接和短连接

Dubbo协议使用的是长链接

原因：长连接一般都是不会关闭的，像油厂到加油站用管道输油。这个管道可以不停的传输一些比较小的请求。

​			开启和销毁通道都是很费时的。但是他们占用服务器的内存空间会比较大

​			短链接一般是会关闭的，像油厂通过有车输油。这个管道可以输送大型的请求，但是会关闭，

​			所以一般客户端与服务器之间用短链接，服务器与服务器之间用长链接

![image-20211112172637154](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211112172637154.png)

![image-20211112172845455](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211112172845455.png)

长连接和短连接接： 

所谓长连接，指在一个连接上可以连续发送多个数据包，在连接保持期间，如果没有 数据包发送，需要双方发检测包。短连接是指通讯双方有数据交互时，就建立一个连接，数 据发送完成后，则断开此连接，即每次连接只完成一项业务的发送。 

长连接多用于操作频繁，点对点的通讯，而且连接数不能太多情况。例如：数据库的 连接用长连接。像 Web 网站的 http 服务一般都用短链接，因为长连接对于服务端来说会耗 费一定的资源，而像 Web 网站频繁的用，使用短连接会更省一些资源，并发量大，但每个 用户无需频繁操作情况下需用短连好。

### Dubbo的组件

五个组件：  提供者，消费者，监控器，注册器，容器（Spring）

![image-20211112173132457](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211112173132457.png)

# Dubbo的直连方式

## 配置提供者

![image-20211113160448232](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211113160448232.png)



1.配置相关依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.example</groupId>
  <artifactId>dubbo01_directcontext</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>dubbo01_directcontext Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
<!--    spring-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>4.3.16.RELEASE</version>
    </dependency>
<!--springmvc-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.3.16.RELEASE</version>
    </dependency>
<!--    dubbo-->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>dubbo</artifactId>
      <version>2.6.2</version>
    </dependency>
  </dependencies>

  <build>

  </build>
</project>
```

2.定义实体类，这是要在网络中传输的数据，所以需要去实现序列化接口

![image-20211114202212456](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211114202212456.png)

3.定义接口和接口实现类

```java
package com.wk.dubbo.service;

import com.wk.dubbo.model.User;

public interface UserService {
    User queryUserById(Integer id);
}
```

```java
package com.wk.dubbo.service.impl;

import com.wk.dubbo.model.User;
import com.wk.dubbo.service.UserService;

public class UserServiceImpl implements UserService {
    @Override
    public User queryUserById(Integer id) {
        User user=new User();
        user.setAge(23);
        user.setId(id);
        user.setUsername("陈郡阳");
        return user;
    }
}
```

4.配置dubbo服务提供者的核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo
       http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
<!--
    服务提供者声明名称：必须保证服务名称的唯一性，它的名称是dubbo内部使用的唯一标识
-->
<!--    <dubbo:application name="01_directlink_provider"></dubbo:application>-->
    <dubbo:application name="01_directlink_provider"></dubbo:application>
<!--
    访问服务协议的名称及端口号，dubbo官方推荐使用的是dubbo协议，默认端口是20880
    name：协议的名字               port：端口号
-->
    <dubbo:protocol name="dubbo" port="20880"></dubbo:protocol>
<!--
    暴露服务的接口  dubbo:service
    interface:暴露接口的全限定类目   ref：接口引用的实现类在spring容器中的表示    register：表示连接的方式
-->
    <dubbo:service interface="com.wk.dubbo.service.UserService" ref="userService" registry="N/A"></dubbo:service>
<!--将接口的实现类加载到spring容器中-->
    <bean id="userService" class="com.wk.dubbo.service.impl.UserServiceImpl"></bean>

</beans>
```

5.添加监听器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:dubbo_userservice_provider.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

</web-app>
```

注意：直连的方式获取远程接口需要通过获取提供者jar包依赖的方式来得到接口

## 配置消费者

![image-20211114202700766](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211114202700766.png)

1.添加依赖                     注意，消费者需要得到提供者的接口需要导入提供者依赖的jar包

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.example</groupId>
  <artifactId>dubbo02_directlink_consumer</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>dubbo01_directcontext Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <!--    spring-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>4.3.16.RELEASE</version>
    </dependency>
    <!--springmvc-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.3.16.RELEASE</version>
    </dependency>
    <!--    dubbo-->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>dubbo</artifactId>
      <version>2.6.2</version>
    </dependency>
<!--    依赖提供者的接口-->
    <dependency>
      <groupId>org.example</groupId>
      <artifactId>dubbo01_directcontext</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>
  </dependencies>

  <build>

  </build>
</project>
```

2.设置dubbo的核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"

       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo
       http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <!--声明服务消费者的名称，保证唯一性-->
<!--    <dubbo:application name="002_link_consumer"></dubbo:application>-->
    <dubbo:application name="002_link_consumer"></dubbo:application>
<!--
    引用远程服务接口
    id:远程服务接口对象名称
    interface:调用远程接口的全限定名称
    url：访问服务接口的地址
    registry：连接的方式
-->
    <dubbo:reference id="userService"
                     interface="com.wk.dubbo.service.UserService"
                     url="dubbo://localhost:20880"
                     registry="N/A">
    </dubbo:reference>



</beans>
```

3.写controller类

![image-20211114203012215](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211114203012215.png)

4.配置中央调度器

要注意配置注解驱动的包，不然后面可能会启动不了

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">
<!--    扫描组件-->
    <context:component-scan base-package="com.wk.dubbo.web"></context:component-scan>
<!--    配置注解驱动-->
<!--    <mvc:annotation-driven></mvc:annotation-driven>-->
    <mvc:annotation-driven></mvc:annotation-driven>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
</beans>
```

5.注册中央调度器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:application.xml,classpath:dubbo_consumer.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

**注：只要是分布式开发的话，实体类都要实现序列化**

# Dubbo官方推荐的连接方式

![image-20211115090359204](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211115090359204.png)

# 注册中心

## 为什么使用注册中心

对于服务提供方，它需要发布服务，而且由于应用系统的复杂性，服务的数量、类型也不断膨胀；对于服务消费方，它最关心如何获取到它所需要的服务，而面对复杂的应用系统， 需要管理大量的服务调用。

而且，对于服务提供方和服务消费方来说，他们还有可能兼具这两种角色，即需要提供服务，有需要消费服务。 通过将服务统一管理起来，可以有效地优化内部应用对服务发布/ 使用的流程和管理。服务注册中心可以通过特定协议来完成服务对外的统一。

## 注册中心是什么？

Zookeeper 是一个高性能的，分布式的，开放源码的分布式应用程序协调服务。简称 zk。Zookeeper 是翻译管理是动物管理员。可以理解为 windows 中的资源管理器或者注册表。他是一个树形结构。这种树形结构和标准文件系统相似。ZooKeeper 树中的每个节点被称为Znode。和文件系统的目录树一样，ZooKeeper 树中的每个节点可以拥有子节点。每个节点表示一个唯一服务资源。Zookeeper 运行需要 java 环境。

![image-20211117084219066](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211117084219066.png)

## 下载zookeeper

### windows下载zookeeper

下载后

复制一份zoo_sample文件改名为zoo文件

![image-20211117084439792](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211117084439792.png)

创建data文件，存储临时数据

![image-20211117084821074](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211117084821074.png)

配置zoo文件

![image-20211117084747031](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211117084747031.png)

启动zookeeper

![image-20211117084909975](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211117084909975.png)

### linux下载zookeeper

①解压zookeeper压缩包

![image-20211124222626836](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211124222626836.png)

②创建data文件，并记录data文件地址

![image-20211124222753153](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211124222753153.png)

③进入conf文件并复制zoo_sample文件，命名为zoo文件，并修改zoo文件

![image-20211124223044622](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211124223044622.png)

④修改zoo文件

![image-20211124223334010](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211124223334010.png)

⑤开启zookeeper

在bin目录下输入./zkServer.sh start

![image-20211124223758737](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211124223758737.png)

在bin目录下输入./zkServer.sh stop



⑥查看zookeeper是否开启了ps -ef | grep zook

![image-20211124223749186](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211124223749186.png)

## 使用注册中心例子

### 创建一个接口项目

接口项目中存放，传输的实体类和暴露的接口。    

**注：实体类需要序列化**

![image-20211117085311760](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211117085311760.png)

![image-20211117085341856](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211117085341856.png)

### 创建提供者项目

pom文件

文件中记得加入暴露接口类的依赖和zookeeper的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.example</groupId>
  <artifactId>007_zk_provider</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>007_zk_provider Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <!--    spring-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>4.3.16.RELEASE</version>
    </dependency>
    <!--springmvc-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.3.16.RELEASE</version>
    </dependency>
    <!--    dubbo-->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>dubbo</artifactId>
      <version>2.6.2</version>
    </dependency>
<!--    暴露接口类-->
    <dependency>
      <groupId>org.example</groupId>
      <artifactId>006_zk_interface</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>
<!--    zookeeper依赖-->
    <dependency>
      <groupId>org.apache.curator</groupId>
      <artifactId>curator-framework</artifactId>
      <version>4.1.0</version>
    </dependency>
  </dependencies>

  <build>

  </build>
</project>
```

暴露接口的实现类

![image-20211117085459070](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211117085459070.png)

配置dubbo配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo
       http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
<!--    声明dobbo服务器的名称，保证唯一性-->
    <dubbo:application name="007_zk_provider"></dubbo:application>
<!--    声明dubbo使用的协议的名称和端口号-->
    <dubbo:protocol port="20880" name="dubbo"></dubbo:protocol>
<!--    现在要使用zookeeper    指定注册中心的地址和端口号-->
    <dubbo:registry address="zookeeper://localhost:2181"></dubbo:registry>
<!--    暴露服务接口-->
    <dubbo:service interface="com.bjpowernode.dubbo.service.UserService" ref="UserServiceImpl"></dubbo:service>
<!--    接口实现类-->
    <bean id="UserServiceImpl" class="com.bjpowernode.dubbo.service.impl.UserServiceImpl"></bean>
</beans>
```

web.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
	//监听器
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    //扫描配置文件
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:dubbo-zk-provider.xml</param-value>
    </context-param>
</web-app>
```

### 创建一个消费者项目

pom文件

pom文件要依赖接口项目和zookeeper的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.example</groupId>
  <artifactId>008_zk_consumer</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>008_zk_consumer Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <!--    spring-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>4.3.16.RELEASE</version>
    </dependency>
    <!--springmvc-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.3.16.RELEASE</version>
    </dependency>
    <!--    dubbo-->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>dubbo</artifactId>
      <version>2.6.2</version>
    </dependency>
    <!--    接口-->
    <dependency>
      <groupId>org.example</groupId>
      <artifactId>006_zk_interface</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>
    <!--    zookeeper依赖/注册中心的依赖-->
    <dependency>
      <groupId>org.apache.curator</groupId>
      <artifactId>curator-framework</artifactId>
      <version>4.1.0</version>
    </dependency>
  </dependencies>

  <build>

  </build>
</project>
```

springmvc的配置文件

**注意annotation的类**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">
<!--    扫描器-->
    <context:component-scan base-package="com.bjpowernode.dubbo.web"></context:component-scan>
<!--    注册中心-->
    <mvc:annotation-driven></mvc:annotation-driven>
<!--    视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
</beans>
```

dubbo配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
<!--    声明消费者唯一性，保证唯一性-->
    <dubbo:application name="dubbo-zk-consumer"></dubbo:application>
<!--    指定注册中心-->
    <dubbo:registry address="zookeeper://localhost:2181"></dubbo:registry>
<!--    引用远程接口服务-->
<!--    <dubbo:service interface="com.bjpowernode.dubbo.service.UserService"></dubbo:service>-->
    <dubbo:reference id="userService" interface="com.bjpowernode.dubbo.service.UserService"></dubbo:reference>
</beans>
```

controller类

![image-20211117090336848](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211117090336848.png)

web-xml文件

扫描dubbo配置文件，和springmvc配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:application.xml,classpath:dubbo-zk-consumer.xml</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

## linux使用注册中心

只需要将dubbo中的注册中心的地址改成虚拟机的地址就可以了，还需要开启虚拟机中的zookeeper，开启文件就是windows中的文件./

# 负载均衡



# 一些错误

SpringCacheInterceptor错误

![image-20211113190720256](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211113190720256.png)



![image-20211113190755490](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211113190755490.png)

原因：注册驱动导错了

![image-20211113191131248](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211113191131248.png)

![image-20211113191145396](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211113191145396.png)

修改成正确的注解驱动即可

![image-20211113191258266](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20211113191258266.png)





linux中开启注册中心会遇到的错误

Error:Zookeeper is not connected yet!

原因：
1.pom.xml引入的坐标版本与服务器的Zookeeper版本不一致
2.linux的防火墙未关闭