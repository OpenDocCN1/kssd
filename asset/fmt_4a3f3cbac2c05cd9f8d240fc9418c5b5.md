
### 关系运算 >= 时

```
004080F4  |.  833D 98924000>cmp      dword ptr [409298], 64
004080FB  |.  7C 19         jl       short 00408116
004080FD  |.  A1 08934000   mov      eax, dword ptr [409308]
00408102  |.  BA 2C814000   mov      edx, 0040812C                     ;  ASCII "Hello World"
00408107  |.  E8 48BDFFFF   call     00403E54
0040810C  |.  E8 07ABFFFF   call     00402C18
00408111  |.  E8 16A5FFFF   call     0040262C
```

[源]
```pascal
if Size >= 100 then
      begin
          Writeln('Hello World');
      end;
```

### 对单个字符取地址，并递增

```
00407C68  |.  C705 9C974000>mov      dword ptr [40979C], 00408294
00407C72  |.  FF05 9C974000 inc      dword ptr [40979C]
```

[源]
```pascal
szcChar := @ccc;
szcChar := szcChar + 1;
```

### 取回原指针指向的内容

```
00407C68  |.  C705 9C974000>mov      dword ptr [40979C], 00408294
00407C72  |.  A1 9C974000   mov      eax, dword ptr [40979C]
00407C77  |.  A3 9C974000   mov      dword ptr [40979C], eax

00407C7C  |.  A1 9C974000   mov      eax, dword ptr [40979C]
00407C81  |.  8A00          mov      al, byte ptr [eax]   //由于是字符指针，所以是 byte(8)
00407C83  |.  A2 98824000   mov      byte ptr [408298], al
```

[源]
```pascal
szcChar := @ccc;
szcChar := szcChar;     
ccb := szcChar^;  //取内容
```

### 不同赋值类型的反汇编。

**整数**

```
00407C68  |.  C705 94824000>mov      dword ptr [408294], 11D7
```

[源]
```pascal
rel := 4567;
```

**字符，布尔，浮点，**

```
00407C68  |.  C605 94824000>mov      byte ptr [408294], 1
00407C6F  |.  C605 98824000>mov      byte ptr [408298], 34
00407C76  |.  C705 9C974000>mov      dword ptr [40979C], CCCCCCDD  //0.
00407C80  |.  C705 A0974000>mov      dword ptr [4097A0], 3FDCCCCC  //45
```

[源]
```pascal
tBool:= True;
ccc  := '4';
tReal:= 0.45;
```

### 为一个字符串赋值

```
00407C68  |.  B8 9C974000   mov      eax, 0040979C
00407C6D  |.  BA 8C7C4000   mov      edx, 00407C8C                     ;  ASCII "Hello World"
00407C72  |.  E8 D1B9FFFF   call     00403648//调用此函数是在为字符传赋值. 
```

为字符传赋值要使用两个地址。

### 批量对 string 进行赋值并没有变化指令

```
00407C8C  |.  B8 94824000   mov      eax, 00408294                     ;要填充的字符串变量地址
00407C91  |.  BA CC7C4000   mov      edx, 00407CCC                     ;  ASCII "Hello World"
00407C96  |.  E8 ADB9FFFF   call     00403648
00407C9B  |.  B8 98824000   mov      eax, 00408298
00407CA0  |.  BA E07C4000   mov      edx, 00407CE0                     ;  ASCII "Beijing"
00407CA5  |.  E8 9EB9FFFF   call     00403648
00407CAA  |.  B8 9C824000   mov      eax, 0040829C
00407CAF  |.  BA F07C4000   mov      edx, 00407CF0                     ;  ASCII "GuNiang"
00407CB4  |.  E8 8FB9FFFF   call     00403648
```

[源]
```pascal
szStrN := 'Hello World';
      szStrA := 'Beijing';
      szStrZ := 'GuNiang';
```

### 拼接两个字符串

```
00407D00  |.  B8 94824000   mov      eax, 00408294
00407D05  |.  BA 487D4000   mov      edx, 00407D48                     ;  ASCII "Hello World"
00407D0A  |.  E8 39B9FFFF   call     00403648
00407D0F  |.  B8 98824000   mov      eax, 00408298
00407D14  |.  BA 5C7D4000   mov      edx, 00407D5C                     ;  ASCII "Beijing"
00407D19  |.  E8 2AB9FFFF   call     00403648
00407D1E  |.  B8 9C824000   mov      eax, 0040829C
00407D23  |.  8B0D 98824000 mov      ecx, dword ptr [408298]          ;  heloMain.00407CE8
00407D29  |.  8B15 94824000 mov      edx, dword ptr [408294]          ;  heloMain.00407CDC
00407D2F  |.  E8 80BBFFFF   call     004038B4
```

[源]
```pascal
szStrN := 'Hello World';
      szStrA := 'Beijing';
      szStrZ :=  szStrN + szStrA;
```

### (2)二 第二部分 流程控制，循环，对象，类型，结构等。

#### 条件分支语句， if .... then

```
00408118  |.  833D 98924000>cmp      dword ptr [409298], 0
0040811F  |.  74 19         je       short 0040813A
00408121  |.  A1 14934000   mov      eax, dword ptr [409314]
00408126  |.  BA 50814000   mov      edx, 00408150                     ;  ASCII "Hello World"
0040812B  |.  E8 24BDFFFF   call     00403E54
00408130  |.  E8 E3AAFFFF   call     00402C18
00408135  |.  E8 F2A4FFFF   call     0040262C
```

[源]
```pascal
if rel <> 0 then
      begin
          Writeln('Hello World');
      end;
```

#### 条件分支语句， if ... then ... else

```
00408118  |.  833D 98924000>cmp      dword ptr [409298], 0
0040811F  |.  74 1B         je       short 0040813C
00408121  |.  A1 14934000   mov      eax, dword ptr [409314]
00408126  |.  BA 68814000   mov      edx, 00408168                     ;  ASCII "Hello World"
0040812B  |.  E8 24BDFFFF   call     00403E54
00408130  |.  E8 E3AAFFFF   call     00402C18
00408135  |.  E8 F2A4FFFF   call     0040262C
0040813A  |.  EB 19         jmp      short 00408155
0040813C  |>  A1 14934000   mov      eax, dword ptr [409314]
00408141  |.  BA 68814000   mov      edx, 00408168                     ;  ASCII "Hello World"
00408146  |.  E8 09BDFFFF   call     00403E54
0040814B  |.  E8 C8AAFFFF   call     00402C18
00408150  |.  E8 D7A4FFFF   call     0040262C
```

[源]
```pascal
if rel <> 0 then
      begin
          Writeln('Hello World');
      end
      else
      begin
          Writeln('Hello World');
      end;
```

#### 条件分支语句 if ... then ...  else if     then  .... else

```
00408118  |.  833D 98924000>cmp      dword ptr [409298], 0
0040811F  |.  74 1B         je       short 0040813C
00408121  |.  A1 14934000   mov      eax, dword ptr [409314]
00408126  |.  BA 8C814000   mov      edx, 0040818C                     ;  ASCII "Hello World"
0040812B  |.  E8 24BDFFFF   call     00403E54
00408130  |.  E8 E3AAFFFF   call     00402C18
00408135  |.  E8 F2A4FFFF   call     0040262C
0040813A  |.  EB 3D         jmp      short 00408179

0040813C  |>  833D 98924000>cmp      dword ptr [409298], 0
00408143  |.  75 1B         jnz      short 00408160
00408145  |.  A1 14934000   mov      eax, dword ptr [409314]
0040814A  |.  BA 8C814000   mov      edx, 0040818C                     ;  ASCII "Hello World"
0040814F  |.  E8 00BDFFFF   call     00403E54
00408154  |.  E8 BFAAFFFF   call     00402C18
00408159  |.  E8 CEA4FFFF   call     0040262C
0040815E  |.  EB 19         jmp      short 00408179

00408160  |>  A1 14934000   mov      eax, dword ptr [409314]
00408165  |.  BA 8C814000   mov      edx, 0040818C                     ;  ASCII "Hello World"
0040816A  |.  E8 E5BCFFFF   call     00403E54
0040816F  |.  E8 A4AAFFFF   call     00402C18
00408174  |.  E8 B3A4FFFF   call     0040262C
```

[源]
```pascal
if rel <> 0 then
      begin
          Writeln('Hello World');
      end
      else if rel = 0 then
      begin
          Writeln('Hello World');
      end
      else
      begin
          Writeln('Hello World');
      end;
```

#### 多分支语句 case ....  of  .... end;

```
00407C8C  |.  A1 94824000   mov      eax, dword ptr [408294]
00407C91  |.  83E8 01       sub      eax, 1                           ;  Switch (cases 0..1)
00407C94  |.  72 04         jb       short 00407C9A
00407C96  |.  74 17         je       short 00407CAF
00407C98  |.  EB 28         jmp      short 00407CC2
```
