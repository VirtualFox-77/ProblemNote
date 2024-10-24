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

## 2、

# 二、服务器相关（RockyLinux）

## 1、DockerHub停止服务问题

## 2、本地安装MySQL的相关配置

## 3、本地安装Redis的相关配置

## 4、本地安装Nginx的相关配置


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

## 2、Typora常用快捷键及markdown语法
