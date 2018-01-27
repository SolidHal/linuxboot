# linuxboot
The LinuxBoot project allows you to replace your server's firmware with Linux.

Supported server mainboards
===
* qemu emulated Q35 systems
* [Intel S2600WF](https://trmm.net/S2600wf)
* [Dell R630](https://trmm.net/NERF)
* Winterfell Open Compute node
* Monolake Open Compute node

Build instructions
===
You need to provide:
* The vendor UEFI firmware for the mainboard
* A Linux kernel built with the `CONFIG_EFI_BDS` option enabled
* An `initrd.cpio` file with enough tools to `kexec` the rest of the system.

For the `initrd`, the [Heads firmware](http://osresearch.net/) or
[u-root](https://github.com/u-root/u-root) systems work well.
Both will build minimal runtimes that can fit into the few megabytes
of space available.

Configure the build system:

    make \
    	BOARD=s2600wf \
    	KERNEL=../path/to/bzImage \
    	INITRD=../path/to/initrd.cpio.xz \
    	config

This will write the values into the `.config` file so that you don't
need to specify them each time.

For everything except qemu, you'll need to copy the vendor ROM dump
to `boards/$(BOARD)/$(BOARD).rom`.  Due to copyright restrictions, we can't
bundle the ROM images in this tree and you must supply your own ROM from
your own machine.  qemu can built its own ROM from the `edk2` tree,
so this is not necessary.

Now run `make` and if all goes well you will end up with a file in
`build/$(BOARD)/linuxboot.rom` that can be flashed to your machine.

For qemu you can launch the emulator by running:

    make run

This will use your current terminal as the serial console, which
will likely mess with the settings.  After killing qemu by closing
the window you will need to run `stty sane` to restore the terminal
settings (echo is likely turned off, so you'll have to type this in
the blind).


Adding a new mainboard
===

Copy `Makefile.board` from one of the other mainboards and edit it to match
your new board's ROM layout.  The qemu one is not the best example since it has
to match the complex layout of OVMF; most real mainboards are not this messy.

You'll need to figure out which FVs have to be preserved, how much space
can be recovered from the ME region, etc.  The per-board makefile needs
to set the following variables:

* `FVS`: an ordered list of IFD, firmware volumes and padding
* `linuxboot-size`: the final size of the ROM image in bytes (we should verify this against the real ROM instead)


More info
===
* https://www.linuxboot.org/
* https://trmm.net/LinuxBoot_34c3

