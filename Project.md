# 1. 前端切换主题颜色

### @mixin 和 @include

1. 普通用法

- 定义混合指令@mixin

```style
@mixin large-text {
  font: {
    family: Arial;
    size: 20px;
    weight: bold;
  }
  
  color: #ff0000;
  &:after {
      content: ".";
      display: block;
      height: 0;
      clear: both;
      visibility: hidden;
  }
}
```

- 引用混合样式 @include

```style
.page-title {
  @include large-text;
  padding: 4px;
  margin-top: 10px;
}
```

2. 传参

```js
@mixin sexy-border($color, $width) {
  border: {
    color: $color;
    width: $width;
    style: dashed;
  }
}
p { @include sexy-border(blue, 1in); } 
或者关键字传参
p { @include sexy-border($color:blue); }
```

3. 为便于书写，`@mixin` 可以用 `=` 表示，而 `@include` 可以用 `+` 表示，所以上面的例子可以写成：

```style
@mixin apply-to-ie6-only {
  * html {
    @content;
  }
}
@include apply-to-ie6-only {
  #logo {
    background-image: url(/logo.gif);
  }
}

=apply-to-ie6-only
  * html
    @content

+apply-to-ie6-only
  #logo
    background-image: url(/logo.gif)
```

### 插值语句#{}

```scss
$name: foo;
$attr: border;
p.#{$name} {
  #{$attr}-color: blue;
}
```

编译为

```css
p.foo {
  border-color: blue; }
```

### & 父级元素

```js
.a {
    &:hiover   即a:hover
}

```

## 1.1 切换主题

- theme.scss

```js
@mixin font_color(
  $font_color_black, //黑色主题时字体颜色
  $font_color_white, //白色主题时字体颜色
  $font_weight_black: normal, //是否需要加粗
  $font_weight_white: normal //是否需要加粗
) {

  // 主题为黑色时的字体颜色
  [data-theme="#{$theme-black}"] & { //记住这里一定要有 & 符号，否则样式不生效
    color: $font_color_black;
    font-weight: $font_weight_black;
  }

  // 主题为白色时的字体颜色
  [data-theme="#{$theme-white}"] & { //类似于 [data-theme="xxx"] a {}
    color: $font_color_white;
    font-weight: $font_weight_white;
  }
}
//第一个为属性值，第二个为黑色主题时的颜色，第三个参数为白色主题时的参数，最后一个参数为接受不确定的参数个数，类似于reset参数
@mixin setAttribute($attribute, $bg_color_black, $bg_color_white, $value...) {

  // 主题为黑色时的背景颜色
  [data-theme="#{$theme-black}"] & {
    #{$attribute}: $bg_color_black $value; //#{}为插值语法解析成字符串 多用在属性值以及选择器上比如#{border}-right: 1px solid #fff;
  }

  // 主题为白色时的背景颜色
  [data-theme="#{$theme-white}"] & {
    #{$attribute}: $bg_color_white $value;
  }
}
```

-  vue.config.js 添加全局scss文件

```js

  chainWebpack: config => {
    const oneOfsMap = config.module.rule('scss').oneOfs.store;
    oneOfsMap.forEach(item => {
      item
        .use('sass-resources-loader') // 实现全局变量/方法 注入
        .loader('sass-resources-loader')
        .options({
          // Or array of paths
          // 添加全局scss文件
          resources: [
            'src/assets/scss/theme-mixin.scss',
            'src/assets/scss/theme-var.scss',
            'src/assets/scss/common.scss'
          ]
        })
        .end();
    });
  }
```

- App.vue 切换主题时 设置该属性

```html
  window.document.documentElement.setAttribute('data-theme', newVal);
```

# 2. 数据库更新前端刷新

## 1.websocket

```js

var websocket = null;
 
//判断当前浏览器是否支持WebSocket
if ('WebSocket' in window) {
  websocket = new WebSocket("ws://localhost:8086/websocket/1");
} else {
  alert('当前浏览器 Not support websocket')
}
 
//连接发生错误的回调方法
websocket.onerror = function() {
  console.log("WebSocket连接发生错误");
};
 
//连接成功建立的回调方法
websocket.onopen = function() {
  console.log("WebSocket连接成功");
}
 
//接收到消息的回调方法
websocket.onmessage = function(event) {
  //返回数据转JSON
  var json=JSON.parse(event.data);
  //result为bootstrap table 返回数据
  var rows=result.rows;
  for(var i=0;i<rows.length;i++){
    var row=rows[i];
    if(row.id==json.id){
      //判断列Id相同时刷新表格
      //$('#dataGrid').bootstrapTable('updateByUniqueId', {index: i, row: row});'refresh'
      $('#dataGrid').bootstrapTable('refresh');
    }
  }
  console.log(event.data);
}
 
//连接关闭的回调方法
websocket.onclose = function() {
  console.log("WebSocket连接关闭");
}
 
//监听窗口关闭事件，当窗口关闭时，主动去关闭websocket连接，防止连接还没断开就关闭窗口，server端会抛异常。
window.onbeforeunload = function() {
  closeWebSocket();
}
 
//关闭WebSocket连接
function closeWebSocket() {
  websocket.close();
}
```

## 2.sockjs

> Sock.JS的一大好处在于提供了浏览器兼容性。优先使用原生WebSocket，如果在不支持websocket的浏览器中，会自动降为轮询的方式。 
>
> Stomp.js（处理传输协议）

```js
import SockJS from 'sockjs-client'
import Stomp from 'stompjs'
export default {  
  data() {
    return {
      socketUrl: 'http://127.0.0.1:8080/ws/web-socket'
    }
  },
  mounted() {    
    this.initWebSocket()
  },
  methods: {
    // 接收到消息并对消息做处理
    onMessageReceived(payload) {
      var message = JSON.parse(payload.body)
      console.info('Message', message)
    },
    // 连接成功
    successCallback() {
      console.info('onConnected')
      this.stompClient.subscribe('/topic/topicid', this.onMessageReceived)
      this.stompClient.send('/app/msg',
        {},
        JSON.stringify({ sender: 'sender', type: 'JOIN' })
      )
    },
    initWebSocket() {
      this.socket = new SockJS(this.socketUrl)// 连接服务端
      this.stompClient = Stomp.over(this.socket)
      this.stompClient.connect({}, (frame) => {
        this.successCallback()
      }, () => {
        this.reconnect(this.socketUrl, this.successCallback)
      })
    },
    // 断开重连使用定时器定时连接服务器
    reconnect(socketUrl, successCallback) {
      console.info('in reconnect function')
      let connected = false
      const reconInv = setInterval(() => {
        console.info('in interval' + Math.random())
        this.socket = new SockJS(socketUrl)
        this.stompClient = Stomp.over(this.socket)
        this.stompClient.connect({}, (frame) => {
          console.info('reconnected success')
          // 连接成功，清除定时器
          clearInterval(reconInv)
          connected = true
          successCallback()
        }, () => {
          console.info('reconnect failed')
          console.info('connected:' + connected)
          if (connected) {
            console.info('connect .... what?')
          }
        })
      }, 2000)
    },
    // 前端websocket发送消息
    sendMessage() {
      var chatMessage = {
        sender: 'sender',
        content: 'Hello Server!',
        type: 'MSG'
      }
      this.stompClient.send('/app/msg', {}, JSON.stringify(chatMessage))
    }
  }
}


//简写
  created() {
      if (this.stompClient) {
        this.stompClient.disconnect();
      }
      const sockjs = new Sockjs();
      this.stompClient = Stompjs.over(sockjs);
      this.stompClient.debug = false;
      this.stompClient.connect(
        {},
        success => {
          console.log(success);
        },
        err => {
          console.log(err);
        }
      );
    },
```

## 3.webworker

http://www.ruanyifeng.com/blog/2018/07/web-worker.html

## 4.postmessage

```js
1.postMessage

window.postMessage() 是HTML5的一个接口，专注实现不同窗口不同页面的跨域通讯。

为了演示方便，我们将hosts改一下：127.0.0.1 crossDomain.com，现在访问域名crossDomain.com就等于访问127.0.0.1。

这里是http://localhost:9099/#/crossDomain，发消息方：

<template>

  <div>

    <button @click="postMessage">给http://crossDomain.com:9099发消息</button>

    <iframe name="crossDomainIframe" src="http://crossdomain.com:9099"></iframe>

  </div>

</template>

<script>

export default {

  mounted () {

    window.addEventListener('message', (e) => {

      // 这里一定要对来源做校验

      if (e.origin === 'http://crossdomain.com:9099') {

        // 来自http://crossdomain.com:9099的结果回复

        console.log(e.data)

      }

    })

  },

  methods: {

    // 向http://crossdomain.com:9099发消息

    postMessage () {

      const iframe = window.frames['crossDomainIframe']

      iframe.postMessage('我是[http://localhost:9099], 麻烦你查一下你那边有没有id为app的Dom', 'http://crossdomain.com:9099')

    }

  }

}

</script>

这里是http://crossdomain.com:9099，接收消息方：

<template>

  <div>

    我是http://crossdomain.com:9099

  </div>

</template>

<script>

export default {

  mounted () {

    window.addEventListener('message', (e) => {

      // 这里一定要对来源做校验

      if (e.origin === 'http://localhost:9099') {

        // http://localhost:9099发来的信息

        console.log(e.data)

        // e.source可以是回信的对象，其实就是http://localhost:9099窗口对象(window)的引用

        // e.origin可以作为targetOrigin

        e.source.postMessage(`我是[http://crossdomain.com:9099]，我知道了兄弟，这就是你想知道的结果：${document.getElementById('app') ? '有id为app的Dom' : '没有id为app的Dom'}`, e.origin);

      }

    })

  }

}

</script>
```

- 结果可以看到：

![](E:\study\note\pic\nginx_6.png)



# 3.gsap

## 1.捕捉数字变动

```js
data:{
    return {
        bell: {
            totolAppCountTweendNum:0
        }
    }
}
// 方法内
gsap.to(this.$data.bell, 1, { totolAppCountTweendNum: newVal });
```

# 4. worker和sharedworker

https://www.cnblogs.com/baimulan/p/11562010.html

# 5.WebView

## 本节引言

> 本节给大家带来的是Android中的一个用于显示网页的控件：**WebView**(网页视图)。
>
> 现在Android应用 层开发的方向有两种：客户端开发和HTML5移动端开发！
>
> 所谓的HTML5端就是：HTML5 + CSS + JS来构建 一个网页版的应用,而这中间的媒介就是这个WebView,而Web和网页端可以通过JS来进行交互,比如, 网页读取手机联系人,调用手机相关的API等！
>
> 而且相比起普通的客户端开发,HTML5移动端有个优势： 可以用百分比来布局,而且如果HTML5端有什么大改,我们不用像客户端那样去重新下一个APP,然后 覆盖安装,我们只需修改下网页即可！而客户端...惨不忍睹,当然HTML5也有个缺点,就是性能的问题, 数据积累,耗电问题,还有闪屏等等...
>
> 另外,针对这种跨平台我们可以使用其他的第三方快速开发 框架,比如PhoneGap,对了,还有现在网络上很多一键生成APP类的网站,用户通过拖拉,设置图片 之类的简单操作就可以生成一个应用,大部分都是用的HTML5来完成的！有模板,直接套,你懂的~ 好的,话不多说,开始本节内容！

## 1.什么是WebView？

> 答：Android内置webkit内核的高性能浏览器,而WebView则是在这个基础上进行封装后的一个 控件,WebView直译网页视图,我们可以简单的看作一个可以嵌套到界面上的一个浏览器控件！

# 6. nginx上部署项目

## 1.服务器上部署

1. 下载Xshell

![](pic\nginx_xshell.png)

云服务器 => 控制台 => 实例 =>当前区域 => 查看服务器ip和端口号 => Xshell中进行连接

![](pic\nginx_1.png)

2. 下载niginx

   ```text
   yum install -y nginx
   ```

3. 修改nginx配置

   - 点击打开文件夹

     ![](pic\nginx_2.png)

   - 将前端dist置入

     ![](pic\nginx_3.png)

   - 配置nginx

   ![](pic\nginx_4.png)

   - 进行编辑

     ```js
     user nginx;
     worker_processes auto;
     error_log /var/log/nginx/error.log;
     pid /run/nginx.pid;
     
     # Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
     include /usr/share/nginx/modules/*.conf;
     
     events {
         worker_connections 1024;
     }
     
     http {
         log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                           '$status $body_bytes_sent "$http_referer" '
                           '"$http_user_agent" "$http_x_forwarded_for"';
     
         access_log  /var/log/nginx/access.log  main;
     
         sendfile            on;
         tcp_nopush          on;
         tcp_nodelay         on;
         keepalive_timeout   65;
         types_hash_max_size 2048;
     
         include             /etc/nginx/mime.types;
         default_type        application/octet-stream;
         
         include /etc/nginx/conf.d/*.conf;
     
         server {
             listen       8080;    //设置端口号
             ssl_session_timeout 20m; //配置超时
             server_name  localhost;
             # Load configuration files for the default server block.
             include /etc/nginx/default.d/*.conf;
     
             location / {
     		root         		/usr/share/nginx/html/dist;  //包地址
     		index  index.html 	index.htm;
                 		try_files $uri $uri/ 	/index.html;  //history模式
             }
     
             error_page 404 /404.html;
             location = /404.html {
             }
     
             error_page 500 502 503 504 /50x.html;
             location = /50x.html {
             }
         }
     ```
  ```
   
4. 重启nginx    在etc中执行以下命令(etc/nginx)
   
      ![](pic\nginx_5.png)

## 2.nginx相关配置

- Nginx 常用的几个命令：

> ```
> etc/nginx -t                   # 查看nginx状态
> etc/nginx -s reload            # 重新载入配置文件
> etc/nginx -s reopen            # 重启 Nginx
> etc/nginx -s stop              # 停止 Nginx
> ```

- nginx 文件结构

  - 1、**全局块**：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
  - 2、**events块**：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
  - 3、**http块**：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
  - 4、**server块**：配置虚拟主机的相关参数，一个http中可以有多个server。
  - 5、**location块**：配置请求的路由，以及各种页面的处理情况。

  ```js
  ...              #全局块
  
  events {         #events块
     ...
  }
  
  http      #http块
  {
      ...   #http全局块
      server        #server块
      { 
          ...       #server全局块
          location [PATTERN]   #location块
          {
              ...
          }
          location [PATTERN] 
          {
              ...
          }
      }
      server
      {
        ...
      }
      ...     #http全局块
  }
              
              
  //示例
  
  user nginx;
  worker_processes auto;
  error_log /var/log/nginx/error.log;
  pid /run/nginx.pid;
  
  # Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
  include /usr/share/nginx/modules/*.conf;
  
  events {
      worker_connections 1024;
  }
  
  http {
      log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';
  
      access_log  /var/log/nginx/access.log  main;
  
      sendfile            on;
      tcp_nopush          on;
      tcp_nodelay         on;
      keepalive_timeout   65;
      types_hash_max_size 2048;
  
      include             /etc/nginx/mime.types;
      default_type        application/octet-stream;
  
      # Load modular configuration files from the /etc/nginx/conf.d directory.
      # See http://nginx.org/en/docs/ngx_core_module.html#include
      # for more information.
      include /etc/nginx/conf.d/*.conf;
  	#代理配置参数
       # proxy_connect_timeout 180;
       # proxy_send_timeout 180;
       # proxy_read_timeout 180;
       # proxy_set_header Host $host;
       # proxy_set_header X-Forwarder-For $remote_addr;
      server {
          listen       8080;#监听端口
          server_name  localhost; #监听地址
          # Load configuration files for the default server block.
          include /etc/nginx/default.d/*.conf;
  
          location / {
  		root         		/usr/share/nginx/html/dist;
  		index  index.html 	index.htm;
  		# proxy_pass  http://mysvr; #配置代理 请求转向mysvr 定义的服务器列表
  	
          
  		# deny 127.0.0.1;  #拒绝的ip
          # allow 172.18.5.54; #允许的ip    
          try_files $uri $uri/ 	/index.html;#配置history模式
          }
  
          error_page 404 /404.html;
          location = /404.html {
          }
  
          error_page 500 502 503 504 /50x.html;
          location = /50x.html {
          }
      }
  }
  ```

- 负载均衡

  - 代理与负载均衡

  ```js
  upstream mysvr { 
      server 192.168.10.121:3333;
      server 192.168.10.122:3333;
  }
  server {
      ....
      location  ~*^.+$ {         
          proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表         
      }
  }
      
  ```
```
  
1、热备：如果你有2台服务器，当一台服务器发生事故时，才启用第二台服务器给提供服务。服务器处理请求的顺序：AAAAAA突然A挂啦，BBBBBBBBBBBBBB.....
  
```
  upstream mysvr { 
      server 127.0.0.1:7878; 
      server 192.168.10.121:3333 backup;  #热备     
  }
```
  
2、轮询：nginx默认就是轮询其权重都默认为1，服务器处理请求的顺序：ABABABABAB....
  
```
  upstream mysvr { 
      server 127.0.0.1:7878;
      server 192.168.10.121:3333;       
  }
```
  
3、加权轮询：跟据配置的权重的大小而分发给不同服务器不同数量的请求。如果不设置，则默认为1。下面服务器的请求顺序为：ABBABBABBABBABB....
  
```
  upstream mysvr { 
      server 127.0.0.1:7878 weight=1;
      server 192.168.10.121:3333 weight=2;
  }
```
  
4、ip_hash:nginx会让相同的客户端ip请求相同的服务器。
  
```
  upstream mysvr { 
      server 127.0.0.1:7878; 
      server 192.168.10.121:3333;
      ip_hash;
  }
```
  
5、如果你对上面4种均衡算法不是很理解，可以查看[Nginx 配置详解](https://www.runoob.com/w3cnote/nginx-setup-intro.html)，可能会更加容易理解点。
  
到这里你是不是感觉nginx的负载均衡配置特别简单与强大，那么还没完，咱们继续哈，这里扯下蛋。
  
关于nginx负载均衡配置的几个状态参数讲解。
  
  - down，表示当前的server暂时不参与负载均衡。
  - backup，预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的压力最轻。
  - max_fails，允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误。
- fail_timeout，在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。
  
```
  upstream mysvr { 
      server 127.0.0.1:7878 weight=2 max_fails=2 fail_timeout=2;
      server 192.168.10.121:3333 weight=1 max_fails=2 fail_timeout=1;    
  }

# 7. tooltip

```css
 <div class="box">
    <span class="tooltip tooltip-up" data-msg="上提示信息"> 提示在上</span>
    <span class="tooltip tooltip-down" data-msg="上提示信息"> 提示在上</span>
    <span class="tooltip tooltip-left" data-msg="上提示信息"> 提示在上</span>
    <span class="tooltip tooltip-right" data-msg="上提示信息"> 提示在上</span>
  </div>

.box{
  width:800px;
  text-align:center;
  margin:50px auto;
}
a{
  display:inline-block;
  margin:0 50px;
}


/* tooltip css */
.tooltip {
  position: relative;
  color: #333;
  text-align: center;
  font-size: 14px;
  cursor: pointer;
}

.tooltip:hover::before {
  word-break:keep-all;           /* 不换行 */
  white-space:nowrap; 
  content: attr(data-msg);
  position: absolute;
  padding: 2px 6px;
  display: block;
  color: #333;
  border: 1px solid #333;
  border-radius: 5px;
  font-size: 14px;
  line-height：20px;
}

.tooltip:hover::after {
  content: "";
  position: absolute;
}

.tooltip-up:hover::before {
  top: -30px;
  left:50%;
 -webkit-transform: translateX(-50%);
    -ms-transform: translateX(-50%);
    transform: translateX(-50%);
}

.tooltip-up:hover::after {
  border-top: 5px solid #333;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  top: -5px;
  left: 50%;
  -webkit-transform: translateX(-50%);
    -ms-transform: translateX(-50%);
    transform: translateX(-50%);
}
.tooltip-down:hover::before {
  bottom: -30px;
  left:50%;
 -webkit-transform: translateX(-50%);
    -ms-transform: translateX(-50%);
    transform: translateX(-50%);
}

.tooltip-down:hover::after {
  border-bottom: 5px solid #333;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  bottom: -5px;
  left: 50%;
  -webkit-transform: translateX(-50%);
    -ms-transform: translateX(-50%);
    transform: translateX(-50%);
}
.tooltip-left:hover::before {
  top: -2px;
  left: -8px;
 -webkit-transform: translateY(-50%);
    -ms-transform: translateY(-50%);
    transform: translateY(-50%);
  transform: translateX(-100%);
}

.tooltip-left:hover::after {
  border-left: 5px solid #333;
  border-top: 5px solid transparent;
  border-bottom: 5px solid transparent;
  top: 50%;
  left:-8px;
  -webkit-transform: translateY(-50%);
    -ms-transform: translateY(-50%);
    transform: translateY(-50%);
}
.tooltip-right:hover::before {
  top: -2px;
  right: -8px;
 -webkit-transform: translateY(-50%);
    -ms-transform: translateY(-50%);
    transform: translateY(-50%);
  transform: translateX(100%);
}

.tooltip-right:hover::after {
  border-right: 5px solid #333;
  border-top: 5px solid transparent;
  border-bottom: 5px solid transparent;
  top: 50%;
  right:-8px;
  -webkit-transform: translateY(-50%);
    -ms-transform: translateY(-50%);
    transform: translateY(-50%);
}
```

# 8. 获取数据类型

- typeof和instance和Object.prototype.toString

1. **typeof **返回值：number,boolean,string,function,object,undefined

   > null, object 和 array均识别为object
   >
   > 用法:  typeof a = "undefined"

2. **instanceof**:  用来检测 constructor.prototype 是否存在于参数 object 的原型链上。

   > object instanceof constructor
   >
   > 用法:
   >
   > function Car(make, model, year) {
   >   this.make = make;
   >   this.model = model;
   >   this.year = year;
   > }
   > const auto = new Car('Honda', 'Accord', 1998);
   >
   > console.log(auto instanceof Car);
   > // expected output: true

   - 可以用来判断对象类型

   > function a () {}            a instanceof Function   // true
   >
   > *const* b = {};                 b instanceof Object;    // true

3. **Object.prototype.toString**

   对于 `Object.prototype.toString()` 方法，会返回一个形如 `"[object XXX]"` 的字符串。

   如果对象的 `toString()` 方法未被重写，就会返回如上面形式的字符串。

   ```js
   ({}).toString();     // => "[object Object]"
   Math.toString();     // => "[object Math]"
   ```

   但是，大多数对象，`toString()` 方法都是重写了的，这时，需要用 `call()` 或 `Reflect.apply()` 等方法来调用。

   ```js
   var x = {
     toString() {
       return "X";
     },
   };
   x.toString();                                     // => "X"
   Object.prototype.toString.call(x);                // => "[object Object]"
   ```

- 获取数据类型的方法封装

  ```js
  function typeOf(target) {
    const toString = Object.prototype.toString;
    const map = {
      '[object Boolean]': 'boolean',
      '[object Number]': 'number',
      '[object String]': 'string',
      '[object Function]': 'function',
      '[object Array]': 'array',
      '[object Date]': 'date',
      '[object RegExp]': 'regExp',
      '[object Undefined]': 'undefined',
      '[object Null]': 'null',
      '[object Object]': 'object'
    };
    return map[toString.call(target)];
  }
  
  // 深拷贝
  function deepCopy(data) {
    const t = typeOf(data);
    let o;
  
    if (t === 'array') {
      o = [];
    } else if (t === 'object') {
      o = {};
    } else {
      return data;
    }
  
    if (t === 'array') {
      for (let i = 0; i < data.length; i++) {
        o.push(deepCopy(data[i]));
      }
    } else if (t === 'object') {
      for (const i in data) {
        o[i] = deepCopy(data[i]);
      }
    }
    return o;
  }
  
  ```


# 9.Promise 实现原理

```javascript
  // 判断变量否为function
  const isFunction = variable => typeof variable === 'function'
  // 定义Promise的三种状态常量
  const PENDING = 'PENDING'
  const FULFILLED = 'FULFILLED'
  const REJECTED = 'REJECTED'

  class MyPromise {
    constructor (handle) {
      if (!isFunction(handle)) {
        throw new Error('MyPromise must accept a function as a parameter')
      }
      // 添加状态
      this._status = PENDING
      // 添加状态
      this._value = undefined
      // 添加成功回调函数队列
      this._fulfilledQueues = []
      // 添加失败回调函数队列
      this._rejectedQueues = []
      // 执行handle
      try {
        handle(this._resolve.bind(this), this._reject.bind(this)) 
      } catch (err) {
        this._reject(err)
      }
    }
    // 添加resovle时执行的函数
    _resolve (val) {
      const run = () => {
        if (this._status !== PENDING) return
        this._status = FULFILLED
        // 依次执行成功队列中的函数，并清空队列
        const runFulfilled = (value) => {
          let cb;
          while (cb = this._fulfilledQueues.shift()) {
            cb(value)
          }
        }
        // 依次执行失败队列中的函数，并清空队列
        const runRejected = (error) => {
          let cb;
          while (cb = this._rejectedQueues.shift()) {
            cb(error)
          }
        }
        /* 如果resolve的参数为Promise对象，则必须等待该Promise对象状态改变后,
          当前Promsie的状态才会改变，且状态取决于参数Promsie对象的状态
        */
        if (val instanceof MyPromise) {
          val.then(value => {
            this._value = value
            runFulfilled(value)
          }, err => {
            this._value = err
            runRejected(err)
          })
        } else {
          this._value = val
          runFulfilled(val)
        }
      }
      // 为了支持同步的Promise，这里采用异步调用
      setTimeout(run, 0)
    }
    // 添加reject时执行的函数
    _reject (err) { 
      if (this._status !== PENDING) return
      // 依次执行失败队列中的函数，并清空队列
      const run = () => {
        this._status = REJECTED
        this._value = err
        let cb;
        while (cb = this._rejectedQueues.shift()) {
          cb(err)
        }
      }
      // 为了支持同步的Promise，这里采用异步调用
      setTimeout(run, 0)
    }
    // 添加then方法
    then (onFulfilled, onRejected) {
      const { _value, _status } = this
      // 返回一个新的Promise对象
      return new MyPromise((onFulfilledNext, onRejectedNext) => {
        // 封装一个成功时执行的函数
        let fulfilled = value => {
          try {
            if (!isFunction(onFulfilled)) {
              onFulfilledNext(value)
            } else {
              let res =  onFulfilled(value);
              if (res instanceof MyPromise) {
                // 如果当前回调函数返回MyPromise对象，必须等待其状态改变后在执行下一个回调
                res.then(onFulfilledNext, onRejectedNext)
              } else {
                //否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
                onFulfilledNext(res)
              }
            }
          } catch (err) {
            // 如果函数执行出错，新的Promise对象的状态为失败
            onRejectedNext(err)
          }
        }
        // 封装一个失败时执行的函数
        let rejected = error => {
          try {
            if (!isFunction(onRejected)) {
              onRejectedNext(error)
            } else {
                let res = onRejected(error);
                if (res instanceof MyPromise) {
                  // 如果当前回调函数返回MyPromise对象，必须等待其状态改变后在执行下一个回调
                  res.then(onFulfilledNext, onRejectedNext)
                } else {
                  //否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
                  onFulfilledNext(res)
                }
            }
          } catch (err) {
            // 如果函数执行出错，新的Promise对象的状态为失败
            onRejectedNext(err)
          }
        }
        switch (_status) {
          // 当状态为pending时，将then方法回调函数加入执行队列等待执行
          case PENDING:
            this._fulfilledQueues.push(fulfilled)
            this._rejectedQueues.push(rejected)
            break
          // 当状态已经改变时，立即执行对应的回调函数
          case FULFILLED:
            fulfilled(_value)
            break
          case REJECTED:
            rejected(_value)
            break
        }
      })
    }
    // 添加catch方法
    catch (onRejected) {
      return this.then(undefined, onRejected)
    }
    // 添加静态resolve方法
    static resolve (value) {
      // 如果参数是MyPromise实例，直接返回这个实例
      if (value instanceof MyPromise) return value
      return new MyPromise(resolve => resolve(value))
    }
    // 添加静态reject方法
    static reject (value) {
      return new MyPromise((resolve ,reject) => reject(value))
    }
    // 添加静态all方法
    static all (list) {
      return new MyPromise((resolve, reject) => {
        /**
         * 返回值的集合
         */
        let values = []
        let count = 0
        for (let [i, p] of list.entries()) {
          // 数组参数如果不是MyPromise实例，先调用MyPromise.resolve
          this.resolve(p).then(res => {
            values[i] = res
            count++
            // 所有状态都变成fulfilled时返回的MyPromise状态就变成fulfilled
            if (count === list.length) resolve(values)
          }, err => {
            // 有一个被rejected时返回的MyPromise状态就变成rejected
            reject(err)
          })
        }
      })
    }
    // 添加静态race方法
    static race (list) {
      return new MyPromise((resolve, reject) => {
        for (let p of list) {
          // 只要有一个实例率先改变状态，新的MyPromise的状态就跟着改变
          this.resolve(p).then(res => {
            resolve(res)
          }, err => {
            reject(err)
          })
        }
      })
    }
    finally (cb) {
      return this.then(
        value  => MyPromise.resolve(cb()).then(() => value),
        reason => MyPromise.resolve(cb()).then(() => { throw reason })
      );
    }
  }
```

