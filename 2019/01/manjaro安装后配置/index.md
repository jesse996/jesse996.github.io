# Manjaro安装后配置


## 修改源

```bash
sudo pacman-mirrors -i -c China -m rank
```

### 设置 archlinuxcn 源。

修改 /etc/pacman.conf 最后增加

```bash
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
#Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

## 更新系统，安装 archlinuxcn-keyring:

```bash
sudo pacman -Syu
sudo pacman -S archlinuxcn-keyring
sudo pacman -Syu
```

## 修改 Home 下的目录为英文：

修改目录映射文件名；

```bash
gedit ~/.config/user-dirs.dirs
```

修改为以下内容：

```bash
XDG_DESKTOP_DIR="$HOME/Desktop"
XDG_DOWNLOAD_DIR="$HOME/Downloads"
XDG_TEMPLATES_DIR="$HOME/Templates"
XDG_PUBLICSHARE_DIR="$HOME/Public"
XDG_DOCUMENTS_DIR="$HOME/Documents"
XDG_MUSIC_DIR="$HOME/Music"
XDG_PICTURES_DIR="$HOME/Pictures"
XDG_VIDEOS_DIR="$HOME/Videos"
```

将 Home 目录下的中文目录名改为对应的中文名；

```bash
cd ~
mv 公共 Public
mv 模板 Templates
mv 视频 Videos
mv 图片 Pictures
mv 文档 Documents
mv 下载 Downloads
mv 音乐 Music
mv 桌面 Desktop
```

## 安装常用软件

### yay

```bash
sudo pacman -S yay
```

### JetBrains Mono 字体

```bash
yay -S ttf-jetbrains-mono
fc-cache -fv #刷新字体。
```

### vscode

```bash
yay -S visual-studio-code-bin
```

### gnome-keyring(用 vscode 自带的同步在 kde 环境下需要下载这个，不然报错：The name org.freedesktop.secrets was not provided by any .service files)

```bash
yay -S gnome-keyring
#密码设置为空，不然一打开vscode就会要你输入密码
#如果已经输入了密码，可以下载seahorse,再修改密码为空
```

### chome

```bash
yay -S google-chrome
```

### 设置 chrome 为默认浏览器:

在设置里面设置默认浏览器为 chrome，再把 html 关联类型设置为用 chrome 打开

### oh-my-zsh

```bash
#via curl
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

#或者 via wget
#sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/plugins/zsh-syntax-highlighting
#git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/plugins/zsh-syntax-highlighting
#sudo pacman -S autojump
```

设置`~/.zshrc`

```bash
ZSH_THEME="agnoster"
#export NVM_LAZY_LOAD=true

plugins=(
  git
  zsh-syntax-highlighting
  zsh-autosuggestions
  #autojump
  #zsh-nvm
)
```

最后刷新

```bash
source ~/.zshrc
```

### 输入法 fcitx5

```bash
yay -S fcitx5-im fcitx5-chinese-addons

echo "INPUT_METHOD  DEFAULT=fcitx5" > ~/.pam_environment
echo "GTK_IM_MODULE DEFAULT=fcitx5" >> ~/.pam_environment
echo "QT_IM_MODULE  DEFAULT=fcitx5" >> ~/.pam_environment
echo "XMODIFIERS    DEFAULT=\@im=fcitx5" >> ~/.pam_environment

cp /usr/share/applications/fcitx5.desktop ~/.config/autostart/ #自启动
```

### v2ray

```bash
yay -S v2ray qv2ray
```

### idea

```bash
yay -S intellij-idea-ultimate-edition
```

## rustup

```bash
yay -S rustup
rustup install stable
```

## qq/tim

```bash
yay -S deepin.com.qq.office #tim
yay -S deepin.com.qq.im #qq
#如果打不开，安装gnome-settings-daemon
#yay -S gnome-settings-daemon
#再设置/usr/lib/gsd-xsettings为自启动
#系统设置->开机或关机->自动启动->添加脚本->输入`/usr/lib/gsd-xsettings`
#重启
```

## Jdk

```bash
yay -S jdk

#查看jdk状态
archlinux-java status
#设置默认jdk
#sudo archlinux-java set java-14-jdk
```

### 网易云音乐

```bash
yay -S iease-music
```

## 美化

```bash
sudo pacman -S papirus-icon-theme
sudo pacman -S plank
sudo pacman -S variety
```

[KDE 美化和优化](https://zhuanlan.zhihu.com/p/100656626)

[美化](https://zhuanlan.zhihu.com/p/89847601)

