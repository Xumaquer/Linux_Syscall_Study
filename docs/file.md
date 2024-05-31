
# open
```c
int open(const char *pathname, int flags, ... /* mode_t mode */);
```

Opens a file descriptor according to the specified `pathname`, with the specified `flags` (Must include `O_RDONLY`, `O_WRONLY` or `O_RDWR`)
- `O_RDONLY`    : The file can only be read
- `O_WRONLY`    : The file can only be written to
- `O_RDWR`      : The file can be read and written to
- `O_APPEND`    : Before each write, the file is positioned at the end
- `O_ASYNC`     : Enable signal-driven I/O, generates a signal `SIGIO` by default, when input or output becomes possible
- `O_CLOEXEC`   : Closes the file on exec
- `O_CREAT`     : If pathname does not exist, create it, if this flag is used, `mode` parameter must be used
- `O_DIRECT`    : Try to minimize cache effects of the I/O to and from this file.
- `O_DIRECTORY` : If `pathname` is not a directory, the function fails.
- `O_DSYNC`     : Write operations on this file will only return after the output data has been fully transfered, as if followed by a call to `fdatasync`
- `O_EXCL`      : Ensures this call creates the file and fails otherwise, must normally be used with `O_CREAT` except when dealing with block devices.
- `O_LARGEFILE` : Allows files that can't be represented by `off_t` but can by `off64_t` to be opened.
- `O_NOATIME`   : Do not update the file last access time when the file is `read`, the process either needs a `CAP_FOWNER` or the UID of the process match the file's
- `O_NOCTTY`    : If `pathname` refers to a terminal device, it will not become the process's controlling terminal even if the process doesn't have one.
- `O_NOFOLLOW`  : If the trailing component of a `pathname`is a symbolic link, then the open fails
- `O_NONBLOCK`  : When possible, the file is opened in nonblocking mode, neither `open` nor any subsequent I/O operations will block (this flag doesn't affect `select`, `poll` and `epoll`)
- `O_PATH`      : Instead of opening the file, uses it for indicating a location or do operations purely at file descriptor level (all write/read attempts will fail and doesn't require any permission)
- `O_SYNC`      : Write operations on the file will complete after the output data and metadata has been fully transfered, as if followed by a call to `fsync`
- `O_TMPFILE`   : Create a unnamed temporary regular file, the `pathname` argument specifies a directory, if `O_EXCL` is not present, the file can be made permanent with `linkat`
- `O_TRUNC`     : If the file already exists and is a regular file with write access, it will be truncated to length 0

and if `O_CREAT` or `O_TMPFILE` is specified, the `mode` parameter must be used, which can have a bitwise OR of the
following values :
- `S_IRWXU` : User has read,write and execute permission
- `S_IRUSR` : User has read permission
- `S_IWUSR` : User has write permission
- `S_IXUSR` : User has execution permission
- `S_IRWXG` : Group has read,write and execute permission
- `S_IRGRP` : Group has read permission
- `S_IWGRP` : Group has write permission
- `S_IXGRP` : Group has execute permission
- `S_IRWXO` : Others have read,write and execute permission
- `S_IROTH` : Others have read permission
- `S_IWOTH` : Others have write permission
- `S_IXOTH` : Others have execute permission
- `S_ISUID` : Set user ID bit
- `S_ISGID` : Set group ID bit
- `S_ISVTX` : Sticky bit

# creat
```c
int creat(const char *pathname,mode_t mode);
```
Equivalent to a call to `open` with `flags` equal to `O_CREAT|O_WRONLY|O_TRUNC`

# read 
```c
ssize_t read(int fd, void buf[.count], size_t count);
```
Reads `count` bytes in the address `buf` from `fd` file descriptor, returns the number of read bytes or -1 on error

# readv
```c
    ssize_t readv(int fd, const struct iovec *iov, int iovcnt);
```
Similar to `read`, but instead of reading into 1 buffer, reads into `iovcnt` buffers described by pointers to
`struct iovec`

```c
struct iovec {
    void   *iov_base;  /* Starting address */
    size_t  iov_len;   /* Size of the memory pointed to by iov_base. */
};
```

# pread
```c
    ssize_t pread(int fd, void buf[.count], size_t count,off_t offset);
```
Similar to `read`, but starts reading from the specified file `offset` without modyfing the offset afterwards

# preadv
```c
    ssize_t preadv(int fd, const struct iovec *iov, int iovcnt, off_t offset);
```
Combination of `pread` and `readv`, has both an `offset` and arrays of `struct iovec`

# preadv2
```c
    ssize_t preadv2(int fd, const struct iovec *iov, int iovcnt, off_t offset,int flags);
```
Extended version of `preadv`, the `flag` argument contains a bitwise OR of zero or more of the following flags :

- `RWF_DSYNC`  : Provide per-write equivalent of `O_DSYNC`, only works on `pwritev2`
- `RWF_HIPRI`  : High priority read/write, only works with `O_DIRECT`
- `RWF_SYNC`   : Provide per-write equivalent of `O_SYNC`, only works on `pwritev2`
- `RWF_NOWAIT` : Do not wait for data which is not immediately available, only works on `preadv2`
- `RWF_APPEND` : Provide per-write equivalent of `O_APPEND`, 
only works on `pwritev2`

# write
```c
    ssize_t write(int fd, const void buf[.count], size_t count);
```
Writes `count` bytes from the address `buf` in `fd` file descriptor, returns the number of written bytes or -1 on error

# writev
```c
    ssize_t writev(int fd, const struct iovec *iov, int iovcnt);
```
Similar to `write`, but instead of writing 1 buffer, writes `iovcnt` buffers described by pointers to
`struct iovec`(already described in `readv`)

# pwrite
```c
    ssize_t pwrite(int fd, const void buf[.count], size_t count, off_t offset);
```
Similar to `write`, but starts writing from the specified file `offset` without modyfing the offset afterwards

# pwritev
```c
    ssize_t pwritev(int fd, const struct iovec *iov, int iovcnt, off_t offset);
```
Combination of `pwrite` and `writev`, has both an `offset` and arrays of `struct iovec`

# pwritev2
```c
    ssize_t pwritev(int fd, const struct iovec *iov, int iovcnt, off_t offset, int flags);
```
Extended version of `pwritev`, but with a `flag` parameter, that's already been described in the docs of `preadv2`




# stat
```c
    int stat(const char *restrict pathname, struct stat *restrict statbuf);
```
Returns information about a file located in `pathname`, filling `statbuf`, returns 0 on sucess and -1 on error

```c
       struct stat {
           dev_t      st_dev;      /* ID of device containing file */
           ino_t      st_ino;      /* Inode number */
           mode_t     st_mode;     /* File type and mode */
           nlink_t    st_nlink;    /* Number of hard links */
           uid_t      st_uid;      /* User ID of owner */
           gid_t      st_gid;      /* Group ID of owner */
           dev_t      st_rdev;     /* Device ID (if special file) */
           off_t      st_size;     /* Total size, in bytes */
           blksize_t  st_blksize;  /* Block size for filesystem I/O */
           blkcnt_t   st_blocks;   /* Number of 512 B blocks allocated */

           /* Since POSIX.1-2008, this structure supports nanosecond
              precision for the following timestamp fields.
              For the details before POSIX.1-2008, see VERSIONS. */

           struct timespec  st_atim;  /* Time of last access */
           struct timespec  st_mtim;  /* Time of last modification */
           struct timespec  st_ctim;  /* Time of last status change */

       #define st_atime  st_atim.tv_sec  /* Backward compatibility */
       #define st_mtime  st_mtim.tv_sec
       #define st_ctime  st_ctim.tv_sec
       };
```


# lstat
```c
    int lstat(const char *restrict pathname, struct stat *restrict statbuf);
```
Identical to `stat` except that if `pathname` refers to a symbolic link, returns information about the link instead of the referenced file 

# fstat
```c
    int fstat(int fd, struct stat *statbuf);
```
Similar to `stat` but uses a file descriptor instead of a file name


# fstatat 
General interface that can provide the behavior of `stat`, `lstat` and `fstat`, if `pathname` is relative, then it is relative to the directory referred by `dirfd`, has also a extra
`flags` parameter that can have 0 or more of the following flags ORed :
- `AT_EMPTY_PATH`      : If `pathname` is a empty string, operate on the file referred by `dirfd`, in this case `dirfd` can be any type of file, behavior that is similar to `fstat`
- `AT_NO_AUTOMOUNT`    : Don't automount the terminal ("basename") component of `pathname`
- `AT_SYMLINK_NOFOLLOW`: If `pathname` is a symbolic link, do not dereference it, instead return information about the link itself, like `lstat`



# ustat
```c
    [[deprecated]] int ustat(dev_t dev, struct ustat *ubuf);
```
Fills `ubuf` with information about a mounted filesystem. `dev` is a device number identifiying a device containing a mounted file system.
Has been deprecated and is provided only for compatibility. New programs should use `statfs` instead.

```c
    struct ustat{
        daddr_t f_tfree;      /* Total free blocks */
        ino_t   f_tinode;     /* Number of free inodes */
        char    f_fname[6];   /* Filsys name */
        char    f_fpack[6];   /* Filsys pack name */
    }
```

# statfs
```c
    int statfs(const char *path,struct statfs *buf);
```
FIlls `buf` with information about a mounted filesystem. `path` is the pathname of any file within the mounted filesystem.

```c
    struct statfs {
        __fsword_t f_type;    /* Type of filesystem (see below) */
        __fsword_t f_bsize;   /* Optimal transfer block size */
        fsblkcnt_t f_blocks;  /* Total data blocks in filesystem */
        fsblkcnt_t f_bfree;   /* Free blocks in filesystem */
        fsblkcnt_t f_bavail;  /* Free blocks available to
                                unprivileged user */
        fsfilcnt_t f_files;   /* Total inodes in filesystem */
        fsfilcnt_t f_ffree;   /* Free inodes in filesystem */
        fsid_t     f_fsid;    /* Filesystem ID */
        __fsword_t f_namelen; /* Maximum length of filenames */
        __fsword_t f_frsize;  /* Fragment size (since Linux 2.6) */
        __fsword_t f_flags;   /* Mount flags of filesystem
                                (since Linux 2.6.36) */
        __fsword_t f_spare[xxx];
                        /* Padding bytes reserved for future use */
    };
```

The following bit mask flags can be found on `f_flags`:
- `ST_MANDLOCK`   : Mandatory locking is permitted on filesystem 
- `ST_NOATIME`    : Do not update access times
- `ST_NODEV`      : Disallow access to device special files on this filesystem
- `ST_NODIRATIME` : Do not update directory access times
- `ST_NOEXEC`     : Execution of programs is disallowed on this filesystem
- `ST_NOSUID`     : The set-user-ID and set-group-ID bits are ignored by `exec` for executable files on this filesystem
- `ST_RDONLY`     : This filesystem is mounted read-only
- `ST_RELATIME`   : Update atime relative to mtime/ctime
- `ST_SYNCHRONOUS`: Writes are synched to the filesystem immediately
- `ST_NOSYMFOLLOW`: Symbolic links are not followed when resolving paths


# fstatfs 
```c
    int fstatfs(int fd, struct statfs *buf);
```
Similar to `statfs` but uses a file descriptor instead




# select
```c
    int select(int nfds, fd_set *_Nullable restrict readfds,
               fd_set *_Nullable restrict writefds,
               fd_set *_NUllable restrict exceptfds,
               struct timeval *_Nullable restrict timeout);
```
Allows a program to monitor multiple file descriptors, waiting until one or more become "ready" for some class of I/O operation, `nfds` should be set to the highest
file descriptor used in any supplied `fd_set`, the `readfds`, `writefds` and `exceptfds` sets will be cleared of all file descriptors except those that are signaled for
the corresponding event (ready for reading, ready for write, an error occured) and `timeout` specifies the amount of time waited for a event to occur.

```c
    struct timeval {
        time_t      tv_sec;         /* seconds */
        suseconds_t tv_usec;        /* microseconds */
    };
```

```c
    void FD_CLR(int fd, fd_set *set);
    int  FD_ISSET(int fd, fd_set *set);
    void FD_SET(int fd, fd_set *set);
    void FD_ZERO(fd_set *set);
```

`fd_set` is generally implemented as an array of bytes in which each bit indicates a monitored file descriptor and has a fixed limit specified by `FD_SETSIZE` definition which is defined as 1024 on glibc meaning file descriptors greater than 1023 wouldn't work, the definitions above are function-like macros to operate on `fd_set` which should be used for better portability, but modern applications should really use `poll` or `epoll` instead.


# pselect
```c
int pselect(int nfds, fd_set *_Nullable restrict readfds,
                  fd_set *_Nullable restrict writefds,
                  fd_set *_Nullable restrict exceptfds,
                  const struct timespec *_Nullable restrict timeout,
                  const sigset_t *_Nullable restrict sigmask);
```
Similar to `select`, but uses a `struct timespec` instead of `struct timeval`, providing nanosecond precision, and has a extra `sigmask` parameter,
which replaces the current signal mask for this syscall only (restoring the old signal mask after this call)


# poll
```c
    int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

Performs a similar task to `select`, waits for `nfds` file descriptors to become ready to perform I/O for `timeout` milliseconds,
`fds` points to a array of `struct pollfd` with `nfds` elements, the `fd` field should be filled with the file descriptors,
`events` with a bitmask specifying the events of `fd` the application is interested, the function fills `revents`with the events that occured on the respective file descriptor and 
returns the number of elements which had been signaled, 0 on timeout or -1 on error.

The bits that may be set on `events` are the following :
- `POLLIN`    : There is data to read
- `POLLPRI`   : There is some exceptional condition on the file descriptor
- `POLLOUT`   : Writing is now possible
- `POLLDRHUP` : Stream socket peer closed connection or shut down writing half of connection
- `POLLERR`   : Error condition (only returned in `revents`, ignored in `events`)
- `POLLHUP`   : Hang up, on sockets or pipes indicates that the other end closed the connection (only returned in `revents`, ignored in `events`)
- `POLLINVAL` : Invalid request, fd not open (only returned in `revents`, ignored in `events`)
- `POLLRDNORM`: Equivalent to `POLLIN`
- `POLLRDBAND`: Priority band data can be read
- `POLLWRNORM`: Equivalent to `POLLOUT`
- `POLLWRBAND`: Priority data may be written

```c
    struct pollfd {
        int   fd;         /* file descriptor */
        short events;     /* requested events */
        short revents;    /* returned events */
    };
```

# ppoll
```c
    int ppoll(struct pollfd *fds, nfds_t nfds,
             const struct timespec *_Nullable tmo_p,
             const sigset_t *_Nullable sigmask);
```
Similar to `poll`, but uses a `struct timespec` for timeout and applies a `sigmask` during execution while restoring the old sigmask on return, behavior that's similar to `pselect`


