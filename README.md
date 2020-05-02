# springdb

## 创建SQL文件用于设置用户与数据库


```
CREATE USER 'springuser'@'localhost' IDENTIFIED BY '!Spring123';
CREATE DATABASE springdb;
GRANT ALL ON springdb.* TO 'springuser'@'localhost';

```
## 创建springboot项目
主要依赖：
- web
- jpa
- mysql
- thymeleaf

一共四个依赖, 全是Spring Boot官方支持的库, 知识点讲解:

### Thymeleaf
是现代服务端的Java模板引擎, 我们用它来生成HTML页面.
### JPA 
是Java Persistence API, 也就是Java持久层API, 它的本质是一套将Java的对象映射到数据库中的关系表的标准, 而Spring-Boot的JPA依赖包含一个重要子依赖, 你一定听过它的名字: Hibernate. 它是JPA的具体实现, 也是Spring Boot的默认JPA实现.官方文档相关知识点阅读
### MySQL 
是用来实现从Java到MySQL连接的一个中间件.
### Web 
是Spring Boot重要核心组件, 网络应用的必须品, 它包含了Tomcat容器, Spring MVC等核心组件.

所以我们也可以看到Spring Boot其实相当于一个依赖打包器, 比如网络模块, 大家都需要Tomcat容器, 也需要Spring MVC框架, 那索性就放到一个包里, 也就是Web包, 这样一个依赖就解决了问题.

## 配置application.properties文件

pplication.properties文件的作用有点类似于pom, 但也不太一样, pom是管理你应用和其他库的依赖关系, 而application.properties则是去设置, 或是配置这些依赖, 是Spring应用的重要组成部分.

Spring Boot官方文档推荐阅读

该文件可以在maven工程的`src/main/java/resources/applicatio.properties`下找到.

在该文件中输入如下属性:

```
spring.datasource.username=niudai
spring.datasource.password=niudai
spring.datasource.url=jdbc:mysql://localhost:3306/springdb
spring.jpa.hibernate.ddl-auto=create
```
- 第三行的url就是我们数据库的地址, 3306是MySQL默认的本地端口, 而springdb正是我们之前创建的数据库.
- 
- 第一行和第二行声明了我们之前创建的用户名密码和账户名, 都为niudai
- 
- 第四行的create为开发模式, 就是每次应用启动都重新创建一个新表, 原有的表会清空, 开发结束后可以将其设置为none.
- 
- 声明了这些之后, 启动应用后, Spring会自动用上述的信息连接并登陆你的MySQL中名为springdb的数据库.

## 创建Entity
加了@Entity注解, Spring框架会得知这个类是一个Entity, Hibernate会把这个类映射到数据库中的关系表, 这个关系表的列与User的所有属性一一对应.

相当于SQL语言中的:

```
CREATE TABLE User(id int, name varchar(255), email varchar(255), password varchar(255));
```
对应实现代码为：

```java
package com.dblink.demo;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class User {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Integer id;
    private String name;
    private String email;
    private String password;

    @Override
    public String toString() {
        return "{" +
                " id='" + getId() + "'" +
                ", name='" + getName() + "'" +
                ", email='" + getEmail() + "'" +
                ", password='" + getPassword() + "'" +
                "}";
    }

    public String getPassword() {
        return this.password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Integer getId() {
        return this.id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return this.email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```
### Controller
Controller是接受用户的url请求, 将url映射到某个具体的方法, 处理url请求, 再返回一个Model的东西, 是Spring MVC中的C层.

知识点: Spring MVC中的三层分别是Model, View, Controller. 当用户向你的服务器发送了请求后, 比如HTTP请求, Controller先将请求映射到某个方法上, 方法根据该请求进行相应处理, 返回Model, 这个Model可以被理解成一个抽象的数据模型, 它储存了网页必须包含的信息, 但是它不是网页本身, Model会被送到View层, 也就是用户界面层, View层将自己本身的模板和Model内部的数据结合成为完整的一个页面, 作为response返还给用户, 用户便看到了你的页面, 但是现在随着前后端的分离, View层的意义已经不大,作为后端开发, 主要专注于Model和Controller.

知识点讲解:

@AutoWired 被注有它的注解会被Spring得知, Spring会自动为你注入依赖. 比如上述的userRepository, Spring在运行过程中会通过IoC容器为你注入一个UserRepository的实例给userRepository. 相关知识点: 依赖注入、设计模式.
@GetMapping 将使用Get方法的HTTP请求映射到被注的方法. 相关知识点: HTTP请求, HTTP方法.
@RequestParam 将HTTP请求中用户传入的参数映射到变量中括号内指定变量中. 相关知识点: HTTP参数
@ResponseBody 它表示该方法返回的值就是Response本身, 不需传递至View被渲染, 用户直接得到该方法的返回值.
其余讲解在代码内的注解中.

创建一个文件: UserController.java:

```
package com.dblink.demo;

import java.util.List;

import com.dblink.demo.User;
import com.dblink.demo.UserRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class UserController {
    private static final Logger log = LoggerFactory.getLogger(UserController.class);

    @Autowired
    private UserRepository userRepository;

    @GetMapping(path="/add") // Map ONLY GET REQUESTs.
    public @ResponseBody String addNewUser (@RequestParam String name
            , @RequestParam String email, @RequestParam String password, User user) {
        // @ResponseBody means the returned String is a response, not a view name.
        user.setName(name);
        user.setEmail(email);
        user.setPassword(password);
        userRepository.save(user);
        log.info(user.toString()+" saved to the repo");
        return "Saved";
    }

    /**
     * 登陆方法, 用户输入邮箱和密码, 查询数据库检验是否有该账户,如果有,
     * 返回原先页面 ,登陆成功。
     * @param email 用户邮箱
     * @param password 用户密码
     * @param model Spring MVC中的Model，用来储存经过controller处理后的信息，再由View层渲染
     *         得到前端页面。
     * @return
     */
    @GetMapping(path = "/login")
    public String login(@RequestParam String email, @RequestParam String password, Model model) {
        List<User> users = userRepository.findByEmail(email);
        // 如果数据库中未查到该账号:
        if (users == null) {
            log.warn("attempting to log in with the non-existed account");
            return "该用户不存在";
        } else {
            User user = users.get(0);
            if (user.getPassword().equals(password)) {
                // 如果密码与邮箱配对成功:
                model.addAttribute("name", user.getName());
                log.warn(user.toString()+ " logged in");
            } else {
                // 如果密码与邮箱不匹配:
                model.addAttribute("name", "logging failed");
                log.warn(user.toString()+ " failed to log in");
            }
            return "index";
        }
    }

    /**
     * 查看所有用户的注册信息，按照Spring Boot的设定，以Json的形式输送给用户端。
     * @return
     */
    @GetMapping(path="/all")
    public @ResponseBody Iterable<User> getAllUsers() {
        return userRepository.findAll();
    }

    /**
     * 域名的根目录，然后返回的“index”会映射到
     * java/resources/templates/index.html文件。
     * @param name
     * @return
     */
    @GetMapping(path="/")
    public String welcomePage(@RequestParam(name="name", required=false, defaultValue="World")
                                      String namel){
        return "index";
    }


}
```
如果说User代表每个用户, 或是用户的信息, 那么UserRepository就代表储存这些用户的"库", 我们创建一个UserRepository.java, 注意它是一个接口, 也就是Interface, 而不是Class.

你可能会疑惑这只是一个接口, 并没有具体实现, 如何完成对User信息的储存, 但事实上你只需声明这个接口, 剩下的交给Spring, 它会自动为你进行它的实现.

该类继承了CrudRepository, 也就是支撑"增删改查"的一个Repository, 你可以在别的地方直接调用这个接口的方法, 方法名有规范, 比如你想通过邮箱查找, 那你就使用findByEmail(String email)方法, Spring会自动帮你将findByEmail转换成SQL语言中的`SELECT * FROM UserRepository WHERE email = 'email'

```
package com.dblink.demo;

import java.util.List;

import com.dblink.demo.User;
import org.springframework.data.repository.CrudRepository;

public interface UserRepository extends CrudRepository<User, Integer> {
    List<User> findByEmail(String email);
    void deleteByEmail(String email);
}
```

在resources/templates/目录下创建一个index.html文件, 它便是我们应用的入口界面.

```
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">

<head>
    <title>MAC登陆注册系统</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <link rel="stylesheet" href="index.css">
</head>

<body>
<h2>MAC内部注册系统: </h2>

<p th:text="'欢迎 ' + ${name} + '!'" />
<div class="add">
    <h2>注册新账号</h2>
    <form action="/add" method="GET">
        用户名:<input type="text" name="name"><br>
        邮箱:<input type="text" name="email"><br>
        密码:<input type="password" name="password"><br>
        <input type="submit" value="注册">
    </form>
</div>
<div class="login">
    <h2>登陆已有账号</h2>
    <form action="/login" method="GET">
        邮箱:<input type="text" name="email"><br>
        密码:<input type="password" name="password"><br>
        <input type="submit" value="登陆">
    </form>
</div>
<div class="all">
    <h2><a href="/all">查看所有注册账号信息</a></h2>
</div>
</body>

</html>
```
