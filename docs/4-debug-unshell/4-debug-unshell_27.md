

# ProcDump 脚本

> 原文：[`www.pediy.com/kssd/pediy02/forum164.htm`](https://www.pediy.com/kssd/pediy02/forum164.htm)

 标题: 先把 PECompact1.34 和 UPX1.01 和 Aspack2.1 的脚本让你们测试。有哪位高手已经用 ProcDump 写了针对 AspProtect v1.0 的 Script？不知道写 Script 的方法是否可行？   
  
内容:   
\[4'My PECompact v1.34\]   
L1=OBJR   
L2=LOOK 9C,60   
L3=BP   
L4=WALK   
L5=OBJR   
L6=LOOK C3   
L7=BP   
L8=WALK   
L9=OBJR   
LA=LOOK 61,9D,50,68,??,??,??,??,C2,04,00   
LB=BP   
LC=STEP   
OPTL1=00000000   
OPTL2=01010001   
OPTL3=01010001   
OPTL4=00030000   
OPTL5=00000000   
  
  
\[2'My Aspack2.??-2.1\]   
L1=OBJR   
L2=LOOK EB   
L3=BP   
L4=WALK   
L5=OBJR   
L6=LOOK 61,75,08,B8,??,??,??,??,C2,0C,00   
L7=BP   
L8=STEP   
OPTL1=00000000   
OPTL2=01010001   
OPTL3=01010001   
OPTL4=00030000   
OPTL5=00000000   
  
\[1'My Upx1.01\]   
L1=OBJR   
L2=LOOK EB,10   
L3=WALK   
L4=OBJR   
L5=LOOK 61,E9   
L6=BP   
L7=STEP   
OPTL1=00000000   
OPTL2=01010001   
OPTL3=01010001   
OPTL4=00030000   
OPTL5=00000000   4dd466d0e1e0cc123afbe504da2462ca