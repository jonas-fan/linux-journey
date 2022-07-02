# Linux Journey

## Table of Contents

- [File I/O](#file-io)
- [Times and Dates](#times-and-dates)

## File I/O

By design, file descriptors and pseudo devices for each process are prepared by system during process startup. Processes are able to directly access them, or by preprocessor symbols.

| Description     | File descriptor | Preprocessor symbol | Pseudo device                  |
|-----------------|-----------------|---------------------|--------------------------------|
| Standard input  | `0`             | `STDIN_FILENO`      | `/dev/stdin` (or `/dev/fd/0`)  |
| Standard output | `1`             | `STDOUT_FILENO`     | `/dev/stdout` (or `/dev/fd/1`) |
| Standard error  | `2`             | `STDERR_FILENO`     | `/dev/stderr` (or `/dev/fd/2`) |

For instance,

```bash
$ date | cat -
Mon Jun 20 21:27:23 CST 2022

$ date | diff -u <(sleep 1 && date) /dev/stdin
--- /dev/fd/12	2022-06-20 22:19:58.000000000 +0800
+++ /dev/stdin	2022-06-20 22:19:58.000000000 +0800
@@ -1 +1 @@
-Mon Jun 20 22:19:59 CST 2022
+Mon Jun 20 22:19:58 CST 2022
```

## Times and Dates

Time zone:
- All of settings reside in `/usr/share/zoneinfo`
- The system time zone is `/etc/localtime` linking to one of entries in `/usr/share/zoneinfo`
- Configureble by `TZ` environment variable of applications

```bash
$ TZ=":Asia/Taipei" date -R
Sat, 31 Oct 2020 16:41:30 +0800

$ TZ=":Canada/Eastern" date -R
Sat, 31 Oct 2020 04:41:30 -0400
```

Locale:
- All of settings reside in `/usr/share/locale`
- Format `language[_territory[.codeset]][@modifier]`
- Configureble by `LANG` environment variable of applications

```bash
$ LANG=en_US date
Sat Oct 31 17:03:16 CST 2020

$ LANG=de_DE date
Sa 31 Okt 2020 17:03:38 CST
```

The following diagram briefly shows relationships between calendar and broken-down time.

(Excerpt from [The Linux Programming Interface](https://man7.org/tlpi/index.html))

```
* by `TZ` environment variable
+ by locale setting

+----------------+           +--------------------+
|  fixed-format  |           |   user-formated,   |
|     string     |           |  localized string  |
+----------------+           +--------------------+
 ^              ^                     ^ |
 |    asctime() |        strftime()*+ | | strptime()+ 
 |              |                     | v
 |           +-----------------------------+
 |           |         struct tm           |
 | ctime()*  |      broken-down time       |
 |           +-----------------------------+
 |              ^                     ^ |
 |     gmtime() |        localtime()* | | mktime()*
 |              |                     | v
+------------------------------------------+    +------------------+
|                 time_t                   |    |  struct timeval  |
|              calendar time               |    |  calendar time   |
+------------------------------------------+    +------------------+
                    ^ |                                 ^ |
             time() | | stime()          gettimeofday() | | settimeofday()
                    | v                                 | v 
+------------------------------------------------------------------+
|                              Kernel                              |
+------------------------------------------------------------------+
```
