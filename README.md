# 介绍
目标通过运行go.sh脚本，即可部署整个百百后台系统

环境要求centos7+docker+docker-compose

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

# 阿里云docker源使用方式
```
{
  "registry-mirrors": ["https://iw3lcsa3.mirror.aliyuncs.com"]
}
```
`以上内容写入到/etc/docker/daemon.json,如果没有daemon.json文件自己创建一个`

# 修改IP地址

 修改docker-compose.yml
 ```
 搜索[frp.server.address] 改成服务器外网ip
 
 搜索[frptunnel.server.address] 改成服务器外网ip

  搜索[guac.guac.hostname] 改成服务器外网ip
 ```

 修改/data/frpscontrol和/data/frpstunnel下的frps.ini文件
 ```
 [token_auth_url]和[port_check_url]的IP、端口改成平台IP和端口
 
 ```


# 开始一键部署
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

# 访问百百系统
[http://127.0.0.1:8080/login](http://127.0.0.1:8080/login)

![登录.png](https://img-blog.csdnimg.cn/2020072523185896.png)

# 客户端安装
[客户端](https://github.com/baibaicloud/prober)