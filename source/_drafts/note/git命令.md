#### 初始化
git init
git add .
git commit -m "init"

#### 添加远程仓库
git remote add origin #{git remote url}

git push -u origin master

把本地库的内容推送到远程，用git push命令，实际上是把当前分支master推送到远程。

由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

#### 在项目目录下下设置本地用户名，邮箱
git config --local user.email "vncnliu@gmail.com"
git config --local user.name "vncnliu"


