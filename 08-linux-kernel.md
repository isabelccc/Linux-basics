# 8. Linux Kernel

## What is the kernel?

The **Linux kernel** is the core of the OS. It is:

- **Monolithic** — Handles CPU scheduling, memory management, and many operations itself.
- **Modular** — Can extend via **dynamically loaded kernel modules**.

## Main tasks

1. **Memory management**
2. **Process management**
3. **Device drivers**
4. **System calls and security**

User programs run in **user space** and access hardware/data via **system calls** (e.g. open file, allocate memory). The kernel runs in **kernel space** (kernel code, modules, drivers).

## Kernel and user space

- **Kernel space** — Kernel code, extensions, device drivers.
- **User space** — Applications (C, Java, Python, containers, etc.). They request services via system calls.

## Version and info

```bash
uname          # Prints "Linux"
uname -r       # Kernel release (e.g. 5.10.0-...)
uname -a       # All info (kernel, hostname, arch, etc.)
```
