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

## 3、若依的一点东西

> 今天用若依框架的时候，生成出来的代码并不是很贴合我现在的开发，所以就查了查资料（问了问ChatGPT），发现还可以这么玩？所以就姑且记录一下。

### 1.若依的代码生成器

其实若依生成代码是有一套模板在的，就在`vm`包下。

![image-20241106192745122](./assets/image-20241106192745122.png)

`vm`包下包含了所有的模板类文件，若依已经帮我们分别整理好了。

![image-20241106193148613](./assets/image-20241106193148613.png)

```xml
<sql id="select${ClassName}Vo">
        select#foreach($column in $columns) ${tableAlias}.$column.columnName#if($foreach.count != $columns.size()),#end#end from ${tableName} ${tableAlias} 
    <!-- 使用表名的简写 -->
            left join sys_user u on ${tableAlias}.user_id = u.user_id
            left join sys_dept d on ${tableAlias}.dept_id = d.dept_id
</sql>
```

`mapper.xml.vm`文件中我主要是改了一下表连接，因为我的业务要求对所有数据权限进行统一管理，所以我给每条数据都绑定了`user_id`和`dept_id`（不得不说若依的RBAC是真的NICE，对我这种接近于独立开发的人来说省了大功夫了）

上面的代码中，我还添加了一个`tableAlias`也就是表别名和数据范围过滤的条件。需要在`VelocityUtils`类中修改`prepareContext`方法。获取表别名的方法也放出来（就随便一写，有改进方法可以交流一下）

```java
// 获取表名的简写
String tableAlias = extractAbbreviation(genTable.getTableName());


VelocityContext velocityContext = new VelocityContext();
// 将表名的简写添加到 VelocityContext
velocityContext.put("tableAlias", tableAlias);
velocityContext.put("dataScopePlaceholder", "<!-- 数据范围过滤 --> ${params.dataScope}");
/**
 * 将带有下划线的字符串提取简称
 * @param input 带有下划线的字符串
 * @return
 */
private static String extractAbbreviation(String input) {
    // 先将输入的字符串转为小写
    StringBuilder abbreviation = new StringBuilder();

    // 遍历输入的字符串
    boolean afterUnderscore = false;  // 标志位，表示是否遇到下划线之后
    for (int i = 0; i < input.length(); i++) {
        char currentChar = input.charAt(i);

        if (i == 0 || afterUnderscore) {  // 如果是第一个字符，或者遇到下划线后的第一个字符
            abbreviation.append(currentChar);  // 将字符添加到简称中
            afterUnderscore = false;  // 重置标志
        }

        // 判断是否遇到下划线
        if (currentChar == '_') {
            afterUnderscore = true;  // 设置标志位
        }
    }

    return abbreviation.toString();
}
```

另外在`controller.java.vm`中我在`list`方法上加了`@DataScope(deptAlias = "d",userAlias = "u")`用作数据范围过滤。这样基本上就可以达到我要的效果了。

### 2.用不到的ID Generator

写完才发现若依有UUID生成器，想想自己这个小体量也用不上这么复杂的ID，就写了一个业务ID生成器。

```java
import com.ruoyi.common.utils.DateUtils;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Random;

@Component
@ConfigurationProperties(prefix = "ruoyi.customIdPrefix")
public class BusinessIDGenerator {

    private List<String> prefix;

    public List<String> getPrefix() {
        return prefix;
    }

    public void setPrefix(List<String> prefix) {
        this.prefix = prefix;
    }

    public String generatorId(Integer prefixIndex) {
        Random random = new Random();
        //取得application.yml文件中配置好的前缀数组
        List<String> prefix = this.getPrefix();
        //当前时间格式化为 yyyyMMddHHmmssSSS  TIMENOWADDMS是我自己规定的 格式到毫秒
        String dateTimeNow = DateUtils.dateTimeNow(DateUtils.TIMENOWADDMS);
        //为了防止撞车整了个三位随机数
        int randomNumber = random.nextInt(999);
        String suffixes =  String.format("%03d",randomNumber);
        //拼起来就是我自己造的ID
        return prefix.get(prefixIndex) + dateTimeNow + suffixes;
    }
}
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

## 3、ruoyi-vue3利用electron-quick-start项目打包成exe

> 建议使用`electron-build`，不要用这个！
>
> （但是如果已经把前端项目布署到服务器上，倒是可用这个`electron-packager`
>
> 关键代码就一行（嗯，你懂的）

### 1) 糊弄版

下载`electron-packager`并在`package.json`中的`script`模块中添加`"packager": "electron-packager . 你的应用名称 --platform=win32 --out=./out/ --arch=x64 --app-version=1.0.0 --icon=你的图标位置 --overwrite"`

在`main.js`中添加`mainWindow.loadURL("http://你的应用IP:你的应用端口");`

`npm run packager`回车

秒了（DDDD）

### 2) 正经版

（^_^）


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
git merge master --allow-unrelated-histories
#查看合并状态
git status
#处理合并冲突后
git add <解决后的文件>
git commit
git push

```

## 2、Typora常用快捷键及markdown语法
