
# DELPHI 内联汇编的一点心得

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-723.htm`](https://www.pediy.com/kssd/pediy08/pediy8-723.htm)

DELPHI 内联汇编好象有得天独厚的优势，尤其是 API 的调用，处理好各成员参数后，可以直接 CALL API 名称。利用内联汇编我们可以为程序添加各种异常，添加反调试代码，添加花指令，还可以令某些难写的注册机简单化等。我把 DELPHI 内联汇编的一点点使用心得写出来，目的是想与大家交流，挖掘更多的这方面的领域。也希望大家能够指正一些错误。

## 一、数据格式

- 整型数据：8 位的用 `AL` 返回，16 位的用 `AX` 返回，32 位的用 `EAX` 返回。
- `BYTE`（8 位）：`BYTE`、`CHAR`、`SHORTINT`、`BOOLEAN`
- `WORD`（16 位）：`SMALLINT`、`WORD`
- `DWORD`（32 位）：`INTEGER`、`LONGWORD`、`ANSISTRING`、`POINTER`、`CLASS`、`LONGINT`、`STRING`
- `ST`（0）：`SINGLE`、`DOUBLE`、`EXTENDED`、`COM`

如：

```pascal
var
  ByteVar: Byte;
  WordVar: Word;
  IntVar: Integer;

begin
  asm
    MOV     AL, ByteVar
    MOV     BX, WordVar
    MOV     ECX, IntVar
  end;
end;
```

- 实型：用 `ST`（0）返回
- 指针：用 `EAX` 返回
- 长字符串：用 `EAX` 返回其所在地址
- 变量：可用 `@Result` 返回
- 16 进制的表示方式：
  - 如果是 0-9 开头的 16 进制值直接在后面加 `H` 或在前面加 `$` 号，如：`1AH`，`$1A`
  - 如果是字母开头的前面加 `0` 再在后面跟 `H`，或者直接用 `$` 号，如：`0AH`，`$A`
- 几个修饰符：
  - `OFFSET`：返回内存地址中的立即数
  - `[....]`：返回内存地址，与 `OFFSET` 相反，如：`MOV EAX, OFFSET [XXXX]` = `MOV EAX, [OFFSET XXXX]` = `MOV EAX, XXXX`
  - `HIGH`：返回高 8 位的立即数
  - `LOW`：返回低 8 位的立即数
  - `&`：防止变量与汇编中的寄存器同名而在前面加 `&` 号，如：
    ```
    EAX: INTEGER;  ...
    MOV &EAX, 10H  // 这里的 &EAX 不是 EAX 寄存器
    ```
  - `.`：`.号`的一种用法：
    ```pascal
    var
      STR: Word;
    ...
    MOV DL, STR.Byte 或 DL, Byte(STR)
    ```

## 二、嵌入式汇编的格式

Delphi 是使用 `ASM`...`END` 来标志汇编语句

如：

```pascal
ASM
  mov al, 1
  mov bl, al
  END;
```

一个简单加法函数：

```pascal
FUNCTION SUM(X, Y: INTEGER): INTEGER;
BEGIN
  ASM
    MOV EAX, X
    ADD EAX, Y
    MOV @RESULT, EAX
  END;
END;
```

`Byte` 转换为 16 进制字符串：

```pascal
function ByteToHex(Src: Byte): String;
begin
  SetLength(Result, 2);
  asm
    MOV         EDI, [Result]
    MOV         EDI, [EDI]
    MOV         AL, Src
    MOV         AH, AL          // Save to AH
    SHR         AL, 4           // Output High 4 Bits
    ADD         AL, '0'
    CMP         AL, '9'
    JBE         @@OutCharLo
    ADD         AL, 'A'-'9'-1

@@OutCharLo:
    AND         AH, $f
    ADD         AH, '0'
    CMP         AH, '9'
    JBE         @@OutChar
    ADD         AH, 'A'-'9'-1

@@OutChar:
    STOSW
  end;
end;
```

## 三、可用的寄存器

- 32 位寄存器：`EAX` `EBX` `ECX` `EDX` `ESP` `EBP` `ESI` `EDI`
- 16 位寄存器：`AX` `BX` `CX` `DX` `SP` `BP` `SI` `DI`
- 8 位寄存器：`AL` `BL` `CL` `DL`（低 8 位），`AH` `BH` `CH` `DH`（高 8 位）
- 16 位段寄存器：`CS` `DS` `SS` `ES`
- 32 位段寄存器：`FS` `GS`
- 以及协处理器寄存器堆栈 `ST`

一个 `ASM` statement 必须保护 `EDI`，`ESI`，`ESP`，`EBP` 和 `EBX` 寄存器，但是可以自由的修改 `EAX`，`ECX` 和 `EDX` 寄存器。

默认情况下，delphi 使用“register”方式，若参数在 3 个以内，将分别使用 `eax`、`edx` 和 `ecx`，如果超过三个，则用堆栈传递。返回参数的存放视长度而定，例如 8 位用 `al` 返回，16 位用 `ax`，32 位用 `eax`，64 位用两个 32 位寄存器 `edx:eax`，其中 `eax` 是低位。如果你想用 `EBX` 寄存器，接得这样写：

```pascal
asm
  push ebx
  mov ebx, 2
  mov IntVar, ebx
  pop ebx
end;
```

如：`FUNCTION(T1, T2, T3: INTEGER): INTEGER`，可以默认为 `T1` 存在 `EAX` 中，`T2` 存在 `EDX` 中，`T3` 存在 `ECX` 中。

DELPHI 的标签名一般都以 `@` 开头，比如 `@exit`、`@001` 等

## 四、CALL 的应用

在汇编中写代码要保存寄存器现场（保存使用前的寄存器状态，使用 `Push` 压栈和 `Pop` 从栈中弹出），不过这一切对于 Delphi 的嵌入式汇编是没有必要的（除非你自己要使用 `Push` 和 `Pop`），因为 Delphi 已经帮你做了，不必担心会使数据丢掉。

DELPHI 内联汇编中同样可以用汇编中 `CALL` 的功能。

比如：

```pascal
asm
  CALL @1
  JMP @EXIT

@1:
  MOV EAX, 1
  MOV SN, EAX
  RETN

@exit:
  end;
```

`call` 的第二种用法，在汇编代码中直接调用 `FUNCTION` 函数，如：

```pascal
function cacl(eax: integer): integer;
var
  ...
begin
  ....
  Result := code;
end;
```

调用时可以直接 `CALL CACL`:

```pascal
ASM
  CALL CACL
  ...
  END
```

编语句中的 `Call` 语句，可以用于调用其它过程，既可以是其它汇编程序段也可以是 Delphi 中的标准过程：

例如：假设新建一个窗体并在上面加了一个按钮，在 `Click` 事件中写入以下代码

```pascal
procedure TForm1.Button1Click(Sender: TObject);
begin
  showmessage('ok');
end;
```

再写一个过程 `_x`

```pascal
function TForm1._x(var i: smallint): integer;
asm
  call button1click
end;
```

执行 `_x` 的结果就可以显示消息框。

## 五、调用 API 函数

DELPHI 内联汇编中调用 API 函数非常简单，比如调用 `PostQuitMessage`:

```pascal
Asm
  Push 0
  Call PostQuitMessage // 或 Call SYSTEM.PostQuitMessage
End
```

调用 `MessageBox` 函数：

```pascal
procedure TForm1.Button2Click(Sender: TObject);
var
  szTitle: string;
  szCaption: string;
begin
  szTitle := '您好！';
  szCaption := '这是一个在内嵌汇编中调用 stdcall 类型函数的例子.';
  asm
    PUSH MB_OK+MB_ICONINFORMATION
    PUSH szTitle
    PUSH szCaption
    PUSH 0
    CALL MessageBox
  end;
end;
```

调用 `GetFileSize` 函数

```pascal
function stdcalldemo: Integer;
var
  FH: THandle;
begin
  FH := FileOpen('c:\root.ini', fmOpenRead);
  asm
    push 0
    push FH
    call GetFileSize
    mov @Result, eax
  end;
  Result := GetFileSize(FH, 0); // 此句就相当于上面的汇编调用方式
  FileClose(FH);
end;

procedure TForm1.Button1Click(Sender: TObject);
begin
  showmessage(IntToStr(stdcalldemo));
end;
```

## 六、汇编的调试

如果发现汇编通不过，就要注意停下后的位置的代码：如变量的跟踪、断点、堆栈查看……可以在关键代码处下断，停下后打开 View 菜单的 Debug Windows 的 CPU VIEW 窗口，`F8` 单步跟踪。

```text
如:
Unit1.pas.62:          PUSH MB_OK+MB_ICONINFORMATION
00455E8F 6A40          push $40

Unit1.pas.63:          PUSH szTitle
00455E91 FF75FC        push dword ptr [ebp-$04]

Unit1.pas.64:          PUSH szCaption
00455E94 FF75F8        push dword ptr [ebp-$08]

Unit1.pas.65:          PUSH 0
00455E97 6A00          push $00

Unit1.pas.66:          CALL MessageBox
00455E99 E8FA0DFBFF    call MessageBox

Unit1.pas.69:          end;
00455E9E 33C0          xor eax,eax
```

## 七、常见易错语句

- `MOV ESI, DWORD PTR SS:[TEXT]` 对（`TEXT` 为 `STRING`）
- `MOV ESI, DWORD PTR [TEXT]` 对（段寄存器 `CS`, `DS`, `SS`, `ES` 可省略）
- `MOV AL, DWORD PTR SS:[TEXT]` 错（`DWORD` 为 32 位）
- `MOV AL, BYTE PTR [TEXT]` 对
- `MOV EAX, BYTE PTR [TEXT]` 错（`DWORD` 为 8 位）
- `MOV AL, WORD PTR [TEXT]` 错（`DWORD` 为 16 位）
- `MOV EAX, WORD PTR [TEXT]` 错
- `MOV AX, WORD PTR [TEXT]` 对
- `MOV [A], EAX`
- `MOV BYTE PTR[EDI], 'A'`
- `MOV AL, [EDI]`
- `MOV EAX, X` // `X` 指向的值赋值给 `EAX`
- `MOV EAX, [EAX]` // `X` 指向的地址赋值给 `EAX`
- `MOV DWORD PTR[ESP], EAX` 对
- `MOV DWORD PTR[ECX], EAX` 对
- `MOV DWORD PTR[ECX+4], EAX` 对
- `MOV DWORD PTR[ECX+EDI], EAX` 对
- `MOV DWORD PTR[ESI], EAX` 错
- `MOV DWORD PTR[EDX], EAX` 错
- `MOV DWORD PTR[EBX], EAX` 错
- ...

比较详细的例子可以看一下

`http://bbs.pediy.com/showthread.php?s=&threadid=26382`

`http://bbs.pediy.com/showthread.php?s=&postid=222986#post222986`

里的注册机代码，更多的内容还需要你补充…。

