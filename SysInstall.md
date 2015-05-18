# 0.简介 #

本文介绍了安装弦歌Linux的步骤。

# 1.安装方式 #

## 1.1 软件包组成 ##
弦歌Linux由4大类软件包组成，分别称为Stage0, Stage1, Stage2, Stage3.<br>
<table><thead><th>名称</th><th>简介</th></thead><tbody>
<tr><td>Stage0</td><td>目地和哺乳动物胎盘类似: 一个能chroot进去的软件环境并能编译安装其他软件包。将弦歌Linux移植到到新平台只需先交叉编译Stage0即可。</td></tr>
<tr><td>Stage1</td><td>包括让系统启动的命令行工具, 无图形界面，可以构建各种服务器</td></tr>
<tr><td>Stage2</td><td>包括图形界面，媒体格式库，Desktop等，可以构建日常生活用桌面系统。本次发布的<a href='xiange_x86_64_LiveUSB_4G_install_guide.md'>Xiange-x86-64-LiveUSB-4G-201404</a> 就由Stage0, stage1, stage2软件包组成，正好放入一个4G U盘。</td></tr>
<tr><td>Stage3</td><td>其他不能收录进Stage2的大型软件包， Xiange-x86_64-LiveUSB-8G会包含Stage3软件包，如google earth, libreoffice等。</td></tr></tbody></table>

<h2>1.2 安装方式: LiveUSB</h2>
弦歌Linux支持2种安装方式: LiveUSB方式和源码编译方式。LiveUSB方式需要下载弦歌Linux发布的LiveUSB包，解压后用dd工具直接写到U盘上。具体可以参考<a href='xiange_x86_64_LiveUSB_4G_install_guide.md'>LiveUSB安装指南</a>, 里面也有将LiveUSB安装到硬盘的步骤。<br>
<br>
<h2>1.3 安装方式: 源码编译方式</h2>
源码编译方式需要使用源码包管理器gpkg下载编译并安装指定的软件包。细分可分为两种: 手动安装和自动脚本安装。gpkg的文档请参见<a href='xgpkg.md'>xgpkg</a>.<br>

<h2>1.4.手动安装</h2>
手动安装即通过手动的方式安装所需软件包，如:<br>
<pre><code>gpkg -i linux-header <br>
</code></pre>

安装Linux头文件, gpkg -i glibc 安装C库， gpkg -i bin-utils安装二进制工具链等。可以用这种方式安装日常所需但没收录到发行版的软件包， 可以用这种方式安装整个系统，步骤和LFS手册类似，只是细节交给gpkg处理。<br>

用户只需输入简单的gpkg -i 软件包命令，gpkg在后台自动做如下工作:<br>
<br>
<ul><li>从/etc/xgparas文件读取编译优化参数。没有此文件则使用默认优化参数。<br>
</li><li>找到制定软件包的编译脚本。<br>
</li><li>读取脚本内容，查看本软件包依赖哪些软件包。<br>
</li><li>安装依赖的软件包以满足依赖关系。<br>
</li><li>检查/var/xiange/source目录下有没有编译所需的源码，如果没有，自动下载。<br>
</li><li>下载支持断点续传。<br>
</li><li>解压下载的文件，并自动打好补丁(如果有的话)<br>
</li><li>自动按用户制定的编译参数和选项配置软件包，并记录配置信息到文件以备后用。<br>
</li><li>编译软件包<br>
</li><li>如果软件包有自动测试，执行自动测试并记录结果到文件。<br>
</li><li>将软件包安装到临时目录，并自动去除调试信息，压缩文档，去掉多余的本地化文件等瘦身。<br>
</li><li>将临时目录打包， 采集文件指纹信息以备后用。<br>
</li><li>安装编译好的二进制包到系统。安装完成后二进制包并不删除，留在/var/xiange/packages目录以备后用。<br>
</li><li>安装过程支持滚动升级，即使更新正在运行的C库也没问题。<br>
</li><li>最后执行脚本指定的程序配置系统以完成安装。</li></ul>

<h2>1.5.自动脚本安装</h2>
gpkg内置自动重构stage0, stage1, stage2, stage3的脚本，如执行<br>
<pre><code>gpkg stage0<br>
</code></pre>
可以重新编译安装所有stage0软件包。在这之前你可以修改编译参数以做更好的优化。<br>
<br>
<h2>1.6关于交叉编译</h2>
默认编译安装的目标是/目录。如果设定了XGROOT环境变量，如:<br>
<pre><code>export XGROOT=/mnt/image<br>
</code></pre>

则根目录会变为/mnt/image, 适合交叉编译。编译参数会从$XGROOT/etc/xgparas读取。<br>
<br>
<h1>完</h1>
<h1>由于LiveUSB构建完毕，后面文档已基本无用。</h1>



<h3>3.1 准备主分区</h3>

<ul><li>首先需要一个主分区，至少5G以上。可以用fdisk/cfdisk从硬盘空闲空间生成，也可以使用现有无用分区。本文以/dev/sda4分区为例。<br>
</li><li>格式化主分区, 推荐使用ext4. 命令如下（需要root身份）:<br>
<pre><code>mkfs.ext4 /dev/sda4<br>
</code></pre>
</li><li>挂载到工作目录。本文以/mnt/xg为例，命令如下:<br>
<pre><code>mount /dev/sda4 /mnt/xg<br>
</code></pre></li></ul>


<h3>3.2 准备交换分区</h3>
<ul><li>弦歌系统基于源码安装，有大量的编译工作，如果有多于1GB的内存, 且启动tmpfs, 编译工作在tmpfs(可以理解为内存)中完成，可以大大提高编译速度，延长硬盘使用寿命。<br>
</li><li>先检查系统是否已经启动tmpfs, 使用df -t tmpfs命令, 如能看到如下信息，说明tmpfs 已经启动。<br>
<pre><code>sword@XGSvr /mnt/xg $df -t tmpfs -h<br>
文件系统	      容量  已用  可用 已用%% 挂载点<br>
tmpfs                  10G  588K   10G   1% /dev/shm<br>
</code></pre>
</li><li>如果tmpfs没有启动，需要用命令mount -t tmpfs none /dev/shm -o size=3G 挂载。<br>
</li><li>tmpfs需要3G左右，如果空间不够，需要增加swap空间。</li></ul>




<h3>3.4 chroot到工作环境</h3>
<blockquote>在工作目录下执行gpkg的chroot子命令，代码如下:<br>
<pre><code>bash tools/bin/gpkg -chroot /mnt/xg<br>
/mnt/xg check OK<br>
I have no name!:/#<br>
</code></pre>
</blockquote><blockquote>gpkg的chroot会挂载 tmp,proc,sys,dev目录，并拷贝/etc/resolv.conf文件保证chroot后能正常上网。最后执行真正的chroot命令切换到工作目录。<br>
注意在宿主系统中运行gpkg需要指定bash，否则默认从/tools/bin目录找bash, 会找不到。chroot后运行gpkg则没有这个问题。</blockquote>


<blockquote>成功后，能看到如下信息:<br>
<pre><code>I have no name!:/# ls<br>
dev  lost+found  proc  sys  tmp  tools	var  <br>
I have no name!:/# <br>
<br>
</code></pre></blockquote>



<h3>3.5 更新编译脚本库</h3>
<ul><li>在chroot的环境里,执行gpkg --sync, 可以看到如下信息:<br>
<pre><code>I have no name!:/etc# gpkg --sync<br>
Initialized empty Git repository in /var/xiange/xglibs/.git/<br>
remote: Counting objects: 26, done.<br>
remote: Compressing objects: 100% (20/20), done.<br>
remote: Total 26 (delta 5), reused 0 (delta 0)<br>
Receiving objects: 100% (26/26), done.<br>
Resolving deltas: 100% (5/5), done.<br>
I have no name!:/etc# <br>
</code></pre>
</li></ul><blockquote>gpkg --sync 会利用git下载位于github的软件包脚本库，放入/var/xiange/xglibs目录。</blockquote>

<ul><li>如果失败,是网络连接问题,请检查网络连接和/etc/resolv.conf</li></ul>

<h3>3.6 安装baselayout</h3>
<ul><li>执行gpkg -s base 检查能否从库中找到baselayout<br>
<pre><code>I have no name!:/# gpkg -s base<br>
* sys-apps/baselayout<br>
	Available: 0.2.1<br>
	Installed:	 None<br>
	Desc.    : Base layout for Xiange Linux<br>
	Homepage : http://code.google.com/p/xiangelinux<br>
<br>
I have no name!:/# <br>
</code></pre>
</li><li>gpkg -s 可以指定部分包文件名进行搜索。</li></ul>

<ul><li>执行gpkg -i baselayout 安装baselayout, 系统会自动下载安装,信息如下:<br>
<pre><code>I have no name!:/# gpkg -i baselayout<br>
Begin download baselayout-0.2.1...<br>
downloading from http://xiangelinux.googlecode.com/files/baselayout-0.2.1.tar.bz2, retry 0...<br>
--2010-04-16 11:00:57--  http://xiangelinux.googlecode.com/files/baselayout-0.2.1.tar.bz2<br>
Resolving xiangelinux.googlecode.com... 72.14.203.82<br>
Connecting to xiangelinux.googlecode.com|72.14.203.82|:80... connected.<br>
HTTP request sent, awaiting response... 200 OK<br>
Length: 890 [application/x-bzip2]<br>
Saving to: `/var/xiange/sources/baselayout-0.2.1.tar.bz2.tmp'<br>
<br>
100%[========================================&gt;] 890         --.-K/s   in 0.002s  <br>
<br>
2010-04-16 11:00:57 (418 KB/s) - `/var/xiange/sources/baselayout-0.2.1.tar.bz2.tmp' saved [890/890]<br>
<br>
Download OK<br>
[Unpack] dir=/tmp/xiange/sources<br>
`/bin/sh' -&gt; `/tools/bin/bash'<br>
# make all directorys.<br>
Install to /tmp/xiange/pack/baselayout-0.2.1<br>
`/tmp/xiange/pack/baselayout-0.1.0/usr/man' -&gt; `share/man'<br>
`/tmp/xiange/pack/baselayout-0.1.0/usr/doc' -&gt; `share/doc'<br>
`/tmp/xiange/pack/baselayout-0.1.0/usr/info' -&gt; `share/info'<br>
`/tmp/xiange/pack/baselayout-0.1.0/usr/local/man' -&gt; `share/man'<br>
`/tmp/xiange/pack/baselayout-0.1.0/usr/local/doc' -&gt; `share/doc'<br>
`/tmp/xiange/pack/baselayout-0.1.0/usr/local/info' -&gt; `share/info'<br>
# password/group<br>
Done<br>
Install files to / ...<br>
Install OK<br>
`/bin/bash' -&gt; `/tools/bin/bash'<br>
`/bin/cat' -&gt; `/tools/bin/cat'<br>
`/bin/echo' -&gt; `/tools/bin/echo'<br>
`/bin/grep' -&gt; `/tools/bin/grep'<br>
`/bin/pwd' -&gt; `/tools/bin/pwd'<br>
`/bin/stty' -&gt; `/tools/bin/stty'<br>
`/usr/bin/perl' -&gt; `/tools/bin/perl'<br>
`/usr/lib/libgcc_s.so' -&gt; `/tools/lib/libgcc_s.so'<br>
`/usr/lib/libgcc_s.so.1' -&gt; `/tools/lib/libgcc_s.so.1'<br>
`/usr/lib/libstdc++.so' -&gt; `/tools/lib/libstdc++.so'<br>
`/usr/lib/libstdc++.so.6' -&gt; `/tools/lib/libstdc++.so.6'<br>
`/bin/sh' -&gt; `bash'<br>
I have no name!:/# <br>
<br>
</code></pre></li></ul>

<ul><li>安装完毕后，执行gpkg -I 检查所有已经安装的软件包, 可以看到安装时间, 包内文件有效内容大小, 和名称/版本.<br>
<pre><code>I have no name!:/# gpkg -I<br>
2010-04-16 11:00:58      1K baselayout-0.2.1<br>
I have no name!:/# <br>
<br>
</code></pre>
</li><li>执行 gpkg -l baselayout列出包中所有文件。<br>
<pre><code>I have no name!:/# gpkg -l baselayout<br>
d|      240| /usr<br>
l|       10| /usr/info<br>
l|        9| /usr/doc<br>
l|        9| /usr/man<br>
d|      180| /usr/share<br>
d|       40| /usr/share/zoneinfo<br>
d|       40| /usr/share/terminfo<br>
d|       40| /usr/share/misc<br>
d|      200| /usr/share/man<br>
d|       40| /usr/share/man/man8<br>
d|       40| /usr/share/man/man7<br>
d|       40| /usr/share/man/man6<br>
d|       40| /usr/share/man/man5<br>
d|       40| /usr/share/man/man4<br>
d|       40| /usr/share/man/man3<br>
d|       40| /usr/share/man/man2<br>
d|       40| /usr/share/man/man1<br>
d|       40| /usr/share/locale<br>
d|       40| /usr/share/info<br>
d|       40| /usr/share/doc<br>
d|      220| /usr/local<br>
l|       10| /usr/local/info<br>
l|        9| /usr/local/doc<br>
l|        9| /usr/local/man<br>
d|      180| /usr/local/share<br>
d|       40| /usr/local/share/zoneinfo<br>
d|       40| /usr/local/share/terminfo<br>
d|       40| /usr/local/share/misc<br>
d|      200| /usr/local/share/man<br>
d|       40| /usr/local/share/man/man8<br>
d|       40| /usr/local/share/man/man7<br>
d|       40| /usr/local/share/man/man6<br>
d|       40| /usr/local/share/man/man5<br>
d|       40| /usr/local/share/man/man4<br>
d|       40| /usr/local/share/man/man3<br>
d|       40| /usr/local/share/man/man2<br>
d|       40| /usr/local/share/man/man1<br>
d|       40| /usr/local/share/locale<br>
d|       40| /usr/local/share/info<br>
d|       40| /usr/local/share/doc<br>
d|       40| /usr/local/src<br>
d|       40| /usr/local/sbin<br>
d|       40| /usr/local/lib<br>
d|       40| /usr/local/include<br>
d|       40| /usr/local/bin<br>
d|       40| /usr/src<br>
d|       40| /usr/sbin<br>
d|       40| /usr/lib<br>
d|       40| /usr/include<br>
d|       40| /usr/bin<br>
d|       40| /tmp<br>
d|       40| /root<br>
d|      260| /var<br>
d|       60| /var/xiange<br>
d|       60| /var/xiange/db<br>
d|       60| /var/xiange/db/sys-apps<br>
d|       40| /var/xiange/db/sys-apps/baselayout<br>
d|       40| /var/local<br>
d|       80| /var/lib<br>
d|       40| /var/lib/locate<br>
d|       40| /var/lib/misc<br>
d|       40| /var/cache<br>
d|       40| /var/opt<br>
d|       40| /var/spool<br>
d|       40| /var/run<br>
d|       40| /var/mail<br>
d|       40| /var/log<br>
d|       40| /var/lock<br>
d|       40| /var/tmp<br>
d|       40| /srv<br>
d|       40| /sbin<br>
d|       80| /media<br>
d|       40| /media/cdrom<br>
d|       40| /media/floppy<br>
d|       40| /opt<br>
d|       40| /mnt<br>
d|       40| /lib<br>
d|       40| /home<br>
d|       80| /etc<br>
 |      193| /etc/group<br>
 |       87| /etc/passwd<br>
d|       40| /boot<br>
d|       40| /bin<br>
<br>
File: 2, Dir: 75, Link 6, Size: 1K<br>
</code></pre>
</li><li>注意目录和链接的大小是tmpfs目录下的大小，拷贝到工作分区后，目录占4096个。所以包大小不统计目录和链接大小，只统计文件有效内容大小。</li></ul>

<ul><li>可以用 gpkg -D baselayout 删除软件包, 但在进行下一步前, 要重新执行 gpkg -i baselayout 安装.<br>
</li><li>包安装完毕后, 会自动生成二进制包(<b>.xgp), 放在/var/xiange/packages目录下. 如baselayout.xgp. 下次不想编译时, 可以直接安装二进制包: gpkg -ib /var/xiange/packages/baselayout-0.2.1.xgp<br>
</li><li>安装完baselayout后， 用gpkg --unchroot /mnt/xg, 然后重新执行bash tools/bin/gpkg -chroot /mnt/xg, 既可以去除"I have no name"的提示。<br>
</li><li>文件信息和configure/make check的屏幕记录放在/var/xiange/db/目录下。注意有的软件包没有config过程和make check 过程。</li></ul></b>


<h3>3.7 这时可以自由运行gpkg, 安装需要的软件包。</h3>
<blockquote>安装的列表如下:<br>
注意：随时可以从手动安装切换到自动安装, 用命令 gpkg stage1 即可。gpkg从/var/xiange/xglibs/stages/stage1文件读取待安装包列表，逐一安装。已经手动安装过的包会自动跳过。<br>
注意2: 如果gmp因为64bit的问题configure失败，执行export ABI=32, 再次gpkg -i gmp 或gpkg stage1即可。</blockquote>

<pre><code>     1	baselayout-0.2.1<br>
     2	man-pages-3.24<br>
     3	linux-headers-2.6.32.11<br>
     4	glibc-2.10.1<br>
     5	binutils-2.20<br>
     6	gmp-4.3.2<br>
     7	mpfr-2.4.2<br>
     8	gcc-4.4.3<br>
     9	gpkg-0.10.4<br>
    10	berkeley-db-4.7.25<br>
    11	sed-4.2.1<br>
    12	pkg-config-0.23<br>
    13	ncurses-5.7<br>
    14	util-linux-ng-2.17.2<br>
    15	e2fsprogs-1.41.11<br>
    16	coreutils-8.5<br>
    17	iana-etc-2.30<br>
    18	m4-1.4.14<br>
    19	bison-2.4.2<br>
    20	procps-3.2.8<br>
    21	grep-2.6.3<br>
    22	readline-6.1<br>
    23	bash-4.1<br>
    24	libtool-2.2.6<br>
    25	gdbm-1.8.3<br>
    26	inetutils-1.7<br>
    27	zlib-1.2.5<br>
    28	perl-5.12.0<br>
    29	autoconf-2.65<br>
    30	automake-1.11.1<br>
    31	bzip2-1.0.5<br>
    32	diffutils-2.9<br>
    33	gawk-3.1.7<br>
    34	findutils-4.4.2<br>
    35	flex-2.5.35<br>
    36	gettext-0.17<br>
    37	groff-1.20.1<br>
    38	grub-1.98<br>
    39	gzip-1.4<br>
    40	iproute2-2.6.33<br>
    41	kbd-1.15.2<br>
    42	less-436<br>
    43	make-3.81<br>
    44	man-db-2.5.7<br>
    45	module-init-tools-3.11.1<br>
    46	patch-2.6.1<br>
    47	psmisc-22.11<br>
    48	shadow-4.1.4.2<br>
    49	file-5.04<br>
    50	sysklogd-1.5<br>
    51	sysvinit-2.86<br>
    52	tar-1.23<br>
    53	texinfo-4.13<br>
    54	udev-153<br>
    55	python-2.6.2<br>
    56	linux-kernel-2.6.32.11 (参见下节说明)<br>
    57	git-1.7.0.6<br>
    58	vim-7.2.411<br>
</code></pre>

<h3>3.8 安装内核</h3>
<ul><li>内核也通过gpkg安装, 前提是先准备内核配置文件. 默认内核配置文件是/boot/config-xiange.<br>
</li><li>这个文件由baselayout包创建，缺省是一个通用内核的.config配置文件。<br>
</li><li>用户可以将适用于自己系统的.config文件覆盖缺省文件， 运行gpkg -i linux-kernel 命令启动内核编译, gpkg自动解压内核到临时目录, 并将/boot/config-xiange文件拷贝到内核源码目录,改名为.config, 然后启动编译过程 (make && make modules)<br>
</li><li>编译完成后, gpkg会将bzImage和config安装到/boot目录, 模块安装到/lib/modules目录, 并生成内核文件列表信息.<br>
</li><li>用这种方法，内核文件和模块也会有记录，并采集MD5校验码，比直接安装内核要好。<br>
</li><li>内核安装完毕后会从/tmp/xiange/sources目录清除。如果想编译依赖于内核源码的模块，需要先还原内核源码（必须在原来的编译地址还原)，步骤如下:<br>
<pre><code>    mkdir -p /tmp/xiange/sources<br>
    cd /tmp/xiange/sources<br>
    tar xf /var/xiange/sources/linux-2.XXXX.tar.bz2<br>
    cp /boot/config-xiange linux-2XXX/.config<br>
    make prepare<br>
    make modules_prepare<br>
</code></pre>
</li></ul><blockquote>然后再编译模块就可以了。</blockquote>

<h3>3.9 准备启动</h3>
<ul><li>设置密码，<br>
</li><li>修改/etc/fstab，设置根系统。<br>
</li><li>安装GRUB2<br>
<ol><li>GRUB2的安装十分简单， 执行 grub-install 设备名即可。如系统在/dev/sda8, 执行<br>
<pre><code>    grub-install /dev/sda8<br>
</code></pre></li></ol></li></ul>

<ol><li>加入配置文件到 /boot/grub/grub.cfg<br>
<pre><code><br>
# Timeout for menu<br>
set timeout=5<br>
<br>
# Set default boot entry as Entry 0<br>
set default=0<br>
<br>
#video settings<br>
#insmod video<br>
#insmod vbe<br>
#insmod gfxterm<br>
#insmod jpeg<br>
#<br>
#loadfont /boot/grub/ascii.pff<br>
#set gfxmode="1400x1050x24"<br>
#terminal_output.gfxterm<br>
#background_image /boot/grub/test.jpg<br>
<br>
menuentry "Xiange-2.6.32.11" {<br>
    set root=(hd0,8)<br>
    linux /boot/vmlinuz-2.6.32.11 root=/dev/sda8 quiet<br>
}<br>
<br>
enuentry "Windows 7" {<br>
    set root=(hd0,1)<br>
    chainloader +1<br>
}<br>
<br>
</code></pre>
</li></ol><ul><li>删除/tools目录，就像剪断脐带...<br>
</li><li>执行 gpkg -unchroot /mnt/xg 退出系统<br>
</li><li>卸载/mnt/xg: umount /mnt/xg<br>
</li><li>删除/etc/rc.d/rc3.d/S97trackpoint, 这个只对Thinkpad的小红点有用...否则开机会提示错误。<br>
<pre><code>/etc/rc.d/rc3.d<br>
rm S97trackpoint<br>
</code></pre>
</li><li>重新启动, 新的Linux系统诞生. 如果内核没配置好... 就是大脑发育不全，小心了:)</li></ul>

<h3>3.10 安装Stage2</h3>

<blockquote>Stage2 包含Xorg 7.5, GTK2, wine, 必要的图形/声音库，firefox, pidgin, 和桌面管理器fvwm, 可打造一个工作环境。<br>
</blockquote><ul><li>首先更新编译脚本库<br>
<pre><code>gpkg --sync<br>
</code></pre></li></ul>

<ul><li>升级gpkg到0.10.8, 貌似这个有点难度...因为目前核心包不支持滚动升级。<br>
<pre><code>#删除gpkg的安装信息<br>
rm -rf /var/xiange/db/sys-apps/gpkg/<br>
<br>
#然后强行安装,直接覆盖<br>
gpkg -i gpkg<br>
<br>
#成功后执行gpkg,看版本信息<br>
gpkg -v<br>
0.10.8<br>
</code></pre></li></ul>

<ul><li>自动安装 stage2的话， 只需输入<br>
<pre><code>gpkg stage2<br>
</code></pre></li></ul>

自动安装下列软件包:<br>
<pre><code>     1	x11protos-7.5<br>
     2	x11utils-7.5<br>
     3	ed-1.4<br>
     4	freetype-2.3.12<br>
     5	expat-2.0.1<br>
     6	fontconfig-2.8.0<br>
     7	libxml2-2.7.6<br>
     8	libgpg-error-1.7<br>
     9	libgcrypt-1.4.5<br>
    10	libxslt-1.1.26<br>
    11	x11libs-7.5<br>
    12	libdrm-2.4.14<br>
    13	mesa-7.6<br>
    14	libpng-1.2.43<br>
    15	x11apps-7.5<br>
    16	appres-1.0.2<br>
    17	xfontsel-1.0.2<br>
    18	x11fonts-7.5<br>
    19	wqy-zenhei-0.8.38<br>
    20	wqy-bitmap-1.0.0<br>
    21	xml-parser-2.36<br>
    22	intltool-0.40.6<br>
    23	xkeyboard-config-1.7<br>
    24	pixman-0.15.20<br>
    25	xinit-1.1.1<br>
    26	xterm-253<br>
    27	xorg-server-1.7.1<br>
    28	x11drivers-7.5<br>
    29	curl-7.20.1<br>
    30	libjpeg-8<br>
    31	startup-notification-0.9<br>
    32	giflib-4.1.6<br>
    33	libtiff-3.9.2<br>
    34	openjpeg-1.3<br>
    35	lcms-1.19<br>
    36	libmng-1.0.10<br>
    37	aalib-1.4.5<br>
    38	alsa-lib-1.0.23<br>
    39	alsa-plugins-1.0.23<br>
    40	alsa-utils-1.0.23<br>
    41	slim-1.3.1<br>
    42	dbus-1.2.24<br>
    43	cairo-1.8.10<br>
    44	glib2-2.22.5<br>
    45	pango-1.28.0<br>
    46	atk-1.30.0<br>
    47	shared-mime-info-0.71<br>
    48	hicolor-icon-theme-0.12<br>
    49	gtk2-2.18.9<br>
    50	gperf-3.0.4<br>
    51	dbus-glib-0.86<br>
    52	hal-0.5.14<br>
    53	imlib2-1.4.3<br>
    54	vte-0.22.5<br>
    55	trayer-1.0<br>
    56	habak-0.2.5<br>
    57	lua-5.1.4<br>
    58	conky-1.8.0<br>
    59	cmake-2.8.1<br>
    60	sakura-2.3.8<br>
    61	libnotify-0.4.5<br>
    62	libIDL-0.8.13<br>
    63	librsvg-2.26.3<br>
    64	tcl-8.5.8<br>
    65	tk-8.5.8<br>
    66	sqlite-3.6.23.1<br>
    67	firefox-3.6.3<br>
    68	sun-jdk-6.20<br>
    69	fcitx-3.6.2<br>
    70	fvwm-2.5.29<br>
    71	links-2.2<br>
    72	babl-0.1.2<br>
    73	gegl-0.1.2<br>
    74	viewres-1.0.2<br>
    75	poppler-data-0.4.2<br>
    76	poppler-0.12.4<br>
    77	fribidi-0.10.9<br>
    78	libgsf-1.14.18<br>
    79	libwmf-0.2.8.4<br>
    80	wv-1.2.7<br>
    81	abiword-2.8.4<br>
    82	epdfview-0.1.7<br>
    83	gnutls-2.8.6<br>
    84	pidgin-2.6.6<br>
    85	xlockmore-5.30<br>
    86	libexif-0.6.19<br>
    87	libgphoto2-2.4.9.1<br>
    88	libexif-gtk-0.3.5<br>
    89	gtkam-0.1.17<br>
    90	imagemagick-6.6.1.5<br>
    91	mpg123-1.12.1<br>
    92	wine-1.1.43<br>
    93	skype-2.1.0.81<br>
    94	icoutils-0.29.1<br>
    95	ctags-5.8<br>
    96	lpc21isp-1.79<br>
    97	gimp-2.6.8<br>
    98	git-1.7.0.6<br>
    99	freeglut-2.6.0<br>
   100	ftgl-2.1.3.5<br>
   101	openal-1.12.854<br>
   102	freealut-1.1.0<br>
   103	glpng-1.45<br>
   104	chromium-0.9.14<br>
   105	gamin-0.1.10<br>
   106	pcmanfm-0.5.2<br>
   107	xml-simple-2.18<br>
   108	icon-naming-utils-0.8.90<br>
   109	tango-icon-theme-0.8.90<br>
   110	gtkglext-1.2.0<br>
   111	gliv-1.9.6<br>
   112	celestia-1.6.0<br>
   113	fvwm-xiange<br>
</code></pre>

<h3>3.11 启动Stage2</h3>
<ul><li>首先配置X服务器， 在/root下执行<br>
<pre><code>Xorg -configure<br>
</code></pre>
并把生成的xorg.conf.new拷贝到 /etc/X11/xorg.conf<br>
</li><li>将文泉驿字体路径放到xorg.conf里, misc之后的位置。<br>
<pre><code>        FontPath     "/usr/lib/X11/fonts/misc/"<br>
        FontPath     "/usr/share/fonts/wenquanyi/wqy-bitmap"<br>
        FontPath     "/usr/share/fonts/wenquanyi/wqy-zenhei"<br>
</code></pre>
</li><li>成功后尝试用startx启动X. 可用看到twm, 先忍受一会儿:)<br>
</li><li>如果启动不成功，需要都X的log文件，或修改x11drivers的安装脚本，看看所需的驱动是否在内<br>
</li><li>配置pango，在xterm里：<br>
<pre><code>pango-querymodules &gt; /etc/pango/pango.modules<br>
</code></pre>
否则依赖pango的程序，字体都是大方块<br>
</li><li>配置GTK<br>
<pre><code>gdk-pixbuf-query-loaders &gt; /etc/gtk-2.0/gdk-pixbuf.loaders<br>
gtk-query-immodules-2.0 &gt; /etc/gtk-2.0/gtk.immodules<br>
</code></pre>
否则..很多图标不显示，且中文输入法不起作用。<br>
</li><li>创建一个user用户<br>
<pre><code>    groupadd xiange<br>
    useradd -m -g xiange -G audio,video xiange<br>
    passwd xiange<br>
</code></pre>
注意audio和video,否则一会儿会没声音，不能使用OPENGL图形加速。<br>
创建后/home/xiange目录会有一些配置信息:<br>
<pre><code>sword@XGSvr /home/xiange $ls -la<br>
总用量 32<br>
drwxr-xr-x 4 xiange xiange 4096  5月 14 11:07 .<br>
drwxr-xr-x 5 root   root   4096  5月 14 11:07 ..<br>
-rw-r--r-- 1 xiange xiange   21  5月 14 11:06 .bash_logout<br>
-rw-r--r-- 1 xiange xiange 2180  5月 14 11:06 .conkyrc<br>
drwxr-xr-x 2 xiange xiange 4096  5月 14 11:06 .fvwm<br>
-rw-r--r-- 1 xiange xiange   29  5月 14 11:06 .gtkrc-2.0<br>
drwxr-xr-x 2 xiange xiange 4096  5月 14 11:06 .wallpapers<br>
-rw-r--r-- 1 xiange xiange  462  5月 14 11:06 .xinitrc<br>
</code></pre>
可用把喜欢的图像拷贝到.wallpaper目录， 每次启动时，fvwm会随机选择一张做背景。</li></ul>


<ul><li>自动通过slim启动X,<br>
<pre><code>cd /etc/rc.d/rc3.d<br>
ln -sv /etc/rc.d/init.d/slim S99slim<br>
ln -sv /etc/rc.d/init.d/slim K01slim<br>
</code></pre></li></ul>

<ul><li>加入alsa, 开关机时能听到语音提示<br>
<pre><code>cd /etc/rc.d/rc3.d<br>
ln -sv /etc/rc.d/init.d/alsa S90alsa<br>
ln -sv /etc/rc.d/init.d/alsa K10alsa<br>
</code></pre></li></ul>

<blockquote>重新启动，顺利的话，就能看到和抓图一样的界面。</blockquote>


<h1>4.自动脚本安装</h1>
自动脚本安装通过脚本文件，自动调用gpkg安装所需软件包。自动安装时支持4中安装情景，称为情景0(又称Stage0), 情景1, 情景2, 情景3. 介绍如下:<br>
<br>
<table><thead><th>情景</th><th>简介</th></thead><tbody>
<tr><td>Stage0</td><td>完全从0开始安装，只需下载安装脚本，指定安装的目的分区(或目的目录),运行脚本即可。<br>脚本自动下载所需软件包，从头构建工具链，构建根系统，安装软件包，直至系统完成。</td></tr>
<tr><td>Stage1</td><td>比Stage0简化些，不需在本地构建工具链。预下载内容包括工具链和安装脚本。<br>指定安装的目的分区(或目的目录),运行脚本。<br>脚本自动下载所需软件包，构建根系统，安装软件包，直至系统完成。<br>启动命令: gpkg stage1 </td></tr>
<tr><td>Stage2</td><td>比Stage1简化些，不需要额外的工具链。预下载内容包括跟文件系统，已预装好编译环境和包管理系统。<br>用户只需要在此基础上, 调用gpkg安装自己喜欢的软件包即可。</td></tr>
<tr><td>Stage3</td><td>Stage2的扩充，包括X系统, gtk2, qt4, fvwm, firefox等预装软件包。</td></tr></tbody></table>

用户可以根据需求，从任何一种情景开始安装。情景0和情景1需要编译构建根系统，情景2和情景3解压后直接得到根系统。<br>
根系统准备好后，编译内核，安装Grub, 创建用户，设置密码，然后重新启动即可。<br>
<br>
<h1>1.安装需求</h1>

<table><thead><th>硬件需求</th><th>任何Linux支持的硬件系统，目前只在PC机上测试<br>5G以上的硬盘空间<br>128MB以上的内存</th></thead><tbody>