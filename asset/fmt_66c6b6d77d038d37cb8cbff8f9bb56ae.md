
## 反汇编代码片段

```
004010EC     0345 0C              add eax,dword ptr ss:[ebp+C]  
004010EF     59                  pop ecx  
004010F0     5A                  pop edx  
004010F1     5F                  pop edi  
004010F2     5B                  pop ebx  
004010F3     5E                  pop esi  
004010F4     C9                  leave  
004010F5     C2 0C00             retn 0C  
```

运用上面的子程序取得 API 函数的格式如下：

```
00401020     68 0736F319         push 19F33607             API 函数特征值  
00401025     FF75 FC             push dword ptr ss:[ebp-4]   API 函数所属的 DLL 的基址  
00401028     6A 00               push 0  
0040102A     E8 B2120000         call 00401021               搜索 API 函数的子程序  
```

运行后 `eax` 就是 API 函数的内存指针

## 反调试实例说明

下面就我编写的一个反调试实例 `EnableWindow`（在看雪论坛查反调试技术实例）做下说明：

这个程序用到了二个函数，`user32.dll` 有 `GetForegroundWindow` 和 `EnableWindow`，当然要取得 `user32.dll` 的基址，须调用 `kernel32.dll` 的 `LoadLibraryA`。

```
004010F8     55                  push ebp  
004010F9     8BEC                mov ebp,esp  
004010FB     83EC 20             sub esp,20  
004010FE     53                  push ebx  
004010FF     56                  push esi  
00401100     57                  push edi  
00401101     E8 FAFEFFFF         call KrnlBase.00401000          ; 取得 kernel32.dll 基址  
00401106     8945 FC             mov dword ptr ss:[ebp-4],eax  
00401109     68 ADD13441         push 4134D1AD                    ; LoadLibraryA 的特征值  
0040110E     FF75 FC             push dword ptr ss:[ebp-4]         ; kernel32.dll 基址  
00401111     6A 00               push 0  
00401113     E8 09FFFFFF         call KrnlBase.00401021           ; 搜索 API 函数的子程序  
00401118     8945 F8             mov dword ptr ss:[ebp-8],eax  
0040111B     E8 07000000         call KrnlBase.00401127           ; 自定位 user32 的地址  
00401120     75 73               jnz short KrnlBase.00401195       ; user32  
00401122     65:72 33            jb short KrnlBase.00401158  
00401125     3200                xor al,byte ptr ds:[eax]  
00401127     FF55 F8             call dword ptr ss:[ebp-8]         ; 调用 LoadLibraryA  
0040112A     8BF0                mov esi,eax  
0040112C     68 3F235577         push 7755233F                     ; GetForegroundWindow 的特征值  
00401131     56                  push esi  
00401132     6A 00               push 0  
00401134     E8 E8FEFFFF         call KrnlBase.00401021  
00401139     8945 F4             mov dword ptr ss:[ebp-C],eax  
0040113C     68 09D99B70         push 709BD909                     ; EnableWindow 的特征值  
00401141     56                  push esi  
00401142     6A 00               push 0  
00401144     E8 D8FEFFFF         call KrnlBase.00401021  
00401149     8945 F0             mov dword ptr ss:[ebp-10],eax  
0040114C     FF55 F4             call dword ptr ss:[ebp-C]         ; user32.GetForegroundWindow  
0040114F     8BF8                mov edi,eax  
00401151     6A 00               push 0                              ; 禁止窗口  
00401153     57                  push edi  
00401154     FF55 F0             call dword ptr ss:[ebp-10]        ; user32.EnableWindow  
00401157     6A 01               push 1                              ; 允许窗口  
00401159     57                  push edi  
0040115A     FF55 F0             call dword ptr ss:[ebp-10]        ; user32.EnableWindow  
0040115D     5F                  pop edi  
0040115E     5E                  pop esi  
0040115F     5B                  pop ebx  
00401160     83C4 20             add esp,20  
00401163     5D                  pop ebp  
00401164     C3                  retn  
```

注意：运行禁止窗口的 `00401154` 之后，整个 `OD` 将被禁止，你只能在任务管理器关闭 `OD`。把禁止窗口和允许窗口一起运行比如在 `40115D` 设断就可以避免。

本程序在 `2003`、`XP` 和 `WINDOWS 7` 运行通过。

### 函数特征值取得的方法

先把 API 函数字符换成二进制代码，如 `EnableWindow`，先把它换成二进制 `456E61626C6557696E646F7700`，注意后面有 `00` 结束，这项工作可以在 `C32ASM` 中进行，然后在 `0040107A` 下断，在数据窗口跟踪 `eax` 的值，并且在数据窗口把 API 函数的二进制字符粘贴进去，然后再在 `004010C5` 下断，`eax` 里的值就是 API 函数的特征值。

## 上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `KrnlBase.rar`  
|
