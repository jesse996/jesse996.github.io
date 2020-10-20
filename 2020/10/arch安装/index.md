# Arch安装


**主要还是依据 arch wiki 来安装，这个只是作为一个参考**

### **验证启动模式（BIOS 还是 EFI）**

要验证启动模式，请用下列命令列出  **[efivars](https://wiki.archlinux.org/index.php/Efivars)**  目录：

```bash
# ls /sys/firmware/efi/efivars
```

如果命令没有错误地显示了目录，则系统以 UEFI 模式启动。 如果目录不存在，系统可能以  **[BIOS](https://en.wikipedia.org/wiki/BIOS)**  模式 (或  **[CSM](https://en.wikipedia.org/wiki/Compatibility_Support_Module)**  模式) 启动。

### **连接到因特网**

用下面步骤设置网络：

-   确保系统已经启用了  **[网络接口](https://wiki.archlinux.org/index.php/Network_configuration#Network_interfaces)**，用  **[ip-link(8)](https://jlk.fjfi.cvut.cz/arch/manpages/man/ip-link.8)**  检查:

    ```bash
    ip link
    ```

-   对于无线网络，请确保无线网卡未被  **[rfkill](https://wiki.archlinux.org/index.php/Rfkill)**  禁用。
-   要连接到网络:
    -   有线以太网 —— 连接网线
    -   WiFi —— 使用 **[iwctl](https://wiki.archlinux.org/index.php/Iwctl)** 验证无线网络
-   配置网络连接:
    -   **[DHCP](https://wiki.archlinux.org/index.php/DHCP)**: 动态 IP 地址和 DNS 服务器分配 (由  **[systemd-networkd](https://wiki.archlinux.org/index.php/Systemd-networkd)**  和  **[systemd-resolved](https://wiki.archlinux.org/index.php/Systemd-resolved)**  提供) 对于  **[有线](https://gitlab.archlinux.org/archlinux/archiso/-/blob/master/configs/releng/airootfs/etc/systemd/network/20-ethernet.network)**  和  **[无线](https://gitlab.archlinux.org/archlinux/archiso/-/blob/master/configs/releng/airootfs/etc/systemd/network/20-wireless.network)**  网络接口来说应该能开箱即用。
    -   静态 IP 地址: 按照  **[Network configuration#Static IP address](https://wiki.archlinux.org/index.php/Network_configuration#Static_IP_address)**  进行操作。
-   用  **[ping](<https://en.wikipedia.org/wiki/ping_(networking_utility)>)**  检查网络连接:

```bash
ping archlinux.org
```

### **更新系统时间**

使用  **[timedatectl(1)](https://jlk.fjfi.cvut.cz/arch/manpages/man/timedatectl.1)**  确保系统时间是准确的：

```bash
timedatectl set-ntp true
```

可以使用  `timedatectl status`  检查服务状态。

### **建立硬盘分区**

磁盘若被系统识别到，就会被分配为一个**[块设备](https://en.wikipedia.org/wiki/zh:%E8%AE%BE%E5%A4%87%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F#.E5.91.BD.E5.90.8D.E7.BA.A6.E5.AE.9A)**，如  `/dev/sda`, `/dev/nvme0n1`  或  `/dev/mmcblk0`。可以使用  **[lsblk](https://wiki.archlinux.org/index.php/Lsblk)**  或者  fdisk  查看：

```bash
fdisk -l

```

结果中以  `rom`，`loop`  或者  `airoot`  结束的可以被忽略

对于一个选定的设备，以下的分区是必须要有的：

-   一个根分区（挂载在  **[根目录](https://en.wikipedia.org/wiki/Root_directory)**）`/`；
-   要在  **[UEFI](https://wiki.archlinux.org/index.php/UEFI)**  模式中启动，还需要一个  **[EFI 系统分区](https://wiki.archlinux.org/index.php/EFI_system_partition)**。

用 cfdisk 分区

### **分区示例**

UEFI：
`/` 、 `swap` 、`/boot`

### **格式化分区**

当分区建立好了，这些分区都需要使用适当的  **[文件系统](<https://wiki.archlinux.org/index.php/File_systems_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>)**  进行格式化。举个例子，如果根分区在  `/dev/sdX1`  上并且要使用 Ext4 文件系统，运行：

```bash
 mkfs.ext4 /dev/sdX1
```

如果创建了  **[交换分区](<https://wiki.archlinux.org/index.php/Swap_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>)** (例如  `/dev/sdX2`)，请使用  **[mkswap(8)](https://jlk.fjfi.cvut.cz/arch/manpages/man/mkswap.8)**  将其初始化：

```bash
 mkswap /dev/sdX2
 swapon /dev/sdX2
```

### **挂载分区**

将根分区**[挂载](https://wiki.archlinux.org/index.php/Mount)**到  `/mnt`，例如：

```bash
mount /dev/sdX1 /mnt
#mkdir /mnt/boot  #如果没有就要先创建这个文件夹
mount /dev/sdX2 /mnt/boot
```

然后使用  **[mkdir(1)](https://jlk.fjfi.cvut.cz/arch/manpages/man/mkdir.1)**  创建其他剩余的挂载点（比如  `/mnt/efi`）并挂载其相应的分区。

稍后  **[genfstab(8)](https://jlk.fjfi.cvut.cz/arch/manpages/man/genfstab.8)**  将自动检测挂载的文件系统和交换空间。

生成分区表：

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

由于这步比较重要，所以我们需要输出生成的文件来检查是否正确，执行以下命令：

```bash
cat /mnt/etc/fstab
```

应该有 3 个，`/`,`/boot`,`swap`

### **选择镜像**

文件  `/etc/pacman.d/mirrorlist`  定义了软件包会从哪个**[镜像源](https://wiki.archlinux.org/index.php/Mirrors)**下载。在 LiveCD 启动的系统上，在连接到因特网后，**[reflector](https://wiki.archlinux.org/index.php/Reflector)**  会通过选择最近一个小时已同步的 HTTPS 镜像并按下载速率对其进行排序来更新镜像列表。

在列表中越前的镜像在下载软件包时有越高的优先权。您或许想检查一下文件，看看是否满意。如果不满意，可以相应的修改  `/etc/pacman.d/mirrorlist`  文件，并将地理位置最近的镜像源挪到文件的头部，同时也应该考虑一些其他标准。

这个文件接下来还会被  pacstrap  拷贝到新系统里，所以请确保设置正确。

### **安装必须的软件包**

使用  **[pacstrap](https://git.archlinux.org/arch-install-scripts.git/tree/pacstrap.in)**  脚本，安装  **[base](https://www.archlinux.org/packages/?name=base)**  软件包和 Linux **[内核](https://wiki.archlinux.org/index.php/Kernel)**以及常规硬件的固件：

```bash
pacstrap /mnt base linux linux-firmware base-devel
```

除了这些包以外，我们还可以安装许多其他的包，其中一个比较重要的就是一定要装上一个网络管理器，不然你的新系统启动以后连不上网就傻了。

```bash
pacstrap /mnt networkmanager vim
```

networkmanager 用来联网，vim 用来编辑配置文件。只要能连上网，别的软件都可以晚点装，所以目前装这么多东西就足够了。

进入新系统：

```bash
arch-chroot /mnt
```

这里顺便说一下，如果以后我们的系统出现了问题，只要插入 U 盘并启动， 将我们的系统根分区挂载到了/mnt 下（如果有 efi 分区也要挂载到/mnt/boot 下），再通过这条命令就可以进入我们的系统进行修复操作。

设置时区和硬件时间：

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systoh
```

本地化设置，使用 vim 打开`/etc/locale.gen`，把`en_US.UTF-8 UTF-8`和`zh_CN.UTF-8 UTF-8`取消注释，然后保存退出。运行下面两条命令：

```bash
locale-gen
echo LANG=en_US.UTF-8 >> /etc/locale.conf
```

设置网络(这里的 myhostname 取一个自己喜欢的名字)：

```bash
echo myhostname >> /etc/hostname
```

使用 vim 打开/etc/hosts，在里面输入（myhostname 换成上面自己取的名字）：

```bash
127.0.0.1	localhost
::1	localhost
127.0.1.1	myhostname.localdomain	myhostname
```

设置 root 用户的密码（连输两遍，输入时无显示）：

```bash
passwd
```

intel 的 CPU 安装`intel-ucode`，amd 的 CPU 安装`amd-ucode`。如果不太确定自己的 cpu 型号，可以安装一个`neofetch`，然后使用`neofetch`进行查看。

```bash
*# 可选项*
pacman -S neofetch
neofetch *# 查看cpu和显卡信息# 必选项*
pacman -S intel-ucode
pacman -S amd-ucode *# 根据实际情况安装*
```

## **安装`Bootloader`**

这里我们安装最流行的`Grub2`。**（如果曾经装过`Linux`，记得删掉原来的`Grub`，否则不可能成功启动）**

-   首先安装`os-prober`和`ntfs-3g`这两个包，它可以配合`Grub`检测已经存在的系统，自动设置启动选项。

```bash
pacman -S os-prober ntfs-3g
```

---

### **如果为 BIOS/MBR 引导方式：**

-   安装`grub`包：

```bash
pacman -S grub
```

-   部署 grub

```bash
grub-install --target=i386-pc /dev/sdx （将sdx换成你安装的硬盘）
```

-   生成配置文件：

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

---

### **如果为 EFI/GPT 引导方式：**

-   安装`grub`与`efibootmgr`两个包：

```bash
pacman -S grub efibootmgr dialog
pacman -S network-manager-applet   mtools dosfstools base-devel linux-headers
```

-   部署`grub`：

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
```

-   生成配置文件：

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

如果报`warning failed to connect to lvmetad，falling back to device scanning.`错误。参照[这篇文章](https://www.pckr.co.uk/arch-grub-mkconfig-lvmetad-failures-inside-chroot-install/)，简单的方法是编辑`/etc/lvm/lvm.conf`这个文件，找到`use_lvmetad = 1`将`1`修改为`0`，保存，重新配置 grub。

如果报`grub-probe: error: cannot find a GRUB drive for /dev/sdb1, check your device.map`类似错误，并且`sdb1`这个地方是你的 u 盘，这是 u 盘`uefi`分区造成的错误，对我们的正常安装没有影响，可以不用理会这条错误。

---

**如果你是多系统，请注意上面一节中对`os-prober`这个包的安装。**

**强烈建议使用如下命令检查是否成功生成各系统的入口，如果没有正常生成会出现开机没有系统入口的情况：**

```bash
vim /boot/grub/grub.cfg
```

检查接近末尾的`menuentry`部分是否有`windows`或其他系统名入口。

如果你没有看到 Arch Linux 系统入口或者该文件不存在，请先检查/boot 目录是否正确部署 linux 内核：

```bash
ls /boot
```

查看是否有 initramfs-linux-fallback.img initramfs-linux.img intel-ucode.img vmlinuz-linux 这几个文件，如果都没有，说明 linux 内核没有被正确部署，很有可能是/boot 目录没有被正确挂载导致的，确认/boot 目录无误后，可以重新部署 linux 内核：

```bash
pacman -S linux
```

再重新生成配置文件，就可以找到系统入口。还要重新生成 fstab

退出系统，准备关机了：

```bash
exit
```

**如果挂载了`/mnt/boot`，先`umount /mnt/boot`，再`umount /mnt`，否则直接`umount /mnt`：**

```bash
umount /mnt/boot
umount /mnt
reboot
```

## 安装网络

```bash
systemctl enable --now NetworkManager
nmtui #用这个图像连接网络

#或者用下面的方法：
#nmcli device wifi list # 搜索网络，一下子没出来可以过几秒再试试
#nmcli device wifi connect SSID password password
# 上面的SSID和password分别换成自己家网络的wifi名和密码# 显然取一个不带中文的wifi名还是蛮重要的
```

## **声卡驱动**

```bash
sudo pacman -S alsa-utils pulseaudio-alsa pulseaudio
```

### **安装 Xorg**

`Xorg`是`Linux`下的一个著名的开源图形服务，我们的桌面环境需要`Xorg`的支持。

执行如下命令安装`Xorg`及相关组件：

```bash
sudo pacman -S xorg
```

## **安装桌面环境**

`Linux`下有很多著名的桌面环境如`Xfce`、`KDE(Plasma)`、`Gnome`、`Unity`、`Deepin`等等，它们的外观、操作、设计理念等各方面都有所不同， 在它们之间的比较与选择网上有很多的资料可以去查。

更多桌面环境的安装指南请见下面的链接：

[https://wiki.archlinux.org/index.php/Desktop_environment#List_of_desktop_environments](https://wiki.archlinux.org/index.php/Desktop_environment#List_of_desktop_environments)

### **安装 KDE(Plasma)**

直接安装软件包组（包含了很多软件包）即可

```bash
sudo pacman -S plasma kde-applications packagekit-qt5
```

packagekit-qt5 是为了让 Discover 工作正常

### **安装桌面管理器**

安装好了桌面环境包以后，我们需要安装一个图形化的桌面管理器来帮助我们登录并且选择我们使用的桌面环境，这里我推荐使用`sddm`。

---

### kde**安装 sddm**

```bash
sudo pacman -S sddm
systemctl enable sddm
```

---

### gnome 安装 gdm

---

同时你可能需要安装工具栏工具来显示网络设置图标（某些桌面环境已经装了，但是为了保险可以再装一下）：

```bash
sudo pacman -S network-manager-applet
```

## 双系统和 win10 时间不一样：

```bash
sudo timedatectl set-local-rtc true
```

## 安装字体

```bash

yay -S noto-fonts-cjk ttf-dejavu
```

# **GNOME Shell 扩展**

可以在 aur 中搜索  gnome-shell-extension-扩展名   安装，如：

```bash
**yay -S gnome-shell-extension-dash-to-dock**
```

或者到  [GNOME Shell Extensions](https://extensions.gnome.org/)  网站搜索下载，然后 gnome-tweak-tool 里安装启用。也可以使用浏览器安装，需要安装插件，请参考[Arch Wiki](<https://wiki.archlinux.org/index.php/GNOME_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E6%89%A9%E5%B1%95>)。

以下是我在使用的一些扩展：

-   User Themes 启用后可自定义 shell 主题
-   Place status indicator 显示文件管理器导航菜单
-   Removable drive menu  显示可移除设备（如 U 盘）拔插提示
-   Workspace indicator  在顶栏显示当前示工作区的序号
-   Top panel workspace scroll 在顶栏上滚动鼠标滚轮来快速切换工作区
-   Dash to Dock 可以将左侧的 dash 改为置于底部的 dock 栏
-   OpenWeather  在顶栏显示天气情况
-   Media player indicator 显示音乐播放器的状态
-   Battery status  显示电池电量百分比
-   Netspeed  在顶栏上显示网速

## 降级软件包

下载 downgrade

## 问题：

```
 [pulseaudio] bluez5-util.c: GetManagedObjects() failed: org.freedesktop.systemd1.NoSuchUnit: Unit dbus-org.bluez.service not found.

Solution:

# systemctl enable bluetooth.service
```

### guake 在 wayland 下问题解决：

在系统设置的设置快捷键下设置一个快捷键 `f12` ,命令是 `guake -t`

参考：

[2020 Archlinux 双系统安装教程（超详细）](https://zhuanlan.zhihu.com/p/138951848)

[Installation guide (简体中文)](<https://wiki.archlinux.org/index.php/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>)

[以官方 Wiki 的方式安装 ArchLinux](https://www.viseator.com/2017/05/17/arch_install/)

[安装 archlinux 系统【2020.03.15】](https://zhuanlan.zhihu.com/p/112541071)

图形界面安装：

[ArchLinux 安装后的必须配置与图形界面安装教程](https://www.viseator.com/2017/05/19/arch_setup/)

[](https://starrycat.me/archlinux-install-gnome-desktop.html#i)

小新 air14 睡眠问题：

[Lenovo IdeaPad 5 14are05](<https://wiki.archlinux.org/index.php/Lenovo_IdeaPad_5_14are05#Suspend_issues_(S3_sleep_fix)>)

