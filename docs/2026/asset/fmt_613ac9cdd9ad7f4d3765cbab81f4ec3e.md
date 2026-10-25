
# [原创]IDA －－ idc delphi 类表的建立

IDA -- idc delphi 类表的建立

原创：dietrich_teickner@arcor.de  
修改：gzgzlxg@hotmail.com

IDA分析delphi程序时，对类表（包括vmt表），不能获得很好的效果，给分析delphi程序带来许多困难，在IDA中使用手工分析这些类表，工作量之大不可用言语表达，而且要求分析者熟悉delphi的类表结构。

最近在网上下载了一个搜索delphi类表的 IDC 脚本，对这个脚本做了一些改动，使脚本能够建立清晰的类表，创建类表名和注解。IDC 脚本是类C语法，所以就不在多加解释。有不明白处请读脚本。

这个脚本能够正确的建立所有的类表delphi字符串，类表的命名因为手头没有这些资料可能不正确，如果你有这方面的资料可以自行修改，当然最好给我返回一份，这也是做人的起码道德。分析一个完整的delphi需要几分钟时间，但对代码和一些C字符串、Unicode字符串不做处理，这个还需要你自己用手工或其他辅助的IDC去解决。另外经过整理后，在使用IDA的分析命令对程序重新分析，因为有了类表，IDA就能很好的分析所有的代码了。

在使用过程中如果发现bug，请发mail给我。

## 说明

IDA 的脚本语言有许多bug和缺陷，尤其是循环语句，结果经常叫你哭笑不得，所以这个脚本的循环语句都是用递减的，但如果你使用其他的方法，会有什么结果你自己试过就知道了（4.7-4.8版，4.9版我没有用过不清楚）。还有一些函数形同虚设，返回的结果有很大的随意性，所以对C字符串和Unicode字符串没有做处理。

祝你使用愉快。

gzgzlxg  
2006-02-27

## 下面是一些分析实例

这是IDA分析完成后的结果。

```text
CODE:00401000 CODE            segment para public 'CODE' use32
CODE:00401000                 assume cs:CODE
CODE:00401000                 ;org 401000h
CODE:00401000                 assume es:nothing, ss:nothing, ds:nothing, fs:nothing, gs:nothing
CODE:00401000                 add     al, 10h
CODE:00401002                 inc     eax
CODE:00401003                 add     [ebx], al
CODE:00401005                 pop     es
CODE:00401006                 inc     edx
CODE:00401007                 outsd
CODE:00401008                 outsd
CODE:00401009                 ins     byte ptr es:[edi], dx
CODE:0040100A                 db      65h
CODE:0040100A                 popa
CODE:0040100C                 outsb
CODE:0040100D                 add     [eax], eax
CODE:0040100F                 dd 1000000h, 0, 5004010h, 736C6146h, 72540465h
CODE:00401023                 jnz     short loc_40108A
CODE:00401025
CODE:00401025 loc_401025:                             ; CODE XREF: HEADER:00400F53j
CODE:00401025                 lea     eax, [eax+0]
CODE:00401028                 sub     al, 10h
CODE:0040102A                 inc     eax
CODE:0040102B                 add     [edx], al
CODE:0040102D                 add     al, 43h
CODE:0040102F                 push    17261h
CODE:00401034                 dd 0FF000000h, 90000000h, 401040h, 6D530801h, 696C6C61h
CODE:00401034                 dd 2746Eh, 0FFFFFF80h, 9000007Fh, 401058h, 6E490701h, 65676574h
CODE:00401034                 dd 472h, 0FFFF8000h, 0C08B7FFFh, 401070h, 79420401h, 16574h
CODE:00401034                 dd 0FF000000h, 90000000h, 401084h, 6F570401h
CODE:00401088                 jb      short loc_4010EE
CODE:0040108A
CODE:0040108A loc_40108A:                             ; CODE XREF: CODE:00401023j
CODE:0040108A                 add     eax, [eax]
CODE:0040108C                 dd 0FF000000h, 900000FFh, 401098h, 61430801h, 6E696472h
CODE:0040108C                 dd 56C61h, 0FF000000h, 90FFFFFFh
CODE:004010AC dword_4010AC    dd 4010B0h              ; DATA XREF: sub_40E95C+65r
CODE:004010AC                                         ; sub_40E95C+A4r ...
CODE:004010B0                 dd 7453060Ah, 676E6972h, 401104h, 7 dup(0), 401104h, 4
CODE:004010B0                 dd 0, 403F90h, 403F9Ch
CODE:004010EC                 db 0A0h, 3Fh
CODE:004010EE
CODE:004010EE loc_4010EE:                             ; CODE XREF: CODE:00401088j
CODE:004010EE                 inc     eax
CODE:004010EF                 add     [edi+edi+3F980040h], ah
CODE:004010F6                 inc     eax
CODE:004010F7                 add     al, ah
CODE:004010F9                 cmp     al, 40h
CODE:004010FB                 add     ah, bh
CODE:004010FD                 cmp     al, 40h
CODE:004010FF                 add     [eax], bh
CODE:00401101                 cmp     eax, 54070040h
CODE:00401106                 dec     edi
CODE:00401107                 bound   ebp, [edx+65h]
CODE:0040110A                 arpl    [eax+edx+11h], si
CODE:0040110E                 inc     eax
CODE:0040110F                 add     [edi], al
CODE:00401111                 pop     es
CODE:00401112                 push    esp
CODE:00401113                 dec     edi
CODE:00401114                 bound   ebp, [edx+65h]
CODE:00401117                 arpl    [esp+eax+11h], si
CODE:0040111B                 inc     eax
CODE:0040111C                 dd 0, 6000000h, 74737953h, 6D65h, 401130h, 49490A0Fh, 7265746Eh
CODE:0040111C                 dd 65636166h, 0, 1, 0, 0C000h, 0, 79530646h, 6D657473h
CODE:0040111C                 dd 0FFFF0003h, 244483CCh, 0C5E9F804h, 83000053h, 0F8042444h
CODE:0040111C                 dd 53E3E9h, 24448300h, 0EDE9F804h, 0CC000053h, 40115DCCh
CODE:0040111C                 dd 40116700h, 40117100h, 100h, 2 dup(0), 0C000h, 0, 40117D46h
CODE:0040111C                 dd 800h, 0, 408D00h, 4011F8h, 401189h, 6 dup(0), 4011F8h
CODE:0040111C                 dd 0Ch, 4010B8h, 403F90h, 406500h, 40650Ch, 403FA4h, 403F98h
CODE:0040111C                 dd 40651Ch, 403CFCh, 403D38h, 6E495411h, 66726574h, 64656361h
CODE:0040111C                 dd 656A624Fh, 0C08B7463h, 401210h, 44540904h, 54657461h
CODE:0040111C                 dd 1656D69h
```

这是经过IDC整理之后：
