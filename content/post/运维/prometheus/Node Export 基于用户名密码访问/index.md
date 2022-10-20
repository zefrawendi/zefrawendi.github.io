---
title: "为NodeExport配置访问密码"
date: 2022-10-20T09:52:48+08:00
lastmod: 2022-10-20T09:52:48+08:00
draft: false
description: "Node Export 基于用户名密码访问"
categories: ["运维"]
tags: ["监控","prometheus","exporter"]
slug: "node_exporter_password"
---

Export采集指标的地址谁都可以访问，这里可以使用基础认证使用用户名密码方式去采集被监控端，也就是访问接口使用用户名密码提高安全性

## 配置被监控端

这个配置文件里面是要访问我暴露的指标需要的用户名密码

用户名和密码，密码是采用了一定的加密方式的，而不是写明文

生成密码

```bash
[root@localhost ~]# yum install httpd-tools –y
```

下面就是输入123456之后加密的密码，将这个密码保存在配置文件当中

```bash
[root@localhost ~]# htpasswd -nBC 12 '' | tr -d ':\\\\n'
New password:
Re-type new password:
$2y$12$y4PaNc0UM0Jzi07jJf6zcuRFyp2GlH6F5rUKcE.xk3Aug2khcqa7m
```

修改其配置文件

```bash
[root@localhost node_exporter]# vim config.yml
[root@localhost node_exporter]# cat config.yml
basic_auth_users:
  prometheus: $2y$12$y4PaNc0UM0Jzi07jJf6zcuRFyp2GlH6F5rUKcE.xk3Aug2khcqa7m
```

现在让export引用这个配置文件

```bash
[root@localhost node_exporter]# vim /usr/lib/systemd/system/node_exporter.service
ExecStart=/usr/local/node_exporter/node_exporter  --web.config=/usr/local/node_exporter/config.yml

[root@localhost node_exporter]# systemctl daemon-reload
[root@localhost node_exporter]# systemctl restart node_exporter
```

输入prometheus加上密码123456

![https://img-blog.csdnimg.cn/20210125112335914.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NTU2NDE0,size_16,color_FFFFFF,t_70](https://img-blog.csdnimg.cn/20210125112335914.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NTU2NDE0,size_16,color_FFFFFF,t_70)

同时现在普罗米修斯没有配置，可以看到采集不了数据了

![https://img-blog.csdnimg.cn/20210125112449192.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NTU2NDE0,size_16,color_FFFFFF,t_70](https://img-blog.csdnimg.cn/20210125112449192.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NTU2NDE0,size_16,color_FFFFFF,t_70)

## 配置普罗米修斯

启用用户名密码访问

```bash
[root@localhost prometheus]# vim /usr/local/prometheus/prometheus.yml
  - job_name: 'webserver'
    basic_auth:
      username: prometheus
      password: 123456
    static_configs:
    - targets: ['192.168.179.99:9100','192.168.179.102:9100']

[root@localhost prometheus]# ./promtool check config prometheus.yml
Checking prometheus.yml
  SUCCESS: 0 rule files found
```

可以看到数据可以正常采集了

![https://img-blog.csdnimg.cn/20210125112829583.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NTU2NDE0,size_16,color_FFFFFF,t_70](https://img-blog.csdnimg.cn/20210125112829583.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NTU2NDE0,size_16,color_FFFFFF,t_70)