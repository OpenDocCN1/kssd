
# 两种方法搞定 ACProtect 1.09(Anti-Debug + Code Splicing + Stolen Code)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-581.htm`](https://www.pediy.com/kssd/pediy08/pediy8-581.htm)

**【文章标题】**: 两种方法搞定 ACProtect 1.09(Anti-Debug + Code Splicing + Stolen Code)  
**【文章作者】**: wynney  
**【软件名称】**: Calories  
**【下载地址】**: 打包下载  
**【保护方式】**: Anti-Debug + Code Splicing + Stolen Code  
**【编写语言】**: Borland Delphi 4.0 - 5.0  
**【操作平台】**: XP SP2  
**【作者声明】**: 凑热闹~^\_^

***

## 一、前言

前几天在某国外论坛上看到某老外求助脱这个软件，遂下载下来瞧了瞧，蛮有点意思。很巧合，在看雪论坛上同样看到了某兄弟关于 Anti-Debug 求助，突然想起了这个软件。对于这个有点心得，于是，抽空就写这篇文章。这里说的两种方法，其实只是前面的反 Anti-Debug + 找 Stolen Code + 找伪 OEP 的方法不一样而已，后面修复 IAT、处理 Code Splicing 的方法是一样的。所以我将把两种方法的反 Anti-Debug + 找 Stolen Code + 找伪 OEP 的过程分开来讲，修复 IAT、处理 Code Splicing 的方法放在最后一起讲

## 二、方法一

### 2.1、反 Anti-Debug + 找 Stolen Code + 找伪 OEP

使用看雪老大的 `HideOD` 插件，设置如下：

1. 选上：`Process32Next`，`HideNtDebutBit`，`None`
2. 忽略除了内存访问、指定异常外的所有异常(也就是说这 2 个不打勾)

`OD` 载入目标程序，`HideOD` 插件点下 `HIDE`，单步 `F8` 一下

```
00645000 C>  60                  pushad                         ; EP,F8
00645001     85F5               test ebp,esi                   ; ESP=0012FFA4
00645003     46                  inc esi
00645004     E8 01000000        call Calories.0064500A
00645009   - 73 83              jnb short Calories.00644F8E
hr 0012FFA4
Shit+F9，中断在最后一次内存访问异常
```

```
0064B08E     CD 01              int 1                          ; 中断在最后一次内存访问异常
0064B090     40                  inc eax
0064B091     40                  inc eax
0064B092     0BC0               or eax,eax
0064B094     0F85 B6000000      jnz Calories.0064B150
注意看堆栈
```

```
0012FF58  /0012FFE0  指针到下一个 SEH 记录
0012FF5C  |0064B072  SE 句柄                        ；右键--转存中跟随，看转存窗口
0012FF60  |53C89D53
```

```
0064B072  0C245C8B                          ；右键--断点--内存访问，Shit+F9 中断下来
0064B076  00B88383
0064B07A  33020000
```

```
0064B072     8B5C24 0C         mov ebx,dword ptr ss:[esp+C]    ; 中断在这，F2，继续 Shit+F9
0064B076     8383 B8000000 02  add dword ptr ds:[ebx+B8],2
0064B07D     33C0               xor eax,eax
0064B07F     C3                 retn
```

```
0064B177     8B048E            mov eax,dword ptr ds:[esi+ecx*4]    ; 中断在这，F2，继续 Shit+F9
0064B17A     8B5C8E 04         mov ebx,dword ptr ds:[esi+ecx*4+4]
0064B17E     2BC3               sub eax,ebx
```

```
0064B18B     89048E            mov dword ptr ds:[esi+ecx*4],eax    ; 中断在这，此时取消 2 个 F2 断点，1 个内存访问断点，在 retn 处 F4
0064B18E     49                 dec ecx
0064B18F   ^ EB E1            jmp short Calories.0064B172
0064B191     61                 popad
0064B192     61                 popad
0064B193     C3                 retn                            ; 3 个断点取消完毕后，F4
中断在 retn 后，Ctrl+T-->命令是一个-->push ebp-->Ctrl+F11 运行跟踪，由于我们开始下的 hr esp 的缘故会中断在 Stolen Code 处
```

```
0065FAF3     55                  push ebp                     ; Stolen Code 第一句
0065FAF4     8BEC                mov ebp,esp                  ; Stolen Code 第二句
0065FAF6     90                  nop
0065FAF7     90                  nop
0065FAF8     90                  nop
0065FAF9     60                  pushad
0065FAFA     60                  pushad
0065FAFB     E8 00000000         call Calories.0065FB00
55 8B EC
删除硬件断点，Alt+M-->00401000 段 F2-->Shift+F9 中断在伪 OEP
```

```
00569498     42                  inc edx                        ; OEP
00569499     44                  inc esp
0056949A     A6                  cmps byte ptr ds:[esi],byte pt>
0056949B     B9 04000000         mov ecx,4                      ; 中断在这里，FOEP
005694A0     6A 00               push 0
005694A2     6A 00               push 0
```

## 三、方法二

### 3.1、反 Anti-Debug + 找 Stolen Code + 找伪 OEP

`OD` 的隐藏插件设置随便(我设置的是全部勾选上)，忽略所有的异常！`OD` 载入目标程序

```
00645000 C>  60                  pushad                         ; EP
00645001     85F5               test ebp,esi                   ; ESP=0012FFA4
00645003     46                  inc esi
00645004     E8 01000000        call Calories.0064500A
Ctrl+G-->CreateToolhelp32Snapshot-->找到段尾-->F2 下断
```

```
7C864835     5E                  pop esi
7C864836     C9                  leave
7C864837     C2 0800             retn 8                         ; CreateToolhelp32Snapshot 的段尾，F2
F9 运行，中断下来，注意看 EAX=6C(在你那里可能不是这个值)-->右键归零(把 EAX 清 0)
再次 F9，此时注意看 EAX=68(在你那里可能不是这个值)-->再次右键归零(把 EAX 清 0)
此时 EAX 的值就是 Anti-Debug 的校验值，所以，清除为 0，就可以防止 Anti-Debug
取消 7C864837 处的断点，F9 运行，由于前面下 hr ESP 的缘故，中断在 Stolen Code
```

```
0065FAF3     55                  push ebp                     ; Stolen Code 第一句
0065FAF4     8BEC                mov ebp,esp                  ; Stolen Code 第二句
0065FAF6     90                  nop
0065FAF7     90                  nop
0065FAF8     90                  nop
55 8B EC
硬件断点不要取消，继续 F9，第 2 次硬件中断
```

```
0065FAFA     60                  pushad                       ; 第 2 次硬件中断
0065FAFB     E8 00000000         call Calories.0065FB00
0065FB00     5E                  pop esi
硬件断点不要取消，继续 F9，第 3 次硬件中断
```

```
0065FE8F     60                  pushad                       ; 第 3 次硬件中断
0065FE90     E8 00000000         call Calories.0065FE95
0065FE95     5E                  pop esi
0065FE96     83EE 06             sub esi,6
硬件断点不要取消，继续 F9，第 4 次硬件中断
```

```
0065FECB    /EB 01              jmp short Calories.0065FECE    ; 第 4 次硬件中断
0065FECD   -|E9 FF25D4FE        jmp FF3A24D1
0065FED2     65:009B 94560000   add byte ptr gs:[ebx+5694],bl
0065FED9     0000               add byte ptr ds:[eax],al
0065FEDB     0000               add byte ptr ds:[eax],al
0065FEDD     0000               add byte ptr ds:[eax],al
此时可以删除硬件断点，F8
```

```
0065FECE   - FF25 D4FE6500      jmp dword ptr ds:[65FED4]      ; F8 到了这里，继续 F8，即可来到 FOEP
0065FED4     9B                 wait
0065FED5     94                 xchg eax,esp
0065FED6     56                 push esi
0065FED7     0000               add byte ptr ds:[eax],al
0065FED9     0000               add byte ptr ds:[eax],al
0065FEDB     0000               add byte ptr ds:[eax],al
0065FEDD     0000               add byte ptr ds:[eax],al
0065FEDF     0043 46            add byte ptr ds:[ebx+46],al
```

```
00569498     42                  inc edx                        ; OEP
00569499     44                  inc esp
0056949A     A6                  cmps byte ptr ds:[esi],byte pt>
0056949B     B9 04000000         mov ecx,4                      ; 来到这里，FOEP
005694A0     6A 00               push 0
005694A2     6A 00               push 0
005694A4     49                  dec ecx
```

## 四、还原 Stolen Code + 试探性 Dump + 修复 IAT

修补好 Stolen Code 之后

```
00569498     55                  push ebp                     ; 修补好 OEP，右键新建 EIP
00569499     8BEC                mov ebp,esp
0056949B     B9 04000000         mov ecx,4
005694A0     6A 00               push 0
```

试探性 `Dump`：使用 `LoadPE` 纠正镜像，完整脱壳，打开 `ImportREC`，`OEP=00169498`，获取输入表，有很多无效的，使用 `ACProtect` 插件修复

最后有两个指针无效，根据经验判断全部还原成 `MessageBoxA`，保存下树文件，`FixDump!`

试运行程序发现无法运行。

用另外一个 `OD` 载入脱壳修复的文件(我们到达 `OEP` 的 `OD` 还有用，不能关哦)，细细查看，发现这里

```
005694D6     BA 9C985600         mov edx,Calories.0056989C
005694DB     E8 E899EEFF         call Calories.00452EC8        ；Enter
005694E0     8B0D 68F65600       mov ecx,dword ptr ds:[56F668] ; Calories.005707D8
```

```
00452EED     8BC3                mov eax,ebx
00452EEF     E8 8CFFFFFF         call Calories.00452E80
00452EF4     8B45 FC             mov eax,dword ptr ss:[ebp-4]
00452EF7     8BD6                mov edx,esi
```



```
 00452EF9     E8 EE13FBFF       call Calories.004042EC          ; Enter

 00452EFE     75 06             jnz short Calories.00452F06

引用:    

 00404303     8B46 FC           mov eax,dword ptr ds:[esi-4]

 00404306     E8 AC222400       call Calories.006465B7          ; Enter

 0040430B     77 02             ja short Calories.0040430F

 0040430D     01C2             add edx,eax

引用:    

 ......

 ......

 006465B7   - FF25 BC721400     jmp dword ptr ds:[1472BC]     ；来到这里

 006465BD   - FF25 C0721400     jmp dword ptr ds:[1472C0]

 006465C3   - FF25 C4721400     jmp dword ptr ds:[1472C4]

 006465C9   - FF25 C8721400     jmp dword ptr ds:[1472C8]

 006465CF   - FF25 CC721400     jmp dword ptr ds:[1472CC]

 ......

 ......

 00646701   - FF25 98731400     jmp dword ptr ds:[147398]

 00646707   - FF25 9C731400     jmp dword ptr ds:[14739C]

 0064670D   - FF25 A0731400     jmp dword ptr ds:[1473A0]

 00646713   - FF25 A4731400     jmp dword ptr ds:[1473A4]

 00646719   - FF25 A8731400     jmp dword ptr ds:[1473A8]

 0064671F   - FF25 AC731400     jmp dword ptr ds:[1473AC]

 ......

 ......
```

呵呵，就是`Code Splicing`在作怪了。

## 五、处理`Code Splicing`+`Dump`+`Fix Dump`

这里使用`Splicing Remover`，在网上找到了一个`Splicing Remover`不完整的`Delphi`源码，幸好我学的就是`Delphi`，于是自己补充完整了！

```
引用:    

 00646701   - FF25 98731400     jmp dword ptr ds:[147398]     ; 右键--转存中跟随--内存地址

 00646707   - FF25 9C731400     jmp dword ptr ds:[14739C]

 0064670D   - FF25 A0731400     jmp dword ptr ds:[1473A0]

 00646713   - FF25 A4731400     jmp dword ptr ds:[1473A4]

 00646719   - FF25 A8731400     jmp dword ptr ds:[1473A8]

 0064671F   - FF25 AC731400     jmp dword ptr ds:[1473AC]

 ......

 ......
```

发现`Code Splicing`所在区段为：

内存映射，项目`5`

- `地址`=`00140000`
- `大小`=`0000A000` (`40960.`)
- `物主`=`00140000` （自身）
- `区段`=``
- `类型`=`Priv 00021004`
- `访问`=`RW`
- `初始访问`=`RW`

打开`Splicing Remover`，按图示填写。`Splicing Remover`修复完毕之后，我们稍微找一个地方对比下(因为地方很多！)

![](img/09a5ccc1566f79db140669c5a9dc1279.jpg)

![](img/d9ed92ecffd91b6ea73ac7354f1c38fa.jpg)

![](img/1835e2f1d9545efe3432091a6ce58d1e.jpg)

![](img/8a34ae32640b75a8aeea9c3106cd230d.jpg)

修复前：

```
引用:    

 00404303     8B46 FC           mov eax,dword ptr ds:[esi-4]

 00404306     E8 AC222400       call Calories.006465B7          ; Enter

 0040430B     77 02             ja short Calories.0040430F

 0040430D     01C2             add edx,eax
```

修复后：

```
引用:    

 00404303     8B46 FC           mov eax,dword ptr ds:[esi-4]

 00404306     8B57 FC           mov edx,dword ptr ds:[edi-4]   ; Enter

 00404309     29D0             sub eax,edx

 0040430B     77 02             ja short Calories.0040430F

 0040430D     01C2             add edx,eax
```

像这样的地方还有很多很多，大家有兴趣可以仔细看看找找，好现在我们需要重新`Dump`了！

使用`LoadPE`纠正镜像，完整脱壳，打开`ImportREC`，直接载入我们保存的树文件，`Fix Dump`！

运行正常！~最后优化减肥下吧！

## 六、`Splicing Remover`主要代码(`Delphi`的)

```
引用:    

 procedure TForm1.Button1Click(Sender: TObject);

 var PID,PHD,Start,Size,SPStart,SPSize,Cnt,I,Go,GJMP:DWORD;
      Buf:byte; JMP:WORD;
      MBuf:array[0..4] of byte;
 
 begin
   Memo1.Clear;
 
   if Edit3.Text='' then Exit;
 
   PID:=HextoInt(Edit3.Text);
 
   Start:=HextoInt(Edit1.Text);
 
   Size:=HextoInt(Edit2.Text);
 
   SPStart:=HextoInt(Edit4.Text);
 
   SPSize:=HextoInt(Edit5.Text);
 
   PHD:=OpenProcess(PROCESS_ALL_ACCESS,False,PID);
 
   for I:=Start to Start+Size do begin
 
     ReadProcessMemory(PHD,Pointer(I),@Buf,Sizeof(Buf),Cnt);
 
     if (Buf=$E8) and (I+8<Start+Size) then begin
 
       ReadProcessMemory(PHD,Pointer(I+1),@Go,Sizeof(Go),Cnt);
 
       ReadProcessMemory(PHD,Pointer(I+Go+5),@JMP,Sizeof(JMP),Cnt);
 
       if JMP=$25FF then begin
 
         ReadProcessMemory(PHD,Pointer(I+Go+7),@GJMP,Sizeof(GJMP),Cnt);
 
         if (GJMP>=SPStart) and (GJMP<SPStart+SPSize) then begin
 
           ReadProcessMemory(PHD,Pointer(GJMP),@GJMP,Sizeof(GJMP),Cnt);
 
           if (GJMP>=SPStart) and (GJMP<SPStart+SPSize) then begin
 
             Memo1.Lines.Add(inttohex(I,8)+' <-- '+inttohex(GJMP,8));
 
             ReadProcessMemory(PHD,Pointer(GJMP),@MBuf,Sizeof(MBuf),Cnt);
 
             WriteProcessMemory(PHD,Pointer(I),@MBuf,Sizeof(MBuf),Cnt);
 
           end; 
 
         end;
 
       end;
 
     end;
 
   end;
 
   ShowMessage('修复完毕^\_^');
 
 end;
```

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 08 月 10 日 7:52:30

`4dd466d0e1e0cc123afbe504da2462ca`
