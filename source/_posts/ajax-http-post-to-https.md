title: ajax http post to https
date: 2016-01-16 22:19:51
tags:
- ajax
- http
- https
categories: ajax

---
在本地同一个域http页面请求https页面，其实已经跨域了，我们知道出于安全问题的考虑，浏览器是不允许跨域访问的，那么如果使用ajax来实现跨域访问呢，开发中常用的解决方法很多，比较常用的是JSONP方法、window.name，JSONP方法是一种非官方方法，而且这种方法只支持GET方式，不如POST方式安全。即使使用jquery的jsonp方法，type设为POST，也会自动变为GET

经查资料：
如果跨域使用POST方式，可以使用创建一个隐藏的iframe来实现，但这样会比较麻烦
但是通过服务端设置Access-Control-Allow-Origin来实现跨越访问比较简单

在这里只对Access-Control-Allow-Origin做个小测试
先新建两个jsp页面，一个是test.jsp用来发送请求，accept.jsp用来接收请求
test.jsp
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<script src="js/jquery.min.js" type="text/javascript"></script>
<title>Insert title here</title>
</head>
<body>
	<form id="form">
		<input name="t" value="123"/>
		<input type="button" value="submit" onclick="ajaxSubmit(this)"/>
	</form>
</body>
<script>
	function ajaxSubmit(frm) {
		$.ajax({
		    url: "/accept.jsp",
		    type: "POST",
		    data: $("#form").serialize(),
		    success: function(result) {
		    	console.log(result);
		    }
		});
	}
</script>
</html>
```
accept.jsp
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%
	out.println("success-------->");
%>
```
这里accept页面简单的输出success

在浏览器上看看页面效果：
![](http://7xjw47.com1.z0.glb.clouddn.com/78892217.png)
点击submit之后 查看浏览器控制台 正常返回seccess
![](http://7xjw47.com1.z0.glb.clouddn.com/78924244.png)

那么假设把test.jsp中form的提交的url改成https之后呢。PS：具体本地怎么测试https请参考另一篇[本地openssl生成证书](https://dengliming.github.io/2016/01/16/openssl-generate-certificate/)

```jsp
function ajaxSubmit(frm) {
		$.ajax({
		    url: "https://localhost/accept.jsp",
		    type: "POST",
		    data: $("#form").serialize(),
		    success: function(result) {
		    	console.log(result);
		    }
		});
	}
```

再看看浏览器效果：出现No 'Access-Control-Allow-Origin' header is present on the requested resource
![](http://7xjw47.com1.z0.glb.clouddn.com/79044598.png)

然后在accept.jp加入headerAccess-Control-Allow-Origin

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%
	response.setHeader("Access-Control-Allow-Origin", "*");
	out.println("success-------->");
%>
```

* 表示允许任何域名跨域访问 一般线上不这么用，会指定某个域名
最后来看下效果：请求成功返回
![](http://7xjw47.com1.z0.glb.clouddn.com/79534410.png)
