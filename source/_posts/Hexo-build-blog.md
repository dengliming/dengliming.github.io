title: Hexo搭建博客
date: 2015-06-21 21:19:51
tags:
- hexo
categories: notes

---

# 今天终于开通了我的博客，这是一个全新的开始。

下面是我搭建独立博客的过程

准备工作：
Nodejs安装
Git安装
Hexo配置

nodejs环境配置可参考资料[nodejs环境配置](http://www.cnblogs.com/bicabo/p/3630777.html)

Hexo安装，要用全局安装，加-g参数

> E:\> npm install -g hexo

如果执行这个命令出现
npm WARN optional dep failed, continuing fsevents@0.3.6
则改为使用
>  E:\>npm install --unsafe-perm --verbose -g hexo

安装成功之后可查看hexo版本

> E:\>hexo version
hexo-cli: 0.1.7
os: Windows_NT 6.1.7601 win32 x64
http_parser: 1.0
node: 0.10.32
v8: 3.14.5.9
ares: 1.9.0-DEV
uv: 0.10.28
zlib: 1.2.3
modules: 11
openssl: 1.0.1i

安装好后，我们就可以使用Hexo创建项目了

> E:\nodebook>hexo init blog
INFO  Copying data to E:\nodebook\blog
INFO  You are almost done! Don't forget to run 'npm install' before you start blogging with Hexo!


我们看到当前在目录下，出现了blog文件夹，包括初始化的文件。
进入blog目录，启动Hexo服务器（启动之前需执行npm install）

> E:\nodebook>cd blog
>  E:\nodebook>npm install
> E:\nodebook>hexo server


启动成功后可以在浏览器输入：http://localhost:4000/ 即可访问

接下来发布到github

 - 需要在[github](https://github.com/)上注册一个账号
 - 建立与你用户名对应的仓库，仓库名必须为your_user_name.github.com
 - 添加ssh公钥

 前两步比较简单，最终建成的仓库如下：
 ![](http://7xjw47.com1.z0.glb.clouddn.com/201506215.png)

第三步：添加ssh公钥

输入以下指令（邮箱就是你注册Github时候的邮箱）

> ssh-kengen -t rsa -C "1196767447@qq.com"


然后键入以下指令：

> ssh-agent -s

继续输入指令：

> ssh-add ~/.ssh/id_rsa

如果出现Could not open a connection to your authentication agent.
先输入

> $ eval `ssh-agent`
Agent pid 11368

然后再输入：

> $ ssh-add
Identity added: /c/Users/DLM/.ssh/id_rsa (/c/Users/DLM/.ssh/id_rsa)

复制ssh公钥

> $ clip< ~/.ssh/id_rsa.pub

然后到github上面，点击Settings
![](http://7xjw47.com1.z0.glb.clouddn.com/201506211.png)

点击SSH keys
![](http://7xjw47.com1.z0.glb.clouddn.com/201506212.jpg)

点击Add SSH key
![](http://7xjw47.com1.z0.glb.clouddn.com/201506213.png)

输入Title，作为这个key的描述，然后这个Key就是刚刚拷贝的，你直接粘贴就好
![](http://7xjw47.com1.z0.glb.clouddn.com/201506214.png)

最后测试一下是否成功了

> $ ssh -T git@github.com
The authenticity of host 'github.com (192.30.252.129)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? yew
Please type 'yes' or 'no': yes
Warning: Permanently added 'github.com,192.30.252.129' (RSA) to the list of know
n hosts.
Hi dengliming! You've successfully authenticated, but GitHub does not provide sh
ell access.

这个说明已经成功了。


最后进入blog目录，生成静态页面

> hexo clean

最后，通过命令进行部署。

> hexo deploy


OK，我们的博客就已经完全搭建起来了，在浏览器输入（当然，是你的用户名）：
http://yourname.github.io/


以后发布博客的部署步骤
hexo clean
hexo generate
hexo deploy