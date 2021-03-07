# 文档目标
通过运行`go.sh`脚本，即可部署整个百百后台系统

# 环境要求

### 服务器系统centos7,至少7

### 服务器系统需要安装docker+docker-compose
[点击查看docker+docker-compose安装方式](https://blog.csdn.net/weixin_38989540/article/details/107436628)

`注意：要开启防火墙`

```
systemctl enable firewalld.service
systemctl start firewalld.service
firewall-cmd --get-active-zones
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --zone=public --add-port=7000/tcp --permanent
firewall-cmd --zone=public --add-port=8100/tcp --permanent
firewall-cmd --reload
```

`需要范围开放端口,每个远控控制需要消耗一个端口，端口会进行重复利用回收`

端口从7001到7050

```
firewall-cmd --zone=public --add-port=7001/tcp --permanent
firewall-cmd --zone=public --add-port=7002/tcp --permanent
firewall-cmd --zone=public --add-port=7003/tcp --permanent
...
...
firewall-cmd --zone=public --add-port=7048/tcp --permanent
firewall-cmd --zone=public --add-port=7049/tcp --permanent
firewall-cmd --zone=public --add-port=7050/tcp --permanent
```

### 阿里云docker源使用方式，能够加快下载docker镜像速度
```
{
  "registry-mirrors": ["https://iw3lcsa3.mirror.aliyuncs.com"]
}
```
`以上内容写入到/etc/docker/daemon.json,如果没有daemon.json文件自己创建一个`

# 文件准备
### 下载部署文件

```
git clone https://github.com/baibaicloud/deployment-platform.git
cd deployment-platform
```

### 修改IP地址

 修改docker-compose.yml
 ```
 搜索[frp.server.address] 改成服务器外网ip
 
 搜索[frptunnel.server.address] 改成服务器外网ip

 搜索[guac.guac.hostname] 改成服务器外网ip
 ```

 修改`./data/frpscontrol`和`./data/frpstunnel`下的frps.ini文件
 ```
 [token_auth_url]和[port_check_url]的IP、端口改成平台IP和端口
 
 ```


# 开始部署
```
cd 到deployment目录下
添加go.sh执行权限
chmod +x go.sh
./go.sh
```
```
`打印以下内容就部署成功`
Creating baibai-frpscontrol ... done
Creating baibai-guacd       ... done
Creating baibai-mysql       ... done
Creating baibai-frpstunnel  ... done
Creating baibai-platform    ... done
success
[root@localhost deployment]# 
```

`要重新部署系统，删除/data目录再执行go.sh即可`
`如果没部署成功，请确保docker环境正常、docker-compose环境正常`

# 访问百百系统
[http://127.0.0.1:8080/login](http://127.0.0.1:8080/login)

![登录.png](https://img-blog.csdnimg.cn/2020072523185896.png)

# 客户端安装
[客户端开源地址](https://github.com/baibaicloud/prober)
### 客户端文件下载
[客户端下载地址](https://github.com/baibaicloud/prober/releases)，下载最新版本，分别有windown和linux版本。

客户端下载完毕之后把两个客户端文件放到服务器的`/data/platform/apps/`目录下，文件名称不用修改。

然后在百百系统的WEB登录界面右上角【客户端下载】菜单点击进行下载。

# 远程控制测试

360可能会报病毒，请大家放心使用。

远程控制支持两种协议，vnc和rdp。

要测试vnc很简单，直接发起vnc就可以，无需安装vnc客户端。

要测试rdp，需要确保被控制的PC 3389端口是可用的，这个一定要确认。如何确认3389端口可用，很简单，找另一个电脑直接使用微软自带的远程发起控制，如果能够成功就可用，失败的自己找原因。


# HTTPS部署

百百客户端支持配置https的URL，百百平台系统不提供https功能，需要借助第三方工具提供，比如：nginx。

### nginx 配置https代理

```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       4431 ssl;
        server_name  localhost;

        ssl_certificate      /etc/nginx/ssl/test.pem;
        ssl_certificate_key  /etc/nginx/ssl/test.key;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        # 这里指向百百平台系统和端口
        location / {
            proxy_pass http://192.168.0.100:8080/;
        }
    }

}

docker rm -f nginx && docker run -dt --name=nginx --net=host -v /data/nginx/ssl:/etc/nginx/ssl -v /data/nginx/nginx.conf:/etc/nginx/nginx.conf nginx

```

`注意：部署https时，http端口可用不用暴露到公网，但要暴露给frp服务`

# 插个私人广告 BB-API HTTP请求工具
致力于打造简洁、免费、好用的HTTP模拟请求工具，自动生成接口文档。

帮助您公司、团队、个人提高开发效率。

官网地址：http://api.app-yun.com/bbapi/index.html
