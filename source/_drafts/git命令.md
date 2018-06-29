 配置ssh
 生成密钥
 ssh-keygen -t rsa -b 4096 -C "vncnliu@gmail.com"
 启动ssh-agent
 eva $(ssh-agent -s)
 添加密钥
  ssh-add mjoys
  测试连接
  ssh -T git@github.com

  Warning: Permanently added the RSA host key for IP address '13.250.177.223' to the list of known hosts.
  Hi vncnliu! You've successfully authenticated, but GitHub does not provide shell access.
修改远程地址
 git remote set-url origin url

在命令行输入命令:

git config --global credential.helper store
这一步会在用户目录下的.gitconfig文件最后添加：

[credential]
    helper = store
push 代码
push你的代码 (git push), 这时会让你输入用户名和密码, 这一步输入的用户名密码会被记住, 下次再push代码时就不用输入用户名密码!这一步会在用户目录下生成文件.git-credential记录用户名密码的信息。

在项目目录下下设置本地用户名，邮箱
git config --local user.email "vncnliu@gmail.com"
git config --local user.name "vncnliu"