# Process Scheduling

## Intro
* OS needs to switch from one process to another in very short time frame

## Scheduling Policy

### Traditional Unix objectives
* Fast proccess response time
* Good throughput for background jobs
* Avoid process starvation
* Balance low vs high priority processes

### Linux scheduling info
* Each procesws runs in an arbitrary time slice
* CPU time is divided into slices
    * q: what does this mean?
* One process per time slice
* Scenario: a process may not terminate when its time slice expires
    * A process switch may take place
* Time sharing relies timer interrupts, transparent to processes
    * q: what does transparent mean in practice?
### Priority rankings
* May need complicated algorithms to derive the current priority of a process
    * q: what type of factors feed the algo?
* End result is process has a value to tell scheduler how appropriate it is to run it.
* Process generation is dynamic
* Scheduler keeps track of what processes are doing, adjusts priorities periodically
    * Dynamically increase priority of processes that haven't been run for "a while"
    * Dynamically decrease priority of long running processes
    * q: what type of process portfolio is bad with this context?
### Process Resource Usage
* Processes are normally either I/O or CPU bound
* I/O - process waits for I/O operation to complete
* CPU - calculations that take lots of CPU time

### Types of Processes
* Interactive processes
    * Interact constantly with users
    * Wait for keypresses + mouse operations
    * Seems like lots of I/O operations
    * Process needs to be woken up quickly to respond
        * Consequence is that user sees a delay in response on GUI
        * i.e typing a letter has a 2 second delay to render
    * Average delay must be 50 - 150 ms
        * Need to also keep delay bounded
        * Consequence is that user sees erratic delays in response on GUI
    * Typical applications
        * Command shells
        * Text editors
        * Browsers
* Batch processes
    * No user interaction, often run in background
    * Don't need to be responsive
        * Penalized by scheduler as a result
    * Typical applications
        * language compilers
        * remote web servers
        * database search engines
        * computation focused app
* Real-time processes
    * Have very stringent scheduling requirements
    * Can never be blocked by lower-priority processes
    * Short guaranteed response time with minimum variance
    * Typical applications
        * Video + sound apps
        * Robot controllers
        * Data collection tools from physical sensors
* Scheduler behavior
    * Real time applications are specially recognized by scheduler
    * Hard to tell difference between batch vs interactive
    * Have hueristic to guess which one
        * Uses past behavior of the process
        * Probably includes I/O operations
    * Programmers can change scheduling priorities via system calls

### Process Preemption
* Linux processes are preemptible
* Flow
    * Process enters TASK_RUNNING state
    * Kernel checks current priority of Process A is greater than priority of current running Process B
    * If it is, current Process B is interrupted
    * Scheduler is invoked to select another process to run
        * Usually is the process that just entered the TASK_RUNNABLE state
    * Process B may be preempted if its time slice expires
    * Process B gets the TIF_NEED_RESCHED flag set
    * Scheduler invoked when timer interrupt handler terminates
* Prempted process
    * They aren't suspened
    * Still in TASK_RUNNING state
    * Not using CPU though

### Appropriate Time Frame for Quantam/time slice
* Cannot be too short or too long
    * Too short = too many process switches, too much CPU effort there
    * Too long = processes don't look to be executing concurrently
        * Each runnable process stops for too long time
        * May degrade responsiveness of a system
* General rule of thumb
    * Make it as long as possible before it impacts system response time

## Scheduling Algorithm

### Intro
* Scheduling algorithm ued to be simple
    * At every process switch, the kernel:
        * Scans list of runnable processes
        * Compute priorities
        * Select best process to run
    * Time spent selecting best process is directly related to number of runnable processes that exist
        * Too costly if too many processes being run at once
        * Too much time spent on process analysis
* New scheduling algorithm scales much better
    * Selects process in constant time
    * Each cpu has its own queue of runnable processes
    * Better distinguishes interactive vs batch processes
    * Users of interactive programs feel new linux is more responsive than previous versions
* Scheduler always succeeds in finding a process to run
    * Always at least one runnable process: the Swapper process
        * Swapper only executes when CPU cannot execute other processes
        * q: does this have any functional value, or is it just a filler?
* Scheduling classes
    * SCHED_FIFO
        * First in first out 
        * For real time processes
        * If there are no other higher priority real time processes, just keep running current process
    * SCHED_RR
        * Round robin scheduler
        * For real time processes
        * Fair assignment of CPU time to all real time processes with the same priority
    * SCHED_NORMAL
        * Normal scheduler
    * Very different behaviors depending on tif its a real time or conventional process

### Scheduling of Conventional Processes
* Conventional process has a static priority
    * Used by scheduler to rate the process with respect to other conventional processes
    * Values range from 100 (highest priority) to 139 (lowest priority)
* New process inherits static priority of parent
* User can change the priority via apis

### Base Time Quantum
* Static priority basically determines the base time quantum of a process
* Basic formula
    * Priority 120 is arbitrary cutoff for formula
    * < 120 has a 4x multiplier
    * (140 - x) * 5  if x >= 120
    * (140 - x) * 20  if x < 120
* Higher priority get larger slices compared to lower priority
* There are 5 preset values

### Dynamic Priority and Average Sleep Time
* Processes have both a dynamic and static priority
* Dynamic priority is the number looked by scheduler to run
* Dynamic priority: max (100, min (  static priority - bonus + 5, 139))
    * bonus depends on average sleep time of process
* Average sleep time is average number of nanoseconds the process spent sleeping
    * It does not just measure elapsed time
    * TASK_INTERRUPTIBLE vs TASK_UNINTERRUPTIBLE
    * These contribute to average time differently
    * Average sleep time can never be larger than 1 second
    * Also used to determine if process is batch or interactive
        * dynamic priority <= 3 x static priority / 4 + 28 

### Active and Expired Processes
* Higher priority processes shouldn't simply lock out lower priority processes
* Two disjoint process lists
    * Active processes
        * Runnable processes that haven't exhausted the time quantum
        * Allowed to run
    * Expired processes
        * Runnable processes exhausted time quantum
        * Forbidden to run until all active processes expire
* A little more complex b/c scheduler tries to boost performance of interactive processes

