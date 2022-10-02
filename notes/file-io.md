# File I/O

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
--- /dev/fd/12  2022-06-20 22:19:58.000000000 +0800
+++ /dev/stdin  2022-06-20 22:19:58.000000000 +0800
@@ -1 +1 @@
-Mon Jun 20 22:19:59 CST 2022
+Mon Jun 20 22:19:58 CST 2022
```
