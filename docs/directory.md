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



