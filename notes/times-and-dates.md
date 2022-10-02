# Times and Dates

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
