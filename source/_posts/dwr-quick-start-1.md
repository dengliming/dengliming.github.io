title: DWR快速入门（一）
date: 2016-02-26 23:00:51
tags:
- dwr
categories: dwr

---
## 前言
DWR(Direct Web Remoting)是一个WEB远程调用框架.利用这个框架可以让AJAX开发变得很简单.利用DWR可以在客户端利用JavaScript直接调用服务端的Java方法并返回值给JavaScript就好像直接本地客户端调用一样
<!-- more -->
## 快速开始

### 1.下载DWR JAR 文件
这里直接使用maven下载
```xml
<dependency>
    <groupId>org.directwebremoting</groupId>
    <artifactId>dwr</artifactId>
    <version>3.0.1-RELEASE</version>
</dependency>
```

### 2.下载Commons Logging JAR
```xml
<dependency>
    <groupId>commons-logging</groupId>
	<artifactId>commons-logging</artifactId>
	<version>1.2</version>
</dependency>
```

### 3.web.xml配置DWR servlet
在项目WEB-INF/web.xml里面添加以下配置
```java
<servlet>
  <display-name>DWR Servlet</display-name>
  <servlet-name>dwr-invoker</servlet-name>
  <servlet-class>org.directwebremoting.servlet.DwrServlet</servlet-class>
  <init-param>
     <param-name>debug</param-name>
     <param-value>true</param-value>
  </init-param>
</servlet>
<servlet-mapping>
  <servlet-name>dwr-invoker</servlet-name>
  <url-pattern>/dwr/*</url-pattern>
</servlet-mapping>
```

### 4.编写自定义java bean
```java
package com.test.bean;
public class Demo {
	public String sayHello(String name) {
		return "hello" + name ;
	}
}
```

### 5.创建dwr配置文件
在WEB-INF下创建dwr.xml,配置如下：
```xml
<!DOCTYPE dwr PUBLIC
    "-//GetAhead Limited//DTD Direct Web Remoting 3.0//EN"
    "http://getahead.org/dwr/dwr30.dtd">
<dwr>
  <allow>
    <create creator="new" javascript="JDate">
      <param name="class" value="java.util.Date"/>
    </create>
    <create creator="new" javascript="Demo">
      <param name="class" value="com.test.bean.Demo"/>
    </create>
  </allow>
</dwr>
```

### 6.启动应用
启动应用并在浏览器访问http://localhost/dwr/
![](/images/15056296.png)

具体怎么在我们的应用中利用他们来编写javascript呢?
点击Demo进入以下页面：
![](/images/15094875.png)

这个页面介绍了我们自定义的Javabean以及怎么通过javascript来调用，从内容中可以知道只需要在自己编写的html或者jsp页面中引入
 <script type='text/javascript' src='/dwr/engine.js'></script>
  <script type='text/javascript' src='/dwr/interface/Demo.js'></script>
就可以调用Demo类中的方法了

### 7.编写测试html TestDWR.html
```html
<!doctype html>
<html>
	<head>
		<script type="text/javascript" src="dwr/engine.js"></script>
		<script type="text/javascript" src="dwr/interface/Demo.js"></script>
	</head>
<body>
</body>
<script>
	Demo.sayHello('dwr', {
		  callback: function(str) {
		  	alert(str);
		  }
	});
</script>
</html>
```
至此可以看到页面js成功调用Demo类的sayHello方法返回字符串并弹窗显示，是不是有点意思。今天先搞个小demo，有时间接着深入理解原理，呵呵。




参考：
> * [DWR官网入门教程][1]

  [1]: http://directwebremoting.org/dwr/introduction/getting-started.html


