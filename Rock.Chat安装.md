### 参考文档
> [Rocket.Chat docs](https://rocket.chat/docs/)    



这次使用这个Rocket.Chat 是为了填补上次的Caddy的坑,上次的Caddy并没有展现Caddy的魅力,这次我用Caddy的反向代理来部署Rocket.Chat这个服务.    


Rocket.Chat 是一种类似 Slack 的开源聊天软件.

### **Rocket.Chat 的特性**

+ 群组聊天
+ 直接通信
+ 私聊群
+ 桌面通知
+ 媒体嵌入
+ 链接预览
+ 文件上传
+ 语音/视频聊天
+ 截图
+ 支持你目前使用的任何平台

我将用Caddy的反向代理和自动签发https证书来搭建Rocket.Chatl, 我将在Ununtu18.04(LTS)上来搭建

<br>


### 部署环境
+ 一台 Ubuntu18.04 (LTS)的服务器
+ 域名 (非必须,我将演示非域名和域名的不同环境)

<br>

### 不使用用Caddy反向代理

连接vps, 然后更新操作系统
```shell
sudo apt update && sudo apt upgrade -y
```

用snap安装Rocket.Chat
```shell
sudo snap install rocketchat-server
```

然后你就安装完毕了

然后在浏览器里输入你的`ip:3000`就可以访问你的Rocket.Chat


<br>


### 使用Caddy反向代理

安装Caddy可以参考之前的文档[CaddyServer搭建web服务器](https://www.redwolf233.top/archives/caddyserver%E6%90%AD%E5%BB%BAweb%E6%9C%8D%E5%8A%A1%E5%99%A8)

编辑Caddyfile文件
```shell
sudo vim /etc/caddy/Caddyfile
```

输入以下内容
```shell
http://:80 {
	reverse_proxy * 127.0.0.1:3000
	encode zstd gzip
}
```

然后就可以通过域名访问了

<br>

### 用域名访问

**首先你的去域名服务商,添加一个A记录指向你的服务器ip**

编辑Caddyfile文件
```shell
sudo vim /etc/caddy/Caddyfile
```

输入以下内容
```shell
yourdomain {
	tls youremail
    reverse_proxy * 127.0.0.1:3000
	encode zstd gzip
}
```
**只需要将yourdomain 改成你的解析的域名, youremail 改成你的邮箱**

For Example
```shell
chat.redwolf233.top {
	tls 1374034750@qq.com
    reverse_proxy * 127.0.0.1:3000
	encode zstd gzip
}
```

然后你就可以访问你的域名来进行操作了,下面给几张Photos
![](https://www.redwolf233.top/upload/2020/3/%E6%B7%B1%E5%BA%A6%E6%88%AA%E5%9B%BE_%E9%80%89%E6%8B%A9%E5%8C%BA%E5%9F%9F_20200302155847-8d82a00449d94da388a928014c8f917e.png)


![](https://www.redwolf233.top/upload/2020/3/%E6%B7%B1%E5%BA%A6%E6%88%AA%E5%9B%BE_%E9%80%89%E6%8B%A9%E5%8C%BA%E5%9F%9F_20200302155035-d900f458e2c14d3ea2ecb187f7c8e5c4.png)
