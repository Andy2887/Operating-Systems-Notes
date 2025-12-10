# RAID

RAID stands for **Redundant Array of Independent Disks**. It is a technology that combines multiple physical disk drives to create a single, superior virtual disk.



The key benefits of using a RAID array are to:

- **Reduce the impact of failure** by storing data redundantly on multiple disks.

- **Increase capacity** by making multiple disks available for storage.

- **Increase throughput** by accessing data in parallel on multiple disks.

**RAID Implementation**

RAID can be implemented in two ways:

- **Software RAID:** The operating system (OS) is responsible for assembling the multiple disks into a RAID.

- **Hardware RAID:** A specialized controller card coordinates the multiple disks, presenting the interface of one disk to the OS.

**Common RAID Levels**

- **RAID 0 – Striping:** Distributes data across multiple disks (typically two or more) for twice the peak throughput. While it increases both throughput and capacity, the failure of a single disk is catastrophic, making its Mean Time To Failure worse.

- **RAID 1 – Mirroring:** Copies data onto two disks to tolerate the failure of one. It is virtually impossible to lose data unless all disks fail simultaneously. Read throughput is better than a single disk because different blocks can be read in parallel, but capacity is the same as a single disk, and cost per byte is greater.

- **RAID 4/5/6 – Parity:** These levels use parity bits to check for errors and rebuild data.
  
  - **RAID 4:** Uses one dedicated disk to store a corresponding parity chunk for a set of data chunks (a stripe). This allows it to tolerate the loss of any one disk. However, the parity disk can become a bottleneck for writes, limiting throughput.
  
  - **RAID 5 (Distributed Parity):** Distributes the parity chunks across all the disks to avoid the small-write bottleneck of RAID 4, making it the "winner in practice". It has good throughput and cost efficiency, and it can tolerate the failure of one disk.
  
  - **RAID 6 (Double Parity):** Adds a second disk and keeps two parity chunks per stripe, allowing it to tolerate the failure of two disks. This makes sense for larger arrays (typically more than 8 disks). Parity can only fix a single error, so the second parity chunk is computed differently to allow for the recovery of two failed drives.
