# 使用SpringBoot进行应用开发

#### 1.基于maven进行开发的简单步骤

1.创建一个简单的maven项目(不选模板)
2.编写pom文件，引用父工程和依赖
```
    <!--    1.引用父工程-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.4</version>
    </parent>

    <!--    2.根据开发需求导入依赖-->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```
3.编写一个MainApplication类

```java
package com.cimb.boot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * 1.创建一个类，类名随意
 * 2.添加@SpringBootApplication注解，标识这是一个springboot应用，该类即主程序类
 *
 * @author Administrator
 */
@SpringBootApplication
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }
}

```

4.此时springboot项目已经创建完成，可以添加一个controller类进行测试


```
package com.cimb.boot.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


/**
 * @Controller 表示这是一个控制层
 * @ResponseBody 表示这个类的每一个方法都是直接返回给浏览器的数据，而不是跳转页面
 * @RestController 整合上面的两个注解
 */
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String handleHello() {
        return "Hello,SpringBoot2";
    }
}
```
	打开浏览器，访问 `localhost:8080/hello` 即可看到项目的运行结果。


5.要对项目进行配置，只需要在resource目录下创建`application.properties`文件，即可在文件里面对项目的所有可配置项进行配置。

6.要对项目进行部署，只需要在pom.xml中引入`fat jar`,

```
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```
即可通过maven-install将程序打包成可执行jar包。

