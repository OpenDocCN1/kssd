
# [原创]一个驱动木马的分析

【文章标题】: 一个驱动木马的分析  
【文章作者】: prince  
【作者邮箱】: cracker_prince@163.com  
【作者QQ号】: 812937  
【软件名称】: Trojan program Trojan-Downloader.Win32.Agent.bbb(卡巴命名)  
【下载地址】: d92K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2F1M7r3q4U0K9#2)9J5k6h3y4F1i4K6u0r3N6X3W2W2N6%4c8Z5M7X3g2S2k6q4)9J5k6i4m8Z5M7q4)9K6c8Y4c8A6k6q4)9K6c8o6R3%4x3K6y4Q4x3U0k6W2P5s2c8J5j5g2)9K6c8s2m8S2k6$3g2Q4x3U0f1K6c8o6p5`.  
【加壳方式】: 无  
【编写语言】: ---  
【使用工具】: IDA 5.0, WinDbg 6.6.007.5, VMWare 5.5.  
【操作平台】: WINDOWS XP  
【软件介绍】: 某木马or恶意软件的driver部分  
【作者声明】: 水平有限，了解不多，仅作参考。

---

## 前言

大概2个多星期前看到在Unpack『 病毒木马 』区Lvg同学发了这个样本，就用业余时间看了看。本人纯属入门级选手，所以各位看到有什么解释的不妥或者直接让您喷饭地方，还请不吝赐教，谢谢。

## 详细过程

由于是驱动文件，不考虑极端情况，就认为它没壳，直接IDA之。

入口：

```asm
.text:00010F5B ; ***************************************************************
.text:00010F5B ; int __stdcall start(PDRIVER_OBJECT DriverObject,int)
.text:00010F5B                 public start
.text:00010F5B start           proc near
.text:00010F5B DriverObject    = dword ptr  4
.text:00010F5B arg_4           = dword ptr  8
.text:00010F5B                 push    [esp+arg_4]     ; RegistryPath
.text:00010F5F                 call    sub_10966       ; 参数为：RegistryPath；
.text:00010F5F                                         ; 修改SDT，hook关键函数
.text:00010F64                 test    al, al          ; 判断hook是否成功
.text:00010F66                 jnz     short loc_10F6F ; 成功则跳
.text:00010F68                 mov     eax, 0C0000001h ; 否则返回STATUS_UNSUCCESSFUL（0C0000001h）
.text:00010F6D                 jmp     short locret_10FD2
.text:00010F6F loc_10F6F:                              ; CODE XREF: start+Bj
.text:00010F6F                 push    esi
.text:00010F70                 push    edi
.text:00010F71                 xor     edi, edi        ; edi清零
.text:00010F73                 xor     esi, esi        ; esi清零
.text:00010F75                 inc     esi
.text:00010F76                 mov     dword_1160C, esi ; dword_1160C = 1
.text:00010F7C                 mov     dword_11608, edi ; dword_11608 = 0
.text:00010F82                 mov     dword_115FC, esi ; dword_115FC = 1
.text:00010F88                 mov     dword_115F8, edi ; dword_115F8 = 0
.text:00010F8E                 mov     dword_11604, edi ; dword_11604 = 0
.text:00010F94                 mov     dword_11600, edi ; dword_11600 = 0
.text:00010F9A                 mov     dword_11614, edi ; dword_11614 = 0
.text:00010FA0                 mov     Handle, edi     ; Handle = 0
.text:00010FA6                 call    sub_10EFF       ; 创建线程 - 设置注册表注册驱动和打开dll和sys文件
.text:00010FAB                 call    sub_10C61       ; ZwCreateFile打开"\SystemRoot"，判断hook是否成功?
.text:00010FB0                 test    al, al          ; 利用返回值判断是否打开成功
.text:00010FB2                 jz      short loc_10FBC ; 打开失败则跳
.text:00010FB4                 push    esi
.text:00010FB5                 call    sub_106B9       ; 用原始的NtCreateFile打开
.text:00010FB5                                         ; "\SystemRoot\system32\drivers\esqyo.sys" 和
.text:00010FB5                                         ; "\SystemRoot\system32\ydljs.dll"并保存句柄
.text:00010FB5                                         ; 目的是防止这两个文件被关闭和删除
.text:00010FBA                 jmp     short loc_10FC5
.text:00010FBC loc_10FBC:                              ; CODE XREF: start+57j
.text:00010FBC                 push    [esp+8+DriverObject] ; DriverObject
.text:00010FC0                 call    sub_10D1B       ; 如果出现什么问题则调用IoRegisterDriverReinitialization
.text:00010FC0                                         ; 重新再来一遍。狠...
.text:00010FC5 loc_10FC5:                              ; CODE XREF: start+5Fj
.text:00010FC5                 push    edi             ; Remove - edi = 0 （FALSE - 创建hookRoutine）
.text:00010FC6                 push    offset NotifyRoutine ; NotifyRoutine - （监视函数）
.text:00010FC6                                         ; 该函数监视explorer.exe进程启动，跟目标同时启动打开驱动、dll，写注册表等动作
.text:00010FCB                 call    PsSetCreateProcessNotifyRoutine ; 监视进程创建及销毁
.text:00010FD0                 pop     edi
.text:00010FD1                 pop     esi
.text:00010FD2 locret_10FD2:                           ; CODE XREF: start+12j
.text:00010FD2                 retn    8
.text:00010FD2 start           endp
; *****************************************************************************
```

跟进`sub_10966`:

```asm
.text:00010966 ; arg_0
.text:00010966 sub_10966       proc near               ; CODE XREF: start+4p
.text:00010966 arg_0    = dword ptr  8
.text:00010966                 push    ebx             ; arg_0
.text:00010967                 call    sub_10857       ; 对一些常量字符串进行解码还原
.text:0001096C                 xor     ebx, ebx
.text:0001096E                 push    ebx             ; CSDVersion - NULL
.text:0001096F                 push    offset BuildNumber ; BuildNumber
.text:00010974                 push    offset MinorVersion ; MinorVersion
.text:00010979                 push    offset MajorVersion ; MajorVersion
.text:0001097E                 call    PsGetVersion    ; 获取系统版本号，build number
.text:00010983                 call    sub_107F0       ; 在当前进程的EPROCESS结构中搜索字符串"System",
.text:00010983                                         ; 返回的偏移值后面要用到
.text:00010988                 cmp     eax, ebx        ; 检查是否搜索到了指定字符串
.text:0001098A                 mov     dword_118F0, eax ; 保存字符串在EPROCESS结构中的偏移,
.text:0001098A                                         ; 即dword_118F0 = EPROCESS.ImageFileName
.text:0001098F                 jnz     short loc_10998 ; 找到则跳走
.text:00010991                 xor     al, al          ; 如果没找到指定字符串，则al 清零
.text:00010993                 jmp     loc_10A2C       ; 跳到函数结尾返回
.text:00010998 loc_10998:                              ; CODE XREF: sub_10966+29j
.text:00010998                 push    ebp
.text:00010999                 push    esi
.text:0001099A                 push    edi
.text:0001099B                 push    40h             ; size_t
.text:0001099D                 push    ebx             ; int
.text:0001099E                 mov     esi, offset word_11618
.text:000109A3                 push    esi             ; void *
.text:000109A4                 call    memset          ; 变量word_11618清零 （大小为0x40）
.text:000109A9                 mov     edi, ds:wcsncpy
.text:000109AF                 push    1Fh             ; size_t
.text:000109B1                 push    offset s_Esqyo  ; "esqyo"
.text:000109B6                 push    esi             ; wchar_t *
.text:000109B7                 call    edi ; wcsncpy   ; 拷贝字符串"esqyo"至变量word_11618
.text:000109B9                 push    40h             ; size_t
.text:000109BB                 push    ebx             ; int
.text:000109BC                 mov     esi, offset unk_116A0
.text:000109C1                 push    esi             ; void *
.text:000109C2                 call    memset          ; 将变量unk_116a0清零 （大小为0x40）
```
