
# 小弟遇到一个汇编方面的小问题 请高手帮忙解答 感激！

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-106.htm`](https://www.pediy.com/kssd/pediy09/pediy09-106.htm)

> 小弟用 `delphi` 编程 打算内嵌入一段汇编 主要的意思是把 `esi` 所指向的字符传传递到 `str` 里

> 小弟用 `mov str,esi` 不成功 求教高手 我应该如何做？

> 代码:

```
procedure TForm1.Button1Click(Sender: TObject);
var
  str:String;
  s:String;
begin
  // 从 Edit1.Text 中取一个字符串
  //我不知到你 esi 中的字符串是从那里飞来的，所以只能自己凭想象弄一个
  s:=Edit1.Text;
  asm
    mov  esi, s

    // LStrLAsg 的入口参数
    // ->    EAX     pointer to dest
    //       EDX     source
    lea  eax, str
    mov  edx, esi

    call system.@LStrLAsg
  end;
  // 将 str 里的字符串送到 Edit2 中，证明字符串被传递到 str 中了
  // 哈哈，是不是太简单了。
  Edit2.Text:=str;
end;
```

> 下面再给出一个可能是你想要的方法

> 代码:

```
procedure TForm1.Button2Click(Sender: TObject);
type
  StrRec = packed record
    allocSiz: Longint;
    refCnt: Longint;
    Length: Longint;
  end;
const
  skew = sizeof(StrRec);
var
  str:string;
  s:string;
  D:Dword;
begin
  s:=Edit1.Text;
  asm
    mov  esi, s

    // 使用 var 声名的变量 str 只是一个空的指针，在实际使用时必须初始化。
    // LStrSetLength 入口参数
    // ->    EAX     Pointer to str
    //       EDX new length
    lea  eax, str
    mov  edx, [esi-skew].StrRec.Length
    push edx
    call system.@LStrSetLength

    mov  edi,str
    pop  ecx
    cld
    rep  movsb
  end;
  Edit2.Text:=str;
end;
```

> 感谢 `gzgzlxg` 大哥的回答，不过我不是要把 `exit1.text` 的内容传递给 `str` 由于我是 `HOOK` 了一段程序的内存 类似打补丁那样 我是要把 `ESI` 寄存器执行到这里的字符串传递给 `str` 请问这样应该如何做？

> 就使用第二个方法，关键你需要自己先得到这个在 `esi` 中的字符串的长度，然后使用类似上面的方法，你先删除第一句：

> `mov  esi, s   //esi 已经有你字符串的地址了`

> 然后修改下面这句：

> `mov  edx, [esi-skew].StrRec.Length`

> `mov  edx, esi 中字符串的长度`

> 然后就 OK 了。

> 再学不会，我也无法教你了，这么简单的修改一下都做不到，前面给出的方法是给你一个思路，程序还是要自己写的，这个思路的关键，就是要知道 `Delphi` 内部有一套自己管理字符串的方法，你不能简单的用 `rep movsb` 这种指令将字符串传递到 `Delphi` 的字符串中去，必须按照 `Delphi` 的内部机制去处理，其中第一个方法的前提是，这两个字符串必须是 `Delphi` 格式的字符串，你那个 `esi` 中的字符串显然是不符合这个要求的，所以要使用第二个方法，第一个方法中，`Delphi` 的那条命令已经自动为目标字符串分配了内存，第二个方法中你就必须自己去为目标字符串分配内存，否则就会发生不可估计的错误。`LStrSetLength` 函数就是为那个字符串分配内存，这样你就可以将 `esi` 中的字符串传递到 `Delphi` 中了。

> 发过简单的例子,看对你有没有帮助

```
    push    0
    push    ecx
    lea     edx, dword ptr [esi+8]
    push    edx
    call    005C3350
    add     esp, $0C
```

> 假入上面的代码是一个加密过程,`edx` 指向需要加密的数据,现在要在加密过程执行之前把数据读出来

```
    push    0
    push    ecx
    lea     edx, dword ptr [esi+8]
    push    edx
    call    GetEnPacket  //005C3350          把 005C3350 替换为 GetEnPacket
    add     esp, 0C
```

```
var
  SaveDataAddr:DWORD;
  temp:DWORD;
  funcaddr:DWORD;

procedure ReadData;
var
  TempData: DWORD;
  Len: DWORD;
  str: string;
  Mem:PChar;
begin
  TempData := SaveDataAddr - 4;
  Len := PWORD(TempData)^;   //得到数据长度
  GetMem(Mem,Len);
  CopyMemory(Mem,pointer(TempData),Len);//数据复制到 mem
  //一些处理
  FreeMem(Mem);
end;

procedure GetEnPacket;
begin
  asm
    mov SaveDataAddr,edx    //edx 指向加密前的数据
  end;
  asm
    pushad
  end;
  ReadData; //读取数据
  asm
    popad
  end;
  asm
    pop temp  //为了堆栈平衡
    mov funcaddr, $005C32F0
    CAll funcaddr  //call 原来的加密过程 .如果果 005C32F0
    push Temp   //为了堆栈平衡
  end;
end;
```

> 不知道这个行不行,我也经常要在 `Delphi` 中加入汇编,`Delphi` 中有很多类似的函数,有些直接就用汇编写的,我都是自己改的,下面这个是改自 `Move` 函数,本来我这个函数里的变量都是局部的,所以对于 `Source,Str` 和 `Count` 都是用的 `Mov`,如果是全局的话有可能要改成 `Lea`,具体调试下就知道

> 代码:

```
//       LEA     ESI,Source    {假设此时 ESI 已经指向了你的源 Str,此步忽略}
       LEA     EDI,str       {假设你的 Str 是全局变量的话}
                       {如果是局部变量的话应该要 Mov Edi,Str,自己试下}

       MOV     ECX,count     {Count 为你要复制的长度,全局或局部好像也要处理下}
       MOV     EAX,ECX

       CMP     EDI,ESI
       JA      @@down
       JE      @@exit

       SAR     ECX,2         { copy count DIV 4 dwords       }
       JS      @@exit

       REP     MOVSD

       MOV     ECX,EAX
       AND     ECX,03H
       REP     MOVSB         { copy count MOD 4 bytes        }
       JMP     @@exit

@@down:
       LEA     ESI,[ESI+ECX-4] { point ESI to last dword of source     }
       LEA     EDI,[EDI+ECX-4] { point EDI to last dword of dest       }

       SAR     ECX,2         { copy count DIV 4 dwords       }
       JS      @@exit
       STD
       REP     MOVSD

       MOV     ECX,EAX
       AND     ECX,03H      { copy count MOD 4 bytes        }
       ADD     ESI,4-1      { point to last byte of rest     }
       ADD     EDI,4-1
       REP     MOVSB
       CLD
@@exit:
```

> `Sam.com` 的方法其实就是 `Delphi Vcl System.pas` 中 `Move` 函数的源代码，把这个源代码抄过来不如直接使用这个源代码，在 `Delphi` 中对 `Vcl` 中的内部函数的调用需要指明该函数所在文件的文件名，这样你就可以直接使用这个 `Move` 函数了，没有必要将已经现成的东西抄过来使用。

> 代码:

```
// ->EAX     Pointer to source
//   EDX     Pointer to destination
//   ECX     Count
 mov  eax,esi
 mov  edx,str
 mov  ecx,长度
 call system.MOVE
```

> 问题的关键还在于没有为字符串分配内存。这样运行结果难以预料。

> 引用: `Sam.com` 的方法其实就是 `Delphi Vcl System.pas` 中 `Move` 函数的源代码，把这个源代码抄过来不如直接使用这个源代码，在 `Delphi` 中对 `Vcl` 中的内部函数的调用需要指明该函数所在文件的文件名，这样你就可以直接使用这个 `Move` 函数了，没有必要将已经现成的东西抄过来使用

> 嗯~~我这个只是把他源代码简单的拿出来用~如果直接 `Call system.Move` 的话,不知道会不会把 `Move` 这一段编译成如下,我试过在驱动中调用这个函数的话,他是按这个来编译了,驱动执行就死机,但我直接用汇编就没问题了,不知道是什么原因,对于字符串分配内存的问题直接来个 `Str:=''不就行了吗?:`

> 代码:

```
procedure Move(const Source; var Dest; count : Integer);
{$IFDEF PUREPASCAL}
var
  S, D: PChar;
  I: Integer;
begin
  S := PChar(@Source);
  D := PChar(@Dest);
  if S = D then Exit;
  if Cardinal(D) > Cardinal(S) then
    for I := count-1 downto 0 do
      D[i] := S[i]
  else
    for I := 0 to count-1 do
      D[i] := S[i];
end;
{$ELSE}
asm
.......
```

> 不会，如果没有定义纯种的 `Pascal`，就不会发生这样的事。

> 你可以通过 `CPU` 调试窗口，看到这个 `MOVE` 的具体代码。

> 引用: 对于字符串分配内存的问题直接来个 `Str:=''不就行了吗?:`

> `Delphi` 的字符串管理机制非常复杂，当选择 `Huge String` 模式，即支持巨型字符串时，对字符串进行操作，只要字符串的长度发生变化，都要重新分配内存来存放新的字符串：

> 代码:

```
Str='';
```

> 这条指令进行时，`Delphi` 调用的是 `LStrClr` 函数，`LStrClr` 函数非常复杂，对于一个已经分配过内存的字符串，`LStrClr` 将释放原来分配的内存，然后重新建立一个空字符串，对于没有初始化过的字符串，`LStrClr` 将分配相应的内存，建立一个没有长度的字符串。

> 如果没有选择 `Huge String` 模式，所有字符串都是定长的，这时，`Str:='';`就直接分配了 256 字节长度的内存，而且在字符串操作过程中，不会再分配内存。

> 代码:

```
  StrRec = packed record
    allocSiz: Longint;
    refCnt: Longint;
    Length: Longint;
  end;
```

> 上面的结构是 `Huge String` 模式下 `Delphi` 的字符串结构，存放在一个字符串的前面，这样做的目的是兼容 `C` 格式的字符串，`Delphi` 中的任何字符串指针都是指向这个字符串的起始位置，而不是指向这个字符串的结构位置，这样对一个 `Delphi` 的字符串可以直接当作 `C` 字符串来操作，但用户进行类似操作的时候，字符串操作的长度不要大于 `allocSiz` 的长度，这个长度是 `Delphi` 在初始化字符串是为该字符串分配的实际空间大小，`Length` 是该字符串的实际长度，`allocSiz >= Length`。`refCnt` 是一个字符串使用计数器，如果这个计数器等于 0 时，将释放这个字符串所占据的内存，所以如果对该字符串多次引用，即使你认为自己释放了该字符串，只要 `refCnt` 大于零，`Delphi` 就不会释放该字符串所占据的内存。

> 多谢提醒,忘记了 `String` 是不好控制,我平时是这样来做的,如果按楼主的要求,就先定义一个 `array of byte,`先用 `Move` 把 `Esi` 指向的字符串复制到 `Buf` 去,然后再用下面这个函数提取 `String` 出来,这样应该能解决了吧~

> 代码:

```
var
  Buf:array [0..255] of byte;

str:=BytesToString(Buf,0,Count);

function BytesToString(Bytes: array of Byte; StartPos, Len: Integer): string;
var
  I: Integer;
  TmpStr: string;
begin
  TmpStr:='';
  for I:=StartPos to StartPos+Len-1 do
    if (Bytes[i]=0) then
    begin
      Result:=TmpStr;
      Exit;
    end
    else
      TmpStr:=TmpStr+Char(Bytes[i]);
  Result:=TmpStr;
end;
```

4dd466d0e1e0cc123afbe504da2462ca
