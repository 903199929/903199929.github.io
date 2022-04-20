# Nginx

环境: Apple M1 ARM64，Docker

打开命令行：`docker run --name nginx -p 8080:80 -d nginx`  
就可以运行Nginx，镜像没有下载也没有关系，会自动拉取最新版本的Nginx，所以说，还有什么环境会比M1芯片的Mac更糟糕呢。

## Nginx挂载

---

Nginx的配置文件修改还是挺频繁的，不能总是进到容器里边再修改吧。有一说一vi修改文件真是极其麻烦，所以将配置文件挂载到本地可以说是迫在眉睫。  
1. 在本地新建三个文件夹，名字可以任取，这里演示用的是这三个`config`,`data`,`logs`。（记住文件夹的路径，待会有大用）
2. 将docker中的文件复制到本地，`docker cp nginx:/etc/nginx /Users/xx/nginx/config/   ### nginx配置文件`,`docker cp nginx:/usr/share/nginx/html /Users/xx/nginx/data/html/   ### 资源内容文件`(以空格分隔，前边的路径是容器里边的路径，后边的是本地路径也就是刚刚新建文件夹的路径)
3. 挂载到本地（刚刚创建的Nginx已经没用了，可以删掉），`docker run --name nginx -p 8080:80 -v /Users/xx/nginx/config/nginx:/etc/nginx -v /Users/xx/nginx/data/html:/usr/share/nginx/html -v /Users/xx/nginx/logs:/var/log/nginx -d nginx`(注意路径，以`:`为分隔，前边的是本地路径，后边的是容器路径)

不出意外的话，应该是挂载成功了。

## Nginx反向代理

---

打开挂载后的`config`文件夹，配置文件被分成了`nginx.conf`和`conf.d`下的`default.conf`其中`http`和`events`在`nginx.conf`中配置，`server`在`default.conf`中配置。

``` conf
server {
    listen       80; # 容器内部的端口
    listen  [::]:80;
    server_name  localhost; #容器内部的ip

    location / {
        proxy_pass http://本地Ip:端口号; # 宿主机的ip+服务端口号
    }
}
```

配置好以后不要忘记重启Nginx，接着再通过`localhost:8080`访问即可。

**Tips:** 如果配置有问题的话看看下方的日志`error.log`

## 反向代理缓存

---

Nginx缓存可以对用户已经访问过的内容在Nginx服务器本地建立副本，这样在一段时间内再次访问该数据，就不需要通过Nginx服务器再次向后端服务器发出请求，所以能够减少Nginx服务器与后端服务器之间的网络流量，减轻网络阻塞，同时还能减小数据传输延迟，提高用户访问速度。

配置也很简单
1. 在`http`模块的第一行加上`proxy_cache_path /存放路径 levels=1:2 keys_zone=缓存名:50m max_size=1g inactive=10m; `（存放路径需要自行建立）其中`levels`表示缓存存放的层级，`keys_zone`为缓存名字和占用内存大小。`inactive`为缓存生效时间。
2. 在`server`里的`location`模块中  
   ```conf
    location / {
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for; #设置请求头
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   Host             $http_host;
        
        proxy_cache 缓存名; #与上一步的缓存名字对应
        proxy_pass http://localhost/; #代理路径
        proxy_cache_valid 200 304 302 10m; # 响应为200，304，302的请求缓存十分钟
        proxy_cache_key $host$uri$is_args$args; #url做key，用于更新缓存
    }
   ```
如此即可
