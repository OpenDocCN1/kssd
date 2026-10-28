
`004016CD    56              push    esi`

`004016CE    68 38214000     push    402138                                         ; ASCII "QQ2010"` 

`004016D3    E8 08FDFFFF     call    004013E0                                       ; 通过注册表查找QQ路径,找到路径之后在查找其目录下的tssafeedit.dat文件是否存在`

判断`msimg32.dll`文件是否存在,存在就说明病毒已经运行过了,如果不存在就创建一个,看名称就知道是准备用DLL 劫持技术了!

然后删除`AutoLogin.dat`文件,看名称就可以猜到这是QQ自动登录用的文件

然后判断`"C:\Program Files\Tencent\QQ\Users\All Users\QQ\Registry.db"`文件是否存在,如果存在的话就会执行下面的代码

```
00401BE1    8D4C24 2C       lea     ecx, dword ptr ss:[esp+2C]
00401BE5    51              push    ecx                                            ; ntdll.7C92F641
00401BE6    E8 65FCFFFF     call    00401850                                       ; 29.00401850
```

```
00401BEB    8B15 A4364000   mov     edx, dword ptr ds:[4036A4]                     ; 29.00400000
00401BF1    81C2 18960100   add     edx, 19618
00401BF7    52              push    edx                                            ; ntdll.KiFastSystemCallRet
00401BF8    E8 93000000     call    00401C90                                       ; 打开自己  读入附加数据
```

```
00401C0A    8A1C30          mov     bl, byte ptr ds:[eax+esi]
00401C0D    80F3 73         xor     bl, 73
00401C10    881C30          mov     byte ptr ds:[eax+esi], bl
00401C13    40              inc     eax
00401C14    3D 87850000     cmp     eax, 8587
00401C19  ^ 72 EF           jb      short 00401C0A                                 ; 解密一个放在数据段另一个PE 文件
```

```
00401C44    51              push    ecx                                            ; MSVCRT.77BFC2E3
00401C45    68 87850000     push    8587
00401C4A    50              push    eax
00401C4B    56              push    esi                                            ; 29.0041108C
00401C4C    E8 2F160000     call    00403280                                       ; 解密一个放在数据段另一个PE 文件(第二层)
```

```
00401C51    8B15 B4364000   mov     edx, dword ptr ds:[4036B4]
00401C57    81C2 1C110000   add     edx, 111C
00401C5D    52              push    edx
00401C5E    E8 CDFAFFFF     call    00401730                                       ; 然后在偏移111C 处将读取出来的附加数据覆盖上去
```

```
00401C63    A1 B8364000     mov     eax, dword ptr ds:[4036B8]
00401C68    8B0D B4364000   mov     ecx, dword ptr ds:[4036B4]
00401C6E    50              push    eax
00401C6F    8D9424 30010000 lea     edx, dword ptr ss:[esp+130]
00401C76    51              push    ecx
00401C77    52              push    edx
00401C78    E8 E3FAFFFF     call    00401760                                       ; 这里就是增肥了,随机生成一些字节,循环写入N 次,放在文件尾,可以直接咔嚓掉
```

[ATTACH]57983[/ATTACH]

[ATTACH]57984[/ATTACH]

[ATTACH]57985[/ATTACH]

然后退出,自删除,等QQ运行后加载`msimg32.dll`之后盗取QQ密码!

楼下分析`msimg32`(重点),该木马只需要删除`msimg32.dll`就等于清除了木马,该文件被隐藏,请用`XUETR`等工具删除!

![](https://bbs.kanxue.com/upload/attach/201106/155970_7lfy3u6uf5zc59k.jpg)

![](https://bbs.kanxue.com/upload/attach/201106/155970_pko3ly8abqeacbm.jpg)

![](https://bbs.kanxue.com/upload/attach/201106/155970_rb4blysobxqdzfj.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [3.jpg](javascript:void(0);)
  
  （45.92kb，2015次下载）

- [2.jpg](javascript:void(0);)
  
  （56.09kb，2012次下载）

- [4.jpg](javascript:void(0);)
  
  （17.83kb，2008次下载）

- [5.jpg](javascript:void(0);)
  
  （36.93kb，2010次下载）

- [6.jpg](javascript:void(0);)
  
  （26.06kb，2004次下载）

- [29.rar](javascript:void(0);)
  
  （75.35kb，97次下载）
