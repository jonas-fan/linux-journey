# Linux Journey

## Table of Contents

- [File I/O](#file-io)

### File I/O

By design, file descriptors and pseudo devices for each process are prepared by system during process startup. Processes are able to directly access them, or by preprocessor symbols.

| Description     | File descriptor | Preprocessor symbol | Pseudo device                                       |
|-----------------|-----------------|---------------------|-----------------------------------------------------|
| Standard input  | `0`             | `STDIN_FILENO`      | `/dev/stdin` (or `/dev/fd/0` -> `/proc/self/fd/0`)  |
| Standard output | `1`             | `STDOUT_FILENO`     | `/dev/stdout` (or `/dev/fd/1` -> `/proc/self/fd/1`) |
| Standard error  | `2`             | `STDERR_FILENO`     | `/dev/stderr` (or `/dev/fd/2` -> `/proc/self/fd/2`) |

For instances,

```bash
$ date | cat -
Mon Jun 20 21:27:23 CST 2022

$ date | diff <(sleep 1 && date) /dev/stdin
1c1
< Mon Jun 20 22:14:35 CST 2022
---
> Mon Jun 20 22:14:34 CST 2022
```
