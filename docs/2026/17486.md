

# 《The Shellcoder's handbook》第十章_Solaris 破解入门

## 10 Solaris 破解入门

很长一段时间以来，Solaris 主要支持高端的 Web 和数据库服务。尽管 Solaris 有 Interl 发行版，但绝大多数的 Solaris 还是运行在 SPARC 平台之上。我们在本章将把精力放在 SPARC 上的 Solaris，而它也是影响深远的操作系统之一。Solaris 在以前被称为 SunOS，当然这样的称呼已渐渐被大家遗忘了，现在常见的版本是 2.6，7，8，和 9。

当其它的操作系统倾向于在默认安装情况下只提供基本服务时，Solaris 9 仍提供了大量的网络服务，例如，默认安装的 Solaris 9 启用近 20 个 RPC 服务。在以前，RPC 服务里涌现了大量的漏洞；现在，在网络上可以获得大量的 Solaris 源代码，似乎预示着在 RPC 里可能会发现更多的漏洞。

从历史上说，几乎所有的 Solaris RPC 服务都出现过漏洞（sadmind，cmsd，statd，automount 通过 statd，snmXdmid，dmispd，cachefsd，等等），也发现了通过 inetd 使用的远程错误，如 telnetd，/bin/login（通过 telnetd 和 rshd），dtspcd，lpd 等。Solaris 在缺省状态下有大量带 setuid 位的文件，因此，在正式使用 Solaris 前，应该对它进行仔细地加固。

当然，Solaris 也内置了一些安全功能，包括进程记帐、审计、和可选的 non-executable 栈。从管理员的立场来看，启用这个选项是值得的，因为它提供了一定程度上的保护。

注：大家不要把这里的破解（exploit）和通常所说的破解（crack）搞混淆了。

前两天特忙，这算是迟到的新年祝福吧！

祝大家的劳动都有收获

祝大家的生活天天快乐

Arhat.ptg @ 2007.1.1

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [10.solaris 破解入门.rar](javascript:void(0);)

  （500.75kb，292次下载）

