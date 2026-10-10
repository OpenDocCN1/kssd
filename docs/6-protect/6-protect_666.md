

# 关于 Code Virtualizer pcode 解密的一种方法

> 原文：[`www.pediy.com/kssd/pediy12/137265.html`](https://www.pediy.com/kssd/pediy12/137265.html)

【详细过程】  
   1.VM 的解密算法  
    一般情况下 Code Virtualizer 的解密有三个运算:add,sub,xor,有如下计算;  
      loads/b/w/d  
      operand eax/ax/al ebx/bx/bl         //与 key 进行计算  
      operand eax/ax/al imm32/imm16/imm8    
      operand eax/ax/al imm32/imm16/imm8  
      operand ebx/bx/bl eax/ax/al         //更新 key  
    其中 esi 指向 pcode 的数据，ebx 为解码 key,也就是进入虚拟机前压入堆栈的值:  
      push xxxxxxxx      //解码 key,正常等于 esi  
      jmp  VMStart  
   2.方法  
    让 VM 代码在 emulator 中执行，记录寄存器和堆栈状态，并 HOOK 相应的操作，应该可以得到 pcode 的 id  
   3.x86 emulator  
    应该有好多，本次采用的是 PyEmu,并结合 IDA 和 IDAPython(Python 为 2.6 版本)  
   4.明文 pcode 的获取  
   设置 emulator 初始环境，设置 EIP,指向虚拟机取指令处,为了使 emulator 顺利执行,安装异常处理函数，当发现异常时记录异常状态，重设 EIP，指向下一指令(有问题??),并在更新 key 处监视 eax/ax/al 的值，这就是明文 pcode,对 python 不熟悉(缩进实在无语，有可能是我的编辑器有问题),这个 emulator BUG 多多，不可能完整跑下来，修改了几处，以 dump\_wmimmc.sys 为例，简单测试了取指令:  

代码:

```
import sys
sys.path.append(r'D:\\reverse\\IDA\\python\\pyemu')
sys.path.append(r'D:\\reverse\\IDA\\python\\pyemu\\lib')
from PyEmu import \*
emu = IDAPyEmu()
#debug=0,1,2,>2
emu.debug(0)
vlizer\_start = SegByName(".reloc")
vlizer\_end   = SegEnd( vlizer\_start ) 
#load vlizer section
for i in range(vlizer\_start,vlizer\_end,4):
   emu.set\_memory(i,Dword(i))
print "\[\*\] Finished loading vlizer section into memory,Start at %x,End at %x,size:%x"%(vlizer\_start,vlizer\_end,vlizer\_end-vlizer\_start)
flag=1
instack=0
#eip=fetchcode
emu.set\_register("eip",0x10013bd6)
#ebx=key
emu.set\_register("ebx",0x10018cca)
#esi=ebx+delta
emu.set\_register("esi",0x10018cca)
#edi=vmctx=0x10013880+delta
emu.set\_register("edi",0x10013880)

while True:
    eip=emu.get\_register("eip")
    mnemonic=GetMnem(eip)
    op1=GetOpnd(eip,0)
    op2=GetOpnd(eip,1)
    #push eax,ax
    if mnemonic=="push":
       if GetOpType(eip,0)==1 and GetOperandValue(eip,0)==0:
          instack=1
    #pop eax,ax
    elif mnemonic=="pop":
       if GetOpType(eip,0)==1 and GetOperandValue(eip,0)==0:
          instack=0
    #push eax,ax
    #pop eax,ax
    elif mnemonic=="mov":
       if op1=="\[esp\]":
          if GetOpType(eip,1)==1 and GetOperandValue(eip,1)==0:
             instack=1
       elif op2=="\[esp\]":
          if GetOpType(eip,0)==1 and GetOperandValue(eip,0)==0:
             instack=0
    elif mnemonic=="add" or mnemonic=="sub" or mnemonic=="xor":
       if  GetOpType(eip,0)==1 and (GetOperandValue(eip,0)==0 or GetOperandValue(eip,0)==0x10):
          #operand eax/ax/al ebx/bx/bl
          if GetOpType(eip,1)==1 and (GetOperandValue(eip,1)==3 or GetOperandValue(eip,1)==0x13) and flag:
             print "%s %s,%s"%(mnemonic,op1,op2)
             flag=0
          #operand eax/ax/al imm32/imm16/imm8
          else:
             if not instack:
                if GetOpType(eip,1)==1:
                   print "%s %s,%x"%(mnemonic,op1,emu.get\_register(op2))
                elif GetOpType(eip,1)==5:
                   print "%s %s,%x"%(mnemonic,op1,GetOperandValue(eip,1))
    #operand ebx/bx/bl eax/ax/al
    if (GetOpType(eip,0)==1 and (GetOperandValue(eip,0)==3 or GetOperandValue(eip,0)==0x13)) and (GetOpType(eip,1)==1 and (GetOperandValue(eip,1)==0 or GetOperandValue(eip,1)==0x10)):
      print "%s %s,%s"%(mnemonic,op1,op2)
      print "handleindex:%x"%emu.get\_register("eax")
      break
    emu.execute()

```

 输出:  
    sub al,e9  
    add al,bl  
    add al,e9  
    xor al,d2  
    sub al,f1  
    add bl,al  
    handleindex:2d  
 最后清理后的取指令为:  
    loadsb  
    add al,bl  
    xor al,d2  
    sub al,f1  
    add bl,al  
    movzx eax,al  
    jmp dword ptr \[edi+eax\*4\]  
  测试可行，起个抛砖引玉的作用  
  附件是修改的 PyEmu 和测试文件

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || pyemu.rar  
 |