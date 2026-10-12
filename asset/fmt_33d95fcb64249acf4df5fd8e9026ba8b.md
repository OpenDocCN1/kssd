
```asm
006144D8  |.  E8 B309DFFF   CALL DEPOT.00404E90

006144DD  |.  8BF0         MOV ESI,EAX

006144DF  |.  85F6         TEST ESI,ESI

006144E1  |.  7E 26        JLE SHORT DEPOT.00614509             //是否有用户名?过去这里下面开始计算

006144E3  |.  BB 01000000  MOV EBX,1                            //EBX=1 计数器

006144E8  |>  8D4D EC      /LEA ECX,DWORD PTR SS:[EBP-14]

006144EB  |.  8B45 FC      |MOV EAX,DWORD PTR SS:[EBP-4]        //EAX=用户名

006144EE  |.  0FB64418 FF  |MOVZX EAX,BYTE PTR DS:[EAX+EBX-1]   //取其中一位的 ASCII 码(关键看计数器)

006144F3  |.  33D2         |XOR EDX,EDX                         //清 EDX,等下用来保存

006144F5  |.  E8 EE5BDFFF  |CALL DEPOT.0040A0E8                 //由刚才的 ASCII 码转为字符.(数字字符,也就是说刚才你那一位 ASCII 码是 31,那么这里就转为纯字符的 31!)

006144FA  |.  8B55 EC      |MOV EDX,DWORD PTR SS:[EBP-14]

006144FD  |.  8D45 F8      |LEA EAX,DWORD PTR SS:[EBP-8]        //EAX=转换结果

00614500  |.  E8 9309DFFF  |CALL DEPOT.00404E98                 //排列起来,先来后到!

00614505  |.  43           |INC EBX                             //计数器+1

00614506  |.  4E           |DEC ESI

00614507  |.^ 75 DF         \JNZ SHORT DEPOT.006144E8           //循环

00614509  |>  \8B45 F8     MOV EAX,DWORD PTR SS:[EBP-8]         //排列后的结果.我输入的用户名是 12345678,到这里结果就是 3132333435363738

0061450C  |.  E8 7F09DFFF  CALL DEPOT.00404E90                  //取位数

00614511  |.  8BF0         MOV ESI,EAX

00614513  |.  85F6         TEST ESI,ESI

00614515  |.  7E 2C        JLE SHORT DEPOT.00614543             //是否有?下面进行第二轮转换!

00614517  |.  BB 01000000  MOV EBX,1                            //熟悉的计数器

0061451C  |>  8B45 F8      /MOV EAX,DWORD PTR SS:[EBP-8]        //EAX=上边转换的结果

0061451F  |.  E8 6C09DFFF  |CALL DEPOT.00404E90                 //取位数

00614524  |.  2BC3         |SUB EAX,EBX                         //看清楚,位数-计数器,可以知道这次是反着取!

00614526  |.  8B55 F8      |MOV EDX,DWORD PTR SS:[EBP-8]        //EDX=上边转换的结果

00614529  |.  8A1402       |MOV DL,BYTE PTR DS:[EDX+EAX]        //反着取其中一位 ASCII 码

0061452C  |.  8D45 E8      |LEA EAX,DWORD PTR SS:[EBP-18]

0061452F  |.  E8 8408DFFF  |CALL DEPOT.00404DB8                 //取 ASCII 码反方向

00614534  |.  8B55 E8      |MOV EDX,DWORD PTR SS:[EBP-18]

00614537  |.  8D45 F4      |LEA EAX,DWORD PTR SS:[EBP-C]

0061453A  |.  E8 5909DFFF  |CALL DEPOT.00404E98                 //反着写,比如说我刚才的是 3132333435363738,经过这里循环以后就变成 8373635343332313

0061453F  |.  43           |INC EBX                             //计数器+1

00614540  |.  4E           |DEC ESI

00614541  |.^ 75 D9         \JNZ SHORT DEPOT.0061451C           //循环

00614543  |>  8D45 F8      LEA EAX,DWORD PTR SS:[EBP-8]         //EAX=3132333435363738  第一次循环结果

00614546  |.  50           PUSH EAX

00614547  |.  B9 04000000  MOV ECX,4

0061454C  |.  BA 01000000  MOV EDX,1

00614551  |.  8B45 F4      MOV EAX,DWORD PTR SS:[EBP-C]         //EAX=8373635343332313  第二次循环结果

00614554  |.  E8 8F0BDFFF  CALL DEPOT.004050E8                  //截取前 4 位

00614559  |.  8D45 F4      LEA EAX,DWORD PTR SS:[EBP-C]

0061455C  |.  50           PUSH EAX

0061455D  |.  B9 04000000  MOV ECX,4

00614562  |.  BA 05000000  MOV EDX,5

00614567  |.  8B45 F4      MOV EAX,DWORD PTR SS:[EBP-C]         //EAX=8373635343332313  第二次循环结果

0061456A  |.  E8 790BDFFF  CALL DEPOT.004050E8                  //再截取 4 位

0061456F  |.  8B45 F8      MOV EAX,DWORD PTR SS:[EBP-8]         //EAX=8373

00614572  |.  E8 1909DFFF  CALL DEPOT.00404E90                  //取位数

00614577  |.  83F8 04      CMP EAX,4

0061457A  |.  7D 2F        JGE SHORT DEPOT.006145AB             //大于等于 4 跳,我的就是等于~不用经过下面计算~(也就是说,当你输入的用户名是 2 位以下的时候,要经过下面第三个循环的计算!)

0061457C  |.  8B45 F8      MOV EAX,DWORD PTR SS:[EBP-8]

0061457F  |.  E8 0C09DFFF  CALL DEPOT.00404E90

00614584  |.  8BD8         MOV EBX,EAX

选择部分(1)

00614586  |.  83FB 03      CMP EBX,3                            //这里开始如果用户名小于三位，进行下面补位运算

00614589  |.  7F 20        JG SHORT DEPOT.006145AB

0061458B  |>  8D4D E4      /LEA ECX,DWORD PTR SS:[EBP-1C]

★   这里无非就一种结果:

1>就是用户名只有是 1 位的时候,转成 ASCII 码就是 2 位,EAX=2 计算,结果固定!

0061458E  |.  8BC3         |MOV EAX,EBX                         //此时取你用户名 ASCII 码位数

00614590  |.  C1E0 02      |SHL EAX,2                           //逻辑左移 2 位(当乘法用)

00614593  |.  33D2         |XOR EDX,EDX

00614595  |.  E8 4E5BDFFF  |CALL DEPOT.0040A0E8                 //结果换成字符

0061459A  |.  8B55 E4      |MOV EDX,DWORD PTR SS:[EBP-1C]

0061459D  |.  8D45 F8      |LEA EAX,DWORD PTR SS:[EBP-8]

006145A0  |.  E8 F308DFFF  |CALL DEPOT.00404E98

006145A5  |.  43           |INC EBX

006145A6  |.  83FB 04      |CMP EBX,4

006145A9  |.^ 75 E0         \JNZ SHORT DEPOT.0061458B           //循环

ˉ

006145AB  |>  8B45 F4      MOV EAX,DWORD PTR SS:[EBP-C]         //上边第二次截取的结果 EAX=6353

006145AE  |.  E8 DD08DFFF  CALL DEPOT.00404E90                  //再取

006145B3  |.  83F8 04      CMP EAX,4

006145B6  |.  7D 2F        JGE SHORT DEPOT.006145E7             //大于等于 4 跳,我还是等于~不用经过下面计算~(也就是说,当你输入用户名时候大于 2 位又小于 4 位的时候,要经过下面第三个循环的计算!这里不明白的朋友可以综合上边的解释看一下,我就不占篇幅了!)

006145B8  |.  8B45 F4      MOV EAX,DWORD PTR SS:[EBP-C]

006145BB  |.  E8 D008DFFF  CALL DEPOT.00404E90

006145C0  |.  8BD8         MOV EBX,EAX

006145C2  |.  83FB 03      CMP EBX,3

006145C5  |.  7F 20        JG SHORT DEPOT.006145E7

选择部分(2)

006145C7  |>  8D4D E0      /LEA ECX,DWORD PTR SS:[EBP-20]       //此时取你经过上边运算剩余用户名 ASCII 码位数

★   这里无非两种结果:

1>是用户名是 1,2 位,到这里已经没有剩余,此时 EAX=0 去计算.得到结果一样

2>是用户名是 3 位,到这里还有 1 位用户名剩余,转为 ASCII 码就是 2 位剩余.此时 EAX=2.结果相应改变!

006145CA  |.  8BC3         |MOV EAX,EBX

006145CC  |.  C1E0 02      |SHL EAX,2                           //逻辑左移 2 位(当乘法用)

006145CF  |.  33D2         |XOR EDX,EDX

006145D1  |.  E8 125BDFFF  |CALL DEPOT.0040A0E8                 //结果转换为字符!

006145D6  |.  8B55 E0      |MOV EDX,DWORD PTR SS:[EBP-20]

006145D9  |.  8D45 F4      |LEA EAX,DWORD PTR SS:[EBP-C]

006145DC  |.  E8 B708DFFF  |CALL DEPOT.00404E98

006145E1  |.  43           |INC EBX

006145E2  |.  83FB 04      |CMP EBX,4

006145E5  |.^ 75 E0         \JNZ SHORT DEPOT.006145C7

ˉ

006145E7  |>  8D45 F0      LEA EAX,DWORD PTR SS:[EBP-10]

006145EA  |.  BA 74466100  MOV EDX,DEPOT.00614674                ;  ASCII "DEPw245d54k"  //出现字符串!       ★  注册码中的字符

006145EF  |.  E8 7C06DFFF  CALL DEPOT.00404C70

006145F4  |.  8D45 DC      LEA EAX,DWORD PTR SS:[EBP-24]

006145F7  |.  50           PUSH EAX

006145F8  |.  B9 04000000  MOV ECX,4

006145FD  |.  BA 01000000  MOV EDX,1

00614602  |.  8B45 F0      MOV EAX,DWORD PTR SS:[EBP-10]        //EAX=DEPw245d54k

00614605  |.  E8 DE0ADFFF  CALL DEPOT.004050E8                    //再取前 4 位

0061460A  |.  FF75 DC      PUSH DWORD PTR SS:[EBP-24]             //结果压栈,下 D 命令可看到 DEPw

0061460D  |.  68 88466100  PUSH DEPOT.00614688                    //下 D 可看到-字符(看来注册码有它连接)

00614612  |.  FF75 F8      PUSH DWORD PTR SS:[EBP-8]              //下 D 可看到 8373,是什么我不用多说了吧?不明白的看上边!

00614615  |.  8D45 D8      LEA EAX,DWORD PTR SS:[EBP-28]

00614618  |.  50           PUSH EAX

00614619  |.  B9 05000000  MOV ECX,5                              //看到这里我想不难猜到这两句话的意思了,EDX 5 就是取的位数,而 ECX 5 就是步长,相当于从字符串的第几位取~

0061461E  |.  BA 05000000  MOV EDX,5

00614623  |.  8B45 F0      MOV EAX,DWORD PTR SS:[EBP-10]        //EAX=DEPw245d54k(看了上边两句,也就是说这次从这串字符的第 5 位开始,一共取 5 位)

00614626  |.  E8 BD0ADFFF  CALL DEPOT.004050E8                    //实现

0061462B  |.  FF75 D8      PUSH DWORD PTR SS:[EBP-28]             //下 D 可到看结果 245d5

0061462E  |.  68 88466100  PUSH DEPOT.00614688                    //字符-

00614633  |.  FF75 F4      PUSH DWORD PTR SS:[EBP-C]              //下 D 可看到 6353

00614636  |.  8BC7         MOV EAX,EDI

00614638  |.  BA 06000000  MOV EDX,6
```
