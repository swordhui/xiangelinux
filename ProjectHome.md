# 弦歌Linux #

[English](main_en.md) [简体中文](http://code.google.com/p/xiangelinux/)

![https://sourceforge.net/p/xiangelinux/wiki/Home/attachment/xiange.png](https://sourceforge.net/p/xiangelinux/wiki/Home/attachment/xiange.png)
## 轻量级面向源码的包管理系统，基于Bash和C，精简易理解，易移植，易扩充 ##
## 支持手动/自动安装，体验全盘控制系统的快感 ##
## 基于源码的Linux发行版，默认针对当前CPU优化，体验极限速度 ##
## 支持二进制软件包 ##
## 支持包文件校验，防止系统被非法修改 ##


---



# 下载地址 #

| |版本|下载地址|平台|备注与文档|
|:|:-----|:-----------|:-----|:--------------|
|![http://sourceforge.net/p/xiangelinux/wiki/applications/attachment/new32.png](http://sourceforge.net/p/xiangelinux/wiki/applications/attachment/new32.png)| **Xiange-x86\_64-LiveUSB-ALL-201406** |[百度云](http://yun.baidu.com/share/link?shareid=1535139273&uk=3557704361)|x86\_64|纯64位桌面系统, 2G U盘安装, 6G软件容量, 用dd安装.  [安装指南](xiange_x86_64_LiveUSB_4G_install_guide.md)|
|![http://sourceforge.net/p/xiangelinux/wiki/applications/attachment/new32.png](http://sourceforge.net/p/xiangelinux/wiki/applications/attachment/new32.png)| **Xiange-i686-LiveUSB-ALL-201406** |[百度云](http://yun.baidu.com/share/link?shareid=3075881173&uk=3557704361)|i686  |32位桌面系统, 2G U盘安装, 6G软件容量，用dd安装.  [安装指南](xiange_x86_64_LiveUSB_4G_install_guide.md)|
| | **Xiange-x86\_64-LiveUSB-4G-201405** |[百度云](http://pan.baidu.com/s/1sjNNpNZ)|x86\_64|纯64位桌面系统, 4GU盘安装, 4G软件容量，用dd安装.  [安装指南](xiange_x86_64_LiveUSB_4G_install_guide.md)|
| | **Xiange-x86\_64-LiveUSB-4G-201404** |[1.source\_forge](http://sourceforge.net/projects/xiangelinux/files/Xiange-x86_64-LiveUSB-4G.tar.xz/download) [2.百度云](http://pan.baidu.com/s/1i3DTLuX)|x86\_64|纯64位桌面系统, 4G U盘安装, 4G软件容量, 用dd安装.  [安装指南](xiange_x86_64_LiveUSB_4G_install_guide.md)|
| | **Xiange-loongson-stage1-1.0** |[google\_code](https://xiangelinux.googlecode.com/files/stage1-loongson2f-n32-1.0.tar.lzma) |mips, 龙芯|龙芯N32  [安装指南](readme_xiangelinux_for_loongson_1.md) [服务器版安装说明](xiange_for_loongson_server.md) |




---



## News ##
  * 2014-07-01  wayland,weston和GTK+3测试成功，准备构建xiange-wayland版本，以wayland,gtk+3,webkitgtk为核心。
  * 2014-06-18  gpkg-0.10.47发布，加入'gpkg -cklfs', 可以自动下载解析LFS/BLFS文档，找出哪些编译脚本可以更新。
  * 2014-06-05  域名[xiangelinux.com](http://www.xiangelinux.com)开始工作， wenruohuan正在建设网站。
  * 2014-06-01  Xiange-i686-LiveUSB-ALL-201406 发布, 只需2G U盘就可安装。
  * 2014-05-29  Xiange-x86\_64-LiveUSB-ALL-201406 发布, 只需2G U盘就可安装。加入libreoffce和图形化硬盘安装程序。[安装指南下载](xiange_x86_64_LiveUSB_4G_install_guide.md)
  * 2014-05-15  Xiange-x86\_64-LiveUSB-4G-201405 发布, 只需解压后dd拷贝到U盘即可。[安装指南下载](xiange_x86_64_LiveUSB_4G_install_guide.md)
  * 2014-04-30  弦歌Linux正式发布! 首发的是x86\_64-LiveUSB-4G版，只需解压后dd拷贝到U盘即可。[安装指南](xiange_x86_64_LiveUSB_4G_install_guide.md) [下载](http://sourceforge.net/projects/xiangelinux/files/Xiange-x86_64-LiveUSB-4G.tar.xz/download)
  * 2014-04-17  Stage2发布倒计时: Initramfs构建完毕，支持root=UUID=“xx”启动方式, 解决了泛用内核定位root的问题.
  * 2014-03-20  弦歌Linux Stage2 (x86\_64)构建中，Xorg-7.7 NetworkManager Slim+FVWM启动成功.
  * 2014-03-14  弦歌Linux Stage1 (x86\_64)发布.  [安装说明](xiange_201403_x86_64_install.md)
  * 2014-03-14  gpkg-0.10.40.1发布.[下载](http://www.sourceforge.net/p/xiange-gpkg)
  * 2014-03-13  弦歌Linux在Sourceforge建立项目xiange-gpkg. [gpkg](http://www.sourceforge.net/p/xiange-gpkg)
  * 2014-03-13  弦歌Linux在Sourceforge建立项目xiangelinux. [弦歌Linux](http://www.sourceforge.net/p/xiangelinux)
  * 2014-01-09  x86\_64工具链构建完毕, 最小系统启动成功.
  * 2014-01-05  arm-eabi工具链构建完毕, 弦歌在嵌入设备上启动成功.
  * 2014-01-02  gpkg: 加入XGROOT,支持交叉编译安装. 加入-ip, 支持只打包不安装.
  * 2014-01-01  gpkg: 加入-pack, 将指定目录所有文件打包, 然后可以用-ib安装.
  * 2010-11-05  gpkg-0.10.11发布, 支持颜色化输出, 改进了chroot
  * 2010-11-03  龙芯版N32弦歌系统1.0在Lemote Yeeloong 8089\_B上启动成功
  * 2010-11-02  龙芯版N32弦歌系统1.0在万由NAS-II上启动成功
  * 2010-11-01  龙芯版N32弦歌系统1.0在Lemote 2F迷你电脑上启动成功
  * 2010-11-01  首个支持龙芯2F的弦歌Linux N32系统正式放出。请点这里[下载](http://code.google.com/p/xiangelinux/downloads/detail?name=stage1-loongson2f-n32-1.0.tar.lzma&can=2&q=#makechanges)
  * 2010-10-06  支持龙芯2F的弦歌Linux N32系统1.0-RC1版正式放出。如果测试没有问题，它将是我们发布的第一个龙芯2F平台的正式版本。
  * 2010-09-13  支持龙芯2F的弦歌Linux N32系统0.2版正式放出，[点这里查看](http://code.google.com/p/xiangelinux/wiki/loongson)
  * 2010-09-06  第一个支持龙芯2F平台的弦歌Linux N32系统启动成功，开始测试。
  * 2010-09-01  龙芯平台的N32bit stage0构建成功，开始基于N32于龙芯的弦歌系统.
  * 2010-08-11  龙芯平台的64bit stage0构建成功，开始基于stage0安装适用于龙芯的弦歌系统.
  * 2010-08-09  龙芯64位工具链自动交叉编译成功，开始制作龙芯64位的stage0二进制包.
  * 2010-06-11  欢迎王江伟<jonsk.echo@gmail.com>加入开发组
  * 2010-05-21  加入命令gpkg -rdep，反查依赖于指定库的软件包。
  * 2010-05-19  加入了festival,stardict,xsel等。festival居然没有官方安装脚本.
  * 2010-05-18  加入openoffice-bin,unrar等编译脚本
  * 2010-05-14  Stage2 自测成功，进入公开测试阶段，包括Xorg7.5, GTK2, Firefox, pidgin, wine等，请参见[SysInstall](SysInstall.md)
  * 2010-05-13  gpkg-0.10.8发布，支持"gpkg stage2"
  * 2010-05-11  Stage1手动/自动安装自测成功，进入公开测试阶段。在[LinuxSir](http://www.linuxsir.org/bbs/thread367410.html)的LFS版发布。
  * 2010-05-07  转入弦歌Linux工作，共安装284个软件包, 列表: [worklist](worklist.md)
  * 2010-04-29  xorg-7.5 启动成功, 共安装171个软件包. 列表: [xorglist](xorglist.md)
  * 2010-04-28  弦歌Linux Stage1 自动编译成功
  * 2010-04-27  弦歌Linux Stage1 启动成功, 脱离/tools运行. 包含57个基本软件包.



---


## 屏幕抓图 ##
桌面使用了Fvwm2, 左上角是快捷启动区，每个图标代表一种应用程序类别，点击鼠标左键出现本类别应用程序列表，点击右键直接启动对应的程序。上方中间是最小化程序列表，程序最小化后放在这里，点击即可恢复正常。右上角是托盘区和系统状态区，显示系统信息如当前CPU运行频率，温度，电池容量，内存使用情况，日期时间，CPU使用图表，硬盘访问密度图表和网络使用图表。<br>
顺便也演示了gpkg的一些用法: 列出已安装软件包，搜索软件包，显示包文件，修改/显示编译参数。<br>
<img src='https://sourceforge.net/p/xiangelinux/wiki/Home/attachment/fvwm-xg-03.png' />


<br><br>
下图显示了二进制包的操作。删除软件包后，再次安装编译生成的二进制包。<br>
<img src='https://sourceforge.net/p/xiangelinux/wiki/Home/attachment/gpkg-binary-inst.png' />

<h2><a href='ScreenShot.md'>更多抓图</a></h2>


<hr />

<h2>Wiki:</h2>
<table><thead><th>1</th><th><a href='NameRule.md'>版本命名规则</a></th><th>swordhuihui</th></thead><tbody>
<tr><td>2</td><td><a href='RoadMap.md'>项目规划</a>       </td><td>swordhuihui</td></tr>
<tr><td>3</td><td><a href='SysInstall.md'>安装方法</a>    </td><td>swordhuihui</td></tr>
<tr><td>4</td><td><a href='xgpkg.md'>包管理系统说明</a></td><td>swordhuihui</td></tr>
<tr><td>5</td><td><a href='xglibs.md'>编译脚本写作方法</a></td><td>swordhuihui</td></tr>
<tr><td>6</td><td><a href='xgfileinfo.md'>xgfileinfo需求</a></td><td>pinkme005  </td></tr>
<tr><td>7</td><td><a href='fvwm_xiange_usage.md'>fvwm-xiange使用说明</a></td><td>swordhuihui</td></tr>
<tr><td>8</td><td><a href='readme_xiangelinux_for_loongson_1.md'>弦歌Linux 1.0 龙芯N32 发行说明</a></td><td>echo.jonsk </td></tr>
<tr><td>9</td><td><a href='NeedYouHelp.md'>需要您的帮助</a></td><td>Xiange Team</td></tr>
<tr><td>10</td><td><a href='otherwiki.md'>其他文档列表</a></td><td>swordhuihui</td></tr></tbody></table>

<hr />
<h3>弦歌Linux由6个子项目组成:</h3>
<h3>1. <a href='xgpkg.md'>包管理系统gpkg</a></h3>
<h3>2. <a href='xglibs.md'>编译脚本库</a></h3>
<h3>3. baselayout (RC系统)</h3>
<h3>4. <a href='fvwm_xiange_usage.md'>桌面fvwm-xiange</a></h3>
<h3>5. 内核配置文件库</h3>
<h3>6. <a href='build_tools.md'>tools构建脚本</a></h3>


<hr />

<h2>已完成功能</h2>
<ul><li>提供所有包文件校验机制，防止其他程序恶意修改系统文件。<br>
</li><li>软件包默认采用-march=native编译， 针对你的CPU优化，达到速度极限<br>
</li><li>简单的编译脚本，写新的编译脚本只需复制模板, 花几分钟修改. 这样可以拥有无限的软件包资源.<br>
</li><li>包编译脚本采用git管理，方便大家共享，很容易组建社区。<br>
</li><li>从头记录所有安装入系统的文件。可以根据文件路径判断文件属于哪个软件包。<br>
</li><li>可以搜索指定目录下不属于任何软件包的文件(即运行时生成的文件)<br>
</li><li>可以搜索指定目录下属于多个软件包的文件(即包文件冲突)<br>
</li><li>使用tmpfs进行编译操作，充分利用大内存。拥有2G内存时，所有编译都在内存中完成，不损伤硬盘。<br>
</li><li>自动以压缩方式记录软件包执行configure时的屏幕输出， 和执行make test时的屏幕输出，同包文件信息放在一起，方便以后查询软件包配置/测试情况。<br>
</li><li>自动对/bin,/sbin,/usr/bin,/usr/sbin目录下的文件进行strip all操作，去掉无用数据段<br>
</li><li>自动对/lib,/usr/lib目录下文件进行strip操作，去掉调试信息。<br>
</li><li>自动对/usr/share/man, /usr/man目录下的文件进行gzip操作，节省硬盘空间。<br>
</li><li>所谓自动，即包编译脚本不用关心，由gpkg统一完成，针对所有软件包。<br>
</li><li>软件编译后自动生成二进制包，下次安装时可以直接安装二进制包， 不用重新编译。注意默认二进制包只能在相同CPU的机器上通用。<br>
</li><li>gpkg已解决软件包依赖关系，自动安装依赖包。<br>
</li><li>已支持滚动升级， 可以系统永久使用。<br>
</li><li>LiveUSB构建完成。<br>
</li><li>已完成284个常用软件的安装脚本 列表: <a href='worklist.md'>worklist</a></li></ul>

<h2>计划中的功能</h2>

<ul><li>制作LiveCD<br>
</li><li>自动解析网页，获取软件包最新版本</li></ul>




<hr />
<h2>适合的用户群:</h2>
<h4>完成过LFS, 想找最具LFS特色发行版的用户</h4>
<h4>Gentoo用户，但感觉Gentoo太复杂不易掌控</h4>
<h4>Arch用户，想追求速度极限</h4>

可以加入弦歌Linux的邮件列表讨论问题<br>
<ul><li>向 xiange-request@freelists.org 发标题为'subscribe'的邮件订阅列表， 或从网站<a href='http://www.freelists.org/list/xiange'>http://www.freelists.org/list/xiange</a> 订阅<br>
</li><li>有新话题时只需向 xiange@freelists.org 发mail即可</li></ul>

<hr />
<h2>Git Hub</h2>
弦歌Linux使用git管理开发中的代码,使用方法如下:<br>
<ul><li>创建<a href='https://github.com/'>git hub</a>账号，按git hub说明加入ssh的公钥<br>
</li><li>包管理器 <a href='http://github.com/swordhui/gpkg/'>gpkg</a>, 欢迎watch和folk. folk后用 <a href='http://github.com/swordhui/gpkg'>git clone git@github.com:swordhui/gpkg.git</a> 下载源码<br>
</li><li>软件包编译脚本库 <a href='http://github.com/swordhui/xglibs'>xglibs</a>, folk后用<a href='http://github.com/swordhui/xglibs'>git clone git@github.com:swordhui/xglibs.git</a> 下载源码</li></ul>

