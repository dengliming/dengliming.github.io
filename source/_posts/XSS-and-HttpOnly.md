title: XSS与HttpOnly
date: 2015-12-27 21:12:51
tags:
- HttpOnly
- XSS
categories: notes
---

1.什么是HttpOnly?

如果您在cookie中设置了HttpOnly属性，那么通过js脚本将无法读取到cookie信息，这样能有效的防止XSS攻击，具体一点的介绍请google进行搜索

2.HttpOnly的设置样例

```java
response.setHeader("Set-Cookie", "cookiename=value;
Path=/;Domain=domainvalue;Max-Age=seconds;HTTPOnly");
```
具体参数的含义再次不做阐述，设置完毕后通过js脚本是读不到该cookie的，但使用如下方式可以读取

```java
Cookie cookies[]=request.getCookies();
```

下面测试一下：
```java
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
<%
	response.setHeader("Set-Cookie",
			"sessionId=abc;Path=/;Max-Age=20*60*1000;");
%>
</head>
<body>
	<script>
		alert(getCookie('sessionId'));
		//读取cookies
		function getCookie(name) {
			var nameEQ = name + "=";
			var ca = document.cookie.split(';');
			for (var i = 0; i < ca.length; i++) {
				var c = ca[i];
				while (c.charAt(0) == ' ')
					c = c.substring(1, c.length);
				if (c.indexOf(nameEQ) == 0)
					return c.substring(nameEQ.length, c.length);
			}
			return null;
		}
	</script>
</body>
</html>
```
运行结果为： abc
![](http://7xjw47.com1.z0.glb.clouddn.com/2015122702.png)

下面改成HttpOnly看看
```java
response.setHeader("Set-Cookie",
			"sessionId=abc;Path=/;Max-Age=20*60*1000;HttpOnly");
```
运行结果为：null
![](http://7xjw47.com1.z0.glb.clouddn.com/2015122701.png)

