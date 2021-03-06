### 参考文档

> [CaddyServer官方文档](https://caddyserver.com/)

CaddyServer是基于go语言写的一款搭建web前端的软件,之前我搭建网站都是用的宝塔面板,用起来超级好用但是就是搭建时间比较长,而这款软件Caddy搭建网站基本上就是2-3分钟就可以搭建一个网站.今天我就演示一下如何用这个CaddyServer来搭建一个Ariang的面板


**PS:我用的是CaddyServer的v2版本.v2版本和v1的版本语法不兼容**


### Caddy安装

首先我们得下载Caddy的执行文件,然后放入到/usr/bin/目录下

```shell
sudo wget -O /usr/bin/caddy https://github.com/caddyserver/caddy/releases/download/v2.0.0-beta.15/caddy2_beta15_linux_amd64 && chmod +x /usr/bin/caddy
```

然后可以输入一下`caddy version` 来检测caddy命令是否在你的$PATH下

然后创建一个caddy的组
```shell
groupadd --system caddy
```

然后创建一个用户caddy,和具有可写的目录
```shell
useradd --system \
	--gid caddy \
	--create-home \
	--home-dir /var/lib/caddy \
	--shell /usr/sbin/nologin \
	--comment "Caddy web server" \
	caddy
```

然后我们可以下载一个官方给的[caddy.service](https://raw.githubusercontent.com/caddyserver/dist/master/init/caddy.service)的文件放到`/etc/systemd/system/caddy.service`
```shell
sudo wget -O /etc/systemd/system/caddy.service https://raw.githubusercontent.com/caddyserver/dist/master/init/caddy.service
```

之后我们重启systemctl,和启动caddy
```shell
sudo systemctl daemon-reload
sudo systemctl enable caddy
sudo systemctl start caddy
```

然后我们可以验证一下是否启动,如果出现`(active) running`就说明好了
```shell
sudo systemctl status caddy
```

### Caddyfile编写

Caddy可以让我们用Caddyfile来启动caddy接下来我将用Caddyfile来写一个AriaNg的网站.

首先我们的去下载一个AriaNg的代码放入到`/var/www/ariang`目录下并且解压缩
```shell
sudo wget -O /var/www/ariang/ariang.zip https://github.com/mayswind/AriaNg/releases/download/1.1.4/AriaNg-1.1.4.zip && sudo unzip -o /var/www/ariang/ariang.zip -d /var/www/ariang/ && sudo rm /var/www/ariang/ariang.zip
```

然后我们去`/etc/caddy/`目录新建一个文件`Caddyfile`然后开始写配置文件
```shell
cd /etc/caddy/ && sudo touch Caddyfile
```

然后用vim来写一段配置文件到Caddyfile
```shell
vim Caddyfile
```

然后把这一段代码粘贴进去
```shell
http://:6088 {
    file_server {
        root /var/www/ariang
    }
    encode zstd gzip
}
```

最后输入以下命令,如果没报错就可以了
```shell
caddy reload
```

*ps 6088是端口,可以自行修改*

之后我们上浏览器输入`http://127.0.0.1:6088` or `http://localhost:6088`就可以看到网站了

当然CaddyServer还有很多功能
+ 虚拟主机
+ 反向代理
+ 静态文件分发
+ 负载均衡
+ FastCGI支持
+ MarkDown渲染
+ Gzip压缩
+ URL重写
+ 重定向
+ 文件浏览服务

本次就用了CaddyServer的虚拟主机搭建了一个小的网站,CaddyServer还有很多好玩的等着大家去探索.


此处贴一张网站的Photo
![Ariang](https://file.redwolf233.top/blogpic/ece6ff007aebcd10564143755298ca37.png)

