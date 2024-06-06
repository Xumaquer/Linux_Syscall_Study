# Signal List
Signals are 


# signal
```c
    typedef void (*sighandler_t)(int);

    sighandler_t signal(int signum, sighandler_t handler);
```
Sets a action to be done when `signum` signal is delivered, `handler` sets the action and can be 
- `SIG_DFL` for the default action
- `SIG_IGN`to ignore this signal
- A pointer to a signal handling function that receives the `signum` and will be called when the signal is delivered

This function returns the previous signal handler or `SIG_ERR`on error (which is implementation defined), but it's only portable if you
use `SIG_DFL` or `SIG_IGN` as the definition of `sighandler_t` varies across systems, so `sigaction` is preferred


# rt_sigaction
```c
    int rt_sigaction(int signum,
                     const struct sigaction *_Nullable restrict act,
                     struct sigaction *_Nullable restrict oldact);
```
Changes the action taken by a process on receipt of a specific signal, `signum` is the signal and can be any valid signal execpt `SIGKILL` and `SIGSTOP`,
`act` is the new action for the signal and `oldact` is filled with the previous action

```c
    struct sigaction {
        void     (*sa_handler)(int);
        void     (*sa_sigaction)(int, siginfo_t *, void *);
        sigset_t   sa_mask;
        int        sa_flags;
        void     (*sa_restorer)(void);
    };
```
`sa_handler` works in the same way as `signal`'s `handler` parameter

`sa_mask` is a mask of signals which should be blocked during execution of the signal handler

`sa_flags` specifies a set of flags which modify the behavior of the signal, zero or more of the following flags bitwise ORed :
- `SA_NOCLDSTOP`     : If `signum` is `SIGCHILD`do not receive notifications when child processes stop
- `SA_NOCLDWAIT`     : If `signum` is `SIGCHILD`do not transform children into zombies when they terminate
- `SA_NODEFER`       : Do not add the signal to the thread's signal mask while the handler is executing, unless the signal is in `act.sa_mask`
- `SA_ONSTACK`       : Call the signal handler on a alternate signal stack provided by `sigaltstack`
- `SA_RESETHAND`     : Restore the signal action to the default upon entry to the signal handler
- `SA_RESTART`       : Provide behavior compatible with BSD signal semantics by making certain syscalls restartable across signals
- `SA_RESTORER`      : Not intended for application use, used by C libraries to indicate that `sa_restorer` contains a `signal trampoline`
- `SA_SIGINFO`       : The signal handler takes three arguments, not one, in this case `sa_sigaction` should be set instead of `sa_handler`
- `SA_UNSUPPORTED`   : Used to dynamically probe for flag bit support
- `SA_EXPOSE_TAGBITS`: Normally, when delivering a signal, an architeture-specific set of bits are cleared from the `si_addr` of `siginfo_t`, if this is set, they will be preserved


```c
void handler(int sig, siginfo_t *info,void *ucontext);
```
`sig` is the signal number, `info` is a pointer to a `siginfo_t` and `ucontext` is a pointer to a `ucontext_t` structure cast to `void*`
```c
    siginfo_t {
        int      si_signo;     /* Signal number */
        int      si_errno;     /* An errno value */
        int      si_code;      /* Signal code */
        int      si_trapno;    /* Trap number that caused
                                    hardware-generated signal
                                    (unused on most architectures) */
        pid_t    si_pid;       /* Sending process ID */
        uid_t    si_uid;       /* Real user ID of sending process */
        int      si_status;    /* Exit value or signal */
        clock_t  si_utime;     /* User time consumed */
        clock_t  si_stime;     /* System time consumed */
        union sigval si_value; /* Signal value */
        int      si_int;       /* POSIX.1b signal */
        void    *si_ptr;       /* POSIX.1b signal */
        int      si_overrun;   /* Timer overrun count;
                                    POSIX.1b timers */
        int      si_timerid;   /* Timer ID; POSIX.1b timers */
        void    *si_addr;      /* Memory location which caused fault */
        long     si_band;      /* Band event (was int in
                                    glibc 2.3.2 and earlier) */
        int      si_fd;        /* File descriptor */
        short    si_addr_lsb;  /* Least significant bit of address
                                    (since Linux 2.6.32) */
        void    *si_lower;     /* Lower bound when address violation
                                    occurred (since Linux 3.19) */
        void    *si_upper;     /* Upper bound when address violation
                                    occurred (since Linux 3.19) */
        int      si_pkey;      /* Protection key on PTE that caused
                                    fault (since Linux 4.6) */
        void    *si_call_addr; /* Address of system call instruction
                                    (since Linux 3.5) */
        int      si_syscall;   /* Number of attempted system call
                                    (since Linux 3.5) */
        unsigned int si_arch;  /* Architecture of attempted system call
                                    (since Linux 3.5) */
    }
```

The `si_code` field is used to describe the reason the signal was sent 
- `SI_USER`   : Sent with `kill`
- `SI_KERNEL` : Sent by the kernel
- `SI_QUEUE`  : `sigqueue`
- `SI_TIMER`  : POSIX timer expired
- `SI_MESGQ`  : POSIX message queue state changed
- `SI_ASYNCIO`: AIO completed
- `SI_SIGIO`  : Queued `SIGIO`
- `SI_TKILL`  : `tkill` or `tgkill` 
The following values can be placed in `si_code` for a `SIGILL` signal :
- `ILL_ILLOPC` : Illegal opcode
- `ILL_ILLOPN` : Illegal operand
- `ILL_ILLADR` : Illegal address mode
- `ILL_ILLTRAP`: Illegal trap
- `ILL_PRVOPC` : Privileged opcode
- `ILL_PRVREG` : Privileged register
- `ILL_COPROC` : Coprocessor error
- `ILL_BADSTK` : Internal stack error
The following values can be placed in `si_code` for a `SIGFPE` signal:
- `FPE_INTDIV` : Integer divide by zero
- `FPE_INTOVF` : Integer overflow
- `FPE_FLTDIV` : Floating-point divide by zero
- `FPE_FLTOVF` : Floating-point overflow
- `FPE_FLTUND` : Floating-point underflow
- `FPE_FLTRES` : Floating-point inexact result
- `FPE_FLTINV` : Floating-point invalid operation
- `FPE_FLTSUB` : Subscript out of range
The following values can be placed in `si_code` for a `SIGSEGV` signal:
- `SEGV_MAPERR` : Address not mapped to object
- `SEGV_ACCERR` : Invalid permissions for mapped object
- `SEGV_BNDERR` : Failed bound checks
- `SEGV_PKUERR` : Access was denied by memory protection keys
The following values can be placed in `si_code` for a `SIGBUS` signal:
- `BUS_ADRALN`   : Invalid address alignment
- `BUS_ADRERR`   : Nonexistent physical memory
- `BUS_OBJERR`   : Object-specific hardware error
- `BUS_MCERR_AR` : Hardware memory error consumed on a machine check, action required
- `BUS_MCERR_AO` : Hardware memory error detected in process but not consumed, action optional
The following values can be placed in `si_code` for a `SIGTRAP` signal:
- `TRAP_BKPT`   : Process breakpoint
- `TRAP_TRACE`  : Process trace trap
- `TRAP_BRANCH` : Process taken branch trap (IA64 only)
- `TRAP_HWBKPT` : Hardware breakpoint/watchpoint
The following values can be placed in `si_code` for a `SIGCHLD` signal:
- `CLD_EXITED`   : Child has exited
- `CLD_KILLED`   : Child was killed
- `CLD_DUMPED`   : Child terminated abnomally
- `CLD_TRAPPED`  : Traced child has trapped
- `CLD_STOPPED`  : Child has stopped
- `CLD_CONTINUED`: Stopped child has continued
The following values can be placed in `si_code` for a `SIGIO`/`SIGPOLL` signal:
- `POLL_IN` : Data input available
- `POLL_OUT`: Output buffers available
- `POLL_MSG`: Input message available
- `POLL_ERR`: I/O error
- `POLL_PRI`: High priority input available
- `POLL_HUP`: Device disconnected
The following values can be placed in `si_code` for a `SIGSYS` signal: 
- `SYS_SECCOMP` : Triggered by a `seccomp` filter rule

returns 0 on sucess and -1 on error


# rt_sigprocmask
```c
    //glibc wrapper
    int sigprocmask(int how, const sigset_t *_Nullable restrict set,
                       sigset_t *_Nullable restrict oldset);

    //syscall
    int rt_sigprocmask(int how, const kernel_sigset_t *_Nullable set,
                       kernel_sigset_t *_Nullable oldset, size_t sigsetsize);
```
Used to fetch and/or change the signal maks of the calling thread, the signal mask is a set of signal whose delivery is blocked for the caller,
The behavior of the call is dependent on the value of `how` :
- `SIG_BLOCK`   : The set of blocked signals is the union of the current set and the `set` argument
- `SIG_UNBLOCK` : The signals in `set` are removed from the current set of blocked signals. It is permissible to attempt to unblock a signal which is not blocked
- `SIG_SETMASK` : The set of blocked signals is set to `set`
If `oldset`is non-NULL , it is filled with the previous value of the signal mask
If `set` is NULL then the signal mask is unchanged (`how` is ignored)
It's not possible to block `SIGKILL` or `SIGSTOP` attemts to do so are ignored, returns 0 on sucess and -1 on failure

# rt_sigreturn
```c
    int rt_sigreturn(...);
```
Exists as a way to implement signal handlers, during transition to a user space process with a pending signal, the Linux kernel creates a new frame on the
user-space stack saving various pieces of process context (processor status word, registers, signal mask, and signal stack settings)
During the transition back to user mode the signal handler is called, and upon return from the handler, the control passes to a piace of user-code called the `signal trampoline`,
which calls `rt_sigreturn` to undo that, on modern systems it resides either in `vdso` or in the C library, which informs the kernel of the location of the trampoline by placing it's address
in the `sa_restorer` of `sigaction` structure while also setting the `SA_RESTORER` flag in `sa_flags`
The saved process context information is placed in a `ucontext_t` structure which is visible within the signal handler
`rt_sigreturn` never returns as it restores the previous user context 

# rt_sigpending
```c
    int sigpending(sigset_t *set);
```
Fills in `set` the set of signals that are pending for delivery to the calling thread (signals which have been raised while blocked),
returns 0 on sucess and -1 on failure

# rt_sigtimedwait
```c
    int sigtimedwait(const sigset_t *restrict set,
                     siginfo_t *_Nullable restrict info,
                     const struct timespec *restrict timeout);
```
Suspends execution of the calling thread until one of the signals in `set` is pending or the period specified by `timeout` elapses,
if `timeout` is NULL then it has no timeout, on sucess returns a signal number and on failure returns -1 

# rt_sigqueueinfo
```c
    int rt_sigqueueinfo(pid_t tgid, int sig, siginfo_t *info);
```
Sends the signal `sig` to the thread group with the ID `tgid`, the `info` argument specifies data to accompany the signal,
which can be used to pass user data in `info.si_value`, while `info.si_code` should be one of the `SI_*` codes except `SI_USER`, `SI_KERNEL` and `SI_TKILL`
Returns 0 on success and -1 on error


# rt_tgsigqueueinfo
```c
    int rt_tgsigqueueinfo(pid_t tgid, pid_t tid, int sig, siginfo_t *info);
```
Similar to `rt_sigqueueinfo` but sends the signal and data to a single thread specified by the combination of `tgid` (thread group id) and a `tid`, a thread in that group

# sigaltstack
```c
    int sigaltstack(const stack_t *_Nullable restrict ss, 
                    stack_t *_Nullable restrict old_ss);
```
Allows a thread to define a new alternate signal stack and/or retrieve the state of an existing alternate signal stack, it is normally used by allocating memory for a new stack,
using `sigaltstack` to inform the system of it's existence and location, then use `SA_ONSTACK` flag on `sigaction`, 

```c
    typedef struct {
        void  *ss_sp;     /* Base address of stack */
        int    ss_flags;  /* Flags */
        size_t ss_size;   /* Number of bytes in stack */
    } stack_t;
```
`ss.flags` can have either 0 or the following flag : 
- `SS_AUTODISARM` : Clear the alternate signal stack settings on entry to the signal handler, when it returns, the previous signal stack settings are restored
`ss.ss_sp` has the address of the stack and `ss_size` the size of the stack, `old_ss.flags` can also have one of the following values:
- `SS_ONSTACK`    : The thread is currently executing on the alternate signal stack
- `SS_DISABLE`    : The alternate signal stack is currently disabled 
- `SS_AUTODISARM` : The alternate signal has been marked to be autodisarmed as described above
Returns 0 on success or -1 on error

# kill 
```c
    int kill(pid_t pid, int sig);
```
Used to send any signal to any process group or process
- If `pid` is 0, sends the signal to all processes in the process group of the calling process
- If `pid` is -1, sends the signal to every process the calling process has permission to send signals to, except for process 1
- If `pid` is less than -1, then `sig` is sent to every process in the process group whose ID is `-pid`
Returns 0 on success and -1 on error

# tkill
```c
    [[deprecated]] int tkill(pid_t tid, int sig);
```
Sends the signal `sig` to the thread ID `tid`, it's a obsolete predecessor to `tgkill`,
as it can end up killing the wrong thread if the old one is terminated and the ID is recycled

# tgkill
```c
    int tgkill(pid_t tgid, pid_t tid, int sig);
```
Sends the signal `sig` to the thread ID `tid` in the thread group `tgid`, returns 0 on sucess and -1 on error
