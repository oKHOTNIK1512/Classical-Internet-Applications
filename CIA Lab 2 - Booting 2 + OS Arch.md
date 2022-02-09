---
tags: Classical Internet Applications
---
:::success
# CIA Lab 2 - Booting 2 + OS Arch
**Name: Ivan Okhotnikov**
:::


## Task 1: Booting the OS
:::info
**1. What is a UEFI OS loader and where does the Ubuntu OS loader reside on the system?**

When you turn on the computer, the first thing that starts is the UEFI download. 
In turn, UEFI checks the boot configuration from NVRAM (variables that are necessary for startup and paths to boot loaders are stored there).
Based on these variables, the boot loader or the operating system kernel is launched.

The Ubuntu OS boot loader is located in the ```/boot/efi/EFI/BOOT folder```

<center>

![](https://i.imgur.com/xUgOqK9.png)
Figure 1: Ubuntu bootloader location folder
</center>
:::

:::info
**2. Describe in order all the steps required for booting the computer (until the OS loader starts running. ) Ubuntu uses the UEFI OS loader to load and run the GRUB boot loader.**

*Necessary steps to start:*
- Security Phase (SEC)
- EFI Pre-Initialization (PEI)
- Driver Execution Environment ( DXE)
- Boot Device Selection (BDS)
- Transient System Load (TSL)
- Runtime (RT)

---

*The `SEC` performs several tasks:*
- Creates a temporary memory storage
- Is the root of trust
- Handles system restart events

`PEI` is the first step of working with the system in the embryo state. Uses the processor cache to use as a system call stack for EFI preloading
- Complements the permanent memory
- Describes volumes in Handover blocks (HOB)

The `DXE` phase consists of connecting a set of drivers and their manager

The `BDS` phase loads the device drivers and tries to initialize the boot selection

The `TSL` phase performs the startup of the bootloader from the operating system

The `RT` phase starts the kernel driver and turns off some UEFI boot services
:::

:::info 
**3. What is the purpose of the GRUB boot loader in a UEFI system?**
The purpose of GRUB is to load and transfer control to the operating system kernel. The kernel, in turn, initializes the rest of the OS
:::

:::info
**4. How does the Ubuntu OS loader load the GRUB boot loader?**
    When the GRUB boot menu is displayed, you will be able to see the operating system that is responsible for booting (if there is more than one system). The responsible operating system will be the first in the list
:::

:::info
**5. Explain how the GRUB boot loader, in turn, loads and run the kernel by answering these 3 questions:**

---
**a. What type of filesystem is the kernel on?**
The kernel configuration and information about it is located in the /proc directory, the type of which is proc

---
**b. What type(s) of filesystem does UEFI support?**
The FAT12, FAT16 and FAT32 file systems are supported in UEFI

---
**c. What does the GRUB boot loader, therefore, have to do to load the kernel?**
You need to find out the location of the kernel using its parameters
:::

:::info 
**6. Do you need an OS loader and/or boot loader to load a Linux kernel with UEFI? Explain why or why not.**

UEFI is already able to load the Linux kernel out of the box using the EFISTUB solution. In any case, UEFI will be able to load the EFI executable file, even if there are no OS loaders
:::

:::info
**7. In an MBR-based system, the GRUB bootloader is distributed over the disk in multiple parts.**

---
**a. How many parts (or stages) does GRUB have in such a system, and what is their task?**
Since the MBR contains only 440 available bytes for initialization code, and GRUB implements many complex tasks, this amount of memory is not enough for it. To solve this problem, the work of GRUB based on MBR is divided into 3 stages
**1** -starting the primary loader from the MBR, which in turn proceeds to stage 1.5 (if any) or to stage 2)
**1.5** -implements support for reading files from the file system (stage 2 is stored in it)
**2** - loading the main GRUB code, which supports all file systems, starts displaying the boot menu.

---
**b. Where are the different stages found on the disk?**
**1** - First 440 byte of MBR
**2** - It can be located in the sectors between the MBR and the first partitions, or in the first sectors of the partition, if there is some free space at the beginning
**3** - In a file on disk
:::

## 2. Initializing the OS

:::info
1. Describe the entire startup process of Ubuntu in the default installation. The subquestions below are leaders to help you along, they must be answered but by no means represent the entire startup process of Ubuntu. . .
a. What is the first process started by the kernel?
b. Where is the configuration kept for the started process?
c. It starts multiple processes. How is the order of execution defined?
d. ...
:::

## Answer:

The Ubuntu startup process starts with the start of the Linux kernel. The kernel, in turn, starts the systemd process (previously it was the SysV init process), which in parallel mode starts the child processes that are necessary for operation.
Its configuration file is stored in `/etc/systemd/system.conf`
Although the execution of child processes is parallel, the startup order for some target processes is very important and can affect the successful startup of the system.
`Sysinit.target` will be able to start only after completing the tasks of mounting file systems (the configuration is described in `/etc/fstab/`), connecting swap files, configuring the initial value of the random number generator, initializing low-level services and configuring cryptographic systems (if at least one file system is encrypted)
When the sysinit.d target task is completed, other processes can be started in parallel.
`basic.target` is also a target process and will be executed only when all the tasks necessary to run it are completed. No task will be executed in parallel with it.

The final target process, if there is a graphical interface, will be `graphical.target`
If the graphical interface does not exist, the process will be stopped on the previous target process `multi-user.target`, which configures a non-graphical multi-user system

To see the scheme of launching processes (Figure 3), which will be launched after starting linux, you can see the output of the `man bootup`command

<center>

![](https://i.imgur.com/HObz8uY.png)
Figure 2: Process startup scheme
</center>


## 3. Basic OS interaction

:::info
**Binaries and scripts**
*Make a table listing five different binaries or interpreter scripts, and their type*

To display information about 5 different files, you need to run the command: 
```file /usr/bin/* | tail -5 | awk '{print $1" "$2}'```

The first column is the name of the executable file, the second column is its type

<center>

![](https://i.imgur.com/2Iqggy5.png)
Figure 3: The resulting table
</center>
:::

:::info
**Tracing binaries**
**Use “strace” to find what other systems call besides “stat” “zsh” uses before executing an “execve” system call to start a given binary (e.g. “/bin/ls”)?**


```strace zsh -c /bin/ls```
| system call | description |
| --- | --- |
| execve | to run the program |
| arch_prctl | to setting the state for a thread or process, depending on the architecture |
| brk | to change the size of a data segment |
| geteuid | to get the user ID (effective) |
| getgid | to get the group ID (real) |
| getegid | to get the group ID (effective) |
| readlink | to read a value from a symbolic link |
| pipe | to create a pipe |
| dup | to duplicate a file descriptor |
| getppid | to get the process id (parent) |
| getpid | to get the process id |
| getuid | to get the user ID (real) |
| socket | to create an endpoint (for communication) |
| connect | to initialize a connection to a socket |
| access | to check the user's access to the file |
| mprotect | to install protection on the memory area |
| mmap | to make map and unmap for files or devices in memory |
| lseek | to change the offset of a file for reading or writing |
| munmap | to cancel the display of pages in memory |
| uname | to get information about the current kernel and its name |
| ioctl | for working with input or output devices |
| prlimit64 | to get and set the value of resource limits |
| read | to read information from a file descriptor |
| openat | to get a file descriptor |
| fcntl | to manipulate a file descriptor |
| rt_sigaction | to study and change the action signal |
| getrusage | to get the resources used |
| close | to close a file descriptor |
| rt_sigprocmask | to receive and change blocked signals |

:::

:::info
**Stracing strace**
*1. Run “strace /bin/pwd” and save the result
2. Also run “strace strace /bin/pwd” and save the result. How are these outputs related? Explain the “2” in “write(2, ...”.*

They differ in that the first one tracks system calls for /bin/pwd, and the second one tracks /bin/pwd tracking
In `write(2,...)`, the first parameter `2` is the identifier of the file descriptor that is responsible for the standard error stream (stderr)
:::

:::info
**ELF format**
**1. Execute `readelf -Wh /bin/ls`. Match the results for the ELF header with the information on ELF’s Wikipedia page. Is this a definitive source for the ELF format? If not, what is?**

In the readelf command, the `W` flag indicates an output width of more than 80 characters, and `h` indicates the display of `ELF headers`

The information is completely consistent with the wikipedia page
**ELF Header:**
| Name | Value |
| --- | --- |
| Magic | 7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 |
| Class | ELF64 |
| Data | 2's complement, little endian |
| Version | 1 (current) |
| OS/ABI | UNIX - System V |
| ABI Version | 0 |
| Type | DYN (Shared object file) |
| Machine | Advanced Micro Devices X86-64 |
| Version | 0x1 |
| Entry point address | 0x5850 |
| Start of program headers | 64 (bytes into file) |
| Start of section headers | 132000 (bytes into file) |
| Flags | 0x0 |
| Size of this header | 64 (bytes) |
| Size of program headers | 56 (bytes) |
| Number of program headers | 9 |
| Size of section headers | 64 (bytes) |
| Number of section headers | 28 |
| Section header string table index | 27 |

<center>

![](https://i.imgur.com/UxUJMyk.png)
Figure 4: Output of the `readelf -Wh /bin/ls` command
</center>
:::


:::info
**3. Also inspect the section and program headers, using the command `readelf -WlS /bin/ls`.**

```
W - Allow output of more than 80 characters in width
l - Show program headers
s - Show character tables
```

<center>

![](https://i.imgur.com/0gENpwH.png)
Figure 5: Program output
</center>

**a. Explain which of the two header types (section or program) is used in which context (loader/runtime or linker/relocation)?**

*Relocation and linker are used by section header
Loader and runtime is used by program header*

---


**b. Explain the use of each of the following sections: .text, .data, .rodata and .bss. (Hint: Consider Type and Flg)**

| ELF section | Description | Type | Attributes |
| --- | --- | --- | --- |
| .text | Executable instructions or text | SHT_PROGBITS | SHF_ALLOC+SHF_EXECINSTR |
| .data | Initialized data| SHT_PROGBITS | SHF_ALLOC+SHF_WRITE |
| .rodata | Read-only data. | SHT_PROGBITS | SHF_ALLOC |
| .bss | Uninitialized data. The section does not take up space in the file | SHT_NOBITS | SHF_ALLOC+SHF_WRITE |

**Flags:**

| Name	|Description | Value |
| --- | --- | --- |
| SHF_WRITE	| Data that should be writable during the execution of the process | 0x1 |
| SHF_ALLOC	| Takes up memory during the execution of the process | 0x2 |
| SHF_EXECINSTR	| Executed machine instructions | 0x4 |

---

**c. What does the program header with Type INTERP contain?**

*The header with the INTERP type contains the path to the program interpreter*
:::

:::info
**4. Read your favorite ELF binary with hexdump utility.**

```
hexdump -C /bin/pwd
```

The ```-C``` flag is used to display offsets in hexadecimal format



**a. show the virtual address and physical address.**

`40 00 00 00 00 00 00 00` - p_vaddr
`40 00 00 00 00 00 00 00` - p_paddr

<center>

![](https://i.imgur.com/U0vRiMT.png)
Figure 6: Physical and virtual address for `/bin/pwd`
</center>

**b. show/explain the entry point of the ELF binary.**

The address of the point /bin/pwd corresponds to `00 1d 00 00 00 00 00 00`, where the process will start running.
The length depends on the system architecture. In our case, 64 bits (0x40)

<center>

![](https://i.imgur.com/QdNI88O.png)
Figure 7: Entry point of the ELF binary
</center>
:::

:::info
**5. Find the loaded libraries of your shell by executing ``lsof -p $$``.
Also, look at the memory image of your shell by executing `cat /proc/$$/maps`.**

`-p` - flag for the lsoft program, with the definition of the process ID
`$$` - process ID of the current shell instance

<center>

![](https://i.imgur.com/tWJCdGT.png)
Figure 8: Output of the `lsof -p $$`

![](https://i.imgur.com/I9ucCFA.png)
Figure 9: Output ot the `cat /proc/$$/maps`
</center>

**Why is for instance the libc library mapped into memory multiple times?**
The same library can be displayed multiple times, because their use is not limited to a single process.
:::

## 4. The gcc compiler

:::success
Make your own version of the “Hello SNE!”-program, by just only varying the string. Can you come up with a creative text? Find all four phases in the compilation by running “gcc -v -o hello hello.c”.

Now we are going to do all the stages separately in order to inspect the different intermediate steps:
:::

:::info
**1. Run “gcc -E hello.c -o hello.i”, producing the file “hello.i”.
What are the numbers at the end of the lines in “hello.i” supposed to mean?**

After entering the command, a preprocessor file `hello.i` was created, the numbers in its output are linear markers.
Their types:
1 - The beginning of a new file
2 - Return to the file
3 - The text was taken from the system header file
4 - The text should be considered as an implicit extern `C` block

<center>

![](https://i.imgur.com/z4SfqEK.png)
Figure 10: Part of the output from the `hello.c` file
</center>

:::

:::info
**2. Run “gcc -S hello.i”, producing the file “hello.s”
Inspect “hello.s”. Where has “printf” gone?
Remove the newline at the end of the string in “hello.c” and look again at the resulting “hello.s”. Explain.**

The printf function has been replaced by the system call The `printf ()` function was replaced by the system call `call puts@PLT`, in which `PLT` is a procedure communication table (it is necessary to localize fixes on machines at boot time that have access to relative addressing)

The difference between a file containing a line break and one that does not have a line break is that the system call `call puts@PLT` is removed and added instead
```
movl $0, %eax
call printf@PLT
```
<center>

![](https://i.imgur.com/DhbeIJF.png)
Figure 11: `hello.s` with `\n`

![](https://i.imgur.com/Bhu1tF5.png)
Figure 12: `hello.s` without `\n`
</center>

:::

:::info
**3. Run “gcc -c hello.s”, producing the file “hello.o”.
Use “objdump -rd hello.o” to inspect this relocatable object file.
Explain why this piece of machine code is (not yet) executable.**

A file with the `.o` extension is an object file. It is not executable due to the lack of a linker that adds a system-friendly ELF header indicating the executable nature of the file
It also does not contain links to other libraries and modules

<center>

![](https://i.imgur.com/gBG2QsA.png)
Figure 13: Output of the `objdump` program 
</center>

:::

:::info
**4. Run “gcc -o hello hello.o”, producing the file “hello”.
Use “ltrace ./hello” to see what library calls your program makes.
Run “ltrace -S” to also see the system calls.
What is the actual exit code of your program? Can you explain this?
Remark: You can generate all the intermediate files at once by using the “-save-temps” option of “gcc”**

The program did not make any library calls. The actual status code as a result of a successful compilation is 229 because it is 2021 mod 256 = 229

<center>

![](https://i.imgur.com/Ojvl4xy.png)
Figure 14: Output of the `ltrace` program
</center>

:::

### Stripped and non-stripped binaries (Bonus)

:::info
**1. Breifly explain what is stripped and non-stripped baniries and write at least two pros and cons.**

A stripped binary file differs from an non-stripped file in that an non-stripped file contains debugging information.

**Positive:**
- Debugging information is useful if you need to search for errors and problems in the program
- Helps with reverse engineering

**Minuses:**
- The file size increases
- Performance decreases
:::

:::info
**2. Compile the below code in two versions e.g. stripped and non-stripped and prove your above explanation.**

To create a stripped file, you need to install the `libgss-dev` module
You can create a non-stripped file using the `-g` flag for the gcc command
For example: `gcc -g stripped_test.c -o non-stripped`


<center>

![](https://i.imgur.com/PBtm9UG.png)
Figure 15: Content of stripped binary

![](https://i.imgur.com/LRSazO3.png)
Figure 16: Content of non-stripped binary
</center>
:::

:::info
**3. Can you find entry point in a stripped binary file, if so, use a debug tool and find the entry point of the previously compiled stripped file. If no, why?**

Since the entry point is located in the ELF header, it will be easy to find it. To get the entry point, run the command `readelf -h stripped`.

<center>

![](https://i.imgur.com/cMipSr9.png)
Figure 17: The entry point for the stripped file
</center>
:::


## The file descriptor (Bonus)

:::success
**Now we will look at the different file descriptors and learn what happens when a process opens a file.**
:::

:::info
**1. Explain why we need file descriptors?**
A file descriptor is a means for working with an I/O stream by means of a unique identifier returned
:::

:::info
**2. What is a file descriptor table in a process and what items it holds?**
The file descriptor table is a table that contains information about open files and their processes used.
:::

:::info
**3. List file descriptors of your current bash session.**

The Figure 18 does not show the output of file descriptors for the current bash session:
0 - standard input STDIN
1 - standard output stdOUT
2 - standard error stdERR 
255 - descriptor of the bash
<center>

![](https://i.imgur.com/rzYDoEm.png)
Figure 18: List file descriptors
</center>
:::

:::info
**4. Where are the locations of file descriptor (linux)?**
The file descriptor is located in the `/proc/$$/fd` folder, where `$$` is the ID of the current bash session
:::

:::info
**Redirecting file descriptors.**
Briefly explain and provide at least two examples of redirecting file descriptors.

File descriptor redirection - the ability of the command shell to redirect standard streams to a location that the user has defined

For example:
`ls > output.txt` - writes information 
`rm < input.txt` - reading arguments from a file

:::

## References:
1. [UEFI booting](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface#UEFI_booting)
2. [GNU GRUB](https://www.gnu.org/software/grub/)
3. [Boot Sequence](https://edk2-docs.gitbook.io/edk-ii-build-specification/2_design_discussion/23_boot_sequence)
4. [GRUB BOOT STAGES](https://www.system-rescue.org/disk-partitioning/Grub-boot-stages/)
6. [Special systemd units](https://www.freedesktop.org/software/systemd/man/systemd.special.html)
7. [Linux manual page](https://man7.org)
8. [Linux System Call Table](https://thevivekpandey.github.io/posts/2017-09-25-linux-system-calls.html)
9. [File descriptors](https://it.wikireading.ru/3264)
10. [Executable and Linkable Format](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)
11. [Special Section Indexes](https://refspecs.linuxbase.org/elf/gabi4+/ch4.sheader.html)
14. [Preprocessor Output](https://gcc.gnu.org/onlinedocs/cpp/Preprocessor-Output.html)
15. [Stripped binary](https://en.wikipedia.org/wiki/Stripped_binary)