# Manjaro安装后配置


## 修改源

```
sudo pacman-mirrors -i -c China -m rank
sudo pacman -Syu
sudo pacman -S archlinux-keyring
```

### 设置 archlinuxcn 源。

修改 /etc/pacman.conf
最后增加

```
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch

```

## 更新系统

```
sudo pacman -Syu
```

## 安装`archlinuxcn-keyring:`

```
sudo pacman -S archlinuxcn-keyring
sudo pacman -Syu
```

---

## 修改 Home 下的目录为英文：

修改目录映射文件名；

```
gedit ~/.config/user-dirs.dirs
```

修改为以下内容：

```
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

```
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

重启系统。

---

## 安装 chrome

```
sudo pacman -S google-chrome
```

---

## 安装和使用 oh-my-zsh

### 安装 oh-my-zsh 的配置

查看本地有哪几种 shell

```
cat /etc/shells
```

manjaro 默认已经安装了 zsh

### 安装 oh-my-zsh 的配置文件

```
#via curl
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

#或者 via wget
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

### 安装插件

#### 安装 zsh-autosuggestions

```
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/plugins/zsh-autosuggestions
```

#### 安装 zsh-syntax-highlighting

```

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/plugins/zsh-syntax-highlighting
```

#### 安装 zsh-nvm

```
git clone https://github.com/lukechilds/zsh-nvm ~/.oh-my-zsh/custom/plugins/zsh-nvm
```

#### 安装 autojump

```
sudo pacman -S autojump
```

在`~/.zshrc`中找到

```
plugins=(
  git
)
```

在括号中 git 的下一行添加插件名称使其生效

```
plugins=(
  git
  zsh-syntax-highlighting
  zsh-autosuggestions
  autojump
  zsh-nvm
)
```

### 更改主题&&lazyload nvm

```
gedit ~/.zshrc
```

```
ZSH_THEME="agnoster"
export NVM_LAZY_LOAD=true
```

刷新配置，使之生效

```
 source ~/.zshrc
```

---

### 设置 chrome 为默认浏览器:

在设置中把 html 关联类型设置为用 chrome 打开

