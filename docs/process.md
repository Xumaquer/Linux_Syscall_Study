# getrusage
```c
    int getrusage(int who, struct rusage *usage);
```
Returns resource usage measures for `who`, which can be one of the following : 
- `RUSAGE_SELF`    : Returns resource usage statistics for the calling process
- `RUSAGE_CHILDREN`: Return resource usage statistics for all children of the calling process that have terminated and been waited for
- `RUSAGE_THREAD`  : Return resource usage statistics for the calling thread (`_GNU_SOURCE` needs to be defined) 

```c
    struct rusage {
        struct timeval ru_utime; /* user CPU time used */
        struct timeval ru_stime; /* system CPU time used */
        long   ru_maxrss;        /* maximum resident set size */
        long   ru_ixrss;         /* integral shared memory size */
        long   ru_idrss;         /* integral unshared data size */
        long   ru_isrss;         /* integral unshared stack size */
        long   ru_minflt;        /* page reclaims (soft page faults) */
        long   ru_majflt;        /* page faults (hard page faults) */
        long   ru_nswap;         /* swaps */
        long   ru_inblock;       /* block input operations */
        long   ru_oublock;       /* block output operations */
        long   ru_msgsnd;        /* IPC messages sent */
        long   ru_msgrcv;        /* IPC messages received */
        long   ru_nsignals;      /* signals received */
        long   ru_nvcsw;         /* voluntary context switches */
        long   ru_nivcsw;        /* involuntary context switches */
    };
```
Returns 0 on success and -1 on error


# pause
```c
    int pause(void);
```
Causes the calling process (or thread) to sleep until a signal is delivered that either terminates the process or causes the invocation of a signal-catching function

# wait4
```c
    pid_t wait4(pid_t pid, int *_Nullable wstatus, int options,
                struct rusage *_Nullable rusage);
```
Waits for state changes in a child of the calling process and obtains information about the child whose state has changed, a state change occurs when the child terminates,
is stopped or resumed by a signal, if a child is terminated, calling `wait` allows the system to release the resources associated with the child, if wait is not done,
then the child remains in a "zombie" state.

A child is specified by `pid`, which can be : 
- `< -1` : Waits for any child process whose process group ID is equal to the absolute value of `pid`
- `-1`   : Waits for any child process
- `0`    : Waits for any child process whose process group ID is equal to that of the calling process
- `> 0`  : Waits for any child process whose process ID is equal to the value of `pid`

`options` is a bit wise OR of zero or more of the following flags:
- `WNOHANG`     : Returns immediately if no child has exited 
- `WUNTRACED`   : Returns if a child has stopped (but not traced via `ptrace`)
- `WCONTINUED`  : Returns if a stopped child has been resumed by a delivery of `SIGCONT`

If `wstatus` is not null, it is filled with status information to the `int` it points to, this integer can be inspected with the following macros :
- `WIFEXITED`   : Returns true if the child terminated normally, that is, by calling `_exit` or returning from `main()`
- `WEXITSTATUS` : Returns the exit status of the child, this consists on the least significant 8 bits of the `status` argument specified on `_exit` or the return of `main()`
- `WIFSIGNALED` : Returns true if the child process was terminated by a signal
- `WTERMSIG`    : Returns the number of the signal that caused the child process to terminate, should be employed if `WIFSIGNALED` returned true
- `WCOREDUMP`   : Returns true if the child produced a core dump, this should only be employed if `WIFSIGNALED` returned true
- `WIFSTOPPED`  : Returns true if the child processs was stopped by delivery of a signal
- `WSTOPSIG`    : Returns the number of the signal which caused the child to stop, should only be employed if `WIFSTOPPED` returned true
- `WIFCONTINUED`: Returns true if the child process was resumed by delivery of `SIGCONT`

If `rusage` is not null, then it is filled as if `getrusage` was called, returns 0 on success and -1 on error

# waitid
```c
    int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options, struct rusage *_Nullable rusage);
```
Similar to `wait4` 
`idtype` can be one of the following flags:
- `P_PID`   : Wait for the child whose process ID matches `id`
- `P_PIDFD` : Wait for the child reffered to by the PID file descriptor in `fd` 
- `P_PGID`  : Wait for any child whose process group ID matches `id`, if `id` is zero, then wait for any child that's in the same process group
- `P_ALL`   : Waits for any child, `id` is ignored
The child state changes to wait are specified by ORing one or more of the following flags in `options` (flags starting with "__" apply only for children created by `clone`): 
- `WEXITED`    : Wait for children that have terminated
- `WSTOPPED`   : Wait for children that have been stopped by delivery of a signal
- `WCONTINUED` : Wait for children that have been resumed by delivery of `SIGCONT`
- `WNOHANG`    : Same as in `wait4`
- `WNOWAIT`    : Leave the child in a waitable state, a later wait call can be used to again retrieve the child status information
- `__WCLONE`   : Wait for "clone" children only
- `__WALL`     : Waits for all children, regardless of type ("clone" or "non-clone")
- `__WNOTHREAD`: Do not wait for children of other threads in the same thread group

Upon successful return, fills the following fields of `infop` 
- `si_pid`   : Process ID of the child
- `si_uid`   : The real user ID of the child
- `si_signo` : Always set to `SIGCHLD`
- `si_status`: Either the exit status of the child or the signal that caused the child to terminate, stop or continue
- `si_code`  : Set to one of
    - `CLD_EXITED`   : Child called `_exit`
    - `CLD_KILLED`   : Child was killed by signal
    - `CLD_DUMPED`   : Child was killed by signal and dumped core
    - `CLD_STOPPED`  : Child was stopped by signal
    - `CLD_TRAPPED`  : Child has been trapped
    - `CLD_CONTINUED`: Child continued by `SIGCONT`

# _exit
```c
    [[noreturn]] void _exit(int status);
```
Terminates the calling thread immediately, `status & 0xFF` is returned to the parent as the thread's exit status

# exit_group
```c
    [[noreturn]] void exit_group(int status);
```
Terminates all threads in the calling process's thread group

# nanosleep
```c
    int nanosleep(const struct timespec *req,
                  struct timespec *_Nullable rem);
```
Suspends the execution of the calling thread until either `*req` time has elapsed or a signal that triggers the invocation of a handler in the calling thread or that terminates the process, if it's interrupted by a signal, it returns -1 and `rem` is filled with the remaining time otherwise it returns 0

# getpid
```c
    pid_t getpid(void);
```
Returns the process ID of the calling process

# getppid
```c
    pid_t getppid(void);
```
Returns the process ID of the parent of the calling process.

# getuid
```c
    uid_t getuid(void);
```
Returns the real user ID of the calling process

# geteuid
```c
    uid_t geteuid(void);
```
Returns the effective user ID of the calling process


# pidfd_open
```c
    int pidfd_open(pid_t pid, unsigned int flags);
```
Creates a file descriptor that refers to the process whose PID is specified in `pid`, `flags` argument either has 0 or the following flag : 
- ` PIDFD_NONBLOCK` : Returns a nonblocking file descriptor, if the process has not yet terminated, then waiting on it will immediately return `EAGAIN` instead of blocking 

# pidfd_getfd
```c
    int pidfd_getfd(int pidfd, int targetfd, unsigned int flags);
```
Returns a duplicate of `targetfd` file descriptor of the process referred by `pidfd`, `flags` is reserved for future use and should be 0,
if the call fails, returns -1 instead


# pidfd_send_signal
```c
    int pidfd_send_signal(int pidfd, int sig, siginfo_t *_Nullable info, unsigned int flags);
```
Sends `sig` signal to the process referred by `pidfd`, `info` should be populated as if using `rt_sigqueueinfo`, `flags` is reserved for future use and should be 0,
returns 0 on sucess and -1 on error

# fork
```c
    pid_t fork(void);
```
Creates a new process by duplicating the calling process, the PID of the child process is returned to the parent and 0 is returned to the child, on failure -1 is returned to the parent and no child is created


# vfork
```c
    pid_t vfork(void);
```
Has the same effect as `fork`, except that the behavior is undefined if the process created by `vfork` modifies any data other than a variable of type `pid_t` used to store the return value from `vfork`,
or returns from the function in which `vfork` was called, or calls any other function before sucessfully calling `_exit` or calls `execve`.
Calling this suspends the calling thread until the child terminates or calls `execve`, it uses the same address space as the calling thread until a call to `execve`


# clone
```c
    // on x86-64 architecture
    long clone(unsigned long flags, void *stack,
               int *parent_tid, int *child_tid,
               unsigned long tls);
    // on x86-32, ARM, ARM64, PA-RISC, arc, Power PC, xtensa and MIPS architecture
    long clone(unsigned long flags, void *stack,
               int *parent_tid, unsigned long tls,
               int *child_tid);
    // on cris and s390 architecture
    long clone(void *stack, unsigned long flags,
               int *parent_tid, int *child_tid,
               unsigned long tls);

    //glibc wrapper
    int clone(int (*fn)(void *_Nullable), void *stack, int flags,
              void *_Nullable arg, ... /* pid_t *_Nullable parent_tid, 
                                          void *_Nullable tls,
                                          pid_t *_Nullable child_tid */);
```
Creates a new child process, in a manner similar to `fork` but provides precise control using `flags` which is a bit mask of zero or more of the constants below :
- `CLONE_CHILD_CLEARTID` : Clear the child thread ID at the location pointed by `child_tid` when the child exits, do a futex wakeup at that address
- `CLONE_CHILD_SETTID`   : Sets the child thread ID at the location pointed by `child_tid` when the child exits, do a futex wakeup at that address
- `CLONE_CLEAR_SIGHAND`  : Signal disposition in the child thread are reset to their default disposition
- `CLONE_DETACHED`       : Causes the parent to receive a signal when the child terminates, now it's ignored because `CLONE_THREAD` exists
- `CLONE_FILES`          : Calling process and the child process share the same file descriptor table
- `CLONE_FS`             : Shares the same filesystem information, this includes the root of the filesystem, current working directory and the umask
- `CLONE_INTO_CGROUP`    : Allows the child process to be created iin a different version 2 cgroup, the caller needs to pass a file descriptor in `cl_args.cgroup`
- `CLONE_IO`             : The new process shares an I/O context with the calling process
- `CLONE_NEWCGROUP`      : Create the process in a new cgroup namespace
- `CLONE_NEWIPC`         : Create the process in a new IPC namespace
- `CLONE_NEWNET`         : Create the process in a new network namespace
- `CLONE_NEWNS`          : Create the process in a new mount namespace
- `CLONE_NEWPID`         : Create the process in a new PID namespace
- `CLONE_NEWUSER`        : Create the process in a new user namespace
- `CLONE_NEWUTS`         : Create the process in a new UTS namespace
- `CLONE_PARENT`         : The parent of the new child will be the same as the calling process
- `CLONE_PARENT_SETID`   : Store the child thread ID at the location pointed by `parent_tid`
- `CLONE_PID`            : The child process is created with the same process ID as the calling process, ignored nowadays and the same bit was recycled for `CLONE_PIDFD`
- `CLONE_PIDFD`          : The PID file descriptor reffering to the child process is allocated and placed at `pidfd` (for `clone3`) or `parent_tid` (for `clone`)
- `CLONE_PTRACE`         : If the calling process is being traced, then also trace the child
- `CLONE_SETTLS`         : The TLS (Thread Local Storage) descriptor is set to `tls`, `tls` is `struct user_desc*` on x86, on x86-64 it is the new value for %fs register and on other architectures
- `CLONE_SIGHAND`        : The child process shares the same table of signal handlers, but have different signal masks and sets of pending signals.
- `CLONE_STOPPED`        : This flag has been removed, but it used to start the child process initially stopped which had to be resumed with a `SIGCONT` signal
- `CLONE_SYSVEM`         : The child process shares a single list of System V semaphore adjustment values
- `CLONE_THREAD`         : The child is placed in the same thread group as the calling process, must include `CLONE_SIGHAND` and `CLONE_VM` to be included
- `CLONE_UNTRACED`       : The tracing process cannot force `CLONE_PTRACE` on this child process
- `CLONE_VFORK`          : The execution of the calling process is suspended until the child releases its virtual memory resources with `execve` or `_exit`
- `CLONE_VM`             : The calling process and the child process share the same memory space, and `mmap` or `munmap` affect other processes
On sucess, returns the thread ID of the child process in the caller thread of execution


# clone3
```c
    long clone3(struct clone_args *cl_args, size_t size);
```
Similar to `clone`, but uses `cl_args` for the arguments, and `size` should be set to `sizeof(struct clone_args)`, has extra options that were already described
in `clone`

```c
    struct clone_args {
        u64 flags;        /* Flags bit mask */
        u64 pidfd;        /* Where to store PID file descriptor
                            (int *) */
        u64 child_tid;    /* Where to store child TID,
                            in child's memory (pid_t *) */
        u64 parent_tid;   /* Where to store child TID,
                            in parent's memory (pid_t *) */
        u64 exit_signal;  /* Signal to deliver to parent on
                            child termination */
        u64 stack;        /* Pointer to lowest byte of stack */
        u64 stack_size;   /* Size of stack */
        u64 tls;          /* Location of new TLS */
        u64 set_tid;      /* Pointer to a pid_t array
                            (since Linux 5.5) */
        u64 set_tid_size; /* Number of elements in set_tid
                            (since Linux 5.5) */
        u64 cgroup;       /* File descriptor for target cgroup
                            of child (since Linux 5.7) */
    };
```

# execve
```c
    int execve(const char *pathname, char *const _Nullable argv[], char *const _Nullable envp[]);
```
Executes the program referred by `pathname`, this causes the program that is currently being run by the calling process to be replaced with a new program, with newly initialized stack, heap and data
segments. `pathname` must be a binary executable or a interpreter script.
`argv` is a array of pointers to string passed to the new program's command line, by convention `argv[0]` should contain the filename associated with the file being executed. The `argv` array
must be terminated by a NULL pointer
`envp` is a array of pointers to strings, conventinally in the form of `key=value`, which are passed as the environment of the new program, `envp` must be terminated by a NULL pointer

A interpreter script is a text file with execution permission with `!# interpreter_path` as the first line, the interpreter will be invoked against the text file itself, 
for example a python script could be run with `execve` by setting execution permission on the file and having the following contents
```py
    #! /usr/bin/python3
    print("Execve says hello")
```
On success, `execve` does not return, on error -1 is returned

# execveat
```c
    int execveat(int dirfd, const char *pathname,
                char *const _Nullable argv[],
                char *const _Nullable envp[],
                int flags);
```
Similar to `execve` but has a `dirfd` for relative paths and a extra `flags` parameter which is a bit mask that can include zero or more of the following flags : 
- `AT_EMPTY_PATH`       : If `pathname` is an empty string, operate on the file referred to by `dirfd`
- `AT_SYMLINK_NOFOLLOW` : If the file identified by `dirfd` and a non-NULL `pathname` is a symbolic link, the call fails


# personality
```c
    int personality(unsigned long persona);
```
Sets the execution domain for a process, which tells how to map signal numbers into signal actions alongside other things, allowing Linux
to provide limited support for binaries compiled under other UNIX-like OS, if `persona` is set to 0xFFFFFFFF then it returns the current execution domain, 
A list of the available execution domains can be found on `<sys/personality.h>`, the least significant byte defines the personality the kernel should assume, flag
values are as follows : 
- `ADDR_COMPAT_LAYOUT` : Provide legacy virtual address space layout
- `ADDR_NO_RANDOMIZE`  : Disable adress space layout randomization
- `ADDR_LIMIT_32BIT`   : Limit the address space to 32bits
- `ADDR_LIMIT_3GB`     : With this flag set, use 0xC0000000 as the offset at which to search a virtual memory chunk on `mmap`
- `FDPIC_FUNCPTRS`     : User-space function pointers to signal handlers point to descriptors
- `MMAP_PAGE_ZERO`     : Map page 0 as read only
- `READ_IMPLIES_EXEC`  : With this flag set, `PROT_READ` implies `PROT_EXEC` for `mmap`
- `SHORT_INODE`        : No effects (?)
- `STICKY_TIMEOUTS`    : With this flag set, `select`, `pselect` and `ppoll` do not modify the returned timeout argument when interrupted by a signal
- `UNAME26`            : Have `uname` report a 2.6.40+ version number rather than 3.x version number
- `WHOLE_SECONDS`      : No effect
Available execution domains are : 
- `PER_BSD`     : BSD
- `PER_IRIX64`  : IRIX 6 64-bit, implies `STICKY_TIMEOUTS`
- `PER_IRIXN32` : IRIX 6 new 32-bit
- `PER_ISCR4`   : Implies `STICKY_TIMEOUTS`
- `PER_LINUX`   : Linux
- `PER_OSF4`    : OSF/1 v4
- `PER_OSR5`    : Implies `STICKY_TIMEOUTS` and `WHOLE_SECONDS`
- `PER_SCOSVR3` : Implies `STICKY_TIMEOUTS` and `WHOLE_SECONDS`
- `PER_SOLARIS` : Implies `STICKY_TIMEOUTS`
- `PER_SVR3`    : Implies `STICKY_TIMEOUTS` and `SHORT_INODE`
- `PER_SVR4`    : Implies `STICKY_TIMEOUTS` and `MMAP_PAGE_ZERO`
- `PER_UW7`     : Implies `STICKY_TIMEOUTS` and `MMAP_PAGE_ZERO`
- `PER_WYSEV386`: Implies `STICKY_TIMEOUTS` and `SHORT_INODE`
- `PER_XENIX`   : Implies `STICKY_TIMEOUTS` and `SHORT_INODE`
On success `persona` is returned, on error -1 is returned
