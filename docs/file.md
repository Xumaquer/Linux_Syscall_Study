
# open
```c
int open(const char *pathname, int flags, ... /* mode_t mode */);
```

Opens a file descriptor according to the specified `pathname`, with the specified `flags` (Must include `O_RDONLY`, `O_WRONLY` or `O_RDWR`) : 

- `O_RDONLY` : The file can only be read
- `O_WRONLY` : The file can only be written to
- `O_RDWR`   : The file can be read and written to
- `O_APPEND` : Before each write, the file is positioned at the end
- `O_ASYNC` : Enable signal-driven I/O, generates a signal `SIGIO` by default, when input or output becomes possible
- `O_CLOEXEC` : Closes the file on exec
- `O_CREAT` : If pathname does not exist, create it, if this flag is used, `mode` parameter must be used
- `O_DIRECT` : Try to minimize cache effects of the I/O to and from this file.
- `O_DIRECTORY` : If `pathname` is not a directory, the function fails.
- `O_DSYNC`: Write operations on this file will only return after the output data has been fully transfered, as if followed by a call to `fdatasync`
- `O_EXCL` : Ensures this call creates the file and fails otherwise, must normally be used with `O_CREAT` except when dealing with block devices.
- `O_LARGEFILE` : Allows files that can't be represented by `off_t` but can by `off64_t` to be opened.
- `O_NOATIME`: Do not update the file last access time when the file is `read`, the process either needs a `CAP_FOWNER` or the UID of the process match the file's
- `O_NOCTTY`: If `pathname` refers to a terminal device, it will not become the process's controlling terminal even if the process doesn't have one.
- `O_NOFOLLOW`: If the trailing component of a `pathname`is a symbolic link, then the open fails
- `O_NONBLOCK`: When possible, the file is opened in nonblocking mode, neither `open` nor any subsequent I/O operations will block (this flag doesn't affect `select`, `poll` and `epoll`)
- `O_PATH`: Instead of opening the file, uses it for indicating a location or do operations purely at file descriptor level (all write/read attempts will fail and doesn't require any permission)
- `O_SYNC`: Write operations on the file will complete after the output data and metadata has been fully transfered, as if followed by a call to `fsync`
- `O_TMPFILE`: Create a unnamed temporary regular file, the `pathname` argument specifies a directory, if `O_EXCL` is not present, the file can be made permanent with `linkat`
- `O_TRUNC`: If the file already exists and is a regular file with write access, it will be truncated to length 0

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
Reads `count` bytes in the address `buf` from `fd` file descriptor, returns the number of read bytes or -1 on error

# readv
Similar to `read`, but instead of reading into 1 buffer, reads into `iovcnt` buffers described by pointers to
`struct iovec`

```c
struct iovec {
    void   *iov_base;  /* Starting address */
    size_t  iov_len;   /* Size of the memory pointed to by iov_base. */
};
```

# pread
Similar to `read`, but starts reading from the specified file `offset` without modyfing the offset afterwards

# preadv
Combination of `pread` and `readv`, has both an `offset` and arrays of `struct iovec`

# preadv2
Extended version of `preadv`, the `flag` argument contains a bitwise OR of zero or more of the following flags :

- `RWF_DSYNC` : Provide per-write equivalent of `O_DSYNC`, only works on `pwritev2`
- `RWF_HIPRI` : High priority read/write, only works with `O_DIRECT`
- `RWF_SYNC` : Provide per-write equivalent of `O_SYNC`, only works on `pwritev2`
- `RWF_NOWAIT` : Do not wait for data which is not immediately available, only works on `preadv2`
- `RWF_APPEND` : Provide per-write equivalent of `O_APPEND`, 
only works on `pwritev2`

# write
Writes `count` bytes from the address `buf` in `fd` file descriptor, returns the number of written bytes or -1 on error

# writev
Similar to `write`, but instead of writing 1 buffer, writes `iovcnt` buffers described by pointers to
`struct iovec`(already described in `readv`)

# pwrite
Similar to `write`, but starts writing from the specified file `offset` without modyfing the offset afterwards

# pwritev
Combination of `pwrite` and `writev`, has both an `offset` and arrays of `struct iovec`

# pwritev2
Extended version of `pwritev`, but with a `flag` parameter, that's already been described in the docs of `preadv2`




