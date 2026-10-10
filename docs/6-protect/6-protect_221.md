

# OD 自动提取 FLEXLM VENDERCODE 的插件

> 原文：[`www.pediy.com/kssd/pediy10/57943.html`](https://www.pediy.com/kssd/pediy10/57943.html)

曾经研究过一段时间的 FLEXLM,也写了自己的 VENDERCODER 生成器.  
后来因为 ECC 无法解决停了 2 年.   
    看到 Epsylon3 的脚本开发插件,小试一下,编了这个自动提取 VENDERCODE 的插件.  用 V6,V7,V8,V9 都试过几个,可以担取. V10 因为新的 l\_n36\_buf,不提供样本 License,CALL 的地址为 0,所以没继续试(也许提供一个假的 license 也可以正常工作,不过本人没试.  
    这是本人自己动手写的第一个,写得不好别骂我:).  
  
  
// Flexlm Vender data refatch    
// by Tylon  
// Dec . 30 . 2007  
  
var demo  
var avar  
var fadd  
var stp  
var ofile  
  
  lclr  
  mov ofile, "c:\\data.txt"  // noway to  let you define the out-file name  
    
  mov demo, 401000  
  find demo, #21436587#      // use 0x87654321 as 1st ancor  
  mov demo, $RESULT   
  find demo, #78563412#      // the next demo Skey,  
  mov demo, $RESULT  
    
    
  sub demo, 20  
  add demo, 6  
  bp demo                    // adjust to the call  
    
    
  run  
  sti                       // step-into the call  
    
  mov stp,eip                //back up the start eip of the subroutingfor later searching   
  dm \[esp+08\], 14, ofile    // get vendor name  
    
  mov avar, \[esp+0c\]        // v\_ata  
  mov demo, 0;  
vdata:  
  itoa \[avar\];  
  wrta ofile, $RESULT  
  add avar, 4  
  add  demo, 4  
  cmp demo, 164  
  jb vdata  
    
  wrta ofile,"\\r\\n"  
    
//  mov demo, eip+9      //we don't need thisone  
//  itoa \[demo\]  
//  wrta ofile, $RESULT  
  
  findop eip, #ff??#, ff  
  mov fadd, $RESULT  
    
//  if not do this check, it'll lead to good sub    
//  cmp \[avar\],0  
//  je erra  
  
  bc  
  bp $RESULT          //set bp on the call \[???\]   
    
  sub fadd, stp;  
  findop eip, #2500800000#, fadd    // find cmp eax  
  cmp $RESULT,0  
  je findother  
  mov fadd,$RESULT   
//  repl fadd, #2500800000#, #0500800000#,10  
  jmp torun  
findother:  
  findop eip, #81??00800000#,fadd  
  mov fadd, $RESULT  
//  repl fadd, #81??00800000#, #8B??00800000#,12  
    
torun:  
  add fadd,4  
  repl fadd, #74??#, #9090#, 10  
  run  
  sti      //now we are inside the destinely aera  
  wrta ofile, "magic\_4:  "  
  
  mov demo, eip+9            //magic\_4  
  itoa \[demo\]  
  wrta ofile, $RESULT  
    
  wrta ofile, "\\r\\n"  
  wrta ofile, "can't get turns directly,so just dump the asm-code:\\r\\n"  
  
  
//  find eip, #0FBE????????0FBE#,2000  
//  mov stp, $RESULT   
//  find eip, #0FBE??????????0FBE#,2000  
//  mov fadd, $RESULT  
  
//  cmp stp, 0  
//  je start\_2  
//  cmp fadd, 0  
//  je start\_1  
//  cmp fadd, stp  
//  jb start\_2  
//start\_1:  
//  mov fadd, stp  
//  jmp here  
//start\_2:  
//  go fadd  
    
//here:  
  
  findop eip, #0FBE??D?#,2000  
  mov fadd, $RESULT  
    
  opcode fadd  
  wrta ofile, $RESULT  
  wrta ofile, $RESULT\_1  
  add fadd,$RESULT\_2  
  opcode fadd  
  wrta ofile, $RESULT  
  wrta ofile, $RESULT\_1  
  add fadd,$RESULT\_2  
  opcode fadd  
  wrta ofile, $RESULT  
  wrta ofile, $RESULT\_1  
  add fadd,$RESULT\_2  
  opcode fadd  
  wrta ofile, $RESULT  
  wrta ofile, $RESULT\_1  
  add fadd,$RESULT\_2  
  opcode fadd  
  wrta ofile, $RESULT  
  wrta ofile, $RESULT\_1  
  add fadd,$RESULT\_2  
  opcode fadd  
  wrta ofile, $RESULT  
  wrta ofile, $RESULT\_1  
  add fadd,$RESULT\_2  
  opcode fadd  
  wrta ofile, $RESULT  
  wrta ofile, $RESULT\_1  
  add fadd,$RESULT\_2  
  opcode fadd  
  wrta ofile, $RESULT  
  wrta ofile, $RESULT\_1  
  add fadd,$RESULT\_2  
  opcode fadd  
  wrta ofile, $RESULT  
  wrta ofile, $RESULT\_1  
  add fadd,$RESULT\_2  
  opcode fadd  
  wrta ofile, $RESULT  
  wrta ofile, $RESULT\_1  
  add fadd,$RESULT\_2  
  opcode fadd  
  wrta ofile, $RESULT  
  wrta ofile, $RESULT\_1  
//  add fadd,$RESULT\_2  
    
msg "Congradulation!\\r\\n  we've got our vendor data to\\r\\n "C:\\data.txt"\\r\\n Thanks for using! "    
ret  
    
    
  
data\_b:  
  msg "yes"  
  jmp cont1   
ret    
erra:  
  ask "Error: ver > 10, continue handly?"  
ret  
  
  
提到的数据可以用 key10.exe 等工具生成最终的 VENDERCODE.  
  
如果真有需要,本人也可以把自己做的生成器发上来. 

晕,我不知道有没有上传的权限.  
  
呵呵,还好.原码我就不上传了,写得很乱.

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || lmgenkey.rar  
 |