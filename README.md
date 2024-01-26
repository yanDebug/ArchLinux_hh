# ArchLinux和Windows双系统,Windows系统引导项消失解决方法
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
