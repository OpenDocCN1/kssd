
```
"GetProcAddress",
"GetModuleHandleA",
0,
"User32.dll",
"GetKeyboardType",
"WindowFromPoint",
0,
"AdvApi32.dll",
"RegQueryValueExA",
"RegSetValueExA",
"StartServiceA",
0,
"Oleaut32.dll",
"SysFreeString",
"CreateErrorInfo",
"SafeArrayPtrOfIndex",
0,
"Gdi32.dll",
"UnrealizeObject",
0,
"Ole32.dll",
"CreateStreamOnHGlobal",
"IsEqualGUID",
0,
"ComCtl32.dll",
"ImageList_SetIconSize",
0,
0,
};
```

Without these API functions, the library cannot be loaded, and moreover the [`DllregisterServer()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/com/html/4442206b-b2ad-47d7-8add-18002c44c5a2.asp) and [`DllUregisterServer()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/com/html/b71137a7-284e-4521-a3b2-9dad9c9d3c54.asp) will not operate. In `CPECryptor::CryptFile`, I have distinguished between `EXE` files and `DLL` files in the initialization of the new import table object during creation:

没有这些 API 函数，将不能加载 `library`，此外，[`DllregisterServer()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/com/html/4442206b-b2ad-47d7-8add-18002c44c5a2.asp)和[`DllUregisterServer()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/com/html/b71137a7-284e-4521-a3b2-9dad9c9d3c54.asp)也不能工作。在 `CPECryptor::CryptFile` 里，我在形成期间，在新导入表对象的初始化过程中区分 `EXE` 文件和 `DLL` 文件：

```
if(( image_nt_headers->FileHeader.Characteristics
     & IMAGE_FILE_DLL ) == IMAGE_FILE_DLL )
{
    ImportTableMaker = new CITMaker( IMPORT_TABLE_OCX );
}
else
{
    ImportTableMaker = new CITMaker( IMPORT_TABLE_EXE );
}
```

## 8 Preserve the Thread Local Storage
## 8 保存TLS

By using Thread Local Storage (TLS), a program is able to execute a multithreaded process, this performance mostly is used by [Borland](http://www.borland.com/) linkers: [Delphi](http://www.borland.com/us/products/delphi/index.html) and [C++ Builder](http://www.borland.com/us/products/cbuilder/index.html). When you pack a `PE` file, you should take care to keep clean the TLS, otherwise, your packer will not support Borland Delphi and C++ Builder linked `EXE` files. To comprehend TLS, I refer you to section 6.7 of the [Microsoft Portable Executable and Common Object File Format Specification](http://www.microsoft.com/whdc/system/platform/firmware/PECOFF.mspx) document, you can observe the TLS structure by `IMAGE_TLS_DIRECTORY32` in `winnt.h`.

通过使用 TLS（Thread Local Storage），程序可以执行多线程，主要是[Borland](http://www.borland.com/)连接器：[Delphi](http://www.borland.com/us/products/delphi/index.html)和[C++ Builder](http://www.borland.com/us/products/cbuilder/index.html)使用它们。当你打包 `PE` 文件时，应该仔细保持 TLS 的清洁，否则，你的 packer 将不支持 Borland Delphi 和 C++ Builder 连接的 `EXE` 文件。为了理解 TLS，我建议你阅读[Microsoft Portable Executable and Common Object File Format Specification](http://www.microsoft.com/whdc/system/platform/firmware/PECOFF.mspx)文档的 6.7 节，你可以用`winnt.h`中的 `IMAGE_TLS_DIRECTORY32` 观察 TLS 结构。

```
typedef struct _IMAGE_TLS_DIRECTORY32 {
    DWORD   StartAddressOfRawData;
    DWORD   EndAddressOfRawData;
    DWORD   AddressOfIndex;
    DWORD   AddressOfCallBacks;
    DWORD   SizeOfZeroFill;
    DWORD   Characteristics;
} IMAGE_TLS_DIRECTORY32, * PIMAGE_TLS_DIRECTORY32;
```

To keep safe the TLS directory, I have copied it in a special place inside the loader:

为了保持 TLS 目录的安全，我把它复制到 `loader` 里特殊的地方：

```
...
_tls_dwStartAddressOfRawData:   dword_type(0xCCCCCCCC)
_tls_dwEndAddressOfRawData:     dword_type(0xCCCCCCCC)
_tls_dwAddressOfIndex:          dword_type(0xCCCCCCCC)
_tls_dwAddressOfCallBacks:      dword_type(0xCCCCCCCC)
_tls_dwSizeOfZeroFill:         dword_type(0xCCCCCCCC)
_tls_dwCharacteristics:        dword_type(0xCCCCCCCC)
...
```

It is necessary to correct the TLS directory entry in the Optional Header:

有必要在 Optional Header 里纠正 TLS 目录的入口：

```
if(image_nt_headers->
   OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_TLS].
   VirtualAddress!=0)
{
    memcpy(&pDataTable->image_tls_directory,
           image_tls_directory,
           sizeof(IMAGE_TLS_DIRECTORY32));

    dwOffset=DWORD(pData1)-DWORD(pNewSection);
    dwOffset+=sizeof(t_DATA_1)-sizeof(IMAGE_TLS_DIRECTORY32);

    image_nt_headers->
      OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_TLS].
      VirtualAddress=dwVirtualAddress + dwOffset;
}
```

## 9 Inject your code
## 9 注入你的代码

We are ready to place our code inside the new section. Our code is a "Hello World!" message by [`MessageBox()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/winui/winui/windowsuserinterface/windowing/dialogboxes/dialogboxreference/dialogboxfunctions/messagebox.asp) from `user32.dll`.

我们准备把我们的代码放入新区段里。我们的代码利用`user32.dll`的[`MessageBox()`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/winui/winui/windowsuserinterface/windowing/dialogboxes/dialogboxreference/dialogboxfunctions/messagebox.asp)显示"Hello World!"消息。

```
...
push MB_OK | MB_ICONINFORMATION
lea eax,[ebp+_p_szCaption]
push eax
lea eax,[ebp+_p_szText]
push eax
push NULL
call _jmp_MessageBox
// MessageBox(NULL, szText, szCaption, MB_OK | MB_ICONINFORMATION);
...
```

**PE Maker - Step 5**

* [Download source files - 71.7 Kb](http://www.codeproject.com/system/inject2exe/pemaker5.zip)
![](img/a1590f5d544abcbc8b45216703e6e0af.jpg)

## 10 Conclusion
## 10 结论

By this article, you have perceived how easily we can inject code to a portable executable file. You can complete the code by using the source of other packers, create a packer in the same way as [Yoda's Protector](http://yodap.sourceforge.net/), and make your packer undetectable by mixing up with [Morphine](http://www.hxdef.org/download.php) source code. I hope that you have enjoyed this brief discussion of one part of the reverse engineering field. See you again in the next discussion!

通过这篇文章，你看到了向 `PE` 文件中注入代码并不难。你可以用其它 packers 的源码完成编码，以[Yoda's Protector](http://yodap.sourceforge.net/)类似的方式创建 packer，通过与[Morphine](http://www.hxdef.org/download.php)源码混合使你的 packer 无法检测。我希望你能喜欢这篇文章，并能再次见到你！

**Ashkbiz Danehkar**

[Click here](http://www.codeproject.com/script/profile/whos_who.asp?vt=arts&id=1599836) to view Ashkbiz Danehkar's online profile.

**Other popular articles:**

* [Driver Development Part 1: Introduction to Drivers](http://www.codeproject.com/system/driverdev.asp)
  This article will go into the basics of creating a simple driver.
* [Driver Development Part 2: Introduction to Implementing IOCTLs](http://www.codeproject.com/system/driverdev2.asp)
  This article will go deeper into the basics of creating a simple driver.
* [Eliminating Explorer's delay when deleting an in-use file](http://www.codeproject.com/system/NoDeleteDelay.asp)
  How to track down and patch an annoyance in Windows Explorer's code.
* [A simple demo for WDM Driver development](http://www.codeproject.com/system/WDM_Driver_development.asp)
  WDM Driver programming introduction with three Pseudo Drivers.
