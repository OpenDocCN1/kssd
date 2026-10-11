
# Advanced Batch Converter v2.3

> 原文：[`www.pediy.com/kssd/pediy03/forum705.htm`](https://www.pediy.com/kssd/pediy03/forum705.htm)

软件名称: `Advanced Batch Converter v2.3`

软件简介: `Advanced Batch Converter` 是你能简单地用来转换标准图形文件的软件，其最大的特点是可以批量转换图形格式，而且支持的文件很多。

输入格式：

`BMP`，`JPG`，`GIF`，`PNG`，`TIF`，`JPEG`，`RLE`，`DIB`，`PCD`，`ICB`，`ICO`，`WMF`，`TIFF`，`TGA`，`PCX`，`SCR`，`EMF`，`JIF`，`VDA`，`JFIF`，`RGB`，`AFI`，`VST`，`胜利`，`CEL`，`JPE`，`RGBA`，`PIC`，`PCC`，`CUT`，`PPM`，`PGM`，`PBM`，`SGI`，`RLA`，`RPF`，`PSD`，`PDD`，`BW`。

输出格式：

`BMP`，`JPG`，`JPEG`，`GIF`，`PNG`，`PCX`，`TGA`，`EPS`，`TIF`，`TIFF`，`ICO`。

另外，也是简单的支持全格式的图象观察器。

软件下载: `http://www.gold-software.com/abc.html?v20`

找到的万能注册码如下（共二十个）：

**万能注册码(1)**
`7447-UGYGJVNY`

**万能注册码(2)**
`GJnb-GHJty-56`

**万能注册码(3)**
`halyava2000#1`

**万能注册码(4)**
`Fg56+H?554-22`

**万能注册码(5)**
`908923GD#23?k`

**万能注册码(6)**
`MNG-KL54?gFRE`

**万能注册码(7)**
`KK^dfg#ed56Rf`

**万能注册码(8)**
`G=Fg?67#23-Ph`

**万能注册码(9)**
`Kl-HGFHGH^fg6`

**万能注册码(10)**
`HHGf#f%jk-56F`

**万能注册码(11)**
`h&tyfFGG567#d`

**万能注册码(12)**
`h&tyfFGG567#d`

**万能注册码(13)**
`KRT-gH-t$23HH`

**万能注册码(14)**
`jne001callHTY`

**万能注册码(15)**
`OOp-^&ghDFFGH`

**万能注册码(16)**
`PMHG-RTYFGH#7`

**万能注册码(17)**
`LJ-FYU%f78?SF`

**万能注册码(18)**
`UPS-GC-^EDGFG`

**万能注册码(19)**
`86fiTYg`

**万能注册码(20)**
`HPPYBRTHDYPTR`

欢迎光临《阿郎居》

`langlirong@yeah.net`

`http://alang.shangdu.net`

你好！就我个人认为（也许不正确），`abc.exe` 软件不是将注册码写入注册表的，而是每次将你的注册码同这二十个注册码相比较，因为将 `abc.exe` 复制之后启动，还是没有注册。因此我认为不是检测注册表中的注册码。

破结过程很简单：

你只要找到 `0053C652` 处，向下看就可以看到依次二十个注册码。

```
:0053C652 8D45F0                      lea eax, dword ptr [ebp-10]

* Possible StringData Ref from Data Obj -> "halyava2000#1"
                                      |

:0053C655 BAB4CB5300                  mov edx, 0053CBB4

:0053C65A E8098AECFF                  call 00405068

:0053C65F 8D55F0                      lea edx, dword ptr [ebp-10]

:0053C662 58                          pop eax

:0053C663 E8A88BECFF                  call 00405210

:0053C668 0F8468040000                je 0053CAD6

:0053C66E 8B06                        mov eax, dword ptr [esi]

:0053C670 8B8084030000                mov eax, dword ptr [eax+00000384]

:0053C676 8B808C000000                mov eax, dword ptr [eax+0000008C]

:0053C67C BA01000000                  mov edx, 00000001

:0053C681 E85621F7FF                  call 004AE7DC

:0053C686 83C010                      add eax, 00000010

:0053C689 50                          push eax

:0053C68A 8D45E0                      lea eax, dword ptr [ebp-20]

* Possible StringData Ref from Data Obj -> "Fg56+H?554-22"
                                      |

:0053C68D BACCCB5300                  mov edx, 0053CBCC

:0053C692 E8D189ECFF                  call 00405068

:0053C697 8D55E0                      lea edx, dword ptr [ebp-20]

:0053C69A 58                          pop eax

:0053C69B E8708BECFF                  call 00405210

:0053C6A0 0F8430040000                je 0053CAD6

:0053C6A6 8B06                        mov eax, dword ptr [esi]

:0053C6A8 8B8084030000                mov eax, dword ptr [eax+00000384]

:0053C6AE 8B808C000000                mov eax, dword ptr [eax+0000008C]

:0053C6B4 BA01000000                  mov edx, 00000001

:0053C6B9 E81E21F7FF                  call 004AE7DC

:0053C6BE 83C010                      add eax, 00000010

:0053C6C1 50                          push eax

:0053C6C2 8D45D0                      lea eax, dword ptr [ebp-30]

* Possible StringData Ref from Data Obj -> "908923GD#23?k"
                                      |

:0053C6C5 BAE4CB5300                  mov edx, 0053CBE4

:0053C6CA E89989ECFF                  call 00405068

:0053C6CF 8D55D0                      lea edx, dword ptr [ebp-30]

:0053C6D2 58                          pop eax

:0053C6D3 E8388BECFF                  call 00405210

:0053C6D8 0F84F8030000                je 0053CAD6

:0053C6DE 8B06                        mov eax, dword ptr [esi]

:0053C6E0 8B8084030000                mov eax, dword ptr [eax+00000384]

:0053C6E6 8B808C000000                mov eax, dword ptr [eax+0000008C]

:0053C6EC BA01000000                  mov edx, 00000001

:0053C6F1 E8E620F7FF                  call 004AE7DC

:0053C6F6 83C010                      add eax, 00000010

:0053C6F9 50                          push eax

:0053C6FA 8D45C0                      lea eax, dword ptr [ebp-40]

* Possible StringData Ref from Data Obj -> "MNG-KL54?gFRE"
                                      |

:0053C6FD BAFCCB5300                  mov edx, 0053CBFC

:0053C702 E86189ECFF                  call 00405068

:0053C707 8D55C0                      lea edx, dword ptr [ebp-40]

:0053C70A 58                          pop eax

:0053C70B E8008BECFF                  call 00405210

:0053C710 0F84C0030000                je 0053CAD6

:0053C716 8B06                        mov eax, dword ptr [esi]

:0053C718 8B8084030000                mov eax, dword ptr [eax+00000384]

:0053C71E 8B808C000000                mov eax, dword ptr [eax+0000008C]

:0053C724 BA01000000                  mov edx, 00000001

:0053C729 E8AE20F7FF                  call 004AE7DC

:0053C72E 83C010                      add eax, 00000010

:0053C731 50                          push eax

:0053C732 8D45B0                      lea eax, dword ptr [ebp-50]

* Possible StringData Ref from Data Obj -> "KK^dfg#ed56Rf"
                                      |

:0053C735 BA14CC5300                  mov edx, 0053CC14

:0053C73A E82989ECFF                  call 00405068

:0053C73F 8D55B0                      lea edx, dword ptr [ebp-50]

:0053C742 58                          pop eax

:0053C743 E8C88AECFF                  call 00405210

:0053C748 0F8488030000                je 0053CAD6

:0053C74E 8B06                        mov eax, dword ptr [esi]

:0053C750 8B8084030000                mov eax, dword ptr [eax+00000384]

:0053C756 8B808C000000                mov eax, dword ptr [eax+0000008C]

:0053C75C BA01000000                  mov edx, 00000001

:0053C761 E87620F7FF                  call 004AE7DC

:0053C766 83C010                      add eax, 00000010

:0053C769 50                          push eax

:0053C76A 8D45A0                      lea eax, dword ptr [ebp-60]

* Possible StringData Ref from Data Obj -> "G=Fg?67#23-Ph"
                                      |

:0053C76D BA2CCC5300                  mov edx, 0053CC2C

:0053C772 E8F188ECFF                  call 00405068

:0053C777 8D55A0                      lea edx, dword ptr [ebp-60]

:0053C77A 58                          pop eax

:0053C77B E8908AECFF                  call 00405210

:0053C780 0F8450030000                je 0053CAD6

:0053C786 8B06                        mov eax, dword ptr [esi]

:0053C788 8B8084030000                mov eax, dword ptr [eax+00000384]

:0053C78E 8B808C000000                mov eax, dword ptr [eax+0000008C]

:0053C794 BA01000000                  mov edx, 00000001

:0053C799 E83E20F7FF                  call 004AE7DC

:0053C79E 83C010                      add eax, 00000010

:0053C7A1 50                          push eax

:0053C7A2 8D4590                      lea eax, dword ptr [ebp-70]

* Possible StringData Ref from Data Obj -> "Kl-HGFHGH^fg6"
                                      |

:0053C7A5 BA44CC5300                  mov edx, 0053CC44

:0053C7AA E8B988ECFF                  call 00405068

:0053C7AF 8D5590                      lea edx, dword ptr [ebp-70]

:0053C7B2 58                          pop eax

:0053C7B3 E8588AECFF                  call 00405210

:0053C7B8 0F8418030000                je 0053CAD6

:0053C7BE 8B06                        mov eax, dword ptr [esi]

:0053C7C0 8B8084030000                mov eax, dword ptr [eax+00000384]

:0053C7C6 8B808C000000                mov eax, dword ptr [eax+0000008C]

:0053C7CC BA01000000                  mov edx, 00000001

:0053C7D1 E80620F7FF                  call 004AE7DC

:0053C7D6 83C010                      add eax, 00000010

:0053C7D9 50                          push eax

:0053C7DA 8D4580                      lea eax, dword ptr [ebp-80]
```



*   Possible `StringData Ref` from `Data Obj` -> `"HHGf#f%jk-56F"`
    |
    ```
    :0053C7DD BA5CCC5300             mov edx, 0053CC5C
    :0053C7E2 E88188ECFF           call 00405068
    :0053C7E7 8D5580               lea edx, dword ptr [ebp-80]
    :0053C7EA 58                    pop eax
    :0053C7EB E8208AECFF           call 00405210
    :0053C7F0 0F84E0020000         je 0053CAD6
    :0053C7F6 8B06                 mov eax, dword ptr [esi]
    :0053C7F8 8B8084030000         mov eax, dword ptr [eax+00000384]
    :0053C7FE 8B808C000000         mov eax, dword ptr [eax+0000008C]
    :0053C804 BA01000000           mov edx, 00000001
    :0053C809 E8CE1FF7FF           call 004AE7DC
    :0053C80E 83C010               add eax, 00000010
    :0053C811 50                    push eax
    :0053C812 8D8570FFFFFF         lea eax, dword ptr [ebp+FFFFFF70]
    ```

*   Possible `StringData Ref` from `Data Obj` -> `"GJnb-GHJty-56"`
    |
    ```
    :0053C818 BA74CC5300           mov edx, 0053CC74
    :0053C81D E84688ECFF           call 00405068
    :0053C822 8D9570FFFFFF         lea edx, dword ptr [ebp+FFFFFF70]
    :0053C828 58                    pop eax
    :0053C829 E8E289ECFF           call 00405210
    :0053C82E 0F84A2020000         je 0053CAD6
    :0053C834 8B06                 mov eax, dword ptr [esi]
    :0053C836 8B8084030000         mov eax, dword ptr [eax+00000384]
    :0053C83C 8B808C000000         mov eax, dword ptr [eax+0000008C]
    :0053C842 BA01000000           mov edx, 00000001
    :0053C847 E8901FF7FF           call 004AE7DC
    :0053C84C 83C010               add eax, 00000010
    :0053C84F 50                    push eax
    :0053C850 8D8560FFFFFF         lea eax, dword ptr [ebp+FFFFFF60]
    ```

*   Possible `StringData Ref` from `Data Obj` -> `"h&tyfFGG567#d"`
    |
    ```
    :0053C856 BA8CCC5300           mov edx, 0053CC8C
    :0053C85B E80888ECFF           call 00405068
    :0053C860 8D9560FFFFFF         lea edx, dword ptr [ebp+FFFFFF60]
    :0053C866 58                    pop eax
    :0053C867 E8A489ECFF           call 00405210
    :0053C86C 0F8464020000         je 0053CAD6
    :0053C872 8B06                 mov eax, dword ptr [esi]
    :0053C874 8B8084030000         mov eax, dword ptr [eax+00000384]
    :0053C87A 8B808C000000         mov eax, dword ptr [eax+0000008C]
    :0053C880 BA01000000           mov edx, 00000001
    :0053C885 E8521FF7FF           call 004AE7DC
    :0053C88A 83C010               add eax, 00000010
    :0053C88D 50                    push eax
    :0053C88E 8D8550FFFFFF         lea eax, dword ptr [ebp+FFFFFF50]
    ```

*   Possible `StringData Ref` from `Data Obj` -> `"090JkTRfgvb@1"`
    |
    ```
    :0053C894 BAA4CC5300           mov edx, 0053CCA4
    :0053C899 E8CA87ECFF           call 00405068
    :0053C89E 8D9550FFFFFF         lea edx, dword ptr [ebp+FFFFFF50]
    :0053C8A4 58                    pop eax
    :0053C8A5 E86689ECFF           call 00405210
    :0053C8AA 0F8426020000         je 0053CAD6
    :0053C8B0 8B06                 mov eax, dword ptr [esi]
    :0053C8B2 8B8084030000         mov eax, dword ptr [eax+00000384]
    :0053C8B8 8B808C000000         mov eax, dword ptr [eax+0000008C]
    :0053C8BE BA01000000           mov edx, 00000001
    :0053C8C3 E8141FF7FF           call 004AE7DC
    :0053C8C8 83C010               add eax, 00000010
    :0053C8CB 50                    push eax
    :0053C8CC 8D8540FFFFFF         lea eax, dword ptr [ebp+FFFFFF40]
    ```

*   Possible `StringData Ref` from `Data Obj` -> `"KRT-gH-t$23HH"`
    |
    ```
    :0053C8D2 BABCCC5300           mov edx, 0053CCBC
    :0053C8D7 E88C87ECFF           call 00405068
    :0053C8DC 8D9540FFFFFF         lea edx, dword ptr [ebp+FFFFFF40]
    :0053C8E2 58                    pop eax
    :0053C8E3 E82889ECFF           call 00405210
    :0053C8E8 0F84E8010000         je 0053CAD6
    :0053C8EE 8B06                 mov eax, dword ptr [esi]
    :0053C8F0 8B8084030000         mov eax, dword ptr [eax+00000384]
    :0053C8F6 8B808C000000         mov eax, dword ptr [eax+0000008C]
    :0053C8FC BA01000000           mov edx, 00000001
    :0053C901 E8D61EF7FF           call 004AE7DC
    :0053C906 83C010               add eax, 00000010
    :0053C909 50                    push eax
    :0053C90A 8D8530FFFFFF         lea eax, dword ptr [ebp+FFFFFF30]
    ```

*   Possible `StringData Ref` from `Data Obj` -> `"jne001callHTY"`
    |
    ```
    :0053C910 BAD4CC5300           mov edx, 0053CCD4
    :0053C915 E84E87ECFF           call 00405068
    :0053C91A 8D9530FFFFFF         lea edx, dword ptr [ebp+FFFFFF30]
    :0053C920 58                    pop eax
    :0053C921 E8EA88ECFF           call 00405210
    :0053C926 0F84AA010000         je 0053CAD6
    :0053C92C 8B06                 mov eax, dword ptr [esi]
    :0053C92E 8B8084030000         mov eax, dword ptr [eax+00000384]
    :0053C934 8B808C000000         mov eax, dword ptr [eax+0000008C]
    :0053C93A BA01000000           mov edx, 00000001
    :0053C93F E8981EF7FF           call 004AE7DC
    :0053C944 83C010               add eax, 00000010
    :0053C947 50                    push eax
    :0053C948 8D8520FFFFFF         lea eax, dword ptr [ebp+FFFFFF20]
    ```

*   Possible `StringData Ref` from `Data Obj` -> `"OOp-^&ghDFFGH"`
    |
    ```
    :0053C94E BAECCC5300           mov edx, 0053CCEC
    :0053C953 E81087ECFF           call 00405068
    :0053C958 8D9520FFFFFF         lea edx, dword ptr [ebp+FFFFFF20]
    :0053C95E 58                    pop eax
    :0053C95F E8AC88ECFF           call 00405210
    :0053C964 0F846C010000         je 0053CAD6
    :0053C96A 8B06                 mov eax, dword ptr [esi]
    :0053C96C 8B8084030000         mov eax, dword ptr [eax+00000384]
    :0053C972 8B808C000000         mov eax, dword ptr [eax+0000008C]
    :0053C978 BA01000000           mov edx, 00000001
    :0053C97D E85A1EF7FF           call 004AE7DC
    :0053C982 83C010               add eax, 00000010
    :0053C985 50                    push eax
    :0053C986 8D8510FFFFFF         lea eax, dword ptr [ebp+FFFFFF10]
    ```

*   Possible `StringData Ref` from `Data Obj` -> `"PMHG-RTYFGH#7"`
    |
    ```
    :0053C98C BA04CD5300           mov edx, 0053CD04
    :0053C991 E8D286ECFF           call 00405068
    :0053C996 8D9510FFFFFF         lea edx, dword ptr [ebp+FFFFFF10]
    :0053C99C 58                    pop eax
    :0053C99D E86E88ECFF           call 00405210
    :0053C9A2 0F842E010000         je 0053CAD6
    :0053C9A8 8B06                 mov eax, dword ptr [esi]
    :0053C9AA 8B8084030000         mov eax, dword ptr [eax+00000384]
    :0053C9B0 8B808C000000         mov eax, dword ptr [eax+0000008C]
    :0053C9B6 BA01000000           mov edx, 00000001
    :0053C9BB E81C1EF7FF           call 004AE7DC
    :0053C9C0 83C010               add eax, 00000010
    :0053C9C3 50                    push eax
    :0053C9C4 8D8500FFFFFF         lea eax, dword ptr [ebp+FFFFFF00]
    ```

*   Possible `StringData Ref` from `Data Obj` -> `"7447-UGYGJVNY"`
    |
    ```
    :0053C9CA BA1CCD5300           mov edx, 0053CD1C
    :0053C9CF E89486ECFF           call 00405068
    :0053C9D4 8D9500FFFFFF         lea edx, dword ptr [ebp+FFFFFF00]
    :0053C9DA 58                    pop eax
    :0053C9DB E83088ECFF           call 00405210
    :0053C9E0 0F84F0000000         je 0053CAD6
    :0053C9E6 8B06                 mov eax, dword ptr [esi]
    ```



```
:0053C9E8 8B8084030000                mov eax, dword ptr [eax+00000384]

:0053C9EE 8B808C000000                mov eax, dword ptr [eax+0000008C]

:0053C9F4 BA01000000                  mov edx, 00000001

:0053C9F9 E8DE1DF7FF                  call 004AE7DC

:0053C9FE 83C010                      add eax, 00000010

:0053CA01 50                          push eax

:0053CA02 8D85F0FEFFFF                lea eax, dword ptr [ebp+FFFFFEF0]

* Possible StringData Ref from Data Obj -> "LJ-FYU%f78?SF"
                                        |

:0053CA08 BA34CD5300                  mov edx, 0053CD34

:0053CA0D E85686ECFF                  call 00405068

:0053CA12 8D95F0FEFFFF                lea edx, dword ptr [ebp+FFFFFEF0]

:0053CA18 58                          pop eax

:0053CA19 E8F287ECFF                  call 00405210

:0053CA1E 0F84B2000000                je 0053CAD6

:0053CA24 8B06                        mov eax, dword ptr [esi]

:0053CA26 8B8084030000                mov eax, dword ptr [eax+00000384]

:0053CA2C 8B808C000000                mov eax, dword ptr [eax+0000008C]

:0053CA32 BA01000000                  mov edx, 00000001

:0053CA37 E8A01DF7FF                  call 004AE7DC

:0053CA3C 83C010                      add eax, 00000010

:0053CA3F 50                          push eax

:0053CA40 8D85E0FEFFFF                lea eax, dword ptr [ebp+FFFFFEE0]

* Possible StringData Ref from Data Obj -> "UPS-GC-^EDGFG"
                                        |

:0053CA46 BA4CCD5300                  mov edx, 0053CD4C

:0053CA4B E81886ECFF                  call 00405068

:0053CA50 8D95E0FEFFFF                lea edx, dword ptr [ebp+FFFFFEE0]

:0053CA56 58                          pop eax

:0053CA57 E8B487ECFF                  call 00405210

:0053CA5C 7478                        je 0053CAD6

:0053CA5E 8B06                        mov eax, dword ptr [esi]

:0053CA60 8B8084030000                mov eax, dword ptr [eax+00000384]

:0053CA66 8B808C000000                mov eax, dword ptr [eax+0000008C]

:0053CA6C BA01000000                  mov edx, 00000001

:0053CA71 E8661DF7FF                  call 004AE7DC

:0053CA76 83C010                      add eax, 00000010

:0053CA79 50                          push eax

:0053CA7A 8D85D0FEFFFF                lea eax, dword ptr [ebp+FFFFFED0]

* Possible StringData Ref from Data Obj -> "86fiTYg"
                                        |

:0053CA80 BA64CD5300                  mov edx, 0053CD64

:0053CA85 E8DE85ECFF                  call 00405068

:0053CA8A 8D95D0FEFFFF                lea edx, dword ptr [ebp+FFFFFED0]

:0053CA90 58                          pop eax

:0053CA91 E87A87ECFF                  call 00405210

:0053CA96 743E                        je 0053CAD6

:0053CA98 8B06  4dd466d0e1e0cc123afbe504da2462ca
```
