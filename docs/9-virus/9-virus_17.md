
# Billy Belceb 病毒编写教程---DOS 篇

> 原文：[`www.pediy.com/kssd/pediy06/pediy640612.htm`](https://www.pediy.com/kssd/pediy06/pediy640612.htm)

## 优化(Optimization)

有两种类型的优化：结构优化和局部优化。在这一小章里我们将讨论这两种类型。但是首先你必须懂得一件事：在它不影响功能的条件下优化你的代码。如果你作了不起作用的优化，有很多使得它不能工作，你需要修正它，那么你就会犯越来越多的错误...一个永不休止的恶性循环:)

### 结构优化

这是最有效的，也是很难做到和理解的。这种类型的优化通过使用一张纸把你的病毒的代码写在上面后就能够被很容易的理解了。我们这里没有纸，所以让我们想象一种情形...假设你，在你的病毒里，以只读打开文件，关闭，再以读/写方式打开文件，再关闭，这是字节的浪费。对于这种类型的优化，你必须对你能改变和保存哪些字节考虑。解决方法肯定因你的问题不同而不同。

### 局部优化

尽管它能节约很多字节，这个是最简单的方法。它在于单独地改变一些代码行，做相同的事情而能使用更少的字节。

给寄存器清 0：

```
mov  bx,0000h    ; 3 bytes
xor  bx,bx       ; 2 bytes
sub  bx,bx       ; 2 bytes
```

所以，永远不要使用第一个，要选择其它的方式。有一个寄存器可以用其它的方法清零：`DX`。让我们来看：

```
mov  dx,0000h    ; 3 bytes
xor  dx,dx       ; 2 bytes
sub  dx,dx       ; 2 bytes
cwd             ; Convert word to dword ( 1 byte )
```

`CWD` 将在 `AX` 比 `8000h` 小的时候工作。有一种只用一个字节就可以给 `AH` 清零的方法：如果 `AL`<`80h`，你可以使用 `CBW` 指令。

比较：

我们都知道一个很著名的方法，那就是对下面的指令使用特殊的指令：`CMP`。在比较两个寄存器时，你可以使用下面能得到相同结果的方法，没有任何节约：

```
cmp  ax,bx       ; 2 bytes
xor  ax,bx       ; 2 bytes
```

但是如果我们仅仅想知道两个值是否相等，我们可以在任何情况下使用 `XOR`，如果我们在比较一个寄存器和一个值的时候使用 `XOR` 而不使用 `CMP` 就可以节约字节：

```
cmp  ax,0666h    ; 3 bytes
xor  ax,0666h    ; 2 bytes
```

但是，由于 `XOR` 指令的自然属性，我们可以用它来判断一个寄存器是否为 0。但是下面可以用 `OR` 来节约字节...

```
cmp  ax,0000h    ; 3 bytes
or   ax,ax       ; 2 bytes
```

寄存器优化 - `AX`:

你可以利用它来作比较：

```
cmp  bx,0666h    ; 4 bytes
cmp  ax,0666h    ; 3 bytes
```

而且你还可以用一种非常优化的方法把 `AX` 的值赋给另外一个寄存器：

```
mov  bx,ax       ; 2 bytes
xchg ax,bx       ; 1 byte
```

你做这个的时候，`AX` 和 `BX` 的值是否改变都不重要。在一个文件打开后确实很好，因为文件句柄在 `BX` 中更好。

字符串操作：

每一个字符串操作(`MOVS`,`STOS`,`SCAS`...)都是优化形式。让我们看看你利用它来做什么：

\- `MOVS`:从 `DS:[SI]`到 `ES:[DI]`的移动

```
les  di,ds:[si]    ; 3 bytes
movsb             ; If we want a byte ( 1 byte )
movsw             ; If we want a word ( 1 byte )
movsd             ; If we want a dword ( 2 bytes ) 386+
```

\- `LODS`: 把位置 `DS:[SI]`的值放进累加器

```
mov  ax,ds:[si]    ; 2 bytes
lodsb             ; If we want a byte ( 1 byte )
lodsw             ; If we want a word ( 1 byte )
lodsd             ; If we want a dword ( 2 bytes ) 386+
```

\- `STOS`: 把位置 `ES:[SI]`的值放进累加器

```
les  di,al       ; Can't do this!
les  di,ax       ; Can't do this!
stosb             ; If we want a byte ( 1 byte )
stosw             ; If we want a word ( 1 byte )
stosd             ; If we want a dword ( 2 bytes ) 386+
```

\- `CMPS`:比较 `DS:[SI]`和 `ES:[DI]`里的值

```
cmp  ds:[si],es:[di]   ; Can't have 2 segment overrides!
cmpsb                 ; If we want a byte ( 1 byte )
cmpsw                 ; If we want a word ( 1 byte )
cmpsd                 ; If we want a dword ( 2 bytes ) 386+
```

\- `SCAS`:比较累加器的值和 `ES:[DI]`

```
cmp  ax,es:[di]    ; 3 bytes
scasb             ; If we want a byte ( 1 byte )
scasw             ; If we want a word ( 1 byte )
scasd             ; If we want a dword ( 2 bytes ) 386+
```

16 位寄存器：

通常，使用 16 位寄存器比 8 位寄存器更优化。让我们一个 `MOV` 指令的例子：

```
mov  ah,06h       ; 2 bytes
mov  al,66h       ; 2 bytes ( 4 bytes total )
mov  ax,0666h    ; 3 bytes
```

加/减任何 16 位寄存器也更优化：

```
inc  al          ; 2 bytes
inc  ax          ; 1 byte
dec  al          ; 2 bytes
dec  ax          ; 1 byte
```

基和段：

从一个段到另一个段的移动不能直接进行，所以我们必须对它处理：

```
mov  es,ds       ; Can't do this!
mov  ax,ds       ; 2 bytes
mov  es,ax       ; 2 bytes ( 4 bytes total )
push ds          ; 1 byte
pop  es          ; 1 byte ( 2 bytes total )
```

使用 `DI`/`SI` 比使用 `BP` 更先进。

```
mov  ax,ds:[bp]    ; 4 bytes
mov  ax,ds:[si]    ; 3 bytes
```

过程：

如果你使用一个例程很多次，你必须考虑编写一个过程的可能性。这个能优化你的代码。无论如何，对过程的使用不当会使我们的需要相反：代码将会增长。所以，如果你想知道一个例程到过程的转变能否节约字节，你可以使用下面的公式：

`X = [rout. size - (CALL size + RET size)] * number of calls - rout. size`

`CALL` size+ `RET` size 表示 4 个字节。`X` 使我们将会节约的字节。让我们看看节约一些字节的经典函数，文件指针的移动：

```
fpend: mov  ax,4202h    ; 3 bytes
fpmov: xor  cx,cx      ; 2 bytes
  cwd             ; 1 byte
  int  21h        ; 2 bytes
  ret             ; 1 byte
```

我们得到 8 字节+`CALL` 大小...11 字节。让我们看看这个是否能优化我们的代码：

`X = [ 7 - ( 3 + 1 ) ] * 3 - 7`
`X = 2 bytes saved`

毫无疑问，这是一个创造性的计算。你调用这个例程可以多于 3 次(或更少)，使它的大小不同，和更多的东西。

局部优化的最后几点：

\- 使用 `SFT`。在这个结构里你已经有很多有用的信息了，并且你可以没有任何问题地操作它。

\- 使你的编译器至少扫描你的代码三遍来排除无用的 `NOP` 和其它的东西。

\- 使用堆栈。

\- 使用 `MOV` offset 的 `LEA` 指令更优化。
