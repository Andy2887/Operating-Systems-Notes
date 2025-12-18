# RAID

RAID stands for **Redundant Array of Independent Disks**. It is a technology that combines multiple physical disk drives to create a single, superior virtual disk.

The key benefits of using a RAID array are to:

- **Reduce the impact of failure** by storing data redundantly on multiple disks.

- **Increase capacity** by making multiple disks available for storage.

- **Increase throughput** by accessing data in parallel on multiple disks.

## RAID Implementation

RAID can be implemented in two ways:

- **Software RAID:** The operating system (OS) is responsible for assembling the multiple disks into a RAID.

- **Hardware RAID:** A specialized controller card coordinates the multiple disks, presenting the interface of one disk to the OS.

## Common RAID Levels

- **RAID 0 – Striping:** RAID 0 splits (stripes) data evenly across two or more disks. It does not provide redundancy. If you write a file, the system breaks it into blocks and writes block A to Drive 1, block B to Drive 2, and so on simultaneously.

- **RAID 1 – Mirroring:** RAID 1 writes the exact same data to two (or more) drives simultaneously. If you have two 1TB drives, your total storage is still only 1TB, but your data is identical on both.

### The Core Concept: What is Parity?

Before looking at the levels, it helps to understand parity. Parity is a calculated value (usually using the **XOR** operation) used to reconstruct data.

- If you have **Disk A** and **Disk B**, and you store their sum on **Disk C** (Parity), you can lose *any* one of those three disks and figure out what was on it by doing the math with the remaining two.

### RAID 4: Dedicated Parity Disk

**"Striped data, centralized parity."**

RAID 4 stripes data across multiple disks (like RAID 0) but reserves one specific physical disk exclusively for parity data.

- **Architecture:** If you have 4 disks, data is written to Disks 1, 2, and 3. The parity information for *all* those writes is stored on Disk 4.

- **The Bottleneck:** Because every single write operation (no matter which data disk it goes to) requires updating the parity, Disk 4 becomes a massive bottleneck. The parity drive works much harder than the others, limiting write speeds and wearing that drive out faster.

- **Status:** Obsolete. It has largely been replaced by RAID 5, which solves the bottleneck issue.

### RAID 5: Distributed Parity

**"Striped data, rotating parity."**

RAID 5 improves on RAID 4 by eliminating the dedicated parity disk. Instead, it spreads (distributes) the parity blocks across **all** the drives in the array.

- **Architecture:**
  
  - **Stripe 1:** Data is on Disks A, B, C; Parity is on Disk D.
  
  - **Stripe 2:** Data is on Disks A, B, D; Parity is on Disk C.
  
  - **Stripe 3:** Data is on Disks A, C, D; Parity is on Disk B.

- **Performance:**
  
  - **Reads:** Very fast (similar to RAID 0) because data is read from multiple disks simultaneously.
  
  - **Writes:** Slower than RAID 0 (because parity must be calculated), but faster than RAID 4 because there is no single "parity disk" bottleneck.

- **Redundancy:** Can withstand **1 drive failure**. If a drive fails, the controller uses the parity data on the remaining drives to mathematically reconstruct the missing data on the fly.

- **The Risk (The "Write Hole" & URE):** If a drive fails, the array enters "degraded mode." You must replace the drive immediately. During the rebuild (which can take hours or days), the other disks are under heavy load. If a second error (like a Unrecoverable Read Error) occurs during this rebuild, you lose data.

### RAID 6: Double Distributed Parity

**"Striped data, double rotating parity."**

RAID 6 is essentially RAID 5 with an "insurance policy." It calculates **two** independent parity blocks for every stripe of data and distributes them across all drives.

- **Architecture:** Similar to RAID 5, but every data stripe has two parity blocks (often called P and Q). P is usually simple XOR parity; Q uses a more complex Reed-Solomon algorithm.

- **Redundancy:** Can withstand **2 simultaneous drive failures**. You could literally pull two hard drives out of the server while it is running, and it would keep working.

- **Performance:**
  
  - **Reads:** Very fast (similar to RAID 5).
  
  - **Writes:** Slower than RAID 5. The controller has to calculate two different parity sums for every single write operation, which is computationally expensive.

- **Why use it?** With modern high-capacity drives (10TB+), rebuilding a RAID 5 array takes a long time. The risk of a second drive failing during that stress-test is real. RAID 6 protects against that specific scenario.
