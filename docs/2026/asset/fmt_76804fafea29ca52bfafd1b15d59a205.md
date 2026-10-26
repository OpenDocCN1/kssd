
首先我找来RsaKit\_V1.0版本，随机产生一个1024位的密钥，然后输入我们需要注册的用户名，点击加密，得到加密密文，如图：

![](https://bbs.kanxue.com/upload/attach/201507/660136_p01fcxlhdabgu1h.jpg)

此处我们设定的几个关键参数为：

`N`= 6D2E20A939BAB289A52AC6A80BD954FC497EF69369754262513D364DB624D33D902591E443C55448128647E853D4505FDC885788123EEBAB1660066F0E6C4BF520D0D8870F921E79ED63ED049101309635D0E46CCBDE42C67F29DCEA63CF3ABD67AFD81AFFC053AF6CA51D3745B5E97EDAC3AC0B70E48D2CE862AC5823C5F5DD

`E`=1001

`D`= 2849F5537D321FE01B4B05B3CD6B8DA8EB9626778F5AEDFC49EE44B3B89B2E387190EB9D4844F12D0768F5F00DBF3349AC9FC36D3B0E6CF8DD70E1EEE96B9D3B839C43C9EBB4256E6622D1DA9D1EA7E847470643C8C77F3099D384144B653420813FA366F5E0B458EF994FA0D238CA48A0730606767F7B8854958BE926637921

`M`=11ccccccccccc9= 3131636363636363636363636339

`C`= 4EE90DA9A2A43376C529DE6EC71B043A07DF321935B1F50CF8DFC0C0E518DCBE9EE6E120A4C19C4358CF416C2C69260AA89EA5BD29F0CE430D060DF1C666E89EE156B8304B2A8C6447DF03DAC83B888C973526358D498951D271A5D6F29624FC62A7A843A3AB8372811FFCB1223461DF4CC2ECEA5D8762B8720A22B46D7B721B

设计好`N`、`E`、`D`、`M`、`C`之后，那么就是要程序按照我们设计好的计划来运转，下面一步就是要替换程序中的`N` 和`E`，此时本程序的Base64编码相对复杂，我们也可以省去自己去做Base64编码，而可以在程序做完Base64解码后，直接把我们设计的`C`替换，替换时我们可以在此处替换：

```
006B4AE6  PUSH ECX                                 ; /解码后的大小

006B4AE7  MOV EAX,DWORD PTR SS:[EBP-14]            ; |

006B4AEA  PUSH EAX                                 ; |解码后的地址

006B4AEB  MOV EDX,DWORD PTR SS:[EBP+C]             ; |

006B4AEE  PUSH EDX                                 ; |待解码长度

006B4AEF  MOV ECX,DWORD PTR SS:[EBP+8]             ; |

006B4AF2  PUSH ECX                                 ; |待解码地址

006B4AF3  MOV EAX,DWORD PTR SS:[EBP-8]             ; |

006B4AF6  PUSH EAX                                 ; |N 和E存储的地方

006B4AF7  CALL test.0069489C                       ; \test.0069489C
```

对于`N` 和`E` 的替换，我们可以用`WinHEX`打开EXE文件（即附件），通过检索既可以找到`N`的地址为：002FD488（EXE文件偏移地址），然后把上面我们设计好的`N`进行倒序排列，即变为：

`N`= DDF5C52358AC62E82C8DE4700BACC3DA7EE9B545371DA56CAF53C0FF1AD8AF67BD3ACF63EADC297FC642DECB6CE4D0359630019104ED63ED791E920F87D8D020F54B6C0E6F066016ABEB3E12885788DC5F50D453E84786124854C543E49125903DD324B64D363D516242756993F67E49FC54D90BA8C62AA589B2BA39A9202E6D，然后进行替换。

`E`的地址为：002FD488（EXE文件偏移地址）,即将：C7406E69，替换为：00010001

待解码地址我们替换为：4EE90DA9A2A43376C529DE6EC71B043A07DF321935B1F50CF8DFC0C0E518DCBE9EE6E120A4C19C4358CF416C2C69260AA89EA5BD29F0CE430D060DF1C666E89EE156B8304B2A8C6447DF03DAC83B888C973526358D498951D271A5D6F29624FC62A7A843A3AB8372811FFCB1223461DF4CC2ECEA5D8762B8720A22B46D7B721B

这样，我们就完成了RSA KEY的替换。

### 4.2.8 比对用户名

进行完以上的KEY替换之后，我们继续运行程序，来到最后的比对处：

004AF3B0  PUSH EAX                                 ; /String2 => "11ccccccccccc9"

    004AF3B1  MOV EDX,DWORD PTR SS:[EBP-34]            ; |

    004AF3B4  INC EDX

    004AF3B5  PUSH EDX                                 ;  EDX是解码

    004AF3B6  CALL <JMP.&KERNEL32.lstrcmpA>            ; \lstrcmpA

在此处，`EDX`处是存储解码后的最终地址，打开后我们看到：

![](https://bbs.kanxue.com/upload/attach/201507/660136_oso9wbmt782sity.jpg)

确实解密出来之后的数据还是11ccccccccccc9，但是在这个解密数据之前有很多的空字节，如果按照这个结果去比对，结果还是注册失败！因此我们可以通过再给函数006B4A10打补丁的方式能够解决此问题！但是为什么在11ccccccccccc9之前会有这么多的空字节呢？我一直没有找到正确的答案，也很想知道其中的缘由，恳请高人指点！

最后感谢**MistHill**在研究这个软件过程中给予的帮助！

请论坛各位大牛指导！![](https://bbs.kanxue.com/view/img/face/16.gif)

附件： [test.zip](https://bbs.kanxue.com/attach-download-99109.htm)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [register.jpg](javascript:void(0);)

  （44.51kb，4次下载）

- [miyue.jpg](javascript:void(0);)

  （110.78kb，23次下载）

- [jiemi.jpg](javascript:void(0);)

  （30.81kb，5次下载）

- [base64.jpg](javascript:void(0);)

  （32.73kb，4次下载）

- [test.zip](javascript:void(0);)

  （2.05MB，156次下载）
