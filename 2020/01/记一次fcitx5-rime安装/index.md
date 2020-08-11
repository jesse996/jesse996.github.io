# 记一次fcitx5 Rime安装


# 安装

```
sudo pacman -S fcitx5-git  fcitx5-qt5-git kcm-fcitx5-git fcitx5-rime-git
```

# 配置

然后打开 fctix5 配置程序，添加`中州*`，就是 rime。
添加一下内容到`~/.xprofile`

```
export GTK_IM_MODULE=fcitx5
export QT_IM_MODULE=fcitx5
export XMODIFIERS=@im=fcitx
```

fctix5 不会自动启动，添加一下内容到`~/.xprofile`：

```
fcitx5 &
```

KDE 用户可以直接在系统设置模块-自动启动设置

# 双拼 or 五笔

在`~/.local/share/fcitx5/rime/`中 parse 下面 github 中的文件

[https://github.com/jesse996/squirrel_config](https://github.com/jesse996/squirrel_config)

### 皮肤

**修改皮肤**：[传送门](https://github.com/iovxw/fcitx5-simple-theme)

