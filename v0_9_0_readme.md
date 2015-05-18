#版本v0.9.0说明文件

# 0.简介 #

弦歌Linux 0.9.0的使用说明和历史记录.


# 1.使用说明 #

> 弦歌Linux 0.9.0是纯Bash写的自动下载/编译/安装程序，能自动构建工具链/根系统，并自动编译包括Xorg在内的400个左右软件包。<br>
<blockquote>这个版本没有包管理, 具体的项目计划请参见<a href='RoadMap.md'>RoadMap</a></blockquote>

<h1>2.使用方法</h1>

<ul><li>2.1 格式化一个空闲分区，挂接到工作目录，如/mnt/xiange.<br>
</li><li>2.2 解压0.9.0到工作目录<br>
</li><li>2.3 进入工作目录的scripts子目录(解压后得到)<br>
</li><li>2.4 运行./download 下载所需的软件包和补丁. 源码下载到工作目录的source子目录。<br>
<ol><li>如果有软件包下载失败，请手动下载到工作目录的source目录，再次运行./download会自动跳过。<br>
</li><li>也可以直接修改包对应脚本文件的URL地址。<br>
</li></ol></li><li>2.5 下载完毕后，以root身份，运行 ./build<br></li></ul>

<ul><li>2.6 漫长的编译过程，体验刷屏乐趣。<br>
<ol><li>已安装的软件列表在/var/lfs/installed.log,记录了软件包名字和安装时间。<br>
</li><li>编译目录在工作目录的tools/temp子目录。使用tmpfs会加快编译速度，减少磁盘损耗。tmpfs至少要3GB.<br>
</li><li>mkdir /mnt/xiange/tools/temp; mkdir /dev/shm/xgtmp; chmod 1777 /dev/shm/xgtmp; mount --bind /dev/shm/xgtmp /mnt/xiange/tools/temp;</li></ol></li></ul>

<ul><li>2.7 用chroot转到工作目录, 设置密码(passwd), 编辑/etc/fstab<br>
</li><li>2.8 给自己的机器编译安装内核.<br>
</li><li>2.9 安装GRUB 或 GRUB2.<br>
</li><li>2.10 重启.. good luck！<br>
</li><li>2.11 如果启动正常，转到根目录的scripts目录下，执行./blfs, 编译剩下的软件包，包括xorg等</li></ul>



<blockquote>please mail to  sword<swordhui@hotmail.com> if you have any question or suggestion。</blockquote>

<h1>2.历史记录</h1>

<h3>v0.1 2009-07-16</h3>

<ul><li>irst verson.</li></ul>

<h3>v0.2 2009-07-21</h3>

<ul><li>add README and HISTORY. <br>
</li><li>add blfs script, build blfs packages after system boot ok. <br>
</li><li>fix bug: create source directory when running download. <br>
</li><li>resources/eth0: set IP/router more easy. <br>
</li><li>add setclock script, to set system time. <br>
</li><li>fix bug: git compile error. <br>
</li><li>fix bug: samba compile error.<br>
</li><li>fix bug: not create cc symbol link after gcc compile, then blfs packages compile failed.<br>
</li><li>fix bug: e2fsprogs does not install to system. no e2fsck <br>
</li><li>fix bug: no /etc/profile and /etc/DIRCOLORS copied, so no PS1 set when bash running.<br>
</li><li>fix bug: mpfr error when compile gcc pass 1. <br></li></ul>

<h3>v0.3</h3>

<ul><li>add ckupgrade scripte to find the newest packages. <br>
</li><li>remove useless files: scripts_blfs/build<br>
</li><li>add pkg-config, which, dcron, pciutils, libusb, usbutils, httpd, wirelessutils, dhcpcd, ctags<br>
</li><li>check after create man,doc,info symbol link. (buggy?) <br></li></ul>

<h3>v0.4</h3>

<ul><li>add tcp_wrapper, portmap, expat, gdb, and cgdb <br></li></ul>

<h3>v0.5.0</h3>

<ul><li>clean compile directory after package compile OK. (buggy?) <br>
</li><li>add bochs, python, libxml2, libxslt, freetype, fontconfig, Xproto and Xlib. <br></li></ul>

<h3>v0.6 2009-08-16</h3>

<ul><li>finished Xorg 7.4 installation. <br>
</li><li>add a tool script geturl, for X package checking. <br>
</li><li>fixed gdb compile bug. <br>
</li><li>remove bochs due to compile bug.<br>
</li><li>add gpm, libpng, jpeg-6b, giflib, directfb, gperf, XML::parser, intltool, xterm, links <br>
</li><li>fixed bug: doc/man/info symbol check.<br>
</li><li>fixed bug: rc file. <br></li></ul>

<h3>v0.7 2009-08-17</h3>

<ul><li>fixed compile bugs when building Xorg 7.4. <br>
</li><li>added Xorg auto configuration <br></li></ul>

<h3>v0.8 2009-08-17</h3>

<ul><li>add unzip, libtiff, cairo, glib1/2, pango, ATK, GTK+1/2, share-mine-info, jasper <br></li></ul>

<h3>v0.9 2009-08-17</h3>

<ul><li>fixed jasper patch bug. <br>
</li><li>Add Xorg video driver to script: nv, i740, <a href='https://code.google.com/p/xiangelinux/source/detail?r=128'>r128</a> <br>