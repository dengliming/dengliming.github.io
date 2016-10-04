title: 消息队列之HTTPSQS
date: 2016-07-27 21:00:51
tags:
- HTTPSQS
---

##HTTPSQS是什么?
一款基于 HTTP GET/POST 协议的轻量级开源简单消息队列服务，
使用 Tokyo Cabinet 的 B+Tree Key/Value 数据库来做数据的持久化存储。如需了解更多直接参考[张宴博客](http://zyan.cc/httpsqs/).

##安装
此次安装是在centos6.4系统上操作的
安装可以直接参考[张宴博客](http://zyan.cc/httpsqs/)
下面是我安装过程出现的一些问题：

![](http://7xjw47.com1.z0.glb.clouddn.com/13449719.png)


安装gcc编译器即可
```shell
yum install gcc 
```
若出现configure: error: zlib.h is required
```shell
yum install zlib-devel
```

最后启动
```shell
httpsqs -d -p 1218 -x /data0/queue
```
请使用命令“killall httpsqs”、“pkill httpsqs”和“kill cat/tmp/httpsqs.pid”来停止httpsqs。

测试下入队列
http://127.0.0.1:1218/?name=my_queue&opt=put&data=经过URL编码的文本消息&auth=mypass123
![](http://7xjw47.com1.z0.glb.clouddn.com/15645496.png)

接下来使用java客户端操作下（需下载httpsqs4j.jar）:
```java
package httpsqs;
import com.daguu.lib.httpsqs4j.Httpsqs4j;
import com.daguu.lib.httpsqs4j.HttpsqsClient;
import com.daguu.lib.httpsqs4j.HttpsqsStatus;
public class Test {
	public static void main(String[] args) {
		try {
			Httpsqs4j.setConnectionInfo("192.168.1.11", 1218, "UTF-8");
			HttpsqsClient client = Httpsqs4j.createNewClient();
			HttpsqsStatus status = client.getStatus("my_queue");
			System.out.println(status.version);
			System.out.println(status.queueName);
			System.out.println(status.maxNumber);
			System.out.println(status.getLap);
			System.out.println(status.getPosition);
			System.out.println(status.putLap);
			System.out.println(status.putPosition);
			System.out.println(status.unreadNumber);
			//client.putString("test", "test");
			System.out.println(client.getString("my_queue"));
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```
运行结果：
```java
1.7
my_queue
1000000
1
1
1
0
1
test
```

##总结
VMware使用桥接方式
/etc/init.d/iptables stop关闭防火墙


##参考
参考：
> * [张宴博客][1]
[1]: http://zyan.cc/httpsqs/