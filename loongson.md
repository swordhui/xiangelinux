此版本将命名为xiange-N32-loongson-0.2版本。

此版本修正了0.1版本的一些启动问题。

此版本仍是测试版本。

此版本清除了man、info、doc文档，原因是受google上传文件大小限制。

BUG汇报请至jonsk.echo@gmail.com。

下载地址：
http://xiangelinux.googlecode.com/files/xiange-N32-loongson-0.2.tar.bz2

系统安装方法：
将下载的xiange-N32-loongson-0.2.tar.bz2解压至空的分区下。
再设置/boot/boot.cfg文件，使用刚才解压的分区为新的根分区,内核可以使用你的宿主系统的内核。
xiange-N32-loongson-0.2系统带有两个内核，一个是debian的，另一个由fc13提供，如果宿主系统内核无法启动请使用弦歌系统自带内核（推荐使用debian的内核）

软件安装方法参见：
http://code.google.com/p/xiangelinux/wiki/xgpkg


目前已知道的问题：

1、由于我的开发机是NAS，有双网卡，而龙芯盒子只有一个网卡，所以启动时会出现eth1使用dhcp出错的信息,这可能不算是BUG。

2、编译一些程序时出现下列错误：

cc -D\_GNU\_SOURCE -I proc -I/usr/include/ncurses -fno-common -ffast-math -W -Wall -Wshadow -Wcast-align -Wredundant-decls -Wbad-function-cast -Wcast-qual -Wwrite-strings -Waggregate-return -Wstrict-prototypes -Wmissing-prototypes -O2 -s  -Wdeclaration-after-statement -Wpadded -Wstrict-aliasing -fweb -frename-registers -fomit-frame-pointer -fno-inline-functions -c -o pwdx.o pwdx.c

pwdx.c: In function 'main':

pwdx.c:38:15: error: 'PATH\_MAX' undeclared (first use in this function)

pwdx.c:38:15: note: each undeclared identifier is reported only once for each function it appears in

pwdx.c:38:11: warning: unused variable 'buf'

make: **[pwdx.o] Error 1**

此问题可能是由于eglibc和ncurses的编译引起的，但目前这两个包能正常工作。


已解决的问题：

1、vim-7.3在使用libncurses.so.5库时出现无法启动，并提示文件太短。加入ncurses的兼容库支持后vim可以正常使用。

2、/etc/rc.d/init.d/i18n脚本调用了setfont、dumpkeys、loadkeys、kdb\_mode这四个程序，这四个程序可能是由Kbd包提供的，但是考虑到中国用户都是标准的键盘所以没有安装Kbd，因此在启动时出现没有找到程序的错误信息。

3、IPRoute2-2.6.34包没有安装，网络脚本在启动时没有找到ip命令，因此启动网络出错。

4、目前在setclock启动脚本启动时，第16行有问题，它找不到/etc/sysconfig/clock文件。

5、baselayout-loongson64，建立系统目录的脚本需要调整/lib32、/usr/lib32为连接，而不是目录。


未知的问题：

1、系统启动时似乎在fsck分区时出错，目前还不太清楚是程序引起的问题，还是分区本身的问题。

下一版本的介绍：

1、在下一个版本中将开始使用binutils\_2.20.51.20100813作为默认的binutils版本，此版本加入了龙芯的支持。可以不用打上补丁了。

2、除了原有优化参数外，将会使用gcc -Xassemble "-mfix-loongson2f-nop"等参数来进行优化。

3、将会修正一系列的包问题，例如baselayout-loongson64的脚本问题等。

4、系统守护进程脚本将会使用LFS的脚本来进行控制。

5、针对开机速度会进一步进行优化。

6、内核及内核头文件将会使用git://dev.lemote.com/linux-loongson-community，即龙芯公司提供。

7、自动补全脚本将会进行修改。

8、在测试版本中出现的部分软件包将会补替换或是丢弃。




新系统的功能：

1、使用eglibc库，eglibc库针对嵌入式系统有优化，预计会比glibc的性能要高出不少，更精巧。

2、整个平台使用-march=loongson2f编译，针对龙芯2F平台优化。

3、整个平台使用-mabi=n32编译，其性能比O32高出20％～30％。

4、整个平台使用-mplt编译，有了plt的支持，在不需要使用PIC(Position Independent Code)的地方，就不必使用PIC（之前有相当一部分不需要使用PIC的情况会使用PIC），这样可以减少指令数量，减小可执行程序体积，直接和间接的加快软件执行速度。

5、使用轻量级的包管理系统gpkg，用户可以方便安装、删除各种软件。使用gpkg用户可以使用源代码安装软件也可以使用已经编译好的二进制软件包来安装，当使用源代码安装时，用户还可以自定义优化参数。

6、基于CLFS的启动脚本，更快的启动速度，从选择内核到出现登录提示，全程只需要14秒。未来解决启动脚本问题后速度估计可以接近到12秒。

7、强大的自动补全系统，原有的tab键补全功能现在得到了全面的强化。它现在能自动匹配你所输入的命令，例如：当前目录下有aaa.tar文件和aaa目录，当你输入tar x 后按下tab键，会自匹配aaa.tar文件，而不是补全aaa后让用户再次输入。新的补全命令可不止这些，它可以自动补全1000多种模式。

8、新的系统精简了语言支持文件，除了C、POSIX、en\_US外还支持完整的中文系统，utf8、gb18030、gb2312、gbk等等全都支持，同样新系统还带有台湾、新加坡、香港、日本的语言支持，除此之外不支持其它语言。

9、全新的日志系统，新版本中使用了rsyslog来做为日志系统，现在只需要一个进程，就可以记录内核与系统的日志，由于它对Sysklogd的兼容，用户可以方便的从Sysklogd转移到rsyslog上来。

10、新的软件、所有的软件都是目前最新的稳定版本。其中eglibc是由2010年7月29日的CVS库中检出的。