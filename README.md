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

**User-mode / Kernel-mode**

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
 
**User Level Threads**
**Kernel Level Threads**

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
  - Spin lock (above code): use only in multi core systems OR if the critical region is short 
  
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
        - resources must be acquired in order (x &rarr; y): if y is needed, x must be acquired first 
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

# MIPS R3000

**Branch Delay**: Instruction after a branch or jump is always executed prior to destination of jump 
```MIPS
    j       1f
    li      r2, 2           // branch delay slot
    li      r2, 3           // branch delay slot
1:  sw      r2, (r3) 
```


