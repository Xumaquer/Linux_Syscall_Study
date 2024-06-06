# init_module
```c
    int init_module( void module_image[.len],unsigned long len, const char *param_values);
```
Loads an ELF image into kernel space, performs any necessary symbol relocation, initializes module parameters to values provided by the caller, and then runs the module's `init` function,
requires privilege


# finit_module
Similar to `init_module` but uses a file descriptor instead of a name