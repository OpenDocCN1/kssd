
# ncftpd 2.7.1

> 原文：[`www.pediy.com/kssd/pediy05/pediy50057.htm`](https://www.pediy.com/kssd/pediy05/pediy50057.htm)

## ncftpd 2.7.1 之破解

（by： slgx）

注：这是我第一次写 unix 下的破解教程，不当之处多多包涵。如要转载请保留作者姓名，谢谢！

破解之前，介绍一下 `ncftpd` 是 unix 类(如 linux，freebsd，solaris，sco 等)下的一个 ftp 服务器软件。该软件以安全性高，重负载特性好而著称。其对教育用户免费，对商业用户收取$99~199 美元不等，对试用用户则有连接数限制。好，我们开始！

### 第一步：安装 `ida pro 4.17 for windows`。

过程略。(当然如果你精通 unix，会用 unix 下的 `dasm` 脚本，就不必了)

### 第二步：下载 `ncftpd`。 from `http://www.ncftp.com`

过程略。(有很多 unix 版本，各取所需。就是没有 windows 版本。)

### 第三步：用 `winzip` 解包(同样如果你会用 unix 的话请用 `tar -zxvf XXXXXX` 命令解包。)

以 `ncftpd-2.7.1-freebsd4.4-i386-export.tar.gz` 为例。

第一次解除压缩得到 `ncftpd-2.7.1-freebsd4.4-i386-export.tar`

第二次解除 `tar`，得到原始文件。

从中提出 `ncftpd` 文件。

### 第四步：反汇编 `ncftpd`。

打开 `ida`，选择目标文件 `ncftpd`，以 elf 文件格式和自己计算机相对应的 cpu 进行反汇编。

### 第五步

哈哈，写不下去了^\_^。因为要用到 unix 了 :)。如果大家有用过 `ncftpd` 就知道如果没有 licence，在登 陆 `ncftpd` 的时候会出现“unregistered copy”的提示信息。哼哼，很眼熟吧。熟悉 windows 破解的大 侠可能已经知道该怎么做了。对，在 `ida` 中搜索“unregistered copy”字符串。我们来到下面：

以下代码，请从下往上看：

```
.text:0804C090
.text:0804C090 ; Attributes: bp-based frame
.text:0804C090
.text:0804C090 sub_0_804C090 proc near             ; CODE XREF: sub_0_804E478+A6p
.text:0804C090
.text:0804C090 var_18          = dword ptr -18h
.text:0804C090
.text:0804C090               push    ebp
.text:0804C091               mov    ebp, esp
.text:0804C093               sub    esp, 8
.text:0804C096               mov    al, ds:byte_0_8098720<===>ds:byte_0_8098720 应该为非 0h 非 66h。
.text:0804C09B               test    al, al
.text:0804C09D               jz      loc_0_804C1A0<==========>去 " (unregistered copy)"
.text:0804C0A3               cmp    al, 66h
.text:0804C0A5               jz      loc_0_804C12C<==========>不感兴趣，应该是注册为其它版。
.text:0804C0AB               cmp    ds:byte_0_80984A0, 0<====>ds:byte_0_80984A0 必为 0h
.text:0804C0B2               jz      short loc_0_804C0EE<====>去 " (licensed copy)"
.text:0804C0B4               cmp    dword_0_80833E0, 3
.text:0804C0BB               jle    short loc_0_804C0CC
.text:0804C0BD               mov    ds:dword_0_8097D20, 1
.text:0804C0C7
.text:0804C0C7 locret_0_804C0C7:                     ; CODE XREF: sub_0_804C090+99j
.text:0804C0C7               leave
.text:0804C0C8               retn
.text:0804C0C8
.text:0804C0C9               align 4
.text:0804C0CC
.text:0804C0CC loc_0_804C0CC:                         ; CODE XREF: sub_0_804C090+2Bj
.text:0804C0CC               push    8085F10h
.text:0804C0D1               push    3Eh
.text:0804C0D3               push    1
.text:0804C0D5               push    offset aSorryTheBanner ; "Sorry, the \"banner\" option is not avail"...
.text:0804C0DA               call    _fwrite
.text:0804C0DF               mov    [esp+18h+var_18], 1
.text:0804C0E6               call    sub_0_804ECE4
.text:0804C0EB               add    esp, 10h
.text:0804C0EE
.text:0804C0EE loc_0_804C0EE:        ; CODE XREF: sub_0_804C090+22j<======>不用我说了吧
.text:0804C0EE               push    edx
.text:0804C0EF               push    50h
.text:0804C0F1               push    offset aNcftpdServer ; "NcFTPd Server"
.text:0804C0F6               push    offset byte_0_80984A0
.text:0804C0FB               call    sub_0_806E298
.text:0804C100               add    esp, 0Ch
.text:0804C103               push    50h
.text:0804C105               push    offset aLicensedCopy ; " (licensed copy)"<=====>哈哈，看到正解了
.text:0804C10A
.text:0804C10A loc_0_804C10A:                         ; CODE XREF: sub_0_804C090+10Bj
.text:0804C10A                               ; sub_0_804C090+157j
.text:0804C10A               push    offset byte_0_80984A0
.text:0804C10F               call    sub_0_806E4E8
.text:0804C114               add    esp, 10h
.text:0804C117               push    eax
.text:0804C118               push    50h
.text:0804C11A               push    offset aReady_  ; " ready."
.text:0804C11F               push    offset byte_0_80984A0
.text:0804C124               call    sub_0_806E4E8
.text:0804C129               jmp    short locret_0_804C0C7
.text:0804C129
.text:0804C12B               align 4
.text:0804C12C
.text:0804C12C loc_0_804C12C:                         ; CODE XREF: sub_0_804C090+15j
.text:0804C12C               cmp    ds:byte_0_80984A0, 0
.text:0804C133               jz      short loc_0_804C157
.text:0804C135               push    8085F10h
.text:0804C13A               push    3Eh
.text:0804C13C               push    1
.text:0804C13E               push    offset aSorryTheBanner ; "Sorry, the \"banner\" option is not avail"...
.text:0804C143               call    _fwrite
.text:0804C148               mov    [esp+18h+var_18], 1
.text:0804C14F               call    sub_0_804ECE4
.text:0804C154               add    esp, 10h
.text:0804C157
.text:0804C157 loc_0_804C157:                         ; CODE XREF: sub_0_804C090+A3j
.text:0804C157               push    ecx
.text:0804C158               push    50h
.text:0804C15A               push    offset aNcftpdServer ; "NcFTPd Server"
.text:0804C15F               push    offset byte_0_80984A0
.text:0804C164               call    sub_0_806E298
.text:0804C169               add    esp, 0Ch
.text:0804C16C               push    50h
.text:0804C16E               push    offset asc_0_8075AC6 ; " ("
.text:0804C173               push    offset byte_0_80984A0
.text:0804C178               call    sub_0_806E4E8
.text:0804C17D               add    esp, 0Ch
.text:0804C180               push    50h
.text:0804C182               push    offset byte_0_8098720
.text:0804C187               push    offset byte_0_80984A0
.text:0804C18C               call    sub_0_806E4E8
.text:0804C191               add    esp, 0Ch
.text:0804C194               push    50h
.text:0804C196               push    offset asc_0_8075AC9 ; ")"
.text:0804C19B               jmp    loc_0_804C10A
.text:0804C1A0
.text:0804C1A0
.text:0804C1A0 loc_0_804C1A0:                         ; CODE XREF: sub_0_804C090+Dj<=====> 这又是多少?对了是.text:0804C09D
.text:0804C1A0               cmp    ds:byte_0_80984A0, 0
.text:0804C1A7               jz      short loc_0_804C1CB<=================================>原来是从这里跳过去的！
.text:0804C1A9               push    8085F10h
.text:0804C1AE               push    3Ch
.text:0804C1B0               push    1
.text:0804C1B2               push    offset aNcftpdMustBeLi ; "NcFTPd must be licensed prior to using "...
.text:0804C1B7               call    _fwrite
.text:0804C1BC               mov    [esp+18h+var_18], 1
.text:0804C1C3               call    sub_0_804ECE4
.text:0804C1C8               add    esp, 10h
.text:0804C1CB
.text:0804C1CB loc_0_804C1CB:                         ; CODE XREF: sub_0_804C090+117j<=====>这是多少?就是.text:0804C1A7
.text:0804C1CB               push    eax
.text:0804C1CC               push    50h
.text:0804C1CE               push    offset aNcftpdServer ; "NcFTPd Server"
.text:0804C1D3               push    offset byte_0_80984A0
.text:0804C1D8               call    sub_0_806E298
.text:0804C1DD               add    esp, 0Ch
.text:0804C1E0               push    50h
.text:0804C1E2               push    offset aUnregisteredCo ; " (unregistered copy)"<=====>我们将来到这里
```

好了，拿起精神，把你所有的破解细胞调动起来。



尝试一下你在 windows 下的经验，我们往上看

```
.text:0804C1E7               jmp    loc_0_804C10A
.text:0804C1E7 sub_0_804C090  endp
.text:0804C1E7
.text:0804C1EC
```

### 第五步：通过上面的分析我们知道关键点在 `ds:byte_0_8098720` `ds:byte_0_80984A0` 两处地址中变量的赋值情况来决定是否为注册版。

```
.bss:0809871C               db ? ; unexplored
.bss:0809871D               db ? ; unexplored
.bss:0809871E               db ? ; unexplored
.bss:0809871F               db ? ; unexplored
.bss:08098720 byte_0_8098720  db ?; DATA XREF: sub_0_804B1EC+54w<========================>第一处，对其修改的代码地址为.text:0804B240
.bss:08098720                   ; sub_0_804BDEC+Cr ...
.bss:08098721               db ? ; unexplored
.bss:08098722               db ? ; unexplored
.bss:08098723               db ? ; unexplored
    .
    .
    .
.bss:0809849D               db ? ; unexplored
.bss:0809849E               db ? ; unexplored
.bss:0809849F               db ? ; unexplored
.bss:080984A0 byte_0_80984A0  db ?; DATA XREF: sub_0_804B1EC+D2w<===>第二出，对其修改的代码地址为.text:0804B2BE
.bss:080984A0                   ; sub_0_804C090+1Br ...
.bss:080984A1               db ? ; unexplored
.bss:080984A2               db ? ; unexplored
.bss:080984A3               db ? ; unexplored
```

### 第六步：大结局。

```
.text:0804B232               mov    ds:byte_0_8098940, 0
.text:0804B239               mov    ds:byte_0_8098600, 0
.text:0804B240               mov    ds:byte_0_8098720, 0  ;大结局，自己看着办。
.text:0804B247               mov    ds:byte_0_80985C0, 0
.text:0804B24E               mov    ds:byte_0_80987E0, 0
.text:0804B255               mov    ds:byte_0_80982E0, 0
.text:0804B25C               mov    ds:byte_0_8097CE0, 0
.text:0804B263               mov    ds:byte_0_8098580, 0
.text:0804B26A               mov    ds:byte_0_8097FC0, 0
.text:0804B271               mov    ds:byte_0_8098220, 0
.text:0804B278               mov    ds:byte_0_80988C0, 0
.text:0804B27F               mov    ds:byte_0_8098660, 0
.text:0804B286               mov    ds:byte_0_8098500, 0
.text:0804B28D               mov    ds:byte_0_8098120, 0
.text:0804B294               mov    ds:byte_0_8098180, 0
.text:0804B29B               mov    ds:byte_0_80982A0, 0
.text:0804B2A2               mov    ds:byte_0_8098320, 0
.text:0804B2A9               mov    ds:byte_0_8098380, 0
.text:0804B2B0               mov    ds:byte_0_8098420, 0
.text:0804B2B7               mov    ds:byte_0_80980C0, 0
.text:0804B2BE               mov    ds:byte_0_80984A0, 0  ;大结局，自己看着办。
.text:0804B2C5               mov    ds:dword_0_8097D20, 0
.text:0804B2CF               mov    dword_0_8083818, 0
.text:0804B2D9               mov    dword_0_808381C, 0
.text:0804B2E3               mov    dword_0_8083820, 0
.text:0804B2ED               mov    dword_0_8083824, 0Ah
.text:0804B2F7               mov    ds:dword_0_80A3460, 1
.text:0804B301               mov    ds:dword_0_80987C0, 1
.text:0804B30B               mov    ds:dword_0_8099C08, 12Ch
.text:0804B315               mov    dword_0_8083474, 40000h
.text:0804B31F               mov    dword_0_8083478, 40000h
.text:0804B329               mov    ds:dword_0_8098368, 1
.text:0804B333               mov    ds:dword_0_8098700, 0
```

修改完成后把它安装到 unix 下 `/usr/local/sbin/` 目录中配置好脚本运行一下试一试，看是不是搞定了。

完

其实不象 windows 在 unix 下很少遇到需要破解的软件，现在提倡开放源代码，很多商业软件和共享软件都有不错的开源软件来替换（比如 `proftpd` 就不比 `ncftpd` 差），所以 unix 下的破解教程在网上就少了许多。本来这个教程脱胎于我一年前在网上看到的一个老外 linux 逆向工程网站上的一篇破解 `ncftpd` 的文章，不过那是在 linux 下用 `dasm` 反汇编脚本为基础的。今天，在 freebsd 下重装了 `ncftpd`，一时用不了 linux 的 `dasm` 脚本，就试者用 `ida pro`把它给破了。希望大家有机会都来用一下 unix，除了界面不如 windows 软件外，在性能上可以说 unix 绝对是 NO.1 。 4dd466d0e1e0cc123afbe504da2462ca
