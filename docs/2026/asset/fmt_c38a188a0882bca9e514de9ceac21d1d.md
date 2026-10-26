
# FastStone Image Viewer注册算法分析+KeyGen

## 前言

FastStone Image Viewer是一款老牌看图软件，附带编辑/转换图片功能，软件针对个人用户免费，适用于Windows，官方下载：<a href="elink@fe0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3j5i4y4@1M7%4c8G2L8X3g2Q4x3X3g2G2M7X3N6Q4x3V1k6r3f1#2k6A6k6i4N6W2M7V1c8W2N6r3q4A6L8q4)9J5k6h3S2@1L8b7`.`."><mark class="encrypted">359K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3j5i4y4@1M7%4c8G2L8X3g2Q4x3X3g2G2M7X3N6Q4x3V1k6r3f1#2k6A6k6i4N6W2M7V1c8W2N6r3q4A6L8q4)9J5k6h3S2@1L8b7`.`.</mark></a>

出于兴趣，本文以最新版本`v7.5`为例，使用`IDA`和`Windbg`分析注册算法并实现KeyGen。Just for fun：)

## 找验证点

首先查看`FSViewer.exe`，区段/导入表比较完整，没加壳，资源发现`RCData`，可能是Delphi/BC++开发。PE头里`DllCharacteristics`是`0`（无`IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE`），不支持`ASLR`（后面用`IDA`/`Windbg`不用ReBase，直接定位，方便分析）。

![](upload/attach/202003/857678_7GGTENCU7F6EQEF.png)

接下来开始分析注册码判断方式，本文从输入框跟踪用户名引用入手（以前写的<a href="elink@ca0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5%4y4U0x3@1x3H3`.`.">《AudioSrv音频服务故障》</a>有说如何找窗口处理过程）。首先运行程序到注册窗口，`Windbg`附加，输入假码，查看栈回溯。观察之后没有太明显的地方，查看注册框Edit句柄，bp `GetWindowTextW`条件断点未断下，随后bp `NtUserMessageCall`拦截`WM_GETTEXT`，断点命中，查看对应参数获取到用户名，ba r1下内存访问断点，跟踪用户名拷贝，尝试设置多次断点验证，最后定位到关键函数`sub_72F248`。

```
消息参数：
0x000D (13) WM_GETTEXT
函数原型：
NtStatus (*NtUserMessageCall)(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam, ULONG_PTR ResultInfo, DWORD dwType, BOOLEAN bAnsi);
调试前注意函数调用约定，EAX=param1，EDX=param2，ECX=param3，其它参数栈传递。（ADC）
https://en.wikipedia.org/wiki/X86_calling_conventions#cite_note-14
```

## 算法分析

`IDA`加载完毕，Shift+F5打开signature，应用`bcb5rt`（CBuilder 5 runtime），识别到3000多个函数。`sub_72F248`函数代码如下:

```
CODE:0072F282 call @Controls@TControl@GetText$qqrv ; Controls::TControl::GetText(void)
CODE:0072F287 mov eax, [ebp+var_18]
CODE:0072F28A lea edx, [ebp+name] ; 用户名
CODE:0072F28D call @Sysutils@Trim$qqrx17System@AnsiString ; Sysutils::Trim(System::AnsiString)
CODE:0072F292 lea edx, [ebp+var_20]
CODE:0072F295 mov eax, [esi+308h]
CODE:0072F29B call @Controls@TControl@GetText$qqrv ; Controls::TControl::GetText(void)
CODE:0072F2A0 mov eax, [ebp+var_20]
CODE:0072F2A3 lea edx, [ebp+var_1C]
CODE:0072F2A6 call @Sysutils@Trim$qqrx17System@AnsiString ; Sysutils::Trim(System::AnsiString)
CODE:0072F2AB mov eax, [ebp+var_1C] ; 注册码
CODE:0072F2AE lea edx, [ebp+regcode]
CODE:0072F2B1 call @Sysutils@UpperCase$qqrx17System@AnsiString ; Sysutils::UpperCase(System::AnsiString)
CODE:0072F2B6 lea eax, [ebp+var_10] ; 字符串变量 = '' clear
CODE:0072F2B9 call @System@@LStrClr$qqrr17System@AnsiString ; System::__linkproc__ LStrClr(System::AnsiString &)
CODE:0072F2BE mov eax, [ebp+regcode]
CODE:0072F2C1 call strlen ; CBuilder 5 runtime
....
CODE:0072F2FC mov eax, [ebp+var_10]
CODE:0072F2FF call strlen ; CBuilder 5 runtime
CODE:0072F304 cmp eax, 5 ; 5个一组
CODE:0072F307 jz short loc_72F323
CODE:0072F309 mov eax, [ebp+var_10]
CODE:0072F30C call strlen ; CBuilder 5 runtime
CODE:0072F311 cmp eax, 0Bh
CODE:0072F314 jz short loc_72F323
CODE:0072F316 mov eax, [ebp+var_10]
CODE:0072F319 call strlen ; CBuilder 5 runtime
CODE:0072F31E cmp eax, 11h
CODE:0072F321 jnz short loc_72F330
....
CODE:0072F326 mov edx, offset dword_72F70C ; strcat(regcode, '-')
CODE:0072F32B call @System@@LStrCat$qqrv ; System::__linkproc__ LStrCat(void)
```

首先检查注册码是否满足23位，由4组5个大写字母组成，中间横杠分隔，校验时会过滤掉分隔符，最后拆分成8位/8位/4位，(如：ABCDE-FGHIJ-KLMNO-PQRST => ABCDEFG HIJKLMN OPQR)，下面观察两个关键函数。

```
if ( sub_72E770((int)v3, name, code, 0) && sub_72EBFC((int)v3, name, code, 0)
|| sub_72E770((int)v3, name, code, 1) && sub_72EBFC((int)v3, name, code, 1) ) {
// 成功
} else {
// 失败
}
```

由此可知满足其中一种条件即可注册成功，注意函数第四个参数有0和1，具体区别稍后说明。接下来分析`sub_72E770`和`sub_72EBFC`这两个函数。

注：使用`Windbg`调试查看参数及返回值，有些简单的字符串处理，观察数据就能看出来，做个标记即可，就不浪费时间分析了。

`sub_72E770`函数：

```
System::__linkproc__ LStrAddRef(a2);
System::__linkproc__ LStrAddRef(v50);
...
// 大小写转换，StrTrim，StrCat等操作
Sysutils::UpperCase(v51, (_BYTE **)&username);
Sysutils::Trim(v50, &v42);
Sysutils::UpperCase(v42, (_BYTE **)&v43);
...
System::__linkproc__ LStrCat(&v47, v37);
...
// 调试可知，这个函数是字符串拼接
sub_405004(
&v35,
3,
v12,
"me4T6cBLV",
v48,
"CpCwxrvCJZ30pKLu8Svxjhnhut437glCpofVssnFeBh2G0ekUq4VcxFintMix52vL0iJNbdtWqHPyeumkDUC+4AaoSX+xpl56Esonk4=");
// sub_71FB90 需要重点关注
sub_71FB90(*(_BYTE **)(v52 + 788), v35, (int)off_722720); //调用1
// 参数4便是上面说的0 和 1，区别在于连接的字符串不同，生成注册码时选一种即可。
if ( a4 )
{
pop_window(&v32, v48);
v20 = v32;
v19 = *(void **)(*off_98A548[0] + 0x7A8);
pop_window(&v31, v47);
v18 = v31;
System::__linkproc__ WStrCatN(&v33, 3); //连接字符串
System::__linkproc__ LStrFromWStr(&v34, v33); //unicode字符转换
sub_71FB90(*(_BYTE **)(v52 + 784), v34, (int)off_723D50); //调用2
}
else
{
pop_window(&v28, v48);
v20 = v28;
v19 = *(void **)(*off_98A548[0] + 0x7A4);
pop_window(&v27, v47);
v18 = v27;
System::__linkproc__ WStrCatN(&v29, 3); //连接字符串
System::__linkproc__ LStrFromWStr(&v30, v29); //unicode字符转换
sub_71FB90(*(_BYTE **)(v52 + 784), v30, (int)off_723D50); //调用3
}
```

发现`sub_71FB90`函数由3处调用，跟进该函数：

```
CODE:0071FBE3 call dword ptr [edx+40h] ; edx是虚表，hash初始化，obj->init() => sha*_init
CODE:0071FBE6 mov edx, [ebp+var_4]
CODE:0071FBE9 mov eax, edi
CODE:0071FBEB call sub_71FA88
CODE:0071FBF0 mov edx, esi
CODE:0071FBF2 mov eax, edi
CODE:0071FBF4 mov ecx, [eax]
CODE:0071FBF6 call dword ptr [ecx+44h] ; 计算hash值，obj->final() => sha*_final
CODE:0071FBF9 mov eax, edi
CODE:0071FBFB call sub_403E2C
CODE:0071FC00 mov eax, ebx
CODE:0071FC02 mov edx, [eax]
CODE:0071FC04 call dword ptr [edx+38h] ; 获取hash值位数，obj->hash_bits() => sha*_bits
CODE:0071FC07 mov edi, eax
CODE:0071FC09 mov eax, [ebp+var_8] ; 参数1
CODE:0071FC0C call dword ptr [eax+38h] ; 由此可知，参数1也是也是虚表，obj2->key_bits()，返回其函数长度0xA0(160)
CODE:0071FC0F cmp edi, eax ; hash值位数是否大于等于key_bits(160)
CODE:0071FC11 jge short loc_71FC29
CODE:0071FC13 push 0
CODE:0071FC15 mov eax, ebx
CODE:0071FC17 mov edx, [eax]
CODE:0071FC19 call dword ptr [edx+38h] ; sha1 hash值作为输入
CODE:0071FC1C mov ecx, eax
CODE:0071FC1E mov edx, esi
CODE:0071FC20 mov eax, ebx
CODE:0071FC22 mov ebx, [eax]
CODE:0071FC24 call dword ptr [ebx+40h] ;密钥初始化
CODE:0071FC27 jmp short loc_71FC3C
CODE:0071FC29 ; ---------------------------------------------------------------------------
CODE:0071FC29
CODE:0071FC29 loc_71FC29: ; CODE XREF: sub_71FB90+81↑j
CODE:0071FC29 push 0
CODE:0071FC2B mov eax, [ebp+var_8]
CODE:0071FC2E call dword ptr [eax+38h] ;sha512 hash值作为输入
CODE:0071FC31 mov ecx, eax
CODE:0071FC33 mov edx, esi
CODE:0071FC35 mov eax, ebx
CODE:0071FC37 mov ebx, [eax]
CODE:0071FC39 call dword ptr [ebx+40h] ; 密钥初始化
CODE:0071FC3C
CODE:0071FC3C loc_71FC3C: ; CODE XREF: sub_71FB90+97↑j
CODE:0071FC3C mov eax, [ebp+var_8]
CODE:0071FC3F call dword ptr [eax+38h]
```

上面的`obj->init()`由常量特征可知是SHA1和SHA512，对应的hash长度也分别是`0x14`（160位） 和 `0x40`（512位）
