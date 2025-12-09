# Virtualization

### Virtual Machine

A **Virtual Machine (VM)** is an emulated version of a physical computer.

### Emulation

An **emulator** is a piece of software or hardware that enables one computer system (the **host**) to imitate the functions of another computer system (the **guest**) so that the host can run the guest's software.

Real world analogy: an emulator is like a translator.

![image](file:///Users/yuanliheng/Desktop/CS343/Lecture%20Notes/assets/Screenshot%202025-11-29%20at%204.03.08 PM.jpg?msec=1765246348687)

### Hypervisors

A **hypervisor** is a layer of software that creates and runs **virtual machines (VMs)** by abstracting a host machine's resources and allocating them to the guest operating systems.

**Hypervisor versus Emulator**

<u>Emulator translates all code that runs</u>

<u>Hypervisor is only involved in “tricky” operations</u> • Normal application code runs right on the hardware
• Normal OS code also runs right on the hardware
• Privileged code traps into the hypervisor to handle it

### Container

The container engine provide each application with illusion of its own dedicated OS.

![image](/Users/yuanliheng/Desktop/CS343/Lecture%20Notes/assets/Screenshot%202025-11-29%20at%204.01.23 PM.jpg)
