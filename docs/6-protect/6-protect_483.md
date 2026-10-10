

# 在高级语言中隐藏 Call 指令

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-526.htm`](https://www.pediy.com/kssd/pediy07/pediy7-526.htm)

今天写壳的时候想到了一种不需要手动修改直接编译就隐藏了 call 指令的方法，和大家一起分享。先还是说说手动隐藏 call 的简单方法。(我习惯用 Delphi 所以文中是 delphi 代码)  
我在网吧，电脑没有带出来，所以即想即写，有什么错误的地方还望指出来。  
手动隐藏方法：  
比如我们定义这样的一个过程。  
procedure test;  
begin  
end;  
然后在另外一个过程中调用，  
begin  
   test;  
end;  
一般编译后我们就会看到相应的  
00000001 call XXXXXXXX (00000001 和 XXXXXXXX 都是假设的地址)  
我们最常用的隐藏方法就是把代码写成这样  
begin  
   test;  
   asm DB $90,$90,$90,$90,$90,$90 end;  
end;  
我们需要利用 6 个 nop 来为以后的隐藏做准备。  
然后等编译后会出现这样的代码  
00000001 call XXXXXXXX  
00000006 nop  
00000007 nop  
00000008 nop  
00000009 nop  
00000010 nop  
00000011 nop  
  
然后我们手工需要做的是修改这段代码成如下  
00000001 push 00000012  
00000006 push XXXXXXXX  
00000011 ret  
这样代码长度刚好一样  
这样就达到了隐藏效果，但是这样每次都要手工修改麻烦，下面看另外一种隐藏方法  
  
自动隐藏：  
在上面我们看到了手动隐藏的方法，下面来看自动隐藏方法，同样以 test 过程为例。  
我们都知道这样的一种获取 eip 的方法  
00000001 call 00000006  
00000006 pop eax  
在看看这段代码的机器码  
00000001 FF00000000  
00000006 58  
{机器吗可能有错误记性不好}  
那么我们定义两个变量  
var  
  TestEip:cardinal;  
  TestPoi:pointer;  
先不要问用来做什么，马上就清楚拉。  
我们现在把调用代码改以下  
begin  
  Asm  
    DB $FF,$00,$00,$00,$00,$58;  
    Add Eax,$20;{这里要看，这个$20 是这段代码的大小，反正要让 Eax 刚好只向 Ret 后一句的地址}  
    mov TestEip,eax;  
  end;  
  TestPoi:=@Test;  
  asm   
    Push TestEip;  
    Push TestPoi;  
    ret  
  end;  
然后编译，怎么样？call 指令是不是不见了？  
但是这样有一个麻烦，那就是万一 Test 带参数怎么办？  
比如：  
procedure test(a,b,c:cardinal);  
begin  
end;  
那么我们也可以这样调用  
  
begin  
  Asm  
    DB $FF,$00,$00,$00,$00,$58;  
    Add Eax,$2F;{这里要看，这个$20 是这段代码的大小，反正要让 Eax 刚好只向 Ret 后一句的地址}  
    mov TestEip,eax;  
    mov eax,参数 1;  
    mov ecx,参数 2;  
    mov edx,参数 3;  
  end;  
  TestPoi:=@Test;  
  asm  
    mov eax,testeip;   
    Push TestEip;  
    Push TestPoi;  
    ret  
  end;  
只写这么多拉。没有什么技术含量。有什么更好的方法拿出来大家分享一下呀。  
![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg) ![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg) ![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg) ![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg)

 4dd466d0e1e0cc123afbe504da2462ca