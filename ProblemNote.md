# ProblemNote
用以记录一些开发过程中遇到的问题


# 一、Java后端相关

## 1、对接第三方API

### 1.《医疗器械唯一标识管理信息系统数据共享API标准文档v3》

>[医疗器械唯一标识管理信息系统数据共享API标准文档]: https://udi.nmpa.gov.cn/showListInterr.html
>
>吐槽一下这个文档写的是真的垃圾，上面给出的返回字段类型都不对。本来根据这个破文档手动建实体类就够窝火了，返回时挨个挑错真的是想掀桌。

对于我们公司来说，常用的API就三个：`P001`、`P002`、`D003`

其中P接口用于测试连通和获取token，D003接口用以获取公司所需要的代码。 

### 2.支付宝相关API

### 3.微信相关API

## 2、考虑使用Redis作为消息队列（Redis Streams）

> 需要注意的是，如果没有创建消费者组，则不能使用`XACK`命令来清除已经使用`XADD`命令添加的stream键值对。此时需要`DEL`命令

MQ的维护成本太高，而且作为一个后端开发，虽然对Linux很感兴趣，但是完成不了开发任务也不是那么个事儿。

什么？你问运维呢？不好意思，从前端到后端，从开发到运维，从技术选型到实际落地，都是我自己。

```java
/**
  *代码还没写好呢。
  */
```



# 二、服务器相关（RockyLinux）

## 1、DockerHub停止服务问题

> 之前本地（Windows 11）跑Win版的DockerDesktop就发现这个问题了，不过我开魔法之后也能用。可是最近公司搞了台新服务器，用Docker的话就绕不开这个问题了，所以思前想后我决定把应用直接装在服务器上（bushi
>
> 查了一些资料说从Aliyun搞一个镜像，过几天我试试，试出来了会把流程记录下来以备日后再用。

假装有图.jpg

## 2、本地安装MySQL的相关配置



## 3、本地安装Redis的相关配置



## 4、本地安装配置Nginx

### 1. 更新系统

首先，确保你的系统是最新的。打开终端并运行以下命令：

```bash
sudo dnf update
```

### 2. 安装 EPEL 仓库

Nginx 在 EPEL（Extra Packages for Enterprise Linux）仓库中可用，因此需要先安装 EPEL：

```bash
sudo dnf install epel-release
```

### 3. 安装 Nginx

使用以下命令安装 Nginx：

```bash
sudo dnf install nginx
```

### 4. 启动 Nginx

安装完成后，可以启动 Nginx 服务：

```bash
sudo systemctl start nginx
```

### 5. 设置 Nginx 开机自启

如果希望 Nginx 在系统启动时自动启动，可以使用以下命令：

```bash
sudo systemctl enable nginx
```

### 6. 检查 Nginx 状态

确保 Nginx 正在运行：

```bash
sudo systemctl status nginx
```

### 7. 配置防火墙

如果你启用了防火墙，需要允许 HTTP 和 HTTPS 流量。运行以下命令：

```bash
sudo firewall-cmd --permanent --add-port=80/tcp --add-port=443/tcp	#开放多个端口命令
sudo firewall-cmd --permanent --add-service=http	#开放某个服务端口命令
#关闭某个端口或者服务
sudo firewall-cmd --permanent --remove-port=80/tcp
sudo firewall-cmd --permanent --remove-service=http

sudo firewall-cmd --reload	#重载防火墙
sudo firewall-cmd --list-all	#查看所有已开放端口
```

### 8. 测试 Nginx

打开浏览器，输入你的服务器 IP 地址（例如 `http://your_server_ip`）。如果安装成功，你应该能看到 Nginx 的欢迎页面。

### 9. 配置 Nginx

Nginx 的配置文件通常位于 `/etc/nginx/nginx.conf`，你可以根据需要进行自定义配置。

> CentOS好像是在 `/usr/local/nginx/conf/nginx.conf` 这个路径下。（不知道，不清楚）
> （可能是我在CentOS上用Docker的原因吧）

```bash
# user  nobody;		#指定运行 Nginx 的用户。通常设置为没有特权的用户（如 nobody）。
worker_processes  4; 	#设置工作进程的数量。一般建议设置为 CPU 核心数。
pid   /run/nginx.pid; 	#指定存储 Nginx 进程 ID 的文件路径。

events {
    worker_connections  2048; 	# 每个工作进程允许的最大连接数。
}
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;		# 开启高效文件传输。
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;		 # 设置 HTTP Keep-Alive 超时时间，单位为秒。

    #gzip  on;		#这个好像是开启压缩的一个开关
    
    #80端口配置文件
    server {
        listen       80;		 # 监听 HTTP 请求的 80 端口。
        server_name  localhost;		# 设置服务器名称，通常为域名或 IP 地址。

        #charset koi8-r;
		charset utf-8;
		
        #access_log  logs/host.access.log  main;
		
        #定义请求 URI 的处理方式。
        location / {
            root   /usr/share/nginx/html/dist;		 # 设定文档根目录。
            try_files $uri $uri/ /index.html;		 # 如果请求的文件不存在，尝试访问 index.html。
            index  index.html index.htm;		# 默认主页文件。
        }
		
		#代理设置
        location /prod-api/ {
          proxy_set_header Host $http_host;  		# 设置请求头中的 Host 字段。
          proxy_set_header X-Real-IP $remote_addr;	 # 设置真实 IP。
          proxy_set_header REMOTE-HOST $remote_addr; 	# 设置远程主机字段。
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_pass http://localhost:8080/;		# 指定请求转发到的后端服务地址，将请求代理到后端服务（如应用服务器）。
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        
        #错误页面处理
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #防止访问隐藏文件
        location ~ /\. {
            deny  all;
        }
    }
}
```

### 10.SSL证书相关问题




# 三、Web前端相关

## 1、electron国内下载缓慢问题

1. 更换国内镜像（淘宝镜像）

   在CMD中输入`npm config set registry https://registry.npmmirror.com`

2. 在项目根目录下新建名为`.npmrc`的文件

3. 打开文件，写入`ELECTRON_MIRROR="https://npmmirror.com/mirrors/electron/"`

4. 重新执行`npm i`或者`npm install`命令

## 2、下载electron-quick-start项目后没有打包输出的问题

1. 打开PowerShell，切换到对应的文件夹，执行命令`npm install electron-packager --save-dev`

2. 在`package.json`文件的`scripts`中加入`"packager": "electron-packager ./ XXX --out=./out --app-version=0.0.1 --platform=win32  --arch=x64 --ignore=node_modules --overwrite --icon=./dist/favicon.ico"`，（XXX可替换为你对应的项目名称）即：

   ```json
   {
     "name": "electron-quick-start",
     "version": "1.0.0",
     "description": "A minimal Electron application",
     "main": "main.js",
     "scripts": {
       "start": "electron .",
       "packager": "electron-packager ./ XXX --out=./out --app-version=0.0.1 --platform=win32  --arch=x64 --ignore=node_modules --overwrite --icon=./dist/favicon.ico" #加入这行脚本
     },
     "repository": "https://github.com/electron/electron-quick-start",
     "keywords": [
       "Electron",
       "quick",
       "start",
       "tutorial",
       "demo"
     ],
     "author": "GitHub",
     "license": "CC0-1.0",
     "devDependencies": {
       "electron": "^31.3.1",
       "electron-packager": "^17.1.2"
     }
   }
   ```


## 3、


# 四、开发杂项相关

## 1、常用Git命令

> 初始化、克隆等等这种最基础的命令就不写了，就这两板斧，估计够用，不够用再去问ChatGPT（doge）

### 1.远程仓库相关

```bash
#查看远程仓库
git remote -v

#添加远程仓库
git remote add <remote-name> <remote-url>
##在这一步之前，Git会提示你把当前文件夹添加到信任列表里，也会给出对应的命令，CV执行一下就好了。

#git修改远程仓库命令
git remote set-url <remote-name> <new remote-url>
#重命名 origin 为 upstream
git remote rename origin upstream

然后是常用的 `git add .` `git commit -m 'your commit message'` push的时候会提醒你设置上传的分支（我是这么理解的）
git push --set-upstream origin master
```

### 2.分支相关

```bash
#创建并切换到分支
git checkout -b <新分支名>
#重命名分支
git branch -m <旧分支名> <新分支名>
#拉取指定远程分支命令
git pull origin <分支名>
#删除本地分支
git branch -d <分支名>
#强制删除分支
git branch -D <分支名>
#删除远程分支
git push origin --delete <分支名>
#创建本地分支并跟踪远程分支
git checkout -b <新分支名> origin/<远程分支名>

##合并分支
#先切换到目标分支
git checkout <目标分支>
#然后merge你想合并的分支
git merge <待合并的分支>
#查看合并状态
git status
#处理合并冲突后
git add <解决后的文件>
git commit
git push

```

## 2、Typora常用快捷键及markdown语法
