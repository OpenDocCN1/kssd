
# [原创]用户态完美实现"变速齿轮"

## 1. 变速齿轮原理

变速齿轮利用 HOOK API 实现针对 Windows 主流应用程序的变速功能，`QueryPerformanceCounter`、`GetTickCount`、`timeGetTime` 3 个 API 实现的完美加速。

（有些程序如《扫雷》是通过 timer 计时的，Hook 这三个函数还不能实现加速）

为了实现变速效果，按键精灵作者采用公式算法实现了加速减速效果，公式如下：

`Result := 上次返回时间 + Round((当前返回时间 - 上次正常时间) * Power(2, 倍数));`

[注：`Round` 表示取整]

原文：f27K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3K6R3@1y4U0b7^5i4K6u0W2K9s2c8E0

根据这个分析，我们写变速齿轮的思路就出来了：

挂钩这三个函数，修改器返回值。完成~![](https://bbs.kanxue.com/view/img/face/41.gif)

## 2. 逆向这三个 Api

### 1. `GetTickCount`

```
mov edx,7FFE0000h
mov eax,dword ptr ds:[edx]
mul dword ptr ds:[edx+4]
shrd eax,edx,18h
retn
```

### 2. `timeGetTime`

```
cmp dword ptr ds:[76B30014],0
jnz GetTickCount
call Func_1
sub eax,dword ptr ds:[76B30018]
push 0
sbb edx,dword ptr ds:[76B3001C]
push 2710
push edx
push eax
call Func_2
add eax,dword ptr ds:[76B30020]
retn

Func_1:
mov edi,edi
mov edx,dword ptr ds:[7FFE000C]
mov eax,dword ptr ds:[7FFE0008]
cmp edx,dword ptr ds:[7FFE0010]
jnz short winmm.76B12B0B
retn

Func_2:
;// 一堆废话,又臭又长...
;// 从第一个"jnz GetTickCount"的跳转来看,
;// 只要让timeGetTime的返回值跟GetTickCount一样即可~
```

### 3. `QueryPerformanceCounter`

```
mov edi,edi
push ebp
mov ebp,esp
push ecx
push ecx
lea eax,dword ptr ss:[ebp-8]
push eax
push dword ptr ss:[ebp+8]
call ntdll.ZwQueryPerformanceCounter   ;// 看这个~
test eax,eax
jl 0xxxxxxxxx
cmp dword ptr ss:[ebp-8],0
je 0xxxxxxxxx
xor eax,eax
inc eax
leave
retn 4

;// ntdll.ZwQueryPerformanceCounter:
;// 这孙子直接进入内核了....
mov eax,0A5
mov edx,7FFE0300
call dword ptr ds:[edx]
retn 8

;// 看win源码知道, 这孙子进内核后又会调用
;// "KeQueryPerformanceCounter"
;// 而这个内核函数做的也只是查询下_KUSER_SHARED_DATA中的东西
```

## 3. 挂钩实现

开始动手 Hook：（部分源码）

```
//////////////////////////////////////////////////////////////////////////
// 1> GetTickCount 的Hook

// 7C80934A kernel32.GetTi>  BA 0000FE7F        mov edx,7FFE0000
// 7C80934F                  8B02               mov eax,dword ptr ds:[edx]
// 7C809351                  F762 04            mul dword ptr ds:[edx+4]
// 7C809354                  0FACD0 18          shrd eax,edx,18
// 7C809358                  C3                 retn

// Hook地址
DWORD HookAddr_GetTickCount = 0x7C809358;

DWORD dwSpead = 0;
DWORD dwTmp = 0;
DWORD dwStartTime = 0;

_declspec(naked) void HookFunc_GetTickCount()
{
     // 记录初始时间.(回忆前面的变速齿轮公式...)
    if (0 == dwStartTime)
    {
        __asm mov dwStartTime, eax
        __asm retn;
    }

     __asm
    {
        mov ecx, eax ;//now time
        sub ecx, dwStartTime //sub time
        //shl ecx, 1
        mov dwTmp, ecx
        push eax
    }

    dwTmp *= dwSpead;//加速多少倍?

    __asm
    {
        pop eax
        mov ecx, dwTmp
        add eax, ecx
        retn
    }
}
```

Ps: 实际上这个 Hook 是不是相当于改写了这个函数呢?![](https://bbs.kanxue.com/view/img/face/41.gif)

下面一个 Hook 也类似~~

```
//////////////////////////////////////////////////////////////////////////
// 2> timeGetTime

// 76B14E4F >  833D 1400B376 0>CMP DWORD PTR DS:[76B30014],0
// 76B14E56    0F85 97770000   JNZ <JMP.&KERNEL32.GetTickCount>
// 76B14E5C    E8 A8DCFFFF     CALL winmm.76B12B09
// 76B14E61    2B05 1800B376   SUB EAX,DWORD PTR DS:[76B30018]
// 76B14E67    6A 00           PUSH 0
// 76B14E69    1B15 1C00B376   SBB EDX,DWORD PTR DS:[76B3001C]
// 76B14E6F    68 10270000     PUSH 2710
// 76B14E74    52              PUSH EDX
// 76B14E75    50              PUSH EAX
// 76B14E76    E8 07000000     CALL winmm.76B14E82
// 76B14E7B    0305 2000B376   ADD EAX,DWORD PTR DS:[76B30020]
// 76B14E81    C3              RETN

// Hook地址:
DWORD HookAddr_timeGetTime = 0x76B14E4F;

_declspec(naked) void HookFunc_timeGetTime()
{
    __asm
    {
        mov edx, 0x7ffe0000
        mov eax, dword ptr ds:[edx]
        mul dword ptr ds:[edx+4]
        shrd eax, edx, 0x18
    }

    // 记录初始时间.(回忆前面的变速齿轮公式...)
    if(0 == dwStartTime)
    {
        __asm mov dwStartTime, eax
        __asm retn;
    }

    __asm
    {
        mov ecx, eax
        sub ecx, dwStartTime
        mov dwTmp, ecx
        push eax
    }
    // dwSpead加速倍数
    dwTmp *= dwSpead;

    __asm
    {
        pop eax
        mov ecx, dwTmp
        add eax, ecx
        retn
    }
}
```

下面最后一个 Hook 了~![](https://bbs.kanxue.com/view/img/face/28.gif)

```
//////////////////////////////////////////////////////////////////////////
// 3> QueryPerformanceCounter

typedef DWORD (__stdcall * pZwQueryPerformanceCounter)(DWORD, DWORD);

pZwQueryPerformanceCounter ZwQueryPerformanceCounter = NULL;

// 这个函数需要在挂钩之前调用一次, 
//  以完成初始化: 找到ZwQueryPerformanceCounter的地址...
void fuck_ZwQueryPerformanceCounter()
{
    HMODULE hModule = LoadLibrary("ntdll.dll");
    if (NULL == hModule)
    {
        //cout << ">load lib error!" << endl;
        return;
    }

    void * p = GetProcAddress(hModule, "ZwQueryPerformanceCounter");
    if (NULL == p)
    {
        //cout << ">get proc addr error!" << endl;
        goto FUCK_END;
    }
    ZwQueryPerformanceCounter = (pZwQueryPerformanceCounter)p;

FUCK_END:
    //FreeLibrary(hModule);
    //hModule = NULL;
    return;
}

//  ---  QueryPerformanceCounter ----
// 7C80A4C7     8BFF             mov edi,edi
// 7C80A4C9  .  55               push ebp
// 7C80A4CA  .  8BEC             mov ebp,esp   <--
// 7C80A4CC  .  51               push ecx
// 7C80A4CD  .  51               push ecx
// 7C80A4CE  .  8D45 F8          lea eax,dword ptr ss:[ebp-8]
// 7C80A4D1  .  50               push eax                                                      ;  接收一个返回
// 7C80A4D2  .  FF75 08          push dword ptr ss:[ebp+8]                                     ;  入口参数地址

// 7C80A4D5  .  FF15 DC13807C    call dword ptr ds:[<&ntdll.NtQueryPerformanceCounter>]        ;  ntdll.ZwQueryPerformanceCounter

// 7C80A4DB  .  85C0             test eax,eax
// 7C80A4DD  .  0F8C AB750300    jl kernel32.7C841A8E                                          ;  非零则跳
// 7C80A4E3  .  837D F8 00       cmp dword ptr ss:[ebp-8],0
// 7C80A4E7  .  0F84 AB750300    je kernel32.7C841A98

// 7C80A4ED  >  33C0             xor eax,eax
// 7C80A4EF  .  40               inc eax
// 7C80A4F0  >  C9               leave
// 7C80A4F1  .  C2 0400          retn 4

DWORD HookAddr_QueryPerformanceCounter = 0x7C80A4C7;

LARGE_INTEGER liStartTime = {0};
LARGE_INTEGER liTmp = {0};
LARGE_INTEGER *lpPerformanceCount = NULL;// QueryPerformanceCounter的形参...
__declspec(naked) BOOL  __stdcall HookFunc_QueryPerformanceCounter()
{
    __asm
    {
        mov edi, edi
        push ebp
        mov ebp, esp
        push ecx
        push ecx
        lea eax, dword ptr [ebp-8]
        push eax
        push dword ptr [ebp+8] ;//形参

        mov eax, ZwQueryPerformanceCounter
        call eax
    }

    //初始化开始时间
    if ((__int64)0 == *(__int64*)&liStartTime)
    {
        __asm
        {
            mov eax, dword ptr[ebp+8]   //lpPerformanceCount
            lea edx, liStartTime

            mov ecx, dword ptr [eax]
            mov dword ptr [edx], ecx
            add eax, 4
            add edx, 4
            mov ecx, dword ptr [eax]
            mov dword ptr [edx], ecx

            // 返回程序
            xor eax, eax
            inc eax
            leave
            retn 4
        }
    }

    ///// 修改查询回来的时间///////
    __asm mov eax, dword ptr [ebp+8]//lpPerformanceCount
    __asm mov lpPerformanceCount, eax

    *(__int64*)&liTmp = *(__int64*)lpPerformanceCount - *(__int64*)&liStartTime;
    *(__int64*)&liTmp *= __int64(dwSpead);  // 加速倍数
    *(__int64*)lpPerformanceCount += *(__int64*)&liTmp;

    ////////////////////////////////
    __asm
    {
        xor eax, eax
        inc eax
        leave
        retn 4
    }
}
```

Ps: 最后一个 Hook 貌似有点复杂....

其实不难~

也就是将这个函数查询回来的系统时间改掉，改成你需要的倍数~
