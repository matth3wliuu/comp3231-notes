# Overview

**Roles of the OS**
1. Abstract Machine
    - hides hardware details &rarr; programmers to write device independent applications
    -  i.e. file system instead of registers
2. Resource mananger
    - management and allocaton of the computer's resources to processes
      - i.e. CPU, memory, disk drivers...
    - ensures progress and no starvation
 
 **Kernel**
   - portion of the OS running in previledged mode
   - loaded into main memory (RAM) and remains there
   - facilitates safe interactions between software and hardware
 
 **Structure of the OS**
   - userland: where applications are executed
   - kernel: access to all the hardware in the computer
   - systems calls exists to allows applications to interact with hardware sources

**Timer Interrupt**: process's time slice is up
  1. kernel regains control
  2. scheduler chooses the next ready process 
  3. context switch and gives control of CPU to that process

**OS Software**
  - OS functions are machine code programs with more priviledges
  - Execution of an application:
      1. OS places the application into RAM
      2. OS gives control of the CPU to the process running the application
      3. OS retrieves control when either
          - system calls are made
          - timer interupts
         
**OS Requirements** 
- Interleaving execution of processes 
    - maximise CPU ussage
- Interprocess communication
    - process A can notify process B that an event has occur via synchronisation primitives 
- Can allocate resources to applications
- Allow users to create processes

# Processes & Threads

<table border="0">
 <tr>
    <td><b style="font-size:30px">Process</b></td>
    <td><b style="font-size:30px">Thread</b></td>
 </tr>
 <tr>
    <td>execution of a program</td>
    <td>unit of execution </td>
 </tr>
 <tr>
    <td>contains threads</td>
    <td>belongs to a process</td>
 </tr>
  <tr>
    <td>owner of the resources allocated</td>
    <td>operate on the resources allocated</td>
 </tr>
</table>

**Logical Execution Trace**
- From perspective of each process, they are executed from start to finish without stoppage
  - Creates allusion that multiple process are running at the same time
  - Oblivion to the sys calls & interrupts
- Only one program is active at any instant

**Scheduler**
- Queue system responsible for choosing the next ready process to run
- Eeach event (i.e. netword traffic) has its own queue 

**Process Memory Layout** (shared amongst threads)
- text: code executed by CPU
- data: global variables (grows up)
- stack: local variables (grows down)

**User / Kernel mode**

<table border="0">
 <tr>
    <td><b style="font-size:30px">User Mode</b></td>
    <td><b style="font-size:30px">Kernel Mode</b></td>
 </tr>
 <tr>
    <td>Processes scheduled by the kernel</td>
    <td>Activity is associated with a process</td>
 </tr>
 <tr>
    <td>Isolated from each other</td>
    <td>Kernel memory is shared between processes</td>
 </tr>
  <tr>
    <td>No concurrency issues </td>
    <td>Concurrecny issues arise when process concurrently executing in a system call</td>
 </tr>
</table>

**Thread items**
- saved registers, program counter, stack pointer
- stack: local variables, return address 
- state: running, blocked, ready

**Threading Models**
1. Single Threaded
    - Can only do one thing at a time &rarr; process will blocked if no progress can be made
        - chef cooks fries and waits until its finished before moving onto burgers 
2. Multithreaded 
    - Multiple activities can be parallelised 
    - Overlap I/O with execution &rarr; CPU can always run a thread that is not blocked
        - chef on fries, chef on burger and server for customers
        - can always serve new customers even if food not ready
3. Event Model (Async)
    - Thread initiates and I/O operation to execute in the background. 
    - Immediately returns to waiting for events
    - When an operation is complete, thread is interrupted to handle the result 
   
  https://courses.cs.vt.edu/cs5204/fall09-kafura/Presentations/Threads-VS-Events.pdf

**Process Control Block (PCB)**: Data structure used to store information about a process
- used by the OS to manage and execute processes when CPU is free
- contains process id, state, registers, open files etc

**Thread Control Block (TCB)**: Data structure within the kernel which contains information about a thread
- contains thread id, state, program counter, register, pointer to a PCB, etc 

**User Level Threads**                                                                 
- Implemented in the user level using library functions
- OS has no knowledge of user level threads (it sees only a single process)
- If a thread is blocked waiting for another thread in the same process, its state is saved and the dispatched switches to a ready thread.

<table border="0">
 <tr>
    <td><b style="font-size:30px">Pros</b></td>
    <td><b style="font-size:30px">Cons</b></td>
 </tr>
 <tr>
    <td>Thread management is more performant and lightweight (library) </td>
    <td>Limited to cooperative multithreading (no timer interrupt) </td>
 </tr>
 <tr>
    <td>Threads can be synchronised without the OS</td>
    <td>Does not take advantage of multiple CPUs</td>
 </tr>
  <tr>
    <td>OS indepedent </td>
    <td>If thread makes a blocking system call, the whole process blocks</td>
 </tr>
</table>


**Kernel Level Threads**
- Implemented by the kernel and offered to applicaitons as an API
- Information about a thread is stored in kernel TCB
- Threads are managed using system calls such as create, wait, exit, etc

<table border="0">
 <tr>
    <td><b style="font-size:30px">Pros</b></td>
    <td><b style="font-size:30px">Cons</b></td>
 </tr>
 <tr>
    <td> Allows for pre-emptive multithreading </td>
    <td> Creating new threads is expensive </td>
 </tr>
 <tr>
    <td> Can overlap blocking I/O with computation </td>
    <td> Synchronisation requires crossing between user / kernel space (expensive) </td>
 </tr>
  <tr>
    <td> Can take advantage of multiprocessors </td>
    <td></td>
 </tr>
</table>

**Pre-emptive multithreading**: Timer interrupts will be automatically issued by the OS to prevent a process from holding on the CPU.

# Concurrency
  
**Concurrency**: 
  - Ability to executes parts of a program out of order without affect the final result
    - Allows parallel execution of concurrent threads &rarr; improve performance in multi-processor systems
  - No assumptions can be made about 
     1. relative progress of concurrent processes / threads
     2. divisibility of an instruction 
  - Occurs in multiple processes with a single thread OR a single process with multiple threads
  
**Critical Region**
  - Region of code where shared resources are accessed
  - Uncoordinated entry into a critical region &rarr; race condition &rarr; incorrect behaviour 

**Critical Region Requirements**
1. Mutual Exclusion (lock): allowing only one thread to access the critical region at any one time
2. Progress: no process outside a critical region can block another process
3. Bounded: all threads should eventually be able to enter their critical regions

**Disabling Interrupts** Disable before entry & renable after exit 
- Poor solution as applications can be written such that interrupts are not renabled after exit 
- Kernel may not be able to regain control 

**Atomic Instructions**
- Hardware instructions provided by the microprocessor guaranteed to be atomic
- Example: Test & Set Instruction: write to memory location & return its old value
``` C
  int key = 0;  
  void makeMattRich(int accNo, int money) 
  {
      while (testAndSet(&key, 1) == false) { };    // blocked until thread has access to key
          
      int mattHas = get_bal(accNo);
      mattHas = mattHas + money;                    // atomic
      update_bal(accNo, mattHas);
      
      testAndSet(&key, 0);                         // reset key => another process can enter 
      notifyMatt(mattHass);
  }
```
- Busy Waiting: CPU time is wasted waiting for a thread to acquire the lock
- Single core ussage &rarr; busy waiting prevents thread in the critical region to make progress

# Synchronisation Primitives

**Lock**: threads acquire / release a lock when entering / leaving a critical region
  - Spin lock (above code): use only in multi core systems **OR** if the critical region is short 
  
**Semaphore**: control concurrent access to common resources by multiple threads
  - Processes trying to access the resource that the semaphore is protecting will be placed into a waiting queue `P()` if it's not available 
  - When a resource is ready, any process waiting for that resource will be woken up `V()` 
  - Number of processes allowed to progress before blocking is determined by `semaphore.count`
   
**Monitor**: Special module that allow only one method to be inside at any time
  - All shared resources inside are private AND all code inside is mutually exclusive 
  
**Condition Variable**: signalling devices that allows threads to communicate with each other about the state of a condition
  - `cv_wait` will place a thread to sleep until a condition is met
  - `cv_signal` will wake a thread up and allow it to resume execution  

# Deadlock

**Deadlock**: A state in which no processes can progress because each process is waiting for an event that only anoher process can trigger.

**Deadlock Requirements**: 
1. Mutual Exclusion: only one process can access a resource
2. Hold & Wait: process holding a resource can request addition
3. No Pre-emption: resource previous obtained by a process cannot be taken away before completion without side effects
4. Circular Wait: two or more processes in a chain where each is waiting for a resource held by the next member in the chain

**Livelock**: A state in which processes are not blocked but never makes progress
- Two processes are trying to access the same file. However, each process surrenders the access to file if the other process has not yet accessed the file.

**Deadlock solutions**
1. Ignore Deadlocks
    - reasonable if deadlocks rarely occur and or cost of prevention / recovery is high
    - tradeoff between correctness & convenience
2. Deadlock Prevention: enforce resource allocation rules such that a deadlock condition cannot occur 
    - no ME: not plausible since some resources cannot be shared
    - no H&W: process requests all resources needed before starting &rarr; process is never waiting
        - not always possible to determine
        - holds up resources that can be used by other process until it completes
        - **variation**: surrender all resources if it's blocking another process &rarr; livelock
    - no PE: not plausible
    - no CW: can be achieved by resource ordering
        - resources must be acquired in order (`x` &rarr; `y`): if `y` is needed, `x` must be acquired first 
3. Deadlock Detection & Recovery: 
    - apply deadlock detection algorithm to determine if system is deadlocked
    - recover and restore progress by rolling back
    - adds overhead to the OS
4. Dynamic Avoidance:
    - Acquire the maximum of each resource remaining before start
        - not always possible since it's dependent on input
    - Only grant further resource requests if it leads to a safe state 

**Deadlock Detection & Recovery**
- Invariant:  <img src="https://render.githubusercontent.com/render/math?math=\sum_{i=1}^n C_{ij} + A_{j}=E_{j}">
- Detection Algorithm: GET IMAGE FROM KUAKERR

**State Safety**:
- System is currently not deadlocked AND remaining resources can be allocated in some order such that all process can complete even if they request for the maximum of a resource
- Checking state safety: Suppose all processes request the maximum of a resource, can the resources be allocated such that all processes can complete 

**Starvation**: 
- Process never receives the resource that it needs despite the resource repeatedly becoming available. The resource is always allocated to a process of higher priority.

**Branch Delay**: Instruction after a branch or jump is always executed prior to destination of jump 
```MIPS
    j       1f
    li      r2, 2           // branch delay slot
    li      r2, 3           // branch delay slot
1:  sw      r2, (r3) 
```

# Context Switch

**Definition**: Storing and restoring the state associated with a process / thread
- Essential in enabling multiple process to share the same CPU

**Causes of a context swtich**:
- System call: block or on exit()
- Exception: 
- Interrupt: timer interrupt causing the scheduler to execute another process

**Requirements of a context switch**:
- Transparent for process / threads: they should not noticing something else is running while they're dispatched
    - switching between user-level threads from the thread's perspective = sequential execution
- OS must save all state involved 

**Procedure of a context switch**:
1. Process running in user mode &rarr; sp pointing to user-level stack
2. On exception, syscall or interrupt &rarr; sp pointing to kernel stack
3. Trapframe is pushed onto the stack and sp is moved downwards 
4. Execute C code to handle exception, syscall or interrupt &rarr; C activation stack builds up (down)
5. Kernel chooses a target thread / proceess and the remaining kernel state is pushed onto the stack
6. Current sp is stored in the PCB or TCB and sp of the target thread is loaded &rarr; context switched
7. Rewind through the stack of the target (kernel state &rarr; C activation stack &rarr; trapframe &rarr; user mode)

**Trapframe**: Stores all the registers used by the process that called switch

# System Calls

**Basics**:
- Special function calls that provided controlled entry into the kernel from the user space
- Perform privileged operations (access devices, network files etc) and return user mode with result 
- Syscall interfance represents the abstract machine provided by the OS
- Regular function calls cannot enforce security checks

**Previleged Mode Operation**:                                                   
- System, kernel-mode: all instructions and registers are available                
- user-mode: only a subset of the instruction set and only safe registers   
    - affect only the application                                                        
    - cannot interfere with OS (i.e. cli on x86 can disable interrupts)                                                      

![This is an image](user-kernel-register.png)|

**System Call Procedure**:

**Coprocessor 0 (cp0)**:
- Contains exception / interrupt management registers
- Manipulated using kernel instructions `mtc0` and `mfc0`

**c0_status**:

![This is an image](c0_status.png)|
- KU: 0 = kernel and 1 = user mode
- IE: 0 = interrupts off and 1 = interrupts enabled
- KU and IE bits are shifted left per each exception
- c, p, o = current, previous, old

**c0_cause**:

![image](https://user-images.githubusercontent.com/40874612/161461515-e2169b5e-c217-4996-ae2e-33f812070fae.png)
- ExcCode = code number of the exception
    - 0 = interrupt and 8 = system call

**c0_ecp**:
- Exception program counter which points to address of where to restart execution after handling an exception

**Exception Vector**: Stores the code required to handle different exceptions

**Hardware Support for Exception Handling**:
1. Executing code stored at pc in user mode and interrupts on. Interrupts occurs.
2. Instruction at pc is stored into the epc 
3. KU and IE bits in the status register is shifted to the left so `KUc` and `IEc` are 0 (disabled)
4. Exception code is placed into c0_cause (i.e. 0 for interrupt)
5. Address of exception vector placed into pc (cpu now running code in pc in kernel mode with interrupts off)
6. HANDLES THE EXCEPTION
7. Load the address stored in the epc back in the pc and call the `rfe` instruction to shift status bits to the right
8. cpu now running the code in the same state as when the exception occurred


**Loading the epc**:
```MIPS
lw r27, saved_epc
nop
jr r27
rfe
```

# File System

**File System Abstraction**:

<table border="0">
 <tr>
    <td><b style="font-size:30px">User's View</b></td>
    <td><b style="font-size:30px">File System</b></td>
 </tr>
 <tr>
    <td> Uniform namespace (universal file sysyem) </td>
    <td> Collection of storage devices </td>
 </tr>
 <tr>
    <td> Hierarchical structure </td>
    <td> Flat address space </td>
 </tr>
  <tr>
    <td> Arbitrary file sizes </td>
    <td> Fixed sized blocks </td>
 </tr>
 <tr>
    <td> Symbolic filenames  </td>
    <td> Numerical block addresses (numbers) </td>
 </tr>
 <tr>
    <td> Access control </td>
    <td> No access Control </td>
 </tr>
 <tr>
    <td> Tools for formatting, defragmentation, backup, consistency checking </td>
    <td> </td>
 </tr>
</table>

**FS Basics**:
- Filename: convenient textual naming scheme provided by the FS
- File structure: sequence of bytes
- File types: regular files, directories and device files
- File access types:
    - sequential: every instance must read from beginning (no jumps)
    - random: bytes can be read in any order by moving the file pointer (lseek)
- File attributes: protection, owner, flags, time, size etc
- File operations: create, delete, open, ...


**File Organisation and Access**:
- OS suppors unstructured files by organising its content using the ELF format 
- Criterias for designing file layout
    - rapid acess
    - ease of update
    - storage efficiency
- Performance is limited by seek time and rotational delays
    - keep blocks likely to be accessed together close to each other


**Hierarchical Structure and Directories**:
- Directories are special files owned by the OS to provide mappings between filenames and inodes
- Files can be located by following down a path from the root through directories
- Filenames don't have to be unique as long path is unique 


**Unix Storage Stack**:
- File descriptor / Open file tables:
    - Manage / trakck files opened by user level processes 
    - Match system call interface to VFS calls
- Virtual file system (VFS):
    - Presents an unified interface to the mutliple file systems attached
- File sytem:
    - Presents a directory hierarchy that hides the physial location of data on the disk
    - Interact with the blocks provided by the device drivers to create higher abstractions
- Buffer cache: 
    - Keep recently accessed disk blocks in memory to optimise repeated access
- Disk scheduler:
    - Prioritise waiting requests that fetch blocks closer to each other on the driver 
- Device driver:
    - Hides the details of the hardware
    - Presents a sequential interface to the disk for higher software layers
- Disk controller: 
    - Abstracts the circulary structure and geometry of the disk to a linear seqeunce of blocks
    - Provides the interface between the hard-disk and the device driver  
- Hard disk: 
    - Platters containing tracks and sections 

**File System Implementation Requirements**:
- Must map symbolic filenames to a collection of block addresses on the disk
- Track which blocks belong which files
- Track the order in which files are formed from blocks
- Track which blocks are free for allocation

**Internal Fragmentation**: 
- Space wasted within the allocated memory region
- Allocated region may be bigger than memory required (size difference is wasted)

**External Fragmentation**:
- Space wasted outside the allocated memory region
- Enough memory space exist to accomodate a request however it's not usable since non-contiguous

**Trade-offs in FS block sizes**:
- Large blocks require less metadata to be stored
- Smaller blocks wastes less disk space (fewer internal fragmentation)
- Sequential access:
    - larger block size &rarr; fewer I/O operations
- Random access:
    - larger block size &rarr; more unrelated data per load

### File Allocation Methods
**1. Contiguous Allocation**:
- Tracks only the address of the starting block and the length of the file
- Increase performance for sequential operations
- Require max file system at the time of creation (cannot grow)
- Deleting files leads to  external fragmentation

**Dynamic Allocation Stratigies**:
- Disk space is allocated on demand so no pre-allocation is required
- Allocation occurs in fixed-size blocks
- No external fragmentation but potentially internal fragmentation (blocks not fully utilised)
- File blocks are scattered across the disk (reduced performance)
- May require the manipulation of complex datastructures

**1. Linked List Allocation**:
- Each block contains a pointer to the next block in the chain 
- Best for sequential access and does not offer random access
- No external fragmentation
- Files can continue to grow in size as long as there are free blocks
- Blocks end up scattered across the disk due to (allocs and frees) &rarr; decrease file access performance 

**2. File Allocation Table (FAT)**:
- Keep a map of the entire FS in a table where each entry stores the number of the next block of the file 
- Table is stored in disk and replicated in memory to provide fast random access 
- Requires lots of memory for large disk 
- Searching for a free block is slow 

**3. Inode Based FS**:
- Each file has a table that maps inodes to disk block numbers
- Only store table for open files in memory 
- Fast random access like FAT 
- I-nodes are allocated dynamically so strategies are required to find free spaces within the disk
    - Free block list: linked list of all the free blocks in disk that are stored in free blocks so no space wasted
    - Bit Table

### Directory Implementation
- Stored like normal files inside data blocks
- FS assigns special meaning to the content of these files 
    - each directory file is a list of directory entries
    - each entry contain filename, attributes and i-node (map filename to disk blocks)
    
**fixed-size directory entries**: either too small or too big (waste space)

**variable-size directory entries**: freeing directory entries can create external fragmentation

**searching directory listings**: Linear scan, hash lookup, b-tree

### Virtual File System

**Motivation**:
- Systems need to support multiple file system types however modifying system code to understand each is inconvenient
    - Provide single system call interface for different file systems 
- Provide framework that separates file system independent code (interacting with `vfs` & `vnodes`) and file system dependent code (dealing with `inode`)

**Struct vfs**:
- fs independent fields:
    - metadata: block size, max file size etc 
- fs_data:
    - inodes, superblock address etc
- vfs_ops:
    - getroot, read, write etc

**Struct vnode**:
```C
struct vnode {
    int ref_count;                  // number of references to this vnode by file descriptors
    struct spinlock countlock;      // lock to achieve mutual exclusion
    struct vfs / fs *vn_fs;         // pointer to above struct
    void *data;                     // FS specific vnode data (e.g. memory copy of inodes)
    
    const struct vnode_ops *ops;    // array of vnode functions
}
```

**File Descriptors**:
- Each open file has a corresponding file descriptor
- `vnode_ops` uses file descriptors to determine which files to operate on 
Each file descriptor contains:
    - file pointer: off_set to where the next operation (e.g. `read`) occurs at
    - mode: how the file must be used (e.g. readonly)

**FD & OF Table Designs**:
1. Vnode number as a file descriptors:
    - each number has an associated file pointer to the vnode
    - problem:
        - concurrently opening the same file system does not return 2 separate file descriptors 
    
2. Single global open file array
    - each index is a file_descriptor
    - each entry contains a file pointer and pointer to a vnode
    - problem:
        - fd 0-2 should be different per process however there is only one of each in this design
3. Per process file descriptor table
    - Each process has its own open file array and each entry contains a fp and pointer to vnode
    - problems:
        - forking produces a child process that shares the same file pointer as the parent
        - dup2 produces a new file descriptor that shares the same file pointer as the old
            - in the current design, the file pointers will not be synchronised in both cases
4. Per process file descriptor table with global open file table
    - Each process has its own file descriptor table and each entry contains 
        - read / read permissions
        - in use status
        - pointer to an open file
    - All processes share access to the global open file table and each entry contains
        - pointer to vnode
        - number of file descriptors pointing it 
        - file pointer
    - Resolves all the aforementioned problems 


**Buffer**:
- Temporary storage used to transfer data from two entities (e.g. application and disk)
- Stored in kernel RAM for performance
- Use when speed of receiving and processing data is different

**Cache**:
- Fast storage (typically RAM) used to temporarily store data to improve performance for repeated access to the same data
    - unused kernel memory space that may change in size depending on current memory demands
- Before loading a block from disk, check if it's in cache to prevent disk access and improve performance

**Buffering disk blocks**:
- read-ahead can be implemented by pre-loading the next required block into the kernel buffer
    - loading disk blocks into buffer before the application needs it
    - avoid waiting for the next read instruction  
Applications wishes to write a file to disk however its size may not match a full block
1. Application issues the request and writes to the buffer
2. Immediately return back to the program to resume execution and avoid waiting for write to complete
3. The entire block of where the write occurs is transferred into the buffer and updated
3. OS arranges for the entire block to be written back to disk in the backround
- 2 and 3 occur at the same time to improve performance 

**How buffering is related to caching**:
- Data is always read into the buffer since having another copy inside the cache is wasteful
- After use, block should be cached so future access can hit the cached copy instead

**Unix Buffer Cache**:
- On read: hash the device number and block number &rarr; check if match in buffer cache 
    - yes &rarr; used cached copy
    - no &rarr; follow the collision chain
        - if not found &rarr; load block from disk into buffer cache
- Policy must exist to be able to read in new blocks by replacing existing blocks when the buffer cache is full
    1. FIFO: does not reflect access pattern
        - first thing read in ≠ first thing not needed
    2. LRU: not strict LRU
    
 **File system consistency**:
 - When an application writes to disk, the data is expected to survive and make it to the file
 - Strict LRU will keep critical file system data in memory forever if they're frequently used
    - cached disk blocks are prioritised based of how critical they are to the file system consistency
    - usually scheduled for immediate write to disk
- Losing directory, inode blocks etc can corrupt the FS 
- Data blocks that corrupt only the file they're associated with are scheduled periodically to write back to disk (unix: 30 seconds)
 
### Ext2 File System

**Features**
- Block size (1024, 2048, 4096)
- Inode based 

**Inode**
- Each file is an inode on disk
- Each inode has an unique number and contains
    - metadata: access rights, owner, mode, time, ...
    - block index table of a file
    - size = offset of highest byte written to the file 
- Directories map filenames to inode numbers
 
 **Inode Blocks**
- Direct blocks: 
    - block numbers for the first 12 blocks
    - most files are small &rarr; all of its contents will be here
- Single indirection:
    - stores a block number of a block containing more block numbers
    - requires two disk access 
    - max file size = 268 KiB
- Double and triple indirections follows the same idea

**Inode Block Accessing Scheme**
- Find the binary offset by taking the base of each indirection level as 0
- Use the binary offset as index to traverse through the indirection levels 

**Worst Case Access Patterns for Allocated Inode**
- read 1 byte: 4 accesses in triple indirect
- write 1 byte: 4 reads and 1 write in triple indirect

**Worst Case Access Patterns for unallocated Inode**
- watch again lect11 (pg 29)

### s5fs 

**Disk Layout**
- Boot block: contains code to bootstrap (load) the OS
- Super block: contains information about the file system itself 
    - number of inodes, free block and inode list, ...

**Problems**
- inodes at the start and data blocks at the end &rarr; long seek time (travel from inode to block)
- corrupt the only superblock then the entire FS is corrupt
- suboptimal block allocation: eventually resulted in random allocation
- inode free list also randomised over time

### Berkeley Fast File System (FFS)

**Disk Layout**
- collection of equally sized and structured block groups

![image](Disk-layout-of-EXT2-file-system.png)

**Layout of a Block Group**
- superblock:
    - replicated in each group to aid recoverability &rarr; no longer vulnerable to one superblock becoming corrupted
    - overall free inode and block counters
    - metadata indicting whether system checks are required
        - unclean unmount, inconsistency, regular checks based on last unomount and time
- group descriptor:
    - replicated in each group to aid recoverability
    - location of the bitmaps
    - counters for free blocks and inodes in this group
    - number of directories 
- data block bitmap: which data blocks are free or in use
- inode bitmap: which inodes are free or in use

**Performance Improvements**
- block groups place related inodes and data blocks close to each other &rarr; less distance in disk
- pre-allocate up to 8 blocks on write &rarr; better contiguity for content written
- store files within a directory in the same group 

**Ext2fs Directories**
- special file managed by the kernel
- translate filenames to inode numbers
- deleting inode entries
    - setting inode number to 0
    - add length of current inode to length of previous inode

**Hard Links**
- same inode can have multiple names
- when deleting a file by name (removing the directory entry), the FS will delete the underlying inode ⇔ reference count = 0
- hard links cannot be made to files you don't own

**Symbolic Links**
- file that contains a reference to another file or directory
- contains inode and datablock that has a path to the target file
- marked by special file attribute
- can point across FS boundaries (i.e. access files between FS)

**File System Reliability**
- disk writes are buffered in RAM
    - OS crash / power outage &rarr; data lost
    - 3 points of failures are directory entry &rarr; inode &rarr; data blocks
    - writes are periodically committed to disk (i.e. every 30 desconds)
    - `sync` command to force a flush
- FS operations are not atomic
    - incomplete transaction &rarr; FS can become inconsistent & data may be lost
- e2fsck: scans disk after unclean shutdown & attempts to restore FS invariants
- Journalling FS
    - FS updates are kept in a journal
    - replay last journal entries upon unclean shutdown

### Ext3 File System

**Motivation**
- add journalling capabilities to ext2fs
- backwards & forward compatibility with ext2 to build upon its proven performance
- reuse most of the ext2 codebase and tools

**Journalling Approaches**
<table border="0">
 <tr>
    <td><b style="font-size:30px"> Journal FS Data Structure Updates </b></td>
    <td><b style="font-size:30px"> Journal FS Disk Block Updates </b></td>
 </tr>
 <tr>
    <td> ✅ efficient use of journal space </td>
    <td> ❌ even small updates can add a whole block to the journal </td>
 </tr>
 <tr>
    <td> ❌ individual updates are executed separately </td>
    <td> ✅ updates to the same block can be aggregated to a single update </td>
 </tr>
  <tr>
    <td> ❌ journalling layer must understand FS semantics </td>
    <td> ✅ journalling layer is FS independent(easy to implement) </td>
 </tr>
</table>

**Journal Block Device (JBD)**
- JBD is the ext3 journalling layer
- Interface
    - start: start a transation
    - update: update a disk block 
    - complete: completed transactions are buffered in RAM
    - commit: write transaction data to the journal 
    - checkpoint: flush the journal to disk
- can be stored on a block device or file &rarr; ext2 compatibility
- independent of ext3 specific data structures 
    - JBD is only concerned with journalling
    - can be used by any FS that requires journalling

**Journalling Mode**
- metadata + data
    - enforces atomicity of all FS operations
- metadata journalling
    - data blocks are written directly to disk
    - improve performance since the data does not need to be written to JDB

# Memory Management

**Memory Hierarchy**
- register, cache, main memory (RAM), electronic disk, magnetic disk, optical disk, magnetic tapes
- memory needed should be stored from the left 
- fastest to the left and cheapest to the right

**Monoprogramming**
- memory (RAM) is shared by the program and OS 
- fine if only one program is to be running and the memory required is less than memory available 
- poor CPU utilisation if there is a lot of I/O because no other process can run in the background (overlap I/O with execution)
- poor memory utilisation if a particular workflow does not require all the memory
- used in small embedded systems

![image](https://user-images.githubusercontent.com/40874612/164228333-747cb7ee-39ef-4460-9765-44d85cc62f79.png)

**Multiprogramming**
- OS aims to maximise memory and CPU utilisation &rarr; running more than one process at a time
- Workflows with high percentage of I/O wait will required higher degrees of multiprogramming to achieve high CPU utilisation

### Fixed Memory Partition

**Fixed, Equal-Sized**
- memory is divided into fixed equal sized partitions that are either free or busy
- any process that is less than the partition size can be loaded  
- applicable in small embedded system where work flow is static

<table border="0">
 <tr>
    <td><b style="font-size:30px"> Pros </b></td>
    <td><b style="font-size:30px"> Cons </b></td>
 </tr>
 <tr>
    <td> easy to implement </td>
    <td> internal fragmentation &rarr; poor memory utilisation </td>
 </tr>
 <tr>
    <td> minimal OS overhead for processing where programs can be loaded </td>
    <td> process larger than the partition size cannot run </td>
 </tr>
</table>

**Fixed, Variable-Sized**
- memory is partitioned at boot time into various sizes based on the expected workload
    - work load is sometimes unpredictable 
- each partition has a queue and processes are placed into the queue for the smallest partition they fit in

<table border="0">
 <tr>
    <td><b style="font-size:30px"> Pros </b></td>
    <td><b style="font-size:30px"> Cons </b></td>
 </tr>
 <tr>
    <td> reduce internal fragmentation </td>
    <td> added complexity to implement </td>
 </tr>
 <tr>
    <td>  </td>
    <td> partitions may be empty even if other partition have processes in the queue </td>
 </tr>
 <tr>
    <td>  </td>
    <td> process needs to wait until it's their turn in the queue </td>
 </tr>
</table>

**Alternative Queuing Strategy**
- single queue and when a partition becomes idle, any process that fits will be loaded (i.e. small process in large partition if needed)
- trades internal fragmentation for maximum memory utilisation

**Dynamic Memory Partition**
- variable length partitions are allocated on demand from the pool of free memory 
- processes are allocated exactly what it needs
- results in external fragmentation in the free memory range that will be too small for process 

**Requirements of dynamic partition allocation algorithms**
- minimise external fragmentation
- minimise memory overhead of bookkeeping 
- efficiently support merge adjacent free partitions into a larger one 

**Classic Approach**
- available memory is tracked using a linked list 
- each node storess the base address and size to denote the range 
- nodes are stored in ascending order of address to simplify merging adjacent memory ranges
- nodes are stored inside the free memory range itself so it not wasting space

### Dynamic Partitioning Placement Algorithm

**First-fit Algorithm**:
- scans the linked list for the first entry that fits
- if the available area is bigger than required then break it up into allocated and free 

<table border="0">
 <tr>
    <td><b style="font-size:30px"> Pros </b></td>
    <td><b style="font-size:30px"> Cons </b></td>
 </tr>
 <tr>
    <td> finds a match very quickly </td>
    <td> allocation is baised towards the front end of the memory </td>
 </tr>
 <tr>
    <td> easy to implement </td>
    <td> larger blocks tends to be preserved at the end of memory </td>
 </tr>
</table>

**Next-fit Algorithm**
- modified first-fit where it begins searching from where the last request succeeded 
- motivation: spread allocation more evenly throughout the entire memory region to avoid skipping over small holes at the start 
- in practice, the large free space at the end will be broken up into smaller holes and eventually lead to worse external fragmentation than first-fit

**Best-fit Algorithm**
- complete search through the list to find the smallest block that fits the request
- since the smallest possible block is chosen, over time there will be many unuseable external fragmentation regions

**Worst-fit Algorithm**
-  complete search through the list to find the largest available block
-  motivation: to leave useable fragmentation left over 
-  complete searches through the list but does not result in significantly less fragmentation

**Compaction**
- process to reduce external fragmentation by recombining all free memory into a single large block
- relocating programs using pointers may lead to addresses in the free region after compaction being pointed to 
- generally require hardware support that preserves everything used by a program such that all addresses being referred will point to the correct things after relocation

**Logical Addresses & Memory Binding**: 
- logical address: specific locations within a program and they must refer to physical memory during execution

<table border="0">
 <tr>
    <td><b style="font-size:30px"> When does logical addresses become bounded to physical addresses </b></td>
 </tr>
 <tr>
    <td> ❌ compile / link time: compiler will then need to know where the program will be executing at compile time and it needs to recompile if location changes </td>
 </tr>
 <tr>
    <td> ❌ load time: compiler can simply generate relocatable code that will be bounded to physical addresses by the loader. </td>
 </tr>
 <tr>
    <td> ✅ runtime: translation of compile-time address to physical address is handled by special hardware  </td>
 </tr>
</table>

- binding at load time reduces the performance of loading and prevents executable code from being shared between applications running the same program since the code is modified everytime it's loaded

### Hardware Support for Runtime Binding & Protection 

**Running processes in logical addresses**
- for a process `P` to run using logical addresses (i.e. `0x0000` to some limit)
    - an appropriate offset must be added to the current base 
        - achieve relocation & protection memory lower than `P`
    - must enforce a upper boundary to limit the maximum logical address `P` can generate
        - protect memory above `P`

**Base and Limit Registers**
- restrict and relocate the currently active process 
- base and limit registers must be changed at load time, relocation or context swtich to allow a different process to run

<table border="0">
 <tr>
    <td><b style="font-size:30px"> Pros </b></td>
    <td><b style="font-size:30px"> Cons </b></td>
 </tr>
 <tr>
    <td> supports protected multi-processing </td>
    <td> physical memory allocation must be contiguous </td>
 </tr>
 <tr>
    <td>  </td>
    <td> entire process must be in memory </td>
 </tr>
 <tr>
    <td>  </td>
    <td> Does not support partial sharing of address space (i.e code, datastructures, etc) </td>
 </tr>    
</table>

**Timesharing & Swapping**
- require more than just a small number of processes running in main memory at once 
- need to support a mix of active & non-active process of varying longevity
- backing store: fask disk large enough to accomodate copies of all memory images for all users
- processes can be temporarily swapped out from main memory to a backing store and later brought back into memory to continue execution
- low priority (idle) processes are swapped out so higher priority processes can be loaded and executed
- total transfer time is proportional to the amount of memory swapped (usually slow)


# Virtual Memory

**Motivation**
- virtual memory was developed to address the issues of previous approaches of managing multiprocessing
- external fragmentation, protection, swapping and execution of processes larger than main memory

**Paging Overview**
- process' physical memory does not have to contiguous 
- frames: 
    - small equal sized partitions of the physical address space
- pages:
    - small equal sized partitions of a process' virtual (logical) address space 
- page table:
    - data structure managed by the OS that stores the relationship between virtual addresses and physical memory 
    - contains the frame location (physical) of every page (virtual) 
    - hardware is used to translate virtual addresses to physical addresses
- no external fragmentation 
- small internal fragmentation on the last page because
- allows sharing between processes as multiple pages can map to the same frame
- programmers now only have to deal with virtual addresses and not have to worry about where it will be stored in physical memory

**Virtual Address Space**
- the 0th page of every virtual address space is not used to denote NULL 
- a process may only be partially resident so the OS can store pages that are not required for execution on disk and save memory

**Page Faults**
- applications / OS referencing an invalid page triggers a page fault (exception)
- illegal address &rarr; signal or kill the process for protection
    - i.e. dereferencing null
- page not resident 
    - get an available frame, load the page from disk, update page table, load the tlb and restart the faulting instructions 

**Shared Pages**
- single copy of the code can be shared between all processes executing it 
- code must not be self-modifying and lead to inconsistencies
- code must appear at the same virtual address for all processes

**Address Translation**
- programs uses virtual addresses
- TLB translates the virtual address to physical address
- first check the valid bit
    - yes &rarr; entry_lo = page table entry
    - no &rarr; page fault 
- most significant bits are used to index into the page table
- least significant bits are directly in the physical address with no translation
    - each page = 4096 bytes so, 12 bits are enough to store every possible offset 


### Translation Lookaside Buffer (TLB)

**Reloading the TLB**
- given a virtual address, the processor checks the TLB
- if pte is found &rarr; TLB hit &rarr; address is translated
- else, TLB miss &rarr; the virtual address (page number) is used to index into the page table
    - if page table contains a valid entry &rarr; reload the TLB and continue
    - else, page fault and check if the page is on disk
        - if yes, swap it in
        - else, allocate a new frame in page table or raise exception

**TLB Properties**
- TLB holds a subset of the page table entries 
- access is by associative lookup using the virtual page number `{ v_page_number : p_frame_number }`
- can be hardware or software loaded 
- typically have 64 - 128 entries

**TLB & Context Switching**
- TLB is a shared piece of hardware and TLB entries are process specific 
- TLB needs to be flushed on context switch to invalid all entries &rarr; high context switch overhead
- OR TLB entries are tagged with the address space ID (ASID)

### Page Table

**Page Table Structure**
- an array of frame numbers indexed by page number
- each page table entry contains 
    - caching bit: set if the cpu should bypass the cache to ensure data immediately arrives at the destination
    - modified bit: set if changes were made 
    - referenced bit: set if paged was accessed 
    - protection bits: indicate read, write, execute permissions
    - valid bit: to indicate if the translation is valid
    
![image](https://user-images.githubusercontent.com/40874612/164355586-6eb92df8-933b-4070-b1d6-e4d20f55f803.png)

**Problems**
- for a 64 bit architecture, there would be 2^(64-12) page table entries for every process &rarr; too big
- page tables are implemented as data structures in main memory so, a compact representation is required to preserve space but it also needs to be fast to search 

**Two Level Page Table**
- initially set all first level entries to null and only allocate if it's needed 
- given a 32 bit virtual address and 10-10 design, 
    - first 10 bits are used to index into the first level of page table
    - next 10 bits are used to index into the second level of the page table to retrive the page table entry (frame number)
    - remaining 12 bits are the offset
- get the frame from the physical memory by `frame number | offset`

### Inverted Page Table (IPT)
- array of page numbers (physical frames) indexed by frame number 
1. hash the virtual page number (first 20 bits) to produce an index in the hash table
2. extract the index and use it to index into the IPT 
3. match the PID and virtual page number 
    - yes &rarr; use index value as frame number for translation
    - no &rarr; check next candidate using the next index from the current entry
4. null table entry &rarr; page fault

![image](https://user-images.githubusercontent.com/40874612/164362537-9bfd06aa-9d69-466d-9926-e417e6ac9765.png)

- size of IPT is proportional to the size of RAM since it stores all the physical frame that back virtual addresses
- saves a significant amount of space since only one of copy of the page table is needed 
- frames cannot be shared between processes since each physical frame number has only one corresponding PID and virtual page number value

### Hashed Page Table (HPT)
- improvement upon IPT by supporting sharing 
- same fast lookup as IPT
- page table sized based on physical memory size 

![image](https://user-images.githubusercontent.com/40874612/164363448-e5079680-2ef2-4e71-9d7b-9b9450e64020.png)
- both processes 0 and 1 are able to map their virtual page number to physical frame number `0x42`

**Virtual Memory Performance**
- Each virtual memory reference can result in two physical memory accesses
    - one to fetch the page table entry and one to fetch / store the data
    - poor performance if the page table must be consulted on every memory reference (i.e. getting an instruction)
- TLB is used to stored the recently used page table entries to avoid frequently consulting the page table 
- Assuming 99% TLB hit ratio, average number of physical memory access = 0.99 * 1 + 0.01 * 2 = 1.01


### Amdahl's Law
**Law of deiminishing returns**
- overall performance improvement is limited by the fraction of time an enhancement can be used 
- speedup = executionTimeWithoutEnhancement / ExecutionTimeWithEnhancement &rarr; make the common case fast 


### MIPS R300 TLB 

**Address Space Layout**
- kseg0: 
    - 512 megabytes of cacheable kernel mode memory of where kernel code and data is stored
    - region of `0x80000000` to `0x9fffffff` and memory inside is directly translated to physical address by substraction 
- kuseg:
    - 2 gigabytes of user and kernel accessible virtual memory that stores where the stack, code, data, etc of an application
    - context swtich involes switching the page table for kuseg 
    - region of `0x00000000` to `0x80000000` that is translated by the TLB
    - cacheable depending on the `N` bit and 4K page size 
- kseg1:
    - 512 megabytes of non-cacheable kernel mode memory used to access device registers and frame buffers
    - region of `0xA0000000` to `0xC0000000` and memory inside is directly translated to physical address by substraction 
- keg2:
    - TLB translated memory used by the OS if it wants to use virtual memory techniques

**R3000 TLB Handling**
- TLB refill is handled by software 
- TLB refills are expected to be frequent &rarr; CPU is optimised by having a special exception handler just for TLB refills
- Exception vector specifies code at `0x80000000` is copied into kseg0 to handle TLB refills


**TLB & c0 Registers**
- TLB entries can be manipulated using the `c0_entryHi`, `c0_entryLo` and `c0_index` registers
- there are special instructions like `TLBR`, `TLBP`, `TLBWR`, `TLBWI` to achieve the above 


### Demand Paging
- with virtual memory, only parts of the program image need to be resident in memory 
- unused pages can be transferred to disk and reloaded on demand
    - reload is triggered by page fault
    - if a faulting process is blocked, another is scheduled to execute
    - may require a current resident page to be evicted 
    - if the victim is unmodified (indicated by the dirty bit), then it can simply be discarded

**Why Demand Paging**
- programs only need access to its resident set in order to execute at full speed
- if the page it needs is not resident then TLB miss and page faults are required and they introduce delays of microseconds and milliseconds respectively
- using demand paging, less physical memory is required per process so more processes can fit in memory resulting in a higher chance of finding a runnable one
- principle of locality

**Principle of Locality**
- programs tend to reuse data and instructions they used recently (90/10 rule)
- exploiting this locality of references, the instructions and data a program will use in the near future can be reasonably predicted based on its access in the recent past
- temporal locality: recently access items are likely to be accessed again in the near future
- spatial locality: items close to one another in memory tend to be referenced close together in time 

<img width="591" alt="image" src="https://user-images.githubusercontent.com/40874612/164591947-231e753c-8b2a-42ac-9023-4c19889aa7c0.png">


**Working Set**
- working set: pages required by an application in a time window Δ in its memory
- working set with the appropriate Δ can approximate a program's locality
- system should at least keep a process's working set resident 

**Thrashing**
- CPU utilisation tends to increase with the degree of multiprogramming which leads to less memory available for each process
- as a result, some process's working set can no longer fit in RAM &rarr; more page faults required to fetch the pages required to continue execution
- eventually Σ working set sizes > total physical memory and many processes will have insufficient memory
    1. can't find runnable process
    2. decrease CPU utilisation
    3. system becomes I/O limited 
- recovery from thrashing occurs when the system notices increasing page faults and decreasing CPU utilisation
    - suspend process to reduce multiprogramming
    - pages from suspended processes will be moved from resident memory to backing store
    - more physical memory becomes available &rarr; less faults &rarr; faster progress for runnable processes 
    - suspended processes are resumed when memory pressure eases

### Virtual Memory Management Policies
- performance of virtual memory systems is dependent on 
    - page table structure, page size
    - fetch policy, replacement policy and page cleaning policy
    - resident set size 

**Page Size**
- increasing page size 
- optimal size is a workflow independent trade off 

<table border="0">
 <tr>
    <td><b style="font-size:30px"> Pros </b></td>
    <td><b style="font-size:30px"> Cons </b></td>
 </tr>
 <tr>
    <td> decrease number of pages and hence size of the page tablr  </td>
    <td> increase internal fragmentation </td>
 </tr>
 <tr>
    <td> increase TLB coverage and reduce TLB misses since each TLB entry now map to a larger page </td>
    <td> increase page fault latency because more data will need to be read from disk </td>
 </tr>
 <tr>
    <td> increase swapping I/O throughput </td>
    <td> increase working set size so more memory will be needed in resident for a process to execute </td>
 </tr>    
</table>


**Fetch Policy**
- determines when a page in disk should be brought into memory
    - demand paging: loads pages only when page fults occur
        - results in many page faults when a process first starts
    - pre-paging: loads in more pages than needed when disk is idle
        - wastes I/O badnwidth if the pages aren't used
        - especially bad when resident pages are evicted in order to pre-fetch

**Replacement Policy**
- determine which page will be evicted when physical memory becomes full 
    - least likely to be used in the near future 
- kernel code, kernel data structures, I/O buffers and frames with a locked up should not be loaded to disk
1. Optimal replacement policy
    - evict the page that won't be used for the longest time
    - impossible because OS cannot predict whether a page will or will not be used 
2. FIFO policy
    - evict the oldest page
    - age of a page is not a reflection of its usage patterns 
3. LRU policy
    - evict the page that has not been referenced for the longest time (temporal locality)
    - implementation requires time stamp for each page updated on every reference &rarr; inefficient 
4. Clock Page Replacement policy
    - during execution, set the reference bit of a page if it's used 
    - when a page needs to be replaced, iterate through all pages and check if reference is set
        - yes &rarr; reset to 0 and continue
        - no &rarr: evict the page 

**Cleaning Policy**
- clean pages are much cheaper to replace than dirty pages because dirty pages requires writing back to disk the updates
1. demand cleaning
    - page is written to disk only when it is selected for replacement
    - high latency between deciding to replace and when the frame will become available again 
2. pre-cleaning
    - pagedaemon will write pages back to disk in the background
    - overlaps I/O with activity
    - increases the likelihood of replacing clean frames
    
**Resident Set Size**
1. fixed allocation
    - a process can use a fixed number of pages to execute
    - process memory is isolated and they are prevented from interfering each other
    - difficult to achieve high memory utilisation since some processes may have high fault rate while others don't use their allocation
2. variable allocation
    - number of pages allocated to a process varies across the lifetime of the process 
    - global scope
        - OS manages the global list of free frames
        - free frame is added to process's resident set when a page fault occurs
        - replace one from any process if no free frame 
        - ✅  automatic load balancing across the system based on usage 
        - ❌  does not protect important activities from memory hogs &rarr; thrashing 
    - local scope 
        - process is allocated a number of frames based on application type, program requests and other criteria
        - when a page fault occurs, a page from the resident set is chosen to be replaced so execution can continue
        - allocation is re-evaluated periodically based on page-fault rate
            - above upper bound &rarr; increase number of frames
            - below lower bound &rarr; decreate number of frames


# Multiprocessor Systems

**Motivation**
- a single CPU can only go so fast, so a work load that is not I/O or memory bound can be parallelised to take advantage of multiple CPUs
- running more than one processor on the same memory
- sharing hardware (disk etc) between CPU for economical reasons

**Amdahl's Law of Parallelisation**
- given a proportion `P` that can be parallised and the reset serial (`1 - p`), the speed up using `N` processors is given by `1 / ( (1 - p) + P / N )`
- max speed up when `N = ∞`, so max speed up = `1 / (1 - p)
- critical sections cannot be parallised and hence no speedup possible
- minimise serial regions to maximise speedup

**Bus-based Uniform Memory Access Multiprocessor**
- connecting multiple processors on a single bus (electrical wiring) to the memory
    - access to all memory occurs at same speed for all processors
- scalability is limited by bus bandwidth despite efforts of using caching to minimise access 
    - reduced by hardware design and ensuring cache locality and limited data sharing

**Multiprocessor Caching**
- each processor has a local cache in order to reduce access to main memory
    - ideally CPU only access their local cache but not possible since it's only a subset of main memory
    - bus bandwidth is still a bottleneck with more CPUs

**Cache Consistency**
- data stored in disk may be different to what is stored in a CPU's cache so multiple CPU accessing the same address will retrieve different content
- hardware handled issue where writes to one cache will propagate to all entries in appropriate caches (consumes bandwidth)

### Symmetric Multiprocessor (SMP)

**Private OS for each CPU**
- each CPU can only access physical memory that has been allocate &rarr; no concurrency issues
- each CPU handles the system calls from their own OS 

<table border="0">
 <tr>
    <td><b style="font-size:30px"> Pros </b></td>
    <td><b style="font-size:30px"> Cons </b></td>
 </tr>
 <tr>
    <td> easy to implement  </td>
    <td> each processor has its own scheduling queue &rarr; some processors may overload while others are idle  </td>
 </tr>
 <tr>
    <td> scalable because there are no shared serial sections </td>
    <td> each processor has a fixed memory partition &rarr; some processors may thrash while others are under utilised </td>
 </tr>
 <tr>
    <td> no CPU concurrency issues because there is no sharingt </td>
    <td> </td>
 </tr>    
</table>


**Symmetric Multiprocessors (SMP)**
- one OS kernel running on all processors so it can distribute resources and load evenly 
- real concurrency in the kernel where multiple processors may access the same kernel data structures &rarr; race conditions 
    1. use a single mutex to make the entire kernel a critical section
        - one CPU in the kernel at once (multiprocessing obsolete)
        - CPUs waiting to obtain the lock becomes a bottleneck if workload is mostly I/O bound
        - limited by the throughput of the critical section
    2. identify and make largely independent parts of the kernel a critical section
        - allows for more parallelism in the kernel
        - difficult to identify non-interfering parts 
        - kernel activities requiring more than one lock &rar; deadlock &rarr; enforce complex lock ordering scheme 


### Multiprocessor Synchronisation
- unlike uniprocessors, disabling interrupts does not prevent multiple CPUs from executing inside a critical section in parallel &rarr; hardware support

**test & set on SMP**
- multiple CPUs can observe a 0 from TAS and think they can obtain the lock
- TAS is an indivisiable CPU instruction but not indivisible memory instruction 
- solution: hardware blocks all other CPUs from accessing the bus for the duration of a CPU's usage of TAS
- issue: TAS is busy wait &rarr; if lock is held and CPU starts spinning &rarr; continuously block the bus with TAS &rarr; slow every other CPU down 
- caching does not reduce bus contention
    - TAS slows down cache refills
    - TAS requires the same entry in all CPU caches to be invalidated &rarr; single read write instruction will be passed to different CPU caches to allow them to do TAS however this consumes bus bandwidth &rarr; slow down CPU with the lock trying to access main memory
- solution:
    - if the lock is held by a CPU, all other CPUs will read 0 into the address of the lock in their cache 
    - each CPU will wait for it to change locally &rarr; no bus traffic 
    - when a CPU releases the lock, hardware will write 0 into the address of the lock for CPU caches and main memory
    - then a new CPU can acquire the lock (bus traffic)
    ```C
    start:
        while (lock == 1);
        r = TAS(lock);
        if (r == 1) 
            goto start;
    ```
   
**Spinning Lock VS Blocking Lock**
- uniprocessor: 
    - doesn't make sense to use spinlock 
    - time is simply wasted for a thread to acquire the lock instead of letting another thread to run
- multiprocessor:
    - spinlock: time penalty of waiting for a CPU to enter and exit its critical section
    - blocking lock: there is an extra time penalty for context switching back to the original thread of execution
- trade off:
    - more efficient to spin if lock is held for less time than the overhead of context switching between threads
        - spinlocks expects critical sections to be short and prevent other CPUs to spin for long (i.e. no waiting for I/O)

**Preemption & Spinlocks**
- if the spinlock holder is preempted when its timeslice is up, other CPUs will spin until the scheduler selects the holder again
- solution: spinlock is implemented with interrupts off 

# Scheduler

**What is the scheduler & its significance**
- scheduler determines which process will run next based on a variety of criterias such as efficiency, fairness and priority
- it is not important in situations where there is no choice or only thing one thing to run
    - i.e. simple embedded systems like washing machines or early systems that simply ran the next punch tape
- in modern systems, scheduling decisions can have a significant effect on the system's perceived performance and correctness 
    - i.e. email processing and user clicking on button. run application before processing the email &rarr; responsive
- scheduler will be required to make a scheduling decision when a process can no longer continue or when an activity results in more than one ready process
    - interrupts, new process (parent or child), blocking on lock, waiting for I/O, etc 

**I/O & CPU Bound Process Behaviours**

<img width="643" alt="image" src="https://user-images.githubusercontent.com/40874612/164591802-39a5a6a0-b2a2-4d83-aaa7-7688fb9127f8.png">

<table border="0">
 <tr>
    <td><b style="font-size:30px"> CPU Bound </b></td>
    <td><b style="font-size:30px"> I/O Bound </b></td>
 </tr>
 <tr>
    <td> spends most of its time doing computations </td>
    <td> spends most of its time waiting for I/O completion </td>
 </tr>
 <tr>
    <td> time to completion determined by CPU time received </td>
    <td> time to completion determined by I/O request time </td>
 </tr>
</table>

- mix of both CPU bound and I/O bound processes are required to keep both busy
- favour I/O bound processes ahead of CPU bound processes
    - I/O bound processes delay CPU bound processes by very little (time to receive the request) &rarr; more responsive
    - CPU bound processes may delay I/O processes for a very long period (time to complete computation) &rarr; no overlap I/O with computation

**Non-preemptive Scheduling**
- if a process is in the running state, it will continue execution until it complete or blocks on I/O or voluntarily `yield` control back to OS
- a single process can monopolised all the CPU time if yielding does not occur on a regular basis

**Preemptive Scheduling**
- a thread can be interrupted by the OS and moved to the ready state
- scheduler will automatically let another process execute after a timer interrupt (time slice up) or a higher priority process has become ready
- timer interrupts provide fairer usage of the CPU as threads cannot monopolise 

### Scheduling Algorithms

**Categories of Scheduling Algorithms**
- Batch systems
    - no users directly waiting &rarr; optimise for overall system performance
    - use preemptive scheduling since no timer interrupts are required and it's simply an overhead
- Interactive systems
    - users wait for their results &rarr; optimise for user perceived performance (responsiveness)
- Realtime systems
    - jobs must be scheduled such that they meet their deadlines 

**Scheduling Algorithm Requirements**
- all algorithms must ensure
    - all process are given a fair share of the CPU
    - scheduler must enforce the policy that was chosen
    - try to keep all parts of the system busy (overlap I/O with computation) 

<table border="0">
 <tr>
    <td><b style="font-size:30px"> Interactive Algorithms </b></td>
    <td><b style="font-size:30px"> Realtime Algorithms </b></td>
 </tr>
 <tr>
    <td> minimising response time is important to the user's perception of the system's performance </td>
    <td> jobs must meet their deadlines which otherwise result into poor consequences (data loss, etc) </td>
 </tr>
 <tr>
    <td> proportionality is an user's expectation that short / long jobs will have short / long response times &rarr; favour short jobs </td>
    <td> for some applications, an occassional miss is tolerable however, it must be predictable </td>
 </tr>
</table>

**Interactive Scheduling**
1. Round Robin Scheduling
- each process is given a timeslice and when it's up, next process preemptives the current one and runs for its timeslice
- preempted process will be placed at the back of the queue
- implemented with ready queue and regular timer interrupts
- suppose all timeslices are equal length, 
    - increasing length &rarr; less switches 
        - time wasted for context switches
    - decreasing length &rarr; appears more responsive 
        - degenerates into FCFS (scheduler not invoked) if burst legth < time slice length 
- fair & easy to implement but assumes all processes are equally important

2. Priority Based Scheduling
- each process has an associated priority which is used to influence scheduler decisions 
- priority can be determined internalls & externally
    - internal: I/O or CPU bound
    - external: importance to the user 
- implemented with queue within queues and round robin for every process in the inner queue 
- low priority processes may starve if there are also higher priority processes ready
    - priority must be adjusted periodically 

**Traditional Unix Scheduler**
- multi-level queue with round robin at each level 
- levels with lower values (-16 to 0) are higher priority and they are typically for kernel reserved disk and memory tasks (i.e. disk I/O)
- levels with higher values (0 to 16) are lower priority and they are typically for user level applications
- priorities are recalculated every second to
    - prevent low priority processes from starving
    - penalise CPU bound processes (bumped down) 
    - reward I/O bound processes (bumped up)
- `priority = cpu_usage + nice + base`
    - cpu_usage: number of clock ticks which decay overtime to avoid permanently penalising a process
    - nice: user supplied priority adjustment
    - base: hard-wired negative values used to boost I/O bound processes 


### Multiprocessor Scheduling
- process of allocating `X` processes to `Y` CPUs

**Single Shared Ready Queue**
- when a CPU becomes idle, it takes the process with highest priority from the ready queue

<table border="0">
 <tr>
    <td><b style="font-size:30px"> Pros </b></td>
    <td><b style="font-size:30px"> Cons </b></td>
 </tr>
 <tr>
    <td> easy to implement </td>
    <td> queue is a shared resource so a lock must be used. CPU's becoming blocked and queuing to obtain the lock is a bottleneck </td>
 </tr>
 <tr>
    <td> provides automatic load balancing for CPUs </td>
    <td> does not consider that the last CPU a process ran on is more likely to have more related entries in its buffer </td>
 </tr>
</table>

**Affinity & Multi Queue SMP Scheduling**
- favour running a process on the same CPU that it ran on before to take advantage of the cache 
- each CPU has a ready queue where processes are assigned based to their affinity and rough load balancing factor 
- when a CPU invokes the scheduler, a process from its own ready queue is chosen to ensure affinity and remove the need of having to obtain a lock for the common case
- if a CPU's ready queue is empty, it will run a process from a different queue instead going to idle &rarr; load balance to prevent idle CPU

