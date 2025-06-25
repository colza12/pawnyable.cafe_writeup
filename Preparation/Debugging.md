# Debugging the kernel with gdb

For attaching the gdb to the qemu, add the following to run.sh:
```sh
-gdb tcp::12345
```
When I run it, the following error occured.
```
Kernel panic - not syncing: IO-APIC + timer doesn't work!  Boot with apic=debug and send a report.  Then try booting with the 'noapic' option.
CPU: 0 PID: 0 Comm: swapper/0 Not tainted 5.10.7 #1
Hardware name: QEMU Ubuntu 24.04 PC (i440FX + PIIX, 1996), BIOS 1.16.3-debian-1.16.3-2 04/01/2014
Call Trace:
 dump_stack+0x5e/0x74
 panic+0xed/0x289
 ? printk+0x43/0x45
 setup_IO_APIC+0x7d4/0x7f5
 ? clear_IO_APIC+0x34/0x60
 apic_intr_mode_init+0xfd/0x100
 x86_late_time_init+0x1f/0x30
 start_kernel+0x3b8/0x436
 x86_64_start_reservations+0x24/0x26
 x86_64_start_kernel+0x86/0x8a
 secondary_startup_64_no_verify+0xc2/0xcb
```
To address this error, add the noapic option to run.sh.
```sh
# -append "console=ttyS0 loglevel=3 oops=panic panic=-1 nopti nokaslr"
-append "console=ttyS0 loglevel=3 oops=panic panic=-1 nopti nokaslr noapic"
```
If we want to use pwndbg with root privileges, use the following command:
```
$ sudo gdb -x /home/username/.gdbinit
```

## Example
In this chapter, we stopped the breakpoint at commit_creds and checked the memory area pointed to by the RDI register. Let's now check the same thing using gdb in a shell with user privileges (with uid set to 1337 in cttyhack).  
Also compare the case with root privileges (uid=0) and the case with general user privileges (uid=1337 etc.) to see what difference there is in the data passed to the first argument of commit_creds.

Rewrite run.sh as follow:
```
# rootfs_updated.cpio
rootfs.cpio
```
Execute run.sh and attach the gdb to the qemu.  
Set a breakpoint at `commit_creds` (0xffffffff8106e390) and continue.
```
pwndbg> break *0xffffffff8106e390
pwndbg> continue
```
Executing commands such as `ls` in the shell causes GDB to hit a breakpoint and pause execution.  
The first argument (RDI) contains a pointer to kernel space. Inspect the memory it points to.
```
pwndbg> x/16xg 0xffff8880033d5300
0xffff8880033d5300:	0x0000053900000001	0x0000000000000539
0xffff8880033d5310:	0x0000000000000539	0x0000000000000539
0xffff8880033d5320:	0x0000000000000539	0x0000000000000000
0xffff8880033d5330:	0x000001ffffffffff	0x000001ffffffffff
0xffff8880033d5340:	0x000001ffffffffff	0x0000000000000000
0xffff8880033d5350:	0xffff8880033d5080	0xffffffff81e32b80
0xffff8880033d5360:	0xffff8880027505e0	0x0000000000000000
0xffff8880033d5370:	0x0000000000000000	0x0000000000000000
```
For reference, here is the value of RDI under root privileges.
```
pwndbg> x/16xg 0xffff8880033b6800
0xffff8880033b6800:	0x0000000000000001	0x0000000000000000
0xffff8880033b6810:	0x0000000000000000	0x0000000000000000
0xffff8880033b6820:	0x0000000000000000	0x0000000000000000
0xffff8880033b6830:	0x000001ffffffffff	0x000001ffffffffff
0xffff8880033b6840:	0x000001ffffffffff	0x0000000000000000
0xffff8880033b6850:	0xffffffff81e32b00	0xffffffff81e32b80
0xffff8880033b6860:	0xffff8880027505e0	0x0000000000000000
0xffff8880033b6870:	0x0000000000000000	0x0000000000000000
```
