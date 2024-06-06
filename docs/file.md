
# open
```c
int open(const char *pathname, int flags, ... /* mode_t mode */);
```

Opens and returns a file descriptor according to the specified `pathname`, with the specified `flags` (Must include `O_RDONLY`, `O_WRONLY` or `O_RDWR`), returning -1 on error
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

# openat
```c
    int openat(int dirfd, const char *pathname, int flags, ... /*mode_t mode*/);
```
Operates in exactly the same way as `open` except for relative paths, it uses the `dirfd` directory as the "current directory"


# openat2
```c
    int openat2(int dirfd, const char *pathname, struct open_how *how, size_t size);
```
Extended version of `openat`, `size` should be set to `sizeof(struct open_how)`, with `how` being a pointer to a `struct open_how`

```c
    struct open_how {
        u64  flags;    /* O_* flags */
        u64  mode;     /* Mode for O_{CREAT,TMPFILE} */
        u64  resolve;  /* RESOLVE_* flags */
        /* ... */
    };
```

As `openat2` may be extended in the future as `size` provides the expected size of `struct open_how`, currently, it provides `flags` and `mode`
which are already present in `open`, and a extra `resolve` bitmask flag that modifies the way in which all componentes of `pathname` are resolved, and
can have the following flags :
- `RESOLVE_BENEATH`     : Do not permit the path resolution to succeed if any component of the resolution is not a descendant of `dirfd`, rejecting absolute symbolic links and `pathname`s
- `RESOLVE_IN_ROOT`     : Treat the directory reffered by `dirfd` as the root directory allowing a program to restrict path resolution on a per-open basis
- `RESOLVE_NO_MAGILINKS`: Disallow all magic-link resolution
- `RESOLVE_NO_SYMLINKS` : Disallow all symbolic link resolution, implies `RESOLVE_NO_MAGILINKS`
- `RESOLVE_NO_XDEV`     : Disallow traversal of mount points during path resolution
- `RESOLVE_CACHED`      : Make the open fail unless all path components are already in the kernel's lookup cache


# memfd_create
```c
    int memfd_create(const char *name, unsigned int flags);
```
Creates a anonymous RAM-based file and returns a file descriptor that refers to it , `name` is only used for debugging purposes and it will be shown at `/proc/self/fd/` prefixed with `memfd:`,
the following values may be bitwise ORed in `flags` to change the behavior : 
- `MFD_CLOEXEC`      : Sets the close-on-exec flag on the new file descriptor
- `MFD_ALLOW_SEALING`: Allow sealing operations on this file
- `MFD_HUGETLB`      : Will be created with hugetlbfs filesystem using huge pages
- `MFD_HUGE_...`     : Used alongside with `MFD_HUGETLB` to select hugetlb page size (`MFD_HUGE_2MB`, `MFD_HUGE_1GB`,etc)
Returns a new file descriptor on success or -1 otherwise


# memfd_secret
```c
    int memfd_secret(unsigned int flags);
``` 
Creates a anonymous RAM-based file and returns a file descriptor that refers to it, the file created with this function is only visible to processes that have access to the
file descriptor, the memory region is also removed from the kernel page tables and only the page tables of the processes holding the file descriptor map the corresponding physical
memory, the following values may be bitwise ORed in `flags`:
- `FD_CLOEXEC` : Sets the close-on-exec flag on the new file descriptor, causing it to be removed on `execve`

Returns the file descriptor on sucesss or -1 on error

# link
```c
    int link(const char *oldpath,const char *newpath);
```
Creates a new hard link to an existing file, a hard link refers to the same file, so it's impossible to tell which file is the "original",
returns 0 on sucess and -1 on error

# linkat
```c
    int linkat(int olddirfd, const char* oldpath
               int newdirfd, const char* newpath, int flags);
```
Similar to `link` but uses `olddirfd` for relative paths in `oldpath` and `newdirfd` for relative paths in `newpath`, 
the following values can be bitwise ORed in `flags`:

- `AT_EMPTY_PATH`     : If `oldpath` is a empty string, create a link to the file referenced by `olddirfd`
- `AT_SYMLINK_FOLLOW` : By default, `linkat` doesn't dereference `oldpath`, this flag changes that behavior


# symlink
```c
    int symlink(const char *target,const char *linkpath);
```
Creates a symbolic link named `linkpath` linked to `target`, symbolic links may contain relative paths (which will be relative to the link)

# symlinkat
```c
    int symlinkat(const char *target, int newdirfd, const char *linkpath);
```
Similar to `symlink` but uses `newdirfd` as the "current directory" for a relative `linkpath`



# access
```c
    int access(const char *pathname, int mode);
```
Checks if the file at `pathname` has the access specified in `mode`, returning 0 on success and -1 on error or check fail, 
`mode` can have the following values: 
- `F_OK` : Checks if the file exists (this value should be used alone)
- `R_OK` : Checks if the user has read permission  (can be used as a bitwise OR mask)
- `W_OK` : Checks if the user has write permission (can be used as a bitwise OR mask)
- `X_OK` : Checks if the user has execution permission (can be used as a bitwise OR mask)

# faccessat
```c
    int faccessat(int dirfd, const char *pathname, int mode);
```
Similar to `access` but uses `dirfd` for relative paths, the POSIX `faccessat` has a extra `flags` parameter which is honored by the linux syscall `faccessat2`

# faccessat2 
```c
    int faccessat(int dirfd, const char *pathname, int mode, int flags);
```
Similar to `faccessat` but has a `flags` parameter which can be a bitwise OR of the following flags:
- `AT_EACCESS`         : Performs access checks using the effective user and group ID
- `AT_SYMLINK_NOFOLLOW`: If `pathname` is a symbolic link, do not dereference it




# pipe
```c
    int pipe(int pipefd[2]);
```
Fills `pipefd` with a both ends of a pipe, with `pipefd[0]` for read and `pipefd[1]` for write, returns 0 on success and -1 on error

# pipe2
```c
    int pipe2(int pipefd[2],int flags);
```
Similar to `pipe` but has a extra `flags` parameter which can be bitwise ORed with the following values : 
- `O_CLOEXEC`           : Sets the close-on-exec flag on the two new file descriptors
- `O_DIRECT`            : Creates a pipe that performs I/O in "packet" mode, each `write` call generates a packet, and each `read` reads a packet
- `O_NONBLOCK`          : Sets the `O_NONBLOCK` file status on both ends of the pipe
- `O_NOTIFICATION_PIPE` : Since Linux 5.8, general notification mechanism is built on top of the pipe where kernel splices notification messages into userspace opened pipes.


# unlink 
```c
    int unlink(const char *pathname);
``` 
Deletes a name from the filesystem, returns 0 on success and -1 on error

# unlinkat 
```c
    int unlinkat(int dirfd, const char *pathname, int flags);
```
Similar to `unlink`, uses `dirfd` for relative filepath
- `AT_REMOVEDIR` : If specified, performs the equivalent of `rmdir` on `pathname`


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


# readahead
Initiates readahead on a file so that subsequent reads will be satisfied from cache and won't block on disk I/O, with `fd` being the file descriptor,
`offset` the starthing point from which data is to be read and `count` the number of bytes, I/O is performed in whole pages.
Returns 0 on success and -1 on error.

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

# ioctl
```c
    int ioctl(int fd, unsigned long request, char *argp);
```
Manipulates the underlying device parameters of special files, `fd` must be a open file descriptor, `request` is a device-dependent code and `argp` is a untyped pointer 
(from the times before `void*`), usually returns 0 on sucess and -1 on error, but some `ioctl` requests may use the return as output and return any nonnegative value on success.


# lseek
```c
    off_t lseek(int fd, off_t offset, int whence);
```
Repositions `fd` 's file offset based on `offset` and the `whence` flag which can be only one of the following : 
- `SEEK_SET`  :  The file offset is set to `offset` bytes
- `SEEK_CUR`  :  The file offset is set to its current location plus `offset` bytes
- `SEEK_END`  :  The file offset is set to the size of the file plus `offset` bytes
- `SEEK_DATA` :  Adjusts the file offset to the next location in the file greater than or equal to `offset` containing data.
- `SEEK_HOLE` :  Adjusts the file offset to the next hole in the file greater than or equal to `offset`.

`lseek` can seek offsets beyond the end of the file, in which subsequent reads are holes that return null bytes until data is written to the gap, thus the reason for 
`SEEK_DATA` and `SEEK_HOLE` to exist, returns the offset from the beggining of the file, or -1 on error 


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


# newfstatat 
```c
    //Known as "fstatat" in glibc
    int newfstatat(int dirfd, const char *restrict pathname,struct stat *restrict statbuf, int flags); 
```
General interface that can provide the behavior of `stat`, `lstat` and `fstat`, if `pathname` is relative, then it is relative to the directory referred by `dirfd`, has also a extra
`flags` parameter that can have 0 or more of the following flags ORed :
- `AT_EMPTY_PATH`      : If `pathname` is a empty string, operate on the file referred by `dirfd`, in this case `dirfd` can be any type of file, behavior that is similar to `fstat`
- `AT_NO_AUTOMOUNT`    : Don't automount the terminal ("basename") component of `pathname`
- `AT_SYMLINK_NOFOLLOW`: If `pathname` is a symbolic link, do not dereference it, instead return information about the link itself, like `lstat`


# statx
```c
    int statx(int dirfd, const char *restrict pathname, int flags,
              unsigned int mask, struct statx *restrict statxbuf);
```
Extended version of `newfstatat`, fills `statxbuf` with information about the file,
 `flags` has all parameters as in `newfstatat` and addiotional ones for controlling the synchronization done when querying : 
 - `AT_STATX_SYNC_AS_STAT` : Do the same synchronization as `stat` does, this is the default and is very much filesystem-specific
 - `AT_STATX_FORCE_SYNC`   : Force the attributes to be synchronized with the server, this may require a network filesystem to do a data writeback to get correct timestamps
 - `AT_STATX_DONT_SYNC`    : Don't synchronize anything, just take whatever the system has in cache

 `mask` is a argument which specifies the fields of `struct statx` that the user wants to be filled, it can be a bit mask OR of zero or more of the following :
 - `STATX_TYPE`        : Wants `stx_mode & S_FIMT`
 - `STATX_MODE`        : Wants `stx_mode & ~S_FIMT`
 - `STATX_NLINK`       : Wants `stx_nlink`
 - `STATX_UID`         : Wants `stx_uid`
 - `STATX_GID`         : Wants `stx_gid`
 - `STATX_ATIME`       : Wants `stx_atime`
 - `STATX_MTIME`       : Wants `stx_mtime`
 - `STATX_CTIME`       : Wants `stx_ctime`
 - `STATX_INO`         : Wants `stx_ino`
 - `STATX_SIZE`        : Wants `stx_size`
 - `STATX_BLOCKS`      : Wants `stx_blocks`
 - `STATX_BASIC_STATS` : All of the above
 - `STATX_BTIME`       : Wants `stx_btime`
 - `STATX_ALL`         : Same as `STATX_BASIC_STAT` | `STATX_BTIME`
 - `STATX_MNT_ID`      : Wants `stx_mnt_id`
 - `STATX_DIOALIGN`    : Wants `stx_dio_mem_align`


```c
    struct statx_timestamp {
        __s64 tv_sec;    /* Seconds since the Epoch (UNIX time) */
        __u32 tv_nsec;   /* Nanoseconds since tv_sec */
    };

    struct statx {
        __u32 stx_mask;        /* Mask of bits indicating
                                    filled fields */
        __u32 stx_blksize;     /* Block size for filesystem I/O */
        __u64 stx_attributes;  /* Extra file attribute indicators */
        __u32 stx_nlink;       /* Number of hard links */
        __u32 stx_uid;         /* User ID of owner */
        __u32 stx_gid;         /* Group ID of owner */
        __u16 stx_mode;        /* File type and mode */
        __u64 stx_ino;         /* Inode number */
        __u64 stx_size;        /* Total size in bytes */
        __u64 stx_blocks;      /* Number of 512B blocks allocated */
        __u64 stx_attributes_mask;
                                /* Mask to show what's supported
                                    in stx_attributes */

        /* The following fields are file timestamps */
        struct statx_timestamp stx_atime;  /* Last access */
        struct statx_timestamp stx_btime;  /* Creation */
        struct statx_timestamp stx_ctime;  /* Last status change */
        struct statx_timestamp stx_mtime;  /* Last modification */

        /* If this file represents a device, then the next two
            fields contain the ID of the device */
        __u32 stx_rdev_major;  /* Major ID */
        __u32 stx_rdev_minor;  /* Minor ID */

        /* The next two fields contain the ID of the device
            containing the filesystem where the file resides */
        __u32 stx_dev_major;   /* Major ID */
        __u32 stx_dev_minor;   /* Minor ID */

        __u64 stx_mnt_id;      /* Mount ID */

        /* Direct I/O alignment restrictions */
        __u32 stx_dio_mem_align;
        __u32 stx_dio_offset_align;
    };
```


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
Fills `buf` with information about a mounted filesystem. `path` is the pathname of any file within the mounted filesystem.

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


# readlink
```c
    ssize_t readlink(const char *restrict pathname, char *restrict buf, size_t bufsiz);
```
Places the contents of the symbolic link `pathname`in the buffer `buf` which has size `bufsiz`, does not append a null byte to `buf` and silently truncates the contents
(to length of `bufsiz` characters) in case the buffer is too small to hold all of the contents, you can use `lstat` and check `stat.st_size` to check for the exact size
on sucess returns the number of read bytes (if the returned value equals `bufsiz` then truncation may have happened) or -1 on error

# readlinkat 
```c
    ssize_t readlinkat(int dirfd, const char *restrict pathname, char *restrict buf, size_t bufsiz);
```
Similar to `readlink` but uses `dirfd` as the current directory for relative paths


# rename
```c
    int rename(const char *oldpath, const char *newpath);
```
Renames a file, moving it between directories if needed (so this function can be used as both rename and move),
returns 0 on success and -1 on error


# renameat
```c
    int renameat(int olddirfd, const char *oldpath,
                 int newdirfd, const char *newpath);
```
Similar to `rename` but uses `olddirfd` file descriptor as the current directory for relative `oldpath` and `newdirfd` as the current directory for relative `newpath`

# renameat2
```c
    int renameat2(int olddirfd, const char *oldpath,
                  int newdirfd, const char *newpath, unsigned int flags);
```
Has a additional `flags` argument which is a bit mask consisting of zero or more of the following flags : 
- `RENAME_EXCHANGE`  : Atomically exchange `oldpath` and `newpath`
- `RENAME_NOREPLACE` : Don't overwrite `newpath` of the rename, returns an error if `newpath` already exists
- `RENAME_WHITEOUT`  : Creates a "whiteout" object as the source of the rename at the same time (only meaningful on union/overlay file systems)



# chmod
```c
    int chmod(const char *pathname, mode_t mode);
```
Changes a file's mode bits (file permissions) the same flags described in `open` syscall, returns 0 on sucess or -1 on error

# fchmod
Same as `chmod` but works with a file descriptor instead

# fchmodat 
```c
    int fchmodat(int dirfd, const char *pathname, mode_t mode, int flags);
```
Similar to `chmod` but uses `dirfd` as the current directory for relative paths, `flags` can also have the following value
- `AT_SYMLINK_NOFOLLOW` : If `pathname` is a symbolic link, do not dereference it, operate on the link itself


# dup
```c
    int dup(int oldfd);
```
Returns a new file descriptor that refers to the same open file description as `oldfd or -1 on error

# dup2
```c
    int dup2(int oldfd, int newfd);
```
Similar to `dup` but instead of using the lowest-numbered unusued file descriptor, it uses the number in `newfd`, so `newfd` is adjusted to refer
to the same open file descriptor as `oldfd` (this can be useful to redirect `stdin`, `stdout` and `stderr`)

# dup3
```c
    int dup3(int oldfd, int newfd, int flags);
```
Similar to `dup2` but has a extra `flags` parameter that can have the `O_CLOEXEC` flag



# close 
```c
    int close(int fd);
```
Closes a file descriptor, so that it no longer refers to any file and may be reused, returns zero on success or -1 on error

# close_range
```c
    int close_range(unsigned int first,unsigned int last, unsigned int flags);
```
Closes all open file descriptors from `first` to `last` , `flags` is a bitmask containing 0 or more of the following :
- `CLOSE_RANGE_CLOEXEC` : Sets the close-on-exec flag on the range instead of immediately closing them
- `CLOSE_RANGE_UNSHARE` : Unshare the specified file descriptor from any other processes before closing them, avoiding races with other threads sharing the file descriptor table
Returns 0 on success and -1 on error



# select
```c
    int select(int nfds, fd_set *_Nullable restrict readfds,
               fd_set *_Nullable restrict writefds,
               fd_set *_NUllable restrict exceptfds,
               struct timeval *_Nullable restrict timeout);
```
Allows a program to monitor multiple file descriptors, waiting until one or more become "ready" for some class of I/O operation, `nfds` should be set to the highest
file descriptor used in any supplied `fd_set`, the `readfds`, `writefds` and `exceptfds` sets will be cleared of all file descriptors except those that are signaled for
the corresponding event (ready for reading, ready for write, an error occured) and `timeout` specifies the max amount of milliseconds that should be waited until a event occurs.
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


# epoll_create
```c
    int epoll_create(int size);
```
Creates a new `epoll` instance, the `size` argument is ignored but must be greater than zero, returns a file descriptor, or -1 on error.


# epoll_create1
```c
    int epoll_create1(int flags);
```
Similar to `epoll_create`, the following value can be included in `flags`
- `EPOLL_CLOEXEC` : Set the close-on-exec `FD_CLOEXEC` flag in the new file descriptor.


# epoll_ctl
```c
    int epoll_ctl(int epfd, int op, int fd,struct epoll_event *event);
```
Adds or removes a epoll instance to `epollfd`, `fd` is the file descriptor operated,  `op` is the operation being done and `event` is the event to operate with,
returns 0 on sucess and -1 on error, the `op` parameter can have one of the following values:
- `EPOLL_CTL_ADD` : Adds a entry to the epoll file descriptor
- `EPOLL_CTL_MOD` : Modifies the settings with `fd` to the new settings specified in `event`
- `EPOLL_CTL_DEL` : Removes the target file descriptor `fd` from the interest list

```c
    struct epoll_event {
        __poll_t  events; /* Event flag */
        __u64     data;  /* Data returned */
    };
```

The `data` field is the data that should be returned by `epoll_wait` when this file descriptor becomes ready,
the `events` field is a bit mask composed by ORing zero or more event types
- `EPOLLIN`        : The associated file is available for read
- `EPOLLOUT`       : The associated file is available for write
- `EPOLLRDHUP`     : Stream socket peer closed connection, or shut down writing half of connection
- `EPOLLPRI`       : There is an exceptional condition on the file descriptor
- `EPOLLERR`       : Error condition happened on the associated file descriptor
- `EPOLLHUP`       : Hang up happened on the associated file descriptor
- `EPOLLET`        : Requests edge-triggered notification for the file descriptor
- `EPOLLONESHOT`   : Requests one-shot notification for the file descriptor, which means it should be removed from the list after being notified
- `EPOLLWAKEUP`    : If `EPOLLONESHOT` and `EPOLLET` are clear and the process has `CAP_BLOCK_SUSPEND`, ensures that the system doesn't "suspend" or "hibernate" during `epoll_wait`
- `EPOLLEXCLUSIVE` : Sets an exclusive wakeup mode, so that if waiting for multiple instances of the same `fd`, one or more will receive an event instead of alll

It's also important to mention that registering a entry to epoll, registers the actual kernel object backing the file descriptor, so if you call `close` it might or might not
trigger the epoll subscription cleanup, so it's recommended to call `epoll_ctl` with `EPOLL_CTL_DEL` before closing a file descriptor

# epoll_wait
```c
    int epoll_wait(int epfd, struct epoll_event *events, int maxevent, int timeout);
```
Waits for events on the `epoll` instance referred by `epfd`, the buffer pointed by `events` is filled with information about the events that happened, up to 
`maxevents` are filled by `epoll_wait`, the `timeout` argument specifies the max number of milliseconds that `epoll_wait` will block


# epoll_pwait
```c
    int epoll_pwait(int epfd, struct epoll_event *events, int maxevents, int timeout, const sigset_t *_Nullable sigmask);
```
Similar to `epoll_wait`, but has a extra `sigmask` parameter, which replaces the old sigmask only for the duration of this call 


# epoll_pwait2
```c
    int epoll_pwait2(int epfd, struct epoll_event *events, int maxevents, 
                    const struct timespec *_Nullable timeout, const sigset_t *_Nullable sigmask);
```
Similar to `epoll_pwait` but uses a `struct timespec` instead of `int` for better timing precision on `timeout`



# flock
```c
    int flock(int fd, int operation);
```
Apply or remove an advisory lock on the open file specified by `fd`. The argument `operation` is one of the following : 
- `LOCK_SH`: Place a shared lock, more than one process may hold a shared lock for a given file at a given time
- `LOCK_EX`: Place a exclusive lock , only one process may hold an exclusive lock fior a given file at a given time
- `LOCK_UN`: Remove an existing lock held by this process
Additionally these flags can be ORed :
- `LOCK_NB` : This call is nonblocking
This call blocks if an incompatible lock is held by another process, the lock is released by either using this function with `LOCK_UN` or by closing the file,
locks are preserved across `execve`, returns 0 on success and -1 on error



# fdatasync
```c
    int fdatasync(int fd);
```
Flushes all modified in-core data of the file referenced by `fd`, return 0 on success and -1 on error


# fsync
```c
    int fsync(int fd);
```
Similar to `fdatasync` but also flushes metadata associated with the file


# truncate
```c
    int truncate(const char *path, off_t length);
```
Causes the regular file named by `path` to be truncated to a size of precisely `length` bytes, if it is extended then the extended part reads as null bytes
returns 0 on success and -1 on error


# ftruncate
```c
    int ftruncate(int fd, off_t length);
```
Similar to `truncate` but works on a file descriptor instead