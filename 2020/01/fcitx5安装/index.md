# fcitx5安装


## 安装 fcitx5

```bash
yay -S fcitx5-im
```

## 安装输入法

输入法可以选 `rime`,也可以选 `fcitx5-chinese-addons`

如果要用小鹤双拼，`rime` 要下载配置文件，而后者不需要

```bash
yay -S fcitx5-chinese-addons
#yay -S fcitx5-rime    //rime
```

## 配置

添加一下内容到`~/.pam_environment`

```
INPUT_METHOD  DEFAULT=fcitx5
GTK_IM_MODULE DEFAULT=fcitx5
QT_IM_MODULE  DEFAULT=fcitx5
XMODIFIERS    DEFAULT=\@im=fcitx5
```

## 设置开机自启动

实测在 kde 环境下不会自动启动，执行以下命令即可自启动

```bash
cp /usr/share/applications/fcitx5.desktop ~/.config/autostart/
```

也可以直接在系统设置模块-自动启动设置

## rime 设置（双拼等）

在`~/.local/share/fcitx5/rime/`中 parse 下面 github 中的文件

[https://github.com/jesse996/squirrel_config](https://github.com/jesse996/squirrel_config)

## 皮肤

**修改皮肤**：[传送门](https://github.com/iovxw/fcitx5-simple-theme)

## 单行模式

单行模式就是输入时没有候选子上面的拼音。

在设置中，启用“ 在程序中显示预编辑文本 ”即可启用单行模式

