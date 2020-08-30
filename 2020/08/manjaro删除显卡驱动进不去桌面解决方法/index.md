# Manjaro删除显卡驱动进不去桌面解决方法


今天不小心删除了显卡的驱动，导致开机进不去系统。以下是解决方法：

1.`CTRL+ALT+F3` 进入 tty,输入用户名和密码登陆。  
2. 安装驱动

```bash
sudo mhwd -a pci nonfree 0300
```

3.删除`/etx/X11/xorg.conf`,`/etx/X11/xorg.conf.d`

