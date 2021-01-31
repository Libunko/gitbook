# linux 用户空间刷 uboot

1. 查看系统分区 `cat /proc/mtd`
```c
root@OpenWrt:/# cat /proc/mtd
dev:    size   erasesize  name
mtd0: 00060000 00020000 "uboot"
mtd1: 00020000 00020000 "uboot.env"
mtd2: 00300000 00020000 "kernel"
mtd3: 03c80000 00020000 "rootfs"
```
可知uboot分区在mtd0处。

2. 查看 mtd0 分区是否可写 `mtdinfo /dev/mtd0`
```c
root@OpenWrt:/# mtdinfo /dev/mtd0
mtd0
Name:                           uboot
Type:                           nor
Eraseblock size:                131072 bytes, 128.0 KiB
Amount of eraseblocks:          3 (393216 bytes, 384.0 KiB)
Minimum input/output unit size: 1 byte
Sub-page size:                  1 byte
Character device major/minor:   90:0
Bad blocks are allowed:         false
Device is writable:             true
```
如上，Device is writable: true，说明 mtd0 可写，直接执行 `mtd -r write uboot.bin uboot`可直接刷。要是 uboot 分区不可写怎么办呢？

# linux 用户空间 uboot 分区不可写刷 uboot

这里我能想到的办法有两种：
- 如果内核`CONFIG_MTD_NAND_TRANSCEDE=y`，也就是说内核支持 bootargs 传入分区表，我们可以按照内核的分区表写进 bootargs，bootargs 增加 `mtdparts=physmap-flash.0:384k(uboot),128k(uboot.env),3M(kernel),-(rootfs)`
- 如果内核不支持分区表写入 bootargs，只有修改内核里面的分区表，将 uboot 分区的 MTD_WRITEABLE 标志去掉

~~.mask_flags = MTD_WRITEABLE,~~

# bootargs 传入只读分区

只需在分区名字后加 `ro`，例：
- `mtdparts=physmap-flash.0:384k(uboot)ro,128k(uboot.env),3M(kernel),-(rootfs)`

# to t2200

set bootargs 'mtdparts=physmap-flash.0:384k(uboot),128k(uboot.env),3M(kernel),-(rootfs) root=/dev/mtdblock3 rootfstype=jffs2 mem=384M icc_heap_size=4M icc_part_size=384M ddr_limit=2G cram_offset=0x23000 console=ttyS0 noswap nopcie ip=192.254.4.90::192.168.1.1:255.255.0.0::eth2:off hwaddress=eth2,00:34:CB:F4:F0:68'