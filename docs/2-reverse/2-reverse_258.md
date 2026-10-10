

# IceExt 分析文章系列 1.如何判断一个程序是传统程序还是.Net assemblly.

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-821.htm`](https://www.pediy.com/kssd/pediy08/pediy8-821.htm)

我在分析 IceExt 时，想研究一下它的 Startup Code.分析了一半，忽然映入眼中一个函数，起初以为作用仅是分析 PE 格式是否合法，分析到一半，才恍然大悟，是验证它是否.Net Assembly。  
  
原理相当简单，验证数据目录是否有第 16 项(0x0f)，第 16 项是否有值。  
传统的可执行程序是只有 15 项数据目录(0-0x0E)。  
  
.text:01002E25 IsPEdotNetAssembly proc near            ; CODE XREF: start+9Fp  
.text:01002E25                 cmp     word ptr ds:1000000h, 5A4Dh  
.text:01002E2E                 jnz     short loc\_1002E57  
.text:01002E30                 mov     eax, ds:100003Ch  
.text:01002E35                 lea     eax, \[eax+1000000h\]  
.text:01002E3B                 cmp     dword ptr \[eax\], 4550h  
.text:01002E41                 jnz     short loc\_1002E57  
.text:01002E43                 movzx   ecx, word ptr \[eax+18h\]  
.text:01002E47                 cmp     ecx, 10Bh  
.text:01002E4D                 jz      short loc\_1002E6D  
.text:01002E4F                 cmp     ecx, 20Bh  
.text:01002E55                 jz      short loc\_1002E5A  
.text:01002E57  
.text:01002E57 loc\_1002E57:                            ; CODE XREF: IsPEdotNetAssembly+9j  
.text:01002E57                                         ; IsPEdotNetAssembly+1Cj ...  
.text:01002E57                 xor     eax, eax  
.text:01002E59                 retn  
.text:01002E5A ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:01002E5A  
.text:01002E5A loc\_1002E5A:                            ; CODE XREF: IsPEdotNetAssembly+30j  
.text:01002E5A                 cmp     dword ptr \[eax+84h\], 0Eh  
.text:01002E61                 jbe     short loc\_1002E57  
.text:01002E63                 xor     ecx, ecx  
.text:01002E65                 cmp     \[eax+0F8h\], ecx  
.text:01002E6B                 jmp     short loc\_1002E7B  
.text:01002E6D ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:01002E6D  
.text:01002E6D loc\_1002E6D:                            ; CODE XREF: IsPEdotNetAssembly+28j  
.text:01002E6D                 cmp     dword ptr \[eax+74h\], 0Eh  
.text:01002E71                 jbe     short loc\_1002E57  
.text:01002E73                 xor     ecx, ecx  
.text:01002E75                 cmp     \[eax+0E8h\], ecx  
.text:01002E7B  
.text:01002E7B loc\_1002E7B:                            ; CODE XREF: IsPEdotNetAssembly+46j  
.text:01002E7B                 setnz   cl  
.text:01002E7E                 mov     eax, ecx  
.text:01002E80                 retn  
.text:01002E80 IsPEdotNetAssembly endp  
  
如果它是.net assembly，开始代码会怎么做了？分析中。 

 4dd466d0e1e0cc123afbe504da2462ca