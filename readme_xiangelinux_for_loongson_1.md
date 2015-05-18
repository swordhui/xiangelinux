### 弦歌Linux 1.0 龙芯N32 发行说明 ###

|文档版本	|V3|
|:------------|:-|
|修订日期	|10年11月8日|

修订记录
|版本 |修订人	|日期	|备注|
|:------|:---------|:------|:-----|
|v1     |王江伟 |	10年11月2日|	初始版本|
|v2	    |张力辉	|10年11月4日	|加入安装说明, gpkg使用说明, 脚本写作等标题, 重新排版|
|V3	    |王江伟	|10年11月8日	|编写安装说明、修改部分内容、移除gpkg使用说明, 脚本写作标题|

### 1. 简介 ###
弦歌Linux for 龙芯版是一个专门针对龙芯2F cpu进行优化的N32系统，是基于源代码的Linux发行版，使用由弦歌Linux项目组自行开发的包管理系统(gpkg)进行包管理。具有轻量级，易扩充，支持手动/自动安装，二进制软件包安装等特点，非常合适龙芯2F的NAS与防火墙等设备使用。
  * 弦歌Linux for 龙芯版采用滚动升级，通过提供更新的软件包安装脚本来更新系统。
  * 此版本主要针对服务器进行优化。并提供大量的服务器软件。
  * 此版本也提供完整的桌面平台以及一切常用工具，但是这些未经测试（所有的桌面软件在只X86平台测试通过）

### 2. 主要特性 ###

### 2.1 特性介绍 ###

  * 稳定、高效的Linux核心
  * 基于Linux 2.6.36-rc8+核心(2F盒子、2F笔记本)
  * 基于Linux 2.6.30核心(2F NAS)
  * 针对性龙芯2F CPU处理进行专门优化
  * 支持全部的网络防火墙模块以及各种网络模块
  * 增强的管道缓冲功能
  * IPv4 fragmentation offload及缓冲管理
  * 实现系统内核性能优化、系统线程性能优化、网络性能优化等
### 全面的安全 ###
  * 提供ext3、ext4、reiserfs等分区的ACL（访问控制列表）以及EA（扩展属性）支持
  * 提供内核支持的全部防火墙模块
  * 提供安全增强模块，可实现用户账号与强口令机制、强制存取控制功能MAC等功能
  * IPSEC增强功能提高了安全性和性能
### 强大、全面的服务器软件 ###
  * Samba功能增强，并增加了winbind功能。实现与Microsoft Ad服务集成，实现统一认证支持
  * 网页服务器由lighttpd提供。同时还默认安装了全部的模块
  * 提供lighttpd、postegsql、portftpd、ssh、rsyncd、iptable、nfs、ntp等等服务器软件
  * 所有服务器软件都使用龙芯平台特性有指令进行优化。
### 易用性增强 ###
  * 基于弦歌特有的包管理(gpkg)系统，可以方便的升级系统软件
  * 使用简单的简单明了的安装脚本，普通用户即可实现软件的编译定制，优化或是增加新的软件包
  * 支持二进制软件包进行安装。用户可以省去大量的编译时间。
### 高效完整的开发环境 ###
  * 提供C、Python、Perl、PHP、Shell等编程语言开发环境支持
  * 提供完整的可运行在龙芯2F平台上以chroot方式运行的完整N32工具链（toolchain）

### 2.2 在龙芯2F平台上实现的第一 ###
  * 1、第一个原生支持龙芯2F的操作系统。之前基于龙芯2F的系统全部为移植，而非专为龙芯制作
  * 2、第一个在龙芯平台的发行版本中使用eglibc库，之前所有有龙芯平台上运行的Linux发版都是使用的glibc作为默认的库。而eglibc库针对嵌入式系统有优化，性能要高出不少，而且更精巧。
  * 3、第一个对整个系统使用龙芯2F CPU特有指令如：-march=loongson2f、-Xassembler -mfix-loongson2f-nop等指令进行优化的系统。
  * 4、第一个在整个龙芯平台上使用-mplt编译，有了plt的支持，在不需要使用PIC(Position Independent Code)的地方，就不必使用PIC（之前有相当一部分不需要使用PIC的情况会使用PIC），这样可以减少指令数量，减小可执行程序体积，直接和间接的加快软件执行速度。
  * 5、第一个提供全套内核防火墙以及网络模块的龙芯发行版
  * 6、第一个为龙芯2F N32平台打造的服务器发行版本。
### 2.3 其它亮点 ###
  * 1、整个平台使用-mabi=n32编译，其性能比O32高出20％～30％。
  * 2、使用轻量级的包管理系统gpkg，用户可以方便安装、删除各种软件。使用gpkg用户可以使用源代码安装软件也可以使用已经编译好的二进制软件包来安装，当使用源代码安装时，用户还可以自定义优化参数。
  * 3、基于CLFS的启动脚本，更快的启动速度，从选择内核到出现登录提示，全程只需要12秒。非常合适NAS以及防火墙产品。
  * 4、强大的自动补全系统，原有的tab键补全功能现在得到了全面的强化。它现在能自动匹配你所输入的命令，例如：当前目录下有aaa.tar文件和aaa目录，当你输入tar xf  后按下tab键，会自匹配aaa.tar文件，而不是补全aaa后让用户再次输入。新的补全命令可不止这些，它可以自动补全1000多种模式。
  * 5、新的系统精简了语言支持文件，除了C、POSIX、en\_US外还支持完整的中文系统，utf8、gb18030、gb2312、gbk等等全都支持，同样新系统还带有台湾、新加坡、香港、日本的语言支持，除此之外不支持其它语言。
  * 6、全新的日志系统，新版本中使用了rsyslog来做为日志系统，现在只需要一个进程，就可以记录内核与系统的日志，由于它对Sysklogd的兼容，用户可以方便的从Sysklogd转移到rsyslog上来。
  * 7、新的软件、所有的软件都是目前最新的稳定版本。其中eglibc是由2010年9月18日的CVS库中检出的。binutils版本为2.20.51(已加入龙芯补丁支持)
  * 8、vim支持中文输入法，在弦歌for龙芯版中，我们集成了vim的输入法，目前支持极品五笔与搜狗拼音，进入vim后使用Ctrl+\即可以打开和切换输入法。


---


### 3. 安装方法 ###
3.1 准备空白分区
  * 使用fdisk或者cfdisk创建分区规划。至少需要一个交换分区（类别为82）和一个Linux分区（类别为83）。下面是本手册选用的方案，创建包括一个主分区，一个交换分区。请将/dev/hda替换为你自己的磁盘，例如hda等。
  * # fdisk –l /dev/hda

  * Disk /dev/hda: 2096 MB, 2096898048 bytes
  * 16 heads, 63 sectors/track, 4063 cylinders, total 4095504 sectors
  * Units = sectors of 1  512 = 512 bytes
  * Sector size (logical/physical): 512 bytes / 512 bytes
  * I/O size (minimum/optimal): 512 bytes / 512 bytes
  * Disk identifier: 0x4139fc7a

  * Device Boot      Start         End      Blocks   Id  System
  * /dev/hda1              63     3515903     1757920+  83  Linux
  * /dev/hda2         3515904     4095503      289800   82  Linux swap / Solaris
3.2 格式化分区
  * 弦歌Linux龙芯2F内核支持ext3、ext4、Reiserfs、3种文件系统。
  * （主分区使用ext3）
  * # mke2fs -j /dev/hda1

  * （创建并激活交换分区）
  * # mkswap /dev/hda2 && swapon /dev/hda2

  * 另：JFS、XFS文件系统为模块支持如果将分区格式化为JFS或XFS则需将系统放至ext3、ext4、Reiserfs、3种文件系统上。否则将出现无法找到分区的错误。
3.3 挂载分区
  * 挂载文件系统到目录下。
  * （建立挂载目录）
  * # mkdir /mnt/xiange
  * （挂载主分区到/mnt/xiange）
  * # mount /dev/hda1 /mnt/xiange
  * （进入到/mnt/xiange）
  * # cd /mnt/xiange
3.4 下载stage1
  * 首先确保正确设置了日期和时间。推荐使用CST时间（CST China Standard Time UTC+8:00 中国沿海时间即北京时间）。
  * （查看时钟）
  * # date
  * 2010年11月08日 星期一 15:04:01 CST

  * （设置当前日期和时间，如果需要的话）
  * # date –s 20101108 && date –s 15:04:50
  * 2010年11月08日 星期一 15:04:50 CST

  * 下载stage1
  * （使用wget命令下载stage1）
  * # wget http://xiangelinux.googlecode.com/files/stage1-loongson2f-n32-1.0.tar.lzma
3.5 解压
  * （进入/mnt/xiange，解开stage1压缩包）
  * # lzma -d stage1-loongson2f-n32-1.0.tar.lzma
  * # tar xf stage1-loongson2f-n32-1.0.tar
3.6 chroot
  * # mount -t proc proc /mnt/xiange/proc
  * # mount -o bind /dev /mnt/xiange/dev
  * # cp -L /etc/resolv.conf /mnt/xiange/etc/
  * # chroot /mnt/xiange /bin/bash
3.7 修改配置文件
3.7.1修改root密码
  * # passwd root
  * New UNIX password: 设置root的密码
  * Retype new UNIX password: 再次输入root的密码
  * passwd: password updated successfully

3.7.2设定主机名和域名
  * # cd /etc
  * # echo "127.0.0.1 mybox.at.myplace mybox localhost" > hosts

（设置本主机名为mybox）
  * # cat HOSTNAME="mybox">  /etc/conf.d/hostname

3.7.3 编辑/etc/fstab
  * 用实际的分区名代替ROOT和SWAP。记得确认一下文件系统是否与所安装的相匹配。
  * # vi -w /etc/fstab
  * /dev/hda1       /               ext3            defaults        0       0
  * /dev/hda2       swap            swap            pri=1           0       0

3.7.4 更新软件仓库
  * # gpkg --sync

3.7.5 配置网络
  * 固定IP请修改/etc/sysconfig/network-devices/ifconfig.eth0/ipv4文件(如果文件不存在，请自行建立)，内容如下：
  * ONBOOT=yes
  * SERVICE=ipv4-static
  * IP=192.168.1.51
  * GATEWAY=192.168.1.2
  * PREFIX=24
  * BROADCAST=192.168.1.255

  * 使用dhcp请修改/etc/sysconfig/network-devices/ifconfig.eth0/dhcpcd文件(如果文件不存在，请自行建立)，内容如下：
  * ONBOOT="yes"
  * SERVICE="dhcpcd"
  * DHCP\_START="-q"
  * DHCP\_STOP="-k"
  * PRINTIP="yes"

  * 另：对于龙芯2F盒子以及笔记本用户，请删除/etc/sysconfig/network-devices/ifconfig.eth1文件夹。此文件夹是为有2个网卡的龙芯NAS用户准备的。
  * # rm -rf /etc/sysconfig/network-devices/ifconfig.eth1
3.8 重新启动
  * # reboot