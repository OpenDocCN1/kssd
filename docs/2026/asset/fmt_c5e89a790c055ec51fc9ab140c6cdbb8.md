
# Ultra Video Splitter 6.4.1208 算法分析

【文章标题】Ultra Video Splitter 6.4.1208算法分析

【原版下载】<mark class="encrypted">7c5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4G2L8X3g2Q4x3X3c8K6L8$3k6@1i4K6u0W2j5$3!0E0</mark>

【保护方式】序列号

【分析过程】

## 1.关键代码定位

随便输入一组用户名和密码，点击“确定”，弹出“无效注册用户名或注册码”

![](upload/attach/201802/770162_5KUM9XHPC2FZ4FE.png)

可以以此为突破口，OD中载入搜索字符串，但是搜索相关关键字，并没有找到任何有用信息。怀疑是加密了，或者“语言包”(我也是一点一点摸索出来的，很多国外的软件都是这么搞的。没思路时，可以去安装目录下碰碰运气)

![](upload/attach/201802/770162_MXMA6UBM6WCBXXZ.png)

进入安装目录，找到“Languages”文件夹，找到一堆语言包

![](upload/attach/201802/770162_74ZSDGAA3CTWT8X.png)

其中有这个，那么在OD中搜索“Invalid License”，就可以快速定位到关键点了

![](upload/attach/201802/770162_XR3MV73NJ8MFY7R.png)

定位到以下代码段：

```
.text:0040DEFC mov [esp+18h], eax
.text:0040DF00 mov [esp+38h], ebx
.text:0040DF04 cmp dword_4F9FD0, ebx ; 关键点在这个全局变量,ebx在上面已清零
.text:0040DF0A jnz loc_40E076 ; 跳转到注册成功，否则注册失败
.text:0040DF10 push ecx
.text:0040DF11 mov ecx, esp
.text:0040DF13 mov [esp+24h], esp
.text:0040DF17 push offset aNotFoundError ; "Not Found Error"
.text:0040DF1C call sub_4057E0
.text:0040DF21 push ecx
.text:0040DF22 mov byte ptr [esp+40h], 1
.text:0040DF27 mov ecx, esp
.text:0040DF29 mov [esp+2Ch], esp
.text:0040DF2D push offset aInvalidLicense ; "Invalid License"
.text:0040DF32 call sub_4057E0
.text:0040DF37 push ecx
.text:0040DF38 mov byte ptr [esp+44h], 2
.text:0040DF3D mov ecx, esp
.text:0040DF3F mov [esp+34h], esp
.text:0040DF43 push offset aDlg ; "Dlg"
```

IDA中参考引用查看是那里修改了这个全局变量，重点关注这两个`mov`指令

![](upload/attach/201802/770162_FXCGBFJPSHMWCCZ.png)

来到下面这段代码，大致功能是：

1. 先调用`CreateFileMappingA()`创建共享内存区域
2. 再调用`CreateProcessA()`创建进程`avm.exe`，其中命令行参数是" -sfm Ultra_Video_Splitter -checkreg"
3. 具体的算法验证模块是在`avm.exe`进程中完成的。算法验证完成后，会将验证结果写入到之前创建的“内存共享区”中
4. 最终根据“内存共享区”中的结果判断是否注册成功

```
.text:00434AC9 mov dword ptr [esi], offset ??_7CSFMServer@@6B@
.text:00434ACF mov [esi+4], ebx
.text:00434AD2 mov [esi+8], ebx
.text:00434AD5 mov [esi+0Ch], ebx
.text:00434AD8 mov [esi+10h], ebx
.text:00434ADB mov [esi+14h], ebx
.text:00434ADE mov [esi+18h], ebx
.text:00434AE1 mov [esi+1Ch], ebx
.text:00434AE4 call sub_430010 ; 该函数中创建文件映射，共享内存
.text:00434AE9 jmp short loc_434AED
.text:00434AEB ; ---------------------------------------------------------------------------
.text:00434AEB
.text:00434AEB loc_434AEB:
.text:00434AEB xor esi, esi
.text:00434AED
.text:00434AED loc_434AED:
.text:00434AED mov [esp+82Ch+var_810], esi
.text:00434AF1 mov esi, [esi+0Ch]
.text:00434AF4 cmp esi, ebx
.text:00434AF6 jz short loc_434AA1
.text:00434AF8 mov [esp+82Ch+var_818], esi
.text:00434AFC jmp short loc_434B02
.text:00434AFE ; ---------------------------------------------------------------------------
.text:00434AFE
.text:00434AFE loc_434AFE:
.text:00434AFE mov [esp+82Ch+var_818], eax
.text:00434B02
.text:00434B02 loc_434B02:
.text:00434B02 mov edi, [esp+82Ch+var_818]
.text:00434B06 add edi, 10h
.text:00434B09 call _appsys_regdata_ini@0
.text:00434B0E push eax ; Src
.text:00434B0F push 400h ; SizeInBytes
.text:00434B14 push edi ; Dst
.text:00434B15 call _strcpy_s ; 将data.ini文件路径复制到共享内存
.text:00434B1A add esp, 0Ch
.text:00434B1D push offset byte_4F9D80 ; 字符串“Ultra_Video_Splitter”
.text:00434B22 lea eax, [esp+830h+Dest]
.text:00434B26 push offset aSfmSCheckreg ; " -sfm %s -checkreg"
.text:00434B2B push eax ; Dest
.text:00434B2C mov [esp+838h+Dest], bl
.text:00434B30 call _sprintf ; 格式化命令行参数为" -sfm Ultra_Video_Splitter -checkreg"，作为CreateProcessA()函数的参数
.text:00434B35 add esp, 0Ch
.text:00434B38 lea esi, [esp+82Ch+hHandle]
.text:00434B3C call sub_430550 ; 获取当前文件路径“C:\Program Files\Ultra Video Splitter\”
.text:00434B41 mov ecx, [eax]
.text:00434B43 push offset aAvmExe ; "avm.exe"
.text:00434B48 push ecx
.text:00434B49 lea edx, [esp+834h+ApplicationName]
.text:00434B50 push offset aSS_0 ; "%s%s"
.text:00434B55 push edx ; Dest
.text:00434B56 call _sprintf ; 格式化为："C:\Program Files\Ultra Video Splitter\avm.exe"，作为CreateProcessA的参数
.text:00434B5B mov eax, [esp+83Ch+hHandle]
.text:00434B5F add eax, 0FFFFFFF0h
.text:00434B62 add esp, 10h
.text:00434B65 lea ecx, [eax+0Ch]
.text:00434B68 or edx, 0FFFFFFFFh
.text:00434B6B lock xadd [ecx], edx
.text:00434B6F dec edx
.text:00434B70 test edx, edx
.text:00434B72 jg short loc_434B7E
.text:00434B74 mov ecx, [eax]
.text:00434B76 mov edx, [ecx]
.text:00434B78 push eax
.text:00434B79 mov eax, [edx+4]
.text:00434B7C call eax
.text:00434B7E
.text:00434B7E loc_434B7E:
.text:00434B7E lea ecx, [esp+82Ch+hHandle]
.text:00434B82 push ecx ; int
.text:00434B83 mov ecx, dword_4FA4DC
.text:00434B89 lea edx, [esp+830h+Dest]
.text:00434B8D push edx ; lpCommandLine=" -sfm Ultra_Video_Splitter -checkreg"
.text:00434B8E lea eax, [esp+834h+ApplicationName]
.text:00434B95 push eax ; lpApplicationName="C:\Program Files\Ultra Video Splitter\avm.exe"
.text:00434B96 call sub_432750 ; 函数内部调用CreateProcessA创建进程
.text:00434B9B add esp, 0Ch
.text:00434B9E test eax, eax
.text:00434BA0 jz short loc_434BE0
.text:00434BA2 mov esi, [esp+82Ch+hHandle]
.text:00434BA6 mov ebp, ds:WaitForSingleObject
.text:00434BAC push 1 ; dwMillisecondsA
.text:00434BAF call ebp ; WaitForSingleObject
.text:00434BB1 test eax, eax
.text:00434BB3 jz short loc_434BC3
.text:00434BB5
.text:00434BB5 loc_434BB5:
.text:00434BB5 call sub_432600
.text:00434BBA push 1 ; dwMilliseconds
.text:00434BBC push esi ; hHandle
.text:00434BBD call ebp ; WaitForSingleObject
.text:00434BBF test eax, eax
.text:00434BC1 jnz short loc_434BB5
.text:00434BC3
.text:00434BC3 loc_434BC3:
.text:00434BC3 push edi ; edi指向之前创建的文件映射的一块内存，存储是否注册成功的标志
.text:00434BC4 call j__atol
.text:00434BC9 mov ebp, eax
.text:00434BCB add esp, 4
.text:00434BCE cmp ebp, ebx
.text:00434BD0 jnz short loc_434C22 ; 跳转到失败，不跳则成功
.text:00434BD2 mov eax, 1
.text:00434BD7 mov dword_4F9FD0, eax ; 全局变量赋值为1表示成功
.text:00434BDC mov [esp+82Ch+var_814], eax
.text:00434BE0
.text:00434BE0 loc_434BE0:
.text:00434BE0 cmp [esp+82Ch+var_818], ebx
.text:00434BE4 jz short loc_434BF5
.text:00434BE6 push 400h ; Size
.text:00434BEB push ebx ; Val
.text:00434BEC push edi ; Dst
.text:00434BED call _memset
.text:00434BF2 add esp, 0Ch
.text:00434BF5
.text:00434BF5 loc_434BF5:
.text:00434BF5 mov ecx, [esp+82Ch+var_810]
.text:00434BF9 cmp ecx, ebx
.text:00434BFB jz short loc_434C05
.text:00434BFD mov edx, [ecx]
.text:00434BFF mov eax, [edx]
.text:00434C01 push 1
.text:00434C03 call eax
.text:00434C05
.text:00434C05 loc_434C05:
.text:00434C05 mov ecx, [esp+82Ch+var_8]
.text:00434C0C mov eax, [esp+82Ch+var_814]
.text:00434C10 pop edi
.text:00434C11 pop esi
.text:00434C12 pop ebp
.text:00434C13 pop ebx
.text:00434C14 xor ecx, esp
.text:00434C16 call @__security_check_cookie@4
.text:00434C1B add esp, 81Ch
.text:00434C21 retn
.text:00434C22 ; ---------------------------------------------------------------------------
.text:00434C22
.text:00434C22 loc_434C22: ;走到这里注册失败
.text:00434C22 mov dword_4F9FD0, ebx
.text:00434C28 call _appsys_regdata_ini@0 ; appsys_regdata_ini()
.text:00434C2D mov esi, eax
.text:00434C2F call sub_430EE0
.text:00434C34 cmp ebp, 2
.text:00434C37 jnz short loc_434BE0
.text:00434C39 jmp loc_434C40
```

这里需要知道创建文件映射时的“对象名称”，也就是下图的“Ultra_Video_Splitter”，后面在调试`avm.exe`时会用到

![](upload/attach/201802/770162_7XYJHVW6BQC2PRT.png)

接下来进入`avm.exe`分析，OD载入（注意输入命令行参数）。因为使用的是文件映射的方式实现的内存共享，所以在`avm.exe`中指定会打开内存共享对象，所以我们可以对`OpenFileMappingA()`下断点，坐等它来

![](upload/attach/201802/770162_DXFTAMPAC4Q4WKY.png)

下面将调用`MapViewOfFile()`将一个文件映射对象映射到当前应用程序的地址空间，注意返回值就是共享内存的首地址

![](upload/attach/201802/770162_TG68WZ66FE2QANT.png)

到此就可以在验证注册是否成功的那个标志下内存写入断点，等着看是那里修改了该标志。

内存断点断下来之后，中间需要几次回溯，才能定位到注册验证的关键函数（在`AppSys.dll`中）

```
.text:10001D8B push edx ; 假码
.text:10001D8C push eax ; 用户名
.text:10001D8D call _appsys_check@8 ; 算法校验函数
```

大致看一下这个注册函数
