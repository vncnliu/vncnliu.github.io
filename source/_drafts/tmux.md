文本复制
PREFIX ： \[ : C Space : 选择文本: C w:复制并退出 ： PREFIX \]:粘贴

终端中使用命令复制的东西和linux系统粘贴板往往是分开的，使用xclip可以打通这个限制，非常方便。

需要安装 xclip工具。在 Ubuntu 系统上，使用如下命令：
sudo apt-get install xclip
2 . tmux粘贴板中的内容复制到系统粘贴板中,在 .tmux.conf文件里添加如下命令：

bind C-c run " tmux save-buffer - | xclip -i -sel clipboard"
这个配置定义了 PREFIX CTRL-c快捷键来捕获当前缓存区的内容然后通过管道输出到 xclip程序里。

3.系统粘贴板中的内容复制到tmux终端的粘贴板中，在 .tmux.conf文件里添加如下命令：

bind C-v run " tmux set-buffer \"$(xclip -o -sel clipboard)\"; tmux paste-buffer"
