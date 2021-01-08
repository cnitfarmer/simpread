> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [madaidans-insecurities.github.io](https://madaidans-insecurities.github.io/guides/linux-hardening.html)

Linux强化指南
=========

_上次编辑时间：2020年12月25日_

[Linux不是安全的操作系统](../linux.html)。但是，您可以采取一些步骤进行改进。本指南初步说明如何调整地加强Linux的安全性和专有性。本指南试图与发行版无关，并且不限于任何特定的指南。  
  
免责声明：如果您不确定自己在做什么，请不要尝试在这里中应用任何内容。本指南仅关注安全性和专有性，而不关注性能，可用或其他任何内容。  
  
本指南中列出的所有命令都将需要root特权。以 “$” 符号开头的单词表示一个变量，用户之间可能会有所不同，以适应其设置。

内容
--

[1.选择正确的Linux发行版](#choosing-the-right-distro)  
  
[2。内核](#kernel) [增强](#sysctl)  
[2.1稳定与LTS](#stable-vs-lts)  
[2.2 Sysctl](#sysctl)  
[2.2.1内核自我保护](#sysctl-kernel)  
[2.2.2网络](#sysctl-network)  
[2.2.3用户空间](#sysctl-userspace)  
[2.3引导参数](#boot-parameters)  
[2.3.1内核自我保护](#boot-kernel)  
[2.3.2 CPU占用](#cpu-mitigations)  
[2.3 .3结果](#result)  
[2.4 hidepid](#hidepid)  
[2.5减少内核攻击面](#kernel-attack-surface-reduction)  
[2.5。 1引导参数](#kasr-boot-parameters)  
[2.5.2将内核模块列入黑名单](#kasr-kernel-modules)  
[2.5.3 rfkill](#kasr-rfkill)  
[2.6其他内核指针泄漏](#other-kernel-pointer-leaks)  
[2.7限制对sysfs的访问权限](#restricting-sysfs)  
[2.8 linux-hardened](#linux-hardened)  
[2.9 Grsecurity](#grsecurity)  
[2.10 Linux内核Runtime Guard](#lkrg)  
[2.11内核自我保护编译](#kernel-self-compilation)  
  
[3。强制访问控制](#mac)  
  
[4。沙箱](#sandboxing)  
[4.1应用程序沙箱](#application-sandboxing)  
[4.2通用沙箱逸出](#common-sandbox-escapes)  
[4.2.1 PulseAudio](#pulseaudio)  
[4.2.2 D-Bus](#d-bus)  
[4.2.3 GUI隔离](#gui-isolation)  
[4.2.4 ptrace](#ptrace)  
[4.2.5 TIOCSTI](#tiocsti)  
[4.3系统服务沙箱](#systemd-sandboxing)  
[4.4 gVisor](#gvisor)  
[4.5虚拟机](#virtual-machines)  
  
[5.强化的内存分配器](#hardened-malloc)  
  
[6.强化的编译标志](#hardened-compilation-flags)  
  
[7.内存安全语言](#memory-safe-languages)  
  
[8.根帐户](#root)  
[8.1 / etc / securetty](#securetty)  
[8.2限制su](#restricting-su)  
[8.3锁定根帐户](#locking-root)  
[8.4拒绝通过SSH的root登录](#denying-ssh-root-login)  
[8.5增加哈希回合数](#increase-hashing-rounds)  
[8.6限制Xorg根访问](#restricting-xorg)  
[8.7安全访问根](#accessing-root-securely)  
  
[9.防火墙](#firewalls)  
  
[10.标识符](#identifiers)  
  
[10.1主机名和用户名](#hostnames)  
[10.2时区/语言环境/键映射](#timezones-locales-keymaps)  
[10.3机器ID](#machine-id)  
[10.4 MAC地址欺骗](#mac-address-spoofing)  
[10.5时间攻击](#time-attacks)  
[10.5.1 ICMP时间戳](#icmp-timestamps)  
[10.5.2 TCP时间戳](#tcp-timestamps)  
[10.5.3 TCP初始序列号](#tcp-isns)  
[10.5.4时间同步](#time-synchronization)  
[10.6击键指纹](#keystroke-fingerprinting)  
  
[11.文件权限](#file-permissions)  
[11.1 setuid / setgid](#setuid)  
[11.2 umask](#umask)  
  
[12.核心转储](#core-dumps)  
[12.1 sysctl](#core-dumps-sysctl)  
[12.2 systemd](#core-dumps-systemd)  
[12.3 ulimit](#core-dumps-ulimit)  
[12.4 setuid进程](#core-dumps-setuid)  
  
[13.交换](#swap)  
  
[14. PAM](#pam)  
  
[15.微码更新](#microcode)  
  
[16。 IPv6隐私扩展](#ipv6-privacy)  
[16.1 NetworkManager](#ipv6-networkmanager)  
[16.2 systemd-networkd](#ipv6-systemd-networkd)  
  
[17.分区和挂载选项](#partitioning)  
  
[18.熵](#entropy)  
[18.1其他熵源](#additional-entropy-sources)  
[18.2 RDRAND](#rdrand)  
  
[19.以root](#editing-as-root) [身份](#rdrand)[编辑文件](#editing-as-root)  
  
[20.特定于发行版的强化](#distro-specific)  
[20.1 HTTPS软件包管理器镜像](#https-mirrors)  
[20.2 APT seccomp-bpf](#apt-seccomp-bpf)  
  
[21.物理安全性](#physical-security)  
[21.1加密](#encryption)  
[21.2 BIOS / UEFI强化](#bios-uefi)  
[21.3引导程序密码](#bootloader-passwords)  
[21.3.1 GRUB](#grub)  
[21.3.2 Syslinux](#syslinux)  
[21.3.3 systemd-boot](#systemd-boot)  
[21.4经过验证的引导](#verified-boot)  
[21.5 USB](#usbs)  
[21.6 DMA攻击](#dma-attacks)  
[21.7冷启动攻击](#cold-boot-attacks)  
  
[22.最佳做法](#best-practices)  

[1.选择正确的Linux发行版](#choosing-the-right-distro)
---------------------------------------------

选择一个好的Linux发行版有很多因素。

*   避免分发会冻结程序包的发行版，因为[它们通常远远落后于安全更新](../linux.html#debian)。
*   使用与[systemd](https://systemd.io/)以外的init系统一起使用的发行[版](https://systemd.io/)。systemd包含许多不必要的攻击面；[它尝试做的事情远远](https://ewontfix.com/14/)超出了[必要，](https://ewontfix.com/14/)并且超出了初始化系统应做的事情。初始化系统不需要很多代码行即可正常运行。
*   使用[musl](https://musl.libc.org/)作为默认的C库。musl专注于最小化，这会[导致很小的攻击面，](https://github.com/NixOS/nixpkgs/issues/90147#issuecomment-643473182)而其他C库（例如glibc）则过于复杂且容易产生漏洞。例如，与 [musl中极少数](https://www.cvedetails.com/vulnerability-list/vendor_id-16859/product_id-39652/Musl-libc-Musl.html) [漏洞](https://www.cvedetails.com/vulnerability-list/vendor_id-72/product_id-767/GNU-Glibc.html)相比[，glibc中已公开披露了一百多个漏洞](https://www.cvedetails.com/vulnerability-list/vendor_id-72/product_id-767/GNU-Glibc.html)。尽管仅靠计算CVE本身通常是不准确的统计信息，但有时可以用来代表诸如此类的问题。musl还具有[不错的漏洞利用缓解措施](https://www.dustri.org/b/security-features-of-musl.html)，尤其是其 [新的强化内存分配器](https://www.openwall.com/lists/musl/2020/05/13/1)。[](https://www.cvedetails.com/vulnerability-list/vendor_id-16859/product_id-39652/Musl-libc-Musl.html)[](https://www.dustri.org/b/security-features-of-musl.html)[](https://www.openwall.com/lists/musl/2020/05/13/1)
*   最好使用默认情况下使用[LibreSSL](https://www.libressl.org/)而不是 [OpenSSL的发行版](https://www.openssl.org/)。[OpenSSL包含大量完全不必要的攻击面，并且遵循不良的安全做法](https://arstechnica.com/information-technology/2014/04/openssl-code-beyond-repair-claims-creator-of-libressl-fork/)。例如，它仍然保持OS / 2和VMS支持-已有数十年历史的古老操作系统。这些令人讨厌的安全做法导致了可怕的[Heartbleed漏洞](https://en.wikipedia.org/wiki/Heartbleed)。LibreSSL是OpenBSD团队提供的OpenSSL的分支，它采用了 [出色的编程实践](https://en.wikipedia.org/wiki/LibreSSL#Changes)并[消除了许多攻击面](https://opensslrampage.org/)。在LibreSSL成立的第一年，[它缓解了许多漏洞，其中包括一些严重程度很高的漏洞](https://www.openbsd.org/papers/libtls-fsec-2015/mgp00005.html)。

用作强化操作系统基础的最佳发行版是[Gentoo Linux，](https://www.gentoo.org/)因为它可以让您精确地配置系统，以达到理想的效果，这将非常有用，尤其是当我们在后面的章节中使用[更安全的编译标志时](#hardened-compilation-flags)。指南。  
  
但是，由于Gentoo具有很大的可用性缺陷，因此对于许多人来说可能并不可行。在这种情况下，[Void Linux](https://voidlinux.org/)的Musl构建是一个很好的折衷方案。

[2.内核](#kernel)
---------------

内核是操作系统的核心，不幸的是很容易受到攻击。正如Brad Spengler曾经说过的那样，[可以将其视为系统上最大，最易受攻击的setuid根二进制文件](https://grsecurity.net/huawei_hksp_introduces_trivially_exploitable_vulnerability)。因此，对内核进行尽可能多的硬化非常重要。

### [2.1稳定版与LTS内核](#stable-vs-lts)

Linux内核以两种主要形式发布：稳定和长期支持（LTS）。稳定版本是较新的版本，而LTS发行版本是较老的稳定版本，长期以来一直受支持。 [选择上述任何一个发行版本都有许多后果](https://www.grsecurity.net/the_truth_about_linux_4_6)。  
  
Linux内核[未使用CVE正确识别安全漏洞](https://grsecurity.net/reports_of_cves_death_greatly_exaggerated)。这意味着大多数安全漏洞的修复程序不能向后移植到LTS内核。但是稳定版本包含到目前为止进行的所有安全修复。  
  
但是，有了这些修复程序，稳定的内核将包含更多新功能，因此大大增加了内核的攻击面并引入了大量新错误。相反，LTS内核的受攻击面较小，因为这些功能没有被不断添加。  
  
此外，稳定的内核包括更新的强化功能，以减轻LTS内核没有的某些利用。此类功能的一些示例是[Lockdown LSM](https://mjg59.dreamwidth.org/55105.html)和[STACKLEAK GCC插件](https://a13xp0p0v.github.io/2018/11/04/stackleak.html)。  
  
总之，选择稳定或LTS内核时需要权衡。LTS内核具有较少的强化功能，并且并非当时所有的公共错误修复都已向后移植，但是它通常具有较少的攻击面，并可能引入未知错误的可能性较小。稳定的内核具有更多的强化功能，并且包括所有已知的错误修复，但它也具有更多的攻击面以及引入更多未知错误的机会更大。最后，由于攻击面小得多，因此最好使用较新的LTS分支（如4.19内核）。

### [2.2系统](#sysctl)

Sysctl是允许用户配置某些内核设置并启用各种安全功能或禁用危险功能以减少攻击面的工具。要临时更改设置，您可以执行：

```
sysctl -w $可调= $值
```

要永久更改系统，可以根据Linux发行版添加要更改的系统`/etc/sysctl.conf`或其中的相应文件 `/etc/sysctl.d`。  
  
以下是您应更改的建议sysctl设置。

#### [2.2.1内核自我保护](#sysctl-kernel)

```
kernel.kptr_restrict = 2
```

内核指针指向内核内存中的特定位置。 [这些对于利用内核非常有用，](https://kernsec.org/wiki/index.php/Bug_Classes/Kernel_pointer_leak)但是默认情况下不会隐藏内核指针-例如，通过读取的内容，很容易发现它们`/proc/kallsyms`。此设置旨在减轻内核指​​针泄漏。另外，您可以设置 `kernel.kptr_restrict=1`为仅从进程中隐藏内核指针，而没有`CAP_SYSLOG` [功能](#capabilities)。

```
kernel.dmesg_restrict = 1
```

[dmesg](https://en.wikipedia.org/wiki/Dmesg)是内核日志。它公开了大量有用的内核调试信息，但这通常会泄漏敏感信息，例如内核指针。更改上述sysctl会将内核日志限制为该`CAP_SYSLOG` [功能](#capabilities)。

```
kernel.printk = 3  3  3  3
```

尽管值为`dmesg_restrict`，引导过程中内核日志仍将显示在控制台中。在启动过程中能够记录屏幕的恶意软件可能会滥用此功能以获得更高的特权。此选项可防止这些信息泄漏。必须将其与[下面描述的](#boot-kernel)某些引导参数结合使用才能完全有效。

```
kernel.unprivileged_bpf_disabled = 1 
net.core.bpf_jit_harden = 2
```

[eBPF暴露出很大的攻击面](../linux.html#kernel)。因此，必须加以限制。这些系统将eBPF限制为该`CAP_BPF` [功能](#capabilities)（`CAP_SYS_ADMIN`在5.8之前的内核版本上），并启用JIT强化技术，例如[恒定盲化](https://github.com/torvalds/linux/blob/9e4b0d55d84a66dbfede56890501dc96e696059c/include/linux/filter.h#L1039-L1070)。

```
dev.tty.ldisc_autoload = 0
```

这[限制了装载TTY线路规程](https://lkml.org/lkml/2019/4/15/890)的`CAP_SYS_MODULE` [能力](#capabilities)，以防止无特权的攻击者加载与脆弱线路规程`TIOCSETD` 其中的ioctl[已经在之前的一些漏洞被滥用](https://a13xp0p0v.github.io/2017/03/24/CVE-2017-2636.html)。

```
vm.unprivileged_userfaultfd = 0
```

该`[userfaultfd()](https://man7.org/linux/man-pages/man2/userfaultfd.2.html)`系统调用是[经常](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=cefdca0a86be517bc390fc4541e3674b8e7803b0) [被滥用](https://duasynt.com/blog/linux-kernel-heap-spray)利用释放后使用漏洞自由。因此，该sysctl用于限制此syscall的`CAP_SYS_PTRACE` [功能](#capabilities)。

```
kernel.kexec_load_disabled = 1
```

[kexec是一个系统调用，用于在运行时引导另一个内核](https://en.wikipedia.org/wiki/Kexec)。可以滥用此功能来加载恶意内核并在内核模式下获得任意代码执行，因此该sysctl将其禁用。

```
kernel.sysrq = 4
```

该[SysRq键](https://www.kernel.org/doc/html/latest/admin-guide/sysrq.html)暴露了很多的潜在危险的调试功能，未经授权的用户。与通常的假设相反，SysRq不仅是物理攻击的问题， [而且还可以远程触发](https://github.com/xairy/unlockdown)。该sysctl的值使其可以使用户只能使用 [安全注意密钥](https://www.kernel.org/doc/Documentation/SAK.txt)，这对于[安全地访问root](#accessing-root-securely)是必不可少的。或者，您可以简单地将值设置为，`0`以完全禁用SysRq。

```
kernel.unprivileged_userns_clone = 0
```

用户命名空间是在内核中的功能，旨在改善沙盒并使其易于然而对于未经授权的用户访问，[此功能公开权限提升显著内核攻击面，](../linux.html#kernel)所以这次的sysctl限制用户的命名空间的使用`CAP_SYS_ADMIN` [能力](#capabilities)。对于无特权的沙箱，建议使用具有很少攻击面的setuid二进制文件，以最大程度地减少特权升级的可能性。[沙箱部分](#application-sandboxing)将进一步讨论此主题。  
  
请注意，尽管该sysctl仅在某些Linux发行版中存在，因为它需要内核补丁。如果您的内核不包含此修补程序，则可以通过设置来完全禁用用户名称空间（包括root用户）`user.max_user_namespaces=0`。

```
kernel.perf_event_paranoid = 3
```

性能事件会[增加大量内核攻击面，并导致大量漏洞](https://lore.kernel.org/kernel-hardening/1469630746-32279-1-git-send-email-jeffv@google.com/)。此sysctl将性能事件的所有使用限制为 `CAP_PERFMON` [功能](#capabilities)（`CAP_SYS_ADMIN`在5.8之前的内核版本上）。  
  
请注意，此sysctl还需要仅在某些发行版中可用的内核补丁。否则，此设置相当于 `kernel.perf_event_paranoid=2`其 [只限制这个功能的一个子集](https://www.kernel.org/doc/html/latest/admin-guide/perf-security.html#unprivileged-users)。

#### [2.2.2网络](#sysctl-network)

```
net.ipv4.tcp_syncookies = 1
```

这有助于防止[SYN泛洪攻击](https://en.wikipedia.org/wiki/SYN_flood)，这种[攻击](https://en.wikipedia.org/wiki/SYN_flood)是拒绝服务攻击的一种形式，在这种攻击中，攻击者发送大量虚假的SYN请求，以尝试消耗足够的资源以使系统对合法流量不响应。

```
net.ipv4.tcp_rfc1337 = 1
```

这通过丢弃处于时间等待状态的套接字的RST数据包来防止[时间等待暗杀](https://tools.ietf.org/html/rfc1337)。

```
net.ipv4.conf.all.rp_filter = 1 
net.ipv4.conf.default.rp_filter = 1
```

这些启用了从机器所有接口接收到的数据包的源验证。这样可以防止[IP欺骗](https://en.wikipedia.org/wiki/IP_address_spoofing)，其中攻击者发送带有欺诈IP地址的数据包。

```
net.ipv4.conf.all.accept_redirects = 0 
net.ipv4.conf.default.accept_redirects = 0 
net.ipv4.conf.all.secure_redirects = 0 
net.ipv4.conf.default.secure_redirects = 0 
net.ipv6.conf。 all.accept_redirects = 0 
net.ipv6.conf.default.accept_redirects = 0 
net.ipv4.conf.all.send_redirects = 0 
net.ipv4.conf.default.send_redirects = 0
```

这些禁用ICMP重定向接受和发送，以 [防止中间人攻击](https://askubuntu.com/questions/118273/what-are-icmp-redirects-and-should-they-be-blocked)并最大程度地减少信息泄露。

```
net.ipv4.icmp_echo_ignore_all = 1
```

此设置使您的系统忽略所有ICMP请求，以避免[Smurf攻击](https://en.wikipedia.org/wiki/Smurf_attack)，使设备更难以在网络上枚举并[通过ICMP时间戳防止时钟指纹](#icmp-timestamps)。

```
net.ipv4.conf.all.accept_source_route = 0 
net.ipv4.conf.default.accept_source_route = 0 
net.ipv6.conf.all.accept_source_route = 0 
net.ipv6.conf.default.accept_source_route = 0
```

[源路由是一种允许用户重定向网络流量的机制](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security_guide/sect-security_guide-server_security-disable-source-routing)。由于这可用于执行中间人攻击，在中间人攻击中，出于恶意目的将流量重定向，因此上述设置禁用了此功能。

```
net.ipv6.conf.all.accept_ra = 0 
net.ipv6.conf.default.accept_ra = 0
```

恶意的IPv6路由器广告[可能会导致中间人攻击，](https://tools.cisco.com/security/center/resources/ipv6_first_hop)因此应将其禁用。

```
net.ipv4.tcp_sack = 0 
net.ipv4.tcp_dsack = 0 
net.ipv4.tcp_fack = 0
```

这将禁用[TCP SACK](https://tools.ietf.org/html/rfc2018)。SACK[通常被利用，](https://github.com/Netflix/security-bulletins/blob/master/advisories/third-party/2019-001.md) 并且[在许多情况下](https://serverfault.com/questions/10955/when-to-turn-tcp-sack-off)是[不必要的，](https://serverfault.com/questions/10955/when-to-turn-tcp-sack-off)因此如果您不需要它，则应将其禁用。

#### [2.2.3用户空间](#sysctl-userspace)

```
kernel.yama.ptrace_scope = 2
```

[ptrace是一个系统调用，它允许程序更改和检查另一个正在运行的进程](https://www.kernel.org/doc/html/latest/admin-guide/LSM/Yama.html)，从而使攻击者可以轻易地修改其他正在运行的程序的内存。这将ptrace的使用限制为仅具有`CAP_SYS_PTRACE` [能力的](#capabilities)进程。或者，将sysctl设置为`3`完全禁用ptrace。

```
vm.mmap_rnd_bits = 32 
vm.mmap_rnd_compat_bits = 16
```

[ASLR](https://en.wikipedia.org/wiki/Address_space_layout_randomization)是一种常见的漏洞利用缓解措施，它可以使进程的关键部分在内存中的位置随机化。这可能会使各种各样的漏洞利用更困难，因为它们首先需要信息泄漏。上述设置增加了用于mmap ASLR的熵的位数，从而提高了其有效性。  
  
这些系统值必须相对于CPU体系结构进行设置。以上值与x86兼容，但其他体系结构可能有所不同。

```
fs.protected_symlinks = 1 
fs.protected_hardlinks = 1
```

仅当在可全局写入的粘性目录之外，当符号链接和关注者的所有者匹配或目录所有者与符号链接的所有者匹配时，才允许遵循符号链接。这还可以防止没有对源文件的读/写访问权限的用户创建硬链接。这两者都阻止了许多常见的[TOCTOU比赛](https://en.wikipedia.org/wiki/Time-of-check_to_time-of-use)。

```
fs.protected_fifos = 2 
fs.protected_regular = 2
```

这些[阻止了在可能由攻击者控制的环境（](https://github.com/torvalds/linux/commit/30aba6656f61ed44cba445a3c0d38b296fa9e8f5)例如，可写入世界的目录）中[创建文件，](https://github.com/torvalds/linux/commit/30aba6656f61ed44cba445a3c0d38b296fa9e8f5)从而使数据欺骗攻击更加困难。

### [2.3引导参数](#boot-parameters)

引导参数在引导时使用引导加载程序将设置传递给内核。类似于sysctl，某些设置可用于提高安全性。引导加载程序通常在引导参数设置方式上有所不同。下面列出了一些示例，但是您应该研究特定引导加载程序的必需步骤。  
  
如果使用GRUB作为引导程序，请编辑`/etc/default/grub`并将参数添加到该 `GRUB_CMDLINE_LINUX_DEFAULT=`行。  
  
如果使用Syslinux，请编辑`/boot/syslinux/syslinux.cfg`并将其添加到该`APPEND`行中。  
  
如果使用systemd-boot，请编辑您的加载程序条目并将其附加到该`linux`行的末尾。  
  
建议使用以下设置以提高安全性。

#### [2.3.1内核自我保护](#boot-kernel)

```
slab_nomerge
```

这将禁用平板合并，从而通过[防止覆盖合并的缓存中的对象](https://www.openwall.com/lists/kernel-hardening/2017/06/19/33)并[使其更难以影响平板缓存的布局](https://www.openwall.com/lists/kernel-hardening/2017/06/20/10)，从而大大增加了堆利用的难度。

```
slub_debug = FZ
```

这些启用[健全性检查（F）和重新分区（Z）](https://www.kernel.org/doc/html/latest/vm/slub.html)。完整性检查会添加各种检查，以防止某些平板操作中的损坏。Redzoning会在平板周围添加额外的区域，以检测平板何时被覆盖超过其实际大小，从而有助于检测溢出。

```
init_on_alloc = 1 init_on_free = 1
```

这样可以 [在分配和空闲时间期间将内存清零，](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=6471384af2a6530696fc0203bafe4de41a23c9ef)这可以帮助减轻使用后使用的漏洞并清除内存中的敏感信息。  
  
如果您的内核版本低于5.3，则这些选项不存在。而是将“ P”附加到上述`slub_debug` 选项以获取`slub_debug=FZP`并添加`page_poison=1`。由于它们实际上是一种调试功能，恰好具有一些安全性，因此它们在释放时提供的内存擦除形式较弱。

```
page_alloc.shuffle = 1
```

此选项使 [页面分配器空闲列表随机化](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=e900a918b0984ec8f2eb150b8477a47b75d17692)，从而通过降低页面分配的可预测性来提高安全性。这也_提高了_ 性能。

```
pti =开
```

这将启用[内核页表隔离](https://en.wikipedia.org/wiki/Kernel_page-table_isolation)，从而减轻崩溃并防止某些KASLR绕过。

```
vsyscall =无
```

这将禁用[vsyscall，](https://lwn.net/Articles/446528/)因为它们已过时且已被[vDSO](https://en.wikipedia.org/wiki/VDSO)取代。vsyscall也在内存中的固定地址上，使其成为ROP攻击的潜在目标。

```
debugfs =关闭
```

这将禁用debugfs，它会[公开许多有关内核的敏感信息](https://lkml.org/lkml/2020/7/16/122)。

```
哎呀=恐慌
```

有时，某些内核漏洞利用会导致所谓的[“哎呀”](https://en.wikipedia.org/wiki/Linux_kernel_oops)。此参数将导致内核对此类事件感到恐慌，从而防止这些攻击。但是，有时错误的驱动程序会导致无害的操作，这会导致系统崩溃，这意味着此引导参数只能在某些硬件上使用。

```
module.sig_enforce = 1
```

这仅允许加载已用有效密钥签名的内核模块，这通过使加载恶意内核模块更加困难来提高安全性。这可以防止加载所有树外内核模块（包括DKMS模块），[除非您已对其进行签名](https://www.kernel.org/doc/html/latest/admin-guide/module-signing.html)，这意味着诸如VirtualBox或Nvidia驱动程序之类的模块可能不可用，尽管根据您的设置可能并不重要。

```
锁定=机密性
```

该[内核锁定LSM](https://mjg59.dreamwidth.org/55105.html)可以消除许多方法是用户空间的代码可能滥用升级内核特权和提取敏感信息。为了在用户空间和内核之间实现清晰的安全边界，此LSM是必需的。以上选项在机密模式（最严格的选项）中启用此功能。这意味着`module.sig_enforce=1`。

```
mce = 0
```

这导致内核对ECC内存中无法利用的不可纠正的错误感到恐慌。对于没有ECC内存的系统，这是不必要的。

```
安静的日志级别= 0
```

这些参数防止引导期间信息泄漏和必须在组合与使用`kernel.printk`的sysctl[上述记载](#sysctl-kernel)。

#### [2.3.2 CPU缓解](#cpu-mitigations)

最好启用适用于您的CPU的所有CPU缓解措施，以确保您不受已知漏洞的影响。这是启用所有内置缓解措施的列表：

```
spectre_v2 = on spec_store_bypass_disable = on tsx = off tsx_async_abort = full，nosmt mds = full，nosmt l1tf = full，force nosmt = force kvm.nx_huge_pages = force
```

您必须研究系统受其影响的CPU漏洞，并相应地选择上述缓解措施。请记住，您将需要[安装微代码更新](#microcode)，以完全免受这些漏洞的影响。所有这些都可能导致性能显着下降。

#### [2.3.3结果](#result)

如果遵循了以上所有建议（不包括特定的CPU缓解措施），则将具有：

```
slab_nomerge slub_debug = FZ init_on_alloc = 1 init_on_free = 1 page_alloc.shuffle = 1 pti = on vsyscall = none debugfs = off oops = panic module.sig_enforce = 1 lockdown = confidentiality mce = 0安静的日志级别= 0
```

如果将GRUB用作引导加载程序，则 可能需要重新[生成GRUB配置文件](#regenerate-grub-config)才能应用这些[文件](#regenerate-grub-config)。

### [2.4 Hidepid](#hidepid)

`/proc`是一个伪文件系统，其中包含有关系统当前正在运行的所有进程的信息。默认情况下，所有用户都可以访问此程序，这可能使攻击者可以窥探其他进程。要允许用户仅查看自己的进程，而不能查看其他用户的进程，则必须`/proc` 使用`hidepid=2,gid=proc`安装选项进行安装。`gid=proc`使该`proc`组免于使用此功能，因此您可以将特定用户或进程列入白名单。添加这些安装选项的一种方法是编辑`/etc/fstab`并添加：

```
proc / proc proc nosuid，nodev，noexec，hidepid = 2，gid = proc 0  0
```

systemd-logind仍然需要查看其他用户的进程，因此，要使用户会话在systemd系统上正常工作，必须创建 `/etc/systemd/system/systemd-logind.service.d/hidepid.conf`并添加：

```
[服务]
补充组= proc
```

### [2.5减少内核攻击面](#kernel-attack-surface-reduction)

最好禁用并非绝对必要的任何功能，以最大程度地减少潜在的内核攻击面。这些功能不必一定很危险，它们可以只是被删除以减少攻击面的良性代码。切勿禁用您不了解的随机事物。以下是一些可能有用的示例，具体取决于您的设置。

#### [2.5.1引导参数](#kasr-boot-parameters)

引导参数通常可以用来减少攻击面。这样的例子之一是：

```
ipv6.disable = 1
```

这将禁用整个IPv6堆栈，如果您尚未迁移到该堆栈，则可能不需要该堆栈。如果使用的是IPv6，请不要使用此引导参数。

#### [2.5.2将内核模块列入黑名单](#kasr-kernel-modules)

内核允许无特权的用户通过模块自动加载来间接导致某些模块被加载。这使攻击者可以自动加载易受攻击的模块，然后加以利用。一个这样的示例是[CVE-2017-6074](https://github.com/xairy/kernel-exploits/tree/master/CVE-2017-6074)，其中攻击者可以通过启动DCCP连接来触发DCCP内核模块的加载，然后利用该内核模块中的漏洞。  
  
可以通过将文件插入到特定的内核模块中来将其列入黑名单，`/etc/modprobe.d`其中包含有关将哪些内核模块列入黑名单的说明。  
  
该`install`参数`modprobe`指示运行特定命令，而不是像往常一样加载模块。`/bin/false` 是只返回的命令`1`从本质上讲什么也没做 两者都告诉内核运行`/bin/false`而不是加载模块，这将防止攻击者利用该模块。以下是最有可能不需要的内核模块：

```
安装dccp / bin /错误
安装sctp / bin /错误
安装rds / bin /错误
安装tipc / bin /错误
安装n-hdlc / bin /错误
安装ax25 / bin /错误
安装netrom / bin /错误
安装x25 / bin / false
安装rose / bin / false
安装decnet / bin/错误
安装econet / bin /错误
安装af_802154 / bin /错误
安装ipx / bin /错误
安装appletalk / bin /错误
安装psnap / bin /错误
安装p8023 / bin /错误
安装p8022 / bin /错误
安装can / bin / false
安装atm / bin / false
```

特别是模糊的网络协议会增加大量的远程攻击面。此黑名单：

*   DCCP —数据报拥塞控制协议
*   SCTP —流控制传输协议
*   RDS —可靠的数据报套接字
*   TIPC —透明的进程间通信
*   HDLC —高级数据链路控制
*   AX25 —业余X.25
*   网罗
*   X25
*   玫瑰
*   DECnet
*   经济网
*   af_802154 — IEEE 802.15.4
*   IPX —互联网数据包交换
*   苹果对话
*   PSNAP —子网访问协议
*   p8023 — Novell原始IEEE 802.3
*   p8022 — IEEE 802.2
*   CAN —控制器局域网
*   自动取款机

```
安装cramfs / bin / false
安装freevxfs / bin / false
安装jffs2 / bin / false
安装hfs / bin / false
安装hfsplus / bin / false
安装squashfs / bin / false
安装udf / bin / false
```

这将各种稀有文件系统列入黑名单。

```
安装cifs / bin / true
安装nfs / bin / true
安装nfsv3 / bin / true
安装nfsv4 / bin / true
安装gfs2 / bin / true
```

如果不使用网络文件系统，也可以将其列入黑名单。

```
安装生动/ bin / false
```

在[生动的驱动程序](https://www.kernel.org/doc/html/v4.12/media/v4l-drivers/vivid.html)仅用于测试目的有用，[一直权限提升漏洞的原因](https://www.openwall.com/lists/oss-security/2019/11/02/1)，因此应该被禁用。

```
安装蓝牙/ bin / false
安装btusb / bin / false
```

这将禁用[具有安全问题历史记录的](https://en.wikipedia.org/wiki/Bluetooth#History_of_security_concerns)蓝牙。

```
安装uvcvideo / bin / false
```

这会禁用网络摄像头，以防止其被用来监视您。  
  
您也可以将麦克风模块列入黑名单，但是这可能因系统而异。要查找模块的名称，请查看`/proc/asound/modules` 并将其列入黑名单。例如，一个这样的模块是`snd_hda_intel`。  
  
请注意，尽管有时麦克风的内核模块与扬声器的模块相同。这意味着像这样禁用麦克风也可能会无意中禁用任何扬声器。虽然，[扬声器也有可能变成麦克风，](https://arxiv.org/ftp/arxiv/papers/1611/1611.07350.pdf)所以这不一定是负面的结果。  
  
最好从物理上删除这些设备，或者至少在BIOS / UEFI中禁用它们。禁用内核模块并不那么有效。

#### [2.5.3 rfkill](#kasr-rfkill)

可以将无线设备列入黑名单，`rfkill`以进一步减少远程攻击面。要将所有无线设备列入黑名单，请执行：

```
rfkill全部阻止
```

WiFi可以通过以下方式解锁：

```
rfkill取消阻止wifi
```

在使用systemd的系统上，[rfkill在所有会话](https://www.freedesktop.org/software/systemd/man/systemd-rfkill.service.html)中均[保持](https://www.freedesktop.org/software/systemd/man/systemd-rfkill.service.html)不变，但是，在使用其他init系统的系统上，您可能必须创建一个init脚本以在引导时执行这些命令。

### [2.6其他内核指针泄漏](#other-kernel-pointer-leaks)

前面的章节阻止了一些内核指针的泄漏，但是[还有更多的](https://github.com/jonoberheide/ksymhunter)泄漏。  
  
在文件系统上，中存在内核映像和System.map文件`/boot`。`/usr/src`和 `/{,usr/}lib/modules`目录中还包含其他敏感的内核信息。[您应该限制这些目录的文件许可权，以使其只能由root用户读取](#file-permissions)。您还应该删除System.map文件，因为除高级调试外，它们都不需要。  
  
此外，某些日志记录守护程序（例如systemd的日志记录守护程序）`journalctl`包括可用于绕过上述`dmesg_restrict`保护的内核日志。从中删除用户`adm` 组通常足以撤销对这些日志的访问：

```
gpasswd -d adm $ user
```

### [2.7限制对sysfs的访问](#restricting-sysfs)

[sysfs](https://www.kernel.org/doc/html/latest/filesystems/sysfs.html)是一个伪文件系统，它提供大量的内核和硬件信息。通常安装在`/sys`。sysfs导致[大量信息泄漏，尤其是内核指针泄漏](https://www.openwall.com/lists/kernel-hardening/2017/10/05/5)。Whonix的[security-misc软件包](https://github.com/Whonix/security-misc)包括[hide-hardware-info脚本](https://github.com/Whonix/security-misc/blob/master/usr/lib/security-misc/hide-hardware-info)，该[脚本](https://github.com/Whonix/security-misc/blob/master/usr/lib/security-misc/hide-hardware-info)限制访问此目录以及一些`/proc`试图隐藏潜在硬件标识符并防止内核指针泄漏的尝试。该脚本是可配置的，并允许[基于组将特定的应用程序列入白名单](https://www.whonix.org/wiki/Whonix-Workstation_Security_Hardening#Restrict_Hardware_Information_to_Root)。建议应用此方法，并使其在启动时使用init脚本执行。例如，[这是一个系统服务](https://github.com/Whonix/security-misc/blob/master/lib/systemd/system/hide-hardware-info.service)。  
  
为了使基本功能在使用systemd的系统上运行，必须将一些系统服务列入白名单。可以通过创建`/etc/systemd/system/user@.service.d/sysfs.conf` 和添加以下内容来完成：

```
[服务]
补充组= sysfs
```

虽然这不能解决所有问题。许多应用程序可能仍会中断，您需要将其正确列入白名单。

### [2.8 linux强化](#linux-hardened)

某些发行版（例如Arch Linux）包括[强化的内核程序包](https://www.archlinux.org/packages/extra/x86_64/linux-hardened/)。它包含许多强化补丁程序和更注重安全性的内核配置。如果可能，建议安装它。

### [2.9安全](#grsecurity)

[Grsecurity](https://grsecurity.net/)是一组内核修补程序，可以大大提高内核安全性。这些补丁曾经免费提供，但是[现在已经可以商业购买，必须购买](https://grsecurity.net/passing_the_baton)。如果可用，则强烈建议您获取它。Grsecurity提供最新的内核和用户空间保护。

### [2.10 Linux内核运行时防护](#lkrg)

[Linux Kernel Runtime Guard](https://www.openwall.com/lkrg/)（LKRG）是一个内核模块，可确保运行时内核的完整性并检测漏洞。它可以杀死整个类别的内核漏洞；但这不是一个完美的缓解方法，因为[LKRG在设计上可以绕开](https://github.com/milabs/lkrg-bypass)。它仅适用于现成的恶意软件。但是，虽然可能性不大，但LKRG本身可能会像其他任何内核模块一样公开新的漏洞。

### [2.11内核自编译](#kernel-self-compilation)

建议编译您自己的内核，同时启用尽可能少的内核模块和尽可能多的安全功能，以将内核的受攻击面保持在绝对最低限度。  
  
此外，如上所述，应用内核强化补丁，例如[linux-hardened](https://github.com/anthraxx/linux-hardened)或[grsecurity](https://grsecurity.net/)。  
  
发行版编译的内核还具有公共内核指针/符号，这对于漏洞利用非常有用。编译自己的内核会给你独特的内核符号，它与沿`kptr_restrict`，`dmesg_restrict`以及对核泄漏的指针其他加固，将使其大大更难攻击者创建依赖内核指针知识漏洞。  
  
一旦完成内核配置开发， 您就可以从Whonix的[强化内核中](https://www.whonix.org/wiki/Hardened-kernel)汲取灵感或使用它。

[3.强制访问控制](#mac)
----------------

强制访问控制（MAC）系统对程序可以访问的内容进行细粒度的控制。这意味着您的浏览器将无权访问您的整个主目录或类似目录。  
  
最常用的MAC系统是SELinux和AppArmor。SELinux比AppArmor安全得多，因为它的粒度更细。例如，它是基于inode而不是基于path的，允许强制执行 [明显更严格的限制](https://selinuxproject.org/page/ObjectClassesPerms)，可以[过滤内核ioctl](https://kernsec.org/files/lss2015/vanderstoep.pdf)等。不幸的是，这是以难以使用和学习为代价的，因此某些人可能会首选AppArmor。  
  
要在内核中启用AppArmor，必须设置以下[引导参数](#boot-parameters)：

```
apparmor = 1安全性= apparmor
```

要启用SELinux，请设置以下参数：

```
selinux = 1安全性= selinux
```

请记住，仅启用MAC系统本身并不能神奇地提高安全性。您必须制定严格的政策才能充分利用它。例如，要创建AppArmor配置文件，请执行：

```
aa-genprof $ path_to_program
```

打开程序，然后像往常一样开始使用它。AppArmor将检测需要访问哪些文件，并将它们添加到配置文件中（如果您选择）。但是，仅凭这一点不足以提供高质量的配置文件。请参阅[AppArmor文档](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation)以获取更多详细信息。  
  
如果您想更进一步，则可以通过实施initramfs挂钩来设置完整的系统MAC策略，该策略限制每个单个用户空间进程，该挂钩对init系统强制实施MAC策略。这就是[Android使用SELinux的方式](https://source.android.com/security/selinux)，以及[Whonix将来](https://github.com/Whonix/apparmor-profile-everything)如何[使用AppArmor](https://github.com/Whonix/apparmor-profile-everything)。这对于加强实施[最小特权原则的](https://en.wikipedia.org/wiki/Principle_of_least_privilege)强大安全模型是必要的。

[4.沙箱](#sandboxing)
-------------------

### [4.1应用沙箱](#application-sandboxing)

沙箱可让您在隔离的环境中运行程序，该环境对系统的其余部分具有有限的访问权限或完全没有访问权限。您可以使用它们来保护应用程序安全或运行不受信任的程序。  
  
建议与[AppArmor或SELinux](#mac)一起在单独的用户帐户中使用[bubblewrap](https://github.com/containers/bubblewrap)来沙盒程序。您也可以考虑改用[gVisor](#gvisor)，它的优点是为每个来宾提供了自己的内核。 这些方法中的任何一个都可以用来创建一个功能强大的沙箱，并且只暴露最小的攻击面。如果您不想自己创建沙箱，请在完成后考虑使用Whonix的[sandbox-app-launcher](https://www.whonix.org/wiki/Sandbox-app-launcher)。[你不应该使用Firejail](../linux.html#firejail)[](#mac)[](#gvisor)  
  
[](https://www.whonix.org/wiki/Sandbox-app-launcher)[](../linux.html#firejail)。  
  
诸如[Docker](https://www.docker.com/)和[LXC之](https://linuxcontainers.org/)类的容器解决方案通常被误导为沙盒形式。这些太宽容了，无法支持各种应用程序，因此不能认为它们是强大的应用程序沙箱。

### [4.2常见的沙箱逃生](#common-sandbox-escapes)

#### [4.2.1 PulseAudio](#pulseaudio)

[PulseAudio](https://www.freedesktop.org/wiki/Software/PulseAudio/)是一种常见的声音服务器，但 [编写时并未考虑隔离或沙盒操作](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/Developer/AccessControl/)，因此使其成为重复出现的沙盒逃逸漏洞。为防止这种情况，建议您从沙箱中阻止对PulseAudio的访问，或者从系统中完全卸载它。您可以改用[标准ALSA实用程序](https://alsa-project.org/wiki/Main_Page)。

#### [4.2.2 D-总线](#d-bus)

[D-Bus](https://www.freedesktop.org/wiki/Software/dbus/)是台式机Linux上最流行的进程间通信形式，但它也是沙箱转义的另一种常见途径，因为它允许与服务自由交互。[Firejail中](https://github.com/netblue30/firejail/issues/796)就是此类漏洞的一个例子。您应该从沙箱中阻止对D-Bus的访问，或者[使用细粒度规则通过MAC进行调解](https://manpages.debian.org/buster/apparmor/apparmor.d.5.en.html#DBus_rules)。

#### [4.2.3 GUI隔离](#gui-isolation)

[任何Xorg窗口都可以访问另一个窗口](https://theinvisiblethings.blogspot.com/2011/04/linux-security-circus-on-gui-isolation.html)。这允许简单的键盘记录或屏幕截图程序，甚至可以记录诸如root密码之类的内容。您可以使用嵌套的X11服务器（例如[Xpra](https://www.xpra.org/)或[Xephyr](https://freedesktop.org/wiki/Software/Xephyr/) 和bubblewrap）将Xorg窗口沙箱化。默认情况下，  
  
[Wayland](https://wayland.freedesktop.org/)将窗口彼此隔离，这将是一个比Xorg更好的选择，尽管Wayland可能不如Xorg普遍可用，因为它处于开发初期。

#### [4.2.4 ptrace](#ptrace)

正如[前面所讨论的](#sysctl-userspace)，ptrace的是一个系统调用，可能会被滥用到运行沙箱之外平凡的妥协过程。为了防止这种情况，可以通过sysctl启用内核YAMA ptrace限制，也可以在seccomp过滤器中将ptrace syscall列入黑名单。

#### [4.2.5 TIOCSTI](#tiocsti)

[TIOCSTI是一个ioctl](https://linux.die.net/man/4/tty_ioctl)，它允许注入终端命令，并[为攻击者提供了一种简单的机制，可以在同一用户会话内的其他进程之间横向移动](https://www.openwall.com/lists/kernel-hardening/2017/05/29/13)。可以[通过将seccomp过滤器中的ioctl列入黑名单](https://github.com/flatpak/flatpak/commit/a9107feeb4b8275b78965b36bf21b92d5724699e)或使用[bubblewrap的`--new-session`arguments](https://github.com/containers/bubblewrap/commit/06a7f31fe4a035443e25c00ecea1a3c48d77ddc1)来[缓解](https://github.com/flatpak/flatpak/commit/a9107feeb4b8275b78965b36bf21b92d5724699e)这种攻击。

### [4.3系统沙箱](#systemd-sandboxing)

[不建议使用systemd，](#choosing-the-right-distro)但有些可能无法切换。这些人至少可以使用沙盒服务，因此他们只能访问所需的内容。这是一个沙盒系统服务的示例：

```
[服务] 
CapabilityBoundingSet = CAP_NET_BIND_SERVICE
 ProtectSystem =严格
ProtectHome =真
ProtectKernelTunables =真
ProtectKernelModules =真
ProtectControlGroups =真
ProtectKernelLogs =真
ProtectHostname =真
ProtectClock =真
ProtectProc =无形
ProcSubset = PID
 PrivateTmp =真
PrivateUsers =是
PrivateDevices =真
MemoryDe​​nyWriteExecute=真
NoNewPrivileges =真
RestrictRealtime =真
RestrictSUIDSGID =真
RestrictAddressFamilies = AF_INET
 RestrictNamespaces =是
SystemCallFilter =写读了openat接近BRK FSTAT lseek的MMAP的mprotect munmap rt_sigaction rt_sigprocmask IOCTL了nanosleep选择接入的execve的getuid arch_prctl set_tid_address set_robust_list prlimit64 pread64 getrandom
 SystemCallArchitectures =本地
AppArmorProfile =的/ etc /apparmor.d/usr.bin.example
```

所有选项的说明：

*   `CapabilityBoundingSet=`—指定过程具有的[功能](#capabilities)。
*   `ProtectHome=true` —使所有主目录均不可访问。
*   `ProtectKernelTunables=true`—装入内核可调参数，例如通过`sysctl`只读方式修改的可调参数。
*   `ProtectKernelModules=true` —拒绝加载和卸载模块。
*   `ProtectControlGroups=true` —将所有控制组层次结构安装为只读。
*   `ProtectKernelLogs=true` —禁止访问内核日志。
*   `ProtectHostname=true` —禁止更改系统主机名。
*   `ProtectClock` —防止更改系统时钟。
*   `ProtectProc=invisible` —隐藏所有外部进程。
*   `ProcSubset=pid`—只允许访问的pid子集`/proc`。
*   `PrivateTmp=true`—在`/tmp`和上安装一个空的tmpfs `/var/tmp`，因此隐藏其先前的内容。
*   `PrivateUsers=true` —设置一个空的用户名称空间以隐藏系统上的其他用户帐户。
*   `PrivateDevices=true`—`/dev`使用最少的设备创建一个新的安装。
*   `MemoryDenyWriteExecute=true` —强制执行内存W ^ X策略。
*   `NoNewPrivileges=true` —防止特权提升。
*   `RestrictRealtime=true` —防止尝试启用实时调度。
*   `RestrictSUIDSGID=true` —禁止执行setuid或setgid二进制文件。
*   `RestrictAddressFamilies=AF_INET`—将可用的套接字地址系列限制为仅IPv4（`AF_INET`）。
*   `RestrictNamespaces=true` —防止创建任何新的名称空间。
*   `SystemCallFilter=...` —将允许的系统调用限制为绝对最小值。
*   `SystemCallArchitectures=native` —防止从其他CPU架构执行系统调用。
*   `AppArmorProfile=...` —在指定的AppArmor配置文件下运行该进程。

您不能仅将此示例配置复制到您的配置中。每种服务的要求各不相同，因此必须针对每个沙盒进行微调。要了解有关您可以设置的所有选项的更多信息，请阅读[systemd.exec手册页](https://www.freedesktop.org/software/systemd/man/systemd.exec.html)。  
  
如果您使用的系统不是systemd，那么可以使用bubblewrap轻松复制所有这些选项。

### [4.4 g遮阳板](#gvisor)

普通沙箱固有地与主机共享同一内核。您信任我们[已经评估为不安全](#kernel)的内核，[可以](#kernel)正确限制这些程序。[由于主机内核的整个攻击面已完全暴露，因此沙盒中的内核利用程序可以绕过任何限制](https://unit42.paloaltonetworks.com/making-containers-more-isolated-an-overview-of-sandboxed-container-technologies/)。已经进行了一些尝试来限制使用seccomp的攻击面，但不足以完全解决此问题。[gVisor](https://gvisor.dev/)是解决此问题的方法。它为每个应用程序提供了自己的内核，该内核重新实现了Linux内核的很大一部分系统调用，但使用了[内存安全的语言](#memory-safe-languages)，从而提供了更强的隔离性。

### [4.5虚拟机](#virtual-machines)

尽管不是传统的“沙盒”，[虚拟机](https://en.wikipedia.org/wiki/Virtual_machine)通过虚拟化全新系统来分离进程，从而提供了非常强大的隔离性。 [KVM](https://www.linux-kvm.org/page/Main_Page)是内核模块，它允许内核充当虚拟机监控程序，而[QEMU](https://www.qemu.org/)是利用KVM的仿真器。 [Virt-manager](https://virt-manager.org)和[GNOME Boxs](https://en.wikipedia.org/wiki/GNOME_Boxes)都是管理KVM / QEMU虚拟机的良好且易于使用的GUI。[不推荐使用Virtualbox的原因有很多](https://www.whonix.org/wiki/KVM#Why_Use_KVM_Over_VirtualBox.3F)。

[5.强化的内存分配器](#hardened-malloc)
------------------------------

[hardened_malloc](https://github.com/GrapheneOS/hardened_malloc/)是一种硬化的内存分配器，可为堆内存损坏漏洞提供实质性的保护。它很大程度上基于OpenBSD的malloc设计，但具有许多改进。  
  
可以通过`LD_PRELOAD`环境变量针对每个应用程序使用hardened_malloc 。例如，假设您编译的库位于`/usr/lib/libhardened_malloc.so`，则可以执行：

```
LD_PRELOAD = “ /usr/lib/libhardened_malloc.so”  $ program
```

还可以通过全局预加载库来在系统范围内使用它，这是推荐的使用方式。为此，请编辑 `/etc/ld.so.preload`并插入：

```
/usr/lib/libhardened_malloc.so
```

尽管大多数应用程序都可以正常工作，但hardened_malloc可能会破坏某些应用程序。建议使用以下选项编译hardened_malloc以最大程度地减少损坏：

```
CONFIG_SLAB_QUARANTINE_RANDOM_LENGTH = 0 CONFIG_SLAB_QUARANTINE_QUEUE_LENGTH = 0 CONFIG_GUARD_SLABS_INTERVAL = 8
```

您还应该使用[sysctl](#sysctl)设置以下内容，以[适应hardened_malloc创建的大量保护页面](https://github.com/GrapheneOS/hardened_malloc#traditional-linux-based-operating-systems)：

```
vm.max_map_count = 524240
```

[Whonix项目为基于Debian的发行版提供了hardened_malloc软件包](https://www.whonix.org/wiki/Hardened_Malloc)。

[6.强化编译标志](#hardened-compilation-flags)
---------------------------------------

编译自己的程序可以带来很多好处，因为它使您能够优化程序的安全性。但是，执行完全相反的操作并降低安全性很容易-如果不确定自己在做什么，请跳过本节。在基于源的发行版（[例如Gentoo）](#choosing-the-right-distro)上，这将是最简单的，但也可以在其他发行版上[这样](#choosing-the-right-distro)做。  
  
某些编译选项可用于添加其他漏洞利用缓解措施，从而消除整个类别的常见漏洞。您可能听说过常规保护，例如[位置独立可执行文件，堆栈粉碎保护程序，立即绑定，只读重定位和FORTIFY_SOURCE](https://wiki.gentoo.org/wiki/Hardened/Toolchain)但本节将不会覆盖已被广泛采用的内容。相反，它将讨论诸如控制流完整性和影子堆栈之类的_现代_漏洞利用缓解措施。  
  
本节涉及主要用C或C ++编写的本机程序。您必须使用[Clang编译器，](https://clang.llvm.org/)因为这些功能[在GCC上不可用](https://outflux.net/slides/2019/lpc/gcc-and-clang.pdf)。请记住，由于未广泛采用这些缓解措施，因此某些应用程序在启用它们后可能无法运行。

*   [Control-Flow Integrity](https://clang.llvm.org/docs/ControlFlowIntegrity.html)（CFI）是一种缓解漏洞利用的方法，[旨在防止诸如ROP或JOP之类的代码重用攻击](https://blog.trailofbits.com/2016/10/17/lets-talk-about-cfi-clang-edition/)。由于更广泛采用的缓解措施，例如NX使过时的利用技术过时，因此使用这些技术利用了很大一部分漏洞。Clang支持细粒度的前沿CFI，这意味着它可以有效缓解_JOP_攻击。Clang的CFI本身不会减轻_ROP_；您还必须使用下面记录的单独机制。要启用此功能，必须应用以下编译标志：
    
    ```
    -flto -fvisibility =隐藏-fsanitize = cfi
    ```
    
      
    
*   [影子堆栈](https://en.wikipedia.org/wiki/Shadow_stack)通过将程序复制到其他隐藏堆栈中来保护程序的返回地址。然后比较主堆栈和影子堆栈中的返回地址，看两者是否不同。如果是这样，则表明存在攻击，程序将中止，从而减轻了ROP攻击。Clang具有称为[ShadowCallStack](https://clang.llvm.org/docs/ShadowCallStack.html)的功能，可以完成此操作，但是，仅在ARM64上可用。要启用此功能，必须应用以下编译标志：
    
    ```
    -fsanitize =阴影调用堆栈
    ```
    
      
    
*   如果上述ShadowCallStack不是一个选项，则可以选择使用具有相似目标的 [SafeStack](https://clang.llvm.org/docs/SafeStack.html)。但是，不幸的是，此功能 [有许多漏洞，](https://bugs.chromium.org/p/chromium/issues/detail?id=908597)因此效果不甚理想。如果仍然希望启用此功能，则必须应用以下编译标志：
    
    ```
    -fsanitize =安全堆栈
    ```
    
      
    
*   [未初始化的内存](https://en.wikipedia.org/wiki/Uninitialized_variable) 是最常见的内存损坏漏洞之一。Clang有一个选项可以[自动](https://reviews.llvm.org/D54604)使用零或特定模式[初始化变量](https://reviews.llvm.org/D54604)。建议将变量初始化为零，因为使用其他模式比利用预防康复更适合于错误查找。要启用此功能，必须应用以下编译标志：
    
    ```
    -ftrivial-auto- var -init = zero-启用平凡的自动var -init-zero-知道将要从-clang中删除
    ```
    
      
    这种选择的存在[目前正在辩论中](https://lists.llvm.org/pipermail/cfe-dev/2020-April/065221.html)。

组 限制解除  用[内存安全语言](https://en.wikipedia.org/wiki/Memory_safety)编写的程序会自动受到保护，免受各种安全漏洞的影响，这些漏洞包括[缓冲区溢出](https://en.wikipedia.org/wiki/Buffer_overflow)，[未初始化的变量](https://en.wikipedia.org/wiki/Uninitialized_variable)， [售后使用](https://cwe.mitre.org/data/definitions/416.html)等。[Microsoft](https://msrc-blog.microsoft.com/2019/07/16/a-proactive-approach-to-more-secure-code/)和[Google的](https://www.chromium.org/Home/chromium-security/memory-safety)安全研究人员进行的研究证明，已发现的大多数漏洞都是内存安全问题。这样的内存**安全**语言的示例包括[Rust](https://doc.rust-lang.org/nomicon/meet-safe-and-unsafe.html)， [Swift](https://docs.swift.org/swift-book/LanguageGuide/MemorySafety.html)和[Java，](https://en.wikipedia.org/wiki/Java_(programming_language))而内存**不安全**语言的示例包括 [C](https://en.wikipedia.org/wiki/C_(programming_language))和[C ++](https://en.wikipedia.org/wiki/C%2B%2B)。如果可行，应使用内存安全替代品替换尽可能多的程序。

[8.根帐户](#root)
--------------

根目录可以执行任何操作，并且可以访问整个系统。因此，应尽可能将其锁定，以使攻击者无法轻松获得root用户访问权限。

### [8.1 / etc / securetty](#securetty)

该文件`/etc/securetty`指定允许您以root用户身份登录的位置。该文件应保留为空，以便任何人都不能从终端这样做。

### [8.2限制su](#restricting-su)

`su`使您可以从终端切换用户。默认情况下，它尝试以root用户身份登录。为了限制使用`su`给用户的内`wheel`群体，编辑`/etc/pam.d/su`和 `/etc/pam.d/su-l`添加：

```
 需要 验证pam_wheel .so  use_uid
```

您应该在`wheel`组中拥有尽可能少的用户。

### [8.3锁定root帐号](#locking-root)

要锁定root帐户以防止任何人以root身份登录，请执行：

```
passwd -l根
```

在执行此操作之前，请确保您具有获取根目录的另一种方法（例如，从活动USB引导并更改为文件系统的chroot），以免您无意中将自己锁定在系统之外。

### [8.4通过SSH拒绝root登录](#denying-ssh-root-login)

要阻止某人通过SSH以root用户身份登录，请编辑`/etc/ssh/sshd_config`并添加：

```
许可RootLogin  no
```

### [8.5增加散列轮数](#increase-hashing-rounds)

您可以增加阴影使用的哈希回合数，从而通过迫使攻击者计算更多的哈希值来破解您的密码，从而提高哈希密码的安全性。默认情况下，shadow使用5000发子弹，但是您可以将其增加到任意数量。尽管配置的回合越多，登录速度就越慢。编辑`/etc/pam.d/passwd`并添加回合选项。例如：

```
需要密码pam_unix.so sha512 shadow nullok rounds = 65536
```

这使阴影执行65536发子弹。  
  
应用此设置后，密码不会自动重设，因此您需要使用以下方法重置密码：

```
passwd  $用户名
```

### [8.6限制Xorg根访问](#restricting-xorg)

默认情况下，某些发行版以root用户身份运行Xorg。这是一个问题，因为Xorg包含大量古老而又复杂的代码，这增加了巨大的攻击面，并使其更有可能拥有可以获取root特权的漏洞。要阻止它以超级用户身份执行，请编辑`/etc/X11/Xwrapper.config`并添加：

```
needs_root_rights =否
```

[另外，只需切换到Wayland即可](#gui-isolation)。

### [8.7安全访问根](#accessing-root-securely)

有一个[宽范围的方法恶意软件可以使用嗅探根帐户的密码](../linux.html#root)。因此，访问根帐户的传统方式是不安全的。最好根本不访问root，但这实际上是不可行的。本节详细介绍了访问根帐户的最安全方法。在安装操作系统后，应立即应用这些说明，以确保它没有恶意软件。  
  
您绝对不能使用普通用户帐户访问root，因为root可能已被盗用。您也不能直接登录到根帐户。通过执行以下操作，创建一个单独的“管理员”用户帐户，该帐户仅用于访问root用户，而不能用于访问其他用户：

```
useradd管理员
```

通过执行以下命令来设置非常强的密码：

```
密码管理员
```

_仅_ 允许该帐户使用您首选的权限提升机制。例如，如果使用`sudo`，则通过执行以下命令来添加sudoers异常：

```
visudo -f /etc/sudoers.d/admin-account
```

现在输入：

```
admin  ALL =（全部）全部
```

确保没有其他帐户可以访问`sudo`（或您的首选机制）。  
  
现在，要实际登录到该帐户，请先重新启动-例如，这可以防止受损的窗口管理器执行[登录欺骗](https://en.wikipedia.org/wiki/Login_spoofing)。当提供登录提示时，请通过按键盘上的以下组合[键](https://www.kernel.org/doc/Documentation/SAK.txt)来激活[安全注意键](https://www.kernel.org/doc/Documentation/SAK.txt)：

```
Alt + SysRq + k
```

这将杀死当前虚拟控制台上的所有应用程序，从而克服登录欺骗攻击。现在，您可以安全地登录到您的管理员帐户，并使用root用户执行任务。完成后，注销管理员帐户并重新登录到非特权用户帐户。

[9.防火墙](#firewalls)
-------------------

防火墙可以控制传入和传出的网络流量，并且可以用来阻止或允许某些类型的流量。除非有特殊原因，否则应始终阻止所有传入流量。建议设置严格的iptables或nftables防火墙。防火墙必须针对您的系统进行微调，并且没有一个适合所有防火墙的规则集。建议您熟悉创建防火墙规则。该[拱门维基](https://wiki.archlinux.org/index.php/Iptables) 和[手册页](https://linux.die.net/man/8/iptables)都是这种既很好的资源。  
  
这是基本iptables配置的示例，该配置禁止所有传入的网络流量：

```
*滤波器
：INPUT  DROP  [0：0] 
：FORWARD  DROP  [0：0] 
：OUTPUT  ACCEPT  [0：0] 
：TCP  -  [0：0] 
：UDP  -  [0：0] 
-A  INPUT  -m 跟踪连接 - ctstate 相关，ESTABLISHED  -j  ACCEPT 
-A  INPUT  -i  LO  -j  ACCEPT 
-A  INPUT  -m 连接跟踪 --ctstate 无效 -j  DROP 
-A  INPUT  -p  UDP  -m 连接跟踪 --ctstate NEW  -j  UDP 
-A  INPUT  -p  tcp  --tcp-flags  FIN，SYN，RST，ACK  SYN  -m  conntrack  --ctstate  NEW  -j  TCP 
-A  INPUT  -p  udp  -j  REJECT  --reject-with  icmp-port -unreachable 
-A  INPUT  -p  tcp  -j  REJECT  --reject-with  tcp-reset 
-A  INPUT  -j  REJECT  --reject-with  icmp-proto-unreachable 
COMMIT
```

但是，您不应尝试在实际系统上使用此示例。它仅适用于某些台式机系统。

[10.标识符](#identifiers)
----------------------

为了保护隐私，最好最大程度地减少可追溯到您的信息量。

### [10.1主机名和用户名](#hostnames)

请勿在主机名或用户名中添加唯一标识的内容。将它们保留为通用名称，例如“主机”和“用户”，这样您就不能被它们识别。

### [10.2时区/地区/键盘映射](#timezones-locales-keymaps)

如果可能，应将您的时区设置为“ UTC”，将区域设置和键盘映射设置为“ US”。

### [10.3机器ID](#machine-id)

一个[唯一的设备ID](https://www.man7.org/linux/man-pages/man5/machine-id.5.html)存储在`/var/lib/dbus/machine-id`和systemd系统， `/etc/machine-id`也。这些应编辑为通用名称，例如[Whonix ID](https://github.com/Whonix/dist-base-files/blob/master/etc/machine-id)：

```
b08dfa6083e7567a1921a715000001fb
```

### [10.4 MAC地址欺骗](#mac-address-spoofing)

[MAC地址](https://en.wikipedia.org/wiki/MAC_address)是分配给网络接口控制器（NIC）的唯一标识符。每次您连接到网络（例如WiFi或以太网）时，您的MAC地址都会暴露出来。这使人们可以使用它来跟踪您并在本地网络上唯一地标识您。  
  
你应该**不**完全随机化MAC地址。拥有完全随机的MAC地址是显而易见的，并且会对您脱颖而出产生不利影响。MAC地址的OUI（组织唯一标识符）部分标识芯片组的制造商。对MAC地址的这一部分进行随机化处理可能会为您提供以前从未使用过的OUI，数十年来从未使用过的OUI或在您所在的地区极为罕见的OUI，因此使您脱颖而出，很明显地表明您在欺骗MAC地址。  
  
MAC地址的末尾标识您的特定设备，并且可以用来跟踪您。仅对MAC地址的这一部分进行随机化可防止您被跟踪，同时仍使MAC地址看起来可信。  
  
要欺骗这些地址，请先执行以下命令找出您的网络接口名称：

```
IP一
```

接下来，安装[macchanger](https://github.com/alobbs/macchanger)并执行：

```
macchanger -e $ network_interface
```

要在每次引导时随机分配MAC地址，应为您的特定初始化系统创建一个初始化脚本。这是systemd的一个示例：

```
[单位]
描述= macchanger上的eth0
想= network-pre.target
之前= network-pre.target
 BindsTo = SYS-子系统净装置-eth0.device
后= SYS-子系统净装置-eth0.device
 
[服务] 
ExecStart = / usr / bin / macchanger -e eth0
类型=在eshot上

[安装] 
WantedBy = multi-user.target
```

上面的示例`eth0`在启动时欺骗了接口的MAC地址。替换`eth0`为您的网络接口。

### [10.5时间攻击](#time-attacks)

几乎每个系统都有不同的时间。这可用于[时钟偏斜指纹攻击](https://www.whonix.org/wiki/Dev/TimeSync)。几毫秒的差异足以使用户匿名。

#### [10.5.1 ICMP时间戳](#icmp-timestamps)

[ICMP时间戳会在查询答复中泄漏系统时间](http://www.networksorcery.com/enp/protocol/icmp/msg13.htm)。阻止它们的最简单方法是 [阻止与防火墙的传入连接，](#firewalls)或者[使内核忽略ICMP请求](#sysctl-network)。

#### [10.5.2 TCP时间戳](#tcp-timestamps)

[TCP时间戳也会泄漏系统时间](https://web.archive.org/web/20170201160732/https://mailman.boum.org/pipermail/tails-dev/2013-December/004520.html)。内核试图通过[对每个连接使用随机偏移量](https://github.com/torvalds/linux/commit/95a22caee396cef0bb2ca8fafdd82966a49367bb)[来解决此问题，](https://forums.whonix.org/t/do-ntp-and-tcp-timestamps-really-leak-your-local-time/7824/10) 但这[不足以解决问题](https://forums.whonix.org/t/do-ntp-and-tcp-timestamps-really-leak-your-local-time/7824/10)。因此，应禁用TCP时间戳。这可以通过使用[sysctl](#sysctl)设置以下内容来完成：

```
net.ipv4.tcp_timestamps = 0
```

#### [10.5.3 TCP初始序列号](#tcp-isns)

[TCP初始序列号（ISN）是泄漏系统时间的另一种方法](https://bitguard.wordpress.com/2019/09/03/an-analysis-of-tcp-secure-sn-generation-in-linux-and-its-privacy-issues/)。为了减轻这种情况，您必须安装[tirdad内核模块](https://github.com/0xsirus/tirdad)，该[模块](https://github.com/0xsirus/tirdad)会为连接生成随机的ISN。

#### [10.5.4时间同步](#time-synchronization)

时间同步对于匿名性和安全性至关重要。错误的系统时钟可能使您遭受时钟偏斜指纹攻击，或者可以用于为您提供过时的HTTPS证书，从而绕过证书到期或吊销。  
  
最为流行的时间同步方法[NTP](https://en.wikipedia.org/wiki/Network_Time_Protocol)是不安全的，因为[它未经加密和未经](https://blog.hboeck.de/archives/863-Dont-update-NTP-stop-using-it.html)身份[验证](https://blog.hboeck.de/archives/863-Dont-update-NTP-stop-using-it.html)，因此攻击者可以轻易地拦截和修改请求。[NTP还以NTP时间戳格式泄漏本地系统时间，](https://tools.ietf.org/html/rfc5905#page-23)该[格式](https://tools.ietf.org/html/rfc5905#page-23)可用于时钟偏斜指纹识别，如前所述。  
  
因此，您应该卸载所有NTP客户端并禁用[systemd-timesyncd](https://www.freedesktop.org/software/systemd/man/systemd-timesyncd.service.html)如果正在使用。您可以通过安全连接（HTTPS或最好是Torion服务）连接到受信任的网站，而不是NTP，并从HTTP标头中提取当前时间。完成此任务的工具是[sdwdate](https://www.whonix.org/wiki/Sdwdate)或我自己的[secure-time-sync](https://gitlab.com/madaidan/secure-time-sync)。

### [10.6击键指纹](#keystroke-fingerprinting)

可以[通过人们在键盘上输入键的方式](https://arxiv.org/pdf/1609.07612.pdf)来[对人](https://arxiv.org/pdf/1609.07612.pdf)进行[指纹识别](https://arxiv.org/pdf/1609.07612.pdf)。您可以通过输入速度，在两次按键之间的暂停，每次按下和释放的确切时间等方面进行独特的指纹识别。可以使用[KeyTrac](https://www.keytrac.net/en/tryout)在线进行测试。  
  
[kloak](https://github.com/vmonaco/kloak)是一种工具，旨在通过混淆按键和释放事件之间的时间间隔来[克服](https://github.com/vmonaco/kloak)这种跟踪方法。当按键被按下时，它会在应用程序选择之前引入一个随机延迟。虽然，这可能会使某些人感到沮丧，并且不适合他们。  
  
这种跟踪形式一定不能与[笔法](https://en.wikipedia.org/wiki/Stylometry)混淆。

[11.文件权限](#file-permissions)
----------------------------

默认情况下，文件的权限是非常宽松的。您应该在整个系统中搜索权限不当的文件和目录，并对其进行限制。例如，在诸如Debian之类的某些发行版中，用户的主目录是世界可读的。这可以通过执行以下操作来限制：

```
chmod  700 / home / $ user
```

还有另外一些示例`/boot`，`/usr/src`以及`/{,usr/}lib/modules`-这些示例包含内核映像，System.map和其他各种文件，所有这些文件都[可能泄漏有关内核的敏感信息](#other-kernel-pointer-leaks)。要限制对它们的访问，请执行：

```
chmod  700 /启动/ usr / src / lib / modules / usr / lib / modules
```

在基于Debian的发行版上，必须使用[dpkg-statoverride](https://manpages.debian.org/buster/dpkg/dpkg-statoverride.1.en.html)保留文件许可权。否则，它们将在更新期间被覆盖。  
  
Whonix的[SUID Disabler和Permission Hardener](https://www.whonix.org/wiki/SUID_Disabler_and_Permission_Hardener)自动应用本节中详述的步骤。

### [11.1 setuid / setgid](#setuid)

setuid / SUID允许用户使用二进制文件所有者的特权执行二进制文件。这通常用于允许非特权用户使用通常只为root用户保留的某些功能。因此，许多SUID二进制文件都具有特权升级安全漏洞的历史记录。setgid / SGID类似，但适用于组而不是用户。要使用setuid或setgid位查找系统上的所有二进制文件，请执行：

```
查找/-键入f \（-perm -4000 -o -perm -2000 \）
```

然后，您应该删除不使用的程序上的所有不必要的setuid / setgid位，或将其替换为[功能](#capabilities)。  
  
要删除setuid位，执行：

```
chmod u -s  $ path_to_program
```

要删除setgid位，执行：

```
chmod g -s  $ path_to_program
```

要向文件添加功能，请执行：

```
setcap  $ capability + ep $ path_to_program
```

或者，要删除不必要的功能，请执行：

```
setcap -r $ path_to_program
```

### [11.2 umask](#umask)

umask设置新创建文件的默认文件权限。缺省的umask`0022`不是很安全，因为它为系统上的每个用户提供了对新创建文件的读取访问权限。要使所有者以外的任何人都不可读新文件，请编辑`/etc/profile`并添加：

```
umask 0077
```

[12.核心转储](#core-dumps)
----------------------

[核心转储](https://en.wikipedia.org/wiki/Core_dump)通常在某个程序崩溃时包含在特定时间记录的程序内存。它们可能包含敏感信息，例如密码和加密密钥，因此必须将其禁用。  
  
禁用它们的方法主要有三种：sysctl，systemd和ulimit。

### [12.1系统](#core-dumps-sysctl)

通过[sysctl](#sysctl)设置以下设置：

```
kernel.core_pattern = | / bin / false
```

### [12.2系统](#core-dumps-systemd)

创建`/etc/systemd/coredump.conf.d/disable.conf`并添加：

```
[Coredump]
储存=无
```

### [12.3 ulimit](#core-dumps-ulimit)

编辑`/etc/security/limits.conf`并添加：

```
*硬核0
```

### [12.4 setuid进程](#core-dumps-setuid)

即使经过这些设置，以提升的特权运行的进程仍可能会转储其内存。为了防止他们这样做，请通过[sysctl](#sysctl)设置以下内容：

```
fs.suid_dumpable = 0
```

[13.交换](#swap)
--------------

与[核心转储](#core-dumps)类似，[交换或分页将](https://en.wikipedia.org/wiki/Paging)部分内存复制到磁盘，其中可能包含敏感信息。应该将内核配置为仅在绝对必要时与此[sysctl](#sysctl)交换：

```
vm.swappiness = 1
```

[14. PAM](#pam)
---------------

[PAM](http://www.linux-pam.org/)是用于用户身份验证的框架。这就是您登录时使用的内容。您可以通过要求使用强密码或在登录尝试失败时强制延迟来提高安全性。  
  
要强制使用强密码，可以使用[pam_pwquality](https://linux.die.net/man/8/pam_pwquality)。它强制执行密码的可配置策略。例如，如果您希望密码至少包含16个字符（最小），与旧密码（difok）至少6个不同的字符，至少3个数字（dcredit），至少2个大写字母（ucredit），至少2个字符小写（lcredit）和至少3个其他字符（ocredit），然后编辑`/etc/pam.d/passwd`并添加：

```
 需要
密码pam_pwquality.so重试= 2分钟= 16 difok = 6 dcredit = -3 ucredit = -2 lcredit = -2 ocredit = -3force_for_root需要密码 pam_unix.so use_authtok sha512阴影
```

要强制执行延迟，可以使用[pam_faildelay](https://www.man7.org/linux/man-pages/man8/pam_faildelay.8.html)。要在两次失败的登录尝试之间添加至少4秒的延迟以阻止暴力破解尝试，请编辑`/etc/pam.d/system-login`并添加：

```
auth可选pam_faildelay.so delay = 4000000
```

“ 4000000”是4秒（以微秒为单位）。

[15.微码更新](#microcode)
---------------------

微代码更新对于修复关键的CPU漏洞（如[Meltdown和Spectre](https://meltdownattack.com)等）至关重要。大多数发行版都将这些发行版包含在其软件存储库中，例如[Arch Linux](https://wiki.archlinux.org/index.php/Microcode)和[Debian](https://wiki.debian.org/Microcode)。

[16. IPv6隐私扩展](#ipv6-privacy)
-----------------------------

[IPv6地址是从计算机的MAC地址生成的](https://iapp.org/news/a/2011-09-09-facing-the-privacy-implications-of-ipv6/)，从而使您的IPv6地址唯一，并直接绑定到计算机。 [隐私扩展会](https://tools.ietf.org/html/rfc4941)生成一个随机的IPv6地址，以减轻这种形式的跟踪。请注意，如果您 [欺骗了MAC地址](#mac-address-spoofing)或[禁用了IPv6，](#kasr-boot-parameters)则无需执行这些步骤。  
  
要启用这些功能，请通过[sysctl](#sysctl)设置以下设置：

```
net.ipv6.conf.all.use_tempaddr = 2 
net.ipv6.conf.default.use_tempaddr = 2
```

### [16.1 NetworkManager](#ipv6-networkmanager)

要[为NetworkManager](https://blogs.gnome.org/lkundrak/2015/12/03/networkmanager-and-privacy-in-the-ipv6-internet/)启用[隐私扩展](https://blogs.gnome.org/lkundrak/2015/12/03/networkmanager-and-privacy-in-the-ipv6-internet/)，请编辑`/etc/NetworkManager/NetworkManager.conf`并添加：

```
[连接] 
ipv6.ip6-privacy = 2
```

### [16.2系统联网](#ipv6-systemd-networkd)

要[为systemd-networkd](https://www.freedesktop.org/software/systemd/man/systemd.network.html#IPv6PrivacyExtensions=)启用[隐私扩展](https://www.freedesktop.org/software/systemd/man/systemd.network.html#IPv6PrivacyExtensions=)，请创建`/etc/systemd/network/ipv6-privacy.conf`并添加：

```
[网络] 
IPv6PrivacyExtensions =内核
```

[17.分区和挂载选项](#partitioning)
---------------------------

文件系统应分为多个分区，以对其权限进行细粒度控制。可以添加不同的安装选项以限制可以执行的操作：  

*   nodev —禁止使用设备。
*   nosuid —禁止[setuid或setgid位](#setuid)。
*   noexec-禁止执行任何二进制文件。

这些安装选项应尽可能在中设置`/etc/fstab`。如果不能使用单独的分区，请创建绑定安装。一个更安全的例子`/etc/fstab`：

```
// ext4默认为1 1
/ home / home ext4默认值，nosuid，noexec，nodev 1 2
/ tmp / tmp ext4默认值，bind，nosuid，noexec，nodev 1 2
/ var / var ext4默认值，bind，nosuid 1 2
/ boot / boot ext4默认值，nosuid，noexec，nodev 1 2
```

请注意，`noexec`可以[通过shell脚本绕过](https://chromium.googlesource.com/chromiumos/docs/+/master/security/noexec_shell_scripts.md)。

[18.熵](#entropy)
----------------

### [18.1额外的熵源](#additional-entropy-sources)

[熵](https://en.wikipedia.org/wiki/Entropy_(computing))基本上是操作系统收集的随机性，对于诸如加密之类的事情至关重要。因此，最好通过安装额外的随机数生成器（如[Haved](http://www.issihosts.com/haveged/) 和[Jitterentropy）](https://github.com/smuellerDD/jitterentropy-rngd)从各种来源收集尽可能多的熵。  
  
为了使抖动熵正常工作，必须通过创建`/usr/lib/modules-load.d/jitterentropy.conf`并添加以下内容来尽早加载内核模块：

```
jitterentropy_rng
```

### [18.2 RDRAND](#rdrand)

[RDRAND](https://www.felixcloutier.com/x86/rdrand)是用于提供随机数的CPU指令。如果可用，内核会自动将其用作熵源。但是由于它是专有的并且是CPU本身的一部分，因此无法审计和验证其安全性。如果愿意，您甚至无法对代码进行反向工程。这RNG已经[遭受了](https://en.wikipedia.org/wiki/RDRAND#Reception) [从](https://arstechnica.com/gadgets/2019/10/how-a-months-old-amd-microcode-bug-destroyed-my-weekend/) [漏洞](https://nitter.net/RichFelker/status/1125794261839032320) [之前](https://nitter.net/pid_eins/status/1149649806056280069)，其中一些可能是[后门程序](https://archive.nytimes.com/www.nytimes.com/interactive/2013/09/05/us/documents-reveal-nsa-campaign-against-encryption.html)。通过[设置以下引导参数](#boot-parameters)可以不信任此功能：

```
random.trust_cpu =关闭
```

[19.以根用户身份编辑文件](#editing-as-root)
---------------------------------

建议不要以root用户身份运行普通的文本编辑器。大多数文本编辑器可以做的不仅仅是简单地编辑文本文件，而且可以被利用。例如，`vi`以root用户打开并输入`:sh`。现在，您具有一个可以访问整个系统的root外壳，攻击者可以轻松利用该外壳。  
  
一种解决方法是使用`sudoedit`。这会将文件复制到一个临时位置，以普通用户身份打开文本编辑器，编辑该临时文件，并以root用户身份覆盖原始文件。这样，实际的编辑器就不会以root身份运行。要使用`sudoedit`，请执行：

```
sudoedit  $ path_to_file
```

默认情况下，它使用，`vi`但是可以通过`EDITOR`或`SUDO_EDITOR`环境变量切换默认编辑器。例如，使用`nano`，执行：

```
编辑器=无no sudoedit $ path_to_file
```

可以在中全局设置此环境变量`/etc/environment`。

[20.特定于发行版的强化](#distro-specific)
--------------------------------

### [20.1 HTTPS包管理器镜像](#https-mirrors)

默认情况下，Linux发行版通常使用HTTP或HTTP和HTTPS镜像的混合来从其软件存储库下载软件包。人们认为这很好，因为程序包管理器会在安装前验证程序包的签名。然而，从历史上看，[一直](https://security.archlinux.org/ASA-201903-7) [以来](https://security.archlinux.org/ASA-201910-13) [多](https://www.debian.org/security/2016/dsa-3733) [旁路](https://www.debian.org/security/2019/dsa-4371) [的](https://justi.cz/security/2019/01/22/apt-rce.html) [这个](https://justi.cz/security/2018/09/13/alpine-apk-rce.html)。您应将程序包管理器配置为从HTTPS镜像专门下载以进行深度防御。

### [20.2 APT seccomp-bpf](#apt-seccomp-bpf)

自[软件包管理器](https://www.debian.org/releases/buster/amd64/release-notes/ch-whats-new.en.html#apt-sandboxing)Debian Buster以来[，APT已支持可选的seccomp-bpf过滤](https://www.debian.org/releases/buster/amd64/release-notes/ch-whats-new.en.html#apt-sandboxing)。这限制了允许执行APT的系统调用，这可能严重限制攻击者在尝试利用APT中的漏洞时对系统造成危害的能力。要启用此功能，请创建`/etc/apt/apt.conf.d/40sandbox`并添加：

```
APT :: Sandbox :: Seccomp “ true” ;
```

[21.人身安全](#physical-security)
-----------------------------

### [21.1加密](#encryption)

全盘加密可确保对驱动器上的所有数据进行加密，并且不会被物理攻击者读取。大多数发行版都支持在安装过程中启用加密。确保设置了强密码。您也可以使用[dm-crypt](https://wiki.archlinux.org/index.php/Dm-crypt)手动加密驱动器。  
  
请注意，全盘加密不包括`/boot`。这样，仍然可以修改内核，引导加载程序和其他关键文件。为了完全防止篡改，您还必须实施[经过验证的引导](#verified-boot)。

### [21.2 BIOS / UEFI加固](#bios-uefi)

如果您仍在使用旧版BIOS，则应迁移到UEFI，以利用较新的安全功能。  
  
大多数BIOS或UEFI实现都支持设置密码。最好启用它并设置一个非常强的密码。虽然这是很弱的保护，因为重置密码很简单。它通常存储在易失性内存中，因此攻击者只需要能够卸下CMOS电池几秒钟，或者他们就可以使用某些主板上的跳线将其重置。  
  
您还应该禁用所有未使用的设备和引导选项，例如USB引导，以减少攻击面。  
  
通常忽略BIOS或UEFI的更新-确保将其更新。将其与常规操作系统更新一样重要。  
  
此外，请参阅《[NSA的硬件和固件安全指南》](https://github.com/nsacyber/Hardware-and-Firmware-Security-Guidance)。

### [21.3引导程序密码](#bootloader-passwords)

引导加载程序会在引导过程的早期执行，并负责加载操作系统。保护自举程序非常重要。否则，它可能会被篡改-例如，本地攻击者可以通过`init=/bin/bash`在引导时用作内核参数来轻松获得root shell，该参数告诉内核执行`/bin/bash`而不是常规的init系统。您可以通过为引导加载程序设置密码来防止这种情况。  
  
仅设置引导程序密码不足以完全保护它。您还必须[按照以下说明](#verified-boot)设置经过验证的启动。

#### [21.3.1 GRUB](#grub)

要为GRUB设置密码，请执行：

```
grub-mkpasswd-pbkdf2
```

输入您的密码，该密码将生成一个字符串。它将类似于“ grub.pbkdf2.sha512.10000.C4009 ...”。创建`/etc/grub.d/40_password`并添加：

```
设置超级用户= “ $ username ” 
password_pbkdf2 $ username  $ password
```

用生成的字符串替换“ $ password” `grub-mkpasswd-pbkdf2`。“ $ username”将用于被允许使用GRUB命令行，编辑菜单项和执行任何菜单项的超级用户。对于大多数人来说，这只是“根”。  
  
[重新生成配置文件](#regenerate-grub-config)，GRUB现在将受到密码保护。  
  
要限制仅编辑引导参数和访问GRUB控制台，同时仍然允许您引导，编辑，`/boot/grub/grub.cfg`并在“菜单'$ OSName'”旁边添加“ --unrestricted”参数。例如：

```
menuentry  'Arch Linux的' --unrestricted
```

您将需要再次重新生成配置文件以应用此更改。

#### [21.3.2 Syslinux](#syslinux)

Syslinux可以设置主密码或菜单密码。引导任何条目都需要主密码，而引导特定条目仅需要菜单密码。  
  
要为Syslinux设置主密码，请编辑`/boot/syslinux/syslinux.cfg`并添加：

```
MENU MASTER PASSWD $ password
```

要设置菜单密码，请编辑`/boot/syslinux/syslinux.cfg`并在带有您要密码保护的项目的标签内，添加：

```
MENU PASSWD $密码
```

将“ $ password”替换为您要设置的密码。  
  
这些密码可以是纯文本，也可以使用MD5，SHA-1，SHA-256或SHA-512进行散列。建议先使用强哈希算法（例如SHA-256或SHA-512）对密码进行哈希处理，以避免将其存储为明文形式。

#### [21.3.3 systemd-boot](#systemd-boot)

systemd-boot具有防止在引导时编辑内核参数的选项。在`loader.conf`文件中，添加：

```
编辑 号
```

systemd-boot并不正式支持保护内核参数编辑器的密码，但是您可以使用[systemd-boot-password](https://github.com/kitsunyan/systemd-boot-password)来实现。

### [21.4验证启动](#verified-boot)

[经过验证的引导](https://source.android.com/security/verifiedboot/)通过密码验证来确保引导链和基本系统的完整性。这可用于确保物理攻击者无法修改设备上的软件。如果没有经过验证的引导，则一旦获得物理访问权限，就可以轻松绕过上述所有预防措施。经过验证的引导不仅像许多人认为的那样是为了物理安全。它也可以用来防止远程恶意软件持久化-如果攻击者设法破坏了整个系统并获得了很高的特权，则经过验证的引导将在重新引导时还原其更改并确保它们不会持久。  
  
经过验证的最常见的启动实施是[UEFI安全启动](https://www.intel.com/content/www/us/en/support/articles/000006942/boards-and-kits/desktop-boards.html) 但是，这本身并不是一个完整的实现，因为它仅验证引导加载程序和内核，这意味着可以通过一些方法来绕过此操作：

*   仅UEFI安全启动就没有一成不变的信任根，因此物理攻击者仍然可以刷新设备的固件。为了减轻这种情况，请结合使用UEFI安全启动和[Intel Boot Guard](https://mjg59.dreamwidth.org/33981.html)或[AMD Secure Boot](https://blog.cloudflare.com/anchoring-trust-a-hardware-secure-boot-story/)。
*   远程攻击者（或不使用[加密](#encryption)时为物理攻击者）可以简单地修改操作系统的任何其他特权部分。例如，如果他们有修改内核的特权，那么他们也可以修改`/sbin/init`以有效地获得相同的结果。因此，仅验证内核和引导加载程序不会对远程攻击者产生任何影响。为了减轻这种情况，您必须使用 [dm-verity验证](https://www.kernel.org/doc/html/latest/admin-guide/device-mapper/verity.html)基本操作系统，尽管由于传统Linux发行版的布局，这非常困难且麻烦。

通常，很难在传统Linux上实现可观的经过验证的引导实现。

### [21.5个USB](#usbs)

USB设备为物理攻击提供了重要的攻击面。[BadUSB](https://srlabs.de/bites/badusb/)和 [Stuxnet](https://en.wikipedia.org/wiki/Stuxnet)就是这类攻击的[例子](https://srlabs.de/bites/badusb/)。优良作法是禁止所有新连接的USB且仅将受信任设备列入白名单。[USBGuard](https://usbguard.github.io/)对此非常[有用](https://usbguard.github.io/)。  
  
您还可以将其`nousb`用作[内核引导参数](#boot-parameters)来禁用[内核中的](#boot-parameters)所有USB支持。如果使用 [linux-hardened](#linux-hardened)，则可以设置`kernel.deny_new_usb=1` [sysctl](#sysctl)。

### [21.6 DMA攻击](#dma-attacks)

[直接内存访问（DMA）攻击](https://en.wikipedia.org/wiki/DMA_attack)涉及通过插入某些物理设备来完全访问所有系统内存。这可以通过控制设备可访问的内存区域的[IOMMU](https://en.wikipedia.org/wiki/Input%E2%80%93output_memory_management_unit)或将特别易受攻击的内核模块列入黑名单来缓解。  
  
要启用IOMMU，请设置以下[内核引导参数](#boot-parameters)：

```
intel_iommu =开启amd_iommu =开启
```

您只需要为特定的CPU制造商启用该选项，但同时启用这两个选项就没有问题。

```
efi = disable_early_pci_dma
```

此选项通过在非常早的启动期间禁用所有PCI桥接器上的busmaster位来[修复上述IOMMU中的漏洞](https://mjg59.dreamwidth.org/54433.html)。  
  
此外，[Thunderbolt](https://en.wikipedia.org/wiki/Thunderbolt_(interface)#Security_vulnerabilities)和[FireWire](https://en.wikipedia.org/wiki/IEEE_1394#Security_issues)通常容易受到DMA攻击。要禁用它们，请将[这些内核模块列入黑名单](#kasr-kernel-modules)：

```
安装火线核心/ bin / false
安装雷电/ bin / false
```

### [21.7冷启动攻击](#cold-boot-attacks)

一个[冷启动攻击](https://en.wikipedia.org/wiki/Cold_boot_attack)时，它被删除之前，攻击者分析在RAM中的数据发生。使用现代RAM时，冷启动攻击不太实用，因为RAM通常会在几秒钟或几分钟内清除，除非将其放入冷却液（如液氮或冷冻机）中。攻击者将不得不从您的设备中拔出RAM棒，并在几秒钟内将其暴露于液氮中，而用户不会注意到。  
  
如果冷启动攻击是威胁模型的一部分，请在关机后保护计算机几分钟，以确保没有人可以访问您的RAM记忆棒。您也可以将RAM棒焊接到主板上，以使其更难以卡住。如果使用笔记本电脑，请取出电池并直接从充电电缆中拔出电源。关机后拔出电缆，以确保RAM无法获得更多电源以保持生命。  
  
在[内核自我保护启动参数部分中](#boot-kernel)，空闲时内存清零选项将用零覆盖内存中的敏感数据。此外，[强化的内存分配器](#hardened-malloc)可以通过以下方式清除用户空间堆内存中的敏感数据：[CONFIG_ZERO_ON_FREE配置选项](https://github.com/GrapheneOS/hardened_malloc#configuration)。尽管如此，某些数据仍可能保留在内存中。  
  
此外，现代内核还包括[复位攻击缓解措施](https://lwn.net/Articles/730006/)，该命令可命令固件在关机时擦除数据，尽管这需要[固件支持](https://www.trustedcomputinggroup.org/wp-content/uploads/Platform-Reset-Attack-Mitigation-Specification.pdf)。  
  
确保正常关闭计算机，以使以上所述的缓解措施可以开始起作用。  
  
如果以上都不适合您的威胁模型，则可以实施[Tails的内存擦除过程](https://tails.boum.org/contribute/design/memory_erasure/)，该[过程](https://tails.boum.org/contribute/design/memory_erasure/)将擦除大部分内存（[视频内存除外）](https://tails.boum.org/support/known_issues/index.en.html#video-memory)），并已被证明是有效的。

[22.最佳做法](#best-practices)
--------------------------

一旦对系统进行了尽可能多的加固，就应该遵循良好的隐私和安全规范：  
  
1.禁用或删除不需要的东西以最大程度地减少攻击面。  
2.保持更新。配置cron作业或init脚本以每天更新系统。  
3.不要泄漏有关您或您的系统的任何信息，无论看起来多么渺小。  
4.[遵循一般的安全和隐私建议](../security-privacy-advice.html)。  
  
尽管已经进行了加固，但您必须记住[Linux仍然是一个有缺陷的操作系统](../linux.html)，没有任何加固措施可以完全修复它。

[其他指南](#other-guides)
---------------------

您应该进行尽可能多的研究，而不要依赖单一的信息来源。用户是最大的安全问题之一。这些是我认为很有价值的其他指南的链接：  
  
[Arch Linux安全性Wiki页面](https://wiki.archlinux.org/index.php/Security)  
  
[Whonix文档《](https://www.whonix.org/wiki/Documentation)  
  
[NSA RHEL 5强化指南》](https://apps.nsa.gov/iaarchive/library/ia-guidance/security-configuration/operating-systems/guide-to-the-secure-configuration-of-red-hat-enterprise.cfm)（虽然有些过时，但仍包含有用的信息）  
  
[KSPP建议使用内核设置](https://kernsec.org/wiki/index.php/Kernel_Self_Protection_Project/Recommended_Settings)  
  
[kconfig-hardened-check](https://github.com/a13xp0p0v/kconfig-hardened-check/)

[词汇表](#glossary)
----------------

### [重新生成GRUB配置](#regenerate-grub-config)

您可能需要重新生成GRUB配置，以应用对引导加载程序所做的某些更改。在不同的发行版之间，执行此步骤的步骤有时可能会有所不同。例如，在诸如Arch Linux之类的发行版上，应通过执行以下命令来重新生成配置文件：

```
grub-mkconfig -o $ path_to_grub_config
```

“ $ path_to_grub_config”取决于您如何设置系统。通常是`/boot/grub/grub.cfg`或者是，`/boot/EFI/grub/grub.cfg` 但是在执行此命令之前，您应该确保。  
  
另外，在Debian或Ubuntu等发行版上，您应该执行以下命令：

```
更新-grub
```

### [能力](#capabilities)

在Linux内核中，“根特权”分为各种不同的[功能](https://man7.org/linux/man-pages/man7/capabilities.7.html)。这在应用[最小特权原则时](https://en.wikipedia.org/wiki/Principle_of_least_privilege)很有帮助-可以给它们仅授予特定的子集，而不是赋予进步总的root特权。例如，如果程序只需要设置系统时间，则仅需要 `CAP_SYS_TIME` 而不是总根。这可能会限制可能造成的潜在损害，但是，您仍必须手动授予功能，因为[无论如何这些都可能被滥用以获取全部root特权](https://forums.grsecurity.net/viewtopic.php?t=2522)。

[回去](../index.html)

🌓

![](https://www.gstatic.com/images/branding/product/1x/translate_24dp.png)

联合国
===

提供更好的翻译建议

* * *