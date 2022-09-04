# Linux Journey

## Table of Contents

- [File I/O](#file-io)
- [Times and Dates](#times-and-dates)
- [Users and Groups](#users-and-groups)

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

## Users and Groups

When shadowing is enabled, the `password` field in the password and group file will be marked as `x`. All of the encrypted passwords are stored in the shadow password file. 

| Description          | Location        | Format                                                                         |
|----------------------|-----------------|--------------------------------------------------------------------------------|
| Password file        | `/etc/passwd`   | `<username>:<password>:<uid>:<gid>:<comment>:<home>:<shell>`                   |
| Group file           | `/etc/group`    | `<groupname>:<password>:<gid>:<users>`                                         |
| Shadow password file | `/etc/shadow`   | `<username>:<password>:<last-changed>:<min>:<max>:<warn>:<inactive>:<expired>` |

The `password` field in the shadow password file contains an encrypted passoword. Its format can be:

- `$id$salt$encrypted`
- `$id$rounds=yyy$salt$encrypted` (since glibc 2.7)

Here is an example, see [`crypt`](https://man7.org/linux/man-pages/man3/crypt.3.html) in detail:

```c
// MD5 with a salt 'mysalt'
printf("%s\n", crypt("P@ssw0rd", "$1$mysalt$"));  // $1$mysalt$LSxioYd1q6Rl2LG.3IXwm/

// SHA-256 with a salt 'mysalt'
printf("%s\n", crypt("P@ssw0rd", "$5$mysalt$"));  // $5$mysalt$lQvd3eETPGyCrL3GjXojml9LqHjaW8FHyw3Yfoimej9

// SHA-512 with a salt 'mysalt'
printf("%s\n", crypt("P@ssw0rd", "$6$mysalt$"));  // $6$mysalt$45ntUCBpDXhrwoe9WXVnehWwJlop2aG2j1IWdm5hW06OBiAyP/dpu4eMlgAq7EFHzSy7OUCathAiDUOPgjtuA/
```
