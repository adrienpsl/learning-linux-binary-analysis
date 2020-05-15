# ELF Binary Format

#### ELF type
ET_NONE : unknow type or not yet defined
ET_REF  : relocatable file, it's an object, .o
ET_EXEC : executable file
ET_DYN  : Shared object, also know as shared libraries
ET_CORE : Elf core file, dump of a full process image during crash
          or SIGSEGV (segmentation violation)

the Elf start with the elf header.

## Program headers
* Describe segments.
  - are understood by the kernel during loading time, and describe the
    the memory layout of an executable on disk, and how it should
    translate into memory.
* Header start at e_phoff.
* There are five commun program header type, and by that type, what is
  the purpose of that segment.

```c
typedef struct {
    uint32_t   p_type;   (segment type)
    Elf32_Off  p_offset; (segment offset)
    Elf32_Addr p_vaddr;   (segment virtual address)
    Elf32_Addr p_paddr;    (segment physical address)
    uint32_t   p_filesz;   (size of segment in the file)
    uint32_t   p_memsz; (size of segment in memory)
    uint32_t   p_flags; (segment flags, I.E execute|read|read)
    uint32_t   p_align;  (segment alignment in memory)
  } Elf32_Phdr;
```

#### PT_LOAD
An executable will always have at least one,
This header describe loadable segment
    -> the segment will be mapped into memory.

An ELF executable, generally contain theses 2 segments:
* text segment for program code
* data segment for global and dynamic linking information

These segment will be mapped in memory and aligned by the value
stored in p_align.

#### PT_DYNAMIC
It's specific to executables that are dynamically linked and contains
information necessary for it.
This segment contain tagged values and pointers like:
* List of shared libraries that are to bie linked at runtime.
* Address/location of the Global Offset Table (GOT).
* Information about relocation entry.

and some tag name :
    DT_HASH
    DT_STRTAB
    DT_SYMTAB
    DT_RELA...

The dynamic segment segment contains a series of structure that old,
relevant dynamic linking information.
__The d_tag member controls the interpretation of d_un.__

#### PT_NOTE
A segment of type PT_NOTE is used to specify that this exe need
match some conformance, compatibility, and so on.
This serve, os specification information.

#### PT_INTERP
Small segment that describe where the program interpreter is.

#### PT_PHDR
This segment contains the location of the program header table itself.
The command readelf -l show the phdr table.
the entry point,


## Section headers
* Section are not necessary for program execution,
* Section header is only keep in exe file, in linking and debugging.
* Section header only specify where is the data, not the memory layout.
* If the memory section are not, or not accessible, it's mean that the
  binary is striped.
* The sections header can be remove on purpose, making debugging harder,
  objcopy, objdump, and gdb rely on the section headers to locate symbol
  so if the file is stripped, gdb and objdump are useless.
* Section header are convenient for granular inspection,
  they make reverse engineering a lot easier.
* It's possible for a middle engineer to revers that.

```c
typedef struct {
uint32_t   sh_name;      // offset into shdr string table for shdr name
uint32_t   sh_type;      // shdr type I.E SHT_PROGBITS
uint32_t   sh_flags;     // shdr flags I.E SHT_WRITE|SHT_ALLOC
Elf32_Addr sh_addr;      // address of where section begins
Elf32_Off  sh_offset;    // offset of shdr from beginning of file
uint32_t   sh_size;      // size that section takes up on disk
uint32_t   sh_link;      // points to another section
uint32_t   sh_info;      // interpretation depends on section type
uint32_t   sh_addralign; // alignment for address of section
uint32_t   sh_entsize;   // size of each certain entries that may be in section
} Elf32_Shdr;

```

### the main section:

#### .text -> text segment
Contain the program code instruction,
SHT_PROGBITS -> text segment


#### .rodata section -> text segment
The rodata section contains read only, like `print("Hello World\n")
SHT_PROGBITS -> text segment


#### .plt section -> text segment
Procedure linkage table (PLT), contains code necessary
For the dynamic linker, it has code so it'ts
SHT_PROGBITS

#### .data section -> data segment
contains program variable data, program variable data
SHT_PROGBITS

#### .bss section -> data segment
contain uninitialized global data,
data are set to 0 when the program start

#### .got.plt section -> execution ?
* Global Offset Table contain the global offset table.
* Works with the PLT to provide access to imported shared library.
* It's modified by the dynamic linker at runtime.
* This section is often abused by attackers who gain a pointer-sized
  primitive in heap or .bss exploits.
SHT_PROGBITS

#### .dynsym section -> text segment
Contain dynamic symbol information form shared libraries.
SHT_DYNSYM

#### .dynstr -> often at the end
The string table for dynamic symbol

#### .rel.* ->
Relocation contain information about how parts off the ELF object,
or process image need to be fixed up, or modified at linking / runtime.
SHT_REL


#### .hash // .gnu.hash
Contains a hash table for symbol lookup.
next, the hash algo:
```c
uint32_t
dl_new_hash (const char *s)
{
        uint32_t h = 5381;

        for (unsigned char c = *s; c != '\0'; c = *++s)
                h = h * 33 + c;

        return h;
}

// or

h = ((h << 5) + h) + c
```

#### .symtab
Contain symbol information of type ElfN_Sym.
SHT_SYMTAB

#### .strtab
* symbol string table
* ref by st_name in ElfN_Sym
* SHT_STRTAB

#### .shstrtab
* section header string table
* that ptr on the section name, like .text, .bss ...
* ptr by ELF file header as e_shstrndx
* SHT_STRTAB

#### .ctors (constructor) / .dtors (destructor)
* Pointer function to initialization / finalisation code.
* Code that is to be executed before / after main function.
* Hacker use this to implement function before code start,
  to perform an antidebugging trick. Like calling PTRACE_TRACEME.
  In this way, the anti-debugger code is executed before the code himself.

### The text segment will be:
    [.text]: This is the program code
    [.rodata]: This is read-only data
    [.hash]: This is the symbol hash table
    [.dynsym ]: This is the shared object symbol data
    [.dynstr ]: This is the shared object symbol name
    [.plt]: This is the procedure linkage table
    [.rel.got]: This is the G.O.T relocation data

### The data segment:
    [.data]: These are the globally initialized variables
    [.dynamic]: These are the dynamic linking structures and objects
    [.got.plt]: This is the global offset table
    [.bss]: These are the globally uninitialized variables


## ELF symbols
2 sections, ce sont des tables de la structure Elf64_Sym

#### .dynsym:
* contient les symbols exterieur (printf) ...
* ne peux etre supprimer
* Est mapp/e en memoire (readelf -S) -> ALLOC right

#### .symtab:
* contient tout dynsym
* contient tout les symbols du fichier, global, function
* n'est la que pour debug et linking

```c
typedef struct {
uint32_t      st_name;
    unsigned char st_info;
    unsigned char st_other;
    uint16_t      st_shndx;
    Elf64_Addr    st_value;
    Uint64_t      st_size;
} Elf64_Sym;
```


#### st_name
contain offset into the symbol table's string, in .dynstr / strtab,
where the symbol is located, such as printf.

#### st_value
The st_value holds the value of the symbol address, or offset of its location

#### st_size
Size of the symbol, for a global ptr function, it's 4 bytes on 32 bytes.

#### st_other
Define the symbol visibility.

#### st_shndx
Every symabol is defined in relation to some section.
This contain the relevant index of this section.

#### st_info
Specifies the symbol type and binding attribute.
* Symbol types:
  - STT_NOTYPE: symbole is undefined
  - STT_FUNC:   associated with function, or executable code
  - STT_OBJECT: associated with data object
* Symbol binding:
  - STB_LOCAL:    Local symbols are not visible outside its object, like static func an .o
  - STB_GLOBAL    Visible to all object files being combined.
  - STB_WEAK      Same visibility that STB_GLOBAL, but can be overwrite
                  by an other symbol, with same name, but not weak.

##### There is macro to unpacking binding and type
    ELF32_ST_BIND(info) or ELF64_ST_BIND(info) extract a binding from an st_info value
    ELF32_ST_TYPE(info) or ELF64_ST_TYPE(info) extract a type from an st_info value
    ELF32_ST_INFO(bind, type) or ELF64_ST_INFO(bind, type) convert a binding and a type into an st_info value

code source
```c
static inline void foochu()
{ /* Do nothing */ }

void func1()
{ /* Do nothing */ }

_start()
{
        func1();
        foochu();
}
```

* comment que c'est dans la strsym / dymstr

ryan@alchemy:~$ readelf -s test | egrep 'foochu|func1'

index addres   ? type binding
7:    080480d8 5 FUNC LOCAL   DEFAULT 2 foochu
8:    080480dd 5 FUNC GLOBAL  DEFAULT 2 func1


Symbol make life easier for everyone, they are part of the elf
for linking, relocation, readable disassembly, debugging.

si symtab est delete, avec (exemple) :
* compiled with gcc-static
* compiled with gcc-nostdlib
* the binary is striped, with the strip command

Je n'aurai plus aucune information sur :
quoi est une fonction,
quand y a t'il des function call... ca va me rendre la vie bcq plus dur.

je passe de
```shell script
objdump -d test2

Disassembly of section .text:
0000000000400144<foo>:
  400144:   55                      push   %rbp
  400145:   48 89 e5                mov    %rsp,%rbp
  400148:   5d                      pop    %rbp
  400149:   c3                      retq   

000000000040014a <_start>:
  40014a:   55                      push   %rbp
  40014b:   48 89 e5                mov    %rsp,%rbp
  40014e:   e8 f1 ff ff ff          callq  400144 <foo>
  400153:   c9                      leaveq
  400154:   5d                      pop    %rbp
  400155:   c3                 retq
```

a

```shell script
0000000000400144 <.text>:
  400144:   55                      push   %rbp  
  400145:   48 89 e5                mov    %rsp,%rbp
  400148:   5d                      pop    %rbp
  400149:   c3                      retq   
  40014a:   55                      push   %rbp 
  40014b:   48 89 e5                mov    %rsp,%rbp
  40014e:   e8 f1 ff ff ff          callq  0x400144
  400153:   c9                      leaveq
  400154:   5d                      pop    %rbp
  400155:   c3                      retq  
```

car objdump n'est plus capable de retrouver les informaition relative
au function, la seul chose qui me permet de voir que c'est une functoin
c'est le ```push %rbp | mov %rsp, %rbp```

#### Ftrace
Ftrace was create in 2013 by the Author,
It's trace all of the function calls made within the binary
and can also show branch instruction like jump.
si j'ai besoin de decompiler un truc, regader dans Ftrace.


## ELF relocation
Relocation est le process de connecter des reference symbolic avec des
definition symbolic. Relocatable file, doivent avoir des informations
qui explique comment acceder et modifier les informations dans leurs sections.
Les informations qui permettend de faire ca, sont les Relocation entries.

Relocation depend des sections et symbol,
il y a des relocation Records, qui contiennent des informations
sur comment patcher un symbol donne.

Relocation est plus simplement un system de patch, et meme de hot-patching
quand le dynamic linker est actif.

le program linker est : /bin/ld, qui build shared lib et executable,
a besoin de methadonne pour le faire: les __relocation records__



on prends deux .o que l'on va link pour faire un executable,
obj_1.o -> want call foo()
obj_2.o -> has the foo code

Les deux obj vont etre annalyser par le linker, et grace a leurs
relocation records, ils vont etre linker pour faire un exec.

Symbolic ref, vont etre resolu en symbolic definition:
les codes des object files est contruit pour permettre d'etre
relocaliser a une address precise dans le segment de mon exec.

Pour construire ces segments, il faut connaitre ou est le code (sym),
et ou il est (section) : sym -> section -> code >> segment.
Avec toutes ces informations, le code peut etre patch dans le bon segment.

64bit relocation enty:
```c
typedef struct {
        Elf64_Addr r_offset;
        Uint64_t   r_info;
} Elf64_Rel;

// with addend

typedef struct {
        Elf64_Addr r_offset;
        uint64_t   r_info;
        int64_t    r_addend;
} Elf64_Rela;
```

r_offset
    ptr la ou il faut faire une relocation
    Relocation action, decrit comment patch le code qui est a r_offset

r_info
    donne le type de relocation a faire
    l'index dans la table des symbols

r_addend
    constant a ajouter a la value stored dans le relocatable field


exemple:
```c
// gcc -c -m32 -nostdlib
_start()
{
    foo();
}
```
-> -> ->

```shell script
$ objdump -d obj1.o
obj1.o:     file format elf32-i386
Disassembly of section .text:
00000000 <func>:
   0:   55                      push   %ebp
   1:   89 e5                   mov    %esp,%ebp
   3:   83 ec 08                sub    $0x8,%esp
   6:   e8 fc ff ff ff          call 7 <func+0x7>
   b:   c9                      leave  
   c:   c3                      ret   
```

in 6, there is the call to foo() + the value 0xff ff ff fc,
qui represent implicit addend.
call 7, est l'offset de la relocation a patch.

Quand obj_1.o est link a obj_2.o pour faire un exec,
le relocation entry point 7 est process par le linker:
lui demandant de mettre a jour la location 7 (offset 7)
Le linker patch les 4 bytes a l'offset 7, qui contiendront
l'offset de la function foo, apres l'avoir placer dans le binaire.

e8 fc ff ff ff = -4 = -(sizeof(uint32_t)). dword is 4 bytes on 32bit


```shell script
$ readelf -r obj1.o

Relocation section '.rel.text' at offset 0x394 contains 1 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
00000007  00000902 R_386_PC32        00000000   foo
```

inside the section .rel.text, there is one relocation records:
* the field is at offset 7.
* R_386_PC32, is the relocation type, read the man

#### Relocation type
Ca depend vraiment du type, if faut regarder sur le man,
pour celui ci c'est : S + A - P.
* S: la value du symbol a l'index de la relocation entry
* A: l'addend trouve das la relocation entry
* P: est la (section offset or address) ? pas compris, surlinger dans le livre
     a -nop__1



Here the code after linking

```shell script
080480d8 <func>:
 80480d8:   55                      push   %ebp
 80480d9:   89 e5                   mov    %esp,%ebp
 80480db:   83 ec 08                sub    $0x8,%esp
 80480de:   e8 05 00 00 00          call   80480e8 <foo>
 80480e3:   c9                      leave  
 80480e4:   c3                      ret    
 80480e5:   90                      nop
 80480e6:   90                      nop
 80480e7:   90                      nop
 
 080480e8 <foo>:
 80480e8:   55                      push   %ebp
 80480e9:   89 e5                   mov    %esp,%ebp
 80480eb:   5d                      pop    %ebp
 80480ec:   c3                      ret
 ```

J'ai pas mal de mal avec ca, mais je pense qu'il faut juste appliquer
la formule et ca marche tout, seul.


## Relocatable code injection-based binary patching
Les virus complique utilise ces techniques pour creer un tout
nouveau binaire, jolie et puissant.

jouons un jeu, nous somme un attaquant, et on veux printer hello_world
a chaque fais que la function puts est activ/e, en mettant a la place
notre function, evil_puts.

```c
#include <sys/syscall.h>
int _write (int fd, void *buf, int count)
{
  long ret;

  __asm__ __volatile__ ("pushl %%ebx\n\t"
"movl %%esi,%%ebx\n\t"
"int $0x80\n\t""popl %%ebx":"=a" (ret)
                        :"0" (SYS_write), "S" ((long) fd),
"c" ((long) buf), "d" ((long) count));
  if (ret >= 0) {
    return (int) ret;
  }
  return -1;
}
int evil_puts(void)
{
        _write(1, "HAHA puts() has been hijacked!\n", 31);
}
```

## ELF dynamic linking



















####

trouver la section tout a la fin physiquement,
check les droits
je vais dedant et je met a jour pour pointer dans mon algoryhtm
precedent entrypoint-> after.

algo qui va compress tout ca : -> si plus de place -> de la meme taille
que la taille originel.

le code asm -> devient op code ->
dans les bits -> une parti l'opcode?

faire super attention a la taille, car c'est super optimiser.

-> version de gcc -> ubuntu a changer le padding par deffault des binaires
-> 2mg a 2kb

-> ca depend -> si tout va bien, et que je veux savoir pourquoi?
-> apprendre -> raimfaall, et les faire dans l'ordre -> le pb c'est le debug.

utiliser la callstack ->
    pour debug -> la sous_funcion affiche l'erreur
    -> return un boolean qui dit que ca ne marche pas
    -> la function d'apres dis qu'elle aussi a une error
    -> afficher toute la call stack pour dire que je suis passe, et remonter mon error

toujours mettre dans l'o
woody mon wooody.
test avec plein de binaire
pwd, whoiam, php, python.

test avec certain binaire, certain seg si pas d'argument.


papier de recherche -> sur le language informatique.
                    -> comment ca se passe, mini tuto ...
                    -> papier de rechercher, livre.

mots clee, avec les bons polymorphisme...

ring 0 droit du system

antivirus peut changer tout les fichiers,
le virus le detect -> il y a quelque cycles.
pendant ce temps -> le virus sais qu'il s'est fait detecter
virus fait un lien symbolique -> la ou meme

regarder le rust // the book rust -> firefox.
                    -> en rust cargo -> autocompile




















