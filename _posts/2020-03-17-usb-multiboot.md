---
title: "Windows PE + Linux Live CD，把U盘做成双系统启动盘"
categories:
  - 技术
tags:
  - Windows
  - Linux
  - GRUB
  - UEFI
---

继[用U盘安装Windows](/articles/fix-hdd-bad-sectors/)之后，由于恰饭需要，
又得在自己电脑上装个Ubuntu跑ROS。
偏偏家里找不到多余的U盘，为了最大限度地利用空间，
遂决定将Ubuntu Live CD也安装到Windows PE那个U盘上。
在经历一系列踩坑后，才发现整个过程其实意外地简单粗暴，跟普通硬盘上安装双系统并无本质区别。
下面以微PE与Ubuntu为例，说明一下大致的流程。

本文操作基于微PE v2.0 64位、GRUB for Windows v2.02、Ubuntu Live CD Desktop 18.04.4 64-bit，
且没有写出详细的操作。实际应用时请根据自身情况调整。
{: .notice--info}

## 用UEFI启动模式安装微PE

为什么要先安装Windows PE呢？As the saying goes，会哭的孩子有奶吃，
Windows作为最目中无人自以为是的操作系统之一，自然需要优先处理。

![weipe_install_uefi.png](https://i.loli.net/2020/03/17/Og8lD7EpRyHhqTG.png)

注意安装方法里要选支持UEFI的。
分区格式选FAT32是因为后续Ubuntu ISO会放在这里，而直接从ISO启动要求FAT32分区。
如果你打算将来在U盘的大容量分区里存放大文件，那这里可以改成NTFS，
后续再用Disk Genius之类的工具将U盘的EFI分区扩容，以存放Ubuntu ISO。

安装完成后，默认的bootloader是用于启动微PE的，为了避免后续安装GRUB时被覆盖掉，
将EFI分区下的`/EFI/BOOT/bootx64.efi`拷贝一份到`/EFI/WEPE/bootx64.efi`。

## 安装GRUB到U盘

操作步骤跟Linux下差不多，首先到[这里](https://ftp.gnu.org/gnu/grub/)下载GRUB for Windows，
解压并执行[^1]：

```powershell
./grub-install.exe --boot-directory=E:\ --efi-directory=E: --removable --target=x86_64-efi
```

这里假设`E:\`是U盘EFI分区的盘符。这个分区有时可能拿不到盘符，
不要着急，先用磁盘管理或Disk Genius看看能不能分配一个，要是还不行就重启，一般都能解决问题。

## 修改GRUB配置文件

废话少说，关键的两条menuentry：

```
insmod search_fs_file
insmod exfat
insmod ntfs
insmod fat
insmod part_gpt
insmod part_msdos
search --no-floppy --set=isopart --file /.DATA_PARTITION

menuentry "Windows PE (WePE)" {
    insmod chain
    search --no-floppy --set=root --file /WEIPE
    chainloader ($root)/EFI/WEPE/bootx64.efi
}

menuentry "Ubuntu 18.04.4 Desktop ISO - Live CD" {
    set isofile="/BOOTABLE_ISOs/ubuntu-18.04.4-desktop-amd64.iso"
    loopback loop ($isopart)$isofile
    linux (loop)/casper/vmlinuz boot=casper iso-scan/filename=$isofile noprompt noeject --
    initrd (loop)/casper/initrd
}

menuentry "Reboot to UEFI Firmware Setup" {
    fwsetup
}
```

这里面有几个坑：
- GRUB的`search --file`命令的搜索目标只能是分区**根目录**下的**文件**（**不能**是文件夹）；
- Ubuntu ISO需要处于FAT32分区内，否则虽然GRUB能成功引导，但vmlinuz会找不到，还是启动不了；
- 此处的内核参数仅适用于Ubuntu类系统[^2]，其余系统请自行查找文档，在我记忆中，至少Archlinux、openSUSE和Debian都支持从ISO启动。

## Reboot & Enjoy!

当然，你也可以再对GRUB做一些更细致的配置，如更换样式、字体、壁纸，记忆上次选择等等，
但个人认为，一个一年到头也未必用得上几次的启动盘，也没必要搞那么多幺蛾子了🙃


[^1]: https://www.aioboot.com/en/install-grub2-from-windows/
[^2]: https://help.ubuntu.com/community/Grub2/ISOBoot/Examples