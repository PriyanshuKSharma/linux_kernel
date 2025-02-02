# Custom Linux Kernel with BusyBox and `owner` Syscall

This project involves building a custom Linux kernel (version 6.6) with the addition of a custom `owner` syscall, using BusyBox for the initramfs, and running the kernel in a virtual machine (QEMU) for testing. You can also run this custom kernel setup on Windows by converting the process into a `.bat` script.

## Project Structure

```plaintext
/home/priyanshu/
├── linux_kernel/
│   └── linux-6.6/           # The Linux kernel source
├── mnt/
│   └── rootfs/              # The root filesystem for initramfs
├── rootfs.img               # Root filesystem image
└── initramfs.cpio.gz        # Initramfs archive
```

---

## 1. **Clone the Linux Kernel Source**

Clone the Linux kernel source, version 6.6, from [kernel.org](https://www.kernel.org/) or a similar repository.

```bash
cd ~
git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git linux_kernel
cd linux_kernel/linux-6.6
```

---

## 2. **Configure the Kernel**

Before building the kernel, you need to generate the `.config` file, which contains the kernel configuration.

### Using Default Configuration:
```bash
make defconfig
```

### Or Customize the Configuration:
```bash
make menuconfig
```

This will generate the `.config` file required for building the kernel.

---

## 3. **Add the `owner` Syscall**

### 3.1 **Create the Syscall Definition:**
Navigate to `kernel/sys_owner.c` and define the `owner` syscall.

```c
#include <linux/kernel.h>
#include <linux/syscalls.h>

SYSCALL_DEFINE0(owner) {
    printk(KERN_INFO "Custom Kernel by: Priyanshu K Sharma\n");
    return 0;
}
```

### 3.2 **Create the System Call Handler:**
In the same file (`kernel/sys_owner.c`), implement the syscall handler.

```c
asmlinkage long __x64_sys_owner(void) {
    printk(KERN_INFO "Custom Kernel by: Priyanshu K Sharma\n");
    return 0;
}
```

### 3.3 **Update the Syscall Table:**
Modify `arch/x86/entry/syscalls/syscall_64.tbl` by adding the syscall number and defining the `owner` syscall.

```plaintext
440    common  owner  __x64_sys_owner
```

---

## 4. **Set Up BusyBox**

### 4.1 **Install BusyBox:**
Download and build BusyBox. BusyBox is a collection of Unix utilities, useful for your initramfs.

```bash
cd ~/mnt/rootfs
wget https://busybox.net/downloads/busybox-1.35.0.tar.bz2
tar -xjf busybox-1.35.0.tar.bz2
cd busybox-1.35.0
make defconfig
make -j$(nproc)
```

### 4.2 **Copy BusyBox to the Root Filesystem:**
After building BusyBox, copy it to the appropriate location in the root filesystem (`~/mnt/rootfs/bin/`).

```bash
cp busybox ~/mnt/rootfs/bin/
ln -s busybox ~/mnt/rootfs/bin/sh
```

---

## 5. **Create Initramfs with a Custom Init Script**

### 5.1 **Create the `init` Script:**
Create an `init` script that will set up the environment and execute the `owner` syscall.

1. Create a file named `init` in the root of your initramfs (`~/mnt/rootfs/init`).
2. Write the following code into the `init` file:

```bash
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
mount -t devtmpfs none /dev
clear
echo "Welcome to Priyanshu's Custom Kernel!"
exec /bin/sh
```

3. Make the script executable:

```bash
chmod +x ~/mnt/rootfs/init
```

### 5.2 **Create Initramfs:**
Create an initramfs archive (`initramfs.cpio.gz`) using `cpio`.

```bash
sudo find ~/mnt/rootfs | cpio -H newc -o | gzip > ~/linux_kernel/linux-6.6/initramfs.cpio.gz
```

---

## 6. **Building the Kernel**

Now, you need to build the kernel and create the `bzImage`. Use the following command:

```bash
make -j$(nproc) bzImage
```

---

## 7. **Run the Custom Kernel in QEMU**

You can run the kernel and initramfs in QEMU by using the following command:

```bash
qemu-system-x86_64 -kernel ~/linux_kernel/linux-6.6/arch/x86/boot/bzImage -initrd ~/linux_kernel/linux-6.6/initramfs.cpio.gz -append "console=ttyS0 root=/dev/ram rdinit=/init" -nographic
```

This will boot the kernel with your custom initramfs and execute the `init` script.

---

## 8. **Test the `owner` Command**

Once the kernel and initramfs are up and running, you can test the custom `owner` command by running the following command inside the QEMU console:

```bash
owner
```

You should see the message: **"Custom Kernel by: Priyanshu K Sharma"**.

---

## 9. **Converting to a `.bat` File for Windows**

To run the custom Linux kernel setup in a Windows environment using WSL or QEMU, you can create a `.bat` script.

### 9.1 **Install QEMU for Windows:**
1. Download and install QEMU for Windows from [QEMU Official Website](https://www.qemu.org/download/).
2. Ensure that the `qemu-system-x86_64` executable is added to your system's PATH.

### 9.2 **Create the `.bat` Script:**

1. Open a text editor (like Notepad).
2. Write the following script:

```batch
@echo off
echo Starting the custom Linux Kernel...
qemu-system-x86_64 -kernel C:\path\to\your\kernel\bzImage -initrd C:\path\to\your\initramfs.cpio.gz -append "console=ttyS0 root=/dev/ram rdinit=/init" -nographic
pause
```

Replace `C:\path\to\your\kernel\bzImage` and `C:\path\to\your\initramfs.cpio.gz` with the actual paths to your `bzImage` and `initramfs.cpio.gz` files.

3. Save the file with the `.bat` extension, e.g., `run_kernel.bat`.

### 9.3 **Run the `.bat` Script:**

1. Double-click the `.bat` file to start QEMU and load the custom Linux kernel with the initramfs in Windows.

---

## Troubleshooting

1. **Missing `.config` File:**
   If you encounter an error about a missing `.config` file, make sure you’ve run `make defconfig` or `make menuconfig` to generate the `.config` file before building the kernel.

2. **QEMU Errors:**
   If QEMU is unable to open the kernel image or initramfs, check that the file paths are correct and the required files are present in the specified locations.

3. **Syscall Not Working:**
   If the `owner` syscall doesn’t work, check that you’ve added the syscall handler in `sys_owner.c` and updated the syscall table correctly.

---

## Useful Links

- [Linux Kernel Official Source](https://www.kernel.org/)
- [BusyBox Official Source](https://busybox.net/)
- [QEMU Official Documentation](https://www.qemu.org/docs/)
- [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/)
