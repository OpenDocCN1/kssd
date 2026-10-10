

# 代码自我清除，自我加密，解密的实现

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-527.htm`](https://www.pediy.com/kssd/pediy07/pediy7-527.htm)

最近开发壳，但是我的壳主要的安全不在题目上写的，所以就公开在 delphi 中实现代码自我加密解密清除的方法，高手就不用看了，很简单的。  
首先我们要定义几个过程，  
procedure EncryptCode(Badress,size,key:cardinal);//Badress 为加密起始地址，size 为加密大小,key 为加密密钥  
var  
  CTemp:cardinal;  
begin  
  Virtulloc(pointer(Badress),Size,Page\_readwrite,Ctemp);//函数名级不起来拉。  
  asm  
    push eax;  
    push ebx;  
    push ecx;  
    mov eax,badress;  
    mov ebx,size;  
    mov ecx,key;  
    xor dword ptr ds:\[eax\],ecx;  
    add eax,4;  
    dec ebx;  
    db $75,fc;//这里可能不正确，反正向 xor dword ptr ds:\[eax\],ecx;跳就可以拉  
    pop ecx;  
    pop ebx;  
    pop eax;  
  end;  
end;  
  
加密和解密是同一段代码。  
代码清除  
procedure CleanCode(Badress,size,key:cardinal);//Badress 为加密起始地址，size 为加密大小,key 为加密密钥  
var  
  CTemp:cardinal;  
begin  
  Virtulloc(pointer(Badress),Size,Page\_readwrite,Ctemp);//函数名级不起来拉。  
  asm  
    push eax;  
    push ebx;  
    push ecx;  
    mov eax,badress;  
    mov ebx,size;  
    mov ecx,key;  
    xor dword ptr ds:\[eax\],0;  
    add eax,1;  
    dec ebx;  
    db $75,fc;//这里可能不正确，反正向 xor dword ptr ds:\[eax\],ecx;跳就可以拉  
    pop ecx;  
    pop ebx;  
    pop eax;  
  end;  
end;  
现在就可以调用拉。调用方法很简单，但是要注意的是，不能让加密代码加密自己过程内调用到的。当然对于这个 badress 我们用  
call XXXXXXXX;  
pop eax;  
搞定。  
当然还有很多要做的我就不多说拉。相信大家都知道。 

 4dd466d0e1e0cc123afbe504da2462ca