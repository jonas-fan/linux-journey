# Process

## Binary format

Processes are running executables. The binary format of executable is `Executable and Linking Format (ELF)`, which is the replacement of legacy formats, such as `Assembler output (a.out)` and `Common Object File Format (COFF)`.

```bash
$ file $(which file)
/usr/bin/file: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked,
    interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0,
    BuildID[sha1]=895077a840385f977d10a2402d58f8b4edc8bd48, stripped
```

## Memory layout

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
0xffffffffffffffff +---------------+
                   |               |
     128 TiB       |    kernel     |  shared between
   (2^47 bytes)    |               |  all processes
                   |               |
0xffff7fffffffffff |---------------|
                   |               |
    ~16M TiB       | non-canonical |
                   |               |
0x00007fffffffffff |---------------|
                   |   argv, env   |
                   |---------------|
                   |               |
                   |    stack      |
                   |               |
                   |---------------|  <- top of stack
                   |               |
                   |               |
     128 TiB       |               |
   (2^47 bytes)    |---------------|  <- program break
                   |               |
                   |     heap      |
                   |               |
                   |---------------|  <- end
                   |     bss       |
                   |---------------|  <- edata
                   |     data      |
                   |---------------|  <- etext
                   |     text      |
0x0000000000400000 |---------------|
                   |               |
0x0000000000000000 |---------------|
```
