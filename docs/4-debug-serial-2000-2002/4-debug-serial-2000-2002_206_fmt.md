
# ajpegcompr

> 原文：[`www.pediy.com/kssd/pediy03/forum546.htm`](https://www.pediy.com/kssd/pediy03/forum546.htm)

`http://software.wx88.net/down/ajpegcompr.exe`

JPEG 图片压缩软件，没注册让压了之后不让你存！！！

选'`HELP-->PURCHASE A LICENSE-->TELEPHONE`',他会打开网页，关掉它，再重复一次，就可以输入注册码了！

我更改之后它显示注册成功，一存盘它又出来叫我注册，不知道它在其它的什么地方还有检测，请高人出手！

`Dede` 是个好东东！

现在已可以存盘了，但是在 `About` 里还是没注册！

问题是现在想注册也注册不了！输入注册码的窗口根本出不来！我靠。

```
0049F84C  55                     push    ebp
0049F84D  8BEC                   mov     ebp, esp
0049F84F  6A00                   push    $00
0049F851  6A00                   push    $00
0049F853  53                     push    ebx
0049F854  56                     push    esi
0049F855  8BD8                   mov     ebx, eax
0049F857  33C0                   xor     eax, eax
0049F859  55                     push    ebp

* Possible String Reference to: '開;?腓^\[YY\]脥@'

|
0049F85A  681CF94900            push    $0049F91C
***** TRY
|
0049F85F  64FF30                push    dword ptr fs:\[eax\]
0049F862  648920                mov     fs:\[eax\], esp

* Reference to field `TMainForm.OFFS_07A4`
|
0049F865  80BBA407000000        cmp     byte ptr \[ebx+$07A4\], $00
0049F86C  0F8483000000          jz      0049F8F5
0049F872  8BC3                 mov     eax, ebx

* Reference to: `Forms.TCustomForm.GetActiveMDIChild()`
|
0049F874  E8F769FAFF           call    00446270

* Possible reference to class `TImageForm`
|
0049F879  8B1578874900         mov     edx, \[$498778\]

* Reference to: `System..AsClass()`
|
0049F87F  E8D836F6FF           call    00402F5C
0049F884  8BF0                 mov     esi, eax
0049F886  8D55FC               lea     edx, \[ebp\-$04\]

* Reference to control `TMainForm.FileCloseAction` : `TWindowClose`
|
0049F889  8B8644030000        mov     eax, \[esi+$0344\]

* Reference to: `Sysutils.ExtractFileName(System.AnsiString)`
|
0049F88F  E8248EF6FF           call    004086B8
0049F894  8B55FC               mov     edx, \[ebp\-$04\]

* Reference to control `TMainForm.SaveDialog` : `TSavePictureDialog`
|
0049F897  8B83E0040000         mov     eax, \[ebx+$04E0\]
0049F89D  83C06C               add     eax, +$6C

* Reference to: `System..LStrAsg()`
|
0049F8A0  E8F341F6FF           call    00403A98

* Reference to control `TMainForm.SaveDialog` : `TSavePictureDialog`
|
0049F8A5  8B83E0040000         mov     eax, \[ebx+$04E0\]
0049F8AB  8B10                 mov     edx, \[eax\]

* Reference to method `TSavePictureDialog.Execute()`
|
0049F8AD  FF523C               call    dword ptr \[edx+$3C\]
0049F8B0  84C0                 test    al, al
0049F8B2  744A                 jz      0049F8FE 《《《这里就去购买了，所以不能去！
0049F8B4  8D55F8               lea     edx, \[ebp\-$08\]

* Reference to control `TMainForm.SaveDialog` : `TSavePictureDialog`
|
0049F8B7  8B83E0040000         mov     eax, \[ebx+$04E0\]

* Reference to: `Dialogs.TOpenDialog.GetFileName()`
|
0049F8BD  E84617FBFF           call    00451008
0049F8C2  8B55F8               mov     edx, \[ebp\-$08\]
0049F8C5  33C9                 xor     ecx, ecx
0049F8C7  8BC6                 mov     eax, esi
|
0049F8C9  E83A9AFFFF           call    00499308
0049F8CE  8D55F8               lea     edx, \[ebp\-$08\]

* Reference to control `TMainForm.SaveDialog` : `TSavePictureDialog`
|
0049F8D1  8BB3E0040000         mov     esi, \[ebx+$04E0\]
0049F8D7  8BC6                 mov     eax, esi

* Reference to: `Dialogs.TOpenDialog.GetFileName()`
|
0049F8D9  E82A17FBFF           call    00451008
0049F8DE  8B45F8               mov     eax, \[ebp\-$08\]
0049F8E1  8D55FC               lea     edx, \[ebp\-$04\]

* Reference to: `Sysutils.ExtractFilePath(System.AnsiString)`
|
0049F8E4  E89B8DF6FF           call    00408684
0049F8E9  8B55FC               mov     edx, \[ebp\-$04\]
0049F8EC  8BC6                 mov     eax, esi

* Reference to: `Dialogs.TOpenDialog.SetInitialDir(System.AnsiString)`
|
0049F8EE  E87D17FBFF           call    00451070
0049F8F3  EB09                 jmp     0049F8FE
0049F8F5  33D2                 xor     edx, edx
0049F8F7  8BC3                 mov     eax, ebx

* Reference to : `TMainForm.HelpPurchaseItemClick()`-->模仿单机购买项！
|
0049F8F9  E8E2130000           call    004A0CE0
0049F8FE  33C0                 xor     eax, eax
0049F900  5A                   pop     edx
0049F901  59                   pop     ecx
0049F902  59                   pop     ecx
0049F903  648910               mov     fs:\[eax\], edx
```

现在功能已没限制了，但也不要你输入注册码了！

可是在 `ABOUT` 里还是显示“`Unauthorized`”，不知怎么回事？

看了 `1212` 兄写的破解提示，很有启发，也试用 `dede` 反汇编了一把，

想法和 `1212` 兄稍有不同。

```
procedure TMainForm.FileSaveActionExecute(Sender: TObject);{?}

begin

{
0049F7DC  53                     push    ebx
0049F7DD  56                     push    esi
0049F7DE  8BF2                   mov     esi, edx
0049F7E0  8BD8                   mov     ebx, eax
0049F7E2  8BC6                   mov     eax, esi
0049F7E4  8B1578874900         mov     edx, \[$498778\]
0049F7EA  E85537F6FF           call    00402F44
0049F7EF  84C0                 test    al, al
0049F7F1  740F                 jz      0049F802
0049F7F3  8BC6                 mov     eax, esi
0049F7F5  8B1578874900         mov     edx, \[$498778\]
0049F7FB  E85C37F6FF           call    00402F5C
0049F800  EB12                 jmp     0049F814
0049F802  8BC3                 mov     eax, ebx
0049F804  E8676AFAFF           call    00446270
0049F809  8B1578874900         mov     edx, \[$498778\]
0049F80F  E84837F6FF           call    00402F5C
0049F814  80BBA407000000       cmp     byte ptr \[ebx+$07A4\], $00
0049F81B  7420                 jz      0049F83D    <----------- 此地不能去，要 9090--------
0049F81D  80B87403000000       cmp     byte ptr \[eax+$0374\], $00
0049F824  740B                 jz      0049F831
0049F826  33C9                 xor     ecx, ecx
0049F828  33D2                 xor     edx, edx
0049F82A  E8D99AFFFF           call    00499308
0049F82F  EB15                 jmp     0049F846
0049F831  8BD3                 mov     edx, ebx
0049F833  8BC3                 mov     eax, ebx

* Reference to : `TMainForm.FileSaveAsActionExecute`
|
0049F835  E812000000           call    0049F84C
0049F83A  5E                   pop     esi
0049F83B  5B                   pop     ebx
0049F83C  C3                   ret
0049F83D  33D2                 xor     edx, edx
0049F83F  8BC3                 mov     eax, ebx

* Reference to : `TMainForm.HelpPurchaseItemClick`
|
0049F841  E89A140000           call    004A0CE0
0049F846  5E                   pop     esi
0049F847  5B                   pop     ebx
0049F848  C3                   ret
}

end ;

procedure TMainForm.FileSaveAsActionExecute(Sender: TObject);{?}

begin

{
0049F84C  55                     push    ebp
0049F84D  8BEC                   mov     ebp, esp
0049F84F  6A00                   push    $00
0049F851  6A00                   push    $00
0049F853  53                     push    ebx
0049F854  56                     push    esi
0049F855  8BD8                   mov     ebx, eax
0049F857  33C0                   xor     eax, eax
0049F859  55                     push    ebp

* Possible String Reference to: "開;?腓^\[YY\]脥@"
|
0049F85A  681CF94900            push    $0049F91C
***** TRY
|
0049F85F  64FF30                push    dword ptr fs:\[eax\]
0049F862  648920                mov     fs:\[eax\], esp
0049F865  80BBA407000000        cmp     byte ptr \[ebx+$07A4\], $00
```



## 汇编代码片段

```
0049F86C  0F8483000000             jz      0049F8F5    <------------改成 0F85---------------

0049F872  8BC3                     mov     eax, ebx

0049F874  E8F769FAFF               call    00446270

0049F879  8B1578874900             mov     edx, [$498778]

0049F87F  E8D836F6FF               call    00402F5C

0049F884  8BF0                     mov     esi, eax

0049F886  8D55FC                   lea     edx, [ebp-$04]

0049F889  8B8644030000             mov     eax, [esi+$0344]

0049F88F  E8248EF6FF               call    004086B8

0049F894  8B55FC                   mov     edx, [ebp-$04]

0049F897  8B83E0040000             mov     eax, [ebx+$04E0]

0049F89D  83C06C                   add     eax, +$6C

0049F8A0  E8F341F6FF               call    00403A98

0049F8A5  8B83E0040000             mov     eax, [ebx+$04E0]

0049F8AB  8B10                     mov     edx, [eax]

0049F8AD  FF523C                   call    dword ptr [edx+$3C]

0049F8B0  84C0                     test    al, al

0049F8B2  744A                     jz      0049F8FE

0049F8B4  8D55F8                   lea     edx, [ebp-$08]

0049F8B7  8B83E0040000             mov     eax, [ebx+$04E0]

0049F8BD  E84617FBFF               call    00451008

0049F8C2  8B55F8                   mov     edx, [ebp-$08]

0049F8C5  33C9                     xor     ecx, ecx

0049F8C7  8BC6                     mov     eax, esi

0049F8C9  E83A9AFFFF               call    00499308

0049F8CE  8D55F8                   lea     edx, [ebp-$08]

0049F8D1  8BB3E0040000             mov     esi, [ebx+$04E0]

0049F8D7  8BC6                     mov     eax, esi

0049F8D9  E82A17FBFF               call    00451008

0049F8DE  8B45F8                   mov     eax, [ebp-$08]

0049F8E1  8D55FC                   lea     edx, [ebp-$04]

0049F8E4  E89B8DF6FF               call    00408684

0049F8E9  8B55FC                   mov     edx, [ebp-$04]

0049F8EC  8BC6                     mov     eax, esi

0049F8EE  E87D17FBFF               call    00451070

0049F8F3  EB09                     jmp     0049F8FE

0049F8F5  33D2                     xor     edx, edx

0049F8F7  8BC3                     mov     eax, ebx

* Reference to : TMainForm.HelpPurchaseItemClick

|

0049F8F9  E8E2130000               call    004A0CE0

0049F8FE  33C0                     xor     eax, eax

0049F900  5A                       pop     edx

0049F901  59                       pop     ecx

0049F902  59                       pop     ecx

0049F903  648910                   mov     fs:[eax], edx

****** FINALLY

|

* Possible String Reference to: "^[YY]脥@"

|

0049F906  6823F94900               push    $0049F923

0049F90B  8D45F8                   lea     eax, [ebp-$08]

0049F90E  E83141F6FF               call    00403A44

0049F913  8D45FC                   lea     eax, [ebp-$04]

0049F916  E82941F6FF               call    00403A44

0049F91B  C3                       ret

0049F91C  E95F3BF6FF               jmp     00403480

0049F921  EBE8                     jmp     0049F90B

****** END

|

0049F923  5E                       pop     esi

0049F924  5B                       pop     ebx

0049F925  59                       pop     ecx

0049F926  59                       pop     ecx

0049F927  5D                       pop     ebp

0049F928  C3                       ret

}

end ;
```

## 分析与讨论

如此一来，并没有影响注册的对话框，不过，相信输入正确的注册号以后，估计那个 `OF85` 还需要改成 `0F84`。

BTW: 1212 兄，假如在 `49F8B2` 处改动成 `9090`，注册对话框似乎依旧会跳出，恐怕和下面一条 `49F8F3` 有关？

其实我在实际改的时候并不是如我上篇写的那样只改 `jz` 就行了！我只是为少打几个字才说了一下重点！其实你可以在下面这个 `call dword ptr [edx+83c]` 进去，看看到为什么 `AL` 会等于 1，最后我发出现出来对话框其实决定于

我认为你能让注册窗口跳出来强行跳转的结果，没有挖出它的根！

### 关键指令参考

* Reference to method `TSavePictureDialog.Execute()`

```
0049F8AD  FF523C                   call    dword ptr [edx+$3C]

0049F8B0  84C0                     test    al, al

0049F8B2  744A                     jz      0049F8FE    《《《这里就去购买了，所以不能去！

0049F8B4  8D55F8                   lea     edx, [ebp-$08]
```

其实你可以在上面这个 `call dword ptr [edx+83c]` 进去，看看到为什么 `AL` 会等于 1，最后我发出现出来对话框其实决定于:

```
0049F814  80BBA407000000           cmp     byte ptr [ebx+$07A4], $00
```

你也可以锁定 `ebx+07a4`，看它什么时候变成 1。

`[ebx+$07a4]`、`[ebx+$07a5]` 两个地址处的内容非常重要，你跟进了上面我说的那个 `CALL`!

`[ebx+07a4]` 决定是否功能限制，如是为“1”，它会把 `[ebx+07a5]` 置 0，否则以上两个值就相反。

而 `[ebx+07a5]` 决定是否出来填写注册码的对话框，所以如果你能让 `[ebx+07a4]=1`，`[ebx+07a5]=0` 应该就是注册版了！

## 修改方案

最后我的改法如下：

1. 找: `0F8C65010000E871F1FFFF`

   改: `909090909090----------`

2. 找: `0F9C4F010000C645FF01`

   改: `909090909090--------`

3. 找: `7D04C645FF008B45F0`

   改: `EB----------------`

它们的具体 `offset` 我没记下来！你应该可以找得到，看看这几个地方的代码你就会明白了。

以上是我的观点，敬请指正！呵呵。。。。。。

热烈欢迎继续讨论！ `4dd466d0e1e0cc123afbe504da2462ca`
