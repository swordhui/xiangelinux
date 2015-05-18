为什么会有这个工程呢?<br>
我从04年开始玩Linux, 当时从Knoppix开始的。Knoppix是基于Debian的一个LiveCD, 我尝试了将其安装到硬盘，美化，定制内核等。在使用的过程中，发现网络上有很多Gentoo的狂热分子，受其感染，开始使用Gentoo. 直到09年8月，决定从当前的工作辞职。在交接工作的一个月里，开始构建LFS. 大概用了一个月左右，构建了一套LFS系统，并写了自己的自动构建脚本。(即0.9版本). <br><br>
在构建的过程中，从网络上发现了LinuxSir的LFS论坛，于是加入其中。<br>

从此后就使用从头构建LFS系统。后来为了加入包管理器，移植了Gentoo的Portage系统, 但用着很不爽，因为Portage系统庞大很难掌控，于是产生了想做一个包管理器的想法。在LinuxSir发了几篇帖子，请参见:<br>
[<a href='http://www.linuxsir.org/bbs/thread356491.html'>http://www.linuxsir.org/bbs/thread356491.html</a> 一个获取文件信息的C程序］<br>
<a href='http://www.linuxsir.org/bbs/thread356497.html'>标题: Help: 分解文件名的Bash函数</a> <br>
<a href='http://www.linuxsir.org/bbs/thread366340.html'>LFS下滚动升级(rolling upgrade)怎么实现</a><br>


热心的pinkme005, tomgrean, gasboy, lastart, 深空兄等坛友给了回复。2002年4月开始在google code 创建了开源项目弦歌Linux, 写gpkg的代码，并构建了工作用的弦歌Linux系统。