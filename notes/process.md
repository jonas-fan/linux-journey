# Process

- [Executable binary format](#executable-binary-format)
- [Virtual memory layout](#virtual-memory-layout)
- [Process creation and termination](#process-creation-and-termination)
- [Program execution](#program-execution)

## Executable binary format

Processes are running executables. The binary format of executable is `Executable and Linking Format (ELF)`, which is the replacement of legacy formats, such as `Assembler output (a.out)` and `Common Object File Format (COFF)`.

```bash
$ file $(which file)
/usr/bin/file: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked,
    interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0,
    BuildID[sha1]=895077a840385f977d10a2402d58f8b4edc8bd48, stripped
```

## Virtual memory layout

Program consists of sections. Let's have a look:

```bash
$ size $(which file)
text       data     bss     dec     hex filename
  17335    2264     112   19711    4cff /usr/bin/file
```

The following table briefly describes the sections:

| Segment                       | Description                                                                                                                                                                    |
|-------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Text (or code)                | read-only, machine-dependent instructions                                                                                                                                      |
| Data                          | user-initialized (or initialized) data segment, storing those `static` and `global` variables which have an explicit initialization with a value                               |
| Block started by symbol (BSS) | zero-initialized (or uninitialized) data segment, storing those `static` and `global` variables which are not initialized explicitly, they will be set to zero                 |
| Heap                          | containing dynamic allocated memory, shared between all threads in a process                                                                                                   |
| Stack                         | storing function parameters and return address, and also automatic variables which are local variables and will be created/destroyed when program flow enters/leaves the scope |

On Intel x86-64 architecture, a process's virtual memory looks like (see [Linux kernel mm (x86_64)](https://www.kernel.org/doc/Documentation/x86/x86_64/mm.txt) for more details):

```
0xffffffffffffffff  +---------------+
                    |               |
     128 TiB        |    kernel     |  shared between all processes
   (2^47 bytes)     |               |  all processes
                    |               |
0xffff7fffffffffff  |---------------|
                    |               |
    ~16M TiB        | non-canonical |
                    |               |
0x00007fffffffffff  |---------------|
                    |   argv, env   |
                    |---------------|
                    |               |
                    |    stack      |
                    |               |
                    |---------------|  <- top of stack
                    |               |
                    |               |
     128 TiB        |               |
   (2^47 bytes)     |---------------|  <- program break
                    |               |
                    |     heap      |
                    |               |
                    |---------------|  <- end
                    |     bss       |
                    |---------------|  <- edata
                    |     data      |
                    |---------------|  <- etext
                    |     text      |
0x0000000000400000  |---------------|
                    |               |
0x0000000000000000  +---------------+
```

## Process creation and termination

The `fork()`-like system calls are used to create child processes based on the current running process.

```
  Parent               Child
    |
    |
  fork() ----create------+
    |                    |
    |                 execve()
    |                    |
    |                    |
    |                    |
  wait() <---status--- exit()
    |
    |
    v
```

`fork()`, `vfork()`, and `clone()` will eventually call `do_fork()` in kernel space. A forked process shares parts of the resources of the parent. A child process created by `fork()` or `vfork()` starts from the forking point, a child process created by `clone()` starts from a function.

It's worth noting that we are unable to rely on the order in which the parent and child are next scheduled to use CPU resource.

A parent process exits, its child processes will be orphan, which will be managed by `init` process (pid 1). Once a child process terminates, it will pass a signal `SIGCHLD` (or `SIGCLD`) to its parent process, and then will become a zombie. A `wait()`-like system call is required to retrieve the status returned from the zombie and reclaim the resource of the zombie.

The `wait()` system call and variants are as below:

- `wait()`
- `waitpid()`
- `waittid()`

The following macros can be used to investigate termination status.

| Call                   | Description                                                             |
|------------------------|-------------------------------------------------------------------------|
| `WIFEXITED(status)`    | test if normal termination                                              |
| `WIFSIGNALED(status)`  | test if killed by signal, `WTERMSIG(status)` returns the signal number  |
| `WIFSTOPPED(status)`   | test if stopped by signal, `WSTOPSIG(status)` returns the signal number |
| `WIFCONTINUED(status)` | test if continued by signal                                             |

Bit fields of termination status.

```
Normal termination:
15              8 7               0
+----------------+----------------+
|  exit status   |        0       |
+----------------+----------------+


Killed by signal:
15              8 7               0
+----------------+----------------+
|       0        | |   signal     |
+----------------+----------------+
                  +---> core dumped flag

Stopped by signal:
15              8 7               0
+----------------+----------------+
|     signal     |      0x7F      |
+----------------+----------------+


Continued by signal:
15              8 7               0
+----------------+----------------+
|      0xFF      |      0xFF      |
+----------------+----------------+
```

An explicit call of `exit()` can be used to termiate the current process, it will do resources cleanup, flush buffers, and call registered exit handlers, then invoke `_exit()` (UNIX-specific).

When a process terminates, registered exit handlers will be executed in reverse order as a stack. They are registered by the following:

- `atexit()` (C standard)
- `on_exit()` (glibc extension)

## Program execution

With `exec()` family functions, a program can replace its running instance with an `ELF` file or a script, including customized arguments and environments. Because that is replaceing, the process id will not be changed.

A script is able to specifically indicate an interrupter by writting `#!<interrupt-path> <optional-arg>` in the first line of itself.

| Call       | Name (-, p) | Argument (v, l) | Environment (e, -) |
|------------|-------------|-----------------|--------------------|
| `execve()` | pathname    | array           | envp               |
| `execle()` | pathname    | list            | envp               |
| `execv()`  | pathname    | array           | caller's environ   |
| `execl()`  | pathname    | list            | caller's environ   |
| `execvp()` | filename    | array           | caller's environ   |
| `execlp()` | filename    | list            | caller's environ   |
