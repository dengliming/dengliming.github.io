title: nginx之正向代理
date: 2016-05-08 15:00:51
tags:
- nginx
- proxy

---
## 前言
nginx不仅可以来做反向代理，也可以用来做正向代理(透明代理,代理上网)，首先先来了解下什么是正向代理和反向代理以及两者之间的区别。<!-- more -->

## 正文
### 正向代理
正向代理是一个位于客户端和原始服务器之间的服务器， 从而为客户端从原始服务器中取得所需要的数据。客户端向代理服务器发送一个请求，并且写明了地址。之后代理向原始服务器转交并且将获得的内容返回给客户端。客户端必须要进行一些特别的设置才能使用正向代理服务器
 这里引用一个篇博文的例子
```html
A用户无法访问twitter，但是我能访问B服务器，而B服务器可以访问twitter。于是我访问B服务器，告诉它”嗨，伙计，我要访问twitter“，B服务器收到请求后，去访问twitter，twitter把响应信息返回给B服务器，B服务器再把响应信息返回给A。这样，通过B代理服务器，就实现了翻墙。
```

### 反向代理
反向代理正好相反。对于客户端来说，反向代理就好像原始服务器。并且客户端不需要进行任何设置。客户端向reverse proxy中的name-space发送请求，接着反向代理判断请求走向何处，并将请求转交给客户端。这些内容就好似他自己一样。

### 两者区别
1. 从用途上来区分：
正向代理：正向代理用途是为了在防火墙内的局域网提供访问internet的途径。另外还可以使用缓冲特性减少网络使用率
反向代理：反向代理的用途是将防火墙后面的服务器提供给internet用户访问。同时还可以完成诸如负载均衡等功能
2. 从安全性来讲：
正向代理：正向代理允许客户端通过它访问任意网站并且隐蔽客户端自身，因此你必须采取安全措施来确保仅为经过授权的客户端提供服务
反向代理：对外是透明的，访问者并不知道自己访问的是代理。对访问者而言，他以为访问的就是原始服务器

### 配置相关示例
下面主要说下nginx正向代理配置
window下测试的配置：
```xml
#user  nobody;
worker_processes  1;
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#pid        logs/nginx.pid;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
	                  '$http_host $request_uri '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  logs/access.log  main;
    sendfile        on;
    #tcp_nopush     on;
    #keepalive_timeout  0;
    keepalive_timeout  65;
    #gzip  on;
    server{
        resolver 8.8.8.8;
        resolver_timeout 30s;
        listen 80;
        location / {
                proxy_pass http://$http_host$request_uri;
                proxy_set_header Host $http_host;
                proxy_buffers 256 4k;
                proxy_max_temp_file_size 0;
                proxy_connect_timeout 30;
                proxy_cache_valid 200 302 10m;
                proxy_cache_valid 301 1h;
                proxy_cache_valid any 1m;
        }
}
}
```

实际就是获取$http_host$request_uri变量值重新拼接用户访问的url,当然以上配置尚未支持https。
启动nginx之后。下面给出java访问示例：
```java
public static void t1() throws Exception {
		BufferedReader in = null;
		String content = null;
		try {
			// 定义HttpClient
			HttpClient client = new DefaultHttpClient();
			// 实例化HTTP方法
			HttpGet request = new HttpGet("http://www.ruanyifeng.com/blog/2010/02/url_encoding.html");
			// 依次是代理地址，代理端口号，协议类型
			HttpHost proxy = new HttpHost("127.0.0.1", 80);
			client.getParams().setParameter(ConnRoutePNames.DEFAULT_PROXY, proxy);
			HttpResponse response = client.execute(request);
			in = new BufferedReader(new InputStreamReader(response.getEntity().getContent()));
			StringBuffer sb = new StringBuffer("");
			String line = "";
			String NL = System.getProperty("line.separator");
			while ((line = in.readLine()) != null) {
				sb.append(line + NL);
			}
			in.close();
			content = sb.toString();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (in != null) {
				try {
					in.close();// 最后要关闭BufferedReader
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
			// return content;
			System.err.println(content);
		}
	}
```
以上代码是通过本地80端口配置的代理访问http://www.ruanyifeng.com/blog/2010/02/url_encoding.html  ps：代码依赖HttpClient
的jar包
这里就不贴程序打印的结果了。查看下nginx的access.log就知道是否成功访问
```xml
127.0.0.1 - - [07/May/2016:21:56:27 +0800] "GET /blog/2010/02/url_encoding.html HTTP/1.1" 200 99394 "-" www.ruanyifeng.com /blog/2010/02/url_encoding.html "Apache-HttpClient/4.3.6 (java 1.5)" "-"
```

也可以通过以下代码访问：
```java
public static void test() throws Exception {
		BufferedReader in = null;
		String content = null;
		try {
			// 定义HttpClient
			HttpClient client = new DefaultHttpClient();
			// 实例化HTTP方法
			HttpGet request = new HttpGet("http://127.0.0.1/blog/2010/02/url_encoding.html");
			request.setHeader("Host", "www.ruanyifeng.com");
			HttpResponse response = client.execute(request);
			in = new BufferedReader(new InputStreamReader(response.getEntity().getContent()));
			StringBuffer sb = new StringBuffer("");
			String line = "";
			String NL = System.getProperty("line.separator");
			while ((line = in.readLine()) != null) {
				sb.append(line + NL);
			}
			in.close();
			content = sb.toString();
		} finally {
			if (in != null) {
				try {
					in.close();// 最后要关闭BufferedReader
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
			// return content;
			System.err.println(content);
		}
	}
```
效果是一样的。


参考：
> * [正向代理、反向代理、透明代理][1]
> * [nginx正向代理配置][2]

  [1]: http://github.thinkingbar.com/reverseProxy/
  [2]: http://my.oschina.net/duxuefeng/blog/275179


