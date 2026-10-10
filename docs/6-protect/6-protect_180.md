

# 2488 的解狗教程

> 原文：[`www.pediy.com/kssd/pediy04/kanxue419.htm`](https://www.pediy.com/kssd/pediy04/kanxue419.htm)

2488 的破解过程   
  
  
0A20:6352 2EC606866200    MOV      BYTE \[CS:6286\],00   
0A20:6358 803E420001      CMP      BYTE \[42\],01   
0A20:635D 90              NOP      
0A20:635E 90              NOP      
0A20:635F 90              NOP      
0A20:6360 0E              PUSH    CS   
0A20:6361 E8E60A          CALL    6E4A   
0A20:6364 3D0100          CMP      AX,01   
0A20:6367 EB67            JMP      SHORT 63D0   
0A20:6369 3D0600          CMP      AX,06   
0A20:636C 7416            JZ      6384   
0A20:636E 3D0000          CMP      AX,00   
0A20:6371 90              NOP      
0A20:6372 90              NOP      
0A20:6373 2EC606866200    MOV      BYTE \[CS:6286\],00   
0A20:6379 1E              PUSH    DS   
0A20:637A 0E              PUSH    CS   
0A20:637B 1F              POP      DS   
0A20:637C BA4A62          MOV      DX,624A    //这里会显示一些出错信息，   
                                              //通常的 DOS 程序在运行前不应该会显不出错信息。   
                                              //所以这里应该是没有用的，也就是说不要让程序走到这。   
0A20:637F B409            MOV      AH,09   
0A20:6381 CD21            INT      21   
0A20:6383 1F              POP      DS   
0A20:6384 9A236D9A11      CALL    6D23   
                            .   
                            .   
                            .   
0A20:63BD EB0A            JMP      SHORT 63C9   
0A20:63BF 90              NOP      
0A20:63C0 BAB862          MOV      DX,62B8  //这里一定是别的出错信息，   
                                            //因为 INT 21+AH=09 只要改变 DX 的值就可以显示不到的信息   
                                            //了，通常软件会做一个出错信息提示集合。   
0A20:63C3 EB04            JMP      SHORT 63C9   
0A20:63C5 90              NOP      
0A20:63C6 BAD562          MOV      DX,62D5 //同上   
0A20:63C9 B409            MOV      AH,09   
0A20:63CB CD21            INT      21   
0A20:63CD E9B000          JMP      6480   
0A20:63D0 2EC606866201    MOV      BYTE \[CS:6286\],01 //在这里放标志，以后会用到。   
0A20:63D6 803E190001      CMP      BYTE \[19\],01   
0A20:63DB 7525            JNZ      6402   
0A20:63DD 803E420000      CMP      BYTE \[42\],00   
0A20:63E2 7408            JZ      63EC   
0A20:63E4 2E803E866201    CMP      BYTE \[CS:6286\],01 //用到了上面的标志。   
0A20:63EA 7416            JZ      6402   
0A20:63EC 8C0E2800        MOV      \[28\],CS   
0A20:63F0 C70626008762    MOV      WORD \[26\],6287   
0A20:63F6 9A546D9A11      CALL    6D54              
0A20:63FB 0BC2            OR      AX,DX   
0A20:63FD 7403            JZ      6402   
0A20:63FF EB7F            JMP      SHORT 6480   
0A20:6401 90              NOP      
  
  
通常在解狗时会看到很多的标志保存和读取。这些都是关键不要放过。还有，程序运行需要 CALL，所以尽量让程序多走几个 CALL，这样破解的可能   
性比较大，另外，解狗也要经常试，有的时候也要靠运气。   
  
                                                                  罗降神   
                                                                2002.2.18  4dd466d0e1e0cc123afbe504da2462ca