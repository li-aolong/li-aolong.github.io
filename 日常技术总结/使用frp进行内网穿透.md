## 需求

实验室有一台服务器，没有公网IP，想要从外网访问需要内网穿透。花生壳等第三方软件也可以解决此问题，但是有流量限制，并且安装时候需要root权限，对于没有root的账户并不友好。

经过在网上搜索，找到了frp这个开源工具，不需要root权限且没有流量限制，唯一有要求的是需要一个具有公网IP的服务器，我购买了腾讯云来进行搭建。

## 步骤

先在https://github.com/fatedier/frp/releases下载frp所需文件，linux系统下载[linux_amd64](https://github.com/fatedier/frp/releases/download/v0.33.0/frp_0.33.0_linux_amd64.tar.gz)版本。

#### 1 服务端

使用腾讯云服务器linux系统，先解压：

```
tar -xzvf frp_0.33.0_linux_amd64.tar.gz
```

可以`cat frps.ini`查看一下默认配置文件，简单配置不需要修改，配置如下：

```
[common]
bind_port = 7000
```

启动服务端：

```
./frps -c ./frps.ini
```

要想让程序一直在后台运行，使用nohup命令：

```
nohup ./frps -c ./frps.ini
```

#### 2 客户端

客户端同样解压，修改的是`frpc.ini`文件，配置如下：

```
[common]
server_addr = xx.xx.xx.xx
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

只需将server_addr修改成服务端的ip就可以了，同样的方式启动：

```
./frpc -c ./frpc.ini
```

需要后台运行的话使用nohup：

```
nohup ./frpc -c ./frpc.ini
```

使用nohup命令会有nohup.out的相关提示，不用理会，运行完命令后不会直接退出到终端命令行，直接关闭终端界面。

#### 3 连接

在服务端使用命令：

```
ssh -oPort=6000 test@xx.xx.xx.xx
```

test是客户端的用户名，后边的地址是服务端地址，前提是开启ssh服务。

## xshell一键登录

在xshell上配置完服务端的ssh后，在登录脚本中添加两个命令：

```
等待：login	发送：ssh -oPort=6000 test@xx.xx.xx.xx
等待：password	发送：******（客户端密码）
```

然后就可以一键登录了。

