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
