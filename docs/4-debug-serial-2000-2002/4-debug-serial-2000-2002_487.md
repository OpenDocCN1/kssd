
# SWFBrowser 2.93

> 原文：[`www.pediy.com/kssd/pediy04/kanxue340.htm`](https://www.pediy.com/kssd/pediy04/kanxue340.htm)

用 `Ollydbg` 破解 `SWFBrowser 2.93`

作者: `CoolWolF[SCD]`

破解时间: `2001-1-10`

破解工具: `WIN2K` 环境下 `Ollydbg1.05a` `W32dasm9.0` 汉化版

作者主页: `http://swifftools.com/stools`

难度: 中低

说明: 一个极好的 `Flash` 工具,可以把 `SWF` 文件中的各种元素提取出来,闪客必备.

=================================================================

以下文字纯粹是供各位爱好逆向工程同好参考交流,请尊重软件作者的权益

=================================================================

这个我记得在精华区好像已经有了,但还是想再贴一次用 `Ollydbg` 破解的过程,因为越来越发现 `Ollydbg` 的方便之处,有什么不对地方请各位老大指正.

执行程序,弹出注册菜单,按 `Register` 按钮,输入用户名:`CoolWolF[SCD]` 注册码:`650033` 程序提示 `The serial number is invalid` 之后进入主画面.

用 `W32DSM` 打开 `SWFBrowser.exe`(没有加壳),找到:

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A99C8(C)

|

```
:004A9A60 6A00               push 00000000
:004A9A62 668B0DA49A4A00     mov cx, word ptr [004A9AA4]
:004A9A69 B201               mov dl, 01

* Possible StringData Ref from Code Obj ->"The serial number is invalid."

                                  |
:004A9A6B B8409B4A00           mov eax, 004A9B40
:004A9A70 E823B6FAFF           call 00455098
:004A9A75 8BC3               mov eax, ebx
:004A9A77 E8E4FCF9FF           call 00449760
```

很明显是从 `004A99C8` 跳过来的,按 `Shift+F12` 转到 `004A99C8`:

```
:004A99BB 8B55F0              mov edx, dword ptr [ebp-10]
:004A99BE 8BC3               mov eax, ebx
:004A99C0 59                 pop ecx
:004A99C1 E806FEFFFF         call 004A97CC //问题之所在
:004A99C6 84C0               test al, al
:004A99C8 0F8492000000       je 004A9A60 //跳走就完蛋
:004A99CE 6A00               push 00000000
:004A99D0 668B0DA49A4A00     mov cx, word ptr [004AA4]
:004A99D7 B202               mov dl, 02

* Possible StringData Ref from Code Obj ->"Thank you for registering SWF "
                                ->"Browser!"

                                  |
:004A99D9 B8B09A4A00           mov eax, 004A9AB0
:004A99DE E8B5B6FAFF           call 00455098
:004A99E3 B201               mov dl, 01
:004A99E5 A1F4E84400          mov eax, dword ptr [0044E8F4]
:004A99EA E87150FAFF           call 0044EA60
:004A99EF 8BF0               mov esi, eax
:004A99F1 BA01000080          mov edx, 80000001
:004A99F6 8BC6               mov eax, esi
:004A99F8 E83F51FAFF           call 0044EB3C
:004A99FD C6460C01            mov [esi+0C], 01
:004A9A01 B101               mov cl, 01

* Possible StringData Ref from Code Obj ->"Software\\Grooveware Multimedia\\SWF "
                                ->"Browser\\Registration"

                                  |
:004A9A03 BAE09A4A00           mov edx, 004A9AE0
:004A9A08 8BC6               mov eax, esi
:004A9A0A E87152FAFF           call 0044EC80
```

这样就看得比较清楚了,程序先比对用户名和注册码是否匹配,如果是则放入注册表的`[HKEY_CURRENT_USER\Software\Grooveware Multimedia\SWF Browser\Registration]`键,以后每次启动的时候进行检查.

上面的代码很好理解,`004A97CC` 肯定是一个关键 `Call`,那么我们现在可以打开 `Ollydbg`,加载 `SWFBrowser.exe` 运行

在 `004A99C1` 处按 `F2` 下断, 运行程序输入任意的用户名和注册码,按确定后程序被中断:

```
004A99C0  |. 59               POP ECX
004A99C1  |. E8 06FEFFFF      CALL SWFBrows.004A97CC //断在这里
004A99C6  |. 84C0             TEST AL,AL
```

按 `F7` 跟进 然后 `F8` 慢慢往下走

```
004A97CC  $ 55               PUSH EBP
004A97CD  . 8BEC             MOV EBP,ESP
004A97CF  . 6A 00            PUSH 0
004A97D1  . 6A 00            PUSH 0
004A97D3  . 6A 00            PUSH 0
004A97D5  . 6A 00            PUSH 0
004A97D7  . 6A 00            PUSH 0
004A97D9  . 6A 00            PUSH 0
004A97DB  . 6A 00            PUSH 0
004A97DD  . 53               PUSH EBX
004A97DE  . 56               PUSH ESI
004A97DF  . 57               PUSH EDI
004A97E0  . 894D F8          MOV DWORD PTR SS:[EBP-8],ECX
004A97E3  . 8955 FC          MOV DWORD PTR SS:[EBP-4],EDX
004A97E6  . 8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4] //用户名到 EAX
004A97E9  . E8 1AA8F5FF      CALL SWFBrows.00404008 //检查用户名长度的合法性
004A97EE  . 8B45 F8          MOV EAX,DWORD PTR SS:[EBP-8] //假注册码到 EAX
004A97F1  . E8 12A8F5FF      CALL SWFBrows.00404008 //检查注册码长度的合法性
004A97F6  . 33C0             XOR EAX,EAX
004A97F8  . 55               PUSH EBP
004A97F9  . 68 F5984A00      PUSH SWFBrows.004A98F5
004A97FE  . 64:FF30          PUSH DWORD PTR FS:[EAX]
004A9801  . 64:8920          MOV DWORD PTR FS:[EAX],ESP
004A9804  . 33C0             XOR EAX,EAX
004A9806  . 55               PUSH EBP
004A9807  . 68 C6984A00      PUSH SWFBrows.004A98C6
004A980C  . 64:FF30          PUSH DWORD PTR FS:[EAX]
004A980F  . 64:8920          MOV DWORD PTR FS:[EAX],ESP
004A9812  . 33C9             XOR ECX,ECX
004A9814  . B2 01            MOV DL,1
004A9816  . A1 20874A00      MOV EAX,DWORD PTR DS:[4A8720]
004A981B  . E8 4CFCFFFF      CALL SWFBrows.004A946C
004A9820  . 8BD8             MOV EBX,EAX
004A9822  . 33D2             XOR EDX,EDX
004A9824  . 8BC3             MOV EAX,EBX
004A9826  . E8 79F4FFFF      CALL SWFBrows.004A8CA4
004A982B  . 8D45 F4          LEA EAX,DWORD PTR SS:[EBP-C]
004A982E  . BA 10994A00      MOV EDX,SWFBrows.004A9910               ;  ASCII "1232hfbsdjdh2834121" //程序算法的密匙
004A9833  . E8 34A4F5FF      CALL SWFBrows.00403C6C
004A9838  . 8B55 F4          MOV EDX,DWORD PTR SS:[EBP-C]
004A983B  . 8BC3             MOV EAX,EBX
004A983D  . E8 B6F1FFFF      CALL SWFBrows.004A89F8
004A9842  . 8D4D F0          LEA ECX,DWORD PTR SS:[EBP-10]
004A9845  . 8B55 FC          MOV EDX,DWORD PTR SS:[EBP-4]
004A9848  . 8BC3             MOV EAX,EBX
004A984A  . E8 F5F2FFFF      CALL SWFBrows.004A8B44
004A984F  . BA 2C994A00      MOV EDX,SWFBrows.004A992C               ;  ASCII "ewrwk214134g7df2" //同上
004A9854  . 8BC3             MOV EAX,EBX
004A9856  . E8 9DF1FFFF      CALL SWFBrows.004A89F8
004A985B  . 8D4D EC          LEA ECX,DWORD PTR SS:[EBP-14]
004A985E  . 8B55 F0          MOV EDX,DWORD PTR SS:[EBP-10]
004A9861  . 8BC3             MOV EAX,EBX
004A9863  . E8 DCF2FFFF      CALL SWFBrows.004A8B44
004A9868  . C745 E8 EFFFFF> MOV DWORD PTR SS:[EBP-18],-11
004A986F  . 8B45 EC          MOV EAX,DWORD PTR SS:[EBP-14]
004A9872  . E8 DDA5F5FF      CALL SWFBrows.00403E54
004A9877  . 85C0             TEST EAX,EAX
004A9879  . 7E 1A            JLE SHORT SWFBrows.004A9895
004A987B  . 8B45 EC          MOV EAX,DWORD PTR SS:[EBP-14]
004A987E  . E8 D1A5F5FF      CALL SWFBrows.00403E54
004A9883  . 50               PUSH EAX
004A9884  . 8D45 EC          LEA EAX,DWORD PTR SS:[EBP-14]
004A9887  . E8 98A7F5FF      CALL SWFBrows.00404024
004A988C  . 8D4D E8          LEA ECX,DWORD PTR SS:[EBP-18]
004A988F  . 5A               POP EDX
004A9890  . E8 83FCFFFF      CALL SWFBrows.004A9518
004A9895  > 8B45 E8          MOV EAX,DWORD PTR SS:[EBP-18]
004A9898  . 33D2             XOR EDX,EDX
004A989A  . 52               PUSH EDX
004A989B  . 50               PUSH EAX
004A989C  . 8D55 E4          LEA EDX,DWORD PTR SS:[EBP-1C]
004A989F  . B8 20000000      MOV EAX,20
004A98A4  . E8 9FF8F5FF      CALL SWFBrows.00409148
004A98A9  . 8B55 E4          MOV EDX,DWORD PTR SS:[EBP-1C] //真注册码入 EDX 这里你可以看见正确的注册码
004A98AC  . 8B45 F8          MOV EAX,DWORD PTR SS:[EBP-8] //假注册码入 EAX 这里可以看见你输入的注册码
004A98AF  . E8 B0A6F5FF      CALL SWFBrows.00403F64 // 真假注册码比对  (这种比较方法算是比较常见的那种了)
004A98B4  . 75 04            JNZ SHORT SWFBrows.004A98BA
004A98B6  . B3 01            MOV BL,1
004A98B8  . EB 02            JMP SHORT SWFBrows.004A98BC
```

整理: 用户名 `CoolWolF[SCD]` 注册码 `DD5C4800`

如果想再来一次,就删除注册表的`[HKEY_CURRENT_USER\Software\Grooveware Multimedia\SWF Browser\Registration]`键

完

=================================================================



哪位老大能贴一下它的注册机?我对 `BlowFish` 算法实在头疼.

核心代码都是抄的 `BlowFish` 老大的，嘿嘿…………

```asm
;********************
;                  注册机 Asm 文件
;********************

    .386
    .model flat,stdcall
    option casemap:none

include       windows.inc
include       user32.inc
include       kernel32.inc
include       comctl32.inc
include       comdlg32.inc
include       masm32.inc

includelib    masm32.lib
includelib    user32.lib
includelib    kernel32.lib
includelib    comctl32.lib
includelib    comdlg32.lib

DLG_MAIN      equ   100
IDGEN         equ   10
Edit1         equ   11
Edit2         equ   12
BFLEFT        EQU   BFHIGH
BFRIGHT       EQU   BFLOW
IDABOUT       EQU   13
STATIC1       equ   14
STATIC2       equ   15
STATIC3       equ   16
IDI_ICO       equ   101

_ProcDlgMain    PROTO   :DWORD,:DWORD,:DWORD,:DWORD

BlowFish_En      PROTO   :DWORD,:DWORD
BlowFish_Fun    PROTO   :DWORD
BlowFish_Init   PROTO   :DWORD,:DWORD

        .data?

hInstance      dd      ?

        .data

PW_1    db "1232hfbsdjdh2834121"
PW_2    db "ewrwk214134g7df2"

STRABOUT1     DB      "任何问题，提议，请联系",0
STRABOUT2     DB      "firingme@sina.com",0

STRDEF1       DB      "仅以此献给我们可爱的 CCG",0
STRDEF2       DB      " 祝愿它蒸蒸日上！",0

data1         db      1000 dup (0)
data2         db      1000 dup (0)
data3         db      1000 dup (0)

szOK          DB      "OK",0
szABOUT       DB      "ABOUT",0
szID          DB      10 DUP(0)

NAMEERR       DB      "名字长度不能小于 4",0

MAGICLEN    DD 0

szName      db 100 dup (0)
key     dd 1058 dup (0)

Namelen3    dd 0
BFLOW       dd 0
BFHIGH      dd 0
MYBFLOW     DD 0
MYBFHIGH    DD 0
SN          DD 0
LASTSN      DB 10 DUP(0)
Namelen     dd 0
Namelen2    dd 0
Snlen       dd 0
Snlen2      dd 0
Snlen3      dd 0

TABLE1      DB 02BH,02DH,030H,031H,032H,033H,034H,035H,036H,037H,038H,039H,041H,042H,043H,044H
            DB 045H,046H,047H,048H,049H,04AH,04BH,04CH,04DH,04EH,04FH,050H,051H,052H,053H,054H
            DB 055H,056H,057H,058H,059H,05AH,061H,062H,063H,064H,065H,066H,067H,068H,069H,06AH
            DB 06BH,06CH,06DH,06EH,06FH,070H,071H,072H,073H,074H,075H,076H,077H,078H,079H,07AH
            DB 000H,000H,000H,000H,096H,030H,007H,077H,02CH,061H,00EH,0EEH,0BAH,051H,009H,099H
            DB 019H,0C4H,06DH,007H,08FH,0F4H,06AH,070H,035H,0A5H,063H,0E9H,0A3H,095H,064H,09EH
            DB 032H,088H,0DBH,00EH,0A4H,0B8H,0DCH,079H,01EH,0E9H,0D5H,0E0H,088H,0D9H,0D2H,097H
            DB 02BH,04CH,0B6H,009H,0BDH,07CH,0B1H,07EH,007H,02DH,0B8H,0E7H,091H,01DH,0BFH,090H
            DB 064H,010H,0B7H,01DH,0F2H,020H,0B0H,06AH,048H,071H,0B9H,0F3H,0DEH,041H,0BEH,084H
            DB 07DH,0D4H,0DAH,01AH,0EBH,0E4H,0DDH,06DH,051H,0B5H,0D4H,0F4H,0C7H,085H,0D3H,083H
            DB 056H,098H,06CH,013H,0C0H,0A8H,06BH,064H,07AH,0F9H,062H,0FDH,0ECH,0C9H,065H,08AH
            DB 04FH,05CH,001H,014H,0D9H,06CH,006H,063H,063H,03DH,00FH,0FAH,0F5H,00DH,008H,08DH
            DB 0C8H,020H,06EH,03BH,05EH,010H,069H,04CH,0E4H,041H,060H,0D5H,072H,071H,067H,0A2H
            DB 0D1H,0E4H,003H,03CH,047H,0D4H,004H,04BH,0FDH,085H,00DH,0D2H,06BH,0B5H,00AH,0A5H
            DB 0FAH,0A8H,0B5H,035H,06CH,098H,0B2H,042H,0D6H,0C9H,0BBH,0DBH,040H,0F9H,0BCH,0ACH
            DB 0E3H,06CH,0D8H,032H,075H,05CH,0DFH,045H,0CFH,00DH,0D6H,0DCH,059H,03DH,0D1H,0ABH
TABLE2      DD 000000000H,077073096H,0EE0E612CH,0990951BAH,0076DC419H,0706AF48FH,0E963A535H,09E6495A3H
            DD 00EDB8832H,079DCB8A4H,0E0D5E91EH,097D2D988H,009B64C2BH,07EB17CBDH,0E7B82D07H,090BF1D91H
            DD 01DB71064H,06AB020F2H,0F3B97148H,084BE41DEH,01ADAD47DH,06DDDE4EBH,0F4D4B551H,083D385C7H
            DD 0136C9856H,0646BA8C0H,0FD62F97AH,08A65C9ECH,014015C4FH,063066CD9H,0FA0F3D63H,08D080DF5H
            DD 03B6E20C8H,04C69105EH,0D56041E4H,0A2677172H,03C03E4D1H,04B04D447H,0D20D85FDH,0A50AB56BH
            DD 035B5A8FAH,042B2986CH,0DBBBC9D6H,0ACBCF940H,032D86CE3H,045DF5C75H,0DCD60DCFH,0ABD13D59H
            DD 026D930ACH,051DE003AH,0C8D75180H,0BFD06116H,021B4F4B5H,056B3C423H,0CFBA9599H,0B8BDA50FH
            DD 02802B89EH,05F058808H,0C60CD9B2H,0B10BE924H,02F6F7C87H,058684C11H,0C1611DABH,0B6662D3DH
            DD 076DC4190H,001DB7106H,098D220BCH,0EFD5102AH,071B18589H,006B6B51FH,09FBFE4A5H,0E8B8D433H
            DD 07807C9A2H,00F00F934H,09609A88EH,0E10E9818H,07F6A0DBBH,0086D3D2DH,091646C97H,0E6635C01H
            DD 06B6B51F4H,01C6C6162H,0856530D8H,0F262004EH,06C0695EDH,01B01A57BH,08208F4C1H,0F50FC457H
            DD 065B0D9C6H,012B7E950H,08BBEB8EAH,0FCB9887CH,062DD1DDFH,015DA2D49H,08CD37CF3H,0FBD44C65H
            DD 04DB26158H,03AB551CEH,0A3BC0074H,0D4BB30E2H,04ADFA541H,03DD895D7H,0A4D1C46DH,0D3D6F4FBH
            DD 04369E96AH,0346ED9FCH,0AD678846H,0DA60B8D0H,044042D73H,033031DE5H,0AA0A4C5FH,0DD0D7CC9H
            DD 05005713CH,0270241AAH,0BE0B1010H,0C90C2086H,05768B525H,0206F85B3H,0B966D409H,0CE61E49FH
            DD 05EDEF90EH,029D9C998H,0B0D09822H,0C7D7A8B4H,059B33D17H,02EB40D81H,0B7BD5C3BH,0C0BA6CADH
            DD 0EDB88320H,09ABFB3B6H,003B6E20CH,074B1D29AH,0EAD54739H,09DD277AFH,004DB2615H,073DC1683H
            DD 0E3630B12H,094643B84H,00D6D6A3EH,07A6A5AA8H,0E40ECF0BH,09309FF9DH,00A00AE27H,07D079EB1H
            DD 0F00F9344H,08708A3D2H,01E01F268H,06906C2FEH,0F762575DH,0806567CBH,0196C3671H,06E6B06E7H
            DD 0FED41B76H,089D32BE0H,010DA7A5AH,067DD4ACCH,0F9B9DF6FH,08EBEEFF9H,017B7BE43H,060B08ED5H
            DD 0D6D6A3E8H,0A1D1937EH,038D8C2C4H,04FDFF252H,0D1BB67F1H,0A6BC5767H,03FB506DDH,048B2364BH
            DD 0D80D2BDAH,0AF0A1B4CH,036034AF6H,041047A60H,0DF60EFC3H,0A867DF55H,0316E8EEFH,04669BE79H
            DD 0CB61B38CH,0BC66831AH,0256FD2A0H,05268E236H,0CC0C7795H,0BB0B4703H,0220216B9H,05505262FH
            DD 0C5BA3BBEH,0B2BD0B28H,02BB45A92H,05CB36A04H,0C2D7FFA7H,0B5D0CF31H,02CD99E8BH,05BDEAE1DH
            DD 09B64C2B0H,0EC63F226H,0756AA39CH,0026D930AH,09C0906A9H,0EB0E363FH,072076785H,005005713H
            DD 095BF4A82H,0E2B87A14H,07BB12BAEH,00CB61B38H,092D28E9BH,0E5D5BE0DH,07CDCEFB7H,00BDBDF21H
            DD 086D3D2D4H,0F1D4E242H,068DDB3F8H,01FDA836EH,081BE16CDH,0F6B9265BH,06FB077E1H,018B74777H
            DD 088085AE6H,0FF0F6A70H,066063BCAH,011010B5CH,08F659EFFH,0F862AE69H,0616BFFD3H,0166CCF45H
            DD 0A00AE278H,0D70DD2EEH,04E048354H,03903B3C2H,0A7672661H,0D06016F7H,04969474DH,03E6E77DBH
            DD 0AED16A4AH,0D9D65ADCH,040DF0B66H,037D83BF0H,0A9BCAE53H,0DEBB9EC5H,047B2CF7FH,030B5FFE9H
            DD 0BDBDF21CH,0CABAC28AH,053B39330H,024B4A3A6H,0BAD03605H,0CDD70693H,054DE5729H,023D967BFH
            DD 0B3667A2EH,0C4614AB8H,05D681B02H,02A6F2B94H,0B40BBE37H,0C30C8EA1H,05A05DF1BH,02D02EF8DH

pbox    dd 0243f6a88h, 085a308d3h, 013198a2eh, 003707344h, 0a4093822h, 0299f31d0h
    dd 0082efa98h, 0ec4e6c89h, 0452821e6h, 038d01377h, 0be5466cfh, 034e90c6ch
    dd 0c0ac29b7h, 0c97c50ddh, 03f84d5b5h, 0b5470917h, 09216d5d9h, 08979fb1bh

sbox1   dd 0d1310ba6h, 098dfb5ach, 02ffd72dbh, 0d01adfb7h, 0b8e1afedh, 06a267e96h
    dd 0ba7c9045h, 0f12c7f99h, 024a19947h, 0b3916cf7h, 00801f2e2h, 0858efc16h
    dd 0636920d8h, 071574e69h, 0a458fea3h, 0f4933d7eh, 00d95748fh, 0728eb658h
    dd 0718bcd58h, 082154aeeh, 07b54a41dh, 0c25a59b5h, 09c30d539h, 02af26013h
    dd 0c5d1b023h, 0286085f0h, 0ca417918h, 0b8db38efh, 08e79dcb0h, 0603a180eh
```



`dd 06c9e0e8bh, 0b01e8a3eh, 0d71577c1h, 0bd314b27h, 078af2fdah, 055605c60h`

`dd 0e65525f3h, 0aa55ab94h, 057489862h, 063e81440h, 055ca396ah, 02aab10b6h`

`dd 0b4cc5c34h, 01141e8ceh, 0a15486afh, 07c72e993h, 0b3ee1411h, 0636fbc2ah`

`dd 02ba9c55dh, 0741831f6h, 0ce5c3e16h, 09b87931eh, 0afd6ba33h, 06c24cf5ch`

`dd 07a325381h, 028958677h, 03b8f4898h, 06b4bb9afh, 0c4bfe81bh, 066282193h`

`dd 061d809cch, 0fb21a991h, 0487cac60h, 05dec8032h, 0ef845d5dh, 0e98575b1h`

`dd 0dc262302h, 0eb651b88h, 023893e81h, 0d396acc5h, 00f6d6ff3h, 083f44239h`

`dd 02e0b4482h, 0a4842004h, 069c8f04ah, 09e1f9b5eh, 021c66842h, 0f6e96c9ah`

`dd 0670c9c61h, 0abd388f0h, 06a51a0d2h, 0d8542f68h, 0960fa728h, 0ab5133a3h`

`dd 06eef0b6ch, 0137a3be4h, 0ba3bf050h, 07efb2a98h, 0a1f1651dh, 039af0176h`

`dd 066ca593eh, 082430e88h, 08cee8619h, 0456f9fb4h, 07d84a5c3h, 03b8b5ebeh`

`dd 0e06f75d8h, 085c12073h, 0401a449fh, 056c16aa6h, 04ed3aa62h, 0363f7706h`

`dd 01bfedf72h, 0429b023dh, 037d0d724h, 0d00a1248h, 0db0fead3h, 049f1c09bh`

`dd 0075372c9h, 080991b7bh, 025d479d8h, 0f6e8def7h, 0e3fe501ah, 0b6794c3bh`

`dd 0976ce0bdh, 004c006bah, 0c1a94fb6h, 0409f60c4h, 05e5c9ec2h, 0196a2463h`

`dd 068fb6fafh, 03e6c53b5h, 01339b2ebh, 03b52ec6fh, 06dfc511fh, 09b30952ch`

`dd 0cc814544h, 0af5ebd09h, 0bee3d004h, 0de334afdh, 0660f2807h, 0192e4bb3h`

`dd 0c0cba857h, 045c8740fh, 0d20b5f39h, 0b9d3fbdbh, 05579c0bdh, 01a60320ah`

`dd 0d6a100c6h, 0402c7279h, 0679f25feh, 0fb1fa3cch, 08ea5e9f8h, 0db3222f8h`

`dd 03c7516dfh, 0fd616b15h, 02f501ec8h, 0ad0552abh, 0323db5fah, 0fd238760h`

`dd 053317b48h, 03e00df82h, 09e5c57bbh, 0ca6f8ca0h, 01a87562eh, 0df1769dbh`

`dd 0d542a8f6h, 0287effc3h, 0ac6732c6h, 08c4f5573h, 0695b27b0h, 0bbca58c8h`

`dd 0e1ffa35dh, 0b8f011a0h, 010fa3d98h, 0fd2183b8h, 04afcb56ch, 02dd1d35bh`

`dd 09a53e479h, 0b6f84565h, 0d28e49bch, 04bfb9790h, 0e1ddf2dah, 0a4cb7e33h`

`dd 062fb1341h, 0cee4c6e8h, 0ef20cadah, 036774c01h, 0d07e9efeh, 02bf11fb4h`

`dd 095dbda4dh, 0ae909198h, 0eaad8e71h, 06b93d5a0h, 0d08ed1d0h, 0afc725e0h`

`dd 08e3c5b2fh, 08e7594b7h, 08ff6e2fbh, 0f2122b64h, 08888b812h, 0900df01ch`

`dd 04fad5ea0h, 0688fc31ch, 0d1cff191h, 0b3a8c1adh, 02f2f2218h, 0be0e1777h`

`dd 0ea752dfeh, 08b021fa1h, 0e5a0cc0fh, 0b56f74e8h, 018acf3d6h, 0ce89e299h`

`dd 0b4a84fe0h, 0fd13e0b7h, 07cc43b81h, 0d2ada8d9h, 0165fa266h, 080957705h`

`dd 093cc7314h, 0211a1477h, 0e6ad2065h, 077b5fa86h, 0c75442f5h, 0fb9d35cfh`

`dd 0ebcdaf0ch, 07b3e89a0h, 0d6411bd3h, 0ae1e7e49h, 000250e2dh, 02071b35eh`

`dd 0226800bbh, 057b8e0afh, 02464369bh, 0f009b91eh, 05563911dh, 059dfa6aah`

`dd 078c14389h, 0d95a537fh, 0207d5ba2h, 002e5b9c5h, 083260376h, 06295cfa9h`

`dd 011c81968h, 04e734a41h, 0b3472dcah, 07b14a94ah, 01b510052h, 09a532915h`

`dd 0d60f573fh, 0bc9bc6e4h, 02b60a476h, 081e67400h, 008ba6fb5h, 0571be91fh`

`dd 0f296ec6bh, 02a0dd915h, 0b6636521h, 0e7b9f9b6h, 0ff34052eh, 0c5855664h`

`dd 053b02d5dh, 0a99f8fa1h, 008ba4799h, 06e85076ah`

### sbox2

`dd 04b7a70e9h, 0b5b32944h`

`dd 0db75092eh, 0c4192623h, 0ad6ea6b0h, 049a7df7dh, 09cee60b8h, 08fedb266h`

`dd 0ecaa8c71h, 0699a17ffh, 05664526ch, 0c2b19ee1h, 0193602a5h, 075094c29h`

`dd 0a0591340h, 0e4183a3eh, 03f54989ah, 05b429d65h, 06b8fe4d6h, 099f73fd6h`

`dd 0a1d29c07h, 0efe830f5h, 04d2d38e6h, 0f0255dc1h, 04cdd2086h, 08470eb26h`

`dd 06382e9c6h, 0021ecc5eh, 009686b3fh, 03ebaefc9h, 03c971814h, 06b6a70a1h`

`dd 0687f3584h, 052a0e286h, 0b79c5305h, 0aa500737h, 03e07841ch, 07fdeae5ch`

`dd 08e7d44ech, 05716f2b8h, 0b03ada37h, 0f0500c0dh, 0f01c1f04h, 00200b3ffh`

`dd 0ae0cf51ah, 03cb574b2h, 025837a58h, 0dc0921bdh, 0d19113f9h, 07ca92ff6h`

`dd 094324773h, 022f54701h, 03ae5e581h, 037c2dadch, 0c8b57634h, 09af3dda7h`

`dd 0a9446146h, 00fd0030eh, 0ecc8c73eh, 0a4751e41h, 0e238cd99h, 03bea0e2fh`

`dd 03280bba1h, 0183eb331h, 04e548b38h, 04f6db908h, 06f420d03h, 0f60a04bfh`

`dd 02cb81290h, 024977c79h, 05679b072h, 0bcaf89afh, 0de9a771fh, 0d9930810h`

`dd 0b38bae12h, 0dccf3f2eh, 05512721fh, 02e6b7124h, 0501adde6h, 09f84cd87h`

`dd 07a584718h, 07408da17h, 0bc9f9abch, 0e94b7d8ch, 0ec7aec3ah, 0db851dfah`

`dd 063094366h, 0c464c3d2h, 0ef1c1847h, 03215d908h, 0dd433b37h, 024c2ba16h`

`dd 012a14d43h, 02a65c451h, 050940002h, 0133ae4ddh, 071dff89eh, 010314e55h`

`dd 081ac77d6h, 05f11199bh, 0043556f1h, 0d7a3c76bh, 03c11183bh, 05924a509h`

`dd 0f28fe6edh, 097f1fbfah, 09ebabf2ch, 01e153c6eh, 086e34570h, 0eae96fb1h`

`dd 0860e5e0ah, 05a3e2ab3h, 0771fe71ch, 04e3d06fah, 02965dcb9h, 099e71d0fh`

`dd 0803e89d6h, 05266c825h, 02e4cc978h, 09c10b36ah, 0c6150ebah, 094e2ea78h`

`dd 0a5fc3c53h, 01e0a2df4h, 0f2f74ea7h, 0361d2b3dh, 01939260fh, 019c27960h`

`dd 05223a708h, 0f71312b6h, 0ebadfe6eh, 0eac31f66h, 0e3bc4595h, 0a67bc883h`

`dd 0b17f37d1h, 0018cff28h, 0c332ddefh, 0be6c5aa5h, 065582185h, 068ab9802h`

`dd 0eecea50fh, 0db2f953bh, 02aef7dadh, 05b6e2f84h, 01521b628h, 029076170h`

`dd 0ecdd4775h, 0619f1510h, 013cca830h, 0eb61bd96h, 00334fe1eh, 0aa0363cfh`

`dd 0b5735c90h, 04c70a239h, 0d59e9e0bh, 0cbaade14h, 0eecc86bch, 060622ca7h`

`dd 09cab5cabh, 0b2f3846eh, 0648b1eafh, 019bdf0cah, 0a02369b9h, 0655abb50h`

`dd 040685a32h, 03c2ab4b3h, 0319ee9d5h, 0c021b8f7h, 09b540b19h, 0875fa099h`

`dd 095f7997eh, 0623d7da8h, 0f837889ah, 097e32d77h, 011ed935fh, 016681281h`

`dd 00e358829h, 0c7e61fd6h, 096dedfa1h, 07858ba99h, 057f584a5h, 01b227263h`

`dd 09b83c3ffh, 01ac24696h, 0cdb30aebh, 0532e3054h, 08fd948e4h, 06dbc3128h`

`dd 058ebf2efh, 034c6ffeah, 0fe28ed61h, 0ee7c3c73h, 05d4a14d9h, 0e864b7e3h`

`dd 042105d14h, 0203e13e0h, 045eee2b6h, 0a3aaabeah, 0db6c4f15h, 0facb4fd0h`

`dd 0c742f442h, 0ef6abbb5h, 0654f3b1dh, 041cd2105h, 0d81e799eh, 086854dc7h`

`dd 0e44b476ah, 03d816250h, 0cf62a1f2h, 05b8d2646h, 0fc8883a0h, 0c1c7b6a3h`

`dd 07f1524c3h, 069cb7492h, 047848a0bh, 05692b285h, 0095bbf00h, 0ad19489dh`

`dd 01462b174h, 023820e00h, 058428d2ah, 00c55f5eah, 01dadf43eh, 0233f7061h`

`dd 03372f092h, 08d937e41h, 0d65fecf1h, 06c223bdbh, 07cde3759h, 0cbee7460h`

`dd 04085f2a7h, 0ce77326eh, 0a6078084h, 019f8509eh, 0e8efd855h, 061d99735h`

`dd 0a969a7aah, 0c50c06c2h, 05a04abfch, 0800bcadch, 09e447a2eh, 0c3453484h`

`dd 0fdd56705h, 00e1e9ec9h, 0db73dbd3h, 0105588cdh, 0675fda79h, 0e3674340h`

`dd 0c5c43465h, 0713e38d8h, 03d28f89eh, 0f16dff20h, 0153e21e7h, 08fb03d4ah`

`dd 0e6e39f2bh, 0db83adf7h`

> ### sbox3
>
> `dd 0e93d5a68h, 0948140f7h, 0f64c261ch, 094692934h`
>
> `dd 0411520f7h, 07602d4f7h, 0bcf46b2eh, 0d4a20068h, 0d4082471h, 03320f46ah`
>
> `dd 043b7d4b7h, 0500061afh, 01e39f62eh, 097244546h, 014214f74h, 0bf8b8840h`
>
> `dd 04d95fc1dh, 096b591afh, 070f4ddd3h, 066a02f45h, 0bfbc09ech, 003bd9785h`
>
> `dd 07fac6dd0h, 031cb8504h, 096eb27b3h, 055fd3941h, 0da2547e6h, 0abca0a9ah`
>
> `dd 028507825h, 0530429f4h, 00a2c86dah, 0e9b66dfbh, 068dc1462h, 0d7486900h`
>
> `dd 0680ec0a4h, 027a18deeh, 04f3ffea2h, 0e887ad8ch, 0b58ce006h, 07af4d6b6h`
>
> `dd 0aace1e7ch, 0d3375fech, 0ce78a399h, 0406b2a42h, 020fe9e35h, 0d9f385b9h`
>
> `dd 0ee39d7abh, 03b124e8bh, 01dc9faf7h, 04b6d1856h, 026a36631h, 0eae397b2h`
>
> `dd 03a6efa74h, 0dd5b4332h, 06841e7f7h, 0ca7820fbh, 0fb0af54eh, 0d8feb397h`
>
> `dd 0454056ach, 0ba489527h, 055533a3ah, 020838d87h, 0fe6ba9b7h, 0d096954bh`
>
> `dd 055a867bch, 0a1159a58h, 0cca92963h, 099e1db33h, 0a62a4a56h, 03f3125f9h`
>
> `dd 05ef47e1ch, 09029317ch, 0fdf8e802h, 004272f70h, 080bb155ch, 005282ce3h`
>
> `dd 095c11548h, 0e4c66d22h, 048c1133fh, 0c70f86dch, 007f9c9eeh, 041041f0fh`
>
> `dd 0404779a4h, 05d886e17h, 0325f51ebh, 0d59bc0d1h, 0f2bcc18fh, 041113564h`
>
> `dd 0257b7834h, 0602a9c60h, 0dff8e8a3h, 01f636c1bh, 00e12b4c2h, 002e1329eh`



> `dd 0af664fd1h, 0cad18115h, 06b2395e0h, 0333e92e1h, 03b240b62h, 0eebeb922h`
>
> `dd 085b2a20eh, 0e6ba0d99h, 0de720c8ch, 02da2f728h, 0d0127845h, 095b794fdh`
>
> `dd 0647d0862h, 0e7ccf5f0h, 05449a36fh, 0877d48fah, 0c39dfd27h, 0f33e8d1eh`
>
> `dd 00a476341h, 0992eff74h, 03a6f6eabh, 0f4f8fd37h, 0a812dc60h, 0a1ebddf8h`
>
> `dd 0991be14ch, 0db6e6b0dh, 0c67b5510h, 06d672c37h, 02765d43bh, 0dcd0e804h`
>
> `dd 0f1290dc7h, 0cc00ffa3h, 0b5390f92h, 0690fed0bh, 0667b9ffbh, 0cedb7d9ch`
>
> `dd 0a091cf0bh, 0d9155ea3h, 0bb132f88h, 0515bad24h, 07b9479bfh, 0763bd6ebh`
>
> `dd 037392eb3h, 0cc115979h, 08026e297h, 0f42e312dh, 06842ada7h, 0c66a2b3bh`
>
> `dd 012754ccch, 0782ef11ch, 06a124237h, 0b79251e7h, 006a1bbe6h, 04bfb6350h`
>
> `dd 01a6b1018h, 011caedfah, 03d25bdd8h, 0e2e1c3c9h, 044421659h, 00a121386h`
>
> `dd 0d90cec6eh, 0d5abea2ah, 064af674eh, 0da86a85fh, 0bebfe988h, 064e4c3feh`
>
> `dd 09dbc8057h, 0f0f7c086h, 060787bf8h, 06003604dh, 0d1fd8346h, 0f6381fb0h`
>
> `dd 07745ae04h, 0d736fccch, 083426b33h, 0f01eab71h, 0b0804187h, 03c005e5fh`
>
> `dd 077a057beh, 0bde8ae24h, 055464299h, 0bf582e61h, 04e58f48fh, 0f2ddfda2h`
>
> `dd 0f474ef38h, 08789bdc2h, 05366f9c3h, 0c8b38e74h, 0b475f255h, 046fcd9b9h`
>
> `dd 07aeb2661h, 08b1ddf84h, 0846a0e79h, 0915f95e2h, 0466e598eh, 020b45770h`
>
> `dd 08cd55591h, 0c902de4ch, 0b90bace1h, 0bb8205d0h, 011a86248h, 07574a99eh`
>
> `dd 0b77f19b6h, 0e0a9dc09h, 0662d09a1h, 0c4324633h, 0e85a1f02h, 009f0be8ch`
>
> `dd 04a99a025h, 01d6efe10h, 01ab93d1dh, 00ba5a4dfh, 0a186f20fh, 02868f169h`
>
> `dd 0dcb7da83h, 0573906feh, 0a1e2ce9bh, 04fcd7f52h, 050115e01h, 0a70683fah`
>
> `dd 0a002b5c4h, 00de6d027h, 09af88c27h, 0773f8641h, 0c3604c06h, 061a806b5h`
>
> `dd 0f0177a28h, 0c0f586e0h, 0006058aah, 030dc7d62h, 011e69ed7h, 02338ea63h`
>
> `dd 053c2dd94h, 0c2c21634h, 0bbcbee56h, 090bcb6deh, 0ebfc7da1h, 0ce591d76h`
>
> `dd 06f05e409h, 04b7c0188h, 039720a3dh, 07c927c24h, 086e3725fh, 0724d9db9h`
>
> `dd 01ac15bb4h, 0d39eb8fch, 0ed545578h, 008fca5b5h, 0d83d7cd3h, 04dad0fc4h`
>
> `dd 01e50ef5eh, 0b161e6f8h, 0a28514d9h, 06c51133ch, 06fd5c7e7h, 056e14ec4h`
>
> `dd 0362abfceh, 0ddc6c837h, 0d79a3234h, 092638212h, 0670efa8eh, 0406000e0h`

> `sbox4`
>
> `dd 03a39ce37h, 0d3faf5cfh, 0abc27737h, 05ac52d1bh, 05cb0679eh, 04fa33742h`
>
> `dd 0d3822740h, 099bc9bbeh, 0d5118e9dh, 0bf0f7315h, 0d62d1c7eh, 0c700c47bh`
>
> `dd 0b78c1b6bh, 021a19045h, 0b26eb1beh, 06a366eb4h, 05748ab2fh, 0bc946e79h`
>
> `dd 0c6a376d2h, 06549c2c8h, 0530ff8eeh, 0468dde7dh, 0d5730a1dh, 04cd04dc6h`
>
> `dd 02939bbdbh, 0a9ba4650h, 0ac9526e8h, 0be5ee304h, 0a1fad5f0h, 06a2d519ah`
>
> `dd 063ef8ce2h, 09a86ee22h, 0c089c2b8h, 043242ef6h, 0a51e03aah, 09cf2d0a4h`
>
> `dd 083c061bah, 09be96a4dh, 08fe51550h, 0ba645bd6h, 02826a2f9h, 0a73a3ae1h`
>
> `dd 04ba99586h, 0ef5562e9h, 0c72fefd3h, 0f752f7dah, 03f046f69h, 077fa0a59h`
>
> `dd 080e4a915h, 087b08601h, 09b09e6adh, 03b3ee593h, 0e990fd5ah, 09e34d797h`
>
> `dd 02cf0b7d9h, 0022b8b51h, 096d5ac3ah, 0017da67dh, 0d1cf3ed6h, 07c7d2d28h`
>
> `dd 01f9f25cfh, 0adf2b89bh, 05ad6b472h, 05a88f54ch, 0e029ac71h, 0e019a5e6h`
>
> `dd 047b0acfdh, 0ed93fa9bh, 0e8d3c48dh, 0283b57cch, 0f8d56629h, 079132e28h`
>
> `dd 0785f0191h, 0ed756055h, 0f7960e44h, 0e3d35e8ch, 015056dd4h, 088f46dbah`
>
> `dd 003a16125h, 00564f0bdh, 0c3eb9e15h, 03c9057a2h, 097271aech, 0a93a072ah`
>
> `dd 01b3f6d9bh, 01e6321f5h, 0f59c66fbh, 026dcf319h, 07533d928h, 0b155fdf5h`
>
> `dd 003563482h, 08aba3cbbh, 028517711h, 0c20ad9f8h, 0abcc5167h, 0ccad925fh`
>
> `dd 04de81751h, 03830dc8eh, 0379d5862h, 09320f991h, 0ea7a90c2h, 0fb3e7bceh`
>
> `dd 05121ce64h, 0774fbe32h, 0a8b6e37eh, 0c3293d46h, 048de5369h, 06413e680h`
>
> `dd 0a2ae0810h, 0dd6db224h, 069852dfdh, 009072166h, 0b39a460ah, 06445c0ddh`
>
> `dd 0586cdecfh, 01c20c8aeh, 05bbef7ddh, 01b588d40h, 0ccd2017fh, 06bb4e3bbh`
>
> `dd 0dda26a7eh, 03a59ff45h, 03e350a44h, 0bcb4cdd5h, 072eacea8h, 0fa6484bbh`
>
> `dd 08d6612aeh, 0bf3c6f47h, 0d29be463h, 0542f5d9eh, 0aec2771bh, 0f64e6370h`
>
> `dd 0740e0d8dh, 0e75b1357h, 0f8721671h, 0af537d5dh, 04040cb08h, 04eb4e2cch`
>
> `dd 034d2466ah, 00115af84h, 0e1b00428h, 095983a1dh, 006b89fb4h, 0ce6ea048h`
>
> `dd 06f3f3b82h, 03520ab82h, 0011a1d4bh, 0277227f8h, 0611560b1h, 0e7933fdch`
>
> `dd 0bb3a792bh, 0344525bdh, 0a08839e1h, 051ce794bh, 02f32c9b7h, 0a01fbac9h`
>
> `dd 0e01cc87eh, 0bcc7d1f6h, 0cf0111c3h, 0a1e8aac7h, 01a908749h, 0d44fbd9ah`
>
> `dd 0d0dadecbh, 0d50ada38h, 00339c32ah, 0c6913667h, 08df9317ch, 0e0b12b4fh`
>
> `dd 0f79e59b7h, 043f5bb3ah, 0f2d519ffh, 027d9459ch, 0bf97222ch, 015e6fc2ah`
>
> `dd 00f91fc71h, 09b941525h, 0fae59361h, 0ceb69cebh, 0c2a86459h, 012baa8d1h`
>
> `dd 0b6c1075eh, 0e3056a0ch, 010d25065h, 0cb03a442h, 0e0ec6e0eh, 01698db3bh`
>
> `dd 04c98a0beh, 03278e964h, 09f1f9532h, 0e0d392dfh, 0d3a0342bh, 08971f21eh`
>
> `dd 01b0a7441h, 04ba3348ch, 0c5be7120h, 0c37632d8h, 0df359f8dh, 09b992f2eh`
>
> `dd 0e60b6f47h, 00fe3f11dh, 0e54cda54h, 01edad891h, 0ce6279cfh, 0cd3e7e6fh`
>
> `dd 01618b166h, 0fd2c1d05h, 0848fd2c5h, 0f6fb2299h, 0f523f357h, 0a6327623h`
>
> `dd 093a83531h, 056cccd02h, 0acf08162h, 05a75ebb5h, 06e163697h, 088d273cch`
>
> `dd 0de966292h, 081b949d0h, 04c50901bh, 071c65614h, 0e6c6c7bdh, 0327a140ah`
>
> `dd 045e1d006h, 0c3f27b9ah, 0c9aa53fdh, 062a80f00h, 0bb25bfe2h, 035bdd2f6h`
>
> `dd 071126905h, 0b2040222h, 0b6cbcf7ch, 0cd769c2bh, 053113ec0h, 01640e3d3h`
>
> `dd 038abbd60h, 02547adf0h, 0ba38209ch, 0f746ce76h, 077afa1c5h, 020756060h`
>
> `dd 085cbfe4eh, 08ae88dd8h, 07aaaf9b0h, 04cf9aa7eh, 01948c25ch, 002fb8a8ch`
>
> `dd 001c36ae4h, 0d6ebe1f9h, 090d4f869h, 0a65cdea0h, 03f09252dh, 0c208e69fh`
>
> `dd 0b74e6132h, 0ce77e25bh, 0578fdfe3h, 03ac372e6h`

```
.code
;s 盒变换函数
BlowFish_Fun proc uses ebx edi esi edx ecx,BfNum:DWORD
    MOV      ECX,BfNum
    MOV      AL,CL
    AND      EAX,0FFh
    SHR      ECX,08
    MOV      EDX,EAX
    MOV      AL,CL
    MOV      EDI,offset key
    AND      EAX,0FFh
    SHR      ECX,08
    MOV      ESI,EAX
    MOV      EAX,ECX
    SHR      EAX,08
    AND      EAX,0FFh
    AND      ECX,0FFh
    AND      ESI,0FFFFh
    AND      EDX,0FFFFh
    MOV      EAX,[EDI+EAX*4+48h]
    MOV      EBX,[EDI+ECX*4+0448h]
    MOV      ECX,[EDI+ESI*4+0848h]
    ADD      EAX,EBX
    XOR      EAX,ECX
    MOV      ECX,[EDI+EDX*4+0C48h]
    ADD      EAX,ECX
    RET
BlowFish_Fun endp

;BlowFish 加密算法函数
BlowFish_En  proc uses ebx edi esi edx ecx,highbf:DWORD,lowbf:DWORD
    LOCAL    num :DWORD
    MOV      EAX,highbf
    MOV      ECX,lowbf
    MOV      EAX,[EAX]
    MOV      ESI,[ECX]
    MOV      EDI,offset key
    MOV      num,10h
    MOV      EBX,EDI
loc_40108E:
    XOR      EAX,[EBX]
    MOV      EDX,EAX
    invoke   BlowFish_Fun,EAX
    MOV      ECX,num
    XOR      EAX,ESI
    ADD      EBX,4
    DEC      ECX
    MOV      ESI,EDX
    MOV      num,ECX
    JNZ      loc_40108E

    MOV      ECX,[EDI+40h]
    MOV      EDX,[EDI+44h]
    XOR      ECX,EAX
    XOR      EDX,ESI

    MOV      [BFHIGH],EDX
    MOV      [BFLOW],ECX
    RET
BlowFish_En  endp

;BlowFish 初始化函数
BlowFish_Init proc uses ebx edi esi edx ecx,PWD:DWORD,len_PWD:DWORD
    LOCAL    pbox_num18:DWORD
    LOCAL    pbox_num4 :DWORD
    LOCAL    snum      :DWORD
;初始化 s 盒
    MOV      ESI,offset key
```



# Blowfish 初始化与相关汇编函数

## BlowFish_Init 流程

### S 盒初始化
```asm
    MOV     EAX, offset sbox1

    LEA     ECX, [ESI+48h]

loc_401141:

    MOV     EDX, 0100h

loc_401146:

    MOV     EDI, [EAX]

    ADD     EAX, 4

    MOV     [ECX], EDI

    ADD     ECX, 4

    DEC     EDX

    JNZ     loc_401146

    CMP     EAX, offset sbox1+1000h

    JL      loc_401141
```

### P 盒初始化
初始化 p 盒

#### 第一步：原 p 盒与 PWD 逐项异或
```asm
    MOV     EDX, PWD

    MOV     EDI, offset pbox

    XOR     EAX, EAX

    SUB     EDI, ESI

    MOV     pbox_num18, 12h

loc_401173:

    XOR     ECX, ECX

    MOV     pbox_num4, 04

loc_40117D:

    XOR     EBX, EBX

    MOV     BL, [EAX+EDX]

    SHL     ECX, 08

    OR      ECX, EBX

    INC     EAX

    CMP     EAX, len_PWD

    JL      loc_40118E

    XOR     EAX, EAX

loc_40118E:

    MOV     EBX, pbox_num4

    DEC     EBX

    MOV     pbox_num4, EBX

    JNZ     loc_40117D

    MOV     EBX, [EDI+ESI]

    ADD     ESI, 4

    XOR     EBX, ECX

    MOV     ECX, pbox_num18

    MOV     [ESI-04], EBX

    DEC     ECX

    MOV     pbox_num18, ecx

    JNZ     loc_401173
```

#### 用连续的 Blowfish 算法填充 P 盒
```asm
    MOV     EBX, offset key

    XOR     EAX, EAX

    MOV     BFLOW, EAX

    MOV     BFHIGH, EAX

    MOV     ESI, EBX

    MOV     EDI, 09

loc_4011C4:

    LEA     EAX, BFLOW

    LEA     ECX, BFHIGH

    invoke  BlowFish_En, ECX, EAX

    MOV     EAX, BFHIGH

    MOV     ECX, BFLOW

    MOV     [ESI], EAX

    MOV     [ESI+04], ECX

    ADD     ESI, 8

    DEC     EDI

    JNZ     loc_4011C4
```

#### 用连续的 Blowfish 算法填充 S 盒
```asm
    LEA     ESI, [EBX+4Ch]

    MOV     snum, 04      ;4 个 s 盒。

loc_4011F2:

    MOV     EDI, 80H     ;每个盒填充 80h=128 次（每次填充两个数）。

loc_4011F7:

    LEA     ECX, BFLOW

    LEA     EDX, BFHIGH

    invoke  BlowFish_En, EDX, ECX

    MOV     ECX, BFHIGH

    MOV     EDX, BFLOW

    MOV     [ESI-04], ECX

    MOV     [ESI], EDX

    ADD     ESI, 8

    DEC     EDI

    JNZ     loc_4011F7

    DEC     snum

    JNZ     loc_4011F2

    RET
```

`BlowFish_Init endp`

---

## 辅助字符串函数

### MyStrlen
```asm
MyStrlen proc uses ecx edi, szString:DWORD
    MOV     EDI, szString

    XOR     ECX, ECX

    NOT     ECX

    XOR     EAX, EAX

    REPNZ   SCASB

    NOT     ECX

    DEC     ECX

    MOV     EAX, ECX

    RET
MyStrlen endp
```

### MyRev
```asm
MyRev proc uses ebx ecx esi edi, SRString:DWORD, DEString:DWORD, StringLen:DWORD
    LOCAL   Len1:DWORD
    LOCAL   Len2:DWORD

    MOV     EAX, StringLen

    MOV     EBX, 4

    CDQ

    DIV     EBX

    MOV     ECX, EAX

    MOV     ESI, SRString

    MOV     EDI, DEString

@@2:
    MOV     EBX, 4

@@1:
    MOV     AL, [ESI+EBX-1]

    MOV     EDX, 4

    SUB     EDX, EBX

    MOV     [EDI+EDX], AL

    DEC     EBX

    JNZ     @@1

    ADD     ESI, 4

    ADD     EDI, 4

    LOOP    @@2

    MOV     BYTE PTR [EDI], 0

    RET
MyRev endp
```

### MyStrmagic
```asm
MyStrmagic proc uses ebx ecx edx esi edi, SString:DWORD, DString:DWORD, StringLen:DWORD
    LOCAL   CHAR1:BYTE
    LOCAL   CHAR2:BYTE
    LOCAL   MYFLAG:BYTE
    LOCAL   LEN:DWORD

    MOV     MYFLAG, 0

    MOV     EAX, StringLen

    MOV     LEN, EAX

    MOV     EBX, 3

    CDQ

    DIV     EBX

    .IF     EDX != 0
        INC LEN
        MOV MYFLAG, 1
    .ENDIF

    MOV     EDI, DString

    MOV     ESI, 0

    MOV     EBX, 1

loc_4A8898:
    CMP     LEN, EBX

    JL      loc_4A88A8

    MOV     EAX, SString

    MOV     AL, [EAX+EBX-01]

    MOV     CHAR1, AL

    JMP     SHORT loc_4A88AC

loc_4A88A8:
    PUSH    EAX

    XOR     EAX, EAX

    MOV     CHAR1, AL

    POP     EAX

loc_4A88AC:
    MOV     EAX, ESI

    SUB     EAX, 1

    JC      loc_4A88BD    ;ESI=0,执行 4A88BD

    JZ      loc_4A88E9    ;ESI=1,执行 4A88E9

    DEC     EAX

    JZ      loc_4A892A    ;ESI=2,执行 4A892A

    JMP     loc_4A8996

loc_4A88BD:
    XOR     EDX, EDX

    MOV     DL, CHAR1

    SHR     EDX, 02

    MOV     DL, [EDX+TABLE1]

    MOV     [EDI], DL

    INC     EDI

    MOV     AL, CHAR1

    MOV     CHAR2, AL

    JMP     loc_4A8996

loc_4A88E9:
    MOV     DL, CHAR2

    AND     DL, 03

    AND     EDX, 0FFH

    SHL     EDX, 04

    MOV     CL, CHAR1

    AND     CL, 0F0H

    AND     ECX, 0FFH

    SHR     ECX, 04

    OR      EDX, ECX

    MOV     DL, [EDX+TABLE1]

    MOV     [EDI], DL

    INC     EDI

    MOV     AL, CHAR1

    MOV     CHAR2, AL

    JMP     SHORT loc_4A8996

loc_4A892A:
    MOV     DL, CHAR2

    AND     DL, 0FH

    AND     EDX, 0FFH

    SHL     EDX, 02

    MOV     CL, CHAR1

    AND     CL, 0C0H

    AND     ECX, 0FFH

    SHR     ECX, 06

    OR      EDX, ECX

    MOV     DL, [EDX+TABLE1]

    MOV     [EDI], DL

    INC     EDI

    CMP     MYFLAG, 0

    JZ      @@33

    CMP     LEN, EBX

    JZ      loc_4A8992

@@33:
    MOV     DL, CHAR1

    AND     DL, 03FH

    AND     EDX, 0FFH

    MOV     DL, [EDX+TABLE1]

    MOV     [EDI], DL

    INC     EDI

loc_4A8992:
    MOV     CHAR2, 00

loc_4A8996:
    INC     ESI

    CMP     ESI, 3

    JNZ     loc_4A899E

    XOR     ESI, ESI

loc_4A899E:
    INC     EBX

    CMP     LEN, EBX

    JGE     loc_4A8898

    XOR     EAX, EAX

    MOV     [EDI], EAX

    RET
MyStrmagic endp
```

---

## 消息处理函数

### _ProcDlgMain
```asm
_ProcDlgMain proc uses ebx edi esi edx ecx, hWnd:DWORD, wMsg:DWORD, wParam:DWORD, lParam:DWORD
    mov     eax, wMsg

    .if     eax == WM_CLOSE
        invoke EndDialog, hWnd, NULL

    .ELSEIF eax == WM_INITDIALOG
        invoke LoadIcon, hInstance, IDI_ICO
        invoke SendMessage, hWnd, WM_SETICON, ICON_BIG, EAX
        MOV  EAX, FALSE
        RET

    .elseif eax == WM_COMMAND
        mov  eax, wParam
        and  eax, 0ffffh

        .if  eax == IDGEN
            invoke GetDlgItem, hWnd, Edit2
            invoke SetFocus, EAX
            invoke GetDlgItemText, hWnd, Edit1, offset szName, 30
            CMP  EAX, 4
            JC   @ERR
            MOV  Namelen, EAX

            CDQ
            MOV  ECX, 8
            DIV  ECX
            INC  EAX
            MOV  Namelen2, EAX
            XOR  ECX, ECX
            MOV  BL, 8
            SUB  BL, DL
            MOV  CL, BL
            IMUL EAX, 8
            MOV  Namelen3, EAX
            MOV  EAX, Namelen
            DEC  EAX

@@11:
            MOV  [szName+EAX+ECX], BL
            LOOP @@11          ;名字后面添加数字
```



```
invoke MyRev, offset szName, offset data1, Namelen3; 名字反序排列
MOV EAX, Namelen3
mov ebx, 8
idiv ebx
and eax, 0ffh
mov ecx, eax
mov esi, offset data1
MOV EDI, OFFSET data2
MOV MAGICLEN, 0
@@12:
mov ebx, esi
add ebx, 4
invoke BlowFish_Init, offset PW_1, 19
invoke BlowFish_En, esi, ebx
MOV EAX, BFLEFT
MOV EBX, BFRIGHT
MOV [EDI], EAX
MOV [EDI+4], EBX
ADD MAGICLEN, 8
ADD ESI, 8
ADD EDI, 8
LOOP @@12 ; 名字 BlowFish 加密
XOR EAX, EAX
MOV [EDI], EAX
invoke MyRev, offset data2, offset data1, MAGICLEN
INVOKE MyStrmagic, offset data1, OFFSET data3, MAGICLEN; 字符变换
invoke MyStrlen, offset data3
MOV Snlen, EAX
CDQ
MOV ECX, 8
IDIV ECX
INC EAX
MOV Snlen2, EAX
XOR ECX, ECX
MOV BL, 8
SUB BL, DL
MOV CL, BL
imul eax, 8
mov Snlen3, eax
MOV EAX, Snlen
DEC EAX
@@13:
MOV [data3+EAX+ECX], BL
LOOP @@13 ; 中间字符串后面添加数字
invoke MyRev, offset data3, offset data1, Snlen3 ; 中间字符串反序排列
INVOKE MyStrlen, offset data1
mov ebx, 8
idiv ebx
and eax, 0ffh
mov ecx, eax
mov esi, offset data1
MOV EDI, OFFSET data2
MOV MAGICLEN, 0
@@14:
mov ebx, esi
add ebx, 4
invoke BlowFish_Init, offset PW_2, 16 ; 准备产生序列号
invoke BlowFish_En, esi, ebx
MOV EAX, BFLEFT
MOV EBX, BFRIGHT
MOV [EDI], EAX
MOV [EDI+4], EBX
ADD MAGICLEN, 8
ADD ESI, 8
ADD EDI, 8
LOOP @@14 ; 名字 BlowFish 加密
XOR EAX, EAX
MOV [EDI], EAX
invoke MyRev, offset data2, offset data1, MAGICLEN
INVOKE MyStrmagic, offset data1, OFFSET data3, MAGICLEN ; 字符变换
INVOKE MyStrlen, offset data3
XOR EDX, EDX
MOV SN, 0FFFFFFEFH
loc_4A9528:
MOVZX ESI, DX
MOVZX ESI, [data3+ESI]
MOV EDI, SN
AND EDI, 0FFH
XOR ESI, EDI
MOV ESI, [ESI*4+TABLE2]
MOV EDI, SN
SHR EDI, 08
XOR ESI, EDI
MOV SN, ESI
INC EDX
DEC EAX
JNZ loc_4A9528
MOV EAX, SN
INVOKE dw2hex, EAX, OFFSET LASTSN
invoke SetDlgItemText, hWnd, Edit2, offset LASTSN
mov eax, FALSE
ret
@ERR:
invoke SetDlgItemText, hWnd, Edit2, NULL
invoke SendDlgItemMessage, hWnd, Edit1, EM_SETREADONLY, TRUE, NULL
invoke SetDlgItemText, hWnd, STATIC1, offset NAMEERR
invoke SetDlgItemText, hWnd, STATIC2, NULL
invoke SetDlgItemText, hWnd, IDABOUT, OFFSET szOK
MOV EAX, FALSE
RET
.elseif eax == IDABOUT
invoke GetDlgItemText, hWnd, IDABOUT, OFFSET szID, 5
MOVZX EAX, BYTE PTR [szID]
.IF EAX == 79 ; 此时显示的是 OK
invoke GetDlgItem, hWnd, Edit1
invoke SetFocus, eax
invoke SendDlgItemMessage, hWnd, Edit1, EM_SETREADONLY, FALSE, NULL
invoke SetDlgItemText, hWnd, STATIC1, offset STRDEF1
invoke SetDlgItemText, hWnd, STATIC2, offset STRDEF2
invoke SetDlgItemText, hWnd, IDABOUT, offset szABOUT
MOV EAX, FALSE
RET
.ELSE
invoke GetDlgItem, hWnd, Edit2
invoke SetFocus, eax
invoke SendDlgItemMessage, hWnd, Edit1, EM_SETREADONLY, TRUE, NULL
invoke SetDlgItemText, hWnd, STATIC1, offset STRABOUT1
invoke SetDlgItemText, hWnd, STATIC2, offset STRABOUT2
invoke SetDlgItemText, hWnd, IDABOUT, offset szOK
MOV EAX, FALSE
RET
.ENDIF
.endif
.else
mov eax, FALSE
ret
.endif
mov eax, FALSE
RET
_ProcDlgMain endp
; 主程序
start:
invoke InitCommonControls
invoke GetModuleHandle, NULL
mov hInstance, eax
invoke DialogBoxParam, hInstance, DLG_MAIN, NULL, offset _ProcDlgMain, 0
invoke ExitProcess, NULL
end start
end 4dd466d0e1e0cc123afbe504da2462ca
```
