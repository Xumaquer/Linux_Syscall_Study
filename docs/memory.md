

# mmap
```c
    void *mmap(void addr[.length], size_t length, int prot, int flags, int fd, off_t offset);
```
Creates a new mapping in the virtual address space of the calling process. The starting address is specified in `addr`, with 
`length` bytes, if `addr` is NULL the kernel chooses the address, maps at `offset` file position of `fd`, `offset` must be a multiple of 
`sysconf(_SC_PAGE_SIZE)`, after `mmap` returns a pointer to the mapped area or `MAP_FAILED` on error (`(void*)-1`), closing `fd` afterwards does not
invalidate the mapped region, `prot` argument described the desired memory protection of the mapping, which is either `PROT_NONE`or a bitwise OR of one or more flags :
- `PROT_NONE` : Pages may not be accessed
- `PROT_EXEC` : Pages may be executed
- `PROT_READ` : Pages may be read
- `PROT_WRITE`: Pages may be written

The `flags` argument can only have one of the following flags :
- `MAP_SHARED`          : Share this mapping, updates on it are visible to other processes mapping the same region and are carried through the underlying file
- `MAP_SHARED_VALIDATE` : This flag provides the same behavior as `MAP_SHARED`, but fails if there are any unknown flags in `flags`
- `MAP_PRIVATE`         : Create a private copy-on-write mapping, updates to the mapping are not carried through the underlying file.

Additionally, zero or more of the following values can be ORed in `flags`: 
- `MAP_32BIT`           : Put the mapping into the first 2 Gigabytes of the process address space, supported only on x86-64 64 bit programs
- `MAP_ANONYMOUS`       : The mapping is not backed by a file, so `fd` is ignored, but some implementations require it to be -1 and `offset` should be 0
- `MAP_ANON`            : Synonym for `MAP_ANONYMOUS`
- `MAP_DENYWRITE`       : This flag is ignored
- `MAP_EXECUTABLE`      : This flag is ignored
- `MAP_FIXED`           : Don't interpret `addr` as a hint, maps exactly at that address, `addr` must be suitably aligned
- `MAP_FIXED_NOREPLACE` : Similar to `MAP_FIXED` but never clobbers a preexisting mapped range
- `MAP_GROWSDOWN`       : This flag is used for stacks, it indicates to the kernel virtual memory system that the mapping should extend downward in memory
- `MAP_HUGETLB`         : Allocates the mapping using "huge" pages
- `MAP_HUGE_2MB`        : Used alongside `MAP_HUGETLB`to select alternative hugetlb page size
- `MAP_HUGE_2GB`        : Same as `MAP_HUGE_2MB`but for 2GB
- `MAP_LOCKED`          : Mark the mapped region to be locked in the same way as `mlock`
- `MAP_NONBLOCK`        : This flag is only meaningful when used with `MAP_POPULATE`. Don't perform read-ahead, create page tables only for pages that are already in RAM.
- `MAP_NORESERVE`       : Do not reserve swap space for this mapping, when swap space is reserved, one has the guarantee that it is possible to modify the mapping.
- `MAP_POPULATE`        : Populate (prefault) page tables for the mapping, for file mappings it causes read-ahead on the file. Which will help reduce blocking on page faults later.
- `MAP_STACK`           : Allocates the mappings at an address suitable for a process or thread stack
- `MAP_SYNC`            : This flag is available only with the `MAP_SHARED_VALIDATE` mapping type, only supported for files supporting DAX (direct mapping of persistent memory)
- `MAP_UNINITIALIZED`   : Don't clear anonymous pages, inteded to improve performance on embedded devices, only honored if the kernel was configured with `CONFIG_MMAP_ALLOW_UNINITIALIZED`

# mremap
```c
    void *mremap(void old_address[.old_size], size_t old_size, 
                 size_t new_size, int flags, ... /*void *new_address*/);
```
Expands or shrinks an existing memory mapping at `old_address` with `old_size` to `new_size`, returns the address of the new mapping on success or
`MAP_FAILED` on error, the `flags` parameter can be a bitmask of 0 or more of the following flags : 
- `MREMAP_MAYMOVE`  : By default, if there is not sufficient space to expand it in the current location, it fails, with this flags the kernel can relocate the mapping to a new virtual address
- `MREMAP_FIXED`    : Similar to `MAP_FIXED` of `mmap`, if this is specified then `void *new_address` specifies the page-aligned address to move the mapping.
- `MREMAP_DONTUNMAP`: This flag which must be used with `MREMAP_MAYMOVE`, remaps a mapping to a new address without unmapping the one at `old_address`

# munmap
```c
    int munmap(void addr[.length], size_t length);
```
Deletes the mappings for the specified address range, `addr` must be a multiple of page size but `length` does not need to. 
Returns 0 on success and -1 on error

# mprotect
```c
    int mprotect(void addr[.len], size_t len, int prot);
```
Changes the access protections for the calling process's memory pages in the interval [`addr`,`addr + len - 1`],
`prot` can be `PROT_NONE`or a bitwise OR of other values in the following flags:
- `PROT_NONE`     : Pages may not be accessed
- `PROT_EXEC`     : Pages may be executed
- `PROT_READ`     : Pages may be read
- `PROT_WRITE`    : Pages may be written
- `PROT_SEM`      : Memory can be used for atomic operations (currently unused on any architecture)
- `PROT_SAO`      : Memory should have a strong access ordering (specific to PowerPC architecture)
- `PROT_GROWSUP`  : Apply the protection mode up to the end of a mapping that grows upwards
- `PROT_GROWSDOWN`: Apply the protection mode down to the beggining of a mapping that grows downard
Returns 0 on success and -1 on error

# msync
```c
    int msync(void addr[.length], size_t length, int flags);
```
Flushes changes made to the in-core copy of a file that was mapped into memory using `mmap` back to the filesystem, without this there is no guarantee that changes are written back before
`munmap` is called, the `flags` argument should specify exactly `MS_ASYNC` or `MS_SYNC`and may additionaly include `MS_INVALIDATE` : 
- `MS_ASYNC`     : Specifies that an update be scheduled, but the call returns immediately
- `MS_SYNC`      : Requests an update and waits for it to complete
- `MS_INVALIDATE`: Asks to invalidate other mappings of the same file (so that they can be updated with the fresh values just written)
Returns 0 on success and -1 on error


# mincore
```c
    int mincore(void addr[.length], size_t length, unsigned char *vec);
```
Returns a vector that indicates whether pages of the calling process are in RAM and will not cause a disk accesss (page fault) if referenced.
`addr` must be a multiple of the system page size, `length` doesn't need to but is effectively rounded up to the next multiple of the page size, `vec` must point to an array containing at least 
`(length+PAGE_SIZE-1) / PAGE_SIZE` bytes, the least significant bit will be set if the corresponding page is resident in memory
Returns 0 on success or -1 on error

# madvise
```c
    int madvise(void addr[.length], size_t length, int advice);
```
Gives advice or directions to the kernel about the address range in `addr` with size `length`, this function operates on whole pages so `addr` must be page-aligned.
`length` is rounded up to a multiple of a page size, the `advice` parameter can be one of the following
- `MADV_NORMAL`        : No special treatment, this is the default
- `MADV_RANDOM`        : Expects page references in random order (Hence, read ahead may be less useful than normally)
- `MADV_SEQUENTIAL`    : Expect page references in sequential order (Hence, pages in the range can be aggressively read ahead, and may be freed soon after accessed)
- `MADV_WILLNEED`      : Expect access in the near future (Hence, it might be a good idea to read some pages ahead)
- `MADV_DONTNEED`      : Do not expect access in the near future (For the time being, the application is finished with the given resources, so the kernel can free them)
- `MADV_REMOVE`        : Free up a given range of pages and it's associated backing store
- `MADV_DONTFORK`      : Do not make the pages in this range available to the child after a `fork`
- `MADV_DOFORK`        : Undo the effect of `MADV_DONTFORK`
- `MADV_HWPOISON`      : Poison the pages in the range, handle subsequent references to those pages like a hardware memory corruption (only available for processes with `CAP_SYS_ADMIN`)
- `MADV_MERGEABLE`     : Enable kernel samepage merging (KSM), the kernel regularly scan those areas looking for page with the same content and replaces them by a single write-protected page
- `MADV_UNMERGEABLE`   : Undo the effect of `MADV_MERGEABLE`, both flags can only be used if the kernel was configured with `CONFIG_KSM`
- `MADV_SOFT_OFFLINE`  : Soft offline the pages in the range specified, the memory of each page is preserved and the original page is offlined, feature intended for testing memory error-handling code
- `MADV_HUGEPAGE`      : Enables Transparent Huge Pages (THP), the kernel will regurlarly scan areas marked as huge page candidates to replace them with huge pages.
- `MADV_NOHUGEPAGE`    : Ensures that memory in the address range will not be backed by transparent huge pages.
- `MADV_COLLAPSE`      : Performs a best-effort synchronous collapse of the native pages mapped by the memory range into Transparent Huge Pages
- `MADV_DONTDUMP`      : Exclude from a core dump those pages in the range specified. This is useful in applications that have large areas of memory known not to be useful in a core dump
- `MADV_DODUMP`        : Undo the effect of an earlier `MADV_DONTDUMP`
- `MADV_FREE`          : The application no longer requires the pages, the kernel can thus free them but that can be delayed until memory pressure occurs
- `MADV_WIPEONFORK`    : Present the child process with zero-filled memory in this range after a `fork`
- `MADV_KEEPONFORK`    : Undo the effect of an earlier `MADV_WIPEONFORK`
- `MADV_COLD`          : Deactivate a given range of pages
- `MADV_PAGEOUT`       : Reclaim a given range of pages
- `MADV_POPULATE_READ` : Populate page tables readable, faulting in all pages just as if manually reading from each page
- `MADV_POPULATE_WRITE`: Populate page tables writeable, faulting in all pages just as if manually writing from each page
Returns 0 on success and -1 on error

# mlock
```c
    int mlock(const void addr[.len], size_t len);
```
Locks the pages in the range starting at `addr` and continuing for `len` bytes, preventing them from being paged to the swap area
Returns 0 on success or -1 on error

# mlock2
```c
    int mlock2(const void addr[.len], size_t len, unsigned int flags);
```
Similar to `mlock`, but has a extra `flags` that can be 0 or the following constant
-  `MLOCK_ONFAULT`: Locks pages that are currently resident and mark the entire range so that the remaining nonresident pages are locked

# mlockall
```c
    int mlockall(int flags);
```
Similar to `mlock` but locks all 
- `MCL_CURRENT` : Lock all pages which are currently mapped into address space of the process
- `MCL_FUTURE`  : Lock all pages which all become mapped into the address space of the process in the future
- `MCL_ONFAULT` : Used together with `MCL_CURRENT`, `MCL_FUTURE` or both. Mark all mappings to lock pages when they are faulted in.

# munlock
```c
    int munlock(const void addr[.len], size_t len);
```
Unlocks the pages in the address range starting at `addr` and continuing for `len` bytes

# munlockall
```c
    int munlockall(void);
```
Unlocks all pages mapped into the address space of the calling process

# brk
```c
    void *brk(void *addr);
```
Changes the location of the `program break` which defines the end of the process's data segment. 
Increasing the program break has the effect of allocating memory to the process, returns the new program break on success and old program break on error




# process_vm_readv
```c
    ssize_t process_vm_readv(pid_t pid,
                             const struct iovec *local_iov,
                             unsigned long iovcnt,
                             const struct iovec *remote_iov,
                             unsigned long riovcnt,
                             unsigned long flags);
```
Reads the virtual memory from `pid` process from buffers specified at `remote_iov` with `riovcnt` elements into buffers specified at `local_iov` with `iovcnt` elements, 
`flags` is reserved for future use and should be 0, returns the number of read bytes or -1 on error


# process_vm_writev
```c
    ssize_t process_vm_writev(pid_t pid,
                             const struct iovec *local_iov,
                             unsigned long iovcnt,
                             const struct iovec *remote_iov,
                             unsigned long riovcnt,
                             unsigned long flags);
```
Writes to the virtual memory from buffers specified at `local_iov` with `iovcnt` elements into buffers specified at `remote_iov` with `riovcnt` elements,
`flags` is reserved for future use and should be 0, returns the number of written bytes or -1 on error


# process_madvise
```c
    ssize_t process_madvise(int pidfd, const struct iovec *iovec, size_t vlen, int advice, unsigned int flags);
```
Similar to `madvise` but is applied to regions specified by elements of `iovec` which are `vlen` elements, `flags` is reserved for future use and should be 0,
`advise` can have the same values as in`mdavise`, returns the number of bytes advised or -1 on error


# process_mrelease
```c
    int process_mrelease(int pidfd, unsigned int flags);
```
Releases memory of a dying process referred by `pidfd` file descriptor, `flags` is for future use and should be 0
The need for this function comes from the possible delay between `SIGKILL` and the process to free up it's memory


