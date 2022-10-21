# 防火墙配置


### **服务配置**

- **启动**： `systemctl start firewalld.service`
- **查看状态**： `systemctl status firewalld.service`
- **停止**： `systemctl disable firewalld.service`
- **禁用**： `systemctl stop firewalld.service`

### **三种策略**

- **ACCEPT** 允许
- **REJECT** 拒绝
- **DROP** 丢弃

### **1. 查看**

```bash
# 查看激活的域
firewall-cmd --get-active-zones
# 查看开放的端口
firewall-cmd --zone=public --list-ports
# 查看开放的服务
firewall-cmd --zone=public --list-services
# 查看添加的规则
firewall-cmd --zone=public --list-rich-rules
```

### **2. 添加端口**

### **常用命令**

```bash
#开放单个端口
firewall-cmd --zone=public --add-port=80/tcp --permanent

# 开放端口范围
firewall-cmd --zone=public --add-port=8388-8389/tcp --permanent

# 对 147.152.139.197 开放10000端口
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="147.152.139.197/32" port protocol="tcp" port="10000" accept"

# 拒绝端口：
firewall-cmd --permanent --zone=public --add-rich-rule=' rule family="ipv4" source address="47.52.39.197/32" port protocol="tcp" port="10000" reject'

# 开放全部端口给IP
firewall-cmd --permanent --zone=public --add-rich-rule=' rule family="ipv4" source address="192.168.0.1/32" accept'

# 开放全部端口给网段
firewall-cmd --permanent --zone=public --add-rich-rule=' rule family="ipv4" source address="192.168.0.0/16" accept'
```

### **3. 添加服务**

```bash
# 查看全部支持的服务
firewall-cmd --get-service

# 查看开放的服务
firewall-cmd --list-service

# 添加服务,添加https
firewall-cmd --add-service=https --permanent

# 修改对应的配置文件是/etc/firewalld/zones/public.xml
```

### **4. 移除端口**

```bash
# 移除添加的端口(其它的增加的策略也是将add改为remove即可)
firewall-cmd --zone=public --remove-port=80/tcp --permanent
```

### **5. 重载配置**

```bash
# 对路由规则进行修改后，需要重新加载规则才能使规则生效
firewall-cmd --reload
```

### **实战应用**

### **(1)限制ssh访问，仅允许白名单IP登录**

也可以通过在/etc/host.deny里面配置来实现

```bash
# 取消默认开启的没有访问限制的ssh服务，让ssh服务默认情况下拒绝连接
firewall-cmd --permanent --remove-service=ssh

# 允许特定ip或ip段访问22端口的ssh服务
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="22" accept'

# 重载firewall配置，使其生效
firewall-cmd --reload
```

![image](test.jpeg)


