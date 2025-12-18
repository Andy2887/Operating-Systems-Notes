# Embedded Systems OS

## Why Embedded Systems Need Their Own OS

Embedded systems require their own operating systems because normal Linux cannot be used. Reasons include:

1. A typical Linux kernel alone can require tens to hundreds of megabytes of RAM. Many **microcontrollers** (MCUs) or **small microprocessors** (MPUs) in embedded devices might only have a few megabytes of RAM or even just kilobytes of memory.

2. Desktop Linux is optimized for high throughput and relies on fast, multi-core processors. Embedded systems often use slower, low-power CPUs that cannot handle the overhead of a full-featured general-purpose kernel, complex process scheduling, and I/O management.

3. A full Linux kernel constantly manages many processes and services, even when the system appears idle, leading to unnecessary power consumption. And power consumption is a critical design constraint for most embedded devices.

## Typical Embedded OS Design

### Single-Purpose Focus

The **core assumption** in traditional embedded design is that the device has one single, primary function (e.g., a microwave controller, a remote sensor, or a simple medical monitor).

### Combined Kernel and Application

The application code and the OS kernel code are **compiled together** into one program. When the device boots, this single program begins execution. In this model, the **application logic is dominant**. The kernel is often reduced to a minimal set of services needed to support the application.

### Cooperative Multitasking

The single application is typically broken down into **multiple tasks** (sometimes called threads) that appear to run simultaneously. These tasks must **voluntarily yield** control to the OS scheduler so another task can run. If one task hangs or runs too long without yielding, it can block the entire system.

This approach simplifies the kernel design by **removing complex pre-emptive scheduling logic**. The system is designed this way because all tasks belong to the same trusted application and are presumed to **cooperate** for the good of the system.

### Minimalist Kernel with Drivers

The OS kernel itself is stripped down to the bare essentials required for the specific hardware and application.

The largest functional part of the kernel is usually the **hardware drivers**—code that directly controls the hardware.

### No Protection and Minimal Resource Management

The most significant difference from general-purpose OSes is the lack of protection.

- **No Memory Protection:** In many older or simpler embedded systems, there is **no memory management unit (MMU)** or **memory protection unit (MPU)**. This means all code (kernel and application tasks) runs in the same memory space. If one part of the application code has a bug (e.g., writing data to the wrong memory address), it can **corrupt the kernel** or the data of any other task, causing the entire device to crash.

- **Minimal Resource Management:** Since there is only one trusted application, sophisticated resource management (like dynamically allocating and reclaiming memory or enforcing CPU time limits) is often minimal or non-existent. The system relies on the application designer to statically partition and manage resources correctly.
