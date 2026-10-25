
# WebMirror

> 原文：[`www.pediy.com/kssd/pediy03/forum511.htm`](https://www.pediy.com/kssd/pediy03/forum511.htm)

暴力破解 WebMirror

---

软件简介：可以将网站快速下载到本地，以供脱机浏览。它可以提供页面大小和时间的信息。

操作系统:Win95/98/NT/2000

下载地址：http://www.maccasoft.com/webmirror/index.html

破解工具：反汇编工具、编辑器

级别：初级（大笨鸟）

本人初学,如有不正确之处,希望各位大侠给予指正.

本人信箱:langlirong@163.net

---

破解简单过程：

WebMirror 在注册之后，显示注册信息，没有明显的错误提示。

首先打开 w32dsm 装入 WebMirror（好在没有压缩），查找 unregistered，应该有三处还是四处记不准啦，在如下之处注意：

```
:00408261 89C6                   mov esi, eax
:00408263 EB58                   jmp 004082BD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004081AB(C)
|^^^^^^^^^^^^

* Possible StringData Ref from Data Obj ->"Unregistered Evaluation"
                                |
:00408265 BEA9C14200            mov esi, 0042C1A9
:0040826A 8DBC249E000000        lea edi, dword ptr [esp+0000009E]
:00408271 57                    push edi
```

看到画底线处,跳到 004081AB:

```
* Reference To: wmwin.W?GetSerial$:CRegisterDlg$n(pnapna)$CString$$
                                |
:004081A4 E82BA90000            call 00412AD4
:004081A9 85C9                  test ecx, ecx
:004081AB 0F84B4000000          je 00408265    //改 nop
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
:004081B1 89F0                  mov eax, esi
:004081B3 8DBC249E000000        lea edi, dword ptr [esp+0000009E]

* Reference To: wmwin.W?$cv:CString$n()pna
                                |
:004081BA E865DB0000            call 00415D24
:004081BF 89C6                  mov esi, eax
:004081C1 57                    push edi
```

干脆将 004081AB 处的 0F84B4000000 改为 909090909090

保存退出,启动 webmirror 输入用户名和注册码,成功!

慢!为何显示 0 个人使用?看着不舒服,别急!

找到下面之处:

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004081C8(C)
|
:004081DA 5F                    pop edi
:004081DB 83F901                cmp ecx, 00000001
:004081DE 7528                  jne 00408208  //改 je
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Possible StringData Ref from Data Obj ->"1Ue Proa)UlmtdUes% sr"
```

将 jne 改为 je

好啦,该收工啦!

欢迎光临<阿郎据>

langlirong@163.net

http://alang.shangdu.net

10/1/2001  4dd466d0e1e0cc123afbe504da2462ca
