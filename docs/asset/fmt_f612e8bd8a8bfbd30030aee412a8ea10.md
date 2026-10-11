
## 注册码验证算法分析

算法部分先对注册码左边 21 个字符使用了两次查表变换（`BASE32`?），得到一个 18 个字符的字符串。然后对这 18 个字符取 `md5`，得到 `md5` 字符串左边 4 个字符，替换掉其中的 `'0'` 和 `'1'` 后转换成大写，和注册码后 4 个字符比较。

然后用 18 个字符的字符串变换出下面的 key 结构：
*   `0-4` : 未知
*   `5-8` : `>= 8`
*   `9-12` : `0`
*   `13-16` : `0`
*   `17-20` : 协议类型: `1`:Single `2` Multi `3` Academic
*   `21-26` : `0`
*   `27-31` : `1`
*   `32-45` : 未知，形成一整数
*   `46-59` : 随意

这不是一个完整的分析，我没有时间继续去追踪了。老板要发彪了 :(

感兴趣的朋友可以继续分析启动时的验证。

=======================================================================================

【全文完】

## MFC 程序断点调试经验

哈哈，刚才看了些，我那样找 `Button` 断点还是适合这 `MFC` 程序

虽然这是 `VC 7.0` 写的，不过同样可以在 `MFC71.DLL` 库中搜索

```
PUSH    DWORD PTR [EAX+14]
PUSH    DWORD PTR [EBP+C]
PUSH    DWORD PTR [EBP+8]
PUSH    EDI
```

会来到这里

```
7C1718FD    FF75 14          PUSH    DWORD PTR [EBP+14]
7C171900    FF70 10          PUSH    DWORD PTR [EAX+10]
7C171903    FF75 10          PUSH    DWORD PTR [EBP+10]
7C171906    FF70 14          PUSH    DWORD PTR [EAX+14]
7C171909    FF75 0C          PUSH    DWORD PTR [EBP+C]
7C17190C    FF75 08          PUSH    DWORD PTR [EBP+8]
7C17190F    57               PUSH    EDI                         ; WINSPOOL.72F735D8
7C171910    E8 9FFFFFFF      CALL    7C1718B4                    ; MFC71.7C1718B4
```

`7C171900    FF70 10          PUSH    DWORD PTR [EAX+10]` 是消息代号， 在没点 `'ok'` 按钮前，老有个 `0x39` 消息，屏蔽掉它，条件断点 `[EAX+10]!=39`

再去点 `'ok'` 按钮，就会断下来了

`7C171906    FF70 14          PUSH    DWORD PTR [EAX+14]`
内的值显示是 `7C1500F0` 位置

再过去几步到了这，大概是 `CDialog` 类的 `OnOk()` 和 `OnCancal` 虚函数

```
7C1477EC    56               PUSH    ESI                         ; MFC71.7C170060
7C1477ED    6A 01            PUSH    1
7C1477EF    8BF1             MOV     ESI, ECX
7C1477F1    E8 EE5E0000      CALL    7C14D6E4                    ; MFC71.7C14D6E4
7C1477F6    85C0             TEST    EAX, EAX                    ; SnagIt32.008650B8
7C1477F8    74 09            JE      SHORT 7C147803              ; MFC71.7C147803
7C1477FA    6A 01            PUSH    1
7C1477FC    8BCE             MOV     ECX, ESI                    ; MFC71.7C170060
7C1477FE    E8 02000000      CALL    7C147805                    ; MFC71.7C147805
7C147803    5E               POP     ESI                         ; MFC71.7C1746CC
7C147804    C3               RETN
```

往 `CALL    7C14D6E4` 里走。。。。

里边的 `CALL    [EAX+FC]` 函数就是去往 `Snaglt32.exe` 判断注册部分了
