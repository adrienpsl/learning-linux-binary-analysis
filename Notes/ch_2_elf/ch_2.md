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




















