# 使用 QEMU 启动一个最小的 Linux kernel

## 验收标准/目标

- QEMU 配合 bzImage 参数启动
- 不需要图形接口，只需要文本模式
- 需要显示启动日志，日志要包含时间
- 最终需要提供一个shell，可以进行简单交互

## 过程

### 构建 bzImage

```
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.19.1.tar.xz
tar xf linux-5.19.1.tar.xz
cd linux-5.19.1
make allnoconfig # or `make tinyconfig`
make bzImage -j20
```

### 启动 QEMU

```console
$ qemu-system-x86_64 -kernel arch/x86/boot/bzImage
VNC server running on 127.0.0.1:5900
```

需要转为文本模式，使用 `-nographic` 参数

```console
$ qemu-system-x86_64 -kernel arch/x86/boot/bzImage -nographic
```

结果变成空白窗口，退出方法是这样一个序列； `Ctrl` + `A`, `C`, `q`.

添加 `console=ttyS0"

```console
$ qemu-system-x86_64 -kernel arch/x86/boot/bzImage -nographic -append "console=ttyS0"
```

结果仍为空白窗口

### 支持日志输出/串口支持

上面空白的原因是缺少串口驱动，至于为什么我还不清楚。

添加串口支持：

```
make menuconfig
# 启用 > Device Drivers > Character devices > Serial drivers > 8250/16550 and compatible serial support > Console on 8250/16550 and compatible serial port
make bzImage -j20
```

### 启动 QEMU

```console
$ qemu-system-x86_64 -kernel arch/x86/boot/bzImage -nographic -append "console=ttyS0"
```

<details>
  <summary>Console output</summary>

```
Linux version 5.19.1+ (ykg@tb) (gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #38 Tue Aug 16 12
x86/fpu: x87 FPU will use FXSAVE
signal: max sigframe size: 1440
BIOS-provided physical RAM map:
BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
BIOS-e820: [mem 0x0000000000100000-0x0000000007fdffff] usable
BIOS-e820: [mem 0x0000000007fe0000-0x0000000007ffffff] reserved
BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
Notice: NX (Execute Disable) protection cannot be enabled: non-PAE kernel!
SMBIOS 2.8 present.
DMI: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.16.0-0-gd239552ce722-prebuilt.qemu.org 04/01/2014
tsc: Fast TSC calibration using PIT
tsc: Detected 2688.041 MHz processor
last_pfn = 0x7fe0 max_arch_pfn = 0x100000
x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- WT
0MB HIGHMEM available.
127MB LOWMEM available.
  mapped low ram: 0 - 07fe0000
  low ram: 0 - 07fe0000
Zone ranges:
  DMA      [mem 0x0000000000001000-0x0000000000ffffff]
  Normal   [mem 0x0000000001000000-0x0000000007fdffff]
  HighMem  empty
Movable zone start for each node
Early memory node ranges
  node   0: [mem 0x0000000000001000-0x000000000009efff]
  node   0: [mem 0x0000000000100000-0x0000000007fdffff]
Initmem setup node 0 [mem 0x0000000000001000-0x0000000007fdffff]
On node 0, zone DMA: 1 pages in unavailable ranges
On node 0, zone DMA: 97 pages in unavailable ranges
[mem 0x08000000-0xfffbffff] available for PCI devices
clocksource: refined-jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645519600211568 ns
Built 1 zonelists, mobility grouping on.  Total pages: 32382
Kernel command line: console=ttyS0
Dentry cache hash table entries: 16384 (order: 4, 65536 bytes, linear)
Inode-cache hash table entries: 8192 (order: 3, 32768 bytes, linear)
mem auto-init: stack:off, heap alloc:off, heap free:off
Initializing HighMem for node 0 (00000000:00000000)
Checking if this processor honours the WP bit even in supervisor mode...Ok.
Memory: 125360K/130552K available (1879K kernel code, 557K rwdata, 408K rodata, 196K init, 280K bss, 5192K reserved, 0K cma-r)
SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
Console: colour VGA+ 80x25
printk: console [ttyS0] enabled
clocksource: tsc-early: mask: 0xffffffffffffffff max_cycles: 0x26bf1cf4906, max_idle_ns: 440795229292 ns
Calibrating delay loop (skipped), value calculated using timer frequency.. 5376.08 BogoMIPS (lpj=10752164)
pid_max: default: 32768 minimum: 301
Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
process: using AMD E400 aware idle routine
Last level iTLB entries: 4KB 512, 2MB 255, 4MB 127
Last level dTLB entries: 4KB 512, 2MB 255, 4MB 127, 1GB 0
CPU: AMD QEMU Virtual CPU version 2.5+ (family: 0xf, model: 0x6b, stepping: 0x1)
Spectre V1 : Mitigation: usercopy/swapgs barriers and __user pointer sanitization
Spectre V2 : Kernel not compiled with retpoline; no mitigation available!
Spectre V2 : Vulnerable
Spectre V2 : Spectre v2 / SpectreRSB mitigation: Filling RSB on context switch
Performance Events: PMU not available due to virtualization, using software events only.
clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
futex hash table entries: 256 (order: -1, 3072 bytes, linear)
clocksource: pit: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 1601818034827 ns
clocksource: Switched to clocksource tsc-early
platform rtc_cmos: registered platform RTC device (no PNP device found)
workingset: timestamp_bits=30 max_order=15 bucket_order=0
Serial: 8250/16550 driver, 4 ports, IRQ sharing disabled
serial8250: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a 16550A
sched_clock: Marking stable (72255057, 8857152)->(83946397, -2834188)
List of all partitions:
No filesystem could mount root, tried:

Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
CPU: 0 PID: 1 Comm: swapper Not tainted 5.19.1+ #38
Hardware name: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.16.0-0-gd239552ce722-prebuilt.qemu.org 04/01/2014
Call Trace:
 dump_stack_lvl+0x20/0x2a
 dump_stack+0xd/0x10
 panic+0x9a/0x1c9
 mount_block_root+0x183/0x183
 mount_root+0x147/0x14f
 ? wait_for_device_probe+0x1e/0x70
 prepare_namespace+0x108/0x12e
 kernel_init_freeable+0x15f/0x167
 ? rest_init+0x80/0x80
 kernel_init+0x15/0x100
 ? schedule_tail_wrapper+0x9/0xc
 ret_from_fork+0x19/0x24
Kernel Offset: disabled
---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0) ]---
```
</details>



到这里 `VFS: Unable to mount root fs on unknown-block`，查得需要有 initramfs。
处理方式可以是使用 `-initrd` 参数，但在此之前需要构建一个 initramfs 的 image。

### 构建 initramfs image

```console
mkinitramfs -o initramfs.img
```

### 启动 QEMU

```
qemu-system-x86_64 -kernel arch/x86/boot/bzImage -nographic -append "console=ttyS0" -initrd initramfs.img
```

这个得到的结果和之前的没有 `-initrd` 的是一样的，因为还需要开启 initramfs 支持

### 开启 initramfs 支持

```console
make menuconfig
# 开启 General Setup > Initial RAM filesystem and RAM disk (initramfs/initrd) support，不开启下面所有 compressed 支持
make bzImage -j20
```

### 启动 QEMU

```console
$ qemu-system-x86_64 -kernel arch/x86/boot/bzImage -nographic -append "console=ttyS0" -initrd initramfs.img
```

<details>
  <summary>Console output</summary>

```
Linux version 5.19.1+ (ykg@tb) (gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #39 Tue Aug 16 10:48:18 CST 2022
x86/fpu: x87 FPU will use FXSAVE
signal: max sigframe size: 1440
BIOS-provided physical RAM map:
BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
BIOS-e820: [mem 0x0000000000100000-0x0000000007fdffff] usable
BIOS-e820: [mem 0x0000000007fe0000-0x0000000007ffffff] reserved
BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
Notice: NX (Execute Disable) protection cannot be enabled: non-PAE kernel!
SMBIOS 2.8 present.
DMI: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.16.0-0-gd239552ce722-prebuilt.qemu.org 04/01/2014
tsc: Fast TSC calibration using PIT
tsc: Detected 2688.027 MHz processor
last_pfn = 0x7fe0 max_arch_pfn = 0x100000
x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- WT
RAMDISK: [mem 0x04383000-0x07fdffff]
0MB HIGHMEM available.
127MB LOWMEM available.
  mapped low ram: 0 - 07fe0000
  low ram: 0 - 07fe0000
Zone ranges:
  DMA      [mem 0x0000000000001000-0x0000000000ffffff]
  Normal   [mem 0x0000000001000000-0x0000000007fdffff]
  HighMem  empty
Movable zone start for each node
Early memory node ranges
  node   0: [mem 0x0000000000001000-0x000000000009efff]
  node   0: [mem 0x0000000000100000-0x0000000007fdffff]
Initmem setup node 0 [mem 0x0000000000001000-0x0000000007fdffff]
On node 0, zone DMA: 1 pages in unavailable ranges
On node 0, zone DMA: 97 pages in unavailable ranges
[mem 0x08000000-0xfffbffff] available for PCI devices
clocksource: refined-jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645519600211568 ns
Built 1 zonelists, mobility grouping on.  Total pages: 32382
Kernel command line: console=ttyS0
Dentry cache hash table entries: 16384 (order: 4, 65536 bytes, linear)
Inode-cache hash table entries: 8192 (order: 3, 32768 bytes, linear)
mem auto-init: stack:off, heap alloc:off, heap free:off
Initializing HighMem for node 0 (00000000:00000000)
Checking if this processor honours the WP bit even in supervisor mode...Ok.
Memory: 63544K/130552K available (1879K kernel code, 557K rwdata, 408K rodata, 200K init, 280K bss, 67008K reserved, 0K cma-reserved, 0K highmem)
SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
Console: colour VGA+ 80x25
printk: console [ttyS0] enabled
clocksource: tsc-early: mask: 0xffffffffffffffff max_cycles: 0x26bf1020800, max_idle_ns: 440795303640 ns
Calibrating delay loop (skipped), value calculated using timer frequency.. 5376.05 BogoMIPS (lpj=10752108)
pid_max: default: 32768 minimum: 301
Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
process: using AMD E400 aware idle routine
Last level iTLB entries: 4KB 512, 2MB 255, 4MB 127
Last level dTLB entries: 4KB 512, 2MB 255, 4MB 127, 1GB 0
CPU: AMD QEMU Virtual CPU version 2.5+ (family: 0xf, model: 0x6b, stepping: 0x1)
Spectre V1 : Mitigation: usercopy/swapgs barriers and __user pointer sanitization
Spectre V2 : Kernel not compiled with retpoline; no mitigation available!
Spectre V2 : Vulnerable
Spectre V2 : Spectre v2 / SpectreRSB mitigation: Filling RSB on context switch
Performance Events: PMU not available due to virtualization, using software events only.
clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
futex hash table entries: 256 (order: -1, 3072 bytes, linear)
clocksource: pit: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 1601818034827 ns
clocksource: Switched to clocksource tsc-early
platform rtc_cmos: registered platform RTC device (no PNP device found)
Unpacking initramfs...
workingset: timestamp_bits=30 max_order=14 bucket_order=0
Serial: 8250/16550 driver, 4 ports, IRQ sharing disabled
serial8250: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a 16550A
sched_clock: Marking stable (71576537, 8853103)->(83401866, -2972226)
Initramfs unpacking failed: compression method zstd not configured
Freeing initrd memory: 61812K
List of all partitions:
No filesystem could mount root, tried:

Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
CPU: 0 PID: 1 Comm: swapper Not tainted 5.19.1+ #39
Hardware name: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.16.0-0-gd239552ce722-prebuilt.qemu.org 04/01/2014
Call Trace:
 dump_stack_lvl+0x20/0x2a
 dump_stack+0xd/0x10
 panic+0x9a/0x1c9
 mount_block_root+0x183/0x183
 mount_root+0x147/0x14f
 prepare_namespace+0x111/0x137
 kernel_init_freeable+0x164/0x16c
 ? rest_init+0x80/0x80
 kernel_init+0x15/0x100
 ? schedule_tail_wrapper+0x9/0xc
 ret_from_fork+0x19/0x24
Kernel Offset: disabled
---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0) ]---
```
</details>

可以看到这个错误提示 `Initramfs unpacking failed: compression method zstd not configured`，需要开启 ZSTD 支持

### 开启 ZSTD 支持

```console
make menuconfig
# 开启 General Setup > Initial RAM filesystem and RAM disk (initramfs/initrd) support > Support initial ramdisk/ramfs compressed using ZSTD
make bzImage -j20
```

### 启动 QEMU

```console
$ qemu-system-x86_64 -kernel arch/x86/boot/bzImage -nographic -append "console=ttyS0" -initrd initramfs.img
```

<details>
  <summary>Console output</summary>

```
Linux version 5.19.1+ (ykg@tb) (gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #40 Tue Aug 16 10:52:24 CST 2022
x86/fpu: x87 FPU will use FXSAVE
signal: max sigframe size: 1440
BIOS-provided physical RAM map:
BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
BIOS-e820: [mem 0x0000000000100000-0x0000000007fdffff] usable
BIOS-e820: [mem 0x0000000007fe0000-0x0000000007ffffff] reserved
BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
Notice: NX (Execute Disable) protection cannot be enabled: non-PAE kernel!
SMBIOS 2.8 present.
DMI: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.16.0-0-gd239552ce722-prebuilt.qemu.org 04/01/2014
tsc: Fast TSC calibration using PIT
tsc: Detected 2688.031 MHz processor
last_pfn = 0x7fe0 max_arch_pfn = 0x100000
x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- WT
RAMDISK: [mem 0x04383000-0x07fdffff]
0MB HIGHMEM available.
127MB LOWMEM available.
  mapped low ram: 0 - 07fe0000
  low ram: 0 - 07fe0000
Zone ranges:
  DMA      [mem 0x0000000000001000-0x0000000000ffffff]
  Normal   [mem 0x0000000001000000-0x0000000007fdffff]
  HighMem  empty
Movable zone start for each node
Early memory node ranges
  node   0: [mem 0x0000000000001000-0x000000000009efff]
  node   0: [mem 0x0000000000100000-0x0000000007fdffff]
Initmem setup node 0 [mem 0x0000000000001000-0x0000000007fdffff]
On node 0, zone DMA: 1 pages in unavailable ranges
On node 0, zone DMA: 97 pages in unavailable ranges
[mem 0x08000000-0xfffbffff] available for PCI devices
clocksource: refined-jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645519600211568 ns
Built 1 zonelists, mobility grouping on.  Total pages: 32382
Kernel command line: console=ttyS0
Dentry cache hash table entries: 16384 (order: 4, 65536 bytes, linear)
Inode-cache hash table entries: 8192 (order: 3, 32768 bytes, linear)
mem auto-init: stack:off, heap alloc:off, heap free:off
Initializing HighMem for node 0 (00000000:00000000)
Checking if this processor honours the WP bit even in supervisor mode...Ok.
Memory: 63460K/130552K available (1946K kernel code, 557K rwdata, 420K rodata, 204K init, 280K bss, 67092K reserved, 0K cma-reserved, 0K highmem)
SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
Console: colour VGA+ 80x25
printk: console [ttyS0] enabled
clocksource: tsc-early: mask: 0xffffffffffffffff max_cycles: 0x26bf13caccf, max_idle_ns: 440795305070 ns
Calibrating delay loop (skipped), value calculated using timer frequency.. 5376.06 BogoMIPS (lpj=10752124)
pid_max: default: 32768 minimum: 301
Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
process: using AMD E400 aware idle routine
Last level iTLB entries: 4KB 512, 2MB 255, 4MB 127
Last level dTLB entries: 4KB 512, 2MB 255, 4MB 127, 1GB 0
CPU: AMD QEMU Virtual CPU version 2.5+ (family: 0xf, model: 0x6b, stepping: 0x1)
Spectre V1 : Mitigation: usercopy/swapgs barriers and __user pointer sanitization
Spectre V2 : Kernel not compiled with retpoline; no mitigation available!
Spectre V2 : Vulnerable
Spectre V2 : Spectre v2 / SpectreRSB mitigation: Filling RSB on context switch
Performance Events: PMU not available due to virtualization, using software events only.
clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
futex hash table entries: 256 (order: -1, 3072 bytes, linear)
clocksource: pit: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 1601818034827 ns
clocksource: Switched to clocksource tsc-early
platform rtc_cmos: registered platform RTC device (no PNP device found)
Unpacking initramfs...
workingset: timestamp_bits=30 max_order=14 bucket_order=0
Serial: 8250/16550 driver, 4 ports, IRQ sharing disabled
serial8250: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a 16550A
sched_clock: Marking stable (75901156, 8906088)->(86484327, -1677083)
kworker/u2:0 invoked oom-killer: gfp_mask=0x140cc2(GFP_HIGHUSER|__GFP_COMP), order=0, oom_score_adj=0
CPU: 0 PID: 5 Comm: kworker/u2:0 Not tainted 5.19.1+ #40
Hardware name: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.16.0-0-gd239552ce722-prebuilt.qemu.org 04/01/2014
Workqueue: events_unbound async_run_entry_fn
Call Trace:
 dump_stack_lvl+0x20/0x2a
 dump_stack+0xd/0x10
 dump_header+0x53/0x1ff
 ? schedule+0x53/0xb0
 out_of_memory.cold+0x36/0x76
 __alloc_pages+0x7c0/0xaa0
 ? release_pages+0xf8/0x340
 __folio_alloc+0x19/0x20
 __filemap_get_folio+0xf0/0x400
 ? copy_page_from_iter_atomic+0x595/0x800
 pagecache_get_page+0x19/0x60
 grab_cache_page_write_begin+0x16/0x20
 simple_write_begin+0x24/0xb0
 generic_perform_write+0x9a/0x1b0
 __generic_file_write_iter+0x115/0x1b0
 generic_file_write_iter+0x59/0xd0
 __kernel_write+0xf1/0x1b0
 kernel_write+0x66/0x170
 xwrite.constprop.0+0x27/0x79
 do_copy+0x7f/0xa8
 write_buffer+0x1d/0x2c
 flush_buffer+0x21/0x6f
 ? initrd_load+0x38/0x38
 unzstd+0x282/0x306
 ? handle_zstd_error+0x72/0x72
 unpack_to_rootfs+0x141/0x1f6
 ? write_buffer+0x2c/0x2c
 ? initrd_load+0x38/0x38
 do_populate_rootfs+0x47/0x96
 async_run_entry_fn+0x22/0x90
 process_one_work+0x186/0x290
 worker_thread+0x13c/0x3d0
 kthread+0x92/0xb0
 ? rescuer_thread+0x330/0x330
 ? kthread_exit+0x30/0x30
 ret_from_fork+0x19/0x24
Mem-Info:
active_anon:0 inactive_anon:0 isolated_anon:0
 active_file:0 inactive_file:0 isolated_file:0
 unevictable:15015 dirty:0 writeback:0
 slab_reclaimable:65 slab_unreclaimable:180
 mapped:0 shmem:0 pagetables:1 bounce:0
 kernel_misc_reclaimable:0
 free:296 free_pcp:0 free_cma:0
Node 0 active_anon:0kB inactive_anon:0kB active_file:0kB inactive_file:0kB unevictable:60060kB isolated(anon):0kB isolated(file):0kB mapped:0kB dirty:0kB writeback:0kB shmem:0kB writebacs
DMA free:424kB boost:0kB min:240kB low:300kB high:360kB reserved_highatomic:0KB active_anon:0kB inactive_anon:0kB active_file:0kB inactive_file:0kB unevictable:14888kB writepending:0kB pB
lowmem_reserve[]: 0 46 46 46
Normal free:760kB boost:0kB min:760kB low:948kB high:1136kB reserved_highatomic:0KB active_anon:0kB inactive_anon:0kB active_file:0kB inactive_file:0kB unevictable:45172kB writepending:0B
lowmem_reserve[]: 0 0 0 0
DMA: 0*4kB 1*8kB (U) 0*16kB 1*32kB (U) 0*64kB 1*128kB (U) 1*256kB (U) 0*512kB 0*1024kB 0*2048kB 0*4096kB = 424kB
Normal: 2*4kB (UM) 2*8kB (UM) 2*16kB (UM) 0*32kB 1*64kB (M) 1*128kB (U) 0*256kB 1*512kB (M) 0*1024kB 0*2048kB 0*4096kB = 760kB
15020 total pagecache pages
32638 pages RAM
0 pages HighMem/MovableOnly
16773 pages reserved
Tasks state (memory values in pages):
[  pid  ]   uid  tgid total_vm      rss pgtables_bytes swapents oom_score_adj name
Out of memory and no killable processes...
Kernel panic - not syncing: System is deadlocked on memory
CPU: 0 PID: 5 Comm: kworker/u2:0 Not tainted 5.19.1+ #40
Hardware name: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.16.0-0-gd239552ce722-prebuilt.qemu.org 04/01/2014
Workqueue: events_unbound async_run_entry_fn
Call Trace:
 dump_stack_lvl+0x20/0x2a
 dump_stack+0xd/0x10
 panic+0x9a/0x1c9
 out_of_memory.cold+0x5a/0x76
 __alloc_pages+0x7c0/0xaa0
 ? release_pages+0xf8/0x340
 __folio_alloc+0x19/0x20
 __filemap_get_folio+0xf0/0x400
 ? copy_page_from_iter_atomic+0x595/0x800
 pagecache_get_page+0x19/0x60
 grab_cache_page_write_begin+0x16/0x20
 simple_write_begin+0x24/0xb0
 generic_perform_write+0x9a/0x1b0
 __generic_file_write_iter+0x115/0x1b0
 generic_file_write_iter+0x59/0xd0
 __kernel_write+0xf1/0x1b0
 kernel_write+0x66/0x170
 xwrite.constprop.0+0x27/0x79
 do_copy+0x7f/0xa8
 write_buffer+0x1d/0x2c
 flush_buffer+0x21/0x6f
 ? initrd_load+0x38/0x38
 unzstd+0x282/0x306
 ? handle_zstd_error+0x72/0x72
 unpack_to_rootfs+0x141/0x1f6
 ? write_buffer+0x2c/0x2c
 ? initrd_load+0x38/0x38
 do_populate_rootfs+0x47/0x96
 async_run_entry_fn+0x22/0x90
 process_one_work+0x186/0x290
 worker_thread+0x13c/0x3d0
 kthread+0x92/0xb0
 ? rescuer_thread+0x330/0x330
 ? kthread_exit+0x30/0x30
 ret_from_fork+0x19/0x24
Kernel Offset: disabled
---[ end Kernel panic - not syncing: System is deadlocked on memory ]---
```
</details>

最后提示 `System is deadlocked on memory`，先调高内存，可以使用 `-m` 参数。

### 调高内存

默认是 128M 内存，先调高到 256M

```
qemu-system-x86_64 -kernel arch/x86/boot/bzImage -nographic -append "console=ttyS0" -initrd initramfs.img -m 256
```

问题仍在，继续调高到 512M 则可以通过

```console
$ qemu-system-x86_64 -kernel arch/x86/boot/bzImage -nographic -append "console=ttyS0" -initrd initramfs.img -m 512
```

<details>
  <summary>Console output</summary>

```
Linux version 5.19.1+ (ykg@tb) (gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #40 Tue Aug 16 10:52:24 CST 2022
x86/fpu: x87 FPU will use FXSAVE
signal: max sigframe size: 1440
BIOS-provided physical RAM map:
BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
BIOS-e820: [mem 0x0000000000100000-0x000000001ffdffff] usable
BIOS-e820: [mem 0x000000001ffe0000-0x000000001fffffff] reserved
BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
Notice: NX (Execute Disable) protection cannot be enabled: non-PAE kernel!
SMBIOS 2.8 present.
DMI: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.16.0-0-gd239552ce722-prebuilt.qemu.org 04/01/2014
tsc: Fast TSC calibration using PIT
tsc: Detected 2688.036 MHz processor
last_pfn = 0x1ffe0 max_arch_pfn = 0x100000
x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- WT
RAMDISK: [mem 0x1c383000-0x1ffdffff]
0MB HIGHMEM available.
511MB LOWMEM available.
  mapped low ram: 0 - 1ffe0000
  low ram: 0 - 1ffe0000
Zone ranges:
  DMA      [mem 0x0000000000001000-0x0000000000ffffff]
  Normal   [mem 0x0000000001000000-0x000000001ffdffff]
  HighMem  empty
Movable zone start for each node
Early memory node ranges
  node   0: [mem 0x0000000000001000-0x000000000009efff]
  node   0: [mem 0x0000000000100000-0x000000001ffdffff]
Initmem setup node 0 [mem 0x0000000000001000-0x000000001ffdffff]
On node 0, zone DMA: 1 pages in unavailable ranges
On node 0, zone DMA: 97 pages in unavailable ranges
[mem 0x20000000-0xfffbffff] available for PCI devices
clocksource: refined-jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645519600211568 ns
Built 1 zonelists, mobility grouping on.  Total pages: 129918
Kernel command line: console=ttyS0
Dentry cache hash table entries: 65536 (order: 6, 262144 bytes, linear)
Inode-cache hash table entries: 32768 (order: 5, 131072 bytes, linear)
mem auto-init: stack:off, heap alloc:off, heap free:off
Initializing HighMem for node 0 (00000000:00000000)
Checking if this processor honours the WP bit even in supervisor mode...Ok.
Memory: 453316K/523768K available (1946K kernel code, 557K rwdata, 420K rodata, 204K init, 280K bss, 70452K reserved, 0K cma-reserved, 0K highmem)
SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
Console: colour VGA+ 80x25
printk: console [ttyS0] enabled
clocksource: tsc-early: mask: 0xffffffffffffffff max_cycles: 0x26bf188e987, max_idle_ns: 440795259317 ns
Calibrating delay loop (skipped), value calculated using timer frequency.. 5376.07 BogoMIPS (lpj=10752144)
pid_max: default: 32768 minimum: 301
Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
process: using AMD E400 aware idle routine
Last level iTLB entries: 4KB 512, 2MB 255, 4MB 127
Last level dTLB entries: 4KB 512, 2MB 255, 4MB 127, 1GB 0
CPU: AMD QEMU Virtual CPU version 2.5+ (family: 0xf, model: 0x6b, stepping: 0x1)
Spectre V1 : Mitigation: usercopy/swapgs barriers and __user pointer sanitization
Spectre V2 : Kernel not compiled with retpoline; no mitigation available!
Spectre V2 : Vulnerable
Spectre V2 : Spectre v2 / SpectreRSB mitigation: Filling RSB on context switch
Performance Events: PMU not available due to virtualization, using software events only.
clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
futex hash table entries: 256 (order: -1, 3072 bytes, linear)
clocksource: pit: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 1601818034827 ns
clocksource: Switched to clocksource tsc-early
platform rtc_cmos: registered platform RTC device (no PNP device found)
Unpacking initramfs...
workingset: timestamp_bits=30 max_order=17 bucket_order=0
Serial: 8250/16550 driver, 4 ports, IRQ sharing disabled
serial8250: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a 16550A
sched_clock: Marking stable (81926675, 8953222)->(90615322, 264575)
clocksource: tsc: mask: 0xffffffffffffffff max_cycles: 0x26bf188e987, max_idle_ns: 440795259317 ns
clocksource: Switched to clocksource tsc
Freeing initrd memory: 61812K
Freeing unused kernel image (initmem) memory: 204K
Write protecting kernel text and read-only data: 2368k
Run /init as init process
Failed to execute /init (error -2)
Run /sbin/init as init process
Run /etc/init as init process
Run /bin/init as init process
Run /bin/sh as init process
Kernel panic - not syncing: No working init found.  Try passing init= option to kernel. See Linux Documentation/admin-guide/init.rst for guidance.
CPU: 0 PID: 1 Comm: swapper Not tainted 5.19.1+ #40
Hardware name: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.16.0-0-gd239552ce722-prebuilt.qemu.org 04/01/2014
Call Trace:
 dump_stack_lvl+0x20/0x2a
 ? irqentry_enter+0x10/0x20
 dump_stack+0xd/0x10
 panic+0x9a/0x1c9
 ? rest_init+0x80/0x80
 kernel_init+0xfc/0x100
 ret_from_fork+0x19/0x24
Kernel Offset: disabled
---[ end Kernel panic - not syncing: No working init found.  Try passing init= option to kernel. See Linux Documentation/admin-guide/init.rst for guidance. ]---
```
</details>

这里最后提示 `No working init found`, 原因是没有添加可执行文件格式支持

### 添加可执行文件格式支持

```
make menuconfig
# 启用 > Executable file formats >  Kernel support for ELF binaries
# 启用 > Executable file formats >  Kernel support for scripts starting with #!
make bzImage -j20
```

启用 script 支持是因为 initramfs 的 /init 文件就是 `#!` 开头的，shell 本身是 ELF 格式，所以还需要添加 ELF 格式支持。

### 启动 QEMU

```console
$ qemu-system-x86_64 -kernel arch/x86/boot/bzImage -nographic -append "console=ttyS0" -initrd initramfs.img -m 512
```

<details>
  <summary>Console output</summary>

```
Linux version 5.19.1+ (ykg@tb) (gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #41 Tue Aug 16 11:01:53 CST 2022
x86/fpu: x87 FPU will use FXSAVE
signal: max sigframe size: 1440
BIOS-provided physical RAM map:
BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
BIOS-e820: [mem 0x0000000000100000-0x000000001ffdffff] usable
BIOS-e820: [mem 0x000000001ffe0000-0x000000001fffffff] reserved
BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
Notice: NX (Execute Disable) protection cannot be enabled: non-PAE kernel!
SMBIOS 2.8 present.
DMI: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.16.0-0-gd239552ce722-prebuilt.qemu.org 04/01/2014
tsc: Fast TSC calibration using PIT
tsc: Detected 2688.043 MHz processor
last_pfn = 0x1ffe0 max_arch_pfn = 0x100000
x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- WT
RAMDISK: [mem 0x1c383000-0x1ffdffff]
0MB HIGHMEM available.
511MB LOWMEM available.
  mapped low ram: 0 - 1ffe0000
  low ram: 0 - 1ffe0000
Zone ranges:
  DMA      [mem 0x0000000000001000-0x0000000000ffffff]
  Normal   [mem 0x0000000001000000-0x000000001ffdffff]
  HighMem  empty
Movable zone start for each node
Early memory node ranges
  node   0: [mem 0x0000000000001000-0x000000000009efff]
  node   0: [mem 0x0000000000100000-0x000000001ffdffff]
Initmem setup node 0 [mem 0x0000000000001000-0x000000001ffdffff]
On node 0, zone DMA: 1 pages in unavailable ranges
On node 0, zone DMA: 97 pages in unavailable ranges
[mem 0x20000000-0xfffbffff] available for PCI devices
clocksource: refined-jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645519600211568 ns
Built 1 zonelists, mobility grouping on.  Total pages: 129918
Kernel command line: console=ttyS0
Dentry cache hash table entries: 65536 (order: 6, 262144 bytes, linear)
Inode-cache hash table entries: 32768 (order: 5, 131072 bytes, linear)
mem auto-init: stack:off, heap alloc:off, heap free:off
Initializing HighMem for node 0 (00000000:00000000)
Checking if this processor honours the WP bit even in supervisor mode...Ok.
Memory: 453300K/523768K available (1956K kernel code, 561K rwdata, 420K rodata, 204K init, 280K bss, 70468K reserved, 0K cma-reserved, 0K highmem)
SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
Console: colour VGA+ 80x25
printk: console [ttyS0] enabled
clocksource: tsc-early: mask: 0xffffffffffffffff max_cycles: 0x26bf1f278cc, max_idle_ns: 440795293632 ns
Calibrating delay loop (skipped), value calculated using timer frequency.. 5376.08 BogoMIPS (lpj=10752172)
pid_max: default: 32768 minimum: 301
Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
process: using AMD E400 aware idle routine
Last level iTLB entries: 4KB 512, 2MB 255, 4MB 127
Last level dTLB entries: 4KB 512, 2MB 255, 4MB 127, 1GB 0
CPU: AMD QEMU Virtual CPU version 2.5+ (family: 0xf, model: 0x6b, stepping: 0x1)
Spectre V1 : Mitigation: usercopy/swapgs barriers and __user pointer sanitization
Spectre V2 : Kernel not compiled with retpoline; no mitigation available!
Spectre V2 : Vulnerable
Spectre V2 : Spectre v2 / SpectreRSB mitigation: Filling RSB on context switch
Performance Events: PMU not available due to virtualization, using software events only.
clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
futex hash table entries: 256 (order: -1, 3072 bytes, linear)
clocksource: pit: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 1601818034827 ns
clocksource: Switched to clocksource tsc-early
platform rtc_cmos: registered platform RTC device (no PNP device found)
Unpacking initramfs...
workingset: timestamp_bits=30 max_order=17 bucket_order=0
Serial: 8250/16550 driver, 4 ports, IRQ sharing disabled
serial8250: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a 16550A
sched_clock: Marking stable (74551001, 9047694)->(86636626, -3037931)
clocksource: tsc: mask: 0xffffffffffffffff max_cycles: 0x26bf1f278cc, max_idle_ns: 440795293632 ns
clocksource: Switched to clocksource tsc
Freeing initrd memory: 61812K
Freeing unused kernel image (initmem) memory: 204K
Write protecting kernel text and read-only data: 2380k
Run /init as init process
Failed to execute /init (error -8)
Run /sbin/init as init process
Run /etc/init as init process
Run /bin/init as init process
Run /bin/sh as init process
Starting init: /bin/sh exists but couldn't execute it (error -8)
Kernel panic - not syncing: No working init found.  Try passing init= option to kernel. See Linux Documentation/admin-guide/init.rst for guidance.
CPU: 0 PID: 1 Comm: swapper Not tainted 5.19.1+ #41
Hardware name: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.16.0-0-gd239552ce722-prebuilt.qemu.org 04/01/2014
Call Trace:
 dump_stack_lvl+0x20/0x2a
 ? rest_init+0x80/0x80
 dump_stack+0xd/0x10
 panic+0x9a/0x1c9
 ? rest_init+0x80/0x80
 kernel_init+0xfc/0x100
 ret_from_fork+0x19/0x24
Kernel Offset: disabled
---[ end Kernel panic - not syncing: No working init found.  Try passing init= option to kernel. See Linux Documentation/admin-guide/init.rst for guidance. ]---
```
</details>

到这一步时曾卡了一些时间，后来通过 printk 定位到原因是 arch 不匹配问题，kernel 配置时没有开启 `64-bit Kernel` 选项。

### 启用 64-bit kernel

```
make menuconfig
# 启用 64-bit kernel
make bzImage -j20
```

### 启动 QEMU

```console
$ qemu-system-x86_64 -kernel arch/x86/boot/bzImage -nographic -append "console=ttyS0" -initrd initramfs.img -m 512
```

<details>
  <summary>Console output</summary>

```
Linux version 5.19.1+ (ykg@tb) (gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #42 Tue Aug 16 11:08:31 CST 2022
Command line: console=ttyS0
x86/fpu: x87 FPU will use FXSAVE
signal: max sigframe size: 1040
BIOS-provided physical RAM map:
BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
BIOS-e820: [mem 0x0000000000100000-0x000000001ffdffff] usable
BIOS-e820: [mem 0x000000001ffe0000-0x000000001fffffff] reserved
BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
NX (Execute Disable) protection: active
SMBIOS 2.8 present.
DMI: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.16.0-0-gd239552ce722-prebuilt.qemu.org 04/01/2014
tsc: Fast TSC calibration using PIT
tsc: Detected 2688.032 MHz processor
last_pfn = 0x1ffe0 max_arch_pfn = 0x400000000
x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- WT
found SMP MP-table at [mem 0x000f5ba0-0x000f5baf]
RAMDISK: [mem 0x1c383000-0x1ffdffff]
Zone ranges:
  DMA      [mem 0x0000000000001000-0x0000000000ffffff]
  DMA32    [mem 0x0000000001000000-0x000000001ffdffff]
  Normal   empty
Movable zone start for each node
Early memory node ranges
  node   0: [mem 0x0000000000001000-0x000000000009efff]
  node   0: [mem 0x0000000000100000-0x000000001ffdffff]
Initmem setup node 0 [mem 0x0000000000001000-0x000000001ffdffff]
On node 0, zone DMA: 1 pages in unavailable ranges
On node 0, zone DMA: 97 pages in unavailable ranges
On node 0, zone DMA32: 32 pages in unavailable ranges
Intel MultiProcessor Specification v1.4
MPTABLE: OEM ID: BOCHSCPU
MPTABLE: Product ID: 0.1
MPTABLE: APIC at: 0xFEE00000
Processor #0 (Bootup-CPU)
IOAPIC[0]: apic_id 0, version 32, address 0xfec00000, GSI 0-23
Processors: 1
[mem 0x20000000-0xfffbffff] available for PCI devices
clocksource: refined-jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645519600211568 ns
Built 1 zonelists, mobility grouping on.  Total pages: 128736
Kernel command line: console=ttyS0
Dentry cache hash table entries: 65536 (order: 7, 524288 bytes, linear)
Inode-cache hash table entries: 32768 (order: 6, 262144 bytes, linear)
mem auto-init: stack:off, heap alloc:off, heap free:off
Memory: 441752K/523768K available (4096K kernel code, 763K rwdata, 496K rodata, 532K init, 720K bss, 81760K reserved, 0K cma-reserved)
SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
NR_IRQS: 4352, nr_irqs: 48, preallocated irqs: 16
Console: colour VGA+ 80x25
printk: console [ttyS0] enabled
APIC: Switch to symmetric I/O mode setup
..TIMER: vector=0x30 apic1=0 pin1=2 apic2=-1 pin2=-1
clocksource: tsc-early: mask: 0xffffffffffffffff max_cycles: 0x26bf1486760, max_idle_ns: 440795273615 ns
Calibrating delay loop (skipped), value calculated using timer frequency.. 5376.06 BogoMIPS (lpj=10752128)
pid_max: default: 32768 minimum: 301
Mount-cache hash table entries: 1024 (order: 1, 8192 bytes, linear)
Mountpoint-cache hash table entries: 1024 (order: 1, 8192 bytes, linear)
process: using AMD E400 aware idle routine
Last level iTLB entries: 4KB 512, 2MB 255, 4MB 127
Last level dTLB entries: 4KB 512, 2MB 255, 4MB 127, 1GB 0
CPU: AMD QEMU Virtual CPU version 2.5+ (family: 0xf, model: 0x6b, stepping: 0x1)
Spectre V1 : Mitigation: usercopy/swapgs barriers and __user pointer sanitization
Spectre V2 : Kernel not compiled with retpoline; no mitigation available!
Spectre V2 : Vulnerable
Spectre V2 : Spectre v2 / SpectreRSB mitigation: Filling RSB on context switch
Performance Events: PMU not available due to virtualization, using software events only.
clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
futex hash table entries: 256 (order: 0, 6144 bytes, linear)
clocksource: Switched to clocksource tsc-early
platform rtc_cmos: registered platform RTC device (no PNP device found)
Unpacking initramfs...
workingset: timestamp_bits=62 max_order=17 bucket_order=0
Serial: 8250/16550 driver, 4 ports, IRQ sharing disabled
serial8250: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a 16550A
sched_clock: Marking stable (310369976, 7997856)->(322031059, -3663227)
clocksource: tsc: mask: 0xffffffffffffffff max_cycles: 0x26bf1486760, max_idle_ns: 440795273615 ns
clocksource: Switched to clocksource tsc
Freeing initrd memory: 61812K
Freeing unused kernel image (initmem) memory: 532K
Write protecting the kernel read-only data: 8192k
Freeing unused kernel image (text/rodata gap) memory: 2044K
Freeing unused kernel image (rodata/data gap) memory: 1552K
Run /init as init process
Loading, please wait...
mount: mounting udev on /dev failed: No such device
tmpfs: Unknown parameter 'size'
mount: mounting tmpfs on /run failed: Invalid argument
Failed to create socket: Function not implemented
Failed to initialize udev control socket: Function not implemented
Failed to create manager: Function not implemented
Begin: Loading essential drivers ... done.
Begin: Running /scripts/init-premount ... done.
Begin: Mounting root file system ... Begin: Running /scripts/local-top ... done.
Begin: Running /scripts/local-premount ... done.
chvt: can't open console
No root device specified. Boot arguments must include a root= parameter.


BusyBox v1.30.1 (Ubuntu 1:1.30.1-7ubuntu3) built-in shell (ash)
Enter 'help' for a list of built-in commands.

(initramfs) ps
  PID USER       VSZ STAT COMMAND
    1 0         3196 S    /bin/sh /init
    2 0            0 SW   [kthreadd]
    3 0            0 IW   [kworker/0:0-eve]
    4 0            0 IW<  [kworker/0:0H-ev]
    5 0            0 IW   [kworker/u2:0-ev]
    6 0            0 IW<  [kworker/0:1H-ev]
    7 0            0 IW<  [mm_percpu_wq]
    8 0            0 SW   [ksoftirqd/0]
    9 0            0 SW   [oom_reaper]
   10 0            0 IW<  [writeback]
   11 0            0 IW<  [kblockd]
   12 0            0 SW   [kswapd0]
   13 0            0 IW   [kworker/u2:1-ev]
   14 0            0 IW   [kworker/0:1-eve]
   15 0            0 IW   [kworker/u2:2-ev]
   51 0         3060 S    sh -i
   52 0         3060 R    {ps} sh -i
(initramfs)
```
</details>

得到了 shell，但这里还有个问题，如果没有开启 `64-bit kernel`，按说我使用 `qemu-system-i386` 启动应该没有问题，但结果是不行，不知道为什么。

### 添加启动日志时间

```
make menuconfig
# 开启 > Kernel hacking > printk and dmesg options > Show timing information on printks
make bzImage -j20
```

### 启动 QEMU

```console
$ qemu-system-x86_64 -kernel arch/x86/boot/bzImage -nographic -append "console=ttyS0" -initrd initramfs.img -m 512
```

<details>
  <summary>Console output</summary>

```
[    0.000000] Linux version 5.19.1+ (ykg@tb) (gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #43 Tue Aug 16 11:13:06 CST 2022
[    0.000000] Command line: console=ttyS0
[    0.000000] x86/fpu: x87 FPU will use FXSAVE
[    0.000000] signal: max sigframe size: 1040
[    0.000000] BIOS-provided physical RAM map:
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
[    0.000000] BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000000100000-0x000000001ffdffff] usable
[    0.000000] BIOS-e820: [mem 0x000000001ffe0000-0x000000001fffffff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
[    0.000000] NX (Execute Disable) protection: active
[    0.000000] SMBIOS 2.8 present.
[    0.000000] DMI: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.16.0-0-gd239552ce722-prebuilt.qemu.org 04/01/2014
[    0.000000] tsc: Fast TSC calibration using PIT
[    0.000000] tsc: Detected 2688.042 MHz processor
[    0.003265] last_pfn = 0x1ffe0 max_arch_pfn = 0x400000000
[    0.003673] x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- WT
[    0.010164] found SMP MP-table at [mem 0x000f5ba0-0x000f5baf]
[    0.012584] RAMDISK: [mem 0x1c383000-0x1ffdffff]
[    0.013763] Zone ranges:
[    0.013777]   DMA      [mem 0x0000000000001000-0x0000000000ffffff]
[    0.013826]   DMA32    [mem 0x0000000001000000-0x000000001ffdffff]
[    0.013831]   Normal   empty
[    0.013840] Movable zone start for each node
[    0.013861] Early memory node ranges
[    0.013877]   node   0: [mem 0x0000000000001000-0x000000000009efff]
[    0.013968]   node   0: [mem 0x0000000000100000-0x000000001ffdffff]
[    0.014052] Initmem setup node 0 [mem 0x0000000000001000-0x000000001ffdffff]
[    0.014699] On node 0, zone DMA: 1 pages in unavailable ranges
[    0.014876] On node 0, zone DMA: 97 pages in unavailable ranges
[    0.017966] On node 0, zone DMA32: 32 pages in unavailable ranges
[    0.018053] Intel MultiProcessor Specification v1.4
[    0.018106] MPTABLE: OEM ID: BOCHSCPU
[    0.018114] MPTABLE: Product ID: 0.1
[    0.018123] MPTABLE: APIC at: 0xFEE00000
[    0.018355] Processor #0 (Bootup-CPU)
[    0.018647] IOAPIC[0]: apic_id 0, version 32, address 0xfec00000, GSI 0-23
[    0.018744] Processors: 1
[    0.019188] [mem 0x20000000-0xfffbffff] available for PCI devices
[    0.019309] clocksource: refined-jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645519600211568 ns
[    0.020440] Built 1 zonelists, mobility grouping on.  Total pages: 128736
[    0.020520] Kernel command line: console=ttyS0
[    0.021521] Dentry cache hash table entries: 65536 (order: 7, 524288 bytes, linear)
[    0.021828] Inode-cache hash table entries: 32768 (order: 6, 262144 bytes, linear)
[    0.022505] mem auto-init: stack:off, heap alloc:off, heap free:off
[    0.026298] Memory: 441752K/523768K available (4096K kernel code, 763K rwdata, 496K rodata, 532K init, 720K bss, 81760K reserved, 0K cma-reserved)
[    0.028235] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.030667] NR_IRQS: 4352, nr_irqs: 48, preallocated irqs: 16
[    0.036783] Console: colour VGA+ 80x25
[    0.041603] printk: console [ttyS0] enabled
[    0.042368] APIC: Switch to symmetric I/O mode setup
[    0.044650] ..TIMER: vector=0x30 apic1=0 pin1=2 apic2=-1 pin2=-1
[    0.062562] clocksource: tsc-early: mask: 0xffffffffffffffff max_cycles: 0x26bf1e6be34, max_idle_ns: 440795325087 ns
[    0.062955] Calibrating delay loop (skipped), value calculated using timer frequency.. 5376.08 BogoMIPS (lpj=10752168)
[    0.063184] pid_max: default: 32768 minimum: 301
[    0.063717] Mount-cache hash table entries: 1024 (order: 1, 8192 bytes, linear)
[    0.063811] Mountpoint-cache hash table entries: 1024 (order: 1, 8192 bytes, linear)
[    0.071794] process: using AMD E400 aware idle routine
[    0.071931] Last level iTLB entries: 4KB 512, 2MB 255, 4MB 127
[    0.072003] Last level dTLB entries: 4KB 512, 2MB 255, 4MB 127, 1GB 0
[    0.072122] CPU: AMD QEMU Virtual CPU version 2.5+ (family: 0xf, model: 0x6b, stepping: 0x1)
[    0.072357] Spectre V1 : Mitigation: usercopy/swapgs barriers and __user pointer sanitization
[    0.072560] Spectre V2 : Kernel not compiled with retpoline; no mitigation available!
[    0.072580] Spectre V2 : Vulnerable
[    0.072708] Spectre V2 : Spectre v2 / SpectreRSB mitigation: Filling RSB on context switch
[    0.081338] Performance Events: PMU not available due to virtualization, using software events only.
[    0.298901] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
[    0.298901] futex hash table entries: 256 (order: 0, 6144 bytes, linear)
[    0.303961] clocksource: Switched to clocksource tsc-early
[    0.306847] platform rtc_cmos: registered platform RTC device (no PNP device found)
[    0.310951] Unpacking initramfs...
[    0.314049] workingset: timestamp_bits=62 max_order=17 bucket_order=0
[    0.318322] Serial: 8250/16550 driver, 4 ports, IRQ sharing disabled
[    0.318955] serial8250: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a 16550A
[    0.320867] sched_clock: Marking stable (309113943, 8803231)->(322352555, -4435381)
[    1.322641] clocksource: tsc: mask: 0xffffffffffffffff max_cycles: 0x26bf1e6be34, max_idle_ns: 440795325087 ns
[    1.322767] clocksource: Switched to clocksource tsc
[    1.416307] Freeing initrd memory: 61812K
[    1.429885] Freeing unused kernel image (initmem) memory: 532K
[    1.430168] Write protecting the kernel read-only data: 8192k
[    1.431428] Freeing unused kernel image (text/rodata gap) memory: 2044K
[    1.432215] Freeing unused kernel image (rodata/data gap) memory: 1552K
[    1.432368] Run /init as init process
Loading, please wait...
mount: mounting udev on /dev failed: No such device
[    1.601327] tmpfs: Unknown parameter 'size'
mount: mounting tmpfs on /run failed: Invalid argument
Failed to create socket: Function not implemented
Failed to initialize udev control socket: Function not implemented
Failed to create manager: Function not implemented
Begin: Loading essential drivers ... done.
Begin: Running /scripts/init-premount ... done.
Begin: Mounting root file system ... Begin: Running /scripts/local-top ... done.
Begin: Running /scripts/local-premount ... done.
chvt: can't open console
No root device specified. Boot arguments must include a root= parameter.


BusyBox v1.30.1 (Ubuntu 1:1.30.1-7ubuntu3) built-in shell (ash)
Enter 'help' for a list of built-in commands.

(initramfs)
```
</details>


至此目标达成。

## 回顾

为达成目标，需要以下配置

- QEMU 命令

    - qemu_system_x86_64 作为启动命令
    - -kernel bzImage 用来指定内核
    - -nographic 用以禁用图形模式
    - -append "console=ttyS0" 用以支持输出到标准输出
    - -m 512 用以调整内存大小
    - -initrd 用以指定 intramfs.img

- 内核配置

    - 串口驱动支持（Console on 8250/16550 and compatible serial port）
    - initramfs（ZSTD）支持
    - ELF/Script 可执行文件格式支持
    - 64-bit kernel 支持
    - printk 时间支持

- 创建 initramfs.img (mkinitramfs)

## 新问题

- mkinitramfs 干了什么？
- mkinitramfs 生成的 img 为什么可以和新内核配合工作？因为 mkinitramfs 是在 host 机器上生成的，它还不认得新内核
- 为什么不使用 -append "console=ttyS0" 和串口驱动就没有日志输出？这两个配合选项做了什么？
- 为什么内存需要 512MB 才够启动？
- initramfs 为什么一定要有，为什么没有就启动不了，为什么有了就可以？
- 为什么 qemu-system-i386 不能启动未开启 64-bit kernel 的 bzImage？

# 后记

- 在 Ubuntu 16.04 32-bit 版本上创建了一个 initramfs.img，不开启 64-bit kernel 情况下，使用 qemu-system-i386 启动没有问题。只是需要开启 gzip 压缩格式 initramfs 支持
