# Troubleshoot

- [System Metrics](#system-metrics)
    - [proc](#proc)
- [Utilities](#utilities)
    - [ps](#ps)
    - [top](#top)
    - [gstack](#gstack)
    - [strace](#strace)
    - [ltrace](#ltrace)
    - [perf](#perf)
    - [dmesg](#dmesg)
- [Core Files](#core-files)
    - [Capturing application core dumps](#capturing-application-core-dumps)
        - [Using ulimit](#using-ulimit)
        - [Using systemd-coredump](#using-systemd-coredump)
        - [Using gcore](#using-gcore)
    - [Capturing kernel core dumps](#capturing-kernel-core-dumps)
        - [Using kdump](#using-kdump)
        - [Using snapshot dump](#using-snapshot-dump)
    - [Analyzing core dumps](#analyzing-core-dumps)
        - [Using gdb](#using-gdb)
        - [Using crash](#using-crash)
- [Appendix](#appendix)
    - [Setup an environment for analyzing kernel core dumps on Ubuntu/Debian](#setup-an-environment-for-analyzing-kernel-core-dumps-on-ubuntudebian)
    - [Setup an environment for analyzing kernel core dumps on CentOS/RedHat](#setup-an-environment-for-analyzing-kernel-core-dumps-on-centosredhat)
    - [Setup an environment for analyzing kernel core dumps on SuSE](#setup-an-environment-for-analyzing-kernel-core-dumps-on-suse)
    - [System V ABI: AMD64(x86-64)](#system-v-abi-amd64x86-64)

## System Metrics

### [proc](http://man7.org/linux/man-pages/man5/proc.5.html)

The `proc` is a pseudo filesystem which provides an interface to kernel data structures. It is commonly mounted at `/proc`.

#### Main entries in `/proc`

| Entry               | Description                                                              |
|---------------------|--------------------------------------------------------------------------|
| `/proc/<pid>`       | expose process information                                               |
| `/proc/<tid>`       | expose thread information, not visible, same as `/proc/<pid>/task/<tid>` |
| `/proc/self`        | symbolic link to the process's own `/proc/<pid>`                         |
| `/proc/thread-self` | symbolic link to the process's own `/proc/self/task/<tid>`               |
| `/proc/[a-z]*`      | expose system-wide information                                           |

#### Process specific entries in `/proc/<pid>`

| File    | Content                                                                     |
|---------|-----------------------------------------------------------------------------|
| cmdline | command line arguments, separated by null bytes `\0`                        |
| comm    | command, the process's `current->comm`                                      |
| cwd     | symbolic link to the current working directory                              |
| environ | values of environment variables, separated by null bytes `\0`               |
| exe     | symbolic link to the executable of this process                             |
| fd      | directory, which contains all file descriptors                              |
| io      | I/O statistics for the process                                              |
| limit   | process resource limits                                                     |
| maps    | memory maps to executables and library files                                |
| mem     | memory held by this process                                                 |
| root    | symbolic link to the root directory of this process                         |
| stack   | report full stack trace, enable via `CONFIG_STACKTRACE`                     |
| stat    | process status                                                              |
| statm   | process memory status information                                           |
| status  | process status in human readable form                                       |
| task    | directory, which contains all threads information                           |
| wchan   | show the kernel function symbol the task is blocked in (`0` if not blocked) |
| ...     |                                                                             |

## Utilities

### [ps](http://man7.org/linux/man-pages/man1/ps.1.html)

Report a snapshot of current processes.

```bash
# show a process tree
$ ps -efjH

# show threads
$ ps -efT
```

### [top](https://man7.org/linux/man-pages/man1/top.1.html)

Display Linux processes.

```bash
# display all processes or specific processes
$ top [-p <pid>[,...]]

# display a specific process, including its own threads
$ top -Hp <pid>
```

### [gstack](https://linux.die.net/man/1/gstack)

Print a stack trace of a running process.

```bash
# show the stack of a running process
$ gstack <pid>
```

### [strace](http://man7.org/linux/man-pages/man1/strace.1.html)

Trace system calls and signals.

```bash
# get system call statistics of a running process
$ strace -f -c -p <pid>

# trace a running process, including its own threads
$ strace -f -T -ttt -p <pid>

# trace a command
$ strace -f -T -ttt <command>

# trace a command, with proper buffer size
$ strace -f -T -ttt -v -s 4096 <command>
```

### [ltrace](http://man7.org/linux/man-pages/man1/ltrace.1.html)

A library call tracer.

```bash
# trace a running process, including threads
$ ltrace -f -T -ttt -p <pid>
```

### [perf](http://man7.org/linux/man-pages/man1/perf.1.html)

Performance analysis tools for Linux.

```bash
# profile the system, or an existing process
$ perf top [--pid <pid>]

# profile a command
$ perf stat [--event <event>] -- <command>

# profile a command, persist the profiling data (perf.data)
$ perf record [--event <event>] -- <command>

# read profiling data
$ perf report
```

### [dmesg](http://man7.org/linux/man-pages/man1/dmesg.1.html)

Print or control the kernel ring buffer.

```bash
# print the ring buffer
$ dmesg --ctime --decode
```

## [Core Files](https://man7.org/linux/man-pages/man5/core.5.html)

### Capturing application core dumps

#### Using [ulimit](https://man7.org/linux/man-pages/man5/core.5.html)

Make sure resource limitation is disabled, then configure `/proc/sys/kernel/core_pattern` used to name core dumps.

```bash
$ ulimit -c unlimited

$ sudo sysctl --write kernel.core_pattern="core.%e.%p"
kernel.core_pattern = core.%e.%p
```

Let's say your application gets aborted.

```bash
$ sleep 100 &
[1] 643538

$ kill -ABRT $!
[1]+  Aborted                 (core dumped) sleep 100

$ file core.sleep.643538
core.sleep.643538: ELF 64-bit LSB core file, x86-64, version 1 (SYSV), SVR4-style, from 'sleep 100', real uid: 1000, effective uid: 1000, real gid: 1000, effective gid: 1000, execfn: '/usr/bin/sleep', platform: 'x86_64'
```

#### Using [systemd-coredump](https://man7.org/linux/man-pages/man8/systemd-coredump.8.html)

Configure `/proc/sys/kernel/core_pattern` with `systemd-coredump`.

```bash
$ sudo sysctl --write kernel.core_pattern="|/usr/lib/systemd/systemd-coredump %P %u %g %s %t %c %e"
kernel.core_pattern = |/usr/lib/systemd/systemd-coredump %P %u %g %s %t %c %e
```

Again, if your application somehow gets aborted.

```bash
$ sleep 100 &
[1] 643622

$ kill -ABRT $!
[1]+  Aborted                 (core dumped) sleep 100

$ coredumpctl list
TIME                            PID   UID   GID SIG COREFILE  EXE
Tue 2021-07-06 13:53:44 CST  643622  1000  1000   6 present   /usr/bin/sleep

$ coredumpctl dump 643622 --output core.643622
           PID: 643622 (sleep)
           UID: 1000 (jonas)
           GID: 1000 (jonas)
        Signal: 6 (ABRT)
     Timestamp: Tue 2021-07-06 13:53:44 CST (9min ago)
  Command Line: sleep 100
    Executable: /usr/bin/sleep
 Control Group: /user.slice/user-1000.slice/user@1000.service/apps.slice/apps-org.gnome.Terminal.slice/vte-spawn-d4931a52-46db-4c04-9269-853c37421514.scope
          Unit: user@1000.service
     User Unit: vte-spawn-d4931a52-46db-4c04-9269-853c37421514.scope
         Slice: user-1000.slice
     Owner UID: 1000 (jonas)
       Boot ID: be57334cc185403fb53133923cec1e9d
    Machine ID: a969569476df4f5baa85931d178534bc
      Hostname: ub20-x64-dev
       Storage: /var/lib/systemd/coredump/core.sleep.1000.be57334cc185403fb53133923cec1e9d.643622.1625550824000000000000.lz4
       Message: Process 643622 (sleep) of user 1000 dumped core.

                Stack trace of thread 643622:
                #0  0x00007fc712fe6334 __GI___clock_nanosleep (libc.so.6 + 0xe0334)
                #1  0x00007fc712fec047 __GI___nanosleep (libc.so.6 + 0xe6047)
                #2  0x00005604e6f84827 n/a (sleep + 0x5827)
                #3  0x00005604e6f84600 n/a (sleep + 0x5600)
                #4  0x00005604e6f817b0 n/a (sleep + 0x27b0)
                #5  0x00007fc712f2d0b3 __libc_start_main (libc.so.6 + 0x270b3)
                #6  0x00005604e6f8187e n/a (sleep + 0x287e)
```

#### Using [gcore](https://man7.org/linux/man-pages/man1/gcore.1.html)

The `gcore` can be used to obtain a core dump of a running process.

```bash
$ gcore <pid>
```

For example,

```bash
$ sleep 100 &
[1] 643677

$ gcore $!
0x00007fce05518334 in clock_nanosleep () from /lib/x86_64-linux-gnu/libc.so.6
warning: target file /proc/643677/cmdline contained unexpected null characters
warning: Memory read failed for corefile section, 4096 bytes at 0xffffffffff600000.
Saved corefile core.643677
[Inferior 1 (process 643677) detached]
```

### Capturing kernel core dumps

#### Using [kdump](https://www.kernel.org/doc/html/latest/admin-guide/kdump/kdump.html)

Install `kexec-tools` which comprises `kdump`. Make sure memory is reserved (`crashkernel`) and `kdump` is active.

```bash
$ cat /proc/cmdline
BOOT_IMAGE=/vmlinuz-3.10.0-957.el7.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet LANG=en_US.UTF-8

$ systemctl is-active kdump
active
```

And then we are allowed to manually trigger a kernel panic with SysRq. With `kdump`, OS will generate a core dump on the specified location (e.g., `/var/crash/*`), and then perform a reboot automatically.

```bash
$ echo c > /proc/sysrq-trigger
```

#### Using snapshot dump

On certain hypervisors like `VMware`, virtual mahcines snapshost files (e.g., `.vmss`, `.vmsn`, and `.vmem`) can be used to troubleshoot. By [`vmss2core`](https://flings.vmware.com/vmss2core), snapshot files can be converted core dumps.

For newer Linux distors, snapshot files can even be directly loaded for analysis.

```bash
$ crash </path/to/vmlinux> </path/to/vmsn/or/vmss>
```

### Analyzing core dumps

#### Using [gdb](http://man7.org/linux/man-pages/man1/gdb.1.html)

The GNU debugger, usually used to investgate application core dumps.

```bash
# attach a running process
$ gdb -p <pid>

# perform gdb commands on an executable
$ gdb -quite </executable/to/load> -ex "<gdb-cmd>" [-ex "<gdb-cmd>"]

# investigate a core dump with executables and shared objects
$ gdb </binary/to/load>
(gdb) set solib-search-path </path/to/shared/objects>
(gdb) core </coredump/to/load>

# display backtrace
(gdb) bt

# display backtrace of each thread
(gdb) thread apply all bt

# given a thread id, display backtrace
(gdb) thread apply <id> bt
```

#### Using [crash](http://man7.org/linux/man-pages/man8/crash.8.html)

Aka crash-utility, analyze Linux crash dump data or a live system.

- [Setup an environment for analyzing kernel core dumps on Ubuntu/Debian](#setup-an-environment-for-analyzing-kernel-core-dumps-on-ubuntudebian)
- [Setup an environment for analyzing kernel core dumps on CentOS/RedHat](#setup-an-environment-for-analyzing-kernel-core-dumps-on-centosredhat)
- [Setup an environment for analyzing kernel core dumps on SuSE](#setup-an-environment-for-analyzing-kernel-core-dumps-on-suse)

```bash
# check core dump information
$ file vmcore

# or you may like this way
$ strings vmcore | less

# load the core dump with the vmlinux
$ crash </path/to/vmlinux> </path/to/vmcore>

# display help message
crash> help

# display dmesg, the kernel ring buffer
crash> log

# display backtrace
crash> bt

# display all of running processes
crash> ps

# display current task
crash> task [pid]

# display opened file descriptor
crash> files [pid]

# display inode, super block, file type, and full pathanme
crash> files -d <dentry>

# apply each process
crash> foreach <command>

# given a structure and an address, display memory data
crash> <structure-name> <address>

# load the debug version of kernel module
crash> mod -s <name> </path/to/kernel/module>

# given a symbol, display the value
crash> p <symbol>
```

## Appendix

### Setup an environment for analyzing kernel core dumps on Ubuntu/Debian

```bash
# make sure the debug symbol repositories are available
$ sudo cat > /etc/apt/sources.list.d/ddebs.list << END
deb http://ddebs.ubuntu.com $(lsb_release -cs) main restricted universe multiverse
deb http://ddebs.ubuntu.com $(lsb_release -cs)-updates main restricted universe multiverse
deb http://ddebs.ubuntu.com $(lsb_release -cs)-proposed main restricted universe multiverse
END

# import the signing key for debug symbol archives
$ sudo apt install -y ubuntu-dbgsym-keyring
$ sudo apt-get update

# search packages of kernel debug symbols
$ sudo apt-cache search linux-image | grep dbgsym

# install packages
$ sudo apt install -y linux-image-<version>-dbgsym
```

### Setup an environment for analyzing kernel core dumps on CentOS/RedHat

```bash
# make sure the debug symbol repositories are available, e.g., `base-debuginfo`
$ sudo yum --enablerepo=base-debuginfo repolist

# search packages of kernel debug symbols
$ sudo yum --enablerepo=base-debuginfo search --showduplicates \
    kernel-debuginfo | grep 3.10.0-327.el7.x86_64

# install packages
$ sudo yum --enablerepo=base-debuginfo install -y \
    kernel-debuginfo-3.10.0-327.el7.x86_64 \
    kernel-debuginfo-common-x86_64-3.10.0-327.el7.x86_64

# load the core dump with the vmlinux
$ crash /usr/lib/debug/lib/modules/3.10.0-327.el7.el7.x86_64/vmlinux /path/to/vmcore
```

### Setup an environment for analyzing kernel core dumps on SuSE

```bash
# registration is required:
#
#    https://www.suse.com/support/kb/doc/?id=000019587
#
$ sudo SUSEConnect --product "SLES/12-4/x86_64" --regcode <code> --email <email>

# make sure the debug symbol repositories are available
$ sudo zypper modifyrepo --enable SLES12-SP4-Debuginfo-Pool
$ sudo zypper modifyrepo --enable SLES12-SP4-Debuginfo-Updates

# search packages of kernel debug symbols
$ sudo zypper search -s kernel-*-debuginfo*

# install packages
$ sudo zypper install -y kernel-default-debuginfo=4.12.14-94.41.1
```

### [System V ABI: `AMD64(x86-64)`](https://github.com/hjl-tools/x86-psABI/wiki/X86-psABI)

#### x86-64 Registers

| 64-bit Reg |        Usage        |   Who Saves  |
|:----------:|:-------------------:|:------------:|
|    %rax    |     return value    | caller-saved |
|    %rbx    |                     | callee-saved |
|    %rcx    |     4th argument    | caller-saved |
|    %rdx    |     3rd argument    | caller-saved |
|    %rsp    |    stack pointer    |              |
|    %rbp    |    frame pointer    | callee-saved |
|    %rsi    |     2nd argument    | caller-saved |
|    %rdi    |     1st argument    | caller-saved |
|    %rip    | instruction pointer |              |
|    %r8     |     5th argument    | caller-saved |
|    %r9     |     6th argument    | caller-saved |
|    %r10    |      temporary      | caller-saved |
|    %r11    |      temporary      | caller-saved |
|    %r12    |                     | callee-saved |
|    %r13    |                     | callee-saved |
|    %r14    |                     | callee-saved |
|    %r15    |                     | callee-saved |
|    ...     |                     |              |

#### x86-64 Linux Calling Conventions

|              | User Interface | Kernel Interface |
|:------------:|:--------------:|:----------------:|
| Return value |      %rax      |       %rax       |
|    1st arg   |      %rdi      |       %rdi       |
|    2nd arg   |      %rsi      |       %rsi       |
|    3rd arg   |      %rdx      |       %rdx       |
|    4th arg   |      %rcx      |       %r10       |
|    5th arg   |       %r8      |        %r8       |
|    6th arg   |       %r9      |        %r9       |

```
high address  +-------------+
              |             |  <- begin of frame <--+
              |             |                       |
              |             |                       |
              |             |                       |
              |             |                       |
              |             |                       |
              |-------------|     stack frame       |
              |             |                       |
              |    args     |                       |
              | (7th ~ Nth) |                       |
              |             |                       |
              |-------------|                       |
  stack       | return addr |  <- end of frame      |
              |-------------|                       |
              |  old %rbp   |  <- %rbp (optional) --+
              |-------------|
              |             |
              | saved regs  |
              |    and      |
              | local vars  |
              |             |
              |-------------|
              |    args     |
              | (optional)  |  <- %rsp
low address   +-------------+
```

#### x86-64 Linux User Function Calling Convention

```bash
$ cat > hello.c << END
#include <stdio.h>

int foo(int a, int b, int c, int d, int e, int f)
{
    int r = a + b + c + d + e + f;
    r++;
    return r;
}

int main(int argc, char *argv[])
{
    const int r = foo(111, 222, 333, 444, 555, 666);
    printf("r=%d\n", r);
    return 0;
}
END

$ gcc -m64 -c hello.c
$ gdb -q hello.o -ex "disassemble foo" -ex "quit"
Reading symbols from /workspace/g/hello.o...(no debugging symbols found)...done.
Dump of assembler code for function foo:
   0x0000000000000000 <+0>:     push   %rbp                # save caller's frame pointer
   0x0000000000000001 <+1>:     mov    %rsp,%rbp           # set callee's frame pointer
   0x0000000000000004 <+4>:     mov    %rdi,-0x18(%rbp)    # save function 1st arg
   0x0000000000000008 <+8>:     mov    %rsi,-0x20(%rbp)    # save function 2nd arg
   0x000000000000000c <+12>:    mov    %rdx,-0x28(%rbp)    # save function 3th arg
   0x0000000000000010 <+16>:    mov    %rcx,-0x30(%rbp)    # save function 4th arg
   0x0000000000000014 <+20>:    mov    %r8,-0x38(%rbp)     # save function 5th arg
   0x0000000000000018 <+24>:    mov    %r9,-0x40(%rbp)     # save function 6th arg
   0x000000000000001c <+28>:    mov    -0x20(%rbp),%rax
   0x0000000000000020 <+32>:    mov    -0x18(%rbp),%rdx
   0x0000000000000024 <+36>:    add    %rax,%rdx
   0x0000000000000027 <+39>:    mov    -0x28(%rbp),%rax
   0x000000000000002b <+43>:    add    %rax,%rdx
   0x000000000000002e <+46>:    mov    -0x30(%rbp),%rax
   0x0000000000000032 <+50>:    add    %rax,%rdx
   0x0000000000000035 <+53>:    mov    -0x38(%rbp),%rax
   0x0000000000000039 <+57>:    add    %rax,%rdx
   0x000000000000003c <+60>:    mov    -0x40(%rbp),%rax
   0x0000000000000040 <+64>:    add    %rdx,%rax
   0x0000000000000043 <+67>:    mov    %rax,-0x8(%rbp)
   0x0000000000000047 <+71>:    addq   $0x1,-0x8(%rbp)
   0x000000000000004c <+76>:    mov    -0x8(%rbp),%rax     # set return value
   0x0000000000000050 <+80>:    pop    %rbp                # restore caller's frame pointer
   0x0000000000000051 <+81>:    retq
End of assembler dump.
```

#### x86-64 Linux Kernel Function Calling Convention

```bash
$ gdb -q /usr/lib/debug/lib/modules/3.10.0-514.26.2.el7.x86_64/vmlinux -ex "disassemble do_filp_open" -ex "quit"
Reading symbols from /usr/lib/debug/usr/lib/modules/3.10.0-514.26.2.el7.x86_64/vmlinux...done.
Dump of assembler code for function do_filp_open:
   0xffffffff81210670 <+0>:     callq  0xffffffff816994c0 <__fentry__>
   0xffffffff81210675 <+5>:     push   %rbp                   # save caller's frame pointer
   0xffffffff81210676 <+6>:     mov    %ecx,%r8d
   0xffffffff81210679 <+9>:     or     $0x40,%r8d
   0xffffffff8121067d <+13>:    mov    %rsp,%rbp              # set callee's frame pointer
   0xffffffff81210680 <+16>:    push   %r14                   # save caller's data
   0xffffffff81210682 <+18>:    mov    %ecx,%r14d
   0xffffffff81210685 <+21>:    mov    %rdx,%rcx
   0xffffffff81210688 <+24>:    push   %r13                   # save caller's data
   0xffffffff8121068a <+26>:    mov    %rdx,%r13
   0xffffffff8121068d <+29>:    push   %r12                   # save caller's data
   0xffffffff8121068f <+31>:    mov    %rsi,%r12
   0xffffffff81210692 <+34>:    push   %rbx                   # save caller's data
   0xffffffff81210693 <+35>:    mov    %edi,%ebx
   0xffffffff81210695 <+37>:    and    $0xfffffffffffffff0,%rsp
   0xffffffff81210699 <+41>:    sub    $0xa0,%rsp
   0xffffffff812106a0 <+48>:    mov    %rsp,%rdx
   0xffffffff812106a3 <+51>:    mov    %gs:0x28,%rax
   0xffffffff812106ac <+60>:    mov    %rax,0x98(%rsp)
   0xffffffff812106b4 <+68>:    xor    %eax,%eax
   0xffffffff812106b6 <+70>:    callq  0xffffffff8120e480 <path_openat>
   0xffffffff812106bb <+75>:    cmp    $0xfffffffffffffff6,%rax
   0xffffffff812106bf <+79>:    je     0xffffffff81210705 <do_filp_open+149>
   0xffffffff812106c1 <+81>:    cmp    $0xffffffffffffff8c,%rax
   0xffffffff812106c5 <+85>:    je     0xffffffff812106ec <do_filp_open+124>
   0xffffffff812106c7 <+87>:    mov    0x98(%rsp),%rdx
   0xffffffff812106cf <+95>:    xor    %gs:0x28,%rdx
   0xffffffff812106d8 <+104>:   jne    0xffffffff812106e7 <do_filp_open+119>
   0xffffffff812106da <+106>:   lea    -0x20(%rbp),%rsp
   0xffffffff812106de <+110>:   pop    %rbx                   # restore caller's data
   0xffffffff812106df <+111>:   pop    %r12                   # restore caller's data
   0xffffffff812106e1 <+113>:   pop    %r13                   # restore caller's data
   0xffffffff812106e3 <+115>:   pop    %r14                   # restore caller's data
   0xffffffff812106e5 <+117>:   pop    %rbp                   # restore caller's data
   0xffffffff812106e6 <+118>:   retq
   ...
```
