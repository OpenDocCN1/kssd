

# windows 核心编程学习笔记错误处理及 Unicode 处理！

> 原文：[`www.pediy.com/kssd/pediy12/98914.html`](https://www.pediy.com/kssd/pediy12/98914.html)

偶比较菜，所以列出的是偶觉得该注意的基础细节,特别是红色部分！（接受任何形式的 BS- -)  
1.返回值为 HANDLE： 如果函数运行失败，则返回值通常是 NULL，否则返回值为 HANDLE，用于标识你可以操作的一个对象。注意，有些函数会返回一个句柄值 I NVALID\_ HANDLE\_VALUE，它被定义为- 1。函数的 Platform SDK 文档将会清楚地说明该函数运行失败时返回的是 NULL 还是 INVALID \_ HANDLE \_ VALID！  
  
2.2.BOOL 如果函数运行失败，那么返回值是 0，否则返回的是非 0 值。最好对返回值进行测试，以确定它是 0 还是非 0。不要测试返回值是否为 TRUE!  
  
3.3.PVOID 如果函数运行失败，则返回值是 NULL，否则返回 PVOID，以标识数据块的内存地址!  
  
4. WinError.h 头文件包含了 Microsoft 公司定义的错误代码的列表!  
  
5. 每个错误都有 3 种表示法：一个消息 ID（这是你可以在源代码中使用的一个宏，  
以便与 GetLastError 的返回值进行比较），消息文本（对错误的英文描述）和一个号码（应该避免使用这个号码，可使用消息 ID）！  
  
6.注意 GetLastError 能返回线程产生的最后一个错误。如果该线程调用的 Windows 函数运行成功，那么最后一个错误代码就不被改写，并且不指明运行成功。有少数 Windows 函数并不遵循这一规则，它会更改最后的错误代码；但是 Platform SDK 文档通常指明，当函数运行成功时，该函数会更改最后的错误代码。  
  
7.在 Visual Studio 6.0 的 Watch 窗口中键入“@err, hr”，就可以查看当前线程的最后错误代码！  
  
8.5.当调用 strlen 函数时，它在以 0 结尾的单字节字符数组中返回字符的数目。  
  
9.Unicode 是 Apple 和 Xerox 公司于 1 9 8 8 年建立的一个技术标准。  
  
10.Unicode 字符串中的所有字符都是 1 6 位的（两个字节）。它没有专门的字节来指明下一个字节是属于同一个字符的组成部分，还是一个新字符。这意味着你只需要对指针进行递增或递减，就可以遍历字符串中的各个字符，不再需要调用 CharNext、CharPrev 和 IsDBCSLeadByte 之类的函数。  
  
11.对于用字符串填入缓存的函数来说，系统必须首先将 Unicode 字符串转换成非 Unicode 字符串，然后你的应用程序才能处理该字符串。由于系统必须执行所有这些转换操作，因此你的应用程序需要更多的内存，并且运行的速度比较慢。通过从头开始用 Unicode 来开发应用程序，就能够使你的应用程序更加有效地运行。  
  
12.当 Micrsof t 公司将 OM 从 1 6 位 Windows 转换成 Win32 时，公司作出了一个决定，即需要字符串的所有 COM 接口方法都只能接受 Unicode 字符串。这是个了不起的决定，因为 COM 通常用于使不同的组件能够互相进行通信，而 Unicode 则是传递字符串的最佳手段。  
  
13.只需要定义两个宏（UNICODE 和\_ UNICODE），就可以修改然后重新编译该源文件。  
  
14.请注意，所有的 Unicode 函数均以 wcs 开头，wcs 是宽字符串的英文缩写。若要调用 Unicode 函数，只需用前缀 wcs 来取代 ANSI 字符串函数的前缀 str 即可。  
  
15.当然，标准的 C 运行期字符串函数，如  
strcpy、strch r 和 strcat 等，只能对 ANSI 字符串进行操作，不能正确地处理 Unicode 字符串。  
  
16.标准的 C 头文件 String . h 已经作了修改，  
以便定义一个名字为 wchar\_t 的数据类型，它是一个 Unicode 字符的数据类型。  
  
17.可以创建同时为 ANSI 和 Unicode 进行编译的单  
个源代码文件。若要建立双重功能，必须包含 TChar.h 文件，而不是包含 String.h 文件。  
如果在编译源代码文件时定义了  
\_ UNICODE，这些宏就会引用 wcs 这组函数。如果没有定义\_ UNICODE，那么这些宏将引用 s t r 这组宏。若要定义一个 ANSI /Unicode 通用的字符串数组，请使用下面的 TCHAR 数据类型。  
  
18..\_text 宏。  
  
19. Uincode 数据类型 数据类型说明  
WCHAR    Unicode 字符  
PWSTR    指向 Unicode 字符串的指针  
PCWSTR   指向一个恒定的 Unicode 字符串的指针。  
  
20. Windows 头文件也定义了 ANSI / Unicode 通用数  
据类型 PTSTR 和 P CTSTR。  
  
21.请注意，\_ UNICODE 宏用于 C 运行期头文件，而  
UNICODE 宏则用于 Windows 头文件。当编译源代码模块时，通常必须同时定义这两个宏。  
  
22.建议最好使用操作系统函数，而不要使用 C 运行  
期字符串函数.  
  
23.在经典的操作系统函数样式中，操作系统字符串函数名既包含大写字母，也包含小写字母，  
它的形式类似这个样子： St rCat、StrChr、S trCmp 和 StrCpy 等。若要使用这些函数，必须加上 ShlWApi.h 头文件。  
  
24.即使你不打算立即使用 Unicode，最好也应该着手将你的应用程序转换成符合 Unicode 的应  
用程序。下面是应该遵循的一些基本原则：  
 将文本串视为字符数组，而不是 chars 数组或字节数组。  
 将通用数据类型（如 TCHAR 和 PTSTR）用于文本字符和字符串。  
 将显式数据类型（如 BYTE 和 PBYTE）用于字节、字节指针和数据缓存。  
 将 TEXT 宏用于原义字符和字符串。  
 执行全局性替换（例如用 PTSTR 替换 PSTR）。  
 修改字符串运算问题。例如函数通常希望你在字符中传递一个缓存的大小，而不是字节。  
这意味着你不应该传递 sizeof ( szBuffer ) ,而应该传递（sizeo f ( szBuffer ) / sizeof ( TCHAR )。另外，  
如果需要为字符串分配一个内存块，并且拥有该字符串中的字符数目，那么请记住要按字节来  
分配内存。这就是说，应该调用 malloc(nCharacters \*sizeof(TCHAR)), 而不是调用 malloc  
( nCharacters )。在上面所说的所有原则中，这是最难记住的一条原则，如果操作错误，编译器  
将不发出任何警告。  
  
25.对 Unicode 字符串进行操作的函数  
函数描述  
lstrcat 将一个字符串置于另一个字符串的结尾处  
lstrcmp 对两个字符串进行区分大小写的比较  
lstrcmpi 对两个字符串进行不区分大小写的比较  
lstrcpy 将一个字符串拷贝到内存中的另一个位置  
lstrlen 返回字符串的长度（按字符数来计量）  
这些函数是作为宏来实现的，这些宏既可以调用函数的 Unicode 版本，也可以调用函数的  
ANSI 版本，这要根据编译源代码模块时是否已经定义了 UNICODE 而定。  
  
26.而 Windows 函数 lstrcmp 和 lstrcmp i 是作为对 Windows 函数  
CompareString 的调用来实现的。  
GetThreadString 函数  
每次创建一个线程时，它就被赋予一种语言。函数将返回该线程的当前语言设置。  
  
27. Windows 函数 MultiByteToWideChar 用于将多字节字符串转换成宽字符串。  
  
28.一般来说，可以通过下列步骤将多字节字符串转换  
成 Unicode 等价字符串：  
1) 调用 MultiByteToWideChar 函数，为 pWideCharStr 参数传递 NULL，为 cchWideChar 参数  
传递 0。  
2) 分配足够的内存块，用于存放转换后的 U n i c o d e 字符串。该内存块的大小由前面对  
MultiByteToWideChar 的调用返回。  
3) 再次调用 MultiByteToWideChar，这次将缓存的地址作为 pWideCharStr 参数来传递，并  
传递第一次调用 MultiByteToWideChar 时返回的缓存大小，作为 cchWidechar 参数。  
4. 使用转换后的字符串。  
5) 释放 Unicode 字符串占用的内存块。  
  
^\_^