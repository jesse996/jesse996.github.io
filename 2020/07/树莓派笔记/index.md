# 树莓派笔记


{{< music auto="https://music.163.com/#/playlist?id=60198" >}}
{{< script >}}
console.log('Hello LoveIt!');
{{< /script >}}
**1.添加 wpa_supplicant.conf 文件**

在 boot 分区下添加 wpa_supplicant.conf 文件，文件内容写入

```
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
network={
    ssid="my_wifi"
    psk="12345678"
    key_mgmt=WPA-PSK
}
```

**2.开启 ssh**

在 boot 分区下新建 ssh 文件，无内容

**3.插卡，启动**

启动后树莓会自动链接 wifi，但是有个问题，不知道树莓的自动获取的 ip

方法 1：

如果网络中只有一个 pi，可以直接

ping raspberrypi

方法 2：

nmap -sn 192.168.1.0/24

方法 3：

打开路由，看看已连接设备，叫 raspberrypi 的就是

