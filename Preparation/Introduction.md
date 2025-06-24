# Introduction to kernel exploits
Example
```
# wget https://pawnyable.cafe/linux-kernel/LK01/distfiles/LK01.tar.gz
# tar -xzvf LK01.tar.gz
# cd LK01/qemu
~LK01/qemu# mkdir root
~LK01/qemu# cd root; cpio -idv < ../rootfs.cpio
~LK01/qemu/root# find . -print0 | cpio -o --format=newc --null > ../rootfs_updated.cpio
```
Edit run.sh rootfs.cpio → rootfs_updated.cpio
