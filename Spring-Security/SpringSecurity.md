# SpringSecurity

忽略运行前未bean类的非运行时异常
@SuppressWarnings("all")

## 第一章  了解Spring Security

### 1.1概要

Spring 是非常流行和成功的 Java 应用开发框架，Spring Security 正是 Spring 家族中的 成员。Spring Security 基于 Spring 框架，提供了一套 Web 应用安全性的完整解决方 案。

正如你可能知道的关于安全方面的两个主要区域是“认证”和“授权”（或者访问控 制），一般来说，Web 应用的安全性包括用户认证（Authentication）和用户授权 （Authorization）两个部分，这两点也是 Spring Security 重要核心功能。

（1）用户认证指的是：验证某个用户是否为系统中的合法主体，也就是说用户能否访问 该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认 证过程。**通俗点说就是系统认为用户是否能登录**

（2）用户授权指的是验证某个用户是否有权限执行某个操作。在一个系统中，不同用户 所具有的权限是不同的。比如对一个文件来说，有的用户只能进行读取，而有的用户可以 进行修改。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的 权限。**通俗点讲就是系统判断用户是否有权限去做某些事情。**

### 1.2 spring security 原理

基于 Filter , Servlet, AOP 实现身份认证和权限验证

## 第二章 实例驱动学习

### 第一个例子：初探

#### **实现步骤**

1.创建 maven 项目

2.加入依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

3.创建应用启动类

```java
package com.guigu;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Springsecurity01StartApplication {

    public static void main(String[] args) {
        SpringApplication.run(Springsecurity01StartApplication.class, args);
    }

}
```

4.创建controller类

```java
package com.guigu.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class TestController {
    @ResponseBody
    @RequestMapping(value = "/hello",method = RequestMethod.GET)
    public String hello(){
        return "hello,security";
    }
}
```

5.框架生成的用户

用户名： user 

密码： 在启动项目时，生成的临时密码。uuid 

日志中生成的密码： generated security password: 9717464c-fafd-47b3-9995-2c18b24f7336

![image-20220116093836969](E:\Typora\Typora\Spring-Security\image\image-20220116093836969.png)

6.自定义用户名和密码

在springboot配置文件下自定义用户密码

```properties
spring.security.user.name=guigu
spring.security.user.password=guigu
```

7.关闭验证

排除Security配置，不让他使用（调试程序的时候可以用）

使用方法：**@SpringBootApplication(exclude = {SecurityAutoConfiguration.class})**

```java
package com.guigu;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration;

@SpringBootApplication(exclude = {SecurityAutoConfiguration.class})
public class Springsecurity01StartApplication {

    public static void main(String[] args) {
        SpringApplication.run(Springsecurity01StartApplication.class, args);
    }

}
```

### 第二个例子：使用内存中的用户信息

#### **实现步骤**

##### 1.使用WebSecurityConfigurerAdapter 控制安全管理的内容。

需要做的使用：继承 WebSecurityConfigurerAdapter，重写方法。实现

自定义的认证信息。重写下面的方法。

protected void configure(AuthenticationManagerBuilder auth)

##### 2.spring security 5 版本要求密码比较加密，否则报错

java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null

**实现密码加密：**

1）创建用来加密的实现类（选择一种加密的算法）

![image-20220116104700327](E:\Typora\Typora\Spring-Security\image\image-20220116104700327.png)

2）给每个密码加密

![image-20220116104840933](E:\Typora\Typora\Spring-Security\image\image-20220116104840933.png)

例子

config类

```java
package com.guigu.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
@EnableWebSecurity
public class MyWebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        MyWebSecurityConfig web=new MyWebSecurityConfig();
        PasswordEncoder pe=web.passwordEncoder();
        auth.inMemoryAuthentication().withUser("cwh").password(pe.encode("cwh")).roles();
        auth.inMemoryAuthentication().withUser("yzh").password(pe.encode("yzh")).roles();
        auth.inMemoryAuthentication().withUser("admin").password(pe.encode("123456")).roles();
    }
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

}
```

注解： 

1. @Configuration ：表示当前类是一个配置类（相当于是 spring 的 xml 配置文件），在这个类方法的返回值是 java 对象，这些对象放入到 spring /springboot容器中。
2. @EnableWebSecurity：表示启用 spring security 安全框架的功能
3. @Bean：把方法返回值的对象，放入到 spring 容器中

### 第三个例子：基于角色Role的身份认证

基于角色 Role 的身份认证， 同一个用户可以有不同 的角色。同时可以开启对方法级别的认证。

#### 实现步骤

1.设置用户的角色

继承 WebSecurityConfigurerAdapte重写 configure 方法。指定用户的 roles

![image-20220116114609537](E:\Typora\Typora\Spring-Security\image\image-20220116114609537.png)![image-20220116114620612](E:\Typora\Typora\Spring-Security\image\image-20220116114620612.png)

2.在config类的上面加入启用方法级别的注解

@EnableGlobalMethodSecurity(prePostEnabled = true）

```java
/*
*    @EnableGlobalMethodSecurity：启用方法级别的认证
*    prePostEnabled默认是false
*       true：表示可以在controller的方法上使用@PreAuthorize注解和@PostAuthorize注解
* */
@EnableGlobalMethodSecurity(prePostEnabled = true)
```

![image-20220116115047820](E:\Typora\Typora\Spring-Security\image\image-20220116115047820.png)

3.在处理器方法的上面加入角色的信息，指定方法可以访问的角色列表

```java
	@ResponseBody
    @RequestMapping(value = "/hello",method = RequestMethod.GET)
//指定admin，common都可以访问
    @PreAuthorize(value ="hasAnyRole('admin','common')")
    public String hello(){
        return "hello,security";
    }
```

**使用@PreAuthorize 指定在方法之前进行角色的认证。 hasAnyRole('角色名称 1','角色名称 N')**

### 第四个例子，基于 jdbc 的用户认证

其实就是将内存中的用户信息（账号，密码，角色）存在了SpringSecurity框架对象用户信息的表示类UserDetails中，我们不用在内存中写账号，密码，角色了，只需要从数据库中获取即可

#### 概要

1）在 spring security 框架对象用户信息的表示类是 **UserDetails**。**UserDetails** 是一个接口，高度抽象的用户信息类（相当于项目中的 实体 类）

**User 类：是 UserDetails 接口的实现类， 构造方法有三个参数**

username，password, authorities

需要向 spring security 提供 **User 对象**， 这个对象的数据来自数据库 的查询

（authorities（权限）在User对象中是个集合）

2）实现 UserDetailsService 接口

重写方法 UserDetails loadUserByUsername(String var1) 在方法中获取数据库中的用户信息， 也就是执行数据库的查询，条 件是用户名称

#### 实验步骤

1.实现UserDetailsService接口

这个类向SpringSecurity提供UserDetails

```java
package com.bjpowernode.provider;
import com.bjpowernode.dao.UserInfoDao;
import com.bjpowernode.entity.UserInfo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Component;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
@SuppressWarnings("all")
@Component("MyUserDetailService")
public class MyUserDetailService implements UserDetailsService {    
    @Autowired    
    UserInfoDao dao;    
    UserInfo userInfo=null;    
    @Override    
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {        
        User user=null;        
        if(username!=null){            
            Map map = new HashMap();            
            map.put("username",username);            
            List<UserInfo> list=dao.selectByMap(map);            
            userInfo=list.get(0);
            //            拿到返回值UserDetails，User是UserDetails的实现类，
            //            list1是返回值的类型Collection（GrantedAuthority）
            //            SimpleGrantedAuthority是GrantedAuthority的实现类
            //            ，一般用SimpleGrantedAuthority它的构造方法            
            List<GrantedAuthority> list1=new ArrayList<>();
            //            角色必须以"ROLE_"开头            
            GrantedAuthority grantedAuthority=new SimpleGrantedAuthority("ROLE_"+userInfo.getRole());            		list1.add(grantedAuthority);            
            user=new User(userInfo.getUsername(), userInfo.getPassword(), list1);        
        }        
        return user;    
    }
}
```

2.config类

和之前的使用内存存用户信息差不多，只是jdbc它将用户信息全部存在了UserDetails中

```java
package com.bjpowernode.config;import com.bjpowernode.provider.MyUserDetailService;import org.springframework.beans.factory.annotation.Autowired;import org.springframework.beans.factory.annotation.Qualifier;import org.springframework.context.annotation.Configuration;import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
@Configuration@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MyWebSecurityConfig extends WebSecurityConfigurerAdapter {    
    @Autowired    
    @Qualifier("MyUserDetailService")    
    MyUserDetailService myUserDetailService;    
    @Override    
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {        					      		auth.userDetailsService(myUserDetailService).passwordEncoder(new BCryptPasswordEncoder());    
    }
}
```

3.其他其实就是普通的从数据库获取数据的过程，一些（controller，service，dao类）

## 第三章 基于角色的权限

### 3.1 认证和授权

authentication：认证， 认证访问者是谁。 一个用户或者一个其他系 统是不是当前要访问的系统中的有效用户。

authorization：授权， 访问者能做什么

**例子：**

比如说张三用户要访问一个公司 oa 系统。 首先系统要判断张三是 不是公司中的有效用户。

认证：张三是不是有效的用户，是不是公司的职员

授权： 判断张三能否做某些操作， 如果张三是个领导可以批准下 级的请假， 其他的操作



如果张三只是一个普通用户，只能看自己的相关数据， 只能 提交请假申请等等。

### 3.2 RBAC 是什么？

#### 基本思想

RBAC 是基于角色的访问控制（Role-Based Access Control ） 在 RBAC 中，权限与角色相关联，用户通过成为适当角色的成员而得 到这些角色的权限。这就极大地简化了权限的管理。这样管理都是层 级相互依赖的，权限赋予给角色，而把角色又赋予用户，这样的权限 设计很清楚，管理起来很方便。



其基本思想是，对系统操作的各种权限不是直接授予具体的用 户，而是在用户集合与权限集合之间建立一个角色集合。每一种角色 对应一组相应的权限。一旦用户被分配了适当的角色后，该用户就拥 有此角色的所有操作权限。这样做的好处是，不必在每次创建用户时 都进行分配权限的操作，只要分配用户相应的角色即可，而且角色的 权限变更比用户的权限变更要少得多，这样将简化用户的权限管理， 减少系统的开销。



![image-20220116200442960](E:\Typora\Typora\Spring-Security\image\image-20220116200442960.png)

RBAC： 用户是属于角色的， 角色拥有权限的集合。 用户属于某 个角色， 他就具有角色对应的权限



例子：

系统中有张三，李四，他们是普通员工，只能查看数据。

系统中经理，副经理他们能修改数据。



设计有权限的集合，角色: 经理角色，具有修改数据的权限，删除， 查看等等。

普通用户角色： 只读角色，只能看数据 ，不能修改，删除



例子：

让张三，李四是只读的，普通用户角色。 让经理，副经理他们都是 经理角色。

公司以后增加新的普通员工，加入到“普通用户角色”就可以了，不 需要在增加新的角色。

公司增加经理了， 只要加入到“经理角色”就可以了。



*权限：能对资源的操作， 比如增加，修改，删除，查看等等。* 

*角色：自定义的， 表示权限的集合。一个角色可以有多个权限。*



#### RBAC 设计中的表

1.用户表： 用户认证（登录用到的表） 

   用户名，密码，是否启用，是否锁定等信息。

2.角色表：定义角色信息

   角色名称， 角色的描述

3.用户和角色的关系表： 用户和角色是多对多的关系。 

​	一个用户可以有多个角色， 一个角色可以有多个用户。 

4.权限表

​	角色可以有哪些权限

5.角色和权限的关系表 

​	一个角色可以有多少权限，一个权限可以给多少个角色

### 3.3 spring specurity 中认证的接口和类

#### 1） UserDetails：接口

主要作用：表示用户信息的。

boolean isAccountNonExpired(); 账号是否过期 

boolean isAccountNonLocked();账号是否锁定 

boolean isCredentialsNonExpired();证书是否过期 

boolean isEnabled();账号是否启用 

Collection getAuthorities(); 权限集合



UserDetails接口的实现类 User

org.springframework.security.core.userdetails.User



可以：自定义类实现 UserDetails 接口，作为你的系统中的用户类。这 个类可以交给 spring security 使用

#### 2） UserDetailsService 接口

主要作用：获取用户信息，得到是 UserDetails 对象。一般项目中都需 要自定义类实现这个接口，从数据库中获取数据

一个方法需要实现： 

UserDetails loadUserByUsername(String var1) ：

根据用户名称，获取用 户信息（用户名称，密码，角色结合，是否可用，是否锁定等信息）



除了我们自定义以外的一些UserDetailsService 接口的实现类：

##### ①InMemoryUserDetailsManager

作用：在内存中维护用户信息。

​	优点：使用方便。 

​	缺点：数据不是持久的。系统重启后数据恢复原样。

适用范围：项目较小

**例子：**

实验流程：

![image-20220116204745661](E:\Typora\Typora\Spring-Security\image\image-20220116204745661.png)

1.配置InMemoryUserDetailManager创建用户    及密码创建

```java
package com.bjpowernode.config;import org.springframework.context.annotation.Bean;import org.springframework.context.annotation.Configuration;import org.springframework.security.core.userdetails.User;import org.springframework.security.core.userdetails.UserDetailsService;import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;import org.springframework.security.crypto.password.PasswordEncoder;import org.springframework.security.provisioning.InMemoryUserDetailsManager;@Configurationpublic class ApplicationConfig {    @Bean    public PasswordEncoder passwordEncoder(){        return new BCryptPasswordEncoder();    }    @Bean    public UserDetailsService userDetailsService(){        PasswordEncoder passwordEncoder=passwordEncoder();//        创建内存的UserDetailService的实现类对象        InMemoryUserDetailsManager inMemoryUserDetailsManager=new InMemoryUserDetailsManager();        inMemoryUserDetailsManager.createUser(User.withUsername("admin")                .password(passwordEncoder.encode("123456"))                .roles("admin","common").build());        inMemoryUserDetailsManager.createUser(User.withUsername("cwh")                .password(passwordEncoder.encode("cwh"))                .roles("normal").build());        return inMemoryUserDetailsManager;    }}
```

2.创建类继承WebSecurityConfigurerAdapter实现configure（Http）方法

```java
package com.bjpowernode.config;import org.springframework.beans.factory.annotation.Autowired;import org.springframework.context.annotation.Configuration;import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;import org.springframework.security.config.annotation.web.builders.HttpSecurity;import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;public class MySecurityConfig extends WebSecurityConfigurerAdapter {    @Autowired    ApplicationConfig applicationConfig;    @Override    protected void configure(HttpSecurity http) throws Exception {        http.userDetailsService(applicationConfig.userDetailsService());    }}
```

3.其他和第二章的第一个例子一样     创建controller

```java
package com.guigu.controller;import org.springframework.stereotype.Controller;import org.springframework.web.bind.annotation.RequestMapping;import org.springframework.web.bind.annotation.RequestMethod;import org.springframework.web.bind.annotation.ResponseBody;@Controllerpublic class TestController {    @ResponseBody    @RequestMapping(value = "/hello",method = RequestMethod.GET)    public String hello(){        return "hello,security";    }}
```

##### ②JdbcUserDetailsManager 

作用：用户信息存放在数据库中，底层使用 jdbcTemplate 操作数据库。 可以 JdbcUserDetailsManager 中的方法完 成用户的管理

createUser ： 创建用户 

updateUser：更新用户 

deleteUser：删除用户 

userExists：判断用户是否存在



数据库文件地址：

![image-20220116202119457](E:\Typora\Typora\Spring-Security\image\image-20220116202119457.png)

users.ddl 文件

![image-20220116202134971](E:\Typora\Typora\Spring-Security\image\image-20220116202134971.png)

**例子**

此处不展示，其实就是在代码中往数据库里加入数据

不过JdbcUserDetailsManager是可以使用的，可观看动力节点SpringSecurity第33节



![image-20220117165355835](E:\Typora\Typora\Spring-Security\image\image-20220117165355835.png)



### 3.4 基于角色或权限进行访问控制

#### 3.4.0自定义认证登入页面

配置类

```java
package com.bjpowernode.config;

import com.bjpowernode.provider.MyUserDetailService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;


@Configuration
@EnableWebSecurity
public class MyWebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    @Qualifier("MyUserDetailService")
    MyUserDetailService myUserDetailService;
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(myUserDetailService).passwordEncoder(new BCryptPasswordEncoder());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                .loginPage("/login.html")//跳转到我们自己定义的“认证登入”页面
                .loginProcessingUrl("/login")//成功登入的url地址
                .defaultSuccessUrl("/test/index").permitAll()
            		//成功登入后跳转的地址   permitAll()表示这个地址允许不用认证
                .and().authorizeRequests()
                    .antMatchers("/","/test/hello","/login").permitAll()//设置哪些地址可以不用认证
                .anyRequest().authenticated()//anyRequest：其他请求   authenticated()：需要认证
                .and().csrf().disable();//关闭csrf

    }
}

```

页面

```html
<form action="/login"method="post">
用户名:<input type="text"name="username"/><br/>
密码：<input type="password"name="password"/><br/>
<input type="submit"value="提交"/>
</form>
```

**注意：页面提交方式必须为 post 请求，所以上面的页面不能使用，用户名，密码必须为 username,password** 

**原因：**

 **在执行登录的时候会走一个过滤器 UsernamePasswordAuthenticationFilter**

![image-20220119211654284](E:\Typora\Typora\Spring-Security\image\image-20220119211654284.png)

##### **如果想修改usernname和password的名字**

如果修改配置可以调用 usernameParameter()和 passwordParameter()方法。

![image-20220119211713470](E:\Typora\Typora\Spring-Security\image\image-20220119211713470.png)



##### 设置未授权的请求跳转到登录页

![image-20220119211843865](E:\Typora\Typora\Spring-Security\image\image-20220119211843865.png)



#### 3.4.1 hasAuthority 方法

如果当前的主体具有指定的权限，则返回 true,否则返回 false

**例子**

修改配置类

![image-20220119202234644](E:\Typora\Typora\Spring-Security\image\image-20220119202234644.png)

添加一个控制器

![image-20220119202257842](E:\Typora\Typora\Spring-Security\image\image-20220119202257842.png)

给用户登录主体赋予权限

![image-20220119202313484](E:\Typora\Typora\Spring-Security\image\image-20220119202313484.png)



#### 3.4.2hasAnyAuthority 方法

如果当前的主体有任何提供的角色（给定的作为一个逗号分隔的字符串列表）的话，返回 true.

**例子**

![image-20220119204445158](E:\Typora\Typora\Spring-Security\image\image-20220119204445158.png)



#### 3.4.3 hasRole 方法

如果当前的主体有任何提供的角色的话，返回 true.

修改配置文件：**注意配置文件中不需要添加”ROLE_“，因为上述的底层代码会自动添加与之进行匹配。**

但是UserDetailsService实现类需要添加**“ROLE_”**

**例子**

配置类

![image-20220119203911648](E:\Typora\Typora\Spring-Security\image\image-20220119203911648.png)

UserDetailsService实现类

![image-20220119203953734](E:\Typora\Typora\Spring-Security\image\image-20220119203953734.png)



#### 3.4.4 hasAnyRole

如果当前的主体有任何提供的角色（给定的作为一个逗号分隔的字符串列表）的话，返回 true.

**例子**

![image-20220119204431565](E:\Typora\Typora\Spring-Security\image\image-20220119204431565.png)

### 3.5自定义 403 页面

```java
http.exceptionHandling().accessDeniedPage("/takenerror.html");
```

**例子**

配置类

![image-20220119203006800](E:\Typora\Typora\Spring-Security\image\image-20220119203006800.png)

takenerror.html

![image-20220119203101030](E:\Typora\Typora\Spring-Security\image\image-20220119203101030.png)

结果

![image-20220119203043030](E:\Typora\Typora\Spring-Security\image\image-20220119203043030.png)

### 3.6注解使用

#### 3.6.1@Secured

判断是否具有角色，另外需要注意的是这里匹配的字符串需要添加前缀“ROLE_“。

**使用步骤**

1.使用注解先要开启注解功能！（可以在配置类也可以在启动类上加）

@EnableGlobalMethodSecurity(securedEnabled=true)

![image-20220120155121177](E:\Typora\Typora\Spring-Security\image\image-20220120155121177.png)

2.在控制器方法上添加注解

**这里匹配的字符串需要添加前缀“ROLE_“**

![image-20220120155149839](E:\Typora\Typora\Spring-Security\image\image-20220120155149839.png)

3.UserDetailServvice实现类

![image-20220120155227143](E:\Typora\Typora\Spring-Security\image\image-20220120155227143.png)

#### 3.6.2@PreAuthorize

判断是否具有角色，另外需要注意的是这里匹配的字符串需要添加前缀“ROLE_“。

**使用步骤**

1.使用注解先要开启注解功能！（可以在配置类也可以在启动类上加）

@EnableGlobalMethodSecurity(prePostEnabled = true)

![image-20220120155452899](E:\Typora\Typora\Spring-Security\image\image-20220120155452899.png)

2.在控制器方法上添加注解

![image-20220120155513487](E:\Typora\Typora\Spring-Security\image\image-20220120155513487.png)

3.UserDetailServvice实现类

![image-20220120155227143](E:\Typora\Typora\Spring-Security\image\image-20220120155227143.png)



### 3.7用户注销

在配置类中添加退出映射地址

```java
//        logoutUrl:退出的地址
//        logoutSuccessUrl：退出成功后的到达的地址
        http.logout().logoutUrl("/logout").logoutSuccessUrl("/out.html").permitAll();
```



### 3.8 基于数据库的记住我

这个例子没打

**SpringSecurity实现”记住我“的一个原理**

![image-20220120162225474](E:\Typora\Typora\Spring-Security\image\image-20220120162225474.png)

**实现过程**

![image-20220120164310699](E:\Typora\Typora\Spring-Security\image\image-20220120164310699.png)

创建表

```sql
CREATE TABLE `persistent_logins` (
 `username` varchar(64) NOT NULL,
 `series` varchar(64) NOT NULL,
 `token` varchar(64) NOT NULL,
 `last_used` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE 
CURRENT_TIMESTAMP,
 PRIMARY KEY (`series`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

添加数据库的配置文件

```properties
spring:
datasource:
driver-class-name: com.mysql.jdbc.Driver
url: jdbc:mysql://192.168.200.128:3306/test
username: root
password: root
```

 编写配置类

```java
@Configuration
public class BrowserSecurityConfig {
@Autowired
private DataSource dataSource;
@Bean
public PersistentTokenRepository persistentTokenRepository(){
 JdbcTokenRepositoryImpl jdbcTokenRepository = new 
JdbcTokenRepositoryImpl();
// 赋值数据源
jdbcTokenRepository.setDataSource(dataSource);
// 自动创建表,第一次执行会创建，以后要执行就要删除掉！
jdbcTokenRepository.setCreateTableOnStartup(true);
return jdbcTokenRepository;
 }
}

```

修改安全配置类

```java
@Autowired
private UsersServiceImpl usersService;
@Autowired
private PersistentTokenRepository tokenRepository;
// 开启记住我功能
http.rememberMe()
 .tokenRepository(tokenRepository)
 .userDetailsService(usersService);

```

页面添加记住我复选框

```html
记住我：<input type="checkbox"name="remember-me"title="记住密码"/><br/>
```

**此处：name 属性值必须位 remember-me.不能改为其他值(框架规定)**

默认 2 周时间。但是可以通过设置状态有效时间，即使项目重新启动下次也可以正常登 录。 在配置文件中设置

![image-20220120165107048](E:\Typora\Typora\Spring-Security\image\image-20220120165107048.png)

