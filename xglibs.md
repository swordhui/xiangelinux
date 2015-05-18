# 0. 简介 #

本文描述了弦歌包管理系统[gpkg](xgpkg.md)和编译脚本库之间的接口，和示例脚本的写作方法。<br>
每个发行版本都有自己的编译脚本库，如Gentoo的portage, Slackware的slack build, Arch的abs等。目前的问题是没有统一的编译脚本接口，不能互相通用。不过可以互相借鉴。<br>
本文档基于gpkg-0.10.43.<br>
<br>
<br>
<h1>1. 接口</h1>
接口位于gpkg和包编译脚本之间，以规范化的方法约束gpkg和包编译脚本。对编译脚本来说，接口分为输入接口和输出接口两部分。输入接口初始值由gpkg负责，根据用户设置赋值。输出接口赋值由包编译脚本负责，提供给gpkg使用。接口由bash的变量和函数实现。<br>
<br>
<h2>1.1 输入接口</h2>

输入接口主要是路径信息，编译参数等，由变量实现, 列表如下:<br>
<br>
<table><thead><th>变量名称</th><th>描述</th><th>示例取值</th><th>备注</th></thead><tbody>
<tr><td>$T          </td><td>软件包类别</td><td>"app-sys"   </td><td>gpkg根据用户输入设置初值</td></tr>
<tr><td>$N          </td><td>软件包名称</td><td>"autoconfg" </td><td>gpkg根据用户输入设置初值</td></tr>
<tr><td>$V          </td><td>软件包版本</td><td>"2.6.31"    </td><td>gpkg根据用户输入设置初值</td></tr>
<tr><td>$R          </td><td>软件包修订版本</td><td>"-<a href='https://code.google.com/p/xiangelinux/source/detail?r=1'>r1</a>"</td><td>gpkg根据用户输入设置初值,一般为""</td></tr>
<tr><td>$XGPATH_SOURCE</td><td>源码所在的目录</td><td>"/var/xiange/sources"</td><td>      </td></tr>
<tr><td>$XGPATH_DEST</td><td>安装/打包目录</td><td>"/tmp/xiange/pack/$N-$V"</td><td>      </td></tr>
<tr><td>$CFLAGS     </td><td>编译参数</td><td>"-march=native -mtune=native -O2 -fomit-frame-pointer -pipe"</td><td>gpkg根据/etc/xgparas文件内容赋值，用户修改该文件可以修改此设置。</td></tr>
<tr><td>$CXXFLAGS   </td><td>C++编译参数</td><td>同$CFLAGS  </td><td>同上</td></tr>
<tr><td>XGB_CONFIG  </td><td>autotools配置参数</td><td>--prefix=/usr/</td><td>同上</td></tr>
<tr><td>XGB_ARCH    </td><td>平台类型</td><td>"i686"或"x86_64"</td><td>gpkg从/etc/xgparas读取, 没有的话通过uname获取</td></tr>
<tr><td>XGPARA_MAKE </td><td>编译时Make的参数</td><td>"-j 8"      </td><td>gpkg从/etc/xgparas读取, 没有的话通过nproc获取, 可以充分利用CPU加快编译速度。</td></tr></tbody></table>

关于$T, $N, $V, $R, 可以参见<a href='xgpkg.md'>gpkg</a>中的软件包命名规则。<br>
<br>
<br>
<h2>1.2 输出接口: 变量</h2>
这些变量由编译脚本赋值，gpkg使用这些信息执行自动化操作。<br>
<br>
<table><thead><th>变量名称</th><th>描述</th><th>示例取值</th><th>备注</th></thead><tbody>
<tr><td>DESCRIPTION </td><td>软件包简短描述</td><td>"This is a sample skeleton xiange build script file"</td><td>      </td></tr>
<tr><td>HOMEPAGE    </td><td>软件包主页</td><td>"<a href='http://foo.bar.com/'>http://foo.bar.com/</a>"</td><td>      </td></tr>
<tr><td>SRC_URI     </td><td>源码文件地址</td><td>"<a href='http://foo.bar.com/test.tar.bz2'>http://foo.bar.com/test.tar.bz2</a>"</td><td>多于一个文件时，每行一个文件</td></tr>
<tr><td>BIN_URI     </td><td>二进制包地址</td><td>"<a href='http://foo.bar.com/test.tar.bz2'>http://foo.bar.com/test.tar.bz2</a>"</td><td>多于一个文件时，每行一个文件<br>暂不实现</td></tr>
<tr><td>RDEPEND     </td><td>运行时依赖的软件包</td><td>libpng lua-0.5.1</td><td>目前只支持简单的依赖</td></tr>
<tr><td>DEPEND      </td><td>依赖的软件包，包括编译时</td><td>cmake nasm  </td><td>目前只支持简单的依赖</td></tr>
<tr><td>XGB_CONFIG  </td><td>执行configure时的参数</td><td>"CC=gcc --prefix=/usr"</td><td>gpkg会记录到打包文件里</td></tr></tbody></table>

<h2>1.3 输出接口: 函数</h2>

<h3>1.3.1  xgb_init()</h3>
<blockquote>gpkg调用xgb文件的接口函数前，首先调用xgb_init. 再这里可以重新对一些变量赋值。<br>
gpkg在调用其他函数前，一定会调用此函数一次<br>
若脚本不实现此函数，gpkg会调用默认的xgb_init函数，直接返回0.<br></blockquote>

<blockquote>大部分包编译脚本不需要此函数。vim是个例外，它通过xgb_init重新设置SRC_URI:<br>
<pre><code><br>
SRC_URI="ftp://ftp.vim.org/pub/vim/unix/vim-7.4.tar.bz2"<br>
<br>
VIM_PATCH_MIN=1<br>
VIM_PATCH_MAX=94<br>
VIM_PATCH_URL="ftp://ftp.vim.org/pub/vim/patches/7.4"<br>
<br>
<br>
#init <br>
xgb_init()<br>
{<br>
    local name<br>
    local i<br>
<br>
    echo "Init..."<br>
<br>
    for ((i=$VIM_PATCH_MIN; i&lt;=$VIM_PATCH_MAX; i++))<br>
    do<br>
        name=`printf "7.4.%03d" $i`<br>
        SRC_URI+=" $VIM_PATCH_URL/$name "<br>
        echo $name<br>
    done<br>
}<br>
</code></pre></blockquote>

追加了94个补丁文件供下载使用。<br>

googleearth利用xgb_init重新定义了SRC_URI:<br>
<pre><code><br>
SRC_URI="http://dl.google.com/dl/earth/client/current/google-earth-stable_current_i386.rpm"<br>
<br>
#init <br>
xgb_init()<br>
{<br>
    echo "init $N-$V$R build script..."<br>
    if [ "$XGB_ARCH" == "x86_64" ]; then<br>
        SRC_URI="http://dl.google.com/dl/earth/client/current/google-earth-stable_current_x86_64.rpm"<br>
    fi<br>
}<br>
<br>
</code></pre>


<h3>1.3.2 xgb_unpack()</h3>
<blockquote>gpkg在安装指定软件包前，先调用软件包对应脚本文件的unpack函数。<br>
xgb_unpack需要将下载的源码(位于$XGPATH_SOUCE目录)解压到当前目录，缺省是/tmp/xiange/$N-$V$R.<br>
成功返回0, 否则1并显示错误信息.</blockquote>

<h3>1.3.3 xgb_config()</h3>
<blockquote>gpkg在调用玩xgb_unpack后，调用xgb_config配置软件包。<br>
调用xgb_config前，当前目录仍在解压目录(即/tmp/xiange/$N-$V$R/)<br>
xgb_config 首先要进入或创建编译目录，打上所有补丁(如果有的话), 必要时, 可以重新执行autoconf, automake, 生成新的configure文件。然后调用configure配置软件包。配置使用$XGB_CONFIG, 初始值由gpkg设置，如--prefix, --sysconfdir等, 编译脚本可以追加自己特有的配置参数:<br>
<pre><code>    XGB_CONFIG+=" --enable-lua --disable-static "<br>
<br>
    #Third, call configure with $XGB_CONFIG<br>
    ./configure $XGB_CONFIG<br>
</code></pre></blockquote>

<blockquote>成功返回0, 否则返回2并显示错误信息.<br></blockquote>


<h3>1.3.4 xgb_build()</h3>
<blockquote>gpkg调用脚本config成功后，调用build功能编译软件包.<br>
一般来说xgb_build()调用make $XGPARA_MAKE 编译整个软件包，然后根据需要编译文档.<br>
成功返回0, 否则3并显示错误信息.<br></blockquote>


<h3>1.3.5 xgb_install()</h3>
<blockquote>编译完成后，gpkg调用install将软件包安装到临时打包目录/tmp/xiange/$N-$V$R/pack/. 这个位置用$XGPATH_DEST表示。脚本编写者要读注意不能直接安装到根目录！或其他任何$XGPATH_DEST之外的目录， 这样会导致gpkg无法采集文件指纹信息。<br>
对于大部分情况, 指定make DESTDIR=$XGPATH_DEST install 即可，极少不用autotools或cmake的软件包，可以尝试用make --prefix=$XGPATH_DEST/usr。如果prefix也不行，需要脚本编写者读懂Makefile找出相应的方案，这种情况极少发生。<br>
安装成功后返回0, 失败返回4并显示错误信息。</blockquote>


<h3>1.3.6 xgb_postinst()</h3>
<blockquote>gpkg将软件包拷贝到根目录后, 调用脚本的postinst , 执行一些特有的操作, 如创建用户, 修改启动脚本等.这些功能可以调用gpkg提供的接口函数完成。</blockquote>

<h3>1.3.7 xgb_prerm()</h3>
<blockquote>gpkg删除软件包所有文件和目录前，调用prerm函数进行预清除，如删除没有记录到列表中的文件。</blockquote>

<h3>1.3.8 xgb_postrm()</h3>
<blockquote>gpkg删除软件包所有文件和目录后，调用postrm函数进行最后的清除，如删除用户，去掉启动脚本等。。</blockquote>

<h3>1.3.9 xgb_checkupdate()</h3>
<blockquote>需要检查当前软件有没有最新版本时, 调用这个功能. <br>
脚本下载软件所在的网页, 解析网页找到最新的版本, 显示出来。</blockquote>

<blockquote>可以写一个模板文件, 想编译新软件包时, 拷贝模板文件并编译即可.</blockquote>


<h1>2. 示例脚本</h1>
<pre><code>#!/bin/bash<br>
#<br>
# Xiange Linux build scripts<br>
<br>
# Short one-line description of this package.<br>
DESCRIPTION="GNU awk pattern-matching language"<br>
<br>
# Homepage, not used by Portage directly but handy for developer reference<br>
HOMEPAGE="http://www.gnu.org/software/gawk/gawk.html"<br>
<br>
# Point to any required sources; these will be automatically downloaded by<br>
# gpkg. <br>
# $N = package name, such as autoconf, x-org<br>
# $V = package version, such as 2.6.10<br>
<br>
SRC_URI="http://ftp.gnu.org/gnu/gawk/$N-$V.tar.gz"<br>
<br>
<br>
# Binary package URI.<br>
BIN_URI=""<br>
<br>
<br>
# Runtime Depend<br>
RDEPEND=""<br>
<br>
# Build time depend<br>
DEPEND="${RDEPEND}"<br>
<br>
#unpack<br>
xgb_unpack()<br>
{<br>
        #unpard file from $XGPATH_SOURCE to current directory.<br>
        echo "Unpacking to `pwd`"<br>
        tar xf $XGPATH_SOURCE/$N-$V$R.tar.gz<br>
}<br>
<br>
#config<br>
xgb_config()<br>
{<br>
        echo "config $N-$V$R..."<br>
<br>
        #fist, cd build directory<br>
        cd $N-$V$R<br>
<br>
        #second, add package specified config params to XGB_CONFIG<br>
        XGB_CONFIG+=" --libexecdir=/usr/lib "<br>
<br>
        #Third, call configure with $XGB_CONFIG<br>
        ./configure $XGB_CONFIG<br>
}<br>
<br>
#build<br>
xgb_build()<br>
{<br>
        echo "make $N-$V$R..."<br>
<br>
        #run make in current directory<br>
        make<br>
}<br>
<br>
#check<br>
xgb_check()<br>
{<br>
        echo "checking $N-$V$R.."<br>
        make check<br>
}<br>
<br>
#install<br>
xgb_install()<br>
{<br>
        echo "install to $XGPATH_DEST"<br>
<br>
        #install everything to $XGPATH_DEST<br>
        make DESTDIR=$XGPATH_DEST install<br>
}<br>
<br>
<br>
</code></pre>

<h1>3. 模板</h1>

模板文件在/var/xiange/xglibs/template.xgb, 写新的脚本文件时，只需要拷贝并编辑模板文件即可。<br>
<br>
<h1>4. DIY: 动手写测试脚本</h1>

<h2>4.1 复制模板</h2>

gpkg 提供快速拷贝模板生成编译脚本的方法:<br>
<pre><code>gpkg new 类型/软件包-版本号, 例如:<br>
gpkg new x11-apps/gtktest-1.2.3<br>
</code></pre>

gpkg会尝试找任何gtktest的编译脚本，如果找到任何一个版本，就赋值该版本的编译脚本做模板，如果找不到任何版本，就赋值标准模板。<br>
<br>
还有更快速的方法，就是当软件包有旧版本时，不需指定类型，如:<br>
<pre><code>gpkg new vim-7.6<br>
</code></pre>

gpkg会自动寻找目前最新的编译脚本，复制成vim-7.6的编译脚本。<br>
<br>
如果vim-7.6的编译脚本已经存在, gpkg new vim-7.6直接编辑存在的脚本。<br>
<br>
<br>
<h2>4.2 修改信息</h2>

对于没有任何旧版本的新包来说，需要修改DESCRIPTION和HOMEPAGE变量，让其更有意义。修改完后再次用 gpkg -s test, 可以看到信息已更新。<br>
<pre><code>vi test-1.0.0.xgb <br>
<br>
gpkg -s test<br>
<br>
* app-test/test<br>
	Available: 1.0.0<br>
	Installed:	 None<br>
	Desc.    : xiange build script for testing<br>
	Homepage : http://code.google.com/p/xiangelinux/<br>
</code></pre>


<h2>4.3 下载软件包</h2>
只需要修改$SRC_URI 为测试工程的下载地址http://xiangelinux.googlecode.com/files/test-1.0.0.tar.bz2.<br>
<pre><code>test-1.0.0.xgb, 第17行<br>
SRC_URI="http://xiangelinux.googlecode.com/files/test-1.0.0.tar.bz2"<br>
<br>
如果有多个文件，可以每行一个。<br>
</code></pre>

然后执行:<br>
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

文件保存在/var/xiange/sources/test-1.0.0.tar.bz2,<br>
<pre><code>ls -l /var/xiange/sources/test*<br>
-rwxrwxr-x 1 sword root 407  5月 24 16:43 /var/xiange/sources/test-1.0.0.tar.bz2<br>
</code></pre>

<h2>4.4 解压</h2>
修改test-1.0.0.xgb的xgb_unpack函数，如下:<br>
<pre><code>xgb_unpack()<br>
{<br>
        #unpard file from $XGPATH_SOURCE to current directory.<br>
        echo "Unpacking to `pwd`"<br>
        tar xf $XGPATH_SOURCE/`basename $SRC_URI`<br>
}<br>
</code></pre>

可以看到此函数打印当前的路径信息，然后将位于$XGPATH_SOURCE的test-1.0.0.tar.bz2解压到当前路径。为了测试，先堵住xgb_config()的执行,修改如下:<br>
<pre><code><br>
xgb_config()<br>
{<br>
        echo "config $N-$V$R, this is not a error"<br>
        return 1<br>
}<br>
</code></pre>

返回1表示xgb_config()出了问题，gpkg就不会继续执行。这样可以测试解压过程。<br>
启动安装过程:<br>
<pre><code>gpkg -i test<br>
Begin download test-1.0.0...<br>
init test-1.0.0 build script...<br>
test-1.0.0.tar.bz2 exists, Skip.<br>
Download OK<br>
<br>
unpacking test-1.0.0...<br>
Unpacking to /tmp/xiange/sources<br>
config test-1.0.0... this is not a error<br>
[Error] call xgb_config failed<br>
</code></pre>

可以看到先调用xgb_init(), 然后检查文件是否已下载，如果已经下载，开始调用xgb_unpack().<br>
xgb_unpack()将软件包解压到/tmp/xiange/sources目录:<br>
<pre><code>ls -l /tmp/xiange/sources/<br>
总用量 0<br>
drwxr-xr-x 2 sword sword 80  5月 24 16:41 test-1.0.0<br>
</code></pre>

<h2>4.5 配置</h2>
test软件包没有使用autotools, 没有configure的过程。我们在这里打印一些输入参数:<br>
<pre><code>#config<br>
xgb_config()<br>
{<br>
        echo "config $N-$V$R..." <br>
<br>
        cd $N-$V<br>
<br>
        echo ""<br>
        echo "-------------------------------------"<br>
        echo "&gt;&gt;&gt; File List:"<br>
        ls -l<br>
<br>
<br>
        echo "&gt;&gt;&gt; XGB_CONFIG: $XGB_CONFIG"<br>
        echo "&gt;&gt;&gt; CC: $CC"<br>
        echo "&gt;&gt;&gt; CFLAGS: $CFLAGS"<br>
        echo "&gt;&gt;&gt; CXX: $CXX"<br>
        echo "&gt;&gt;&gt; CXXFLAGS: $CXXFLAGS"<br>
        echo "&gt;&gt;&gt; XGPATH_SOURCE: $XGPATH_SOURCE"<br>
        echo "&gt;&gt;&gt; XGPATH_DEST: $XGPATH_DEST"<br>
        echo "&gt;&gt;&gt; current dir: `pwd`"<br>
        echo "-------------------------------------"<br>
<br>
        echo ""<br>
        sleep 1<br>
}<br>
</code></pre>

同时堵住xgb_build()的执行:<br>
<pre><code>#build<br>
xgb_build()<br>
{<br>
        echo "make $N-$V$R... this is not a error"<br>
        return 1<br>
}<br>
</code></pre>

执行如下 gpkg -i test:<br>
<pre><code>gpkg -i test<br>
<br>
Begin download test-1.0.0...<br>
init test-1.0.0 build script...<br>
test-1.0.0.tar.bz2 exists, Skip.<br>
Download OK<br>
unpacking test-1.0.0...<br>
Unpacking to /tmp/xiange/sources<br>
<br>
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
make test-1.0.0... this is not a error<br>
[Error] call xgb_build failed<br>
</code></pre>

可以看到输入参数的取值。<br>

<h2>4.6 编译</h2>

对大部分软件包来说，编译就是执行make. 我们在这里指定编译参数:<br>
<pre><code>#build<br>
xgb_build()<br>
{<br>
        echo "make $N-$V$R..."<br>
        make CC="$CC" CFLAGS="$CFLAGS"<br>
}<br>
<br>
</code></pre>

同时堵住xgb_install()的执行。<br>
<pre><code>#install<br>
xgb_install()<br>
{<br>
        echo "install to $XGPATH_DEST... this is not a error"<br>
<br>
        return 1<br>
<br>
        #install everything to $XGPATH_DEST<br>
        #make DESTDIR=$XGPATH_DEST install<br>
}<br>
</code></pre>

注意不要堵xgb_check()..  gpkg不会检查xgb_check的返回值，因为有些包永远都是测试失败...<br>
<br>
再次执行 gpkg -i test<br>
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
<br>
install to /tmp/xiange/pack/test-1.0.0... this is not a error<br>
[Error] call xgb_install failed<br>
<br>
</code></pre>

可以看到编译参数起作用了。 运行一下test:<br>
<pre><code>/tmp/xiange/sources/test-1.0.0/test <br>
hello, world<br>
</code></pre>

<h2>4.7安装</h2>
将软件包安装到$XGPATH_DEST目录. 在写xgb_install()函数之前，请仔细阅读软件包的Makefile文件。<br>
<pre><code><br>
#install<br>
xgb_install()<br>
{<br>
        echo "install to $XGPATH_DEST..."<br>
<br>
        #install everything to $XGPATH_DEST<br>
        make DESTDIR=$XGPATH_DEST install<br>
}<br>
</code></pre>

然后再次执行 gpkg -i test.<br>
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

Bingo! 安装成功了。我们用gpkg -s test 看一下:<br>
<pre><code>gpkg -s test<br>
<br>
* app-test/test<br>
	Available: 1.0.0<br>
	Installed: 1.0.0 | 2010-05-24 17:32:09 3K<br>
	Desc.    : xiange build script for testing<br>
	Homepage : http://code.google.com/p/xiangelinux/<br>
</code></pre>

用gpkg -I 看一下:<br>
<pre><code><br>
gpkg -I | sort | tail<br>
<br>
2010-05-21 11:14:43    449K conky-1.8.0<br>
2010-05-21 11:25:39    683K libupnp-1.6.6<br>
2010-05-21 11:57:43   7599K amule-2.2.6<br>
2010-05-21 14:10:32  20317K crypto++-5.6.0<br>
2010-05-21 19:17:01    439K libglade-2.6.4<br>
2010-05-21 19:34:47    795K nasm-2.08.01<br>
2010-05-21 20:11:04   6229K snes9x-1.52<br>
2010-05-24 09:33:06     35K gpkg-0.10.9<br>
2010-05-24 17:32:09      3K test-1.0.0<br>
</code></pre>

用 gpkg -l test 看一下:<br>
<pre><code>gpkg -l test<br>
d|       60| /usr<br>
d|       60| /usr/bin<br>
 |     2792| /usr/bin/test<br>
<br>
File: 1, Dir: 2, Link 0, Size: 3K<br>
</code></pre>

看看configure的输出:<br>
<pre><code>vi /var/xiange/db/app-test/test/test-1.0.0.config.gz<br>
</code></pre>

删除软件包:<br>
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

测试安装二进制软件包:<br>
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

运行test<br>
<pre><code>/usr/bin/test <br>
hello, world<br>
</code></pre>

其他的函数可以从脚本里删除。