# Useful devices and files


## Linux Tools

### Gdb
yo known it

### Objdump form GNU binutils
* good at:
  - quick disassembly
  - untampered binaries
* limit:
  - Does not check control flow // todo ?
  - Relies only at the section header, can't open, or analyse if there
    is no section.
* good for basic binaries that are not protected.

```shell script
# See data code in every section:
objdump -D <elf_object>

# View only program code:
objdump -d <elf_object>

# View all symbols: 
objdump -tT <elf_object>
```


### Objcopy
* Analyze and modified ELF object, of any kind
* Often used to modify and copy an ELF section to or from ELF binary.


### Strace
* Based on ptrace(2)
* Use PTRACE_SYSCALL in loop to show information about syscall
* Catch activity, running program and signals
* Useful to for debugging and catch runtime syscall
```shell script
# Trace basic program:
strace /bin/ls -o ls.out

# Attach to an existing process
strace -p <pid> -o daemon.out
```
* Show the descriptor if the syscall takes one
  `SYS_read(3, buf, sizeof(buff));`
* Show all the data that as been read by FD 3
  `strace -e read=3 /bin/ls`

### Ltrace
Work similarly that strace, but for libraries
```shell script
ltrace <program> -o program.out
```


### Ftrace
show call to function inside the binary,
write by the author (cool).

### Readelf
* readelf is one off the best tool to dissecting ELF.
```shell script
# Section header table:
readelf -s <object>

# Program header table:
readelf -l <object>

# Symbol table:
readelf -s <object>

# File header:
readelf -e <object>

# Relocation entries:
readelf -r <object>

# Dynamic segment:
readelf -d <object>
```


## Sys File

### /proc/<pid>/maps
Ce fichier contient le layout pages en memoir, mmap.

address           perms offset  dev   inode   pathname
08048000-08056000 r-xp 00000000 03:0c 64593   /usr/sbin/gpm

### /proc/Kcore
C'est un dynamic core file du kernel linux, au format ELF
et peux etre utiliser pour analiser le noyaux.

### /boot/system.map
Contien tout les symbol pour le kernel

### /proc/kallsyms
Tres similaire au /boot/system.map,
* gerer dynamiquement par le kernel
* il add les lkms ((loadable) linux kernel module) on the fly.

### /proc/iomen
Similaire a /proc/<pid>/maps, mais pour toute la memoir du system,
je peux y voir le code du kernel, en recherchant kernel dedans.

### ECFS
Extended Core File Snapshot, est un outil pour faire des recherches
pousees dans les images.


## Linker-related environment points
Dynamic loader and linking are concept that are inescapable.
Throughout this book we will begin the process of linking, relocations
and dynamic loading.

### The LD_PRELOAD environment variable
The LD_PRELOAD can be set to specify which library should be
dynamically linked before any other libraries. This allow that first
library to erase symbol and overwrite function.
* Allow runtime patching by redirecting shared library function.
* By-pass anti-debugging code
* By-pass userland rootkits

### The LD_SHOW_AUXV  environment variable
* It tells the program loader to display the program's auxiliary vector
  during runtime.
* The auxiliary vector is information placed on the program stack
  by kernel's ELF loading routine.
* Vector is used by the dynamic linker.

### Linker scripts
* Interpret by the linker and give help to understand the program,
  with sections, memory, and symbols
* Linker has is own language.
* It determines how the output, like ELF executable, will be
  organized.
* Organize, which section will be in which segment.
* Gcc does not link, it's an external program














