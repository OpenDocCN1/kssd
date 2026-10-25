
# 一个屏幕录像工具FlashCam&nbsp;(6千字)

**SoftWare:** `Flash Cam 1.68`

是一个屏幕录像工具。

`<http://www.nexusconcepts.com>`

**Tools:** `pe-scan`、`W32Dasm`、`OllyDbg` & 一支笔、一页16开白纸以及微卵的`Win98`

**Cracker:** `lq7972`[`bruceyu13@sina.com`]

**Notes:** 学习学习

用`pe-scan`查壳，是`ASPack`；脱之，存为`Dump.exe`。可以运行。

用`W32Dasm`反汇编，在字符信息中找到`"Registration"`，双击，记下地址。

用`OllyDbg`载入，运行，按`Ctrl+G`，键入`"50ee4c"`，来到下面：

```
* Referenced by a CALL at Addresses:
|:0050F365   , :00516CE0

|
:0050EE4C 55                      push ebp
:0050EE4D 8BEC                    mov ebp, esp
:0050EE4F B909000000              mov ecx, 00000009
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0050EE59(C)

|
:0050EE54 6A00                    push 00000000
:0050EE56 6A00                    push 00000000
;......
;一直到：
* Referenced by a CALL at Address:
|:0050F96E

|
:0050F800 55                      push ebp
:0050F801 8BEC                    mov ebp, esp
:0050F803 83C4EC                  add esp, FFFFFFEC
:0050F806 53                      push ebx
;......
:0050F82A BBDB070000              mov ebx, 000007DB   ebx=7db
;......
:0050F83D BF01000000              mov edi, 00000001   edi=1
;下面是注册算法部分
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0050F888(C)

|
:0050F842 8B45FC                  mov eax, dword ptr [ebp-04] ;公司名，注意换为大写形式了
:0050F845 8A4C38FF                mov cl, byte ptr [eax+edi-01] ;cl=第?位ASCII码
:0050F849 33C0                    xor eax, eax
:0050F84B 8AC1                    mov al, cl
:0050F84D 8D570D                  lea edx, dword ptr [edi+0D];edx=edi+D
:0050F850 F7EA                    imul edx;eax=eax*edx
:0050F852 03D8                    add ebx, eax;ebx=ebx+eax
:0050F854 8BC3                    mov eax, ebx
:0050F856 BBFFC99A3B              mov ebx, 3B9AC9FF ;就是10进制的9个9
:0050F85B 99                      cdq
:0050F85C F7FB                    idiv ebx ;除以这么大的数
:0050F85E 8BDA                    mov ebx, edx
:0050F860 8B45FC                  mov eax, dword ptr [ebp-04]
:0050F863 80F145                  xor cl, 45 ;cl是公司名第?位ASCII码
:0050F866 33C0                    xor eax, eax
:0050F868 8AC1                    mov al, cl
:0050F86A 69C047010000            imul eax, 00000147;eax=eax*147
:0050F870 03D8                    add ebx, eax;ebx=ebx+eax
:0050F872 8BC3                    mov eax, ebx
:0050F874 B9FFC99A3B              mov ecx, 3B9AC9FF
:0050F879 99                      cdq
:0050F87A F7F9                    idiv ecx
:0050F87C 8BDA                    mov ebx, edx
:0050F87E 69C72B300600            imul eax, edi, 0006302B ;eax=edi*6302b
:0050F884 03D8                    add ebx, eax ;ebx=ebx+eax
:0050F886 47                      inc edi ;加1
:0050F887 4E                      dec esi ;减1
:0050F888 75B8                    jne 0050F842 ;循环
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0050F83B(C)

|
:0050F88A 8BC3                    mov eax, ebx
:0050F88C B9FFE0F505              mov ecx, 05F5E0FF ;就是十进制的8个9，数字挺大的
:0050F891 99                      cdq
:0050F892 F7F9                    idiv ecx
:0050F894 8BDA                    mov ebx, edx ;edx是上面除法的余数
:0050F896 8BC3                    mov eax, ebx
:0050F898 B906000000              mov ecx, 00000006
:0050F89D 99                      cdq
:0050F89E F7F9                    idiv ecx ;eax idiv 6，商存eax，余数存edx
:0050F8A0 83C241                  add edx, 00000041 ;(根据下面)余数加上41，转为字符就是第一个注册码
:0050F8A3 8855F7                  mov byte ptr [ebp-09], dl
:0050F8A6 895DF0                  mov dword ptr [ebp-10], ebx
;(根据下面)ebx转换成十进制再换成字符串就是注册码的第二位到第九位-－没有包含每隔三位插入的"-"。另外，如果这里的注册码不足8，在在前面添0，如abcd[F04067495]
:0050F8A9 DB45F0                  fild dword ptr [ebp-10]
:0050F8AC 83C4F4                  add esp, FFFFFFF4
:0050F8AF DB3C24                  fstp tbyte ptr [esp]
:0050F8B2 9B                      wait
:0050F8B3 8D45EC                  lea eax, dword ptr [ebp-14]
:0050F8B6 8A55F7                  mov dl, byte ptr [ebp-09]
:0050F8B9 E8F646EFFF              call 00403FB4
:0050F8BE 8D45EC                  lea eax, dword ptr [ebp-14]
* Possible StringData Ref from Data Obj ->"00-000-000"
                                  |
:0050F8C1 BA0CF95000              mov edx, 0050F90C
:0050F8C6 E8C947EFFF              call 00404094
:0050F8CB 8B45EC                  mov eax, dword ptr [ebp-14]
:0050F8CE 8B55F8                  mov edx, dword ptr [ebp-08]
:0050F8D1 E8D6AFEFFF              call 0040A8AC
:0050F8D6 33C0                    xor eax, eax
:0050F8D8 5A                      pop edx
:0050F8D9 59                      pop ecx
:0050F8DA 59                      pop ecx
:0050F8DB 648910                  mov dword ptr fs:[eax], edx
:0050F8DE 68FBF85000              push 0050F8FB
:0050F8E3 8D45EC                  lea eax, dword ptr ss:[ebp-14]
:0050F8E6 E82145EFFF              call 00403E0C
:0050F8EB 8D45FC                  lea eax, dword ptr ss:[ebp-4]
:0050F8EE E81945EFFF              call 00403E0C
:0050F8F3 C3                      retn
;......
:0050F973 8B45EC                  mov eax, dword ptr ss:[ebp-14] ;注册码
```

## 总结

比较简单，见上。

在跟踪中发现有两个注册码比较特别，应该是万能的：

`926-157-060`

`199-802-143`

## `Flash Cam 1.68` 注册机(Vb6) by `lq7972`

```
'Flash Cam 1.68 注册机(Vb6) by lq7972
Option Explicit

Private Sub Command1_Click()
  Dim i As Integer
  Dim Temp As Double
  Dim Temp1 As Double
  Dim Temp2 As Double
  Dim Temp3 As String
  Temp2 = &H7DB
  For i = 1 To Len(Text1.Text)
    Temp1 = (Temp2 + Asc(UCase(Mid(Text1.Text, i, 1))) * (i + &HD)) Mod &H3B9AC9FF + (Asc(UCase(Mid(Text1.Text, i, 1))) Xor &H45) * &H147
    Temp2 = Temp1 Mod &H3B9AC9FF + i * &H6302B
  Next i
  Temp = Temp2 Mod &H5F5E0FF
  If Len(CStr(Temp)) < 8 Then
    For i = 1 To (8 - Len(CStr(Temp)))
      Temp3 = Temp3 + "0"
    Next i
    Text2.Texet = Chr(Temp Mod &H6 + &H41) + Temp3 + CStr(Temp)
  Else
    Text2.Text = Chr(Temp Mod &H6 + &H41) + CStr(Temp)
  End If
End Sub
```
