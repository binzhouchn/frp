# frp内网穿透

https://github.com/fatedier/frp/blob/dev/conf/frps_full_example.toml<br>

 - frp_0.61.1_linux_amd64.tar.gz

https://github.com/luckjiawei/frpc-desktop 这个frp客户端还是比较好用的<br>
 
 - Frpc-Desktop-1.1.5-arm64.dmg


## 1.配置frps（frp服务端）


### 1.1 下载frp_0.61.1_linux_amd64.tar.gz

```shell
cd /root
tar -xvf frp_0.61.1_linux_amd64.tar.gz
mv frp_0.61.1_linux_amd64 frp
```

### 1.2 修改frps.toml文件

```shell
# frps的通信端口
bindPort = 7100
# 与frpc通信的认证方式和token值
auth.method = "token"
auth.token = "xx12345"
# 面板的端口、账号和密码
webServer.addr = "0.0.0.0"
webServer.port = 7600
webServer.user = "admin"
webServer.password = "admin"

log.to = "./frps.log"
log.level = "info"
log.maxDays = 3

# Note: http port and https port can be same with bindPort
vhostHTTPPort = 80
vhostHTTPSPort = 443

subDomainHost = "domain.com"
```

### 1.3 启动

```shell
nohup ./frps -c frps.toml &
```

### 1.4 建议开启防火墙然后单独添加端口

```shell
firewall-cmd --state #查看防火墙状态，是否是running
systemctl start firewalld #启动防火墙
systemctl enable firewalld #开机启动
firewall-cmd --add-port=7100/tcp --permanent #永久添加7100端口
firewall-cmd --add-port=7600/tcp --permanent #永久添加7600端口
firewall-cmd --reload #重新载入配置，添加规则之后，需要执行此命令
firewall-cmd --zone=public --list-ports #查看已开放的端口
```

## 2.配置frpc（frp客户端）--以mac为例

### 2.1 下载并安装Frpc-Desktop-1.1.5-arm64.dmg

### 2.2 打开app完成配置

系统配置

 - Frp版本：v0.61.1
 - 服务器地址：154.xx.xxx.xx
 - 服务器端口：7100，验证方式token 令牌：xx12345

穿透列表

 - 代理类型：http
 - 代理名称随便取
 - 内网地址：127.0.0.1，内网端口：7777
 - 子域名：frp

### 2.3 域名管理平台添加frp

https://console-domain.oray.com/domain-list/domain-resolution?rootname=domain.com

在以上网站添加一条主机记录frp，记录类型A记录，记录值为公网服务器地址154.xx.xxx.xx


### 2.4 保存后点击启动frpc

启动后通过，访问http://frp.domain.com（154.xx.xxx.xx:80）即可以访问macbook本地127.0.0.1:7777端口服务了！

【注windows和mac类似】

## 3.配置frpc（frp客户端）--以linux为例

### 3.1 下载对应frpc客户端

比如https://github.com/fatedier/frp/releases/download/v0.61.1/frp_0.61.1_linux_amd64.tar.gz

### 3.2 配置frpc.toml文件

```shell
serverAddr = "154.xx.xxx.xx"
serverPort = 7100
auth.method = "token"
auth.token = "xx12345"
transport.heartbeatInterval = 30
transport.heartbeatTimeout = 90

log.to = "./frpc.log"
log.level = "info"
log.maxDays = 3
webServer.addr = "127.0.0.1"
webServer.port = 7600
transport.tls.enable = false

[[proxies]]
name = "jp"
type = "http"
localIP = "127.0.0.1"
localPort = 7777
customDomains=["domain.com"]
subdomain="frp"
```

### 3.3 启动frpc

```shell
nohup ./frpc -c frpc.toml &
```

访问http://frp.domain.com（154.xx.xxx.xx:80）即可以访问linux本地127.0.0.1:7777端口服务了（注意7777端口对应服务要启动哟不然也访问不了）

## 4 TODO

