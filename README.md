# ClashMetaData
一个用旧手机充当ClashMeta监控屏的项目
## 项目初衷
放假在家发现了一个多年前的旧手机（荣耀9i），想着让它发挥点余热，本意是做一个电脑小副屏，但尝试了多种方案，都不理想。遂想到当个监控屏，检测下软路由上的Clash，实际应用起来，发现已有的控制面板未能很好的适配手机端（当然并不影响那些项目是极为优秀，与之相比我写的更像是小孩子的涂鸦）。于是自学了一点vue，利用闲暇之余完成了这个项目，它并不是一个完美的项目，更多是提供一种选择。  
**本人技术很菜，还希望有大佬能带我深入学习vue。**
## 预览图
![image](https://github.com/aafang/ClashMetaData/assets/145802153/b9b487b0-eb3c-4820-a280-d58b31511982)  
## 如何部署
### 1、node.js 直接运行
拉取项目
```
git clone https://github.com/aafang/ClashMetaData.git
```
安装依赖
```
npm i
```
在```.env```修改为您本地ClashMeta接口和密钥  
```
VITE_CLASH_URL = "192.168.1.1:9090"
VITE_CLASH_TOKEN = "123456"
```
运行
```
npm run dev 
```
### 2、软路由 Nginx 部署
如果您的软路系统OpenWrt，而且Web服务是nginx处理的那么请继续。（UHttp的还请自行了解，我用的固件安装Nginx时，会自动把Uhttp服务切换到Nginx）
拉取项目
```
git clone https://github.com/aafang/ClashMetaData.git
```
安装依赖
```
npm i
```
在```.env```修改为您本地ClashMeta接口和密钥  
```
VITE_CLASH_URL = "192.168.1.1:9090"
VITE_CLASH_TOKEN = "123456"
```
编译
```
npm run build
```
如有以下错误，不要害怕，这是因为使用了eval函数解析websocket的数据，提示其不安全。
```
src/App.vue (189:20) Error when using sourcemap for reporting an error: Can't resolve original location of error.
src/App.vue (189:20) Use of eval in "src/App.vue" is strongly discouraged as it poses security risks and may cause issues with minification.
src/App.vue (211:17) Error when using sourcemap for reporting an error: Can't resolve original location of error.
src/App.vue (211:17) Use of eval in "src/App.vue" is strongly discouraged as it poses security risks and may cause issues with minification.
src/App.vue (220:19) Error when using sourcemap for reporting an error: Can't resolve original location of error.
src/App.vue (220:19) Use of eval in "src/App.vue" is strongly discouraged as it poses security risks and may cause issues with minification.
```
然后我们会得到 dlist 目录，将其上传到你的软路由上。链接软路由，将如下配置追加到你的Nginx配置文件中。  
Nginx配置文件的路径在```/etc/nginx/conf.d/shortcuts.conf```
```
server {
      # 监听的端口
      listen 30090;
      listen [::]:30090;
      location / {
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            root /root/dist;  # dist目录在服务器的完整路径
            index index.html index.htm;
      }
}
```
重启Nginx
```
/etc/init.d/nginx restart
```
## 手机端如何食用
浏览器访问你的项目地址即可，例如：
```
http://192.168.1.1:30090
```
理论上一个可横屏，可全屏的浏览浏览器就可完美显示，不排除屏幕太小导致显示异常，可以修改内核版本为固定短字符。  
至于浏览器，作者选用是 Dolphin（海豚浏览器），这个没有特殊要求。

