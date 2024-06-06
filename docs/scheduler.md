# Scheduler Overview


# sched_yield
```c
    int sched_yield(void);
```
Causes the calling thread to relinquish the CPU, the thread is moved to the end of the queue for its static priority and a new thread gets to run,
on linux, `sched_yield` always succeeds

# sched_getparam
```c
    int sched_getparam(pid_t pid, struct sched_param *param);
```
Fills `param` with scheduling parameters for the thread identified by `pid`, if `pid` is zero, then the parameters of the calling thread are retrieved

```c
    struct sched_param{
        ...
        int sched_priority;
        ...
    };
```


# sched_setparam
```c
    int sched_setparam(pid_t pid, const struct sched_param *param);
```
Sets the scheduling parameters associated with the thread ID in `pid`, if `pid` is 0 then the parameters of the calling thread are set. The interpretation of 
`param` depends on the scheduling policy of the thread identified by `pid`

# sched_getscheduler
```c
    int sched_getscheduler(pid_t pid);
```
Returns the current scheduling policy of the thread identified by `pid` or -1 on error


# sched_setscheduler
```c
    int sched_setscheduler(pid_t pid, int policy, const struct sched_param *param);
```
Sets both the scheduling `policy` and parameters for the thread whose ID is specified in `pid`, the policies supported are : 
- `SCHED_OTHER` : The standard round-robin time-sharing policy
- `SCHED_BATCH` : Batch style of execution of processes
- `SCHED_IDLE`  : For running very low priority background jobs
For each of the above policies, `param->sched_priority` must be 0
- `SCHED_FIFO`  : First-in, first-out policy
- `SCHED_RR`    : Round-robin policy
For each of the above, `param->sched_priority` specifies a scheduling priority, `SCHED_RESET_ON_FORK` can also be ORed in `policy` so that children created by `fork` do not inherit it
returns 0 on sucess or -1 on error


# sched_get_priority_max
```c
    int sched_get_priority_max(int policy);
```
Returns the maximum priority value that can be used with the scheduling algorithm defined by `policy` or -1 on error


# sched_get_priority_min
```c
    int sched_get_priority_min(int policy);
```
Returns the minimum priority value that can be used with the scheduling algorithm defined by `policy` or -1 on error

# sched_rr_get_interval
```c
    int sched_rr_get_interval(pid_t pid, struct timespec *tp);
```
Fills `tp` with the round-robin time quantum for the process identified by `pid`, the process should be running under `SCHED_RR` policy, if `pid` is 0 then it gets the time quantum
for the calling process, returns 0 on success and -1 on error

# sched_getaffinity
```c
    int sched_getaffinity(pid_t pid, size_t cpusetsize, cpu_set_t *mask);
```
A thread's CPU affinity mask determines the set of CPUs on which it is eligible to run, this function fills `mask` with `pid`'s thread affinity mask, 
`cpusetsize` should be set to `sizeof(cpu_set_t)`, returns the number of bytes copied into the `mask` buffer (the glibc wrapper returns 0)
or -1 on error

# sched_setaffinity
```c
    int sched_setaffinity(pid_t pid, size_t cpusetsize, cpu_set_t *mask);
```
Sets the thread's CPU affinity mask, `cpusetsize` should be `sizeof(cpu_set_t)`, there are a couple of macros used to set and operate with 
CPU affinity masks `CPU_...` , returns 0 on success and -1 on error

# sched_getattr
```c
    int sched_getattr(pid_t pid, struct sched_attr *attr, unsigned int size, unsigned int flags);
```
Fills `attr` with the scheduling policy and associated attributes for the thread in `pid`, if `pid` is 0 then it is treated as the caller thread's ID
```c
    struct sched_attr {
        u32 size;              /* Size of this structure */
        u32 sched_policy;      /* Policy (SCHED_*) */
        u64 sched_flags;       /* Flags */
        s32 sched_nice;        /* Nice value (SCHED_OTHER,
                                    SCHED_BATCH) */
        u32 sched_priority;    /* Static priority (SCHED_FIFO,
                                    SCHED_RR) */
        /* Remaining fields are for SCHED_DEADLINE */
        u64 sched_runtime;
        u64 sched_deadline;
        u64 sched_period;
    };
```
`size` should be set to `sizeof(sched_attr)`, `flags` is for future use and should be set to 0, returns 0 on success and -1 on error

# sched_setattr
```c
    int sched_setattr(pid_t pid, struct sched_attr *attr, unsigned int flags);
```
Sets the scheduling policy and associated attributes for the thread in `pid`, if `pid` is 0 then it is treated as the caller thread's ID, the fields in 
`attr` are : 
- `size`          : Should be set to `sizeof(struct sched_attr)`
- `sched_policy`  : Has the same `SCHED_` flags as described in `sched_setscheduler`, plus `SCHED_DEADLINE` which provides a deadline scheduling policy
- `sched_flags`   : Contains zero or more of the following flags that are ORed together to control scheduling behavior :
    - `SCHED_FLAG_RESET_ON_FORK` : Children created by `fork` do not inherit privileged scheduling
    - `SCHED_FLAG_RECLAIM`       : This flag allows a `SCHED_DEADLINE` thread to reclaim bandwidth unused by other real-time threads
    - `SCHED_FLAG_DL_OVERRUN`    : This flag allows an application to get informed about run-time overruns in `SCHED_DEADLINE` threads
- `sched_nice`    : Specifies the nice value to be set on `SCHED_OTHER` and `SCHED_BATCH`, this value is a number between -20 (high priority) and +19 (low priority)
- `sched_priority`: This field specifies the static priority to be set when `sched_policy` is `SCHED_FIFO` or `SCHED_RR`
- `sched_runtime` : Specifies the runtime parameter for deadline scheduling expressed in nanoseconds (used for `SCHED_DEADLINE`)
- `sched_deadline`: Specifies the deadline parameter for deadline scheduling expressed in nanoseconds (used for `SCHED_DEADLINE`)
- `sched_period`  : Specifies the period parameter for deadline scheduling expressed in nanoseconds (used for `SCHED_DEADLINE`)


`SCHED_DEADLINE`adds a way for scheduling real time tasks by specifying a period for the task, maximum deadline for it to be done and expected/max runtime time needed,
the kernel mandates that all of the deadline parameters should be at least 1024 (1 microsecond) and that `sched_runtime <= sched_deadline <= sched_period` 

`flags` is for future use and should be set to 0, returns 0 on success and -1 on error