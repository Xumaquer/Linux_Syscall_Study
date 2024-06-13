# mkdir
```c
    int mkdir(const char *pathname, mode_t mode);
```
Creates a directory named `pathname` with `mode` access flags (see `open`), returns 0 on success and -1 on error


# mkdirat
```c
    int mkdirat(int dirfd, const char *pathname, mode_t mode);
```
Similar to `mkdir` but uses `dirfd` as the current directory for relative `pathname`

# rmdir 
```c
    int rmdir(const char *pathname);
```
Deletes a empty directory at `pathname`, returns 0 on success and -1 on error

# getcwd
```c
    char *getcwd(char buf[.size], size_t size);
```
Fills `buf` with the path for the current working directory, returns `buf` on success and NULL on error


# chdir
```c
    int chdir(const char *path);
```
Changes the current working directory of the calling process to the directory specified in `path`, returns 0 on success and -1 on error


# fchdir
Identical to `chdir`, but the directory is given as an open file descriptor



# inotify_init
```c
    int inotify_init(void);
```
Initializes a new inotify instance and returns a file descriptor to it or -1 on error


# inotify_init1
```c
    int inotify_init1(int flags);
```
Similar to `inotify_init` but has a extra `flags` parameter which can be bitwise ORed with the following values :
- `IN_NONBLOCK` : Adds the `O_NONBLOCK` to the open file descriptor (non blocking)
- `IN_CLOEXEC`  : Sets `FD_CLOEXEC` on the open file descriptor (close on `execve`)

# inotify_add_watch
```c
    int inotify_add_watch(int fd, const char *pathname,uint32_t mask);
```
Adds the file or folder at `pathname` to the inotify instance `fd`'s watch list, the watched events are described by `mask` 
which can be bitwise ORed with the following values :
- `IN_ACCESS`        : File was accessed
- `IN_ATTRIB`        : Metadata changed (permissions,timestamp,extended attributes,link count,user/group ID)
- `IN_CLOSE_WRITE`   : File opened for writing was closed
- `IN_CLOSE_NOWRITE` : File or directory not opened for writing was closed
- `IN_CREATE`        : File/directory created in watched directory
- `IN_DELETE`        : File/directory was deleted
- `IN_DELETE_SELF`   : Watched file/directory was deleted (generates `IN_IGNORED` event)
- `IN_MODIFY`        : File was modified 
- `IN_MOVE_SELF`     : Watched file/directory was itself moved
- `IN_MOVED_FROM`    : Generated for the directory containing the old filename when a file is renamed
- `IN_MOVED_TO`      : Generated for the directory containing the new filename when a file is renamed
- `IN_OPEN`          : File or directory was opened

Also, the inotify instance can be `read` filling a `struct inotify_event`
```c
    struct inotify_event {
        int      wd;       /* Watch descriptor */
        uint32_t mask;     /* Mask describing event */
        uint32_t cookie;   /* Unique cookie associating related
                                events (for rename(2)) */
        uint32_t len;      /* Size of name field */
        char     name[];   /* Optional null-terminated name */
    };
```