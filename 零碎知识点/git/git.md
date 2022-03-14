# git指令commit至github

记录commit至github的步骤，防止遗忘

第一次连接GitHub时需创建SSH key并填写至github

```shell
$ ssh-keygen -t rsa -C "youremail@example.com"
```

1. 创建本地仓库

```shell
git init
```

当文件夹中出现`.git`时，证明改文件夹已经作为本地仓库交至git管理

2. 添加文件

```
git add
```



2. 将文件提交至本地git仓库

```shell
git commit -m"添加描述"
```

3. github上创建仓库
4. 将本地仓库与GitHub相关联

```shell
$ git remote add origin git@github.com:Fischer0522/learngit.git
//origin为远程库的名字，learngit.git为仓库的名称
```

5. 推送本地的文件至远程库

```shell
$ git push -u origin master
```

