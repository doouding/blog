---
title: Linux服务与自启动
date: 2017-05-24 16:10:43
tags:
---

## 后台程序
自动启动不免涉及到后台程序
### 后台执行与暂停
我们如果希望一个前台程序切换到后台暂停，那么可以使用快捷键`Ctrl + Z`来使当前执行的任务切换到后台暂停运行。
如果想让一个命令在后台执行，可以在后面添加一个`&`符号。这样子他就会在后台慢慢执行，执行完成后会有一个提示信息。例如

```
$ sslocal -c /etc/shadowsocks.json &
```

### 后台操作
切换到后台的任务可以通过`jobs`命令来查看。每个后台任务条目最前面有一个任务号，可以通过`fg %任务号`来使后台任务切换到前台。
我们刚刚说道`Ctrl + Z`是使程序到后台暂停。那么如果我想让它在后台执行需要怎么做呢？很简单，使用`bg %任务号`就能够使`Stopped`状态的后台程序变成`Running`状态

### 结束后台
使用`kill`命令可以关闭一个后台工作。`kill -signal %工作号`，示例如下
```
$ kill -9 %2
#关闭工作号2的工作，9表示强制结束一个工作
$ kill -2 %3
#2表示向任务输入Ctrl + c这个指令
$ kill -15 %4
#15表示以正常方式结束一个工作
#更多signal有关的选项可以使用man 7 signal查看
#kill后面不带百分号数字表示的是PID，带百分号数字表示是任务号
```

### 脱机管理
不管任务是在前台还是切换到了后台，只要你把终端关闭了，任务就会终止。所以我们需要`nohup`这个命令，它使得任务和终端脱离关系。使用方法如下。该命令所有的输出将会输出到`nohup.out`文件中
```
$ nohup sslocal -c /etc/shadowsocks.json &
```

## System V的服务管理
    注意：CentOS 7，Fedora 15等开始采用systemd来管理服务。抛弃了init管理的模式。
以下内容来自于鸟哥的Linux私房菜旧版：[初识系统服务](http://linux.vbird.org/linux_basic/0560daemons/0560daemons-centos5.php)，新版加入了`Systemd`的管理方式所以对原本的`init`内容有所省略。

System V采用`init`管理方式。系统第一个唤起的程序是`init`。然后这个`init`程序负责唤醒系统所需要的服务。
一般来说，所有的服务启动脚本通通都放在 `/etc/init.d/`底下。需要更改服务状态的时候也是通过这个方法。

- 启动：`/etc/init.d/daemon start`
- 关闭：`/etc/init.d/daemon stop`
- 重新启动：`/etc/init.d/daemon restart`
- 状态观察：`/etc/init.d/deamon status`

同时还有一个`service`脚本能够完成同样的功能，`service daemon start`..等等，就不用输入前面那一串路径了。
服务B有时候会依赖服务A才能启动，如果服务A没有启动，`init`是无法帮忙启动A的，需要管理员自己分析依赖，然后手动处理。

### daemon
系统为了某些功能需要提供一些服务（比如说计划任务）。这些服务实质也是由一些程序提供的，这些程序就称为`daemon`。一般可以把两个词同等看待。一般`daemon`程序后面有一个`d`，比如说`httpd`。
daemon有两个大类

- `stand_alone`：这类服务是单独启动的，不需要其他程序管理。并且启动后就一直常驻内存。这类服务的特点就是相应速度极快。
- `super daemon`：由特殊的`xinetd`或者`inetd`两个程序提供`socket`或者`port`对应的管理。当没有用户要求某socket或者port时，所需要的服务不会启动。若有用户要求时，`xinetd`会负责唤醒相应的服务来处理请求。服务在处理完请求以后就关闭，等到下次又有请求时又重新唤醒。所以这类服务的特点是：响应较慢、但不会长期占用系统资源。
`super daemon`又分为以下两种
   -  multi-threaded：可以并行处理多个请求。
   -  single-threaded：只能同时处理一个请求。

daemon工作方式有两种

 - `signal-control`：有用户端的请求过来时，就去处理。比如打印机服务
 - `interval-control`：每隔一段时间就主动去执行某项工作。比如计划任务

### inittab文件
`init`启动后会读取这个文件的内容。然后依次读取每条进行相应的处理。inittab文件定义了主要三种主要信息

- 系统的默认运行等级
- 如果他们终止了，哪些进程会启动，监视和重新启动
- 当进入一个新的运行等级后，执行什么操作

这个文件每行格式是
```
id:rstate :action:process
```

- id：是条目唯一标识符
- rstate：该条目匹配的运行等级
- action：定义如何运行这些命令
- process：定义要执行的命令


rstate有以下这些类型

类型|作用
:--:|:--:
respawn|进程被终止时便立即重启。init不等待处理结束便继续后续操作
wait|在系统进入到指定的运行级别时便启动相应进程。init等待处理结束才继续后续操作
once|在系统进入到指定运行级别时便启动相应进程，但只有第一次进入该级别时才启动一次
boot|只在系统启动时才运行指定进程。init不等待处理结束便继续后续操作
bootwait|只在系统启动时才运行指定进程。init等待处理结束才继续后续操作
powerfail|init接收到断电信号（SIGPWR）时才运行该进程，不等待处理结束便继续后续操作。
powerwait|init接收到断电信号（SIGPWR）时才运行该进程，等待处理结束才继续后续操作。
powerokwait|在电源restore时启动该进程。不太清楚restore在这里的具体含义。
powerfailnow|在电源快耗尽时启动该进程。
off|如果相应的进程正在运行，那么就发出一个告警信号，等待20秒后，再通过关闭信号强行终止该进程。如果相应的进程并不存在，那么就忽略该登记项。
ondemand|在系统进入相应运行级别时运行一次。
sysinit|在所有boot和bootwait记录前启动，一般仅用于对设备的初始化工作。init等待操作结束才继续执行。
initdefault|指定默认的运行级别。忽略process项。
ctrlaltdel|在init收到SIGINT信号（即Ctrl+Alt+Del被同时按下）时启动相应进程。
kbrequest|在init发现有组合键被按下时执行相应进程。
在上述参数中，sysinit、boot和bootwait的runlevel项被忽略。


### 服务的启动方式
`init`是第一个启动的，然后它会根据当前的运行等级来唤醒不同的服务。当Linux启动以后，它会去运行对应的`/etc/rc.d/rc[runlevel].d/`目录下存放的脚本。脚本的名称一般是`SXXdaemon` `S`为启动该服务，XX是数字表示启动的顺序，数字越低越先执行。当你使用`init runlevel`切换运行等级时，比如说从3转到5。init会自动分析`/etc/rc.d/rc[3].d/`和`/etc/rc.d/rc[5].d/`目录下的脚本，然后启动所需要的服务，就完成了转换。
#### 运行等级
`init`等级分级一般如下

等级|模式|描述
:--:|:--:|:--:
0|停止|就是关机状态啦
1|单人维护模式|仅允许root登录，不启动daemons，不配置网络接口
2|多用户模式|不配置网络接口，不启动deamons
3|带网络的多用户模式|就是正常启动的状态
4|未定义的|没有特别规定这个登记下的模式，可以自定义等级4的操作
5|图形界面模式|运行在模式3下并且提供图形界面
6|重启模式|就是重启啦

#### 服务的相关目录如下
路径|描述
:--:|:--:
/etc/init.d/*|启动脚本放置的位置，几乎所有服务启动脚本都放在这里
/etc/rc.d/init.d/*|和上面那个一样，是一个连接而已
/etc/rc.d/rc[0-6].d/*|存放各个运行等级下会启动的脚本，一般是连接到/etc/init.d/目录下的文件
/etc/sysconfig/*|各个服务的初始化环境设定文件
/etc/xinetd.conf，/etc/xinetd.d/*|super daemon本身的设定文件是/etc/xinetd.conf，其他daemon的设定放置在/etc/xinetd.d/*目录下详细可以看本章开头的网址
/var/run/*|为了方便管理，daemon会将自己的PID记录一份放置到/var/run/*目录中

### 添加开机自启动
`chkconfig`是个用于管理System V系统自启动的软件。所有发行版都带有这个工具。
```
$ chkconfig --list
# 显示服务的自启动状态，输出示例如下所示
acpid   0:off   1:off   2:off   3:on   4:on   5:on   6:off

$ chkconfig --list 3
# 显示运行等级为3时启用的服务

$ chkconfig --list httpd
# 显示httpd的自启动状态

$ chkconfig --level 345 acpid on
# 设置acpid在3 4 5 run level状态下自启动

$ chkconfig --add sslocal
# 添加一个服务名来让chkconfig管理，该服务名必须在/etc/init.d/目录内。添加以后就可以设置它的启动状态了。

$ chkconfig --del sslocal
# 有添加自然就有删除
```
如何设置启动顺序呢？这就需要我们手动修改脚本了，假设我们有一个ss脚本（要放置在/etc/init.d/目录下）
```bash
#!/bin/bash
# chkconfig:35 80 70
# description: Shadowsocks Service
sslocal -c /etc/shadowsocks.json
```
35表示在run level 3和5下执行，80表示启动时的顺序是80，70表示结束时的顺序是70。数字低的先被执行。
`chkconfig`其实就是把一些繁琐的操作简单化了，当你设置一个脚本在run level 3下执行时，`chkconfig`会在`/etc/rc.d/rc3.d`创建一个连接文件。当系统以run level 3启动时，就回去执行`/etc/rc.d/rc3.d`目录下面的文件

## Systemd管理服务
CentOS 7.x 以后，采用了`systemd`的管理方式，这个管理方式有以下优点：

- 以往的`init`是一个服务一个服务启动，所以后面的服务得慢慢等待。而`systemd`则可以同时启动所有服务，启动速度大大加快
- System V要使用`init`，`service`，`chkconfig`三个命令来来管理服务，而`systemd`只需要一个`systemctl`命令搭配`systemd`服务来管理。
- `systemd`可以自动解决依赖，当服务B需要服务A时，`systemd`会自动帮你启动服务A
- System V中把仅有两种服务分类`stand alone`和`super daemon`。`systemd`将每个服务都定义成一个`unit`，`unit`拥有很多分类，service，socket，target，path，snapshot，timer等等
- System V使用不同的`run level`来将不同情况下需要启动的服务分组，`systemd`将许多功能组成一个target来进行配置
- `systemd`是向下兼容init的

缺点是：

- `systemd`采用`systemctl`命令管理，但是`systemctl`这个命令语法有所限制，所以不能使用自定义参数，但是`/etc/init.d/daemon`就是一个纯shell脚本，灵活度相当高。
- 如果服务是用户直接手动输入命令启动的，比如`mysqld`，那么`systemd`是没法管理这个服务的，只有通过`systemctl`启动的服务才能受到`systemd`的管理
- `systemd`启动过程中无法通过标准输入来互动

### systemd 的相关目录
目录|作用
:--:|:--:
/usr/lib/systemd/system|每个服务最主要的启动脚本，类似于以前的/etc/init.d目录
/run/systemd/system/|系统执行过程中所产生的服务脚本，这个优先级比上面的目录高
/etc/systemd/system/|管理员依据系统需求所产生的脚本，这个脚本的优先级又比上一个高
/etc/sysconfig/|几乎所有服务都会将初始化的一些选项设定写入到这个目录下。比如说网络服务就保存在`/etc/sysconfig/network-scripts/`
/run/|运行时暂存文件
/var/lib/|一些会产生数据的服务都会写到/var/lib目录中。比如mysql数据保存在`/var/lib/mysql/`
看系统开机会不会执行某个服务，就是到`/etc/systemd/system/`目录下瞧一瞧就行。`/usr/lib/systemd/system`只是实际存放启动脚本的位置，`/etc/systemd/system/`的文件都是连接到`/usr/lib/systemd/system`目录下的

### systemd的unit类型分类
扩展名|功能
:--:|:--:
.service|比较经常被用到的服务大多是这个类型的。最常见的类型。
.socket|插槽服务，主要是inter-process communication的socket file功能。这个类型的服务一般是比较不会被用到的服务。
.target|就是一堆.service和.socket的集合
.mount/.automount|和文件系统挂载相关的服务
.path|某些需要检测特定目录的服务，比如说打印服务
.timer|循环执行的任务，比如说计划任务之类的

### 通过systemctl管理服务

```
# status 用来查看服务的状态
$ systemctl status mysqld.service
atd.service - Job spooling tools
    Loaded: loaded (/usr/lib/systemd/system/atd.service; enabled)
    Active: active (running) since Mon 2015-08-10 19:17:09 CST; 5h 42min ago

# Loaded这行如果为enabled，enable为开机启动，disable为开机不会启动
# 除了上面两个还有
# static：这个服务不会自己启动，但可能会被其他服务唤醒（依赖）
# mask：这个服务无论如何也不会启动，也无法被其他服务唤醒

# Active折行表示服务当前状态是正在运行（running）还是没有执行（dead）
# Active除了上面的还有
# active(exited)表示运行一次就结束，不会常驻内存。
# active(waiting)表示正在运行，但是正在等待其他事件才能处理。比如打印机要等待作业进来才会工作
# inactive：这个服务没有运行的意思
```
```
# stop 用来正常关闭服务
$ systemctl stop mysqld.service

# start 用来启动服务
$ systemctl start mysqld.service

# disable 关闭开机自启
$ systemctl disable mysqld.service

# enable 开启开机自启
$ systemctl disable mysqld.service

# mask 让服务无论如何也不会被执行；unmask 用来解除
$ systemctl mask mysqld.service

# list-unit 查看目前启动的unit，加上--all才会显示没启动的
$ systemctl list-units 
# --type=可以显示特定类型
$ systemctl list-units --type=target --all

# list-unit-files 显示所有已安装的unit
$ systemctl list-units-files
UNIT FILE           STATE
dev-mqueue.mount    static
dev-hugepages.mount static
# STATE 就是 enable/disabled/mask/static 这些
```
### 管理target
有下面几个主要的 target

target|场景
:--:|:--:
graphical.target|纯文本模式加上界面
multi-user.target|纯文本模式
rescue.target|在无法使用root登入的情况下，systemd在开机时会多加一个额外的暂时系统，与你原本的系统无关。这时你可以取得root的权限来维护你的系统
emergency.target|当无法使用rescue.target时，可以尝试使用这种模式
shutdown.target|关机流程
getty.target|设定你需要几个tty之类，如果要降低tty的项目，可以修改这个东西的配置文件

切换target使用`systemctl`命令。
```
# 获取当前的默认target
$ systemctl get-default

# 设置默认target
$ systemctl set-default multi-user.target

# 不重启的情况下将环境改为纯文本模式
$ systemctl isolate multi-user.target
# 除了使用isolate命令切换，还可以使用便捷命令
$ systemctl poweroff #关机
$ systemctl reboot #重新启动
$ systemctl suspend #睡眠模式，系统状态保存到内存里面
$ systemctl hibernate #休眠模式，系统状态保存到硬盘里面
$ systemctl rescue #强制进入救援模式
$ systemctl emergency #强制进入紧急救援模式
```

### 分析依赖
```
# 列出unit的依赖
$ systemctl list-dependencies mysqld

# 列出哪个服务依赖 mysqld
$ systemctl list-dependencies mysqld --reverse

# 显示当前当前target的依赖
$ systemctl list-dependencies
```