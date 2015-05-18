注: 本文转自ChinaUnix 作者为XPL.

本文针对arm linux, 从kernel的第一条指令开始分析,一直分析到进入 start\_kernel()函数.
我们当前以linux-2.6.19内核版本作为范例来分析,本文中所有的代码,前面都会加上行号以便于和源码进行对照,
例:
在文件init/main.c中:
00478: asmlinkage void init start\_kernel(void)
前面的"00478:" 表示478行,冒号后面的内容就是源码了.

在分析代码的过程中,我们使用缩进来表示各个代码的调用层次.

由于启动部分有一些代码是平台特定的,虽然大部分的平台所实现的功能都比较类似,但是为了更好的对code进行说明,对于平台相关的代码,我们选择 at91(ARM926EJS)平台进行分析.

另外,本文是以uncompressed kernel开始讲解的.对于内核解压缩部分的code,在 arch/arm/boot/compressed中,本文不做讨论.

# 一. 启动条件 #
> 通常从系统上电到执行到linux kenel这部分的任务是由boot loader来完成.
> 关于boot loader的内容,本文就不做过多介绍.
> 这里只讨论进入到linux kernel的时候的一些限制条件,这一般是boot loader在最后跳转到kernel之前要完成的:
  * 1. CPU必须处于SVC(supervisor)模式,并且IRQ和FIQ中断都是禁止的;
  * 2. MMU(内存管理单元)必须是关闭的, 此时虚拟地址对物理地址;
  * 3. 数据cache(Data cache)必须是关闭的
  * 4. 指令cache(Instruction cache)可以是打开的,也可以是关闭的,这个没有强制要求;
  * 5. CPU 通用寄存器0 ([r0](https://code.google.com/p/xiangelinux/source/detail?r=0))必须是 0;
  * 6. CPU 通用寄存器1 ([r1](https://code.google.com/p/xiangelinux/source/detail?r=1))必须是 ARM Linux machine type (关于machine type, 我们后面会有讲解)
  * 7. CPU 通用寄存器2 ([r2](https://code.google.com/p/xiangelinux/source/detail?r=2)) 必须是 kernel parameter list 的物理地址(parameter list 是由boot loader传递给kernel,用来描述设备信息属性的列表,详细内容可参考"Booting ARM Linux"文档).

# 二. starting kernel #

首先，我们先对几个重要的宏进行说明(我们针对有MMU的情况)：


|宏|位置|默认值|说明|
|:--|:-----|:--------|:-----|
|KERNEL\_RAM\_ADDR|arch/arm/kernel/head.S +26|0xc0008000|kernel在RAM中的的虚拟地址|
|PAGE\_OFFSET|include/asm-arm/memeory.h +50|0xc0000000|内核空间的起始虚拟地址|
|TEXT\_OFFSET|arch/arm/Makefile +137|0x00008000|内核相对于存储空间的偏移|
|TEXTADDR|arch/arm/kernel/head.S +49|0xc0008000|kernel的起始虚拟地址|
|PHYS\_OFFSET|include/asm-arm/arch-xxx/memory.h|平台相关|RAM的起始物理地址|


> 内核的入口是stext,这是在arch/arm/kernel/vmlinux.lds.S中定义的:

```
        00011: ENTRY(stext)
```

> 对于vmlinux.lds.S,这是ld script文件,此文件的格式和汇编及C程序都不同,本文不对ld script作过多的介绍,只对内核中用到的内容进行讲解,关于ld的详细内容可以参考ld.info
> 这里的ENTRY(stext) 表示程序的入口是在符号stext.
> 而符号stext是在arch/arm/kernel/head.S中定义的:
> 下面我们将arm linux boot的主要代码列出来进行一个概括的介绍,然后,我们会逐个的进行详细的讲解.

> 在arch/arm/kernel/head.S中 72 - 94 行,是arm linux boot的主代码:
```

00072: ENTRY(stext)                                                        
00073:         msr        cpsr_c, #PSR_F_BIT | PSR_I_BIT | SVC_MODE @ ensure svc mode
00074:                                                 @ and irqs disabled        
00075:         mrc        p15, 0, r9, c0, c0                @ get processor id         
00076:         bl        __lookup_processor_type                @ r5=procinfo r9=cpuid     
00077:         movs        r10, r5                                @ invalid processor (r5=0)?
00078:         beq        __error_p                        @ yes, error 'p'           
00079:         bl        __lookup_machine_type                @ r5=machinfo              
00080:         movs        r8, r5                                @ invalid machine (r5=0)?  
00081:         beq        __error_a                        @ yes, error 'a'           
00082:         bl        __create_page_tables                                       
00083:                                                                     
00084:         /*                                                                 
00085:          * The following calls CPU specific code in a position independent
00086:          * manner.  See arch/arm/mm/proc-*.S for details.  r10 = base of   
00087:          * xxx_proc_info structure selected by __lookup_machine_type      
00088:          * above.  On return, the CPU will be ready for the MMU to be      
00089:          * turned on, and r0 will hold the CPU control register value.     
00090:          */                                                               
00091:         ldr        r13, __switch_data                @ address to jump to after
00092:                                                 @ mmu has been enabled     
00093:         adr        lr, __enable_mmu                @ return (PIC) address     
00094:         add        pc, r10, #PROCINFO_INITFUNC                                
```

其中,73行是确保kernel运行在SVC模式下,并且IRQ和FIRQ中断已经关闭,这样做是很谨慎的.

arm linux boot的主线可以概括为以下几个步骤:
  * 1. 确定 processor type                        (75 - 78行)
  * 2. 确定 machine type                        (79 - 81行)
  * 3. 创建页表                                (82行)
  * 4. 调用平台特定的cpu\_flush函数        (在struct proc\_info\_list中)        (94 行)
  * 5. 开启mmu                                (93行)
  * 6. 切换数据                                 (91行)

> 最终跳转到start\_kernel                        (在switch\_data的结束的时候,调用了 b        start\_kernel)

下面,我们按照这个主线,逐步的分析Code.

## 1. 确定 processor type ##


> arch/arm/kernel/head.S中:

```
00075:         mrc        p15, 0, r9, c0, c0                @ get processor id         
00076:         bl        __lookup_processor_type                @ r5=procinfo r9=cpuid     
00077:         movs        r10, r5                                @ invalid processor (r5=0)?
00078:         beq        __error_p                        @ yes, error 'p'           
```

75行: 通过cp15协处理器的c0寄存器来获得processor id的指令. 关于cp15的详细内容可参考相关的arm手册

76行: 跳转到lookup\_processor\_type.在lookup\_processor\_type中,会把processor type 存储在r5中

77,78行: 判断r5中的processor type是否是0,如果是0,说明是无效的processor type,跳转到error\_p(出错)

lookup\_processor\_type 函数主要是根据从cpu中获得的processor id和系统中的proc\_info进行匹配,将匹配到的proc\_info\_list的基地址存到r5中, 0表示没有找到对应的processor type.

下面我们分析lookup\_processor\_type函数
> arch/arm/kernel/head-common.S中:

```
00145:         .type        __lookup_processor_type, %function
00146: __lookup_processor_type:
00147:         adr        r3, 3f
00148:         ldmda        r3, {r5 - r7}
00149:         sub        r3, r3, r7                        @ get offset between virt&phys
00150:         add        r5, r5, r3                        @ convert virt addresses to
00151:         add        r6, r6, r3                        @ physical address space
00152: 1:        ldmia        r5, {r3, r4}                        @ value, mask
00153:         and        r4, r4, r9                        @ mask wanted bits
00154:         teq        r3, r4
00155:         beq        2f
00156:         add        r5, r5, #PROC_INFO_SZ                @ sizeof(proc_info_list)
00157:         cmp        r5, r6
00158:         blo        1b
00159:         mov        r5, #0                                @ unknown processor
00160: 2:        mov        pc, lr
00161:
00162: /*
00163:  * This provides a C-API version of the above function.
00164:  */
00165: ENTRY(lookup_processor_type)
00166:         stmfd        sp!, {r4 - r7, r9, lr}
00167:         mov        r9, r0
00168:         bl        __lookup_processor_type
00169:         mov        r0, r5
00170:         ldmfd        sp!, {r4 - r7, r9, pc}
00171:
00172: /*
00173:  * Look in include/asm-arm/procinfo.h and arch/arm/kernel/arch.[ch] for
00174:  * more information about the __proc_info and __arch_info structures.
00175:  */
00176:         .long        __proc_info_begin
00177:         .long        __proc_info_end
00178: 3:        .long        .
00179:         .long        __arch_info_begin
00180:         .long        __arch_info_end
```


145, 146行是函数定义

147行: 取地址指令,这里的3f是向前symbol名称是3的位置,即第178行,将该地址存入r3.
> 这里需要注意的是,adr指令取址,获得的是基于pc的一个地址,要格外注意,这个地址是3f处的"运行时地址",由于此时MMU还没有打开,也可以理解成物理地址(实地址).(详细内容可参考arm指令手册)

148行: 因为r3中的地址是178行的位置的地址,因而执行完后:
> r5存的是176行符号 proc\_info\_begin的地址;
> r6存的是177行符号 proc\_info\_end的地址;
> r7存的是3f处的地址.
> 这里需要注意链接地址和运行时地址的区别. r3存储的是运行时地址(物理地址),而r7中存储的是链接地址(虚拟地址).

> proc\_info\_begin和proc\_info\_end是在arch/arm/kernel/vmlinux.lds.S中:

```
        00031:                __proc_info_begin = .;
        00032:                        *(.proc.info.init)
        00033:                __proc_info_end = .;
```

> 这里是声明了两个变量:proc\_info\_begin 和 proc\_info\_end,其中等号后面的"."是location counter(详细内容请参考ld.info)
> 这三行的意思是: proc\_info\_begin 的位置上,放置所有文件中的 ".proc.info.init" 段的内容,然后紧接着是 proc\_info\_end 的位置.

> kernel 使用struct proc\_info\_list来描述processor type.
> > 在 include/asm-arm/procinfo.h 中:

```
        00029: struct proc_info_list {
        00030:         unsigned int                cpu_val;
        00031:         unsigned int                cpu_mask;
        00032:         unsigned long                __cpu_mm_mmu_flags;        /* used by head.S */
        00033:         unsigned long                __cpu_io_mmu_flags;        /* used by head.S */
        00034:         unsigned long                __cpu_flush;                /* used by head.S */
        00035:         const char                *arch_name;
        00036:         const char                *elf_name;
        00037:         unsigned int                elf_hwcap;
        00038:         const char                *cpu_name;
        00039:         struct processor        *proc;
        00040:         struct cpu_tlb_fns        *tlb;
        00041:         struct cpu_user_fns        *user;
        00042:         struct cpu_cache_fns        *cache;
        00043: };
        
        我们当前以at91为例,其processor是926的.
                在arch/arm/mm/proc-arm926.S 中:
        00464:         .section ".proc.info.init", #alloc, #execinstr
        00465:
        00466:         .type        __arm926_proc_info,#object
        00467: __arm926_proc_info:
        00468:         .long        0x41069260                        @ ARM926EJ-S (v5TEJ)
        00469:         .long        0xff0ffff0
        00470:         .long   PMD_TYPE_SECT | \
        00471:                 PMD_SECT_BUFFERABLE | \
        00472:                 PMD_SECT_CACHEABLE | \
        00473:                 PMD_BIT4 | \
        00474:                 PMD_SECT_AP_WRITE | \
        00475:                 PMD_SECT_AP_READ
        00476:         .long   PMD_TYPE_SECT | \
        00477:                 PMD_BIT4 | \
        00478:                 PMD_SECT_AP_WRITE | \
        00479:                 PMD_SECT_AP_READ
        00480:         b        __arm926_setup
        00481:         .long        cpu_arch_name
        00482:         .long        cpu_elf_name
        00483:         .long        HWCAP_SWP|HWCAP_HALF|HWCAP_THUMB|HWCAP_FAST_MULT|HWCAP_VFP|HWCAP_EDSP|HWCAP_JAVA
        00484:         .long        cpu_arm926_name
        00485:         .long        arm926_processor_functions
        00486:         .long        v4wbi_tlb_fns
        00487:         .long        v4wb_user_fns
        00488:         .long        arm926_cache_fns
        00489:         .size        __arm926_proc_info, . - __arm926_proc_info
```


> 从464行,我们可以看到 arm926\_proc\_info 被放到了".proc.info.init"段中.
> 对照struct proc\_info\_list,我们可以看到 cpu\_flush的定义是在480行,即arm926\_setup.(我们将在"4. 调用平台特定的cpu\_flush函数"一节中详细分析这部分的内容.)

从以上的内容我们可以看出: r5中的proc\_info\_begin是proc\_info\_list的起始地址, r6中的proc\_info\_end是proc\_info\_list的结束地址.

149行: 从上面的分析我们可以知道r3中存储的是3f处的物理地址,而r7存储的是3f处的虚拟地址,这一行是计算当前程序运行的物理地址和虚拟地址的差值,将其保存到r3中.

150行: 将r5存储的虚拟地址(proc\_info\_begin)转换成物理地址

151行: 将r6存储的虚拟地址(proc\_info\_end)转换成物理地址

152行: 对照struct proc\_info\_list,可以得知,这句是将当前proc\_info的cpu\_val和cpu\_mask分别存r3, r4中

153行: r9中存储了processor id(arch/arm/kernel/head.S中的75行),与r4的cpu\_mask进行逻辑与操作,得到我们需要的值

154行: 将153行中得到的值与r3中的cpu\_val进行比较

155行: 如果相等,说明我们找到了对应的processor type,跳到160行,返回

156行: (如果不相等) , 将r5指向下一个proc\_info,

157行: 和r6比较,检查是否到了proc\_info\_end.

158行: 如果没有到proc\_info\_end,表明还有proc\_info配置,返回152行继续查找

159行: 执行到这里,说明所有的proc\_info都匹配过了,但是没有找到匹配的,将r5设置成0(unknown processor)

160行: 返回

## 2. 确定 machine type ##

> arch/arm/kernel/head.S中:

```
00079:         bl        __lookup_machine_type                @ r5=machinfo              
00080:         movs        r8, r5                                @ invalid machine (r5=0)?  
00081:         beq        __error_a                        @ yes, error 'a'  
```

79行: 跳转到lookup\_machine\_type函数,在lookup\_machine\_type中,会把struct machine\_desc的基地址(machine type)存储在r5中
80,81行: 将r5中的 machine\_desc的基地址存储到r8中,并判断r5是否是0,如果是0,说明是无效的machine type,跳转到error\_a(出错)

lookup\_machine\_type 函数
下面我们分析lookup\_machine\_type 函数:

> arch/arm/kernel/head-common.S中:

```
00176:         .long        __proc_info_begin
00177:         .long        __proc_info_end
00178: 3:        .long        .
00179:         .long        __arch_info_begin
00180:         .long        __arch_info_end
00181:
00182: /*
00183:  * Lookup machine architecture in the linker-build list of architectures.
00184:  * Note that we can't use the absolute addresses for the __arch_info
00185:  * lists since we aren't running with the MMU on (and therefore, we are
00186:  * not in the correct address space).  We have to calculate the offset.
00187:  *
00188:  *  r1 = machine architecture number
00189:  * Returns:
00190:  *  r3, r4, r6 corrupted
00191:  *  r5 = mach_info pointer in physical address space
00192:  */       
00193:         .type        __lookup_machine_type, %function
00194: __lookup_machine_type:
00195:         adr        r3, 3b
00196:         ldmia        r3, {r4, r5, r6}
00197:         sub        r3, r3, r4                        @ get offset between virt&phys
00198:         add        r5, r5, r3                        @ convert virt addresses to
00199:         add        r6, r6, r3                        @ physical address space
00200: 1:        ldr        r3, [r5, #MACHINFO_TYPE]        @ get machine type
00201:         teq        r3, r1                                @ matches loader number?
00202:         beq        2f                                @ found
00203:         add        r5, r5, #SIZEOF_MACHINE_DESC        @ next machine_desc
00204:         cmp        r5, r6
00205:         blo        1b
00206:         mov        r5, #0                                @ unknown machine
00207: 2:        mov        pc, lr
```

193, 194行: 函数声明

195行: 取地址指令,这里的3b是向后symbol名称是3的位置,即第178行,将该地址存入r3.
> 和上面我们对lookup\_processor\_type 函数的分析相同,r3中存放的是3b处物理地址.

196行:
  * r3是3b处的地址,因而执行完后:
  * r4存的是 3b处的地址
  * r5存的是arch\_info\_begin 的地址
  * r6存的是arch\_info\_end 的地址

> arch\_info\_begin 和 arch\_info\_end是在 arch/arm/kernel/vmlinux.lds.S中:

```
        00034:                __arch_info_begin = .;
        00035:                        *(.arch.info.init)
        00036:                __arch_info_end = .;
```


> 这里是声明了两个变量:arch\_info\_begin 和 arch\_info\_end,其中等号后面的"."是location counter(详细内容请参考ld.info)
> 这三行的意思是: arch\_info\_begin 的位置上,放置所有文件中的 ".arch.info.init" 段的内容,然后紧接着是 arch\_info\_end 的位置.

> kernel 使用struct machine\_desc 来描述 machine type.
> 在 include/asm-arm/mach/arch.h 中:

```
        00017: struct machine_desc {
        00018:         /*
        00019:          * Note! The first four elements are used
        00020:          * by assembler code in head-armv.S
        00021:          */
        00022:         unsigned int                nr;                /* architecture number        */
        00023:         unsigned int                phys_io;        /* start of physical io        */
        00024:         unsigned int                io_pg_offst;        /* byte offset for io
        00025:                                                  * page tabe entry        */
        00026:
        00027:         const char                *name;                /* architecture name        */
        00028:         unsigned long                boot_params;        /* tagged list                */
        00029:
        00030:         unsigned int                video_start;        /* start of video RAM        */
        00031:         unsigned int                video_end;        /* end of video RAM        */
        00032:
        00033:         unsigned int                reserve_lp0 :1;        /* never has lp0        */
        00034:         unsigned int                reserve_lp1 :1;        /* never has lp1        */
        00035:         unsigned int                reserve_lp2 :1;        /* never has lp2        */
        00036:         unsigned int                soft_reboot :1;        /* soft reboot                */
        00037:         void                        (*fixup)(struct machine_desc *,
        00038:                                          struct tag *, char **,
        00039:                                          struct meminfo *);
        00040:         void                        (*map_io)(void);/* IO mapping function        */
        00041:         void                        (*init_irq)(void);
        00042:         struct sys_timer        *timer;                /* system tick timer        */
        00043:         void                        (*init_machine)(void);
        00044: };
        00045:
        00046: /*
        00047:  * Set of macros to define architecture features.  This is built into
        00048:  * a table by the linker.
        00049:  */
        00050: #define MACHINE_START(_type,_name)                        \
        00051: static const struct machine_desc __mach_desc_##_type        \
        00052:  __attribute_used__                                        \
        00053:  __attribute__((__section__(".arch.info.init")) = {        \
        00054:         .nr                = MACH_TYPE_##_type,                \
        00055:         .name                = _name,
        00056:
        00057: #define MACHINE_END                                \
        00058: };        
        
        内核中,一般使用宏MACHINE_START来定义machine type.
        对于at91, 在 arch/arm/mach-at91rm9200/board-ek.c 中:
        00137: MACHINE_START(AT91RM9200EK, "Atmel AT91RM9200-EK"
        00138:         /* Maintainer: SAN People/Atmel */
        00139:         .phys_io        = AT91_BASE_SYS,
        00140:         .io_pg_offst        = (AT91_VA_BASE_SYS >> 1 & 0xfffc,
        00141:         .boot_params        = AT91_SDRAM_BASE + 0x100,
        00142:         .timer                = &at91rm9200_timer,
        00143:         .map_io                = ek_map_io,
        00144:         .init_irq        = ek_init_irq,
        00145:         .init_machine        = ek_board_init,
        00146: MACHINE_END
```


197行: r3中存储的是3b处的物理地址,而r4中存储的是3b处的虚拟地址,这里计算处物理地址和虚拟地址的差值,保存到r3中

198行: 将r5存储的虚拟地址(arch\_info\_begin)转换成物理地址

199行: 将r6存储的虚拟地址(arch\_info\_end)转换成物理地址

200行: MACHINFO\_TYPE 在 arch/arm/kernel/asm-offset.c 101行定义, 这里是取 struct machine\_desc中的nr(architecture number) 到r3中

201行: 将r3中取到的machine type 和 r1中的 machine type(见前面的"启动条件"进行比较

202行: 如果相同,说明找到了对应的machine type,跳转到207行的2f处,此时r5中存储了对应的struct machine\_desc的基地址

203行: (不相同), 取下一个machine\_desc的地址

204行: 和r6进行比较,检查是否到了arch\_info\_end.

205行: 如果不相同,说明还有machine\_desc,返回200行继续查找.

206行: 执行到这里,说明所有的machind\_desc都查找完了,并且没有找到匹配的, 将r5设置成0(unknown machine).

207行: 返回

## 3. 创建页表 ##

通过前面的两步,我们已经确定了processor type 和 machine type.
此时,一些特定寄存器的值如下所示:

```
r8 = machine info       (struct machine_desc的基地址)
r9 = cpu id             (通过cp15协处理器获得的cpu id)
r10 = procinfo          (struct proc_info_list的基地址)
```

创建页表是通过函数 create\_page\_tables 来实现的.

这里,我们使用的是arm的L1主页表,L1主页表也称为段页表(section page table)
L1 主页表将4 GB 的地址空间分成若干个1 MB的段(section),因此L1页表包含4096个页表项(section entry). 每个页表项是32 bits(4 bytes)
因而L1主页表占用 4096 **4 = 16k的内存空间.**

> 对于ARM926,其L1 section entry的格式为可参考arm926EJS TRM):

![http://xiangelinux.googlecode.com/files/armmmu.gif](http://xiangelinux.googlecode.com/files/armmmu.gif)

下面我们来分析 create\_page\_tables 函数:

> 在 arch/arm/kernel/head.S 中:

```
00206:         .type        __create_page_tables, %function
00207: __create_page_tables:
00208:         pgtbl        r4                                @ page table address
00209:
00210:         /*
00211:          * Clear the 16K level 1 swapper page table
00212:          */
00213:         mov        r0, r4
00214:         mov        r3, #0
00215:         add        r6, r0, #0x4000
00216: 1:        str        r3, [r0], #4
00217:         str        r3, [r0], #4
00218:         str        r3, [r0], #4
00219:         str        r3, [r0], #4
00220:         teq        r0, r6
00221:         bne        1b
00222:
00223:         ldr        r7, [r10, #PROCINFO_MM_MMUFLAGS] @ mm_mmuflags
00224:
00225:         /*
00226:          * Create identity mapping for first MB of kernel to
00227:          * cater for the MMU enable.  This identity mapping
00228:          * will be removed by paging_init().  We use our current program
00229:          * counter to determine corresponding section base address.
00230:          */
00231:         mov        r6, pc, lsr #20                        @ start of kernel section
00232:         orr        r3, r7, r6, lsl #20                @ flags + kernel base
00233:         str        r3, [r4, r6, lsl #2]                @ identity mapping
00234:
00235:         /*
00236:          * Now setup the pagetables for our kernel direct
00237:          * mapped region.
00238:          */
00239:         add        r0, r4,  #(TEXTADDR & 0xff000000) >> 18        @ start of kernel
00240:         str        r3, [r0, #(TEXTADDR & 0x00f00000) >> 18]!
00241:
00242:         ldr        r6, =(_end - PAGE_OFFSET - 1)        @ r6 = number of sections
00243:         mov        r6, r6, lsr #20                        @ needed for kernel minus 1
00244:
00245: 1:        add        r3, r3, #1 << 20
00246:         str        r3, [r0, #4]!
00247:         subs        r6, r6, #1
00248:         bgt        1b
00249:
00250:         /*
00251:          * Then map first 1MB of ram in case it contains our boot params.
00252:          */
00253:         add        r0, r4, #PAGE_OFFSET >> 18
00254:         orr        r6, r7, #PHYS_OFFSET
00255:         str        r6, [r0]
        
        ...
        
00314:        mov        pc, lr
00315:        .ltorg
```

206, 207行: 函数声明

208行: 通过宏 pgtbl 将r4设置成页表的基地址(物理地址)
> 宏pgtbl 在 arch/arm/kernel/head.S 中:

```
        00042:        .macro        pgtbl, rd
        00043:        ldr        \rd, =(__virt_to_phys(KERNEL_RAM_ADDR - 0x4000))
        00044:        .endm

        可以看到,页表是位于 KERNEL_RAM_ADDR 下面 16k 的位置
        宏 __virt_to_phys 是在incude/asm-arm/memory.h 中:
        
        00125: #ifndef __virt_to_phys
        00126: #define __virt_to_phys(x)        ((x) - PAGE_OFFSET + PHYS_OFFSET)
        00127: #define __phys_to_virt(x)        ((x) - PHYS_OFFSET + PAGE_OFFSET)
        00128: #endif   
```


下面从213行 - 221行, 是将这16k 的页表清0.

213行: [r0](https://code.google.com/p/xiangelinux/source/detail?r=0) = [r4](https://code.google.com/p/xiangelinux/source/detail?r=4), 将页表基地址存在r0中

214行: 将 [r3](https://code.google.com/p/xiangelinux/source/detail?r=3) 置成0

215行: [r6](https://code.google.com/p/xiangelinux/source/detail?r=6)  = 页表基地址 + 16k, 可以看到这是页表的尾地址

216 - 221 行: 循环,从 [r0](https://code.google.com/p/xiangelinux/source/detail?r=0) 到 [r6](https://code.google.com/p/xiangelinux/source/detail?r=6) 将这16k页表用0填充.

223行: 获得proc\_info\_list的cpu\_mm\_mmu\_flags的值,并存储到 r7中. (宏PROCINFO\_MM\_MMUFLAGS是在arch/arm/kernel/asm-offset.c中定义)


231行: 通过pc值的高12位(右移20位),得到kernel的section,并存储到r6中.因为当前是通过运行时地址得到的kernel的 section,因而是物理地址.

232行: [r3](https://code.google.com/p/xiangelinux/source/detail?r=3) = [r7](https://code.google.com/p/xiangelinux/source/detail?r=7) | ([r6](https://code.google.com/p/xiangelinux/source/detail?r=6) << 20); flags + kernel base,得到页表中需要设置的值.

233行: 设置页表: mem[+ r6 \* 4](r4.md) = [r3](https://code.google.com/p/xiangelinux/source/detail?r=3)
> 这里,因为页表的每一项是32 bits(4 bytes),所以要乘以4(<<2).

上面这三行,设置了kernel的第一个section(物理地址所在的page entry)的页表项

239, 240行: TEXTADDR是内核的起始虚拟地址(0xc0008000), 这两行是设置kernel起始虚拟地址的页表项(注意,这里设置的页表项和上面的231 - 233行设置的页表项是不同的 )
> 执行完后,r0指向kernel的第2个section的虚拟地址所在的页表项.


> /**TODO: 这两行的code很奇怪,为什么要先取TEXTADDR的高8位(Bit[31:24])0xff000000,然后再取后面的8位 (Bit[23:20])0x00f00000**/


242行: 这一行计算kernel镜像的大小(bytes).
> _end 是在vmlinux.lds.S中162行定义的,标记kernel的结束位置(虚拟地址):_

```
        00158                .bss : {
        00159                __bss_start = .;        /* BSS                                */
        00160                *(.bss)
        00161                *(COMMON)
        00162                _end = .;
        00163        }
```

> kernel的size =  _end - PAGE\_OFFSET -1, 这里 减1的原因是因为_end 是 location counter,它的地址是kernel镜像后面的一个byte的地址.

243行: 地址右移20位,计算出kernel有多少sections,并将结果存到r6中

245 - 248行: 这几行用来填充kernel所有section虚拟地址对应的页表项.

253行: 将r0设置为RAM第一兆虚拟地址的页表项地址(page entry)

254行: r7中存储的是mmu flags, 逻辑或上RAM的起始物理地址,得到RAM第一个MB页表项的值.

255行:　设置RAM的第一个MB虚拟地址的页表.

上面这三行是用来设置RAM中第一兆虚拟地址的页表. 之所以要设置这个页表项的原因是RAM的第一兆内存中可能存储着boot params.

这样,kernel所需要的基本的页表我们都设置完了, 如下图所示

![http://xiangelinux.googlecode.com/files/armmmu2.gif](http://xiangelinux.googlecode.com/files/armmmu2.gif)

## 4. 调用平台特定的 cpu\_flush 函数 ##

当 create\_page\_tables 返回之后

此时,一些特定寄存器的值如下所示:
**[r4](https://code.google.com/p/xiangelinux/source/detail?r=4) = pgtbl              (page table 的物理基地址)** [r8](https://code.google.com/p/xiangelinux/source/detail?r=8) = machine info       (struct machine\_desc的基地址)
**[r9](https://code.google.com/p/xiangelinux/source/detail?r=9) = cpu id             (通过cp15协处理器获得的cpu id)** [r10](https://code.google.com/p/xiangelinux/source/detail?r=10) = procinfo          (struct proc\_info\_list的基地址)


在我们需要在开启mmu之前,做一些必须的工作:清除ICache, 清除 DCache, 清除 Writebuffer, 清除TLB等.

这些一般是通过cp15协处理器来实现的,并且是平台相关的. 这就是 cpu\_flush 需要做的工作.

> 在 arch/arm/kernel/head.S中

```
00091:         ldr        r13, __switch_data                @ address to jump to after
00092:                                                 @ mmu has been enabled     
00093:         adr        lr, __enable_mmu                @ return (PIC) address     
00094:         add        pc, r10, #PROCINFO_INITFUNC            
```

第91行: 将r13设置为 switch\_data 的地址

第92行: 将lr设置为 enable\_mmu 的地址

第93行: r10存储的是procinfo的基地址, PROCINFO\_INITFUNC是在 arch/arm/kernel/asm-offset.c 中107行定义.
> 则该行将pc设为 proc\_info\_list的 cpu\_flush 函数的地址, 即下面跳转到该函数.
> 在分析 lookup\_processor\_type 的时候,我们已经知道,对于 ARM926EJS 来说,其cpu\_flush指向的是函数 arm926\_setup


> 下面我们来分析函数 arm926\_setup

> 在 arch/arm/mm/proc-arm926.S 中:

```
00391:         .type        __arm926_setup, #function
00392: __arm926_setup:
00393:         mov        r0, #0
00394:         mcr        p15, 0, r0, c7, c7                @ invalidate I,D caches on v4
00395:         mcr        p15, 0, r0, c7, c10, 4                @ drain write buffer on v4
00396: #ifdef CONFIG_MMU
00397:         mcr        p15, 0, r0, c8, c7                @ invalidate I,D TLBs on v4
00398: #endif
00399:
00400:
00401: #ifdef CONFIG_CPU_DCACHE_WRITETHROUGH
00402:         mov        r0, #4                                @ disable write-back on caches explicitly
00403:         mcr        p15, 7, r0, c15, c0, 0
00404: #endif
00405:
00406:         adr        r5, arm926_crval
00407:         ldmia        r5, {r5, r6}
00408:         mrc        p15, 0, r0, c1, c0                @ get control register v4
00409:         bic        r0, r0, r5
00410:         orr        r0, r0, r6
00411: #ifdef CONFIG_CPU_CACHE_ROUND_ROBIN
00412:         orr        r0, r0, #0x4000                        @ .1.. .... .... ....
00413: #endif
00414:         mov        pc, lr        
00415:         .size        __arm926_setup, . - __arm926_setup
00416:
00417:         /*
00418:          *  R
00419:          * .RVI ZFRS BLDP WCAM
00420:          * .011 0001 ..11 0101
00421:          *
00422:          */
00423:         .type        arm926_crval, #object
00424: arm926_crval:
00425:         crval        clear=0x00007f3f, mmuset=0x00003135, ucset=0x00001134
```



第391, 392行: 是函数声明

第393行: 将r0设置为0

第394行: 清除(invalidate)Instruction Cache 和 Data Cache.

第395行: 清除(drain) Write Buffer.

第396 - 398行: 如果有配置了MMU,则需要清除(invalidate)Instruction TLB 和Data TLB

接下来,是对控制寄存器c1进行配置,请参考 ARM926 TRM.

第401 - 404行: 如果配置了Data Cache使用writethrough方式, 需要关掉write-back.

第406行:　取arm926\_crval的地址到r5中, arm926\_crval 在第424行

第407行:　这里我们需要看一下424和425行,其中用到了宏crval,crval是在 arch/arm/mm/proc-macro.S 中:

```
        00053:         .macro        crval, clear, mmuset, ucset
        00054: #ifdef CONFIG_MMU
        00055:         .word        \clear
        00056:         .word        \mmuset
        00057: #else
        00058:         .word        \clear
        00059:         .word        \ucset
        00060: #endif
        00061:         .endm
```

> 配合425行,我们可以看出,首先在arm926\_crval的地址处存放了clear的值,然后接下来的地址存放了mmuset的值(对于配置了 MMU的情况)

所以,在407行中,我们将clear和mmuset的值分别存到了r5, r6中

第408行:　获得控制寄存器c1的值

第409行:  将r0中的 clear ([r5](https://code.google.com/p/xiangelinux/source/detail?r=5)) 对应的位都清除掉

第410行:　设置r0中 mmuset ([r6](https://code.google.com/p/xiangelinux/source/detail?r=6)) 对应的位

第411 - 413行:　如果配置了使用 round robin方式,需要设置控制寄存器c1的 Bit[16](16.md)

第412行: 取lr的值到pc中.
而lr中的值存放的是 enable\_mmu 的地址(arch/arm/kernel/head.S 93行),所以,接下来就是跳转到函数 enable\_mmu

## 5. 开启mmu ##

> 开启mmu是又函数 enable\_mmu 实现的.

> 在进入 enable\_mmu 的时候, r0中已经存放了控制寄存器c1的一些配置(在上一步中进行的设置), 但是并没有真正的打开mmu,

> 在 enable\_mmu 中,我们将打开mmu.

> 此时,一些特定寄存器的值如下所示:

**[r0](https://code.google.com/p/xiangelinux/source/detail?r=0) = c1 parameters      (用来配置控制寄存器的参数)** [r4](https://code.google.com/p/xiangelinux/source/detail?r=4) = pgtbl              (page table 的物理基地址)
**[r8](https://code.google.com/p/xiangelinux/source/detail?r=8) = machine info       (struct machine\_desc的基地址)** [r9](https://code.google.com/p/xiangelinux/source/detail?r=9) = cpu id             (通过cp15协处理器获得的cpu id)
**[r10](https://code.google.com/p/xiangelinux/source/detail?r=10) = procinfo          (struct proc\_info\_list的基地址)**

> 在 arch/arm/kernel/head.S 中:

```
00146:         .type        __enable_mmu, %function
00147: __enable_mmu:
00148: #ifdef CONFIG_ALIGNMENT_TRAP
00149:         orr        r0, r0, #CR_A
00150: #else
00151:         bic        r0, r0, #CR_A
00152: #endif
00153: #ifdef CONFIG_CPU_DCACHE_DISABLE
00154:         bic        r0, r0, #CR_C
00155: #endif
00156: #ifdef CONFIG_CPU_BPREDICT_DISABLE
00157:         bic        r0, r0, #CR_Z
00158: #endif
00159: #ifdef CONFIG_CPU_ICACHE_DISABLE
00160:         bic        r0, r0, #CR_I
00161: #endif
00162:         mov        r5, #(domain_val(DOMAIN_USER, DOMAIN_MANAGER) | \
00163:                       domain_val(DOMAIN_KERNEL, DOMAIN_MANAGER) | \
00164:                       domain_val(DOMAIN_TABLE, DOMAIN_MANAGER) | \
00165:                       domain_val(DOMAIN_IO, DOMAIN_CLIENT))
00166:         mcr        p15, 0, r5, c3, c0, 0                @ load domain access register
00167:         mcr        p15, 0, r4, c2, c0, 0                @ load page table pointer
00168:         b        __turn_mmu_on
00169:
00170: /*
00171:  * Enable the MMU.  This completely changes the structure of the visible
00172:  * memory space.  You will not be able to trace execution through this.
00173:  * If you have an enquiry about this, *please* check the linux-arm-kernel
00174:  * mailing list archives BEFORE sending another post to the list.
00175:  *
00176:  *  r0  = cp#15 control register
00177:  *  r13 = *virtual* address to jump to upon completion
00178:  *
00179:  * other registers depend on the function called upon completion
00180:  */
00181:         .align        5
00182:         .type        __turn_mmu_on, %function
00183: __turn_mmu_on:
00184:         mov        r0, r0
00185:         mcr        p15, 0, r0, c1, c0, 0                @ write control reg
00186:         mrc        p15, 0, r3, c0, c0, 0                @ read id reg
00187:         mov        r3, r3
00188:         mov        r3, r3
00189:         mov        pc, r13
```

第146, 147行: 函数声明

第148 - 161行:  根据相应的配置,设置r0中的相应的Bit. ([r0](https://code.google.com/p/xiangelinux/source/detail?r=0) 将用来配置控制寄存器c1)

第162 - 165行: 设置 domain 参数r5.([r5](https://code.google.com/p/xiangelinux/source/detail?r=5) 将用来配置domain)

第166行: 配置 domain (详细信息清参考arm相关手册)

第167行: 配置页表在存储器中的位置(set ttb).这里页表的基地址是r4, 通过写cp15的c2寄存器来设置页表基地址.

第168行: 跳转到 turn\_mmu\_on. 从名称我们可以猜到,下面是要真正打开mmu了.
> (继续向下看,我们会发现,turn\_mmu\_on就下当前代码的下方,为什么要跳转一下呢? 这是有原因的. go on)

第169 - 180行: 空行和注释. 这里的注释我们可以看到, r0是cp15控制寄存器的内容, r13存储了完成后需要跳转的虚拟地址(因为完成后mmu已经打开了,都是虚拟地址了).

第181行: .algin 5 这句是cache line对齐. 我们可以看到下面一行就是 turn\_mmu\_on, 之所以

第182 - 183行:  turn\_mmu\_on 的函数声明. 这里我们可以看到, turn\_mmu\_on 是紧接着上面第168行的跳转指令的,只是中间在第181行多了一个cache line对齐.

> 这么做的原因是: 下面我们要进行真正的打开mmu操作了, 我们要把打开mmu的操作放到一个单独的cache line上. 而在之前的"启动条件"一节我们说了,I Cache是可以打开也可以关闭的,这里这么做的原因是要保证在I Cache打开的时候,打开mmu的操作也能正常执行.

第184行: 这是一个空操作,相当于nop. 在arm中,nop操作经常用指令 mov rd, rd 来实现.
> 注意: 为什么这里要有一个nop,我思考了很长时间,这里是我的猜测,可能不是正确的:
> 因为之前设置了页表基地址(set ttb),到下一行(185行)打开mmu操作,中间的指令序列是这样的:

  * set ttb(第167行)
  * branch(第168行)
  * nop(第184行)
  * enable mmu(第185行)

> 对于arm的五级流水线: fetch - decode - execute - memory - write

> 他们执行的情况如下图所示:

![http://xiangelinux.googlecode.com/files/armmmu3.gif](http://xiangelinux.googlecode.com/files/armmmu3.gif)

这里需要说明的是,branch操作会在3个cycle中完成,并且会导致重新取指.

> 从这个图我们可以看出来,在enable mmu操作取指的时候, set ttb操作刚好完成.


第185行: 写cp15的控制寄存器c1, 这里是打开mmu的操作,同时会打开cache等(根据r0相应的配置)

第186行: 读取id寄存器.

第187 - 188行: 两个nop.

第189行: 取r13到pc中,我们前面已经看到了, r13中存储的是 switch\_data (在 arch/arm/kernel/head.S 91行),下面会跳到 switch\_data.

第187,188行的两个nop是非常重要的,因为在185行打开mmu操作之后,要等到3个cycle之后才会生效,这和arm的流水线有关系.
因而,在打开mmu操作之后的加了两个nop操作.

## 6. 切换数据 ##

> 在 arch/arm/kernel/head-common.S 中:

```
00014:         .type        __switch_data, %object
00015: __switch_data:
00016:         .long        __mmap_switched
00017:         .long        __data_loc                        @ r4
00018:         .long        __data_start                        @ r5
00019:         .long        __bss_start                        @ r6
00020:         .long        _end                                @ r7
00021:         .long        processor_id                        @ r4
00022:         .long        __machine_arch_type                @ r5
00023:         .long        cr_alignment                        @ r6
00024:         .long        init_thread_union + THREAD_START_SP @ sp
00025:
00026: /*
00027:  * The following fragment of code is executed with the MMU on in MMU mode,
00028:  * and uses absolute addresses; this is not position independent.
00029:  *
00030:  *  r0  = cp#15 control register
00031:  *  r1  = machine ID
00032:  *  r9  = processor ID
00033:  */
00034:         .type        __mmap_switched, %function
00035: __mmap_switched:
00036:         adr        r3, __switch_data + 4
00037:
00038:         ldmia        r3!, {r4, r5, r6, r7}
00039:         cmp        r4, r5                                @ Copy data segment if needed
00040: 1:        cmpne        r5, r6
00041:         ldrne        fp, [r4], #4
00042:         strne        fp, [r5], #4
00043:         bne        1b
00044:
00045:         mov        fp, #0                                @ Clear BSS (and zero fp)
00046: 1:        cmp        r6, r7
00047:         strcc        fp, [r6],#4
00048:         bcc        1b
00049:
00050:         ldmia        r3, {r4, r5, r6, sp}
00051:         str        r9, [r4]                        @ Save processor ID
00052:         str        r1, [r5]                        @ Save machine type
00053:         bic        r4, r0, #CR_A                        @ Clear 'A' bit
00054:         stmia        r6, {r0, r4}                        @ Save control register values
00055:         b        start_kernel
```

第14, 15行: 函数声明

第16 - 24行: 定义了一些地址,例如第16行存储的是 mmap\_switched 的地址, 第17行存储的是 data\_loc 的地址 ......

第34, 35行: 函数 mmap\_switched

第36行: 取 switch\_data + 4的地址到r3. 从上文可以看到这个地址就是第17行的地址.

第37行:　依次取出从第17行到第20行的地址,存储到r4, [r5](https://code.google.com/p/xiangelinux/source/detail?r=5), [r6](https://code.google.com/p/xiangelinux/source/detail?r=6), [r7](https://code.google.com/p/xiangelinux/source/detail?r=7) 中. 并且累加r3的值.当执行完后, r3指向了第21行的位置.
> 对照上文,我们可以得知:

  * [r4](https://code.google.com/p/xiangelinux/source/detail?r=4) - data\_loc
  * [r5](https://code.google.com/p/xiangelinux/source/detail?r=5) - data\_start
  * [r6](https://code.google.com/p/xiangelinux/source/detail?r=6) - bss\_start
  * [r7](https://code.google.com/p/xiangelinux/source/detail?r=7) - _end
> 这几个符号都是在 arch/arm/kernel/vmlinux.lds.S 中定义的变量:_

```
        00102: #ifdef CONFIG_XIP_KERNEL
        00103:         __data_loc = ALIGN(4);                /* location in binary */
        00104:         . = PAGE_OFFSET + TEXT_OFFSET;
        00105: #else
        00106:         . = ALIGN(THREAD_SIZE);
        00107:         __data_loc = .;
        00108: #endif
        00109:
        00110:         .data : AT(__data_loc) {
        00111:                 __data_start = .;        /* address in memory */
        00112:
        00113:                 /*
        00114:                  * first, the init task union, aligned
        00115:                  * to an 8192 byte boundary.
        00116:                  */
        00117:                 *(.init.task)
        
                ......
               
        00158:         .bss : {
        00159:                 __bss_start = .;        /* BSS                                */
        00160:                 *(.bss)
        00161:                 *(COMMON)
        00162:                 _end = .;
        00163:         }

```

> 对于这四个变量,我们简单的介绍一下:
  * data\_loc 是数据存放的位置
  * data\_start 是数据开始的位置
  * bss\_start 是bss开始的位置
  * _end 是bss结束的位置, 也是内核结束的位置_

> 其中对第110行的指令讲解一下: 这里定义了.data 段,后面的AT(data\_loc) 的意思是这部分的内容是在data\_loc中存储的(要注意,储存的位置和链接的位置是可以不相同的).
> 关于 AT 详细的信息请参考 ld.info




第38行: 比较 data\_loc 和 data\_start

第39 - 43行: 这几行是判断数据存储的位置和数据的开始的位置是否相等,如果不相等,则需要搬运数据,从 data\_loc 将数据搬到 data\_start.
> 其中 bss\_start 是bss的开始的位置,也标志了 data 结束的位置,因而用其作为判断数据是否搬运完成.

第45 - 48行:　是清除 bss 段的内容,将其都置成0. 这里使用 _end 来判断 bss 的结束位置._

第50行: 因为在第38行的时候,r3被更新到指向第21行的位置.因而这里取得r4, [r5](https://code.google.com/p/xiangelinux/source/detail?r=5), [r6](https://code.google.com/p/xiangelinux/source/detail?r=6), sp的值分别是:
  * [r4](https://code.google.com/p/xiangelinux/source/detail?r=4) - processor\_id
  * [r5](https://code.google.com/p/xiangelinux/source/detail?r=5) - machine\_arch\_type
  * [r6](https://code.google.com/p/xiangelinux/source/detail?r=6) - cr\_alignment
  * sp - init\_thread\_union + THREAD\_START\_SP

> processor\_id 和 machine\_arch\_type 这两个变量是在 arch/arm/kernel/setup.c 中 第62, 63行中定义的.
> cr\_alignment 是在 arch/arm/kernel/entry-armv.S 中定义的:

```
        00182:         .globl        cr_alignment
        00183:         .globl        cr_no_alignment
        00184: cr_alignment:
        00185:         .space        4
        00186: cr_no_alignment:
        00187:         .space        4
```

> init\_thread\_union 是 init进程的基地址. 在 arch/arm/kernel/init\_task.c 中:

```
        00033: union thread_union init_thread_union
        00034:         __attribute__((__section__(".init.task"))) =
        00035:                 { INIT_THREAD_INFO(init_task) }; 
```

> 对照 vmlnux.lds.S 中的 的117行,我们可以知道init task是存放在 .data 段的开始8k, 并且是THREAD\_SIZE(8k)对齐的

第51行: 将r9中存放的 processor id (在arch/arm/kernel/head.S 75行) 赋值给变量 processor\_id

第52行: 将r1中存放的 machine id (见"启动条件"一节)赋值给变量　machine\_arch\_type

第53行: 清除r0中的 CR\_A 位并将值存到r4中. CR\_A 是在 include/asm-arm/system.h 21行定义, 是cp15控制寄存器c1的Bit[1](1.md)(alignment fault enable/disable)

第54行: 这一行是存储控制寄存器的值.
> 从上面 arch/arm/kernel/entry-armv.S 的代码我们可以得知.
> 这一句是将r0存储到了 cr\_alignment 中,将r4存储到了 cr\_no\_alignment 中.

第55行: 最终跳转到start\_kernel

### FIN ###