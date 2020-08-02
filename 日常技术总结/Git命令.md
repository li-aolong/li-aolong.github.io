**创建本地仓库**

```
git init	#会生成.git的隐藏文件夹
```

**添加远程仓库**

```
git remote add origin 仓库地址
```

**向云端更新本地数据**：

  ```
  git add -A
  git commit -m '...'
  git push origin master
  ```

 **从云端拉取全部数据**：

  ```
  git pull
  ```

**通过socks5代理**

- 查看自己代理软件本地监听socks5的端口，比如1080
- 然后使用命令，仅对github.com网站使用代理

```
git config --global http.https://github.com.proxy socks://127.0.0.1:1080
git config --global https.https://github.com.proxy socks5://127.0.0.1:1080
```

- 或打开`.gitconfig`文件直接修改
- 这种代理只能使用https协议，即：

```
git clone https://www.github.com/xxxx/xxxx.git
```

