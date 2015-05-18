# 1. 构造调试环境 #

> 由于bochs内建调试功能, 且支持gdb, 用它调试内核会很方便.

## 1.1 构建磁盘镜像 ##

```
dd if=/dev/zero of=hd0.img count=$((63*16*100))
```

> 用这个命令可以构建一个50MB左右的磁盘镜像, 输出结果如下:

```
100800+0 records in
100800+0 records out
51609600 bytes (52 MB) copied, 0.734578 s, 70.3 MB/s
```

> 注意count必须为63\*16的倍数, 否则bochs识别硬盘会有问题.

## 1.2 挂载磁盘镜像 ##

```
losetup /dev/loop0 hd0.img
```

> 这个命令可以将文件绑定到一个loop设备. 如果/dev/loop0不存在, 可以尝试 modprobe loop.

> 然后进行设备初始化:

```
cfdisk -s 63 -h 16 /dev/loop0
```

> 只创建一个主分区就可以. 写入后, 用命令fdisk检查结果:

```
fdisk -lu /dev/loop0


Disk /dev/loop0: 51 MB, 51609600 bytes
16 heads, 63 sectors/track, 100 cylinders, total 100800 sectors
Units = sectors of 1 * 512 = 512 bytes
Disk identifier: 0x00000000

Device Boot      Start         End      Blocks   Id  System
/dev/loop0p1      63           100799   50368+  83  Linux
```

> 将分区1挂载到/dev/loop1.

```
losetup /dev/loop1 hd0.img -o $((63*512))
```

> 格式化/dev/loop1为ext3格式.

```
mkfs.ext3 /dev/loop1
```

> 在mnt下创建img目录, 做以后维护用.

```
mkdir -p /mnt/img
```

> 将loop1挂载到/mnt/img

```
mount /dev/loop1 /mnt/img/
```

> 安装引导程序. 因为我狂热倾向于模块化架构, 所以选择GRUB2.
> 本文以grub-1.97~beta3为示例, 读者可自行安装其他的引导系统如lilo.

```
mkdir /mnt/img/boot
cp -r /usr/lib/grub/i386-pc/ /mnt/img/boot/grub
```

> 生成一个core.img, biosdisk负责读取磁盘, part\_msdos负责处理MBR, ext2负责读取ext3分区.

```
cd /mnt/img/boot/grub/
grub-mkimage -o core.img biosdisk part_msdos ext2[/CODE]
```


```
ls -lh core.img
-rw-r--r-- 1 root root 25K Sep 21 14:28 core.img
```

> 只有区区的25K.. 里面甚至还包含一个小的应急shell, 不过作用不大.

> 安装grub2到(hd0), 根目录在(hd0,1)

```
echo "(hd0)   /dev/loop0" > ./device.map
grub-setup -m ./device.map -d /mnt/img/boot/grub/ -r '(hd0,1)' '(hd0)'
```

> 检查一下安装成果:

```
hexdump -C /dev/loop0 | less
```

> 如果你能看到:

```
00000180  7d e8 2e 00 cd 18 eb fe  47 52 55 42 20 00 47 65  |}.......GRUB .Ge|
00000190  6f 6d 00 48 61 72 64 20  44 69 73 6b 00 52 65 61  |om.Hard Disk.Rea|
```

> 就说明安装成功.

> 清理一下.

```
cd 
umount /mnt/img
losetup -d /dev/loop1
losetup -d /dev/loop0
```

## 1.3 启动测试 ##

> 给上面的hd0.img配一个bochsrc文件, 可以拿bochs示例dlxlinux的配置文件来改, 只需将硬盘换为:

```
ata0-master: type=disk, path="hd0.img", cylinders=100, heads=16, spt=63
```

> 然后启动Bochs, 顺利的话, 你能看到传说中的grub2 shell:

![http://xiangelinux.googlecode.com/files/pic-1.3.png](http://xiangelinux.googlecode.com/files/pic-1.3.png)


# 2. 从启动到保护模式 #

> 为我们的bochs虚拟机编译一个内核. 不必太复杂, 目前我们只关心启动部分.

## 2.1 安装内核 ##

> 退出bochs, 挂载hd0.img:

```
mount hd0.img /mnt/img/ -o loop,offset=$((63*512))
```

> 拷贝bzImage.

```
cp /usr/src/linux/arch/i386/boot/bzImage /mnt/img/boot/vmlinuz-2.6.31
```

> 这两个步骤可以放到内核的Makefile中, 以后每次编译完成后, 自动更新到hd0.img里.

> 然后将下列配置写到/mnt/img/boot/grub/grub.cfg

```

# Timeout for menu
set timeout=10

# Set default boot entry as Entry 0
set default=0

# Entry 0 - Load Linux kernel
menuentry "Linux-2.6.31" {
	set root=(hd0,1)
	linux /boot/vmlinuz-2.6.31 root=/dev/hda1
}
```

> 卸载/mnt/img, 启动测试一下, 顺利的话, 你能看到一个panic:

![http://xiangelinux.googlecode.com/files/pic-2.1.png](http://xiangelinux.googlecode.com/files/pic-2.1.png)


## 2.2 从loader到内核 ##

> 先测试一下bochs的调试能力. bochs以调试模式启动后, 会自动停在BIOS中Reset代码, 地址为0xFFFFFFF0. 我们在bochs console里输入:

```
pb 0x00007c00
```

> 在0x7c00设置一个断点, 这正式GRUB2 MBR代码的入口点. GRUB2负责加载自己的core.img, 在core.img里读取hd0, 解析MBR里的分区信息, 并利用ext2模块找到配置文件, 显示主界面, 并加载用户的选择内核文件(vmlinuz-xxx).


> loader和内核之间的协议, 在内核源码Document/x86/boot.txt里有详尽的描述.

> 内核文件包括实模式代码和保护模式代码. loader读取内核文件的头部信息, 从而得知实模式和保护模式代码的大小. 保护模式部分被加载到0x100000(1MB), 实模式启动代码和数据堆栈段位置可以重定位在0x10000开始的低端内存的任何位置.

## 2.3 内核是怎样链成的 ##

> 首先编译内核的保护模式代码, 生成源码根目录的vmlinux. 这是一个elf格式的文件, 可以用readelf查看.

```
readelf -a /usr/src/linux/vmlinux | less
```

> 这个文件包含所有的符合信息, 容量巨大.

```
ls -lh /usr/src/linux/vmlinux
-rwxr-xr-x 1 root root 4.8M Sep 10 17:28 /usr/src/linux/vmlinux
```

> 文件内容会在后面章节细述.

> 接着将vmlinux洗净压缩. 这个工作在 arch/x86/boot/compressed目录进行. 读Makefile可以看到先进行objcopy操作, 在compressed目录下生成vmlinux.bin.

```
ls -lh /usr/src/linux/arch/x86/boot/compressed/vmlinux.bin
-rwxr-xr-x 1 root root 3.7M Sep 10 17:28 /usr/src/linux/arch/x86/boot/compressed/vmlinux.bin
```

> 因为去掉了调试信息, 文件稍小了一些.

> 接着根据用户的选择进行压缩, 目前支持gz, bz2, lzma三种方式. 我用默认方式, 生成vimlinux.bin.gz.

```
#ls -lh /usr/src/linux/arch/x86/boot/compressed/vmlinux.bin.gz
-rw-r--r-- 1 root root 1.9M Sep 10 17:28 /usr/src/linux/arch/x86/boot/compressed/vmlinux.bin.gz
```

> 可以看到也有近50%的压缩率.

> 紧接着加入自解压部分, 生成新的vmlinux.

```
ls -lh /usr/src/linux/arch/x86/boot/compressed/vmlinux
-rwxr-xr-x 1 root root 2.0M Sep 10 17:28 /usr/src/linux/arch/x86/boot/compressed/vmlinux
```

> 这也是一个elf文件, 可以用readelf查看.


> 接着工作转移到boot目录, 对上面的vmlinux进行strip操作, 生成二进制格式的vmlinux.bin. 这里面包括全部的包含模式代码, 启动时第一条语句会被加载到0x100000(1MB).

> 然后编译实模式代码, 包括已无用的引导扇区和setup部分, 生成setup.elf. 然后进行strip, 生成setup.bin.

> 最后, 利用内核工具"build", 将setup.bin和vmlinux.bin拷贝在一起, 并填上必要的信息如setup部分的大小等, grub等引导程序可以使用的bzImage诞生了.


## 2.4 内核实模式代码 ##

> 实模式代码位于arch/x86/boot. 记得Linux2.6.18之前所有的代码都用汇编写成, 2.6.18之后大部分替换成了C.

> 入口点在header.S文件, 即包含了无用的"引导扇区代码", 也包含了引导程序能识别的头部信息. 第一条可执行语句在偏移0x200的位置(跳过引导扇区), 执行必要的初始化操作, 然后将控制权交给C程序, 即main.c里的main()函数.

> 有了main()函数, 接下来的过程就像读文档一样方便了:) 利用实模式的优势, 调用BIOS执行必要的初始化和参数获取, 并将结果存到结构体boot\_params.

> 在main()的最后调用go\_to\_protected\_mode(). 这个函数位于pm.c, 它打开A20, 初始化协处理器(如果有), 关掉所有中断, 设置空的IDT和最基本的GDT, 接着调用protected\_mode\_jump跳转到保护模式的入口代码0x100000. 这个函数定义在pmjump.S. 注意跳转的时候, boot\_params被放在esi寄存器.

## 2.5 保护模式:自解压过程 ##

> 内核保护模式的入口在arch/x86/boot/compressed/head\_32.S (32位架构). 我们利用bochs的调试功能, 跟着走一遍:

```
pb 0x100000
c
```

> 接着grub会运行, 选择我们编译的kernel, 等实模式代码运行完毕, 就会断在保护模式的入口. 反汇编看一下:

```
u /16 0x100000
```

> 可以看到反汇编代码和head\_32.S一样. (如果不一样.. 报告一起UFO目击事件吧)

> 在这里, esi还是指向来自与实模式的boot\_params. 接下来的任务, 就是拷贝和解压. 目的地在0x1000000(16MB). 解压部分是C语言函数.

> 最后跳转到0x1000000.


# 3. 内核启动 #

## 3.1 平台相关的初始化 ##

> 我们将断点设在内核的入口点 0x1000000(16MB), 继续执行, 内核会自己解压, 并停在入口点.

> 执行反汇编操作:

```
u /16 0x1000000
```

> 对应的汇编文件在 arch/x86/kernel/head\_32.S. 这个是真正的内核入口点. 在这里初始化页表, 并启动分页机制. 打开SMP的化, 进行必要的CPU初始化. 然后初始化IDT, 检查CPU类型, 最后跳转到C语言的i386\_start\_kernel(). 这个函数位于head32.c. 它保留一些内存并做标记后, 跳转到平台无关的start\_kernel(), 位于init/main.c.

<待细化, 欢迎补充>


## 3.2 平台无关的初始化 ##
> start\_kernel()是一个高层函数, 与架构无关, 但指挥架构相关的代码完成剩余的初始化部分.

> <待续>

> <若有异议, 欢迎讨论一起研究>
> 个人学习笔记, 如需转载, 请注明出处: 张力辉 <swordhui@hotmail.com>