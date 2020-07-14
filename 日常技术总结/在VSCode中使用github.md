## 在本地生成rsa密钥对

```
ssh-keygen -t rsa -C "github账号的email"
```

## 配置github的公钥

在github的账户设置中复制公钥

## 若user/.ssh文件夹下已有多个rsa

须配置一个config文件，格式如下：

```
Host server
    HostName xx.xx.xx.xx
    User root
    Port xx
    IdentityFile ~/.ssh/id_rsa_server
Host github
	HostName github.com
	User git
	IdentiteFile ~/.ssh/id_rsa_github
```

