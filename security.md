# Security

### Design For Security

- **Trusted Computing Base (TCB):** The TCB is the small, foundational part of the system that *must* be trusted to enforce the system's overall **security policy**. TCB components include the scheduler, memory management, and parts of the file system and device drivers.

- **Hardware Root-of-Trust:** When the device powers on, the HRoT is the first code executed. It uses cryptographic methods (like hashing and signature verification) to verify the integrity and authenticity of the next stage of the boot process (e.g., the bootloader). If the next component is verified as authentic and untampered, the HRoT hands control to it. This second component then verifies the next, and so on, creating a **Chain of Trust** that extends all the way up to the operating system and applications.

- **Writing Auditable Code:** Code must be easily read and understood by many people to be secure.

- **Sandboxing:** Running untrusted code with restricted access to other parts of the system, which reduces the possible attacks the code can make.

- **Principle of Least Privilege:** Only provide access to the resources necessary for a legitimate purpose.

- **Security Properties:** An OS should enforce:
  
  - **Confidentiality:** Private information remains private (e.g., processes can't read other processes' memory).
  
  - **Integrity:** Mechanisms are not modified without permission (e.g., OS data structures can't be modified by processes).
  
  - **Availability:** Resources are fairly accessible (e.g., network access is shared).

- **Authentication:** The act of proving a user's identity. Generally, there are three ways:
  
  - What you know (passwords).
  
  - What you have (security key, cell phone).
  
  - What you are (biometrics).

### 

### Memory Attack and Defenses

- **Buffer Overflow:** Arrays (buffers) are not bounds checked, allowing data to be written past the end of the array and overwriting data or the stack section. 

- **Return-Oriented Programming (ROP):** It works by chaining together short instruction sequences, or "gadgets," that end in a return instruction. The addresses of these gadgets are placed on the stack to hijack the program's control flow.

**Defenses:**

- **NX Bit (No-Execute):** This hardware feature allows the OS to mark memory regions as writable or executable, but not both, which stops malicious code from being written to and then executed from a data buffer.

- **Address-Space Layout Randomization (ASLR):** This probabilistic defense randomizes the memory locations of libraries and code in virtual memory. This makes it harder for an attacker to predict the arbitrary address needed to send code to.



### Speculative Execution Attacks

**Background:**

- **Speculative Execution:** Modern processors guess what the result of a conditional branch will be (branch prediction) and start executing instructions early to improve performance.

- **Timing Side Channels:** Attacks that leak internal state information based on measurable properties. 
  
  **Example (Password Comparison):** A simple comparison function might stop immediately upon finding the first mismatching character in a password.
  
  - If the attacker guesses the first character correctly, the comparison takes slightly *longer* because it moves to the second character before failing.
  
  - If the attacker guesses the first five characters correctly, the comparison takes even *longer*.










