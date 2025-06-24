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
