# Operating Systems Notes

A comprehensive collection of notes covering operating systems concepts.

## 📚 Study Guide

### 1. **Foundation: Process Management & Scheduling**
Start here to understand how operating systems manage processes and CPU time allocation.

- **[Scheduling](scheduling.md)**
  - Context switching fundamentals
  - Batch systems, interactive systems, and real-time systems
  - Scheduling algorithms (FCFS, SJF, Round Robin, Priority Scheduling)
  - Multi-level feedback queues
  - Real-time scheduling (Rate Monotonic, Earliest Deadline First)

### 2. **Concurrency & Synchronization**
Build on scheduling knowledge to understand how processes coordinate and share resources.

- **[Concurrency](concurrency.md)**
  - Critical sections and race conditions
  - Lock mechanisms (Spinlocks, Mutexes, Semaphores, Monitors)
  - Deadlock detection and prevention
  - Producer-consumer problem
  - Reader-writer problem
  - Thread-safe programming

### 3. **Memory Management**
Learn how operating systems provide memory abstraction and manage physical memory.

- **[Virtual Memory](virtual_memory.md)**
  - Segmentation and segment tables
  - Paging and page tables
  - Translation Lookaside Buffer (TLB)
  - Multi-level page tables
  - Memory protection and bounds checking
  - Page replacement algorithms
  - Working sets and thrashing

### 4. **Storage & Persistence**
Understand how operating systems manage persistent data storage.

- **[File Systems](file_systems.md)**
  - Disk partitions and file system structure
  - Free space tracking with bitmaps
  - File allocation methods (linked lists, allocation tables)
  - Directories and metadata management
  - File system reliability and journaling

### 5. **Hardware Interaction**
Discover how operating systems communicate with peripheral devices.

- **[Devices](device.md)**
  - I/O hierarchy and bus architecture
  - Port-Mapped I/O (PMIO) vs Memory-Mapped I/O (MMIO)
  - Device drivers and interrupts
  - DMA (Direct Memory Access)
  - Polling vs interrupt-driven I/O

### 6. **Advanced Topics**

#### **[Virtualization](virtualization.md)**
Explore how multiple operating systems can run on a single machine.
- Virtual machines and emulation
- Hypervisors (Type 1 and Type 2)
- Containers vs VMs
- Resource allocation in virtualized environments

#### **[Security](security.md)**
Learn security principles and mechanisms in operating systems.
- Trusted Computing Base (TCB)
- Hardware Root-of-Trust and Chain of Trust
- Authentication methods
- Sandboxing and isolation
- Security properties: Confidentiality, Integrity, Availability
- Principle of Least Privilege

#### **[Embedded Systems OS](embedded_systems_os.md)**
Understand operating systems designed for resource-constrained devices.
- Differences from general-purpose OS
- Single-purpose focus and combined kernel-application design
- Cooperative vs preemptive multitasking
- Real-time constraints and memory limitations