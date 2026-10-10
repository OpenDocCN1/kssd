
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```

```
.text:3E586F69 public: void \_\_thiscall CTreeNode::SetElement(class CElement \*) proc near  
.text:3E586F69                                         ; CODE XREF: CTreeNode::CTreeNode(CTreeNode \*,CElement \*)+21p  
.text:3E586F69  
.text:3E586F69 pCElementObject = dword ptr  8  
.text:3E586F69  
.text:3E586F69                 mov     edi, edi  
.text:3E586F6B                 push    ebp  
.text:3E586F6C                 mov     ebp, esp  
.text:3E586F6E                 mov     eax, \[ebp+pCElementObject\]  
.text:3E586F71                 test    eax, eax  
.text:3E586F73                 mov     \[ecx\], eax  
.text:3E586F75                 jz      short loc\_3E586F7D  
.text:3E586F75  
.text:3E586F77                 mov     al, \[eax+14h\]  
.text:3E586F7A                 mov     \[ecx+8\], al  
.text:3E586F7A  
.text:3E586F7D  
.text:3E586F7D loc\_3E586F7D:                           ; CODE XREF: CTreeNode::SetElement(CElement \*)+Cj  
.text:3E586F7D                 pop     ebp  
.text:3E586F7E                 retn    4  
.text:3E586F7E  
.text:3E586F7E public: void \_\_thiscall CTreeNode::SetElement(class CElement \*) endp  
.text:3E586F7E
```

步过以上部分，在此之后，网页代码使用 `createEventObject` 创建了 `event` 的副本，很的是，这个新的 `CEventObj` 类，当然 copy 了原来对象的 `EVENTPARAM` 结构，但是根据\[1\]的解释，在 `EVENTPARAM` 结构内容被 copy 的时候，并没有增加 `CTreeNode` 的访问计数！

`CEvent::Create` 中调用 `EVENTPARAM::EVENTPARAM` 复制 `EVENTPARAM` 结构，但后者的代码中的确没有包括操作其中 `CTreeNode`（在后面的 `CEventObj::get_srcElement` 函数分析中可以看出，`CTreeNode` 类指针在 `EVENTPARAM` 结构的头部）增加访问计数的内容。

引用: `.text:3E6B4BC9`

```
.text:3E6B4BC9 loc\_3E6B4BC9:                           ; CODE XREF: CEventObj::Create(IHTMLEventObj \* \*,CDoc \*,CElement \*,CMarkup \*,int,ushort \*,EVENTPARAM \*,int)+84j  
.text:3E6B4BC9                 push    0D8h            ; dwBytes  
.text:3E6B4BCE                 call    \_MemAlloc(x)  
.text:3E6B4BCE  
.text:3E6B4BD3                 test    eax, eax  
.text:3E6B4BD5                 jz      short loc\_3E6B4BE3  
.text:3E6B4BD5  
.text:3E6B4BD7                 push    \[ebp+arg\_18\]  
.text:3E6B4BDA                 mov     ecx, eax  
.text:3E6B4BDC                 call    EVENTPARAM::EVENTPARAM(EVENTPARAM const \*)  
.text:3E6B4BDC  
.text:3E6B4BE1                 jmp     short loc\_3E6B4BE5  
.text:3E6B4BE1  
```

```
.text:3E582A41  
.text:3E582A41 public: \_\_thiscall EVENTPARAM::EVENTPARAM(struct EVENTPARAM const \*) proc near  
.text:3E582A41                                         ; CODE XREF: CElement::get\_nodeType(long \*)-164050p  
.text:3E582A41                                         ; .text:3E5BFDC3p  
.text:3E582A41                                         ; .text:3E5C2363p  
.text:3E582A41                                         ; CElement::FireStdEvent\_KeyHelper(CTreeNode \*,CMessage \*,int \*,EVENTINFO \*)+7Ap  
.text:3E582A41                                         ; CElement::get\_nodeType(long \*)-7B645p  
.text:3E582A41                                         ; CElement::get\_nodeType(long \*)+5FEBp ...  
.text:3E582A41  
.text:3E582A41 pEventParam     = dword ptr  8  
.text:3E582A41  
.text:3E582A41                 mov     edi, edi  
.text:3E582A43                 push    ebp  
.text:3E582A44                 mov     ebp, esp  
.text:3E582A46                 push    ebx  
.text:3E582A47                 mov     ebx, ecx  
.text:3E582A49                 push    esi  
.text:3E582A4A                 mov     esi, \[ebp+pEventParam\]  
.text:3E582A4D                 push    edi  
.text:3E582A4E                 xor     eax, eax  
.text:3E582A50                 lea     edi, \[ebx+48h\]  
.text:3E582A53                 stosd  
.text:3E582A54                 stosd  
.text:3E582A55                 stosd  
.text:3E582A56                 stosd  
.text:3E582A57                 xor     edx, edx  
.text:3E582A59                 mov     \[ebx+0C4h\], edx  
.text:3E582A5F                 mov     \[ebx+0C8h\], edx  
.text:3E582A65                 mov     \[ebx+0CCh\], edx  
.text:3E582A6B                 mov     \[ebx+0D0h\], edx  
.text:3E582A71                 push    36h  
.text:3E582A73                 pop     ecx  
.text:3E582A74                 mov     edi, ebx  
.text:3E582A76                 rep movsd               ; 直接 copy 了内容  
.text:3E582A78                 mov     eax, \[ebx+64h\]  
.text:3E582A7B                 and     dword ptr \[ebx+0A8h\], 0FFFFFBFFh  
.text:3E582A85                 add     dword ptr \[eax+8\], 8  
.text:3E582A89                 mov     eax, \[ebx+68h\]  
.text:3E582A8C                 cmp     eax, edx  
.text:3E582A8E                 jz      short loc\_3E582A94  
.text:3E582A8E  
.text:3E582A90                 add     dword ptr \[eax+8\], 8  
.text:3E582A90  
.text:3E582A94  
.text:3E582A94 loc\_3E582A94:                           ; CODE XREF: EVENTPARAM::EVENTPARAM(EVENTPARAM const \*)+4Dj  
.text:3E582A94                 mov     eax, \[ebx+6Ch\]  
.text:3E582A97                 cmp     eax, edx  
.text:3E582A99                 jnz     loc\_3E6A4170  
.text:3E582A99  
.text:3E582A9F  
.text:3E582A9F loc\_3E582A9F:                           ; CODE XREF: CElement::get\_nodeType(long \*)+13875j  
.text:3E582A9F                 mov     eax, \[ebx+84h\]  
.text:3E582AA5                 cmp     eax, edx  
.text:3E582AA7                 jnz     loc\_3E6A4179  
.text:3E582AA7  
.text:3E582AAD                 jmp     loc\_3E6A4181  
.text:3E582AAD  
.text:3E582AAD public: \_\_thiscall EVENTPARAM::EVENTPARAM(struct EVENTPARAM const \*) endp  
.text:3E582AAD  
.text:3E582AB2  
.text:3E582AB2 loc\_3E582AB2:                           ; CODE XREF: CElement::get\_nodeType(long \*)+138EEj  
.text:3E582AB2                 pop     edi  
.text:3E582AB3                 pop     esi  
.text:3E582AB4                 mov     eax, ebx  
.text:3E582AB6                 pop     ebx  
.text:3E582AB7                 pop     ebp  
.text:3E582AB8                 retn    4  
.text:3E6A4170  
.text:3E6A4170 loc\_3E6A4170:                           ; CODE XREF: EVENTPARAM::EVENTPARAM(EVENTPARAM const \*)+58j  
.text:3E6A4170                 add     dword ptr \[eax+8\], 8  
.text:3E6A4174                 jmp     loc\_3E582A9F  
.text:3E6A4174  
.text:3E6A4179 ; ---------------------------------------------------------------------------  
.text:3E6A4179  
.text:3E6A4179 loc\_3E6A4179:                           ; CODE XREF: EVENTPARAM::EVENTPARAM(EVENTPARAM const \*)+66j  
.text:3E6A4179                 mov     ecx, \[eax\]  
.text:3E6A417B                 push    eax  
.text:3E6A417C                 call    dword ptr \[ecx+4\]  
.text:3E6A417F                 xor     edx, edx  
.text:3E6A417F  
.text:3E6A4181  
.text:3E6A4181 loc\_3E6A4181:                           ; CODE XREF: EVENTPARAM::EVENTPARAM(EVENTPARAM const \*)+6Cj  
.text:3E6A4181                 mov     eax, \[ebp+lpMem\]  
.text:3E6A4184                 add     eax, 0C4h  
.text:3E6A4189                 lea     ecx, \[ebx+0C4h\]  
.text:3E6A418F                 push    eax  
.text:3E6A4190                 mov     \[ecx\], edx  
.text:3E6A4192                 call    CStr::Set(CStr const &)  
.text:3E6A4192  
.text:3E6A4197                 mov     esi, \[ebp+lpMem\]  
.text:3E6A419A                 lea     eax, \[esi+0C8h\]  
.text:3E6A41A0                 lea     ecx, \[ebx+0C8h\]  
.text:3E6A41A6                 xor     edi, edi  
.text:3E6A41A8                 push    eax  
.text:3E6A41A9                 mov     \[ecx\], edi  
.text:3E6A41AB                 call    CStr::Set(CStr const &)  
.text:3E6A41AB  
.text:3E6A41B0                 lea     eax, \[esi+0CCh\]  
.text:3E6A41B6                 lea     ecx, \[ebx+0CCh\]  
.text:3E6A41BC                 push    eax  
.text:3E6A41BD                 mov     \[ecx\], edi  
.text:3E6A41BF                 call    CStr::Set(CStr const &)  
.text:3E6A41BF
```
