# ArchLinux
## 安装最新版本系统
在对Arch Linux进行安装时，需要进行最新版本的安装，不然会出现各种错误。并且需要经常性的滚动更新。
## 双系统,Windows系统引导项消失解决方法
### 方法一
当您在已经安装了Linux的系统上后来安装Windows时，Windows安装程序通常会覆盖MBR或UEFI引导记录，使得 GRUB 引导加载程序无法启动，直接进入 Windows。为了重新启用 GRUB 并将 Windows 添加到启动菜单中，您可以按照以下步骤操作：
如果你使用的是windows的官方ios镜像则可以跳过第一步，因为Windows11安装程序不会覆盖MBR或UEFI引导记录。
1. 重新安装 GRUB:
使用一个可以启动的 Linux Live USB 启动系统。
打开一个终端窗口，然后使用 `chroot` 环境重新安装 GRUB。
确定您的 Linux 分区（比如 `/dev/sda2`），然后执行以下命令：
```
sudo mount /dev/sda2 /mnt  # 替换 /dev/sda2 为您的 Linux 根分区
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt
grub-install /dev/sda  # 替换 /dev/sda 为您的硬盘
update-grub
exit
```
2. 确保 `os-prober` 已安装并启用:`os-prober` 是一个工具，GRUB 会用它来检测其他操作系统。
确保 `os-prober` 是安装的，并且在 `/etc/default/grub` 文件中没有被禁用。
```
sudo apt-get install os-prober # Debian/Ubuntu 和它们的衍生版
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
3. 检查Windows引导条目:如果 `os-prober` 没有自动检测到 Windows，您可能需要手动添加一个自定义条目到 `/etc/grub.d/40_custom` 文件。
这个文件中的条目格式应该如下所示：
```
menuentry "Windows 10" --class windows --class os {
    insmod ntfs
    search --no-floppy --set=root --fs-uuid YOUR_WINDOWS_PARTITION_UUID
    chainloader +1
}
```
用 `blkid` 命令查找您的Windows系统分区的UUID并替换 `YOUR_WINDOWS_PARTITION_UUID`。

4. 再次更新 GRUB 配置:如果您手动添加了自定义条目，保存文件并运行 `sudo update-grub` 来更新 GRUB 配置。

5. 重启系统:重新启动您的计算机以查看更改是否生效。

请记住，您应该根据您的实际分区和设置替换上述命令中的 `/dev/sda`、`/dev/sda2` 和 `YOUR_WINDOWS_PARTITION_UUID`。如果您的系统使用 UEFI 而非传统 BIOS，您可能需要将 `grub-install` 替换为 `grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB`。

这些步骤通常可以恢复 GRUB 引导加载程序并将 Windows 添加到启动菜单中。如果在执行这些步骤时遇到任何问题，您可能需要根据具体的错误信息进一步调试。

### 方法二
如果你的 Arch Linux 系统已经安装了 GRUB 作为引导加载器，但它没有自动检测到你的 Windows 系统作为启动项，你可以手动添加一个引导项。

首先，确保你的 EFI 分区已经挂载。通常，EFI 分区会自动挂载到 `/boot` 或 `/boot/efi`。你可以通过以下命令检查挂载情况： 
```
lsblk -f
```
EFI 分区一般来说是挂载了的，记住EFI分区的UUID。

如果 EFI 分区没有挂载，你需要先挂载它：
```
sudo mount /dev/nvme0n1p1 /boot/efi
```
其中，`/dev/nvme0n1p1` 是你的 EFI 分区，你应该根据自己的实际情况调整。

然后，你可以按照以下步骤手动添加 Windows 启动项：

1. 编辑 `/etc/grub.d/40_custom` 文件，在其中添加 Windows 启动项的信息。你可以使用任何文本编辑器，例如 `vim`：
```
sudo vim /etc/grub.d/40_custom
```
2. 在文件的末尾添加以下内容：
```
menuentry "Windows Boot Manager" --class windows --class os {
    insmod part_gpt
    insmod fat
    insmod search_fs_uuid
    insmod chain
    search --fs-uuid --set=root YOUR_WINDOWS_EFI_PARTITION_UUID
    chainloader /EFI/Microsoft/Boot/bootmgfw.efi
}
```

请将 `YOUR_WINDOWS_EFI_PARTITION_UUID` 替换为你的 Windows EFI 分区的实际 `UUID`。你可以通过运行 `blkid` 命令来查找它。

3. `:wq`保存并关闭文件。

4. 更新 GRUB 配置：
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
5. 重新启动电脑，检查 GRUB 菜单，看看 Windows 启动项是否已经添加。

请务必谨慎操作，因为任何错误都可能导致系统启动问题。如果你对这个过程不确定，建议备份任何重要数据，并确保你有从其他介质启动的能力，以防需要修复 GRUB。

## 终端使用代理
安装`proxychains-ng`
```
sudo pacman -S proxychains-ng
```
编辑配置文件 
```
sudo vim /etc/proxychains.conf
```
指令前加proxychains
```
proxychains ping www.google.com
```
还有一种方法，比如你使用的内核是clash，则执行
```
export https_proxy=http://127.0.0.1:7897
export http_proxy=http://127.0.0.1:7897
export all_proxy=http://127.0.0.1:7897
```
## 系统半英文半中文解决方法
编辑`~/.config/plasma-localerc`文件
```
sudo vim ~/.config/plasma-localerc
```
编辑为
```
[Formats]
LANG=zh_CN.UTF-8

[Translations]
LC_NUMERIC=zh_CN.en_US
```
