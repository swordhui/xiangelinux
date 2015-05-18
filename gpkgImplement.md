#用户接口gpkg的实现方案

# 0.简介 #

本文介绍了弦歌Linux包管理系统gpkg的实现方案。关于gpkg的功能说明，请参考文档[xgpkg](xgpkg.md)


# 1.下载软件包 #

下载软件包通过gpkg -d 命令, 例如 gpkg -d autoconf. <br>
首先，gpkg需要知道autoconf软件包的下载地址(URL), 和校验码/大小信息。这些信息每个软件包都不同，所以每个软件包都需要一个脚本文件，描述软件包相关的信息。假如autoconf对应的脚本文件为autoconf.xgb (xiange build file), 下载的流程如下<br>
<br>
1.1 得到软件包对应的脚本文件autoconf-2.61.xgb,失败则返回1<br>
<ul><li>可以通过函数find_pack()实现<br>
</li><li>find_pack() 先用ls列出所有包类型目录中的子目录，放入变量pack_type里<br>
</li><li>对pack_type里的个子目录, 用ls列出属于这个类别的所有包，进行匹配操作，如果成功，将结果放入result变量里，如app-sys/autoconf, app-test/autoconf-test<br>
</li><li>继续操作直到所有目录匹配完成<br>
</li><li>对每个result里的结果，找到最新的包编译脚本文件，如app-sys/autoconf/autoconf-2.63.xgb<br>
</li><li>如果搜索到多个结果，让用户选择下载哪个<br>
1.2 调用包编译脚本文件的接口函数xgb_init<br>
</li><li>将步骤1.1得到的xgb文件包含进来，如“. path/app-sys/autoconf/autoconf-2.63.xgb”<br>
</li><li>调用接口函数xgb_init()<br>
1.3 从接口变量SRC_URI获取待下载文件的URL, 可能有多个<br>
1.3 对每个文件， 从源码目录(/var/xiange/sources/)检查文件是否已下载，是的话转1.5<br>
1.4 下载文件到本地(/var/xiange/sources/)，失败后重试。重试次数超过5,失败返回2<br>
1.5 校验MD5Sum和文件大小，失败后重新下载，重试次数超过5， 返回3<br>
1.6 重复直至所有文件下载完毕，返回0表示成功。<br></li></ul>

<h1>2. 软件包脚本文件</h1>

从上面可以看到，每个软件包需要自己的编译脚本，提供以下函数(功能):<br>
<br>
<h3>2.1  变量</h3>
<blockquote>编译脚本文件需要提供如下变量给gpkg：<br></blockquote>

<table><thead><th>变量名称</th><th>描述</th><th>示例取值</th><th>备注</th></thead><tbody>
<tr><td>DESCRIPTION </td><td>软件包简短描述</td><td>"This is a sample skeleton xiange build script file"</td><td>      </td></tr>
<tr><td>HOMEPAGE    </td><td>软件包主页</td><td>"<a href='http://foo.bar.com/'>http://foo.bar.com/</a>"</td><td>      </td></tr>
<tr><td>SRC_URI     </td><td>源码文件地址</td><td>"<a href='http://foo.bar.com/test.tar.bz2'>http://foo.bar.com/test.tar.bz2</a>"</td><td>多于一个文件时，每行一个文件</td></tr>
<tr><td>BIN_URI     </td><td>二进制包地址</td><td>"<a href='http://foo.bar.com/test.tar.bz2'>http://foo.bar.com/test.tar.bz2</a>"</td><td>多于一个文件时，每行一个文件<br>暂不实现</td></tr>
<tr><td>XGB_CONFIG  </td><td>执行configure时的参数</td><td>"CC=gcc --prefix=/usr"</td><td>gpkg会记录到打包文件里</td></tr>
<br>
<br></tbody></table>

<blockquote>编译脚本也可以使用如下gpkg提供的变量(注意在脚本里不要使用这些变量，或修改这些变量的取值)<br>
<table><thead><th>变量名称</th><th>描述</th><th>示例取值</th><th>备注</th></thead><tbody>
<tr><td>N           </td><td>软件包名称</td><td>"autoconfg" </td><td>      </td></tr>
<tr><td>V           </td><td>软件包版本</td><td>"2.6.31"    </td><td>      </td></tr>
<tr><td>T           </td><td>软件包类别</td><td>"app-sys"   </td><td>      </td></tr>
<tr><td>XGPATH_SCRIPT</td><td>编译脚本所在的目录</td><td>"/var/xiange/xglibs/app-sys/autoconf"</td><td>      </td></tr>
<tr><td>XGPATH_SOURCE</td><td>源码所在的目录</td><td>"/var/xiange/sources"</td><td>      </td></tr>
<tr><td>XGPATH_UNPACK</td><td>解压目标目录</td><td>"/tmp/xiange/sources"</td><td>      </td></tr>
<tr><td>XGPATH_DEST </td><td>临时安装目录</td><td>"/tmp/xiange/pack/autoconf-2.6.8"</td><td>      </td></tr>
<tr><td>XGB_CONFIG  </td><td>编译参数</td><td>CFLAGS="-march=native -mtune=native -O2" --prefix=/usr/</td><td>进行configure时, 可以指定$XGB_CONFIG做参数. 软件包特定的部分用 XGB_CONFIG+=" --pack=abc "实现.</td></tr></blockquote></tbody></table>

<blockquote>gpkg在调用编译脚本前，会首先对这些变量赋值。<br>
<br></blockquote>


<h3>2.2  xgb_init()</h3>
<blockquote>gpkg调用xgb文件的接口函数前，首先调用xgb_init. 再这里可以重新对一些变量赋值。<br>
这个函数可以不实现，gpkg会调用默认的xgb_init函数，直接返回0.</blockquote>

<h3>2.3 Remove this</h3>
<blockquote>告诉gpkg软件包的依赖关系. 这是一个变量。<br>
如"gtk2 libxxx"<br>
用空格隔开. 不依赖任何包包时为空字串"".</blockquote>

<blockquote>复杂一点可以支持版本限定, 如<br>
gtk(>2.12) libxxx(<3.0) foo(=2.1)  bar(>2.13<2.15)<br></blockquote>

<blockquote>再复杂一点支持条件编译<br>
如 "gtk[X,-xxx](>2.12) libxxx[opt1,-opt2](=2.1)"</blockquote>



<h3>2.3 xgb_unpack()</h3>
<blockquote>gpkg在安装指定软件包前，先调用软件包对应脚本文件的unpack函数。<br>
xgb_unpack需要将下载的源码(位于$XGSOUCE目录)解压到$XGUNPACK目录，缺省是/tmp/xiange/sources/, 并打上所有补丁, 必要时, 可以重新执行autoconf, automake, 生成新的configure文件<br>
成功返回0, 否则1并显示错误信息.</blockquote>


<h3>2.4 xgb_config()</h3>
<blockquote>gpkg在调用玩xgb_unpack后，调用xgb_config配置软件包。<br>
调用xgb_config前，当前目录仍在$XGUNPACK目录(即/tmp/xiange/sources/)<br>
xgb_config 首先要进入或创建编译目录，然后调用configure配置软件包。配置由三个大部分组成, 一为安装目录, 即--prefix, --sysconfdir等, 这个每个软件包都一样, 用变量$XGCONFPATH表示<br>
$XGCONFPATH=--prefix=/usr --sysconfdir=/etc 还有其他的运行库目录了, 文档目录等.<br></blockquote>

<blockquote>第二部分为C/C++的优化参数, 用$XG_CFLAGS表示, 如-o2等<br></blockquote>

<blockquote>第三步部分为软件包特有的, 可以直接写到脚本里, 或暴露给用户设置(待讨论).<br></blockquote>

<blockquote>成功返回0, 否则返回2并显示错误信息.<br></blockquote>


<h3>2.5 xgb_build()</h3>
<blockquote>gpkg调用脚本config成功后，调用build功能编译软件包.<br>
xgb_build应该调用make编译整个软件包，然后根据需要编译文档<br>
成功返回0, 否则3并显示错误信息.<br></blockquote>


<h3>2.6 xgb_install()</h3>
<blockquote>编译完成后，gpkg调用install将软件包安装到临时打包目录/tmp/xiange/包名称/pack/. 这个位置用$XGDEST表示。脚本编写者要读注意不能直接安装到根目录！<br>
xgb_install应该调用make install将软件包安装在这个临时目录。脚本编写者要读懂make install的代码，确保软件包只安装在临时目录，不会安装在根目录。<br>
安装成功后返回0, 失败返回4并显示错误信息。</blockquote>

<blockquote>xgb_install返回后，gpkg会枚举$XGDEST目录下所有文件，对每个文件使用记录工具xgfileinfo得到文件信息，包括文件名，文件类型，最后修改时间，权限，大小，MD5等信息，记录到数据库文件$XGDEST/var/xiange/db/包名称/file文件，然后将软件包信息，包括软件包名称, 配置参数, 安装时间, 占用磁盘大小等存入$XGDEST/var/xiange/db/包名称/info文件. 然后将软件包编辑脚本拷贝到$D/var/xiange/db/包名称/xgbuild. 最后，将$XGDEST目录的所有文件用tar打成二进制包，如binutils-2.6.8.tar.bz2， 拷贝到/var/xiange/packages目录.这个二进制包可以分发<br></blockquote>

<blockquote>最后一步是将二进制包解压到根目录,然后调用下面要提到的xgb_postinst.</blockquote>


<h3>2.7 xgb_postinst()</h3>
<blockquote>gpkg将软件包拷贝到根目录后, 调用脚本的postinst , 执行一些特有的操作, 如创建用户, 修改启动脚本等.这些功能可以调用gpkg提供的接口函数完成。</blockquote>

<h3>2.8 xgb_prerm()</h3>
<blockquote>gpkg删除软件包所有文件和目录前，调用prerm函数进行预清除，如删除没有记录到列表中的文件。</blockquote>


<h3>2.9 xgb_postrm()</h3>
<blockquote>gpkg删除软件包所有文件和目录后，调用postrm函数进行最后的清除，如删除用户，去掉启动脚本等。。</blockquote>


<h3>2.10 xgb_checkupdate()</h3>
<blockquote>需要检查当前软件有没有最新版本时, 调用这个功能. <br>
脚本下载软件所在的网页, 解析网页找到最新的版本, 显示出来。</blockquote>

<blockquote>可以写一个模板文件, 想编译新软件包时, 拷贝模板文件并编译即可.</blockquote>

<h3>2.11 gpkg和包编译脚本的接口</h3>
<blockquote>gpkg想调用以上功能时，需要将脚本包含进来，直接调用相关的函数即可。如果编译脚本名为autoconf-2.63.xgb,gpkg调用时需要: <br>
. path/autoconf-2.63.xgb<br>
gpkg_getsrc <br>
</blockquote><ol><li>check return value. <br></li></ol>



<h1>3. 脚本库</h1>

<blockquote>软件包脚本库和Gentoo的ebuild库类似, 根据功能类别分组, 放在/var/xiange/xglibs/目录, 每个类别有自己的目录, 如app-sys.<br>
类别目录下为软件目录, 由软件名称决定, 如autoconf, automake等.<br>
软件目录里包含软件各个版本的安装脚本，如autoconf-2.61.xgp, autoconf-2.62.xgp等<br></blockquote>

<blockquote>弦歌的脚本库用git维护，公开地址为 git://github.com/swordhui/xglibs.git <br></blockquote>

<h1>4. 编译脚本可用的函数接口</h1>

<blockquote>一些经常使用的功能可用写成函数，供软件包编译脚本使用。