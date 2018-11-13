###  设置网络

找到网络设备，如enp0s1 
执行
 dhcpcd enp0s1
 给网卡自动分配ip 

systemctl enable dhcpcd@enp0s1.service 
自动联网 

设置静态ip 

/etc/dhcpcd.conf  
追加
```
# Generate SLAAC address using the Hardware Address of the interface
#slaac hwaddr
# OR generate Stable Private IPv6 Addresses based from the DUID
slaac private
noipv4ll
interface enp1s0
static ip_addrss=192.168.1.156/24
static router=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8
```

添加非root用户
useradd -m -G wheel -s /bin/bash xxx

安装xfce
pacman -S xorg-server  xfce4 
测试
进入桌面环境 

exec startxfce4 

退出桌面环境 

pkill X 

设置中文环境 

 

安装中文locale 

Linux中通过locale来设置程序运行的不同环境。常用的中文locale有（最直观的分别是可显示字的数量）： 

zh_CN.GB2312 
zh_CN.GBK 
zh_CN.GB18030 
zh_CN.UTF-8 
zh_TW.BIG-5 
zh_TW.UTF-8 
 

推荐使用UTF-8的locale。对于glibc（>=2.3.6），需要修改/etc/locale.gen文件来设定系统中可以使用的locale（取消对应项前的注释符号「#」即可）： 

en_US.UTF-8 UTF-8 
zh_CN.UTF-8 UTF-8 
 

然后执行locale-gen命令，便可以在系统中使用这些locale。可以通过locale命令来查看当前使用的locale：亦可通过locale -a命令来查看目前可以使用的locale； 

录时读取并运用里面的设置 

单独在图形界面启用中文locale 

如前面所说，可以在~/.xinitrc或~/.xprofile单独设置中文locale。添加如下内容到上述文件最前端注释之后（如果不确定使用哪个文件，可以都添加）： 

（应该放在 exec startxfce4 前面，否则不生效) 

export LANG=zh_CN.UTF-8 
export LANGUAGE=zh_CN:en_US 
export LC_CTYPE=en_US.UTF-8 

没问题编辑用户目录下.xinitrc文件
export LANG=en_US.UTF-8
export LANGUAGE=zh_CN:en_US 
export LC_CTYPE=zh_CN.UTF-8 
exec startxfce4


.bashrc: 每次终端登录时读取并运用里面的设置。 

.xinitrc: 每次startx启动X界面时读取并运用里面的设置 

.xprofile: 每次使用gdm等图形登录时读取并运用里面的设置 

安装字体 

sudo pacman -S  ttf-inconsolata noto-fonts noto-fonts-cjk noto-fonts-emoji 

刷新字体缓存 

fc-cache -vf 

安装登录管理器 

pacman -S slim 

安装slim-themes软件包： 

pacman -S slim-themes archlinux-themes-slim  

软件包 archlinux-themes-slim 包含了多个不同主题。可以在 /usr/share/slim/themes 中查看。在to see the themes available. Enter the theme name on the current_theme line in /etc/slim.conf: 

编辑/etc/slim.conf中的current_theme那行，将"default"改为你想要的主题名： 

/#current_theme       default 
current_theme       archlinux 

安装主题 

主题网站 https://www.xfce-look.org 

下载X-Arc-Collection 

解压到/ usr / share / themes或〜/ .themes（如果需要，创建它（在你的home文件夹中））; 
 
安装对应主题图标 

git clone https://github.com/horst3180/arc-icon-theme --depth 1 && cd arc-icon-theme 
./autogen.sh --prefix=/usr 
sudo make install 

安装输入法
pacman -S  fcitx-rime fcitx-configtool

修改/etc/environment 追加

export GTK_IM_MODULE=fcitx 
export QT_IM_MODULE=fcitx 
export XMODIFIERS="@im=fcitx"

同时保证.xinitrc文件中语言顺序
export LANG=en_US.UTF-8
export LANGUAGE=zh_CN:en_US 
export LC_CTYPE=zh_CN.UTF-8 

安装ss代理客户端
pacman -S shadowsocks-libev
在/etc/shadowsocks创建配置文件confing.json
```json
{
	"server":"xxx.xxx.xxx.xxx",
	"server_port":1080,
	"password":"xxxx",
	"local_adress":"127.0.0.1",
	"local_port":1080,
	"timeout":600,
	"method":"aes-256-cfb"
}
```
测试
 ss-local -c /etc/shadowsocks/config.json
 如没有正确展示端口绑定等信息，检查配置文件
 创建服务文件/usr/lib/systemd/system/ss.service 
 [Unit]
 Description=shadowsocks server
 After=network.target
 
 [Service]
 Type=forking
 ExecStart=/usr/bin/ss-local -c /etc/shadowsocks/config.json -f /tmp/ss
 PrivateTmp=true
 
 [Install]
 WantedBy=multi-user.target

systemctl enable ss.service

安装chromium
以代理方式启动chromium --proxy-server="socks5://127.0.0.1:1080"
启动后可登录google账户，同步信息后正常使用

QQ方案
wine经常崩溃，基本不可用
腾讯的封闭政策导致最靠谱的还是安装虚拟机
安装virtualBox
安装Windows Thin PC 
下载
Thin PC 于 2011 年推出。或许是由于担心它抢占自家桌面系统的市场，微软对该产品逐渐缄口不提。到现在，Thin PC 连一个正经的官网都不存在了。由于第三方渠道下载的镜像无法验证安全性，我们还是要想办法从微软那里下载 Thin PC 的安装文件。老谋深算的 M$ 到头来还是敌不过人民群众的智慧，有人从微软官方服务器把下载链接扒出来了：

http://download.microsoft.com/download/C/D/7/CD789C98-6C1A-43D6-87E9-F7FDE3806950/ThinPC_110415_EVAL_x86fre.iso
Thin PC 据称只有 32 位英文这一个版本。因为它以 Windows 7 作为基础，可以轻松地安装语言包，所以不用担心中文的显示和输入问题。



安装
从光盘引导系统后，你会发现 Thin PC 的安装界面和 Windows 7 是一模一样的！安装请参照任何一个 Windows 7 教程，本文不再叙述。



激活
初次安装后，Windows 会提醒你，请在三天之内激活系统。



Windows 激活的手段大致有两种，其一是破坏本地激活机制，其二是从激活服务器获取临时许可。在几种著名的激活工具中，KMSpico 使用非法的激活服务器向用户提供许可，Microsoft Toolkit 通过在本地伪造激活服务器完成这一过程，而 RemoveWAT 则是改写注册表和系统服务欺骗激活机制。理论上来说，上述适用于 Windows 7 的工具都可以完成 Thin PC 的激活任务。不过，由于 Thin PC 的设计缺陷，用户甚至可以在不使用任何工具的情况下徒手完成永久激活。下面就让我来详细讲解激活流程。

首先，进入控制面板，找到管理工具。



随后进入系统服务控制台。



找到软件保护服务。



禁用并停止此服务。



下一步，进入 C:\Windows\System32 系统目录，找到 slui.exe 这个可执行文件。它负责记录和变更系统的激活状态。我们要做的事情就是把这个文件干掉。



右键 => 属性 => 安全选项卡，我们可以看到各用户对该文件持有的权限。



点击下方的“高级”，编辑文件属主，将文件所有权交付给当前用户（也就是 Users）。





现在回到安全选项卡，编辑当前用户对文件的权限。



给用户加上完全控制的权限，结果如下图所示。



做完这一步，就可以移除程序并清空回收站了。



最后看看系统激活状态，会显示不可用。由于系统的逻辑问题，此时不会采取特别的措施，至此我们就实现了 Thin PC 的永久激活。

参考https://bitmingw.com/2017/04/24/windows-thin-pc-download-install-activate/

与虚拟机交互方面最好是通过远程桌面工具进行单应用链接

其他可通过无缝模式等
virtualBox无缝模式启用
通过设备 - 安装增强功能 如果还不能启用，手动执行虚拟机上我的电脑下VirtualBox Guest Additions下的VBoxWindowsAdditions.exe

如报模块未找到的错误可手动加载模块
modprobe vboxdrv

消息提醒
首先virtualBox虚拟机的硬盘要用固定大小，其次不能开机启动tim，否则会导致设置不能保存。
配置tim的消息提醒声音文件放在共享文件夹中
安装xfce4-notifyd
inotify
启动脚本

监控消息文件夹可能更精确

office组件
sudo pacman -S libreoffice-fresh libreoffice-fresh-zh-cn

安装 tmux, fish
tmux.conf
```bash
code@archlinux ~> more .tmux.conf
set -g prefix C-a
set -g base-index         1     # 窗口编号从 1 开始计数
set -g display-panes-time 10000 # PREFIX-Q 显示编号保留时间，单位 ms
set -g mouse              on    # 开启鼠标
set -g pane-base-index    1     # 窗格编号从 1 开始计数
set -g renumber-windows   on    # 关掉某个窗口后，编号重排
set-option -g default-shell "/usr/bin/fish"  #默认使用fish

setw -g allow-rename      off   # 禁止活动进程修改窗口名
setw -g automatic-rename  off   # 禁止自动命名新窗口

#配置复制模式
#前缀 [ 进入复制模式
#按 space 开始复制，移动光标选择复制区域
#按 Enter(与plugin-yank结合使用y可同时复制到系统剪贴板) 复制并退出copy-mode。
#将光标移动到指定位置，按 PREIFX ] 粘贴
setw -g mode-keys vi

bind C-v run "tmux set-buffer \"$(xclip -o -sel clipboard)\" ; tmux paste-buffer "

# -----------------------------------------------------------------------------
# 使用插件 - via tpm
#   1. 执行 git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
#   2. 执行 bash ~/.tmux/plugins/tpm/bin/install_plugins
# -----------------------------------------------------------------------------

setenv -g TMUX_PLUGIN_MANAGER_PATH '~/.tmux/plugins'

# 推荐的插件（请去每个插件的仓库下读一读使用教程）
# set -g @plugin 'seebi/tmux-colors-solarized'
set -g @plugin 'tmux-plugins/tmux-pain-control'
set -g @plugin 'tmux-plugins/tmux-prefix-highlight'
set -g @plugin 'tmux-plugins/tmux-resurrect'
# set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-yank'
set -g @plugin 'tmux-plugins/tpm'

# tmux-resurrect
set -g @resurrect-dir '~/.tmux/resurrect'

# tmux-prefix-highlight
set -g status-right '#{prefix_highlight} #H | %a %Y-%m-%d %H:%M'
set -g @prefix_highlight_show_copy_mode 'on'
set -g @prefix_highlight_copy_mode_attr 'fg=white,bg=blue'

# 初始化 TPM 插件管理器 (放在配置文件的最后)
run '~/.tmux/plugins/tpm/tpm'

# -----------------------------------------------------------------------------
# 结束
# -----------------------------------------------------------------------------
```
