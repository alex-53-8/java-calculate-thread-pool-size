# Calculate optimal thread pool size in Java
"How many thread to used in an executor pool, 22, 50, or 10" - similar questions were asked many times when a thread pool is created, and we need to choose a number of threads in a pool. If we choose too big number of threads then it could end up with exhausting resources, CPU, memory. If we choose too small number of threads then throughput of a system will be poor as CPU/memory is not used enough. 

To choose properly number of threads for a threads pool we need to understand what type of tasks a thread pool will work with. 
In general, all possible tasks can be divided into two groups: `compute-intensive tasks` or `CPU Bound tasks`, `I/O tasks` or `I/O Bound tasks`.

## Compute-intensive tasks (CPU bound)
Compute-intensive tasks in general are tasks when a CPU is constantly loaded with work and CPU idle time is much shorter comparing with usage time.

```text
CPU | C         | W | C         | W | C         | W | C... 
    |-----------|---|-----------|---|-----------|---|-------->
                                                             Time    
```
where `C` - compute time on CPU, `W` - idling time of CPU, waiting for a work.

In case of compute-intensive tasks a system usually achieves optimum utilization with a thread pool size calculated by a formula below.

```
Threads count = N + 1
```

where `N` - number of CPU cores in a machine, which can obtained in Java with `Runtime.getRuntime().availableProcessors()`

So, if a system has four CPU cores, then optimal maximum number of threads should be five.

## I/O tasks (I/O Bound)
I/O tasks are tasks which involve a lot of communications with different applications on another a machine via network call. For example, when a thread runs a Http request to a remote system or fetches data from a database. I/O tasks are characterised with short processing time and longer idling time - CPU is less loaded with work and a thread waits for data to be obtained from a different machine.

```text
CPU    | C | W     | C | W | C    | W | C... 
       |---|-------|---|---|------|---|-------->
                                               Time    
```
where `C` - compute time on CPU, `W` - idling time of CPU, waiting for a work.

In case of `I/O tasks` 

```
Threads count = N * U * (1 + W/C)
```

where
`N` - number of CPU cores in a machine, which can obtained in Java with `Runtime.getRuntime().availableProcessors()`
`U` - target CPU utilization and 0 <= `U` <= 1
`W/C` - ratio of wait time (`W`) to compute time (`C`)

Let's calculate following example:
1. a system has four CPU cores, so `N` = 4;
2. we target 100% CPU utilization, so `U` = 1;
3. we can estimate roughly for a request waiting time 100ms (`W`) and real calculating time is about 20ms (`C`). 

Then we should expect that threads count should `20` as `4 * 1 * 100/20 = 20`.

## Conclusion
This article describes in general an idea of how to calculate an optimal thread pool size. It would happen that thread pool size should be even greater than calculated one by a formula. Quite many aspects affect thread pool size, like if other applications are working on the same machine and how much CPU time that applications are consuming.

I would highlight that formulas give us an estimate size of a thread pool that should be verified in test and later in a production environment by observing CPU utilization.
