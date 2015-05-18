# 0. 简介 #

本文描述了弦歌Linux包管理系统gpkg的功能。文档内容基于gpkg-0.10.43.
<br><br>


<h1>1. 包命名</h1>

包管理器以软件包为单位，进行搜索，下载，安装，查询，卸载，升级, 校验等操作。每个软件包都有相对唯一的名称，如:<br>
autoconf-2.63-<a href='https://code.google.com/p/xiangelinux/source/detail?r=1'>r1</a> <br>

包名称由三个部分组成, 软件名称, 版本号, 修订号.<br>
<blockquote>软件名称由$N表示(Name), 如autoconf. 允许中间有'-', 如x-org等.<br>
版本号表示为$V , 即Version. 由几个数字组成, 中间用.隔开, 如2.63, 3.1.0.24等. 注意比较时, 2.10 > 2.9.<br>
修订号表示为$R, 为可选项, 必须由小写的-r开头, 后面跟数字.<br><br></blockquote>

如果软件包的两个版本接口不同，可以共存，如gtk2和gtk3, python2和python3, 软件包名字需要加上识别代号，如gtk2-2.3.1, gtk3-3.1.5. gpkg把gtk2和gtk3当作两个不同的软件包处理。<br>

有的软件包命名不规范，如openssl-1.0.1g, 需要将其转为规范名字如openssl-1.0.1.7.<br>
<br>
<h1>2. 搜索/查询软件包</h1>

<table><thead><th>命令格式</th><th>gpkg -s 软件包名</th></thead><tbody>
<tr><td>示例      </td><td>gpkg -s autoconf    </td></tr>
<tr><td>功能      </td><td>显示软件包最新版本号，已安装的版本号（如果已经安装），下载文件大小，安装后占用空间大小</td></tr>
<tr><td>结果      </td><td>返回0表示精确匹配, 返回1表示模糊匹配, 返回2表示匹配失败</td></tr>
<tr><td>补充说明</td><td>支持模糊搜索/正则表达式搜索</td></tr>
<tr><td>状态      </td><td>已完成           </td></tr>
<br><br></tbody></table>

示例:<br>
<pre><code>gpkg -s gcc<br>
<br>
* sys-devel/gcc<br>
	Available: 4.4.3<br>
	Installed: 4.4.3 | 2010-04-27 15:55:50 35520K<br>
	Desc.    : everyone knows gcc..<br>
	Homepage : http://gcc.gnu.org/<br>
<br>
</code></pre>

<h1>3. 下载软件包</h1>

<table><thead><th>命令格式</th><th>gpkg -d 软件包名, 或软件包名-版本号</th></thead><tbody>
<tr><td>示例      </td><td>gpkg -d autoconf或gpkg -d autoconf-2.0.1      </td></tr>
<tr><td>功能      </td><td>先检查本地磁盘，如果文件已下载且校验正确，直接返回成功<br>如果本地没有，则连接软件源头下载。如果发行尚未下载完的部分，直接断点续传。</td></tr>
<tr><td>结果      </td><td>返回0表示下载/校验成功, 返回1下载失败, 返回2表示校验失败</td></tr>
<tr><td>补充说明</td><td>下载包括必要的补丁文件              </td></tr>
<tr><td>状态      </td><td>已完成                                      </td></tr></tbody></table>

示例:<br>
<pre><code>gpkg -d test<br>
<br>
Begin download test-1.0.0...<br>
init test-1.0.0 build script...<br>
downloading from http://xiangelinux.googlecode.com/files/test-1.0.0.tar.bz2, retry 0...<br>
--2010-05-24 16:46:10--  http://xiangelinux.googlecode.com/files/test-1.0.0.tar.bz2<br>
正在解析主机 xiangelinux.googlecode.com... 72.14.203.82<br>
正在连接 xiangelinux.googlecode.com|72.14.203.82|:80... 已连接。<br>
已发出 HTTP 请求，正在等待回应... 200 OK<br>
长度：407 [application/x-bzip2]<br>
正在保存至: “/var/xiange/sources/test-1.0.0.tar.bz2.tmp”<br>
<br>
100%[========================================&gt;] 407         --.-K/s   in 0.003s  <br>
<br>
2010-05-24 16:46:11 (125 KB/s) - 已保存 “/var/xiange/sources/test-1.0.0.tar.bz2.tmp” [407/407])<br>
<br>
</code></pre>


<h1>4. 安装/升级软件包</h1>
<table><thead><th>命令格式</th><th>gpkg -i 软件名称</th></thead><tbody>
<tr><td>示例      </td><td>gpkg -i autoconf    </td></tr>
<tr><td>功能      </td><td>从配置文件/etc/xgparas读取编译优化参数。没有此文件则使用默认优化参数。<br>找到制定软件包的编译脚本。<br>读取脚本内容，查看本软件包依赖哪些软件包。<br>安装依赖的软件包以满足依赖关系。<br>检查/var/xiange/source目录下有没有编译所需的源码，若没有，自动下载。<br>下载支持断点续传。<br>解压下载的文件，并自动打好补丁(如果有的话)<br>自动按用户制定的编译参数和选项配置软件包，并记录配置信息到文件以备后用。<br>编译软件包<br>如果软件包有自动测试，执行自动测试并记录结果到文件。<br>将软件包安装到临时目录，并自动去除调试信息，压缩文档，去掉多余的本地化文件等瘦身。<br>将临时目录打包， 采集文件指纹信息以备后用。<br>安装编译好的二进制包到系统。安装完成后二进制包并不删除，留在/var/xiange/packages目录以备后用。<br>安装过程支持滚动升级，即使更新正在运行的C库也没问题。<br>最后执行脚本指定的程序配置系统以完成安装。<br></td></tr>
<tr><td>结果      </td><td>返回0表示安装成功, 返回1安装失败</td></tr>
<tr><td>状态      </td><td>已完成           </td></tr></tbody></table>

示例:<br>
<pre><code>gpkg -i test<br>
<br>
Begin download test-1.0.0...<br>
init test-1.0.0 build script...<br>
test-1.0.0.tar.bz2 exists, Skip.<br>
Download OK<br>
unpacking test-1.0.0...<br>
Unpacking to /tmp/xiange/sources<br>
config test-1.0.0...<br>
<br>
-------------------------------------<br>
&gt;&gt;&gt; File List:<br>
总用量 8<br>
-rw-r--r-- 1 sword sword 212  5月 24 16:41 Makefile<br>
-rw-r--r-- 1 sword sword  74  5月 24 16:41 test1.c<br>
&gt;&gt;&gt; XGB_CONFIG: --prefix=/usr <br>
&gt;&gt;&gt; CC: gcc<br>
&gt;&gt;&gt; CFLAGS: -march=native -mtune=native -O2 -fomit-frame-pointer -pipe<br>
&gt;&gt;&gt; CXX: g++<br>
&gt;&gt;&gt; CXXFLAGS: -march=native -mtune=native -O2 -fomit-frame-pointer -pipe<br>
&gt;&gt;&gt; XGPATH_SOURCE: /var/xiange/sources<br>
&gt;&gt;&gt; XGPATH_DEST: /tmp/xiange/pack<br>
&gt;&gt;&gt; current dir: /tmp/xiange/sources/test-1.0.0<br>
-------------------------------------<br>
<br>
make test-1.0.0...<br>
gcc -march=native -mtune=native -O2 -fomit-frame-pointer -pipe -o test test1.c<br>
install to /tmp/xiange/pack/test-1.0.0...<br>
checking test-1.0.0..<br>
mkdir -p /tmp/xiange/pack/test-1.0.0/usr/bin<br>
cp test /tmp/xiange/pack/test-1.0.0/usr/bin<br>
&gt;&gt;&gt; strip debug info..<br>
&gt;&gt;&gt; compress man docs..<br>
&gt;&gt;&gt; make packages... please wait a minute<br>
Moving files to / ...<br>
Install OK, cleaning up..<br>
running after package installed...<br>
[***] test-1.0.0 installed to your system.<br>
</code></pre>

<h1>5. 安装二进制软件包</h1>
<table><thead><th>命令格式</th><th>gpkg -ib 软件包路径名称(.xgp)</th></thead><tbody>
<tr><td>示例      </td><td>gpkg -ib /var/xiange/packages/autoconf-2.20.xgp</td></tr>
<tr><td>功能      </td><td>将制定二进制包装入系统。<br>安装成功后，记录软件包信息，包括每个文件路径，大小，校验码等, 并执行包内安装脚本的xgb_postinst函数。<br>安装过程中会做文件检查，如果发行包内文件和系统文件有冲突，会记录冲突信息到/var/xiange/db/conflicts并提示用户10秒钟，然后覆盖冲突文件。</td></tr>
<tr><td>结果      </td><td>返回0表示安装成功, 返回1安装失败</td></tr>
<tr><td>状态      </td><td>已完成                           </td></tr></tbody></table>

示例:<br>
<pre><code>gpkg -ib /var/xiange/packages/test-1.0.0.xgp <br>
<br>
parsing /var/xiange/packages/test-1.0.0.xgp...<br>
installing app-test/test-1.0.0 ...<br>
Moving files to / ...<br>
Install OK, cleaning up..<br>
init test-1.0.0 build script...<br>
running after package installed...<br>
[OK] Binary package test-1.0.0 installed to you system.<br>
</code></pre>

<h1>6. 升级二进制软件包</h1>
<table><thead><th>命令格式</th><th>gpkg -ub 软件包路径名称(.xgp)</th></thead><tbody>
<tr><td>示例      </td><td>gpkg -ub /var/xiange/packages/autoconf-2.20.xgp</td></tr>
<tr><td>功能      </td><td>升级指定软件包。<br>如果该软件包已安装，会在安装时卸载已安装的版本<br>如果没有安装，和gpkg -ib相同<br>安装成功后，记录软件包信息，包括每个文件路径，大小，校验码等, 并执行包内安装脚本的xgb_postinst函数<br>安装过程中会做文件检查，如果发行包内文件和系统文件有冲突，会记录冲突信息到/var/xiange/db/conflicts并提示用户10秒钟，然后覆盖冲突文件。</td></tr>
<tr><td>结果      </td><td>返回0表示安装成功, 返回1安装失败</td></tr>
<tr><td>状态      </td><td>已完成                           </td></tr></tbody></table>

<h1>7. 卸载软件包</h1>
<table><thead><th>命令格式</th><th>gpkg -D 软件名称</th></thead><tbody>
<tr><td>示例      </td><td>gpkg -D autoconf    </td></tr>
<tr><td>功能      </td><td>卸载软件包，先执行包内脚本的xgb_prerm，删除所有包文件，再执行包内脚本文件的xgb_postrm</td></tr>
<tr><td>结果      </td><td>返回0表示卸载成功, 返回1卸载失败</td></tr>
<tr><td>状态      </td><td>已完成           </td></tr></tbody></table>

示例:<br>
<pre><code>gpkg -D test<br>
init test-1.0.0 build script...<br>
running before package delete...<br>
Remove file /usr/bin/test<br>
running after package delete...<br>
remove file /var/xiange/db/app-test/test/test-1.0.0.config<br>
remove file /var/xiange/db/app-test/test/test-1.0.0.xgb<br>
remove file /var/xiange/db/app-test/test/test-1.0.0.file<br>
remove file /var/xiange/db/app-test/test/test-1.0.0.test<br>
remove file /var/xiange/db/app-test/test/test-1.0.0.file<br>
Remove dir /var/xiange/db/app-test/test<br>
Remove dir /var/xiange/db/app-test<br>
</code></pre>


<h1>8. 校验软件包</h1>
<table><thead><th>命令格式</th><th>gpkg -c 软件名称</th></thead><tbody>
<tr><td>示例      </td><td>gpkg -c autoconf    </td></tr>
<tr><td>功能      </td><td>根据记录的md5校验信息，校验软件包的每个文件，看看是否被修改过</td></tr>
<tr><td>结果      </td><td>返回0表示所有文件校验成功, 返回1表示校验失败</td></tr>
<tr><td>状态      </td><td>已完成           </td></tr></tbody></table>

<h1>9. 列出已安装的软件包</h1>
<table><thead><th>命令格式</th><th>gpkg -I </th></thead><tbody>
<tr><td>示例      </td><td>gpkg -I </td></tr>
<tr><td>功能      </td><td>列出所有已安装的软件包，并显示出安装时间，占用空间大小等信息.<br>可用sort排序，nl计算个数</td></tr>
<tr><td>结果      </td><td>返回0表示成功, 返回1表示没有任何软件包</td></tr>
<tr><td>状态      </td><td>已完成</td></tr></tbody></table>

示例:<br>
<pre><code>gpkg -I | sort | tail<br>
2010-05-21 11:08:18    635K gd-2.0.36<br>
2010-05-21 11:13:06   1812K geoip-1.4.6<br>
2010-05-21 11:14:43    449K conky-1.8.0<br>
2010-05-21 11:25:39    683K libupnp-1.6.6<br>
2010-05-21 11:57:43   7599K amule-2.2.6<br>
2010-05-21 14:10:32  20317K crypto++-5.6.0<br>
2010-05-21 19:17:01    439K libglade-2.6.4<br>
2010-05-21 19:34:47    795K nasm-2.08.01<br>
2010-05-21 20:11:04   6229K snes9x-1.52<br>
2010-05-23 14:11:27      1K mplayer-1.0<br>
</code></pre>


<h1>10. 列出包文件</h1>
<table><thead><th>命令格式</th><th>gpkg -l 软件名称</th></thead><tbody>
<tr><td>示例      </td><td>gpkg -l autoconf    </td></tr>
<tr><td>功能      </td><td>列出属于软件包的所有文件信息，包括类型, 路径，大小</td></tr>
<tr><td>结果      </td><td>返回0表示成功, 返回1表示包没有安装</td></tr>
<tr><td>状态      </td><td>已完成           </td></tr></tbody></table>

示例:<br>
<pre><code>gpkg -l conky<br>
<br>
d|       60| /etc<br>
d|       80| /etc/conky<br>
 |     1830| /etc/conky/conky_no_x11.conf<br>
 |     2549| /etc/conky/conky.conf<br>
d|      100| /usr<br>
d|       60| /usr/lib<br>
d|       40| /usr/lib/conky<br>
d|       60| /usr/share<br>
d|       60| /usr/share/man<br>
d|       60| /usr/share/man/man1<br>
 |    25512| /usr/share/man/man1/conky.1.gz<br>
d|       60| /usr/bin<br>
 |   429764| /usr/bin/conky<br>
<br>
File: 4, Dir: 9, Link 0, Size: 449K<br>
</code></pre>

<h1>11. 查找指定文件归属</h1>
<table><thead><th>命令格式</th><th>gpkg -S 文件名</th></thead><tbody>
<tr><td>示例      </td><td>gpkg -S /usr/bin/autoconf</td></tr>
<tr><td>功能      </td><td>查找指定的文件属于哪个软件包</td></tr>
<tr><td>结果      </td><td>返回0表示成功, 返回1表示没有找到, 返回2表示找到多个(冲突)</td></tr>
<tr><td>状态      </td><td>已完成        </td></tr></tbody></table>

示例:<br>
<pre><code><br>
gpkg -S /usr/bin/conky <br>
<br>
app-admin/conky-1.8.0 : F,/usr/bin/conky<br>
</code></pre>

<h1>12. 查找指定路径下的孤儿文件</h1>
<table><thead><th>命令格式</th><th>gpkg -F 路径</th></thead><tbody>
<tr><td>示例      </td><td>gpkg -F /usr/bin</td></tr>
<tr><td>功能      </td><td>搜索指定路径下不属于任何包的文件, 或被多个包所拥有的文件</td></tr>
<tr><td>结果      </td><td>返回0表示成功, 返回1表示有孤儿或冲突</td></tr>
<tr><td>状态      </td><td>已完成     </td></tr></tbody></table>

示例:<br>
<pre><code>sudo gpkg -F /lib<br>
Password:<br>
<br>
Searching orphans in /lib..<br>
     1 O /lib/firmware/ipw2200-ibss.fw<br>
     2 O /lib/firmware/ipw2200-bss.fw<br>
     3 O /lib/firmware/ipw2200-sniffer.fw<br>
     4 O /lib/firmware/LICENSE.ipw2200-fw<br>
Searched 957 files, 4 orphans, 0 conflicts<br>
<br>
</code></pre>


<h1>13. 同步脚本库</h1>
<table><thead><th>命令格式</th><th>gpkg --sync </th></thead><tbody>
<tr><td>示例      </td><td>gpkg --sync </td></tr>
<tr><td>功能      </td><td>用git同步软件包的编译脚本库</td></tr>
<tr><td>结果      </td><td>返回0表示成功, 返回1表示git失败</td></tr>
<tr><td>状态      </td><td>已完成   </td></tr></tbody></table>

示例:<br>
<pre><code>gpkg --sync<br>
Initialized empty Git repository in /var/xiange/xglibs/.git/<br>
remote: Counting objects: 26, done.<br>
remote: Compressing objects: 100% (20/20), done.<br>
remote: Total 26 (delta 5), reused 0 (delta 0)<br>
Receiving objects: 100% (26/26), done.<br>
Resolving deltas: 100% (5/5), done.<br>
</code></pre>

<h1>14. 显示/修改优化参数</h1>
<table><thead><th>命令格式</th><th>gpkg -info </th></thead><tbody>
<tr><td>示例      </td><td>gpkg -info </td></tr>
<tr><td>功能      </td><td>显示目前gpkg使用编译参数</td></tr>
<tr><td>状态      </td><td>已完成  </td></tr></tbody></table>

示例:<br>
<pre><code>gpkg -info<br>
<br>
XGB_ARCH="i686"<br>
XGB_PREFIX=""<br>
CFLAGS="-march=native -mtune=native -O2 -fomit-frame-pointer -pipe"<br>
CXXFLAGS="$CFLAGS"<br>
XG_I18N="en zh_CN"<br>
XGB_CONFIG="--prefix=/usr "<br>
XGPARA_MAKE="-j 8"<br>
</code></pre>

gpkg从0.10.7开始，先检查/etc/xgparas是否存在，如果存在，使用里面的参数作为编译参数, 否则使用默认优化参数。<br>
默认优化参数为"-march=native -mtune=native -O2 -fomit-frame-pointer -pipe". native是很强悍的优化参数，完全针对当前CPU类型做优化，使软件运行达到极限性能。<br>
如想修改编译参数， 可以用命令:<br>
<pre><code>gpkg -info &gt; /etc/xgparas<br>
</code></pre>

然后修改 /etc/xgparas文件，如将native改为i686, 这样软件包可以在大多数CPU上运行(注意会降低优化性能)<br>
修改完成后再次运行gpkg -info, 可以看到修改已生效:<br>
<pre><code>sed -i "s/native/i686/g" /etc/xgparas<br>
<br>
gpkg -info<br>
<br>
XGB_ARCH="i686"<br>
XGB_PREFIX=""<br>
CFLAGS="-march=i686 -mtune=i686 -O2 -fomit-frame-pointer -pipe"<br>
CXXFLAGS="$CFLAGS"<br>
XG_I18N="en zh_CN"<br>
XGB_CONFIG="--prefix=/usr "<br>
XGPARA_MAKE="-j 8"<br>
</code></pre>

XG_I18N控制允许那些本地化字符串安装在/usr/share/locale目录下。默认允许英文和中文资源安装，其他的在安装前剔除。<br>
<br>
XGPARA_MAKE可以控制Make时并行编译进程数目，默认为CPU核心数目。8核CPU可以大大加快编译速度。<br>
<br>
<h1>15. 显示软件包依赖列表</h1>
<table><thead><th>命令格式</th><th>gpkg -p 包名称 </th></thead><tbody>
<tr><td>示例      </td><td>gpkg -p libpng    </td></tr>
<tr><td>功能      </td><td>分层次显示出依赖的尚未安装的软件包</td></tr>
<tr><td>结果      </td><td>返回0表示成功, 返回1表示失败</td></tr>
<tr><td>状态      </td><td>已完成         </td></tr></tbody></table>

示例:<br>
<pre><code>gpkg -p harfbuzz<br>
<br>
 harfbuzz-0.9.26 (Installed)<br>
     glib2-2.38.2 (Installed)<br>
         libffi-3.0.13 (Installed)<br>
         python-2.7.6 (Installed)<br>
         pcre-8.34 (Installed)<br>
     freetype-2.5.2 (Installed)<br>
         libpng-1.6.9 (Installed)<br>
     icu-52.1 (Installed)<br>
         llvm-3.4 (Installed)<br>
     graphite2-1.2.4 (Installed)<br>
         cmake-2.8.12.2 (Installed)<br>
</code></pre>

<h1>16. 显示软件包反向依赖列表</h1>
<table><thead><th>命令格式</th><th>gpkg -rdep 包名称 </th></thead><tbody>
<tr><td>示例      </td><td>gpkg -rdep libpng    </td></tr>
<tr><td>功能      </td><td>分层次显示出所有依赖于指定软件包的已安装软件包</td></tr>
<tr><td>结果      </td><td>返回0表示成功, 返回1表示git失败</td></tr>
<tr><td>状态      </td><td>已完成            </td></tr></tbody></table>

示例:<br>
<pre><code>gpkg -rdep libpng<br>
<br>
Searching rdep for libpng.so..<br>
2601    Found:0   <br>
Result:<br>
<br>
Searching rdep for libpng12.so..<br>
2601    Found:38  <br>
Result:<br>
1    media-gfx/gtkam-0.1.17<br>
2    media-libs/libwmf-0.2.8.4<br>
3    app-office/abiword-2.8.4<br>
4    media-gfx/gimp-2.6.8<br>
5    media-gfx/icoutils-0.29.1<br>
6    media-libs/gd-2.0.36<br>
7    app-text/wv-1.2.7<br>
8    media-libs/librsvg-2.26.3<br>
9    media-gfx/imagemagick-6.6.1.5<br>
10   x11-apps/xcursorgen-1.0.3<br>
11   x11-misc/trayer-1.0<br>
12   x11-libs/vte-0.22.5<br>
13   x11-misc/slim-1.3.1<br>
14   app-text/poppler-0.12.4<br>
15   app-text/stardict-3.0.1<br>
16   x11-misc/xlockmore-5.30<br>
17   media-glx/gliv-1.9.6<br>
18   net-p2p/amule-2.2.6<br>
19   games-emulation/snes9x-1.52<br>
20   app-text/epdfview-0.1.7<br>
21   sci-astronomy/celestia-1.6.0<br>
22   media-libs/gegl-0.1.2<br>
23   x11-wm/fvwm-2.5.29<br>
24   x11-libs/gtk2-2.18.9<br>
25   x11-terms/sakura-2.3.8<br>
26   games-action/chromium-0.9.14<br>
27   x11-libs/pango-1.28.0<br>
28   x11-libs/libnotify-0.4.5<br>
29   x11-misc/pcmanfm-0.5.2<br>
30   www-client/links-2.2<br>
31   net-im/pidgin-2.6.6<br>
32   x11-libs/wxGTK-2.8.11<br>
33   x11-libs/gtkglext-1.2.0<br>
34   www-client/firefox-3.6.3<br>
35   media-libs/imlib2-1.4.3<br>
36   media-libs/libexif-gtk-0.3.5<br>
37   media-libs/glpng-1.45<br>
38   x11-libs/cairo-1.8.10<br>
39   gnome-base/libglade-2.6.4<br>
</code></pre>

<h1>17. 将指定目录打成指定的二进制包</h1>
<table><thead><th>命令格式</th><th>gpkg -pack 目录名 二进制包名 版本 </th></thead><tbody>
<tr><td>示例      </td><td>gpkg -pack . libnew 1.2                     </td></tr>
<tr><td>功能      </td><td>将指定路径的所有文件打包，并采集所有文件指纹。</td></tr>
<tr><td>结果      </td><td>返回0表示成功, 返回1表示git失败 </td></tr>
<tr><td>状态      </td><td>已完成                                   </td></tr>