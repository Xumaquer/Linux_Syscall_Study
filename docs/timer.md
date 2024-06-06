
# alarm
```c
    unsigned int alarm(unsigned int seconds);
```
Arranges for a `SIGALRM` to be delivered to the calling process in `seconds` seconds, if `seconds` is zero, any pending alarm is canceled,
returns the number of seconds remaining until any previously scheduled alarm was due to be delivered, or zero otherwise


# getitimer
```c
    int getitimer(int which, struct itimerval *curr_value);
```
Fills the current value of the timer specified by `which` in `curr_value`, returns 0 on success and -1 on error
The argument `which` specifies the timer used and can be one of the following : 
- `ITIMER_REAL`   : This timer counts down in real time. At each expiration, a `SIGALRM`signal is generated
- `ITIMER_VIRTUAL`: This timer counts down against the user-mode CPU time consumed by the process
- `ITIMER_PROF`   : This timer counts down against the total CPU time consumed by the process, at each expiration a `SIGPROF` signal is generated

```c
    struct itimerval {
        struct timeval it_interval; /* Interval for periodic timer */
        struct timeval it_value;    /* Time until next expiration */
    };

    struct timeval {
        time_t      tv_sec;         /* seconds */
        suseconds_t tv_usec;        /* microseconds */
    };
```

# setitimer
```c
    int setitimer(int which, const struct itimerval *restrict new_value,
                  struct itimerval *_Nullable restrict old_value);
```
Sets the current value of the timer specified in `which`, fills `old_value` with the old timer, with 
`new_value` as the new value for the timer, returns 0 on success and -1 on error (for docs about `which` read `getitimer`)




