
# [原创]CTF不好用的CE瞎搞

最近趁上课之余，刷了一下CTF(<mark class="encrypted">c67K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3g2T1N6h3N6C8N6g2)9J5k6h3y4G2L8g2)9J5c8X3y4Z5j5h3I4D9k6h3&6Y4k6i4y4Q4x3U0V1`.</mark>

不好用的`CE`，言外之意就是说这道题用`CE`不好做，这里先分享一下网上其他大佬的解题方式：<mark class="encrypted">e3eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2y4e0q4U0N6r3!0Q4x3X3g2U0L8$3#2Q4x3V1j5I4x3K6V1&6x3U0b7^5y4g2)9J5c8U0t1@1x3U0V1#2y4U0V1`.</mark>

## 我的解题方式

### 1. 先打开程序，观察

双击程序，出现一个对话框

![图片描述](upload/attach/202006/889500\_JWFNAPDXZJAENGT.png)

之后是主界面，点击`command`可以是值增加1

![图片描述](upload/attach/202006/889500\_A9BNG2FX556MQQE.png)

这里我的思路：

1、爆破：写一个程序，模拟点击10000次
2、尝试修改值，直接修改到9999，再次操作

这里，我果断选择第二个方式（其实第一个试了一下，比较内存了，CPU 也一下子就飚上去了）

首先，查看有没有壳（一般都是无壳的，但是还是查看一下）

![图片描述](upload/attach/202006/889500\_8DAKD2WHAKZY932.png)

如图:程序是`VB`写的，无壳，nice

### 2. CE附加，尝试查看变量

这里我尝试搜索精确数值，但是发现精确值收缩到最后是一个死胡同，因为有时候是可以查找到这些值，但更多时候，这些值可能只是那么几个值才产生变动，基本上从精确值这里切入比较难，而且即使找到了值，并且进行了修改，在这次显示值是修改了的，但是到了下一次时，值也会自动校正。所以我尝试搜索未知初始值，并且通过不断变动值，缩小搜索范围，最终查到以下数据：

![图片描述](upload/attach/202006/889500\_2856VK7BFQMM94E.png)

分别对这些数据进行改写查看，获得以下代码段

```
数据1：
00401D41 - 8B 16 - mov edx,[esi]
00401D43 - 56 - push esi
00401D44 - DD 5E 34 - fstp qword ptr [esi+34] <<
00401D47 - DFE0 - fnstsw ax
00401D49 - A8 0D - test al,0D
数据2：
77E82C6C - 85 C0 - test eax,eax
77E82C6E - 0F84 89C00100 - je ntdll.RtlpUnWaitCriticalSection+7F
77E82C74 - FF 87 FC000000 - inc [edi+000000FC] <<
77E82C7A - C6 45 E6 01 - mov byte ptr [ebp-1A],01
77E82C7E - 8B 75 08 - mov esi,[ebp+08]
77E83105 - 85 C0 - test eax,eax
77E83107 - 0F84 13BC0100 - je ntdll.RtlpUnWaitCriticalSection+A2
77E8310D - FF 83 FC000000 - inc [ebx+000000FC] <<
77E83113 - F7 43 48 00000060 - test [ebx+48],60000000
77E8311A - 0F85 F7000100 - jne ntdll.RtlPcToFileHeader+1DC
数据3：
77E8313B - 75 30 - jne ntdll.RtlQueryPerformanceCounter+7E9
77E8313D - 05 02000100 - add eax,00010002
77E83142 - 89 46 04 - mov [esi+04],eax <<
77E83145 - 39 7D 90 - cmp [ebp-70],edi
77E83148 - 0F85 914E0000 - jne ntdll.LdrUnlockLoaderLock+100D
77E82CDB - 83 F9 02 - cmp ecx,02
77E82CDE - 72 04 - jb ntdll.RtlQueryPerformanceCounter+360
77E82CE0 - 83 40 04 FE - add dword ptr [eax+04],-02 <<
77E82CE4 - F6 46 02 08 - test byte ptr [esi+02],08
77E82CE8 - 0F85 9FEA0100 - jne ntdll.TpDisassociateCallback+652
数据4：
77E82F07 - 3B D8 - cmp ebx,eax
77E82F09 - 0F85 2A7A0500 - jne ntdll.RtlUlonglongByteSwap+DBE9
77E82F0F - 89 02 - mov [edx],eax <<
77E82F11 - 89 4A 04 - mov [edx+04],ecx
77E82F14 - 89 11 - mov [ecx],edx
数据5：
72942039 - 03 F2 - add esi,edx
7294203B - 2B C2 - sub eax,edx
7294203D - 89 37 - mov [edi],esi <<
7294203F - 89 47 04 - mov [edi+04],eax
72942042 - 8D 04 7F - lea eax,[edi+edi*2]
数据6：
7294203B - 2B C2 - sub eax,edx
7294203D - 89 37 - mov [edi],esi
7294203F - 89 47 04 - mov [edi+04],eax <<
72942042 - 8D 04 7F - lea eax,[edi+edi*2]
72942045 - 8D 04 80 - lea eax,[eax+eax*4]
数据7：
779B9962 - 89 44 8F F8 - mov [edi+ecx*4-08],eax
779B9966 - 8B 44 8E FC - mov eax,[esi+ecx*4-04]
779B996A - 89 44 8F FC - mov [edi+ecx*4-04],eax <<
779B996E - 8D 04 8D 00000000 - lea eax,[ecx*4+00000000]
779B9975 - 03 F0 - add esi,eax
数据8：
779B9962 - 89 44 8F F8 - mov [edi+ecx*4-08],eax
779B9966 - 8B 44 8E FC - mov eax,[esi+ecx*4-04]
779B996A - 89 44 8F FC - mov [edi+ecx*4-04],eax <<
779B996E - 8D 04 8D 00000000 - lea eax,[ecx*4+00000000]
779B9975 - 03 F0 - add esi,eax
```
