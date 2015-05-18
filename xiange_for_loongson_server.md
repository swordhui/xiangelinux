弦歌Linux for 龙芯2F N32平台 服务器生产版

本来想罗嗦一下这个版本的特点，不过想想还是算了，估计用这个版本的人都是需要用它来完成一定的任务的人，对哪些特点什么的估计都不太有兴趣，所以还是直接说具体的东西吧，哎，还是罗嗦半天。

此版本是在弦歌Linux for 龙芯2F 1.0版本的基础上集成了服务器软件：

所有的软件还是和以前一样使用了龙芯的专用指令优化过了。

软件的支持：

php使用的是最新的5.3.6版本，在编译时加入了jpg、libxml2、mcrypt、png、gd等软件的支持，同时还启用了mysqli，估计能提升20%～30%的mysql查询速度。

mysql语言支持内置gbk,gb2312,binary,utf8,big5,ascii，不过没有加入ssl支持。用户需要自己建立数据库文件，并修改密码。

samba禁用了swat功能，大家自己修改配置就行了。

postgresql由于使用了mysql，所以postgresql没有提供，有需要的可以使用gpkg -i postgresql自己编译。

lighttpd支持全部的模块。支持以gci或是fcgi方式运行php，支持mysql、ldap、openssl、webdav等功能，同时禁止了ipv6的支持。

apache，默认提供lighttpd做为默认的web服务器，有需要apache的可以自己编译gpkg -i apache

postfix提供mysql的支持，有需要更多支持的请自己修改编译脚本自行编译。


使用方法：

1、硬盘需求:解压后需占用900MB，推荐2G以上的分区。

2、下载程序：http://code.google.com/p/xiangelinux/downloads/list 上下载:
xiange-loongson-2f-n32-server-production-1.0.tar.lzma.path1
xiange-loongson-2f-n32-server-production-1.0.tar.lzma.path2文件。


3、程序使用split进行分割的所以要合并文件
> cat xiange-loongson-2f-n32-server-production-1.0.tar.lzma.path1 xiange-loongson-2f-n32-server-production-1.0.tar.lzma.path2  > xiange-loongson-2f-n32-server-production-1.0.tar.lzma

4、解压到合适的位置，然后参照弦歌Linux安装手册安装就行了。
安装手册位置：http://code.google.com/p/xiangelinux/wiki/readme_xiangelinux_for_loongson_1

磁盘管理工具
mdadm-3.0
lvm-2.2.02.71

网络服务
tcp-wrappers-7.6-ipv6
nfs-utils-1.2.3
portmap-6.0
openldap
mysql-5.1.56
fcgi-2.4.0
php-5.3.6
sqlite-autoconf-3070500
lighttpd-1.4.28
samba-3.5.8
bind
dhcp
iptables
ntp
proftpd
rsync
xinetd
postfix
openldap
openssh

网络工具
ethtool-2.6.38
ncftp

其它库及工具
freetype-2.3.12
db
gd-2.0.36
libjpeg-8
libpng-1.2.43
pcre-8.10
libmcrypt
mhash
mcrypt
libxml2