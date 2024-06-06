# newuname
```c
    //known as just "uname" on glibc
    int newuname(struct utsname *buf);
```
Fills `buf` with system information

```c
    struct utsname {
        char sysname[];    /* Operating system name (e.g., "Linux") */
        char nodename[];   /* Name within communications network
                                to which the node is attached, if any */
        char release[];    /* Operating system release
                                (e.g., "2.6.28") */
        char version[];    /* Operating system version */
        char machine[];    /* Hardware type identifier */
    #ifdef _GNU_SOURCE
        char domainname[]; /* NIS or YP domain name */
    #endif
    };
```