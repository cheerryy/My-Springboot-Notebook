### 0 什么是SpringBoot
SpringBoot通过整合Spring技术栈，简化了Spring开发。

SpingBoot特点——微服务：

 - 是一种架构风格
 - 服务应该是一组小型服务的组合（功能元素动态组合，比如在一个服务器多放点A，少放点B服务）
 - 每个服务都可替换可升级
 - 每个服务可以通过http方式沟通



微服务与单体应用 （all in one）相对。单体应用具有以下优点：
- 开发测试简单
- 部署简单（整个打包成war包即可）
- 拓展简单（提高并发只要相同应用复制到多个服务器即可）

但缺点在于每次修改都要重新部署。

----------------
接下来了解如何从0开始用SpringBoot框架搭建一个HelloWorld程序

目标功能：
> 浏览器发送hello请求，服务器接收请求并处理，给浏览器响应一个hello world字符串

### 1 jdk 与jre安装
[JDK1.8下载、安装和环境配置教程](https://blog.csdn.net/weixin_44084189/article/details/98966787)
该文章同时内含jre安装

### 2 maven安装
[Window系统下的Maven3.3.9安装](https://blog.csdn.net/qq_42881421/article/details/82900849)

### 3 Idea安装
[软件安装管家-idea2019安装教程](https://mp.weixin.qq.com/s/Gh7oVK2K7X2WY6nKACGXiQ)

### 4 SpringBoot安装
SpringBoot不需要手动下载，只需要后续在文件中配置即可自动下载

### 5 修改Maven的setting配置
打开maven的安装目录下`conf\setting.xml`，如`D:\Maven\apache-maven-3.3.9\conf\setting.xml`文件，在`<profiles>`标签内增加下面这段并保存。

```xml
<profile>
    <id>jdk-1.8</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    <properties>
      <maven.compiler.source>1.8</maven.compiler.source>
      <maven.compiler.target>1.8</maven.compiler.target>
      <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
  </profile>
```

### 6 修改Idea中的Maven配置
打开Idea的首页-右下角的Configure-左边找到Maven项

 1. 将Maven home directionory设置为Maven安装目录
 2. 将User setting file设置为第5步修改过的`setting.xml`文件，勾选Override
 3. 将Local repository设置为`maven安装路径\repository`（本来maven安装路径里是没有repository文件夹的，是自己写的路径，表示到时候仓库就放这）
![idea的maven配置](https://img-blog.csdnimg.cn/20201004102633118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)



### 7 开始SpringBoot操作
#### 7.1 创建一个maven工程

 1. 打开idea，new project，选择左侧maven
 2. 修改上方的project sdk为自己安装jdk的目录，next![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004103004858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)

 3. 设置项目名称和存储地址，finish![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004103051877.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)
4. 右下角 启动自动导入，这样在pom文件里面每写一个依赖就会自动加入相关依赖
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004103252322.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)


#### 7.2 导入SpringBoot相关依赖
去Spring官网，SpringBoot的Quick Start，复制依赖（即下面这段），粘贴到一开始就打开的pom文件里面（`<project>`里面，`<version>`下面即可），由于是自动导入，所以会开始下载


```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>1.5.9.RELEASE</version>
</parent>
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>
```
当外部库出现新的maven库就成功了。

但是我用的时候这段没有自动开始下载，改用下面的才成功了，其实理论上导入`spring-boot-starter-web`应该就行了，不清楚原因。

```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>1.5.9.RELEASE</version>
</parent>
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

最终效果如下：![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004103610577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)



#### 7.3 编写Main主程序
用于启动springboot应用
###### 7.3.1 新建Main类
在左侧Src-Main-java文件夹，右键-新建Java class-起名`HelloWorldMainApplication`或者`com.xxx.HelloWorldMainApplication`(这样更好，是放到了包下面，这样起名idea会自动创建对应的包)，最终效果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004103848751.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)

##### 7.3.2 加入注解
在package（如果上一步写的是`com.xxx`的话）和`pulic class`之间写上`import`和`@SpringBootApplication`用于说明是springboot应用。

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloWorldMainApplication{

}
```

##### 7.3.3 写Main class内容
在class里写

```java
@SpringBootApplication
public class HelloWorldMainApplication{
	public static void main(String[]args){
		SpringApplication.run(HelloWorldMainApplication.class,args);
	}
}
```

这里遇到 `cannot resolve method 'run'`的错误，经查询可能是maven包安装问题，尝试以下2个方法后成功。
1. [清空缓存](https://www.cnblogs.com/my-program-life/p/12019231.html)

2. 在cmd下进入项目的根目录下，执行以下代码，用于清理所有的依赖并重新安装
	`mvn dependency:purge-local-repository`



#### 7.4 编写Controller和Service
##### 7.4.1 新建Controller类
在`com.xxx`的包下新建Java类，命名为`controller.HelloController`，则会创建一个controller包，下面有`HelloController`类

##### 7.4.2 加入注解和import
同Main类一样，在Controller类里面加入以下代码：
```java
import org.springframework.stereotype.Controller;
        
@Controller
```

但是由于刚刚更换了maven包，导致dependency没有及时更新，所以出现了import时springframework后面没有stereotype包的问题。

解决：将pom文件重新导入reimport即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004105036767.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)

##### 7.4.3 写controller代码

```java
@Controller
public class HelloController {
    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World";
    }
}
```

`@RequestMapping("/hello")`意为接收浏览器的hello请求
`return "Hello World"`意为Controller向浏览器返回一个"Hello World"字符串

#### 7.5 效果测试
测试方法：
1. 运行主程序的main方法
即点main函数（而非class）旁边的绿色按钮-选择第一个选项
控制台会打印信息，如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100410534760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)


说明tomcat在8080端口已经启动

2. 打开浏览器，输入`localhost:8080`，默认出现白色界面如下，不用管
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004105420419.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)
3. 浏览器输入改成`localhost:8080/hello`表示浏览器发出hello请求
4. 浏览器出现“Hello World”字样，则成功
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004105528597.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)



5. 点击这里可停止该应用
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100410555049.png#pic_center)



#### 7.6 将项目打jar包
##### 7.6.1 加入用于打包的插件

在pom中增加以下代码（用于增加插件）：

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

这里遇到`spring-boot-maven-plugin`出现 `not found` 的错误

解决方法：
1. 据说该包不是maven内部的，所以找不到，因此尝试[在pom和maven\setting里面增加配置](https://blog.csdn.net/qq_38068435/article/details/105895890)，仍失败
2. [参考此方法](https://www.cnblogs.com/vevy/p/12246679.html)写plugin时加上`<version>`标签再reimport一下pom（注意，每次修改完要把pom reimport一下 ，不然不会修改），成功。
	```xml
	<version>2.2.2.RELEASE</version>
	```


##### 7.6.2 开始打包

双击运行package方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004110317602.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)

从控制台可以看到，jar包的位置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004110332588.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)


从文件夹也可看到
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004110342525.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)

##### 7.6.3 运行jar包
现在可以直接在命令行用jar命令启动

 1. cd到jar的目录
 2. 用Java -jar命令运行
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004110455227.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)
浏览器显示如下，运行成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004110506767.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)

### 8 附录：代码汇总
似乎增加@会自动import，而且中途pom文件增加了好几个依赖，因此将完整代码附到下面。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004110737805.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4MzgxNQ==,size_16,color_FFFFFF,t_70#pic_center)
`HelloWorldMainApplication.java`:

```java
package com.atguigu;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.*;


@SpringBootApplication
public class HelloWorldMainApplication {

    public static void main(String[] args) {

        SpringApplication.run(HelloWorldMainApplication.class,args);
    }

}
```

`HelloController.java`:
```java
package com.atguigu.controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {
    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World";
    }
}
```


`pom.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>spring-boot-01-helloworld</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.2.2.RELEASE</version>
            </plugin>
        </plugins>
    </build>
    <pluginRepositories>
        <pluginRepository>
            <id>alimaven spring plugin</id>
            <name>alimaven spring plugin</name>
            <url>https://maven.aliyun.com/repository/spring-plugin</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```


