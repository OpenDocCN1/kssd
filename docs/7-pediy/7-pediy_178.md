
# 求助给 PE 文件增加导出函数

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-819.htm`](https://www.pediy.com/kssd/pediy08/pediy8-819.htm)

看过 `Spring.W` 的 "改造 PE 中的函数为导出函数" 非常兴奋. 有问题请教坛中高手.

（2）构造导出表：

```
My_Export_Table   dd 0 ;Characteristics
My_TimeDateStamp  dd 0   ;TimeDateStamp

                 dw 0   ;MajorVersion
                 dw 0  ;MinorVersion
My_nName         dd My_DLL_nName-ImageBase ;nName
                 dd 1  ;nBase
                 dd 1  ;NumberOfFunctions
                 dd 1    ;NumberOfNames
My_AddressOfFunctions      dd 0x0040A012-ImageBase ;AddressOfFunctions
My_AddressOfNames          dd My_Fun_Name-ImageBase ;AddressOfNames
My_AddressOfNameOrdinals   dd 0  ;AddressOfNameOrdinals

-----------------------------------------------------
My_Fun_Name                db 'MyFunction',0
My_DLL_nName               db 'test.exe',0
```

上面的看不明白, 应该怎样构造导出表?

问题太简单了，可能没有人愿意浪费时间。

你可以通过增加一个段来获得空间，也可以在程序内随便找一个空间来完成，一般都能找到可用空间的。

我将上面给出的表细化了一下，安照实际在内存中的映象来填写，我想应该可以理解了。

```
40A012    MyFunction PROC
           push  ebp
......
......          
           pop   ebp
           ret
40A050    MyFunction ENDP

           ;+++++++++++++++++++ IMAGE_EXPORT_DIRECTORY ++++++++++++++++++++++                      
           IMAGE_EXPORT_DIRECTORY_MYDLL_DLL
40D000 00 00 00 00                dd 0                                      ; Characteristics
40D004 00 00 00 00                dd 0                                      ; TimeDateStamp                 
40D008 00 00                      dw 0                                      ; MajorVersion                
40D00A 00 00                      dw 0                                      ; MinorVersion                
40D00C 28 D0 00 00               dd offset szMyDll_dll  -  offset DOS_HEADER ; Name                      
40D010 01 00 00 00               dd 1                                      ; Base                      
40D014 01 00 00 00               dd 1                                      ; NumberOfFunctions          
40D018 01 00 00 00               dd 1                                      ; NumberOfNames              
40D01C 34 D0 00 00               dd offset AddrOfFun    -  offset DOS_HEADER ; AddressOfFunctions       
40D020 3A D0 00 00               dd offset AddrOfName   -  offset DOS_HEADER ; AddressOfNames           
40D024 3E D0 00 00               dd offset AddrOfOrd    -  offset DOS_HEADER ; AddressOfNameOrdinals     
                                                                          
40D028 4D 79 44 6C szMyDll_dll  db "MyDll.dll",0         
40D02C 6C 2E 64 6C                                      
40D030 6C 00 00 00                                      
40D034 12 A0 00 00 AddOfFun     dd offset MyFunction   -  offset DOS_HEADER ; Pointer to MyFunction
40D038 00 00
40D03A 44 D0 00 00 AddrOfName   dd offset szMyFunction -  offset DOS_HEADER ; Pointer to Function Name String
40D03E 50 D0 00 00 AddrOfOrd    dd offset AddrOrder    -  offset DOS_HEADER ; Pointer to AddressOfNameOrdinals buffer
40D042 00 00
40D044 4D 79 46 75 szMyFunction dd "MyFunction",0
40D048 6E 63 74 69
40D04C 6F 6E 00 00
40D050 00 00        AddrOrder    dw 0
```

另外，你还需要修改 PE 头中的 `Export Directory` 的入口地址，将它指向这个表。同时修改那个 `Size` 值。这里

```
VirtualAddress = D000
Size           = 54
```

4dd466d0e1e0cc123afbe504da2462ca
