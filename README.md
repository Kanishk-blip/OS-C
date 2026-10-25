# COS — A RISC-V Operating System in C

COS is a small educational operating system written in C and RISC-V assembly. It runs on a 32-bit RISC-V virtual machine in QEMU and demonstrates core operating-system concepts such as virtual memory, process scheduling, system calls, memory management, filesystem access, and disk I/O.

The project was developed with reference to the [Operating System in 1,000 Lines](https://operating-system-in-1000-lines.vercel.app/) project and xv6.

## Features

- 32-bit RISC-V kernel bootstrapping
- Supervisor and user execution modes
- Sv32 two-level virtual memory
- Per-process page tables and isolated address spaces
- Cooperative multitasking for up to 8 processes
- RISC-V assembly context switching
- Exception and system-call handling
- Bitmap-based physical page allocator
- Process memory reclamation on exit
- VirtIO block-device driver
- USTAR-based filesystem
- Interactive user-space shell
- Custom C standard-library utilities
- QEMU-based development environment

## Architecture

```text
User-space shell
       |
       | ecall
       v
System-call dispatcher
       |
       +-- Console I/O
       +-- Process management
       +-- Memory diagnostics
       +-- Filesystem operations
                 |
                 v
          USTAR filesystem
                 |
                 v
         VirtIO block driver
                 |
                 v
          QEMU disk image
```

## System Calls

COS implements nine system calls:

| System call | Purpose |
|---|---|
| `SYS_PUTCHAR` | Write a character to the console |
| `SYS_GETCHAR` | Read a character from the console |
| `SYS_EXIT` | Terminate the current process |
| `SYS_READFILE` | Read data from a file |
| `SYS_WRITEFILE` | Write data to a file |
| `SYS_LISTFILE` | List files |
| `SYS_PS` | Display process information |
| `SYS_BITMAP` | Display physical-page usage |
| `SYS_INIT` | Create another shell process |

## Shell Commands

| Command | Description |
|---|---|
| `hello` | Print a test message |
| `ls` | List files in the filesystem |
| `readfile` | Read `hello.txt` |
| `writefile` | Update `hello.txt` |
| `ps` | Display process states |
| `bitmap` | Display allocated physical pages |
| `init` | Create another shell process |
| `exit` | Terminate the current process |

## Memory Management

Each process receives an independent Sv32 page table. User program pages are copied into separately allocated physical pages and mapped with user-mode permissions.

Physical memory is managed through a bitmap allocator containing 512 page entries. With a page size of 4 KiB, the active allocator manages 2 MiB of physical memory.

When a process exits, COS releases:

- User-accessible physical pages
- Second-level page tables
- The process's root page table
- Its process-control-block slot

This allows process IDs and memory pages to be reused.

## Process Management

COS supports up to eight process control blocks. Each process stores:

- Process ID
- Process state
- Saved stack pointer
- Root page-table address
- Kernel stack

The cooperative scheduler switches between runnable processes by saving and restoring RISC-V registers, changing the active page table, flushing the TLB, and selecting the appropriate kernel stack.

## Filesystem and Disk I/O

The kernel communicates with a QEMU VirtIO block device through memory-mapped registers and a 16-entry virtqueue.

At boot, the kernel reads a USTAR disk image into memory and parses its file headers. File modifications are serialized back into USTAR format and written through the VirtIO driver.

The current filesystem supports up to two files, with up to 1 KiB of data per file.

## Project Structure

```text
.
├── common.c       # Shared printf, memory, and string functions
├── common.h       # Shared types, constants, and system-call numbers
├── kernel.c       # Kernel, scheduler, memory, filesystem, and drivers
├── kernel.h       # Kernel data structures and hardware definitions
├── kernel.ld      # Kernel linker script
├── shell.c        # Interactive user-space shell
├── user.c         # User-space system-call wrappers and entry point
├── user.h         # User-space API
├── user.ld        # User application linker script
├── run.sh         # Build and QEMU execution script
├── tmp/           # Files included in the disk image
└── memo/          # Development notes
```

## Requirements

- QEMU with RISC-V 32-bit support
- Clang/LLVM with a RISC-V target
- `llvm-objcopy`
- `tar`
- A Unix-like shell

The paths to Clang and `llvm-objcopy` are currently configured for a Homebrew LLVM installation in `run.sh`. Update them if LLVM is installed elsewhere.

## Build and Run

Make the execution script executable:

```bash
chmod +x run.sh
```

Build the kernel and launch it in QEMU:

```bash
./run.sh
```

After the system boots, an interactive prompt will appear:

```text
> hello
Hello World from shell!

> ls
hello.txt
meow.txt

> ps
pid=-1 runnable
pid=2 runnable

> bitmap
0xffff... current page use 59/512 (pages)
```

Press `Ctrl+A`, followed by `X`, to exit QEMU when using its serial console.

## Current Limitations

COS is an educational operating system and currently does not include:

- Preemptive scheduling
- Hardware interrupt processing
- Timers
- Inter-process communication
- TCP/IP networking
- Multiprocessor support
- General-purpose executable loading
- `fork()` or full `exec()` support
- A production-grade filesystem

## References

- [Operating System in 1,000 Lines](https://operating-system-in-1000-lines.vercel.app/)
- [xv6: A Simple, Unix-like Teaching Operating System](https://github.com/mit-pdos/xv6-riscv)
- [MIT 6.828 / 6.S081](https://pdos.csail.mit.edu/6.828/2023/)
- [Sugawara Laboratory xv6 Materials](https://www.sugawara-lab.jp/lecture.html)

## License

This project is distributed under the MIT License. See `LICENSE` for details.
