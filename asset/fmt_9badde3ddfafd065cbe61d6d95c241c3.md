
# 五笔打字员 2.5

> 原文：[`www.pediy.com/kssd/pediy06/pediy6063.htm`](https://www.pediy.com/kssd/pediy06/pediy6063.htm)

对象：五笔打字员 2.5  
破解人：lordor\[BCG\]  
目的：初学破解，属技术交流，无其它目的，请不要任意散布或用用商业用途。  
工具：`wktvbdebuger1.4e`, `exdec`

假设：

机器码：`818314132231985`

注册码：`987654321`

前几天在学习破解 `vb` 的 `p-code` 的程序破解，有所体会，所以公布这篇笔记。

好了，在 `wktvbdebuger` 中，在反汇编窗口，右击，选 `bpx api list`, 找 `rtcLeftCharVar` 函数，并在上面下断，运行注册，断下来了，看一面地址 `51c39e`，好运行反汇篇程序 `exdec`, 查找地址 `51c39e`，向上找这一 `call` 的第一条

语句，如下：

```
Proc: 51d0e0

51BCD8: 00 LargeBos               
51BCDA: 00 LargeBos               
51BCDC: 4b onErrorGoto           
51BCDF: 00 LargeBos               
```

好了，可以在 `wktvbdebuger` 中在 `51BCD8` 处下断，运行如下：

```
Proc: 51d0e0

51BCD8: 00 LargeBos     ＝＝＞不知这个含义              
51BCDA: 00 LargeBos               
51BCDC: 4b onErrorGoto           
51BCDF: 00 LargeBos               
51BCE1: f5 LitI4:                 0xbb8  3000  (....)  
51BCE6: 71 FStR4                  local_008C
51BCE9: 00 LargeBos               
51BCEB: 27 LitVar_Missing         
51BCEE: 0a ImpAdCallFPR4:         
51BCF3: 35 FFree1Var              local_00B0
51BCF6: 00 LargeBos               
51BCF8: 27 LitVar_Missing         
51BCFB: 0a ImpAdCallFPR4:         
51BD00: 73 FStFPR4                
51BD03: f4 LitI2_Byte:            0x18  24  (.)==>装入一个字节值 24
51BD05: eb CR8I2        ＝＞数据类型转换为 double       
51BD06: 6e FLdFPR4                
51BD09: b3 MulR8                  
51BD0A: Lead0/e6 FnIntR8          
51BD0C: f4 LitI2_Byte:            0x1  1  (.)
51BD0E: eb CR8I2                  
51BD0F: ab AddR8                  
51BD10: e5 CI2R8                  
51BD11: 70 FStI2                  local_0086
51BD14: 35 FFree1Var              local_00B0
51BD17: 00 LargeBos               
51BD19: 6b FLdI2                  local_0086
51BD1C: 70 FStI2                  local_00B6
51BD1F: 00 LargeBos               
51BD21: 6b FLdI2                  local_00B6
51BD24: f4 LitI2_Byte:            0x1  1  (.)
51BD26: c6 EqI2                   
51BD27: 1c BranchF:               51BD5F==>跳到 51BD5F
51BD2A: 00 LargeBos               
51BD2C: 6c ILdRf                  local_008C
51BD2F: 71 FStR4                  local_0090
51BD32: 00 LargeBos               
51BD34: 6c ILdRf                  local_008C
51BD37: f5 LitI4:                 0x168  360  (...h)
51BD3C: aa AddI4                  
51BD3D: 71 FStR4                  local_0090
51BD40: 00 LargeBos               
51BD42: 6c ILdRf                  local_008C
51BD45: f5 LitI4:                 0x2d0  720  (....)
51BD4A: aa AddI4                  
51BD4B: 71 FStR4                  local_0090
51BD4E: 00 LargeBos               
51BD50: 6c ILdRf                  local_008C
51BD53: f5 LitI4:                 0x438  1080  (...8)
51BD58: aa AddI4                  
51BD59: 71 FStR4                  local_0090
51BD5C: 1e Branch:                51c31c
51BD5F: 00 LargeBos               
51BD61: 6b FLdI2                  local_00B6
51BD64: f4 LitI2_Byte:            0x2  2  (.)
51BD66: c6 EqI2      =>与 2 是否相等               
51BD67: 1c BranchF:               51BD9F＝＞跳到 51BD9F
51BD6A: 00 LargeBos               
51BD6C: 6c ILdRf                  local_008C
51BD6F: 71 FStR4                  local_0090
51BD72: 00 LargeBos               
51BD74: 6c ILdRf                  local_008C
51BD77: f5 LitI4:                 0x168  360  (...h)
51BD7C: aa AddI4                  
51BD7D: 71 FStR4                  local_0090
51BD80: 00 LargeBos               
51BD82: 6c ILdRf                  local_008C
51BD85: f5 LitI4:                 0x2d0  720  (....)
51BD8A: aa AddI4                  
51BD8B: 71 FStR4                  local_0090
51BD8E: 00 LargeBos               
51BD90: 6c ILdRf                  local_008C
51BD93: f5 LitI4:                 0x438  1080  (...8)
51BD98: aa AddI4                  
51BD99: 71 FStR4                  local_0090
51BD9C: 1e Branch:                51c31c
51BD9F: 00 LargeBos               
51BDA1: 6b FLdI2                  local_00B6
51BDA4: f4 LitI2_Byte:            0x3  3  (.)
51BDA6: c6 EqI2           ＝＞3 是否等于 8      
51BDA7: 1c BranchF:               51BDDF＝＞不等则跳到 51BDDF
51BDAA: 00 LargeBos               
51BDAC: 6c ILdRf                  local_008C
51BDAF: 71 FStR4                  local_0090
51BDB2: 00 LargeBos               
51BDB4: 6c ILdRf                  local_008C
51BDB7: f5 LitI4:                 0x168  360  (...h)
51BDBC: aa AddI4                  
51BDBD: 71 FStR4                  local_0090
51BDC0: 00 LargeBos               
51BDC2: 6c ILdRf                  local_008C
51BDC5: f5 LitI4:                 0x2d0  720  (....)
51BDCA: aa AddI4                  
51BDCB: 71 FStR4                  local_0090
51BDCE: 00 LargeBos               
51BDD0: 6c ILdRf                  local_008C
51BDD3: f5 LitI4:                 0x438  1080  (...8)
51BDD8: aa AddI4                  
51BDD9: 71 FStR4                  local_0090
51BDDC: 1e Branch:                51c31c
51BDDF: 00 LargeBos               
51BDE1: 6b FLdI2                  local_00B6
51BDE4: f4 LitI2_Byte:            0x4  4  (.)
51BDE6: c6 EqI2           ＝＞8 与 4 是否等
51BDE7: 1c BranchF:               51BE1F＝＞不等则跳
51BDEA: 00 LargeBos               
51BDEC: 6c ILdRf                  local_008C
51BDEF: 71 FStR4                  local_0090
51BDF2: 00 LargeBos               
51BDF4: 6c ILdRf                  local_008C
51BDF7: f5 LitI4:                 0x168  360  (...h)
51BDFC: aa AddI4                  
51BDFD: 71 FStR4                  local_0090
51BE00: 00 LargeBos               
51BE02: 6c ILdRf                  local_008C
51BE05: f5 LitI4:                 0x2d0  720  (....)
51BE0A: aa AddI4                  
51BE0B: 71 FStR4                  local_0090
51BE0E: 00 LargeBos               
51BE10: 6c ILdRf                  local_008C
51BE13: f5 LitI4:                 0x438  1080  (...8)
51BE18: aa AddI4                  
51BE19: 71 FStR4                  local_0090
51BE1C: 1e Branch:                51c31c
51BE1F: 00 LargeBos               
51BE21: 6b FLdI2                  local_00B6
51BE24: f4 LitI2_Byte:            0x5  5  (.)
51BE26: c6 EqI2           ＝＝＞是否等 5   
51BE27: 1c BranchF:               51BE5F＝不则跳
51BE2A: 00 LargeBos               
51BE2C: 6c ILdRf                  local_008C
51BE2F: 71 FStR4                  local_0090
51BE32: 00 LargeBos               
51BE34: 6c ILdRf                  local_008C
51BE37: f5 LitI4:                 0x168  360  (...h)
51BE3C: aa AddI4                  
51BE3D: 71 FStR4                  local_0090
51BE40: 00 LargeBos               
51BE42: 6c ILdRf                  local_008C
51BE45: f5 LitI4:                 0x2d0  720  (....)
51BE4A: aa AddI4                  
51BE4B: 71 FStR4                  local_0090
51BE4E: 00 LargeBos               
51BE50: 6c ILdRf                  local_008C
51BE53: f5 LitI4:                 0x438  1080  (...8)
51BE58: aa AddI4                  
51BE59: 71 FStR4                  local_0090
51BE5C: 1e Branch:                51c31c
51BE5F: 00 LargeBos               
51BE61: 6b FLdI2                  local_00B6
51BE64: f4 LitI2_Byte:            0x6  6  (.)
51BE66: c6 EqI2                 ＝＞等于 6 吗
51BE67: 1c BranchF:               51BE9F＝＞否则跳
51BE6A: 00 LargeBos               
51BE6C: 6c ILdRf                  local_008C
51BE6F: 71 FStR4                  local_0090
51BE72: 00 LargeBos               
51BE74: 6c ILdRf                  local_008C
```
