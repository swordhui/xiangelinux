### 0. 简介 ###
tools目录提供了构建弦歌系统最基本的工具链。作用就像人体的胎盘... 是让新弦歌Linux系统诞生的必要工具。<br>
tools目录收集的工具大部分来自于LFS手册，然后加入了git, <a href='xgpkg.md'>gpkg</a>, 和wget.<br>
用户可以选择使用预编译的<a href='http://xiangelinux.googlecode.com/files/tools-bin-i686-0.10.7.tar.bz2'>二进制的工具链</a><br>
当然也可以按下列步骤自行编译构建。<br>
<br>
<h3>1.将tmpfs挂接到工作目录</h3>
<pre><code>mkdir /dev/shm/xgtmp<br>
chmod 1777 /dev/shm/xgtmp<br>
mkdir /mnt/xg/tmp<br>
chmod 1777 /mnt/xg/tmp<br>
mount --bind /dev/shm/xgtmp /mnt/xg/tmp<br>
</code></pre>


<h3>2.下载编译脚本</h3>

<a href='http://xiangelinux.googlecode.com/files/xgstage0-0.10.7.tar.bz2'>xgstage0-0.10.7.tar.bz2</a> 7.8KB<br>
<ol><li>在工作目录解压, 本文以/mnt/xg为例。得到xgstage0-0.10.7目录。进入后执行./download，脚本会自动下载所需软件包源码<br>
</li><li>如果失败，可以手动下载源码到工作目录的var/xiange/sources子目录, 如 /mnt/xg/var/xiange/sources/abc.tar.bz2. 再次运行./download, 会跳过已下载的软件包。<br>
</li><li>下载成功后，执行./build, 构建工具链。<br>
</li><li>完成后，会得到一个完整的工具链，在/mnt/xg/tools目录下。里面有记录文件installed.log, 记录了每个软件包的名称和安装时间。<br>
</li><li>其实..也不必完全等代./download下载完毕再执行./build. 下载完gcc后就可以执行了，然后边下载边编译，节省时间。<br>
</li><li>彻底编译完成后, 可以删除xgstage0-0.10.7脚本目录</li></ol>

<h3>3. ToDo list</h3>
目前的脚本只在i686平台测试成功。<br>
理论上，脚本稍作修改，可以在任何支持gcc的平台上编译.<br>
wall_copper 同学已经在x86_64上编译成功，并做了部分修改，代码和二进制文件尚未上传，有需要请与他联系。<br>
<br>
<ul><li>更新各工具的版本<br>
</li><li>在更多的平台上编译，并生成二进制包。