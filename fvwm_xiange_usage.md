## 0. 简介 ##
fvwm-xiange是[弦歌Linux](http://code.google.com/p/xiangelinux/)默认使用的窗口管理器，以fvwm2为基础，参考并简化了fvwm-crystal. 抓图如下:<br><br>
<img src='http://sourceforge.net/p/xiangelinux/wiki/Home/attachment/fvwm-xg-01.png' />
<br><br>

<h2>1. 快捷启动区</h2>
屏幕左上角为快捷启动区，分为8个类别，分别为网络，图像，声音，系统，科学相关，Wine,游戏,和办公。每个类别里都有不同的应用程序，在图标上点左键会出现程序列表。例如点网络类会出现如下菜单:<br>
<br>
<h2>2. 最小化图标区</h2>
在屏幕信息条的中间， 应用程序最小化后以图标方式放在这里。点击一下即可还原。<br>
在窗口标题上点击右键，会最小化.<br>
<br>
<h2>3. 系统托盘区</h2>
最小化去的右边是系统托盘区，一些聊天软件如pidgin可用利用这个区域显示系统信息。<br>
<br>
<h2>4. 系统信息区</h2>
右上角是系统信息区， 用conky显示系统信息。<br>
第一行显示系统频率，CPU温度, 电池容量, 内存使用量和系统日期。<br>
第二行显示CPU使用图表, 硬盘使用图表, 网络使用图表， 和系统时间，星期几<br>
硬盘使用图表: 可以通过修改~/.conkyrc文件改变监视的磁盘名称, 默认是sdb.<br>
网络使用图表: 可以通过修改~/.conkyrc文件改变监视的网卡名称，默认是eth1.<br>
<br>
<h2>5. 快捷键</h2>
<pre><code>在桌面空白区域点左键，弹出系统菜单。<br>
在桌面空白区域点右键，会出来term, 默认是Sakura。<br>
<br>
按右边的Alt+L, 会锁住屏幕。<br>
按右边的Alt+E, 会弹出文件浏览器，pcmanfm<br>
按右边的Alt+T, 会弹出term<br>
按右边的Alt+F, 会弹出firefox<br>
按右边的Alt+R, 会朗读选中的英文文本，如果装了festival<br>
按右边的Alt+Esc, 会终止朗读文本<br>
<br>
在窗口标题点右键，会最小化窗口。<br>
<br>
期待加入的功能:<br>
右Alt+M, 最小化窗口<br>
右Alt+C, 关闭窗口<br>
右Alt+方向键，调整窗口大小<br>
</code></pre>

<h2>6. 壁纸</h2>
系统启动时，会从~/.wallpapers目录随机读取一个文件做壁纸，所以可以把好看的桌面都扔到那个目录。<br>
<br>
<h2>7. 编码</h2>
fvwm_xiange默认采用zh_CN.utf-8做系统编码。如果碰到以GBK/GB2312编码的文件，可以用iconv转换<br>
<pre><code>iconv -f GBK -t UTF-8 aaa.txt -o aaa-utf8.txt<br>
</code></pre>

如果GBK编码的文件名，可以用:<br>
<pre><code>LANG="zh_CN.GB2312" sakura -l<br>
</code></pre>

启动一个新的GB2312编码的Term, 将文件改名为英文名字。<br>
<br>
然后再utf-8的term里，将英文名改为中文名。<br>
<br>
<h2>8. ToDo List</h2>
这个界面还有几点待改进的地方:<br>
<br>
<ul><li>右Alt + 1,2,3,4 切换工作区<br>
</li><li>快捷键访问快捷菜单<br>
</li><li>窗口操作图标采用了FVWM默认方式，需要修改一下