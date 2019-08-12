---
layout:     post
title:      Ubuntu-desktop安装后操作备忘
subtitle:   
date:       2019-04-15
author:     gary
header-img: 
catalog: true
tags:
    - Linux
---

# 初始化root密码
sudo passwd

# 修改源
备份
```
cp /etc/apt/sources.list /etc/apt/sources.list.bak
```
然后修改:
aliyun的见 https://opsx.alibaba.com/guide?lang=zh-CN&document=69a2341e-801e-11e8-8b5a-00163e04cdbb，例如：
```
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

#deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
#deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

默认和security的一定开，update看心情，backports的谨慎开，proposed的关掉。引用网上的说明：
- security：仅修复漏洞，并且尽可能少的改变软件包的行为
- update：修复严重但不影响系统安全运行的漏洞，这类补丁在经过QA人员记录和验证后才提供
- backports：backports的团队则认为最好的更新策略是security策略加上新版本的软件（包括候选版本的）。但不会由Ubuntu security team审查和更新。https://help.ubuntu.com/community/UbuntuBackports

也可https://mirrors.ustc.edu.cn/repogen/,以及清华等

## 速度
- ubuntu cn(默认) 280ms+
- ustc 43ms+
- 清华 46ms+
- aliyun 30ms+

# 安装常用软件
## 必备
```
sudo apt-get install net-tools gcc vim git
sudo apt-get install tree iftop sysstat
```

### 配置vim

sudo vi /etc/vim/vimrc.local
```
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
""实用设置
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
" 设置当文件被改动时自动载入
set autoread
" quickfix模式
autocmd FileType c,cpp map <buffer> <leader><space> :w<cr>:make<cr>
"代码补全
set completeopt=preview,menu
"允许插件  
filetype plugin on
"共享剪贴板  
set clipboard=unnamed
"从不备份  
set nobackup
"make 运行
:set makeprg=g++\ -Wall\ \ %
"自动保存
set autowrite
set ruler                   " 打开状态栏标尺
set cursorline              " 突出显示当前行
set magic                   " 设置魔术
set guioptions-=T           " 隐藏工具栏
set guioptions-=m           " 隐藏菜单栏
"set statusline=\ %<%F[%1*%M%*%n%R%H]%=\ %y\ %0(%{&fileformat}\ %{&encoding}\ %c:%l/%L%)\
" 设置在状态行显示的信息
set foldcolumn=0
set foldmethod=indent
set foldlevel=3
set foldenable              " 开始折叠
" 不要使用vi的键盘模式，而是vim自己的
set nocompatible
" 语法高亮
set syntax=on
" 去掉输入错误的提示声音
set noeb
" 在处理未保存或只读文件的时候，弹出确认
set confirm
" 自动缩进
set autoindent
set cindent
" Tab键的宽度
set tabstop=4
" 统一缩进为4
set softtabstop=4
set shiftwidth=4
" 不要用空格代替制表符
set noexpandtab
" 在行和段开始处使用制表符
set smarttab
" 显示行号
set number
" 历史记录数
set history=1000
"禁止生成临时文件
set nobackup
set noswapfile
"搜索忽略大小写
set ignorecase
"搜索逐字符高亮
set hlsearch
set incsearch
"行内替换
set gdefault
"编码设置
set enc=utf-8
set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936
"语言设置
set langmenu=zh_CN.UTF-8
set helplang=cn
" 我的状态行显示的内容（包括文件类型和解码）
set statusline=%F%m%r%h%w\ [FORMAT=%{&ff}]\ [TYPE=%Y]\ [POS=%l,%v][%p%%]\ %{strftime(\"%d/%m/%y\ -\ %H:%M\")}
set statusline=[%F]%y%r%m%*%=[Line:%l/%L,Column:%c][%p%%]
" 总是显示状态行
set laststatus=2
" 命令行（在状态行下）的高度，默认为1，这里是2
set cmdheight=2
" 侦测文件类型
filetype on
" 载入文件类型插件
filetype plugin on
" 为特定文件类型载入相关缩进文件
filetype indent on
" 保存全局变量
set viminfo+=!
" 带有如下符号的单词不要被换行分割
set iskeyword+=_,$,@,%,#,-
" 字符间插入的像素行数目
set linespace=0
" 增强模式中的命令行自动完成操作
set wildmenu
" 使回格键（backspace）正常处理indent, eol, start等
set backspace=2
" 允许backspace和光标键跨越行边界
set whichwrap+=<,>,h,l
" 可以在buffer的任何地方使用鼠标（类似office中在工作区双击鼠标定位）
set mouse=a
set selection=exclusive
set selectmode=mouse,key
" 通过使用: commands命令，告诉我们文件的哪一行被改变过
set report=0
" 在被分割的窗口间显示空白，便于阅读
set fillchars=vert:\ ,stl:\ ,stlnc:\
" 高亮显示匹配的括号
set showmatch
" 匹配括号高亮的时间（单位是十分之一秒）
set matchtime=1
" 光标移动到buffer的顶部和底部时保持3行距离
set scrolloff=3
" 为C程序提供自动缩进
set smartindent
" 高亮显示普通txt文件（需要txt.vim脚本）
 au BufRead,BufNewFile *  setfiletype txt
"自动补全
:inoremap ( ()<ESC>i
:inoremap ) <c-r>=ClosePair(')')<CR>
":inoremap { {<CR>}<ESC>O
":inoremap } <c-r>=ClosePair('}')<CR>
:inoremap [ []<ESC>i
:inoremap ] <c-r>=ClosePair(']')<CR>
:inoremap " ""<ESC>i
:inoremap ' ''<ESC>i
function! ClosePair(char)
    if getline('.')[col('.') - 1] == a:char
        return "\<Right>"
    else
        return a:char
    endif
endfunction
filetype plugin indent on
"打开文件类型检测, 加了这句才可以用智能补全
set completeopt=longest,menu
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
```

### 配置git
```
git config --global user.name "gary0416"
git config --global user.email "408036296@163.com"

git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
```

### 其它配置
```
# 开ssh远程
sudo apt install openssh-server
sudo vi /etc/ssh/sshd_config (解开22端口)
sudo /etc/init.d/ssh restart

# ssh防止空闲中断(注意不是sshd)
sudo vi /etc/ssh/ssh_config
# 后面追加:
ServerAliveInterval 30

# 快速启动(注意pgp key)
sudo sh -c "echo 'deb http://download.opensuse.org/repositories/home:/manuelschneid3r/xUbuntu_18.04/ /' > /etc/apt/sources.list.d/home:manuelschneid3r.list"
wget -O - http://download.opensuse.org/repositories/home:/manuelschneid3r/xUbuntu_18.04/Release.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install albert
# 然后打开,设置快捷键,例如alt+r,然后extentions勾选Applications、System、WebSearch

# 解决:手动设置dns无效,网络切换时总被改到127.0.0.53
sudo apt install resolvconf
修改/etc/resolvconf/resolv.conf.d/tail,增加nameserver 自定义dns.
https://askubuntu.com/a/1012648

# 增加新建文档时的模板
## 文本文档
touch ~/模板/text.txt
## bash脚本
cat << EOF > ~/模板/script.sh
#!/bin/bash
set -o nounset
set -o errexit
#set -o verbose
#set -o xtrace
EOF

# dconf
## nautilus默认使用位置输入框，不用ctrl+l
找到/org/gnome/nautilus/preferences下的always-use-location-entry，设置自定义值true
## 上方日历里显示周
找到/org/gnome/desktop/calendar/下的show-weekdate，打开
```

## 挂载ntfs
```
apt-get install ntfs-config
```
打开后自动配置,会挂到/Media下,分区的Label是文件夹名,且开机自启

## gitkraken
```
sudo snap install gitkraken
网速太慢就proxychains4 wget https://release.axocdn.com/linux/gitkraken-amd64.deb
```

## docker
### docker ce
```
# 推荐,一条命令
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
# 或下面的(翻墙都慢)
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### docker-compose
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

### 非docker用户使用docker
```
sudo usermod -aG docker $USER
```

### kitematic
```
wget https://github.com/docker/kitematic/releases/download/v0.17.7/Kitematic-0.17.7-Ubuntu.zip && unzip Kitematic-0.17.7-Ubuntu.zip
sudo dpkg -i Kitematic-0.17.7-Ubuntu.deb
```

## kubectl
```
sudo snap install kubectl --classic
```

### krew及插件
```
(
  set -x; cd "$(mktemp -d)" &&
  curl -fsSLO "https://storage.googleapis.com/krew/v0.2.1/krew.{tar.gz,yaml}" &&
  tar zxvf krew.tar.gz &&
  ./krew-"$(uname | tr '[:upper:]' '[:lower:]')_amd64" install \
    --manifest=krew.yaml --archive=krew.tar.gz
)
追加PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"

kubectl krew install sniff
```

## jdk
### 多jdk切换
```
sudo update-alternatives --config java
```

### jdk8
```
apt-get install openjdk-8-jdk
sudo vi /etc/profile
追加
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
export PATH="$PATH:$JAVA_HOME/bin"
export JRE_HOME="$JAVA_HOME/jre"
export CLASSPATH=".:$JAVA_HOME/lib:$JRE_HOME/lib"
```

### jdk11
```
sudo add-apt-repository ppa:openjdk-r/ppa \
&& sudo apt-get update -q \
&& sudo apt install -y openjdk-11-jdk
```
注：直接apt install安装openjdk-11-jdk的话，实际安装的是10，见https://stackoverflow.com/questions/52504825/how-to-install-jdk-11-under-ubuntu

## jmc
```
sudo apt-get install mercurial

cd ~/soft && proxychains4 hg clone http://hg.openjdk.java.net/jmc/jmc/
cd jmc/docker && proxychains4 docker-compose up
# docker_jmc_1 exited with code 0 后Ctrl-C退出，对下面的路径创建快捷方式
target/products/org.openjdk.jmc/linux/gtk/x86_64/jmc
```

## idea
goland,pycharm同理
### 安装
```
下载idea
tar xzvf解压
bin/idea.sh
然后在tools里点击创建快捷方式
字体用Source Code Pro
```

### 解决快捷键冲突
#### Ctrl+Alt+左 和 Ctrl+Alt+右:
```
gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-left "[]"
gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-right "[]"
```

#### Ctrl+Alt+B
Fcitx 输入法.全局配置,显示高级选项.找到 切换虚拟键盘 点击按 Esc 取消并重启

## ssh快速登录各种服务器
```
apt install expect
https://github.com/jiangxianli/SSHAutoLogin
配置~/.sshloginrc
vi ~/.zshrc
alias s="/home/zhangtb/soft/my-scripts/SSHAutoLogin/ssh_login"
```

## wps
```
sudo apt-get remove libreoffice-common
wget http://kdl.cc.ksosoft.com/wps-community/download/6758/wps-office_10.1.0.6758_amd64.deb
sudo dpkg -i wps-office_10.1.0.6758_amd64.deb
```
wps字体https://www.cnblogs.com/EasonJim/p/7146587.html

## shadowsocks
1.见Linux Shadowsocks
2.https://github.com/qingshuisiyuan/electron-ssr-backup.配置里取消快捷键.启动里增加执行electron-ssr-0.2.6.AppImage

## chrome
```
sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list'
sudo apt install google-chrome-stable

# 忽略HTTPS的警告，开启远程debug(配合VS Code)
sudo vi /usr/share/applications/google-chrome.desktop
找到几处Exec,后面增加参数 --test-type --ignore-certificate-errors --remote-debugging-port=9222
```

## 搜狗输入法
除了默认的输入法可以用（按shift就能切换，也挺省事），还可以用搜狗。
https://pinyin.sogou.com/linux/?r=pinyin
```
sudo apt-get install fcitx-table
sudo apt remove ibus*
wget http://cdn2.ime.sogou.com/dl/index/1524572264/sogoupinyin_2.2.0.0108_amd64.deb?st=40e2TQUgb8zXglZb_4LIgQ&e=1553588145&fn=sogoupinyin_2.2.0.0108_amd64.deb
sudo dpkg -i sogoupinyin_2.2.0.0108_amd64.deb
```
重启，然后删除其他配置，第一个是键盘-中文，将搜狗放在第二个。偶尔会出现输入法显示异常，点击上面中间时间旁的搜狗图标，随便选个其它皮肤，再切换回来就正常了。

## virtualbox
```
wget https://download.virtualbox.org/virtualbox/6.0.4/virtualbox-6.0_6.0.4-128413~Ubuntu~bionic_amd64.deb
sudo dpkg -i virtualbox-6.0_6.0.4-128413~Ubuntu~bionic_amd64.deb
必要时候apt --fix-broken install
```

## postman
```
sudo apt-get install libcanberra-gtk-module
wget https://dl.pstmn.io/download/latest/linux64
tar xzvf Postman-linux-x64-7.0.7.tar.gz
rm Postman-linux-x64-7.0.7.tar.gz
mv Postman ../soft
# 创建application(或/home/zhangtb/.local/share/applications/下,snap的在/var/lib/snapd/desktop/applications)
sudo vim /usr/share/applications/postman.desktop
[Desktop Entry]
Encoding=UTF-8
Name=Postman
Exec=/home/zhangtb/soft/Postman/Postman
Icon=/home/zhangtb/soft/Postman/app/resources/app/assets/icon.png
Terminal=false
Type=Application
Categories=Development;
```

## wine
见wine,可配合playonlinux使用,安装后可自定义wine版本.
推荐使用deepin-wine，可安装Foxmail等。

## typora(markdown)
```
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt-get update
sudo apt-get install typora
```

## 截图
```
推荐ubuntu软件中搜索深度截图并安装，快捷键启动命令/usr/bin/deepin-screenshot
# flameshot备选
apt install flameshot
配置文件格式:Snap_%Y-%m-%d_%H%M%S,开机启动
进入系统设置-设备-键盘，选择添加自定义快捷键，设置快捷键的命令/usr/bin/flameshot gui和名称FlameShot。然后绑定一下键盘Ctrl+Alt+A
```

## Sublime Text
```
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
sudo apt-get update && sudo apt-get install sublime-text
```
然后安装Package Controll。 https://packagecontrol.io/installation

设置代理，解决梯子问题：
```
Preferences > Package Settings > Package Control > Settings - User，增加：
"http_proxy": "http://127.0.0.1:12333",
"https_proxy": "http://127.0.0.1:12333",
```

Preferences > Package Control，选Install，然后输入插件名安装。
- Pretty JSON  快捷键：Ctrl + Alt + J
- compare Side-By-Side
- A File Icon
- ConvertToUTF8
- Codecs33

配置
```
{
  "auto_find_in_selection": true,
  "auto_match_enabled": true,
  "color_scheme": "Packages/Color Scheme - Default/Mariana.sublime-color-scheme",
  "default_encoding": "UTF-8",
  "default_line_ending": "unix",
  "draw_white_space": "all",
  "font_face": "Source Code Pro",
  "font_size": 14,
  "highlight_line": true,
  "ignored_packages":
  [
    "Vintage"
  ],
 "show_encoding": true,
  "show_line_endings": true,
  "tab_size": 4,
  "theme": "Default.sublime-theme",
  "translate_tabs_to_spaces": true,
  "word_wrap": true
}
```

## notepad++
```
sudo snap install notepad-plus-plus
sudo snap connect notepad-plus-plus:removable-media
# Mandatory Plug
sudo snap connect notepad-plus-plus:process-control
# Optional Plugs
sudo snap connect notepad-plus-plus:hardware-observe
sudo snap connect notepad-plus-plus:cups-control
```

## proxychains4
```
sudo git clone https://github.com/rofl0r/proxychains-ng.git
sudo apt-get install gcc make
cd proxychains-ng
sudo ./configure --prefix=/usr --sysconfdir=/etc
sudo make install
# installs proxychains.conf
sudo make install-config
```
只需要在 /etc/proxychains.conf 保持以下几行能用就OK：
```
strict_chain
proxy_dns
tcp_read_time_out 15000
tcp_connect_time_out 8000
[ProxyList]
socks5 127.0.0.1 1080
```
注意ss要开启允许其它设备连入. 这样 socks5 连接到 1080 端口之后，就可以在命令前面加上 proxychains4 来让程序走 socks5 代理。例如 proxychains4 curl ip.gs可看到是美国

## oh-my-zsh
```
sudo apt-get install zsh
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
vi ~/.zshrc
# 设置
ZSH_THEME="ys"
plugins=(
  git
  z
  kubectl
  zsh-autosuggestions
  zsh-syntax-highlighting
)

source ~/.zshrc
chsh -s $(which zsh)
# 恢复用chsh -s /bin/bash
```

## linux版飞秋
1. apt方式安装的是0.7.4旧版，不兼容飞秋。
2. 目前0.7.6版仍然有BUG，导入网络IP段无法保存。不支持接收飞秋的图片，其余功能正常。
3. git最后Commits on Jun 22, 2019（pr284）。保存聊天记录则闪退，不支持最小化到托盘，不支持收文件，已解决IP段无法保存。
4. 综上，使用第二种方法，0.7.6版：
```
wget http://ftp.br.debian.org/debian/pool/main/i/iptux/iptux_0.7.6-1_amd64.deb
sudo dpkg -i iptux_0.7.6-1_amd64.deb
# 打开首选项，系统，首选网络编码设置成GBK，勾选自动打开聊天窗口，重启iptux
```

附：3.git源码编译
```
# 解决libglog依赖
git clone https://github.com/google/glog.git && cd glog
sudo ./autogen.sh && sudo ./configure && sudo make && sudo make install

https://github.com/iptux-src/iptux.git && cd iptux
meson builddir && ninja -C builddir
sudo ninja -C builddir install
sudo ldconfig
iptux运行
```

## 字体
### Arial
```
sudo apt-get install ttf-mscorefonts-installer
# 如果遇到 下载额外数据文件失败，其实是/usr/share/package-data-downloads有一个文件ttf-mscorefonts-installer，有一大串地址，手动下载下来，然后放到一个文件夹中。也可以在百度网盘里下载。
# sudo dpkg-reconfigure ttf-mscorefonts-installer然后输入刚才手动下载的文件夹，即可正常安装
# 重建缓存
sudo fc-cache -f -v
# 查看是否已安装成功
sudo fc-match Arial
# 清理
sudo rm /usr/share/package-data-downloads/ttf-mscorefonts-installer && rm /var/lib/update-notifier/package-data-downloads/ttf-mscorefonts-installer && rm /var/lib/update-notifier/user.d/data-downloads-failed
```

### 其他字体
在这搜索并下载 http://www.zitixiazai.org/，例如：宋体，微软雅黑，Courier New等。下载后的字体都放下面新建这个文件夹下。
```
sudo mkdir /usr/share/fonts/my
# 重建缓存
sudo mkfontscale && sudo mkfontdir && sudo fc-cache -fv
```

### Adobe Source Code Pro
```
[ -d /usr/share/fonts/my ] || sudo mkdir /usr/share/fonts/my
# 太大，改用下面那句 sudo git clone https://github.com/adobe-fonts/source-code-pro.git /usr/share/fonts/my/source-code-pro
wget https://github.com/adobe-fonts/source-code-pro/archive/release.zip && unzip release.zip && rm release.zip

fc-list  | grep "Source Code Pro"
```

### monaco
```
git clone https://github.com/cstrap/monaco-font
cd monaco-font
./install-font-ubuntu.sh http://www.gringod.com/wp-upload/software/Fonts/Monaco_Linux.ttf
```

## python相关
```
sudo apt-get install python3-pip python3-venv

mkdir ~/.pip/ && vi ~/.pip/pip.conf
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
```

## sdkman
```
curl -s "https://get.sdkman.io" | bash
```

## terminator
```
sudo apt-get install terminator
#dconf里exec从gnome-terminal改为terminator
gsettings set org.gnome.desktop.default-applications.terminal exec terminator

# 配置~/.config/terminator/config:
[global_config]
  always_split_with_profile = True
  enabled_plugins = CustomCommandsMenu, ActivityWatch, LaunchpadCodeURLHandler, APTURLHandler, Logger, MavenPluginURLHandler, LaunchpadBugURLHandler
  focus = system
  handle_size = 0
  suppress_multiple_term_dialog = True
  title_transmit_bg_color = "#555753"
  window_state = maximise
[keybindings]
  edit_tab_title = None
  hide_window = None
  switch_to_tab_1 = <Alt>1
  switch_to_tab_2 = <Alt>2
  switch_to_tab_3 = <Alt>3
  switch_to_tab_4 = <Alt>4
  switch_to_tab_5 = <Alt>5
[layouts]
  [[default]]
    [[[child1]]]
      parent = window0
      profile = default
      type = Terminal
    [[[window0]]]
      parent = ""
      type = Window
[plugins]
[profiles]
  [[default]]
    background_darkness = 0.95
    background_type = transparent
    cursor_color = "#2D2D2D"
    font = Source Code Pro 13
    foreground_color = "#ffffff"
    palette = "#000000:#cc0000:#4e9a06:#c4a000:#3465a4:#75507b:#06989a:#d3d7cf:#555753:#ef2929:#8ae234:#fce94f:#729fcf:#ad7fa8:#34e2e2:#eeeeec"
    scrollback_lines = 3000
    show_titlebar = False
    use_system_font = False
```

## wireshark
```
sudo add-apt-repository ppa:wireshark-dev/stable
sudo apt-get update && sudo apt-get install wireshark
# 然后选yes，或运行下面的一句
sudo dpkg-reconfigure wireshark-common
sudo adduser $USER wireshark
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/dumpcap
```

## snap商店
https://snapcraft.io/
安装只需一条命令，方便。

## ubuntu软件中安装
- pac-vs（ssh工具备用,已由SSHAutoLogin脚本代替）
- MySQL Workbench（备用，仍然主要使用Navicat）

## 其他
安装:
- vscode
- Navicat premium(https://www.jianshu.com/p/5f693b4c9468.乱码：界面、编辑器、记录字体用Noto Sans mono CJK SC Regular)
- smartsvn(授权文件https://blog.csdn.net/liuayng/article/details/70311844)
- 安装并配置maven，gradle，golang

```
# 桌面特效
sudo apt-get install compiz compiz-plugins compizconfig-settings-manager cairo-dock
#开机启动：在启动应用程序添加 #名称与命令中都添加cairo-dock (此dock已由albert替代)

# Dock再次点击时最小化
gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'minimize'
#恢复：gsettings reset org.gnome.shell.extensions.dash-to-dock click-action

# 设置工作区等特性
sudo apt-get install unity-tweak-tool

# gufw图形化管理防火墙
sudo apt-get install gufw

# ubuntu-make
sudo apt-get install ubuntu-make

# 网速显示(可用gnome扩展Simple net speed替换)
# 见https://github.com/GGleb/indicator-netspeed-unity

# gnome扩展
sudo apt-get install chrome-gnome-shell
#然后可以从https://extensions.gnome.org/local/安装,用FF打开并安装附加组件,FF的扩展利用上面安装的shell
# 挑好之后,切换on和off即可安装,例如:
# Coverflow Alt-Tab
# Simple net speed (切换单位到B/s)
# New Mail Indicator

# 可选
sudo apt install gnome-tweak-tool  把上面一行时间旁的日期打开

# vnc
# https://websiteforstudents.com/access-ubuntu-18-04-lts-beta-desktop-via-vnc-from-windows-machines/
# 设置里启用远程即可,然后mstsc,输入ip,选vnc any,输入密码.或使用vnc viewer(windows)/remmina(linux自带)

# Stacer(系统清理和优化工具)
sudo add-apt-repository ppa:oguzhaninan/stacer -y
sudo apt-get update
sudo apt-get install stacer -y

# 解决桌面无法显示托盘图标
sudo apt-get install gnome-shell-extension-top-icons-plus gnome-tweaks
然后打开gnome-tweaks，扩展里启用Topicons Plus

# thunderbird
主题使用Monterail Dark
安装插件https://github.com/Ximi1970/FireTray/releases

# container-diff
curl -LO https://storage.googleapis.com/container-diff/latest/container-diff-linux-amd64 && chmod +x container-diff-linux-amd64 && sudo mv container-diff-linux-amd64 /usr/local/bin/container-diff
```
