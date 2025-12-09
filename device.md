# Device

**The Importance of Devices:** Modern computers, especially interactive and cyber-physical systems, heavily rely on devices for input (keyboard, mouse, Lidar, camera) and output (monitor, headphones, throttle control, brakes).

The OS needs to communicate with devices quickly and efficiently.

##### Connecting to Devices

**I/O Hierarchy:** Devices connect to the CPU via a hierarchy of buses. Fast connections (like the Memory Bus) are physically close to the CPU, while slower, more flexible protocols are farther away (like the Peripheral I/O Bus: SCSI, SATA, USB).

**Buses Defined:** A bus is a common set of wires and a protocol for communication between two or more components.

A **bus** is fundamentally a **shared communication path** for data transfer within a computer system.

##### Talking to Devices

Devices expose an interface of **registers** that the OS reads and writes to.

- **Port-Mapped I/O (PMIO):**
  
  - In PMIO, each I/O device's registers (control, status, and data) are mapped to a distinct **I/O port address** in a separate address space, distinct from the memory address space.
    
    - You use the `OUT` instruction to **write** data *to* a specific I/O port address.
    
    - You use the `IN` instruction to **read** data *from* a specific I/O port address.

- **Memory-Mapped I/O (MMIO):**
  
  - Certain **physical memory addresses** are mapped to I/O devices instead of RAM.
  - Any instruction that accesses memory can access these devices.

##### Device Interactions

- **Synchronous vs. Asynchronous Events:**
  - **Polling (Synchronous):** The CPU repeatedly checks a device's **STATUS** register to see if it is ready or has completed an operation. This is simple but wastes CPU time for slow devices.
  - **Interrupts (Asynchronous):** A device signals the CPU when an event occurs. This stops the current code, runs an **interrupt handler**, and then resumes the original code.
- **Programmed I/O (PIO) vs. Direct Memory Access (DMA):**
  - **Programmed I/O (PIO):** The CPU is involved in reading or writing **every byte** of data to/from the device's data registers using normal memory accesses. This is time-consuming for large transfers.
  - **Direct Memory Access (DMA):** A hardware **DMA controller** is used to handle large data transfers between the device and main memory **without constant CPU involvement**. The CPU only programs the DMA controller with the address, size, and control, and is notified with an interrupt when the transfer is complete.

### Device Driver

A **device driver** is a specialized piece of system software that acts as a **translator** or **interface** between a computer's operating system (OS) and a specific hardware device.

![image](file:///Users/yuanliheng/Desktop/CS343/Lecture%20Notes/assets/Screenshot%202025-11-25%20at%2011.43.39 AM.jpg?msec=1765246323709)

##### Application Layer

Applications interact with devices primarily through **system calls**.

- **Asynchronous I/O (AIO)**
  - **Synchronous I/O:** Calls like `read/write` **block** the process until the operation is complete.
  - **Asynchronous I/O:** Calls (`aio_read`/`aio_write`) enqueue the request and allow the process to continue execution (**non-blocking**). The process later checks the status (`aio_error`) and retrieves the result (`aio_return`).

##### I/O Subsystem

- **Manages Permissions**
- **Routes Calls** to the appropriate driver.
- **Schedules Requests** to drivers (e.g., determining the order of disk requests).
- **Handles Process Memory:**
  - **Address Translation:** Translating user-provided virtual addresses to physical addresses, or copying data.
  - **Buffering:** Holding a copy of data in the kernel, especially for asynchronous operations.

##### Device Driver

The device driver is the **device-specific code** that communicates directly with the hardware.

- **Hardware Communication:** Uses **Port-mapped I/O** or **Memory-mapped I/O** (MMIO).

- **Interaction Design:**
  
  - **Programmed I/O (PIO):** CPU does the job.
  - **Direct Memory Access (DMA):** CPU lets DMA do the job.

- **Two "Halves" of a Driver**:
  
  - **Top Half (The Setup):** The OS calls it. It prepares the device. It sets up the buffers, configures the registers, and tells the hardware: "Go do this job."
  
  - **Bottom Half (Interrupt Handler):** The Hardware calls it. This code runs immediately when the device finishes its job. It is high-priority.
  
  - **Summary of the Flow:**
    
    1. **Top Half:** "Hey Hard Drive, fetch file X." (CPU goes to sleep or does other work).
    
    2. **...Time passes...** (DMA moves data).
    
    3. **Bottom Half:** "Interrupt! The data is here! Wake up!"

- **Virtualization:**
  
  1. **Illusion:** The driver tricks every application into thinking it has the *entire* device to itself.
  
  2. **Multiplexing:** When multiple programs send requests at the same time, the driver intercepts them. It puts them in a **queue**.
  
  3. **Scheduling:** The driver decides the most efficient order to execute them.
