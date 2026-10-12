
# WinRAR v3.0 Beta 4

> 原文：[`www.pediy.com/kssd/pediy04/kanxue466.htm`](https://www.pediy.com/kssd/pediy04/kanxue466.htm)

**破解工具**：`TRW V1.23`、`W32DASM`、`HIEW v6.81`

首先解决启动 `WINRAR` 在左上会看到 `evaluation copy` 字样、在 `About Winrar` 看到 `40 days trial copy`

*   `Referenced by a (U)nconditional or (C)onditional Jump at Address`

    ```
    |:0043621E(C)
    |
    :00436610 833D9CC5490000          cmp dword ptr [0049C59C], 00000000
    :00436617 754D                   jne 00436666
    :00436619 803D6C4B480000          cmp byte ptr [00484B6C], 00
    :00436620 7544                   jne 00436666
    :00436622 803D14494A0000          cmp byte ptr [004A4914], 00
    :00436629 753B                   jne 00436666
    :0043662B 803D38CC480000          cmp byte ptr [0048CC38], 00  -->>这个就是关键
    :00436632 7532                   jne 00436666
    :00436634 A1D0CE4900             mov eax, dword ptr [0049CED0]
    :00436639 83F828                 cmp eax, 00000028 -->>40 天的限制，向上看
    :0043663C 7F04                   jg 00436642
    :0043663E 85C0                   test eax, eax
    :00436640 7D24                   jge 00436666
    ```

看看是哪里使 `[0048CC38]` 值改变

*   `Reference To: USER32.FlashWindow, Ord:0000h`

    ```
    :0043E13D E8B82B0400             Call 00480CFA
    :0043E142 B001                   mov al, 01
    :0043E144 E8E7D7FCFF             call 0040B930  进去看看啰
    :0043E149 A238CC4800             mov byte ptr [0048CC38], al 这里会改变, 向上看
    :0043E14E 6A00                   push 00000000
    :0043E150 FF35009F4900           push dword ptr [00499F00]
    ```

*   `Referenced by a CALL at Addresses`

    ```
    |:004360C0 , :004362EE , :0043E144 , :0043EA90
    |
    :0040B930 55                     push ebp
    :0040B931 8BEC                   mov ebp, esp
    :0040B933 81C404F0FFFF           add esp, FFFFF004
    :0040B939 50                     push eax
    :0040B93A 81C448FDFFFF           add esp, FFFFFD48
    :0040B940 53                     push ebx
    :0040B941 56                     push esi
    :0040B942 57                     push edi
    :0040B943 8885E3FEFFFF           mov byte ptr [ebp+FFFFFEE3], al
    :0040B949 BE34C64800             mov esi, 0048C634
    ```

*   `Possible Reference to Dialog`

    ```
    :0040B94E B838224800             mov eax, 00482238
    :0040B953 E854770600             call 004730AC
    :0040B958 8D95C8FAFFFF           lea edx, dword ptr [ebp+FFFFFAC8]
    ```

*   `Possible StringData Ref from Data Obj -> "rarreg.*"` 这是 `KeyFile`

    ```
    :0040B95E B85F1E4800             mov eax, 00481E5F
    :0040B963 E815F5FFFF             call 0040AE7D
    :0040B968 84C0                   test al, al
    :0040B96A 7514                   jne 0040B980  如果没有这个文件就跳
    :0040B96C 33C0                   xor eax, eax  就改这里~ 33 C0 =>> B0 01 就是 mov al,1
    :0040B96E 8B95E4FEFFFF           mov edx, dword ptr [ebp+FFFFFEE4]
    :0040B974 64891500000000         mov dword ptr fs:[00000000], edx
    :0040B97B E9CF050000             jmp 0040BF4F
    ```

到这里已解决启动 `evaluation copy` 字样、`40 days trial copy` 字样

接下来是解功能限制 `Put authenticity verification`

`ALT+A` 后，在右下看到 `Archiving options` 中其中一个选项~选中后，

出现 `Available in registered version only`

*   `Referenced by a (U)nconditional or (C)onditional Jump at Address`

    ```
    |:00440D82(C)
    |
    :00440DAA 803D38CC480000          cmp byte ptr [0048CC38], 00
    :00440DB1 7410                   je 00440DC3
    :00440DB3 803D34C6480000          cmp byte ptr [0048C634], 00
    :00440DBA 7507                   jne 00440DC3
    :00440DBC C60538CC480000          mov byte ptr [0048CC38], 00 看~这里被设 0
                                      把 C60538CC480000
                                      改 C60538CC480001
    ```

*   `Referenced by a (U)nconditional or (C)onditional Jump at Addresses`

    ```
    |:00440DB1(C), :00440DBA(C)
    |
    :00440DC3 C605E8484A0001          mov byte ptr [004A48E8], 01
    :00440DCA 803DD9484A0000          cmp byte ptr [004A48D9], 00
    :00440DD1 7410                   je 00440DE3
    :00440DD3 33C9                   xor ecx, ecx
    ```

`4dd466d0e1e0cc123afbe504da2462ca`
