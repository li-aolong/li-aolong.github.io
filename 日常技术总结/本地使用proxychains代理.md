## 服务端

首先在服务端配置好socks5，配置信息如下：

```
 主机 (Hostname) = xx.xx.xx.xx
 端口 (Port) = xxxx
 用户名 (Username) = socks（随意）
 密码 (Password) = ******
```

## 客户端

若有root权限，使用命令安装proxychains：

```
rpm -ivh http://download-ib01.fedoraproject.org/pub/fedora/linux/releases/30/Everything/x86_64/os/Packages/p/proxychains-ng-4.13-3.fc30.x86_64.rpm
```

若没有权限，首先下载文件并解压：

```
wget https://sourceforge.net/projects/proxychains-ng/files/proxychains-4.5.tar.bz2
tar -jxvf proxychains-4.5.tar.bz2
```

进入解压后的目录进行安装：

```
cd proxychains-4.5
./configure --prefix=/home/user/proxychains/    #user为具体用户名
make && make install-config
```

至此安装完毕，进行客户端配置。有root权限安装的配置文件在`/etc/proxychains.conf`，用源码安装的配置文件在刚刚定义的位置`/home/user/proxychains/etc/proxychains.conf`，在配置文件最后一行去掉原始参数改成：

```
socks5 服务器ip地址 socks5的端口号 socks5的用户名 对应的密码
```

配置环境变量：

```
vim ~/.bashrc
```

将刚刚解压的文件夹添加进环境变量，最后加入一行：

```
export PATH=$PATH:~/具体位置/proxychains/proxychains-4.5/
```

更新环境变量：

```
source ~/.bashrc
```

#### 使用

有root权限的使用`proxychains`命令，源码安装的使用`proxychains4`命令，在其它命令前加上该命令，则通过代理进行转发，如：

```
proxychains curl myip.ipip.net
```

若返回服务端的地址，则代理成功。