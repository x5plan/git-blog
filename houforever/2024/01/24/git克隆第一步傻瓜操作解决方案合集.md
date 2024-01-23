# git clone第一步傻瓜操作解决方案合集

## ssh克隆报错

```
git clone git@github.com:x5plan/git-blog.git x5plan-git-blog 

Cloning into 'x5plan-git-blog'... 

ssh: connect to host github.com port 22: Connection timed out fatal: Could not read from remote repository. Please make sure you have the correct access rights and the repository exists.
```

1. **网络问题**：你的网络可能无法访问GitHub。尝试访问[https://github.com/x5plan/git-blog看看是否可以正常访问。](https://github.com/x5plan/git-blog看着是否可以将名称可以正常访问。)
2. **SSH密钥问题**：确保你已经添加了正确的SSH密钥到你的GitHub账户。你可以在GitHub的"Settings" -> "SSH and GPG keys"中查看或添加SSH密钥。
3. **SSH配置问题**：你的SSH配置可能有问题。你可以尝试运行`ssh -T git@github.com`来测试SSH连接。如果这个命令返回"Hi [your username]! You've successfully authenticated, but GitHub does not provide shell access."，那么你的SSH配置是正确的。
4. **代理服务器问题**：如果你在使用代理服务器，你可能需要配置SSH以使用代理。



### 逐步排查：

#### 第一类：访问网址

​	裸连不上需要挂梯子。

#### 第二类：（无密钥）

​	终端中执行命令

```
ssh-keygen -t rsa -C "备注"
```

找到.ssh文件夹，复制.id_rsa.pub文件中所有内容，打开github主页【settings】【SSH and GPG keys】【new ssh key】,titile随便写，内容粘贴并保存。

​	再次打开终端执行

```
ssh -T git@github.com
//运行结果出现类似如下
Hi yourname! You've successfully authenticated, but GitHub does not provide shell access.
```

​	如上则成功。

#### 第三类：（有密钥，配置不正确）

```
ssh -T git@github.com

ssh: connect to host github.com port 22: Connection timed out
```

​		出错内容大致意思是端口22的问题，错误提示的是连接github.com的22端口被拒绝了。解决方案是换个端口443端口，具体需要在.ssh文件夹中添加config文件，内容如下：

```
Host github.com
User userna
me  #username可改
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
```

​		注：网上大多是vim命令创建config文件的教程，window11下我采用新建txt后直接删除拓展名的形式（需在文件资源管理器中打开查看中的文件拓展名设置）



再次测试：

```
ssh -T git@github.com
```



## http克隆报错

建议转ssh克隆



#### 尝试打开github页面

#### ping测试

​	终端输入命令：

```
ping github.com
```

​	ping通则可排除git不正常的情况。

```
//取消http代理
git config --global --unset http.proxy
//取消https代理 
git config --global --unset https.proxy
```

​	仍旧不行则可在gitee中导入github仓库然后下载其源码（未测试，建议ssh）

