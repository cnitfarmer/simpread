> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [chsasank.github.io](https://chsasank.github.io/classic_papers/unix-time-sharing-system.html)

丹尼斯·里奇（Dennis M. Ritchie）和汤普森（Ken Thompson） | 1974年7月 | 56分钟阅读。

注意：这_不是_我的文章。这是由Dennis M. Ritchie和Ken Thompson最初于1974年_在ACM通讯中_发表的经典论文。黄色高光/注释是我自己的。[禁用它们。](#)

抽象
--

UNIX是Digital Equipment Corporation PDP-11 / 40和11/45计算机的通用，多用户，交互式操作系统。它提供了即使在较大的操作系统中也很少发现的许多功能，包括：（1）包含可卸卷的分层文件系统；（2）兼容的文件，设备和进程间I / O；（3）启动异步过程的能力；（4）每个用户可以选择的系统命令语言；（5）超过100个子系统，包括十几种语言。本文讨论了文件系统和用户命令界面的性质和实现。

1.简介
----

已有三个版本的UNIX。最早的版本（大约在1969–70年）运行在Digital Equipment Corporation的PDP-7和-9计算机上。第二个版本在不受保护的PDP-11 / 20计算机上运行。本文仅描述了PDP-11 / 40和/ 45 [l]系统，因为它更现代，并且它与较旧的UNIX系统之间的许多差异是由于重新设计发现不足或缺乏的功能导致的。

自1971年2月PDP-11 UNIX投入运行以来，已经有大约40台设备投入使用。它们通常比此处描述的系统小。他们中的大多数人从事的应用程序包括专利申请和其他文本材料的准备和格式化，从贝尔系统内的各种交换机收集和处理故障数据，以及记录和检查电话服务订单。我们自己的安装程序主要用于操作系统，语言，计算机网络和计算机科学中的其他主题的研究，还用于文档准备。

也许 UNIX的最重要成就是证明用于交互使用的功能强大的操作系统无论是在设备上还是在人工上都不必昂贵：UNIX可以在成本仅为40,000美元的硬件上运行，并且在主系统软件上花费的时间不到两年。但是UNIX包含许多功能，即使在更大的系统中也很少提供。但是，希望UNIX的用户会发现系统的最重要特征是其简单性，美观性和易用性。

除了适用于系统的系统外，UNIX下可用的主要程序包括：汇编器，基于QED [2]的文本编辑器，链接加载器，符号调试器，用于类似于BCPL [3]的语言的类型和结构（C）的编译器，用于BASIC的方言，文本格式程序，Fortran编译器，Snobol解释器，自上而下的编译器（TMG）[4]，自下而上的编译器（YACC），套用信函生成器，宏处理器（M6）[5]，和排列索引程序。

还有许多维护，公用事业，娱乐和新颖性计划。所有这些程序都是本地编写的。值得注意的是，该系统完全是自立的。所有UNIX软件都在UNIX下维护；同样，UNIX文档是由UNIX编辑器和文本格式化程序生成并格式化的。

2.硬件和软件环境
---------

在其上安装UNIX的PDP-11 / 45是一台16位字（8位字节）的计算机，其核心内存为144 KB。UNIX占用42K字节。但是，该系统包含大量设备驱动程序，并为I / O缓冲区和系统表分配了足够的空间。一个能够运行上述软件的最小系统，总共可能只需要50K字节的内核。

PDP-11具有一个用于文件系统存储和交换的1M字节固定头磁盘，四个可在可移动磁盘盒上提供2.5M字节的移动头磁盘驱动器以及一个使用可移动40M字节的移动头磁盘驱动器。磁盘包。还有一个高速纸带读取器打孔机，九轨磁带和D型带（各种磁带设施，可以处理和重写单个记录）。除控制台打字机外，还有100个系列数据集附带的14个变速通信接口和201个数据集接口，主要用于将打印输出后台打印到公用行式打印机。还有几种类型的设备，包括Picturephone®接口，语音响应单元，语音合成器，照排机，数字交换网络，

UNIX软件的大部分是用上述C语言编写的[6]。操作系统的早期版本是用汇编语言编写的，但是在1973年夏季，它用C语言重写。新系统的大小比旧系统大大约三分之一。由于新系统不仅易于理解和修改，而且还进行了许多功能改进，包括多程序设计以及在多个用户程序之间共享可重入代码的能力，因此我们认为这种增加的大小是可以接受的。

3.文件系统
------

UNIX最重要的工作是提供文件系统。 ⊕S ：请注意论文如何认为文件很重要。Unix通过文件公开所有功能。 从用户的角度来看，共有三种文件：普通磁盘文件，目录和特殊文件

### 3.1普通文件

文件包含用户放置在文件上的任何信息，例如符号或二进制（对象）程序。系统不希望进行特殊的结构化。文本文件仅由一个字符串组成，其行由换行符分隔。二进制程序是单词序列，因为它们将在程序开始执行时出现在核心存储器中。一些用户程序用更多的结构来处理文件：汇编器生成，加载器期望目标文件采用特定格式。但是，文件的结构由使用它们的程序控制，而不是系统控制。

### 3.2目录

目录提供了文件名和文件本身之间的映射，从而在整个文件系统上引入了一种结构。每个用户都有一个自己文件的目录；他还可以创建子目录来包含方便地一起处理的文件组。目录的行为与普通文件完全相同，不同之处在于目录无法由非特权程序写入，因此系统可以控制目录的内容。但是，具有适当权限的任何人都可以像其他文件一样读取目录。

系统维护几个目录供自己使用。其中之一是_根_目录。通过跟踪目录链中的路径直到找到所需的文件，可以找到系统中的所有文件。此类搜索的起点通常是根本。另一个系统目录包含所有供一般使用的程序。即所有_命令_。但是，正如所看到的，程序不必驻留在此目录中即可执行。

⊕ S: This is a succinct description of path system of unix and Linux Files are named by sequences of 14 or fewer characters. When the name of a file is specified to the system, it may be in the form of a _path name_, which is a sequence of directory names separated by slashes `/` and ending in a file name. If the sequence begins with a slash, the search begins in the root directory. The name `/alpha/beta/gamma` causes the system to search the root for directory `alpha`, then to search `alpha` for `beta`, finally to find `gamma` in `beta`. `gamma` may be an ordinary file, a directory, or a special file. As a limiting case, the name `/` refers to the root itself.

A path name not starting with `/` causes the system to begin the search in the user’s current directory. Thus, the name `alpha/beta` specifies the file named `beta` in subdirectory `alpha` of the current directory. The simplest kind of name, for example `alpha`, refers to a file which itself is found in the current directory. As another limiting case, the null file name refers to the current directory.

The same nondirectory file may appear in several directories under possibly different names. This feature is called _linking_; a directory entry for a file is sometimes called a link. UNIX differs from other systems in which linking is permitted in that all links to a file have equal status. That is, a file does not exist within a particular directory; the directory entry for a file consists merely of its name and a pointer to the information actually describing the file. Thus a file exists independently of any directory entry, although in practice a file is made to disappear along with the last link to it.

Each directory always has at least two entries. The name in each directory refers to the directory itself. Thus a program may read the current directory under the name `.` without knowing its complete path name. The name `..` by convention refers to the parent of the directory in which it appears, that is, to the directory in which it was created.

The directory structure is constrained to have the form of a rooted tree. Except for the special entries `.` and `..`, each directory must appear as an entry in exactly one other,which is its parent. Te reason for this is to simplify the writing of programs which visit subtrees of the directory structure, and more important, to avoid the separation of portions of the hierarchy. If arbitrary links to directories were permitted, it would be quite difficult to detect when the last connection from the root to a directory was severed.

### 3.3 Special Files

Special files constitute the most unusual feature of the UNIX file system. Each I/O device supported by UNIX is associated with at least one such file. Special files are read and written just like ordinary disk files, but requests to read or write result in activation of the associated device. An entry for each special file resides in directory `/dev`, although a link may be made to one of these files just like an ordinary file. Thus, for example, to punch paper tape, one may write on the file `/dev/ppt`. Special files exist for each communication line, each disk, each tape drive, and for physical core memory. Of course, the active disks and the core special file are protected from indiscriminate access.

There is a threefold advantage in treating I/O devices this way: file and device I/O are as similar as possible; file and device names have the same syntax and meaning, so that a program expecting a file name as a parameter can be passed a device name; finally, special files are subject to the same protection mechanism as regular files.

### 3.4 Removable File Systems

Although the root of the file system is always stored on the same device, it is not necessary that the entire file system hierarchy reside on this device. There is a `mount` system request which has two arguments: the name of an existing ordinary file, and the name of a direct-access special file whose associated storage volume (e.g. disk pack) should have the structure of an independent file system containing its own directory hierarchy. The effect of `mount` is to cause references to the heretofore ordinary file to refer instead to the root directory of the file system on the removable volume. In effect, mount replaces a leaf of the hierarchy tree (the ordinary file) by a whole new subtree (the hierarchy stored on the removable volume). After the `mount`, there is virtually no distinction between files on the removable volume and those in the permanent file system. In our installation, for example, the root directory resides on the fixed-head disk, and the large disk drive, which contains user’s files, is mounted by the system initialization program, the four smaller disk drives are available to users for mounting their own disk packs. A mountable file system is generated by writing on its corresponding special file. A utility program is available to create an empty file system, or one may simply copy an existing file system.

There is only one exception to the rule of identical treatment of files on different devices: no link may exist between one file system hierarchy and another. This restriction is enforced so as to avoid the elaborate bookkeeping which would otherwise be required to assure removal of the links when the removable volume is finally dismounted. In particular, in the root directories of all file systems, removable or not, the name `..` refers to the directory itself instead of to its parent.

### 3.5 Protection

Although the access control scheme in UNIX is quite simple, it has some unusual features. Each user of the system is assigned a unique user identification number. When a file is created, it is marked with the user ID of its owner. Also given for new files is a set of seven protection bits. Six of these specify independently read, write, and execute permission for the owner of the file and for all other users.

If the seventh bit is on, the system will temporarily change the user identification of the current user to that of the creator of the file whenever the file is executed as a program. This change in user ID is effective only during the execution of the program which calls for it. The set-user-ID feature provides for privileged programs which may use files inaccessible to other users. For example, a program may keep an accounting file which should neither be read nor changed except by the program itself. If the set-user-identification bit is on for the program, it may access the file although this access might be forbidden to other programs invoked by the given program’s user. Since the actual user ID of the invoker of any program is always available, set-user-ID programs may take any measures desired to satisfy themselves as to their invoker’s credentials. This mechanism is used to allow users to execute the carefully written commands which call privileged system entries. For example, there is a system entry invocable only by the “super-user” (below) which creates an empty directory. As indicated above, directories are expected to have entries for `.` and `..` . The command which creates a directory is owned by the superuser and has the set-user-ID bitset. After it checks its invoker’s authorization to create the specified directory, it creates it and makes the entries for `.` and `..` .

Since anyone may set the set-user-ID bit on one of his own files, this mechanism is generally available without administrative intervention. For example, this protection scheme easily solves the MOO accounting problem posed in [7].

The system recognizes one particular user ID (that of the “super-user”) as exempt from the usual constraints on file access; thus (for example) programs may be written to dump and reload the file system without unwanted interference from the protection system.

### 3.6 I/O Calls

The system calls to do I/O are designed to eliminate the differences between the various devices and styles of access. There is no distinction between "random" and sequential I/O, nor is any logical record size imposed by the system. The size of an ordinary file is determined by the highest byte written on it; no predetermination of the size of a file is necessary or possible.

To illustrate the essentials of I/O in UNIX, Some of the basic calls are summarized below in an anonymous language which will indicate the required parameters without getting into the complexities of machine language programming. Each call to the system may potentially result in an error return, which for simplicity is not represented in the calling sequence.

To read or write a file assumed to exist already, it must be opened by the following call:

```
filep = open(name, flag)
```

`name` indicates the name of the file. An arbitrary path name may be given. The `flag` argument indicates whether the file is to be read, written, or “updated”, that is read and written simultaneously.

The returned value `filep` is called a file descriptor. It is a small integer used to identify the file in subsequent calls to read, write, or otherwise manipulate it.

To create a new file or completely rewrite an old one, there is a `create` system call which creates the given file if it does not exist, or truncates it to zero length if it does exist. `create` also opens the new file for writing and, like `open`, returns a file descriptor.

There are no user-visible locks in the file system, nor is there any restriction on the number of users who may have a file open for reading or writing; although it is possible for the contents of a file to become scrambled when two users write on it simultaneously, in practice, difficulties do not arise. We take the view that locks are neither necessary nor sufficient, in our environment, to prevent interference between users of the same file. They are unnecessary because we are not faced with large, single-file data bases maintained by independent processes. They are insufficient because locks in the ordinary sense, whereby one user is prevented from writing on a file which another user is reading, cannot prevent confusion when, for example, both users are editing a file with an editor which makes a copy of the file being edited.

It should be said that the system has sufficient internal interlocks to maintain the logical consistency of the file system when two users engage simultaneously in such inconvenient activities as writing on the same file, creating files in the same directory or deleting each other’s open files.

Except as indicated below, reading and writing are sequential. This means that if a particular byte in the file was the last byte written (or read), the next I/O call implicitly refers to the first following byte. For each open file there is a pointer, maintained by the system, which indicates the next byte to be read or written. If n bytes are read or written, the pointer advances by n bytes.

Once a file is open, the following calls may be used:

```
n = read(filep, buffer, count)
n = write(filep, buffer, count)
```

Up to `count` bytes are transmitted between the file specified by `filep` and the byte array specified by `buffer`. The returned value `n` is the number of bytes actually transmitted. In the `write` case, `n` is the same as `count` except under exceptional conditions like I/O errors or end of physical medium on special files; in a `read`, however, `n` may without error be less than `count`. If the read pointer is so near the end of the file that reading `count` characters would cause reading beyond the end, only sufficient bytes are transmitted to reach the end of the file; also, typewriter-like devices never return more than one line of input. When a `read` call returns with `n` equal to zero, it indicates the end of the file. For disk files this occurs when the read pointer becomes equal to the current size of the file. It is possible to generate an end-of-file from a typewriter by use of an escape sequence which depends on the device used.

Bytes written on a file affect only those implied by the position of the write pointer and the count; no other part of the file is changed. If the last byte lies beyond the end of the file, the file is grown as needed.

To do random (direct access) I/O, it is only necessary to move the read or write pointer to the appropriate location in the file:

```
location = seek(filep, base, offset)
```

The pointer associated with `filep` is moved to a position `offset` bytes from the beginning of the file, from the current position of the pointer, or from the end of the file, depending on `base`. `offset` may be negative. For some devices (e.g. paper tape and typewriters) `seek` calls are ignored. The actual offset from the beginning of the file to which the pointer was moved is returned in `location`.

**3.6.1: Other I/O Calls**: There are several additional system entries having to do with I/O and with the file system which will not be discussed. For example: close a file, get the status of a file, change the protection mode or the owner of a file, create a directory, make a link to an existing file, delete a file.

4. Implementation of the File System
------------------------------------

⊕ S: Implementation of a toy file system (in rust?) makes for an illustrative exercise. Nice blog idea! As mentioned in §3.2 above, a directory entry contains only a name for the associated file and a pointer to the file itself. This pointer is an integer called the `i-number` (for index number) of the file. When the file is accessed, its `i-number` is used as an index into a system table (the `i-list`) stored in a known part of the device on which the directory resides. The entry thereby found (the file’s `i-node`) contains the description of the file as follows.

1.  Its owner.
2.  Its protection bits.
3.  The physical disk or tape addresses for the file contents.
4.  Its size.
5.  Time of last modification.
6.  The number of links to the file, that is, the number of times it appears in a directory.
7.  A bit indicating whether the file is a directory.
8.  A bit indicating whether the file is a special file.
9.  A bit indicating whether the file is “large” or “small.”

The purpose of an `open` or `create` system call is to turn the path name given by the user into an `i-number` by searching the explicitly or implicitly named directories. Once a file is open, its device, `i-number`, and read/write pointer are stored in a system table indexed by the file descriptor returned by the open or create. Thus the file descriptor supplied during a subsequent call to read or write the file may be easily related to the information necessary to access the file.

When a new file is created, an `i-node` is allocated for it and a directory entry is made which contains the name of the file and the `i-node` number. Making a link to an existing file involves creating a directory entry with the new name, copying the `i-number` from the original file entry, and incrementing the link-count field of the `i-node`. Removing (deleting) a file is done by decrementing the link-count of the `i-node` specified by its directory entry and erasing the directory entry. If the link-count drops to 0, any disk blocks in the file are freed and the `i-node` is deallocated.

The space on all fixed or removable disks which contain a file system is divided into a number of 512-byte blocks logically addressed from 0 up to a limit which depends on the device. There is space in the `i-node` of each file for eight device addresses. A _small_ (nonspecial) file fits into eight or fewer blocks; in this case the addresses of the blocks themselves are stored. For _large_ (nonspecial) files, each of the eight device addresses may point to an indirect block of 256 addresses of blocks constituting the file itself. These files may be as large as 8⋅256⋅512, or l,048,576 (220) bytes.

The foregoing discussion applies to ordinary files. When an I/O request is made to a file whose `i-node` indicates that it is special, the last seven device address words are immaterial, and the list is interpreted as a pair of bytes which constitute an internal _device_ name. These bytes specify respectively a device type and subdevice number. The device type indicates which system routine will deal with I/O on that device; the subdevice number selects, for example, a disk drive attached to a particular controller or one of several similar typewriter interfaces.

In this environment, the implementation of the `mount` system call (§3.4) is quite straightforward. `mount` maintains a system table whose argument is the `i-number` and device name of the ordinary file specified during the mount, and whose corresponding value is the device name of the indicated special file. This table is searched for each (`i-number`, device)-pair which turns up while a path name is being scanned during an open or create; if a match is found, the `i-number` is replaced by 1 (which is the `i-number` of the root directory on all file systems), and the device name is replaced by the table value.

To the user, both reading and writing of files appear to be synchronous and unbuffered. That is immediately after return from a `read` call the data are available, and conversely after a `write` the user’s workspace may be reused. In fact the system maintains a rather complicated buffering mechanism which reduces greatly the number of I/O operations required to access a file. Suppose a `write` call is made specifying transmission of a single byte.

UNIX will search its buffers to see whether the affected disk block currently resides in core memory; if not, it will be read in from the device. Then the affected byte is replaced in the buffer, and an entry is made in a list of blocks to be written. The return from the `write` call may then take place, although the actual I/O may not be completed until a later time. Conversely, if a single byte is read, the system determines whether the secondary storage block in which the byte is located is already in one of the system’s buffers; if so, the byte can be returned immediately. If not, the block is read into a buffer and the byte picked out.

A program which reads or writes files in units of 512 bytes has an advantage over a program which reads or writes a single byte at a time, but the gain is not immense; it comes mainly from the avoidance of system overhead. A program which is used rarely or which does no great volume of I/O may quite reasonably read and write in units as small as it wishes.

The notion of the `i-list` is an unusual feature of UNIX. In practice, this method of organizing the file system has proved quite reliable and easy to deal with. To the system itself, one of its strengths is the fact that each file has a short, unambiguous name which is related in a simple way to the protection, addressing, and other information needed to access the file. It also permits a quite simple and rapid algorithm for checking the consistency of a file system, for example verification that the portions of each device containing useful information and those free to be allocated are disjoint and together exhaust the space on the device. This algorithm is independent of the directory hierarchy, since it need only scan the linearly-organized `i-list`. At the same time the notion of the `i-list` induces certain peculiarities not found in other file system organizations. For example, there is the question of who is to be charged for the space a file occupies, since all directory entries for a file have equal status. Charging the owner of a file is unfair, in general, since one user may create a file, another may link to it, and the first user may delete the file. The first user is still the owner of the file, but it should be charged to the second user. The simplest reasonably fair algorithm seems to be to spread the charges equally among users who have links to a file. The current version of UNIX avoids the issue by not charging any fees at all.

### 4.1 Efficiency of the File System

To provide an indication of the overall efficiency of UNIX and of the file system in particular, timings were made of the assembly of a 7621-line program. The assembly was run alone on the machine; the total clock time was 35.9 sec, for a rate of 212 lines per sec. The time was divided as follows: 63.5 percent assembler execution time, 16.5 percent system overhead, 20.0 percent disk wait time. We will not attempt any interpretation of these figures nor any comparison with other systems, but merely note that we are generally satisfied with the overall performance of the system.

5. Processes and Images
-----------------------

An *image* is a computer execution environment. It includes a core image, general register values, status of open files, current directory, and the like. An image is the current state of a pseudo computer.

A *process* is the execution of an image. While the processor is executing on behalf of a process, the image must reside in core; during the execution of other processes it remains in core unless the appearance of an active, higher-priority process forces it to be swapped out to the fixed-head disk.

The user-core part of an image is divided into three logical segments. The program text segment begins at location 0 in the virtual address space. During execution, this segment is write-protected and a single copy of it is shared among all processes executing the same program. At the first 8K byte boundary above the program text segment in the virtual address space begins a non-shared, writable data segment, the size of which may be extended by a system call. Starting at the highest address in the virtual address space is a stack segment, which automatically grows downward as the hardware’s stack pointer fluctuates.

### 5.1 Processes

Except while UNIX is bootstrapping itself into operation, a new process can come into existence only by use of the `fork` system call:

```
processid = fork(label)
```

When `fork` is executed by a process, it splits into two independently executing processes. The two processes have independent copies of the original core image, and share any open files. The new processes differ only in that one is considered the parent process: in the parent, control returns directly from the `fork`, while in the child, control is passed to location `label`. The `processid` returned by the `fork` call is the identification of the other process.

Because the return points in the parent and child process are not the same, each image existing after a `fork` may determine whether it is the parent or child process.

### 5.2 Pipes

Processes may communicate with related processes using the same system read and write calls that are used for file system I/O. The call

returns a file descriptor `filep` and creates an interprocess channel called a `pipe`. This channel, like other open flies, is passed from parent to child process in the image by the `fork` call. A `read` using a pipe file descriptor waits until another process writes using the file descriptor for the same pipe. At this point, data are passed between the images of the two processes. Neither process need know that a pipe, rather than an ordinary file, is involved.

Although interprocess communication via pipes is a quite valuable tool (see §6.2), it is not a completely general mechanism since the pipe must be set up by a common ancestor of the processes involved.

### 5.3 Execution of Programs

Another major system primitive is invoked by

```
filep = pipe()
```

which requests the system to read in and execute the program named by `file`, passing it string arguments `arg1`, `arg2`,…, `argn`. Ordinarily, `arg1` should be the same string as `file`, so that the program may determine the name by which it was invoked. All the code and data in the process using `execute` is replaced from the file, but open files, current directory, and interprocess relationships are unaltered. Only if the call fails, for example because `file` could not be found or because its execute-permission bit was not set, does a return take place from the `execute` primitive; it resembles a “jump” machine instruction rather than a subroutine call.

### 5.4 Process Synchronization

Another process control system call

```
execute(file, arg1, arg2, ..., argn)
```

causes its caller to suspend execution until one of its children has completed execution. Then `wait` returns the `processid` of the terminated process. An error return is taken if the calling process has no descendants. Certain status from the child process is also available. `wait` may also present status from a grandchild or more distant ancestor; see §5.5.

### 5.5 Termination

Lastly,

terminates a process, destroys its image, closes its open files, and generally obliterates it. When the parent is notified through the `wait` primitive, the indicated `status` is available to the parent; if the parent has already terminated, the status is available to the grandparent, and so on. Processes may also terminate as a result of various illegal actions or user-generated signals (§7 below).

6. The Shell
------------

For most users, communication with UNIX is carried on with the aid of a program called the Shell. The Shell is a command line interpreter: it reads lines typed by the user and interprets them as requests to execute other programs. In simplest form, a command line consists of the command name followed by arguments to the command, all separated by spaces:

```
processid = wait()
```

The Shell splits up the command name and the arguments into separate strings. Then a file with name `command` is sought; `command` may be a path name including the `/` character to specify any file in the system. If `command` is found, it is brought into core and executed. The arguments collected by the Shell are accessible to the command. When the command is finished, the Shell resumes its own execution, and indicates its readiness to accept another command by typing a prompt character.

If file `command` cannot be found, the Shell prefixes the string `/bin/` to command and attempts again to find the file. Directory `/bin` contains all the commands intended to be generally used.

### 6.1 Standard I/O

⊕ S: Discussion of stdout, stdin and stderr follows. The discussion of I/O in §3 above seems to imply that every file used by a program must be opened or created by the program in order to get a file descriptor for the file. Programs executed by the Shell, however, start off with two open files which have file descriptors 0 and 1. As such a program begins execution, file 1 is open for writing, and is best understood as the standard output file. Except under circumstances indicated below, this file is the user’s typewriter. Thus programs which wish to write informative or diagnostic information ordinarily use file descriptor 1. Conversely, file 0 starts off open for reading, and programs which wish to read messages typed by the user usually read this file.

⊕ S: Redirection and filering are some of the best features on the Unix. These are easily implemented thanks to UNIX’s choice to consider every I/O as file. The Shell is able to change the standard assignments of these file descriptors from the user’s typewriter printer and keyboard. If one of the arguments to a command is prefixed by `>`, file descriptor 1 will, for the duration of the command, refer to the file named after the `>`. For example,

ordinarily lists, on the typewriter, the names of the files in the current directory. The command

creates a file called `there` and places the listing there. Thus the argument `>there` means, “place output on `there`.” On the other hand,

ordinarily enters the editor, which takes requests from the user via his typewriter. The command

interprets `script` as a file of editor commands; thus `<script` means, “take input from script.”

Although the file name following `<` or `>` appears to be an argument to the command, in fact it is interpreted completely by the Shell and is not passed to the command at all. Thus no special coding to handle I/O redirection is needed within each command; the command need merely use the standard file descriptors 0 and 1 where appropriate.

### 6.2 Filters

An extension of the standard I/O notion is used to direct output from one command to the input of another. A sequence of commands separated by vertical bars causes the Shell to execute all the commands simultaneously and to arrange that the standard output of each command be delivered to the standard input of the next command in the sequence. Thus in the command line

`ls` lists the names of the files in the current directory; its output is passed to `pr`, which paginates its input with dated headings. The argument `–2` means double column. Likewise the output from `pr` is input to `opr`. This commands pools its input onto a file for off-line printing.

This process could have been carried out more clumsily by

```
exit(status)
```

followed by removal of the temporary files. In the absence of the ability to redirect output and input, a still clumsier method would have been to require the `ls` command to accept user requests to paginate its output, to print in multi-column format, and to arrange that its output be delivered off-line. Actually it would be surprising, and in fact unwise for efficiency reasons, to expect authors of commands such as `ls` to provide such a wide variety of output options.

A program such as `pr` which copies its standard input to its standard output (with processing) is called a _filter_. Some filters which we have found useful perform character transliteration, sorting of the input, and encryption and decryption.

### 6.3 Command Separators: Multitasking

Another feature provided by the Shell is relatively straightforward. Commands need not be on different lines;instead they may be separated by semicolons.

will first list the contents of the current directory, then enter the editor.

A related feature is more interesting. If a command is followed by `&`, the Shell will not wait for the command to finish before prompting again; instead, it is ready immediately to accept a new command. For example,

causes source to be assembled, with diagnostic output going to output; no matter how long the assembly takes, the Shell returns immediately. When the Shell does not wait for the completion of a command, the identification of the process running that command is printed. This identification may be used to wait for the completion of the command or to terminate it. The `&` may be used several times in a line:

```
command arg1 arg2 ... argn
```

does both the assembly and the listing in the background. In the examples above using `&`, an output file other than the typewriter was provided; if this had not been done, the outputs of the various commands would have been intermingled.

The Shell also allows parentheses in the above operations. For example,

prints the current date and time followed by a list of the current directory onto the file `x`. The Shell also returns immediately for another request.

### 6.4 The Shell as a Command: Command files

⊕ S: Description of awesome shell scripts follows. The Shell is itself a command, and may be called recursively. Suppose file `tryout` contains the lines

```
ls
```

The `mv` command causes the file `a.out` to be renamed `testprog`. `a.out` is the (binary) output of the assembler, ready to be executed. Thus if the three lines above were typed on the console, `source` would be assembled, the resulting program named `testprog`, and `testprog` executed. When the lines are in `tryout`, the command

would cause the Shell `sh` to execute the commands sequentially.

The Shell has further capabilities, including the ability to substitute parameters and to construct argument lists from a specified subset of the file names in a directory. It is also possible to execute commands conditionally on character string comparisons or on existence of given files and to perform transfers of control within filed command sequences.

### 6.5 Implementation of the Shell

⊕ S: Shell’s implementation is remarkably simple thanks to the design choices described earlier in the paper. The outline of the operation of the Shell can now be understood. Most of the time, the Shell is waiting for the user to type a command. When the new-line character ending the line is typed, the Shell’s `read` call returns. The Shell analyzes the command line, putting the arguments in a form appropriate for `execute`. Then `fork` is called. The child process, whose code of course is still that of the Shell, attempts to perform an `execute` with the appropriate arguments. If successful, this will bring in and start execution of the program whose name was given. Meanwhile, the other process resulting from the `fork`, which is the parent process, `wait`s for the child process to die. When this happens, the Shell knows the command is finished, so it types its prompt and reads the typewriter to obtain another command.

Given this framework, the implementation of background processes is trivial; whenever a command line contains `&`, the Shell merely refrains from waiting for the process which it created to execute the command.

Happily, all of this mechanism meshes very nicely with the notion of standard input and output files. When a process is created by the `fork` primitive, it inherits not only the core image of its parent but also all the files currently open in its parent, including those with file descriptors 0 and 1. The Shell, of course, uses these files to read command lines and to write its prompts and diagnostics, and in the ordinary case its children — the command programs — inherit them automatically. When an argument with `<` or `>` is given however, the offspring process, just before it performs `execute`, makes the standard I/O file descriptor 0 or 1 respectively refer to the named file. This is easy because, by agreement, the smallest unused file descriptor is assigned when a new file is opened (or created); it is only necessary to close file 0 (or 1) and open the named file. Because the process in which the command program runs simply terminates when it is through, the association between a file specified after `<` or `>` and file descriptor 0 or 1 is ended automatically when the process dies. Therefore the Shell need not know the actual names of the files which are its own standard input and output since it need never reopen them.

Filters are straightforward extensions of standard I/O redirection with pipes used instead of files.

In ordinary circumstances, the main loop of the Shell never terminates. (The main loop includes that branch of the return from `fork` belonging to the parent process; that is, the branch which does a `wait`, then reads another command line.) The one thing which causes the Shell to terminate is discovering an end-of-file condition on its input file. Thus when the Shell is executed as a command with a given input file, as in

the commands in `comfile` will be executed until the end of `comfile` is reached; then the instance of the Shell invoked by `sh` will terminate. Since this Shell process is the child of another instance of the Shell, the `wait` executed in the latter will return, and another command may be processed.

### 6.6 Initialization

⊕ S: Booting and logging-in are described. The instances of the Shell to which users type commands are themselves children of another process. The last step in the initialization of UNIX is the creation of a single process and the invocation (via `execute`) of a program called `init`. The role of `init` is to create one process for each typewriter channel which may be dialed up by a user. The various subinstances of `init` open the appropriate typewriters for input and output. Since when `init` was invoked there were no files open, in each process the typewriter keyboard will receive file descriptor 0 and the printer file descriptor 1. Each process types out a message requesting that the user log in and waits, reading the typewriter, for a reply. At the outset, no one is logged in, so each process simply hangs. Finally someone types his name or other identification. The appropriate instance of `init` wakes up, receives the log-in line, and reads a password file. If the user name is found, and if he is able to supply the correct password, init changes to the user’s default current directory, sets the process’s user ID to that of the person logging in, and performs an execute of the Shell. At this point the Shell is ready to receive commands and the logging-in protocol is complete

Meanwhile, the mainstream path of `init` (the parent of all the subinstances of itself which will later become Shells) does a `wait`. If one of the child processes terminates, either because a Shell found an end of file or because a user typed an incorrect name or password, this path of init simply recreates the defunct process, which in turn reopens the appropriate input and output files and types another login message. Thus a user may log out simply by typing the end-of-file sequence in place of a command to the Shell.

### 6.7 Other Programs as Shell

The Shell as described above is designed to allow users full access to the facilities of the system since it will invoke the execution of any program with appropriate protection mode. Sometimes, however, a different interface to the system is desirable, and this feature is easily arranged.

Recall that after a user has successfully logged in by supplying his name and password, `init` ordinarily invokes the Shell to interpret command lines. The user’s entry in the password file may contain the name of a program to be invoked after login instead of the Shell. This program is free to interpret the user’s messages in any way it wishes.

For example, the password file entries for users of a secretarial editing system specify that the editor `ed` is to be used instead of the Shell. Thus when editing system users log in, they are inside the editor and can begin work immediately; also, they can be prevented from invoking UNIX programs not intended for their use. In practice, it has proved desirable to allow a temporary escape from the editor to execute the formatting program and other utilities.

Several of the games (e.g. chess, blackjack, 3D tic-tac-toe) available on UNIX illustrate a much more severely restricted environment. For each of these an entry exists in the password file specifying that the appropriate game-playing program is to be invoked instead of the Shell. People who log in as a player of one of the games find themselves limited to the game and unable to investigate the presumably more interesting offerings of UNIX as a whole.

7. Traps
--------

The PDP-11 hardware detects a number of program faults, such as references to nonexistent memory, unimplemented instructions, and odd addresses used where an even address is required. Such faults cause the processor to trap to a system routine. When an illegal action is caught, unless other arrangements have been made, the system terminates the process and writes the user’s image on file core in the current directory. A debugger can be used to determine the state of the program at the time of the fault.

⊕ S: How ctrl-c, our all-killer, works is described. Programs which are looping, which produce unwanted output, or about which the user has second thoughts may be halted by the use of the `interrupt` signal, which is generated by typing the “delete” character. Unless special action has been taken, this signal simply causes the program to cease execution without producing a core image file.

There is also a `quit` signal which is used to force a core image to be produced. Thus programs which loop unexpectedly may be halted and the core image examined without prearrangement.

The hardware-generated faults and the interrupt and quit signals can, by request, be either ignored or caught by the process. For example, the Shell ignores quits to prevent a quit from logging the user out. The editor catches interrupts and returns to its command level. This is useful for stopping long printouts without losing work in progress (the editor manipulates a copy of the file it is editing). In systems without floating point hardware, unimplemented instructions are caught, and floating point instructions are interpreted.

8. Perspective
--------------

⊕ S: A succinct discussion and retrospective of the development process. Perhaps paradoxically, the success of UNIX is largely due to the fact that it was not designed to meet any predefined objectives. The first version was written when one of us (Thompson), dissatisfied with the available computer facilities, discovered a little-used system PDP-7 and set out to create a more hospitable environment. This essentially personal effort was sufficiently successful to gain the interest of the remaining author and others, and later to justify the acquisition of the PDP-11/20, specifically to support a text editing and formatting system. Then in turn the 11/20 was outgrown, UNIX had proved useful enough to persuade management to invest in the PDP-11/45. Our goals throughout the effort, when articulated at all, have always concerned themselves with building a comfortable relationship with the machine and with exploring ideas and inventions in operating systems. We have not been faced with the need to satisfy someone else's requirements, and for this freedom we are grateful.

Three considerations which influenced the design of UNIX are visible in retrospect.

First, since we are programmers, we naturally designed the system to make it easy to write, test, and run programs.The most important expression of our desire for programming convenience was that the system was arranged for interactive use, even though the original version only supported one user. We believe that a properly designed interactive system is much more productive and satisfying to use than a “batch” system. Moreover such a system is rather easily adaptable to non-interactive use, while the converse is not true.

Second there have always been fairly severe size constraints on the system and its software. Given the partiality antagonistic desires for reasonable efficiency and expressive power, the size constraint has encouraged not only economy but a certain elegance of design. This may be a thinly disguised version of the "salvation through suffering" philosophy, but in our case it worked.

Third, nearly from the start, the system was able to, and did, maintain itself. This fact is more important than it might seem. If designers of a system are forced to use that system, they quickly become aware of its functional and superficial deficiencies and are strongly motivated to correct them before it is too late. Since all source programs were always available and easily modified online, we were willing to revise and rewrite the system and its software when new ideas were invented, discovered, or suggested by others.

The aspects of UNIX discussed in this paper exhibit clearly at least the first two of these design considerations. The interface to the file system, for example, is extremely convenient from a programming standpoint. The lowest possible interface level is designed to eliminate distinctions between the various devices and files and between direct and sequential access. No large “access method” routines are required to insulate the programmer from the system calls; in fact, all user programs either call the system directly or use a small library program, only tens of instructions long, which buffers a number of characters and reads or writes them all at once.

Another important aspect of programming convenience is that there are no “control blocks” with a complicated structure partially maintained by and depended on by the file system or other system calls. Generally speaking, the contents of a program’s address space are the property of the program, and we have tried to avoid placing restrictions on the data structures within that address space.

Given the requirement that all programs should be usable with any file or device as input or output, it is also desirable from a space-efficiency standpoint to push device-dependent considerations into the operating system itself. The only alternatives seem to be to load routines for dealing with each device with all programs, which is expensive in space, or to depend on some means of dynamically linking to the routine appropriate to each device when it is actually needed, which is expensive either in overhead or in hardware.

Likewise, the process control scheme and command interface have proved both convenient and efficient. Since the Shell operates as an ordinary, swappable user program, it consumes no wired-down space in the system proper, and it may be made as powerful as desired at little cost, In particular, given the framework in which the Shell executes as a process which spawns other processes to perform commands, the notions of I/O redirection, background processes, command files, and user-selectable system interfaces all become essentially trivial to implement.

### 8.1 Influences

The success of UNIX lies not so much in new inventions but rather in the full exploitation of a carefully selected set of fertile ideas, and especially in showing that they can be keys to the implementation of a small yet powerful operating system.

The `fork` operation, essentially as we implemented it, was present in the Berkeley time-sharing system [8]. On a number of points we were influenced by Multics, which suggested the particular form of the I/O system calls [9] and both the name of the Shell and its general functions, The notion that the Shell should create a process for each command was also suggested to us by the early design of Multics, although in that system it was later dropped for efficiency reasons. A similar scheme is used by TENEX [10].

9. Statistics
-------------

The following statistics from UNIX are presented to show the scale of the system and to show how a system of this scale is used. Those of our users not involved in document preparation tend to use the system for program development, especially language work. There are few important “applications” programs.

### 9.1 Overall

<table><tbody><tr><td>72</td><td>user population</td></tr><tr><td>14</td><td>maximum simultaneous users</td></tr><tr><td>300</td><td>directories</td></tr><tr><td>4400</td><td>files</td></tr><tr><td>34000</td><td>512-byte secondary storage blocks used</td></tr></tbody></table>

### 9.2 Per day (24-hour day, 7-day week basis)

There is a “background” process that runs at the lowest possible priority; it is used to soak up any idle CPU time. It has been used to produce a million-digit approximation to the constant e – 2, and is now generating composite pseudoprimes (base 2).

<table><tbody><tr><td>1800</td><td>Commands</td></tr><tr><td>4.3</td><td>CPU hours (aside from background)</td></tr><tr><td>70</td><td>connect hours</td></tr><tr><td>30</td><td>different users</td></tr><tr><td>75</td><td>logins</td></tr></tbody></table>

### 9.3 Command CPU Usage (cut off at 1%)

<table><tbody><tr><td>15.7%</td><td>C compiler</td><td>1.7%</td><td>Fortran compiler</td></tr><tr><td>15.2%</td><td>使用者<h-char unicode="2019" class="biaodian cjk bd-close bd-end punct">'</h-char> 程式</td><td>1.6％</td><td>删除文件</td></tr><tr><td>11.7％</td><td>编辑</td><td>1.6％</td><td>磁带存档</td></tr><tr><td>5.8％</td><td>Shell（用作包含命令时间的命令）</td><td>1.6％</td><td>文件系统一致性检查</td></tr><tr><td>5.3％</td><td>棋</td><td>1.4％</td><td>图书馆维护者</td></tr><tr><td>3.3％</td><td>清单目录</td><td>1.3％</td><td>连接/打印文件</td></tr><tr><td>3.1％</td><td>文件格式器</td><td>1.3％</td><td>分页并打印文件</td></tr><tr><td>1.6％</td><td>备份转储</td><td>1.1％</td><td>打印磁盘使用</td></tr><tr><td>1.8％</td><td>汇编器</td><td>1.0％</td><td>拷贝文件</td></tr></tbody></table>

### 9.4命令访问（截止为1％）

<table><tbody><tr><td>15.3％</td><td>编辑</td><td>1.6％</td><td>调试器</td></tr><tr><td>9.6％</td><td>清单目录</td><td>1.6％</td><td>Shell（用作命令）</td></tr><tr><td>6.3％</td><td>删除文件</td><td>1.5％</td><td>打印磁盘可用性</td></tr><tr><td>6.3％</td><td>C编译器</td><td>1.4％</td><td>列出正在执行的进程</td></tr><tr><td>6.0％</td><td>连接/打印文件</td><td>1.4％</td><td>汇编器</td></tr><tr><td>6.0％</td><td>使用者<h-char unicode="2019" class="biaodian cjk bd-close bd-end punct">'</h-char> 程式</td><td>1.4％</td><td>打印参数</td></tr><tr><td>3.3％</td><td>列出登录系统的人</td><td>1.2％</td><td>拷贝文件</td></tr><tr><td>3.2％</td><td>重命名/移动文件</td><td>1.1％</td><td>分页并打印文件</td></tr><tr><td>3.1％</td><td>文件状态</td><td>1.1％</td><td>打印当前日期/时间</td></tr><tr><td>1.8％</td><td>图书馆维护者</td><td>1.1％</td><td>文件系统一致性检查</td></tr><tr><td>1.8％</td><td>文件格式器</td><td>1.0％</td><td>磁带存档</td></tr><tr><td>1.6％</td><td>有条件地执行另一个命令</td><td>&nbsp;</td><td>&nbsp;</td></tr></tbody></table>

### 9.5可靠性

我们关于可靠性的统计数据比其他数据更为主观。以下结果是我们合并后的记录中最好的。时间跨度超过一年，年份非常早，年份为11/45。

由于软件无法应对导致重复的电源故障陷阱的硬件问题，导致文件系统丢失了一次（五个磁盘中的一个磁盘）。该磁盘上的文件已备份三天。

一种 “崩溃”是计划外的系统重新引导或停止。每隔一天大约发生一次车祸；其中约三分之二是由硬件相关的困难引起的，例如电源中断和对随机位置的莫名其妙的处理器中断。其余是软件故障。最长的不间断运行时间约为两周。服务呼叫平均每三周进行一次，但群集非常密集。总的正常运行时间约为我们24小时365天计划的98％。

_致谢_。我们感谢RH Canaday，LL Cherry和LE McMahon对UNIX的贡献。我们特别感谢R. Morris，MD McIlroy和JF Ossanna的创造力，深思熟虑的批评和不断的支持。

**参考文献**：

1.  数字设备公司。PDP-11 / 40处理器手册，1972年和PDP-11 / 45处理器手册。1971年。
2.  Deutsch，LP和Lampson，BW在线编辑。通讯 ACM 10，12（1967年12月）793–799，803。
3.  Richards，M. BCPL：用于编译器编写和系统编程的工具。进程 AFIPS 1969 SJCC，第一卷 34，AFIPS出版社，新泽西州蒙特维尔，第557–566页。
4.  McClure，RM TMG —一种语法指导的编译器。进程 ACM全国排名20 大会，ACM，1965年，纽约，第262-274页。
5.  大厅。AD M6宏处理器。计算科学技术。代表＃2，贝尔电话实验室，1969年。
6.  Ritchie，DM C参考手册。未发表的备忘录，贝尔电话实验室，1973年。
7.  Aleph-null。计算机娱乐。软件实践和经验1，2（1971年6月至1971年6月），201-204。
8.  Deutsch，LP和Lampson，BW SDS 930分时系统初步参考手册。Doc。1965年10月30日，美国加州大学伯克利分校GENIE项目，1965年4月。
9.  Feiertag。RJ和Organick，EI Multics输入输出系统。进程 第三症状。在歌剧上。Syst。1971年10月18日至20日，普林斯，ACM，纽约，第35至41页。
10.  DC的Bobrow，J。Burchfiel的JD，DL的Murphy和RS TENEX的Tomlinson，这是一种用于PDP-10的分页时间共享系统。通讯 ACM 15，3（1972年3月）135-143。