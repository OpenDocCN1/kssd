
### 汇编代码分析

```
00438074    |.  99                     |cdq  
00438075    |.  F7F9                   |idiv ecx  
00438077    |.  8995 40FFFFFF          |mov dword ptr ss:[ebp-C0],edx   ;edx：余数  
             |                         |  
0043807D    |>  FF85 3CFFFFFF          |inc dword ptr ss:[ebp-C4]  
00438083    |.  83BD 3CFFFFFF 05       |cmp dword ptr ss:[ebp-C4],5  ；loop 5 次  
0043808A    |.^ 7C 80                  \jl short 屏录专家.0043800C  
0043808C    |>  83BD 3CFFFFFF 05       |cmp dword ptr ss:[ebp-C4],5  ；是不是第 5 位  
00438093    |.  0F8C AD080000          jl 屏录专家.00438946    ；小于 5 失败  
```

以下检查第 5 位：

```
00438099    |.  8B85 40FFFFFF          mov eax,dword ptr ss:[ebp-C0]  
0043809F    |.  B9 0A000000            mov ecx,0A  
004380A4    |.  99                     cdq  
004380A5    |.  F7F9                   idiv ecx  
004380A7    |.  8B85 3CFFFFFF          mov eax,dword ptr ss:[ebp-C4]  
004380AD    |.  0FBE8C05 D0FEFFFF      movsx ecx,byte ptr ss:[ebp+eax-130];[ebp+eax-130]=第 5 位注册码  
004380B5    |.  83C1 BF               add ecx,-41  
004380B8    |.  2BCA                   sub ecx,edx      ;edx 通过对 temp_H 计算得到  
004380BA    |.  898D 38FFFFFF          mov dword ptr ss:[ebp-C8],ecx  
004380C0    |.  83BD 38FFFFFF 00       cmp dword ptr ss:[ebp-C8],0  
004380C7    |.  74 0D                 je short 屏录专家.004380D6  ；成功  
004380C9    |.  83BD 38FFFFFF 05       cmp dword ptr ss:[ebp-C8],5  
004380D0    |.  0F85 25080000          jnz 屏录专家.004388FB    ;失败  
```

逻辑伪代码：

```c
static char s[14]="xxxxxx"//注册码  

int count          
int buffer  

for count=0 to 5 do  
  {  cmp temp_D[count],(s[count]-14)  
     jnz fail  
  if count=3 do {  
    buffer=mod(((temp_H+4d44h)*3.14*1.59489633173843711e-1)/186a0h)  
            }  
   }  
  buffer=s[count]-41-mod(buffer/0ah)  
  if buffer=0 or buffer=5 jmp succeed  

fail:  
succeed:  

；****************************************************************  
```

成功分支：

```
004380D6    |>  66:C785 5CFFFFFF 4400  mov word ptr ss:[ebp-A4],44  
004380DF    |.  BA 5AB85000            mov edx,屏录专家.0050B85A  
004380E4    |.  8D45 E4               lea eax,dword ptr ss:[ebp-1C]  
004380E7    |.  E8 ACF10A00           call <屏录专家.sub_4E7298>  
004380EC    |.  FF85 68FFFFFF         inc dword ptr ss:[ebp-98]  
004380F2    |.  8B00                 mov eax,dword ptr ds:[eax]  
004380F4    |.  E8 472E0500          call <屏录专家.@Dialogs@ShowMessage$qqrx17System@AnsiString>  
004380F9    |.  FF8D 68FFFFFF         dec dword ptr ss:[ebp-98]  
004380FF    |.  8D45 E4               lea eax,dword ptr ss:[ebp-1C]  
00438102    |.  BA 02000000          mov edx,2  
```

### 破解总结

不能爆破，因为通过注册后他会把注册明肯注册码保存在`pmlxzj.dll`文件里，再次起动时会再检查，

我的机器码为`"92388863"`，输入注册名为`"ABABABAB"`

下断点在：

```
0043800C    |>  8B85 3CFFFFFF          /mov eax,dword ptr ss:[ebp-C4]  
00438012    |.  0FBE9405 18FFFFFF      |movsx edx,byte ptr ss:[ebp+eax-E8];[ebp+eax-0E8h]=temp_D  
```

可见`[ebp+eax-E8]`指向`"31 35 36 35 30 00"`

可得：

`31h+14h=45h->"E"`

`35h+14h=49h->"I"`

`36h+14h=4Ah->"J"`

`35h+14h=49h->"I"`

`30h+14h=44h->"D"`

`"EIJID"`为前 5 位注册码

再下断点在：

```
004380AD    |.  0FBE8C05 D0FEFFFF      movsx ecx,byte ptr ss:[ebp+eax-130]；[ebp+eax-130]=第 5 位注册码  
004380B5    |.  83C1 BF               add ecx,-41  
004380B8    |.  2BCA                   sub ecx,edx   
```

可得到`edx=3`

所以第 5 位注册码为：`3+41h=44h->"D"` 或 `3+41h+5=49h->"I"` 我的注册码为`"EIJIDD"`或`"EIJIDI"`

通过上面输入注册名与注册码后将通过注册，所有限制都取消，生成`*.avi` 已没有未注册字样，

但生成`*.exe` 时还是有未注册字样，通过重新跟踪发现程序重新运行时又读取注册名与注册码，进行了新的计算，

没能跟出来，但我用了另外一种方法来解决这个问题，就是把`"play.dat"`里未注册字样改为空格，

这样使用起来还没有发现什么问题。

`4dd466d0e1e0cc123afbe504da2462ca`
