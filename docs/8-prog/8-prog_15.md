

# 原创脚本 For RadAsm(HLA.ini)

> 原文：[`www.pediy.com/kssd/pediy10/60535.html`](https://www.pediy.com/kssd/pediy10/60535.html)

原创脚本 For RadAsm(HLA.ini)  
累死人啊  
搞了三日 HLA 终于完工  
  
关于 RadAsm 是多种编程语言 IDE 整合工具  
最终版本为 RadASM 2.2.1.1   
但是它的 HLA.ini 却无法 Work  
只好一边演究 HLA 的命令列指令参数  
一边些修修改改整合进 RadASM

*   ;; --==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--  
    ;;\[Enviroment\]  
    ;;1=path,c:\\hla;c:\\masm32\\bin;  
    ;;2=lib,c:\\hla\\hlalib  
    ;;3=hlainc,c:\\hla\\include  
    ;; --==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--  
    ;; :SetInit  
    ;; Set Path=D:\\OO\\Asm\\Hla199;D:\\OO\\Asm\\m32v9r\\Bin;%Path%  
    ;; Set Lib=D:\\OO\\Asm\\Hla199\\hlalib;D:\\OO\\Asm\\m32v9r\\Lib;%Lib%  
    ;; Set HlaInc=D:\\OO\\Asm\\Hla199\\include  
    ;; Set HlaLib=D:\\OO\\Asm\\Hla199\\hlalib\\hlalib.lib  
    ;; --==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--  
      
      
    \[Enviroment\]  
    1=Path,D:\\OO\\Asm\\Hla199;D:\\OO\\Asm\\m32v9r\\Bin;  
    2=Lib,D:\\OO\\Asm\\Hla199\\hlalib;D:\\OO\\Asm\\m32v9r\\Lib;  
    3=HlaInc,D:\\OO\\Asm\\Hla199\\include  
    4=HlaLib,D:\\OO\\Asm\\Hla199\\hlalib\\hlalib.lib  
      
      
    \[MinorFiles\]  
    ;Used by PthExpl++ addin to delete minor files.  
    MinorFiles=\*.ro.inc;\*.res;\*.obj;\*.link;\*.extpub.inc;\*.data.inc;\*.consts.inc;\*.bss.inc;  
      
    \[CharTab\]  
    2=0251222522222236;20-2F  
    3=1111111111222221;30-3F  
      
    \[Open\]  
    0="Hla Files (\*.hla;\*.hhf),\*.hla;\*.hhf"  
    1="Resource Files (\*.rc),\*.rc"  
    2="Assembly Files (\*.asm;\*.inc),\*.asm;\*.inc"  
    3="Text Files (\*.txt),\*.txt"  
    4="All Files (\*.\*),\*.\*"  
      
    \[CodeMacro\]  
    ;KeyWord,InsertBeforeActiveLine,InsertAfterActiveLine  
    ;{I}=Copy indent from keyword line  
    ;{C}=Cr  
    ;{S}=Space  
    ;{T}=Tab  
    ;{$}=Copy Proc/Struct Name  
    ;The macros are set up to assume AutoIndent on  
    1=begin,{T}{C}{I}{T},{C}{C}{I}end {$}  
    2=if,{C}{I}{T}, {C}{C}{I}endif;  
    3=elseif,{C}{I}{T}  
    4=else, {C}{I}{T}  
    5=while,{C}{I}{T}, {C}{C}{I}endwhile;  
    6=repeat,{C}{I}{T}, {C}{C}{I}until( expr );  
    7=for,{C}{I}{T}, {C}{C}{I}endfor;  
    8=forever,{C}{I}{T}, {C}{C}{I}endfor;  
    9=foreach,{C}{I}{T}, {C}{C}{I}endfor;  
    10=try,{C}{I}{T}, {C}{C}{I}exception{C}{C}{I}endtry;  
    11=switch,{C}{I}{T}, {C}{C}{I}endswitch;  
    12=record,{C}{I}{T}, {C}{C}{I}endrecord;  
    13=union,{C}{I}{T}, {C}{C}{I}endunion;  
    14=class,{C}{I}{T}, {C}{C}{I}endclass;  
    15=#macro,{C}{I}{T}, {C}{C}{I}#endmacro;  
    16=#if,{C}{I}{T}, {C}{C}{I}#endif  
    17=#while,{C}{I}{T}, {C}{C}{I}#endwhile  
    18=#for,{C}{I}{T}, {C}{C}{I}#endfor  
      
    \[Code\]  
    Code={C},procedure  
    Const=;,$ :=  
    Data=;,$ :dword;  
    Macro=#endmacro,#macro  
    Struct=endrecord;,$ record  
      
    \[CodeBlock\]  
    1=begin,end,,,6  
    2=if,endif,elseif,else,0  
    3=$ :record,endrecord;,,,6  
    4=#macro,#endmacro;,,,6  
      
    \[Api\]  
    Trig=  
    Call=Hla\\hlaCall.api  
    Struct=Hla\\hlaStruct.api  
    Const=Hla\\hlaConst.api  
    Word=Hla\\hlaWord.api  
    Message=Hla\\hlaMessage.api  
      
    \[Edit\]  
    Font=Courier New  
    FontHeight=-12  
    FontWeight=400  
    TxtFont=Courier New  
    TxtFontHeight=-12  
    TxtFontWeight=400  
    TabSize=4  
    Backup=9  
    AutoSave=1  
    BacupPath=$P\\Bak\\  
    AutoIndent=1  
    ApiList=0  
    ApiToolTip=1  
    Properties=1  
    MouseWheel=1  
    SaveSize=1  
    Maximize=1  
    ApiConst=1  
    CodeWrite=1  
    TabToSpc=0  
    ApiStruct=1  
    PrnFont=Courier New  
    PrnFontHeight=-12  
    PrnFontWeight=400  
    PrnPage=21000,29700,1000,1000,1000,1000,0  
    PrnOption=2,0,1,1,1  
    PrnColor=0,32768,8421504,255,65280,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760  
    FontCharSet=0  
    TxtFontCharSet=0  
    PrnFontCharSet=0  
    ApiWordConv=1  
    ApiWordLocal=1  
    CodeFiles=.asm.inc.rc.tpl.rad.hla.hhf.  
    ProcsToApi=1  
    ProcInSBar=0  
    LnrFont=Terminal  
    LnrFontHeight=-8  
    LnrFontColor=8388608  
    LnrWidth=4  
    LnrOnOpen=0  
    PageBreak=63  
    PageSize=63  
    ApiShiftSpace=0  
    OpenCollapsed=0  
    FontItalic=0  
    TxtFontItalic=0  
    CmntBlock=/\*,\*/  
    CmntChar=//  
    EnterOnTab=0  
    AutoBrackets=1  
    CodeTooltip=1  
    HexFont=Courier New  
    HexFontHeight=-12  
    HexFontWeight=400  
    HexFontItalic=0  
    HexFontCharSet=0  
    DlgFont=MS Sans Serif  
    DlgFontHeight=-11  
    DlgFontWeight=400  
    DlgFontItalic=0  
    DlgFontCharSet=0  
    ToolFont=MS Sans Serif  
    ToolFontHeight=-11  
    ToolFontWeight=400  
    ToolFontItalic=0  
    ToolFontCharSet=0  
    ThreadBuild=1  
    ChangeNotify=1  
    MinimizeOnBuild=0  
    HiliteLine=0  
    HiliteCmnt=0  
      
    \[Dialog\]  
    Grid=1  
    GridX=3  
    GridY=3  
    SnapToGrid=1  
    ShowSize=1  
    SaveRC=1  
    SimpleProperty=0  
      
    \[Error\]  
    BookMark=3  
    nAsm=99   
    Identify=Error   
    Skip=2   
      
      
    \[Paths\]  
    ;;$A=C:\\Hla  
    $A=D:\\OO\\Asm\\hla199  
    $B=$A  
    $D=$R\\AddIns  
    $H=$A\\Doc  
    $I=$A\\Include  
    $L=$A\\hlalib  
    $P=$R\\Hla\\Projects  
    $S=$R\\Hla\\Sniplets  
    $T=$R\\Hla\\Templates  
    $M=$R\\Hla\\Macro  
      
    \[MakeFiles\]  
    0=.rap  
    1=.rc  
    2=.asm  
    3=.obj  
    4=.res  
    5=.exe  
    6=.def  
    7=.dll  
    8=.txt  
    9=.lib  
    10=.mak  
    11=.hla  
    12=" "  
      
    \[Project\]  
    ;;Type=Console App,Windows App  
    Type=Console App,Windows App (No Res),Windows App  
    Files=hla,hhf,rc,def  
    Folders=Bak,Res,Tmp,Doc  
    ;;MenuMake=Build,Build All,Compile RC,Check Syntax,Run  
    MenuMake=Compile\_RC,Build,Build\_All,Check\_Syntax,Run  
    Group=1  
    GroupExpand=1  
      
      
      
    \[MakeDefNoProject\]  
    ;; --==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--  
    ;; MenuMake=Build,Build All,Compile RC,Check Syntax,Run  
    ;; 0=.rap 1=.rc 2=.asm 3=.obj 4=.res 5=.exe 6=.def 7=.dll 8=.txt 9=.lib 10=.mak 11=.hla 12=" "  
    ;; --==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--  
    ;; Menu=             1,    1,        0,           1,  1,0,0,0,0  
    ;; MenuMake=Compile\_RC,Build,Build\_All,Check\_Syntax,Run  
    ;; --==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--  
    Menu=1,1,0,1,1,0,0,0,0  
    ;;1=0,OT,$B\\hla -p:tmp -w -i:$I -Lib:$L,11  
    ;;2=0,OT,$B\\hla -p:tmp -w -i:$I -Lib:$L,11  
    1=0,OT,$B\\PoRc /r,1  
    2=0,OT,$B\\hla -w -c,11  
    3=0,OT,$B\\hla -w,11,1  
    4=0,OT,$B\\hla -w -s,11  
    5=0,OT,$R\\run.bat,5  
    6=  
      
      
      
    \[Console App\]  
    Files=1,0,0,0,0  
    Folders=1,0,1  
    ;; --==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--  
    ;; MenuMake=Build,Build All,Compile RC,Check Syntax,Run  
    ;; 0=.rap 1=.rc 2=.asm 3=.obj 4=.res 5=.exe 6=.def 7=.dll 8=.txt 9=.lib 10=.mak 11=.hla 12=" "  
    ;; #\*==============================================================\*#  
    ;; Menu=             0,    1,        1,           1,  1,0,0,0,0  
    ;; MenuMake=Compile\_RC,Build,Build\_All,Check\_Syntax,Run  
    ;; --==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--  
    MenuMake=0,1,1,1,1,0,0,0,0  
    ;;1=0,OT,$B\\hla -p:tmp -w -i:$I -Lib:$L,11  
    ;;2=0,OT,$B\\hla -p:tmp -w -i:$I -Lib:$L,11  
    1=0,OT,$B\\PoRc /r,1  
    2=0,OT,$B\\hla -c,11  
    ;; Error Console App No .RC  
    ;;3=0,OT,$B\\hla ,11,1  
    ;; No .RC  
    3=0,OT,$B\\hla ,11  
    4=0,OT,$B\\hla -s,11  
    5=0,OT,$R\\run.bat,5  
    6=  
      
      
      
    \[Windows App (No Res)\]  
    Files=1,1,0,0,0  
    Folders=0,0,0,0  
    MenuMake=0,1,1,1,1,0,0,0,0  
    ;; --==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--  
    ;; MenuMake=Build,Build All,Compile RC,Check Syntax,Run  
    ;; 0=.rap 1=.rc 2=.asm 3=.obj 4=.res 5=.exe 6=.def 7=.dll 8=.txt 9=.lib 10=.mak 11=.hla 12=" "  
    ;; #\*==============================================================\*#  
    ;; Menu=             1,    1,        1,           1,  1,0,0,0,0  
    ;; MenuMake=Compile\_RC,Build,Build\_All,Check\_Syntax,Run  
    ;; --==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--  
    ;;1=0,OT,$B\\hla -p:tmp -w -i:$I -Lib:$L,11  
    ;;2=0,OT,$B\\hla -p:tmp -w -i:$I -Lib:$L,11  
    1=0,OT,$B\\PoRc /r,1  
    2=0,OT,$B\\hla -w -c,11  
    ;;3=0,OT,$B\\hla -w,11,1  
    3=0,OT,$B\\hla -w,11  
    4=0,OT,$B\\hla -w -s,11  
    5=0,OT,$R\\run.bat,5  
    6=  
      
      
      
    \[Windows App\]  
    Files=1,1,1,0,0  
    Folders=1,1,1  
    ;; --==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--  
    ;; MenuMake=Build,Build All,Compile RC,Check Syntax,Run  
    ;; 0=.rap 1=.rc 2=.asm 3=.obj 4=.res 5=.exe 6=.def 7=.dll 8=.txt 9=.lib 10=.mak 11=.hla 12=" "  
    ;; #\*==============================================================\*#  
    ;; Menu=             1,    1,        1,           1,  1,0,0,0,0  
    ;; MenuMake=Compile\_RC,Build,Build\_All,Check\_Syntax,Run  
    ;; --==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--==\*\*==--  
    MenuMake=1,1,1,1,1,0,0,0,0  
    ;;1=0,OT,$B\\hla -p:tmp -w -i:$I -Lib:$L,11  
    ;;2=0,OT,$B\\hla -p:tmp -w -i:$I -Lib:$L,11  
    1=0,OT,$B\\PoRc /r,1  
    2=0,OT,$B\\hla -w -c,11  
    3=0,OT,$B\\hla -w,11,1  
    4=0,OT,$B\\hla -w -s,11  
    5=0,OT,$R\\run.bat,5  
    6=  
      
      
      
    \[MenuMake\]  
    ;;;;Menu=             1,    1,        1,           1,  1,0,0,0,0  
    ;;;;MenuMake=Compile\_RC,Build,Build\_All,Check\_Syntax,Run  
    1=&Compile RC,372,M,1  
    2=&Build,1396,M,2  
    3=Build &All,116,M,3  
    4=-,0,M,  
    5=&Syntax Check,1396,M,4  
    6=-,0,M,  
    7=&Run,884,M,5  
    8=$$,0,M,0  
    9=$$,0,M,0  
    Ver=100  
      
      
      
    \[Color\]  
    UseColor=1  
    Back=2039583  
    Fore=12632256  
    Bar=12550399  
    Cmnt=42008575  
    Str=16777215  
    Oper=2631935  
    C0=8454016  
    C1=16777088  
    C2=16744448  
    C3=12615935  
    C4=12615808  
    C5=11184640  
    C6=4227327  
    C7=16711935  
    C8=33488896  
    C9=16711808  
    Output=12058623  
    Project=12058623  
    Property=12058623  
    Dialog=8421376  
    DivLine=1  
    NoFlicker=0  
    PageBreak=63  
    SelBack=16711680  
    SelText=16777215  
    HiLine1=5395026  
    HiLine2=12644544  
    HiLine3=6052956  
    BarPen=8421504  
    LnrFontColor=8388608  
    C10=285147136  
    C11=16711680  
    C12=16711680  
    C13=16744576  
    C14=12615808  
    C15=16711680  
    Num=8421504  
    Outputtext=8388608  
    Projecttext=8388608  
    Propertytext=8388608  
    CustColors=0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0  
      
    \[KeyWords\]  
    ;  
    ;Registers  
    ;  
    C0=AH AL AX BH BL BP BX CH CL CR0 CR2 CR3 CSEG CX DH DI DL DR0 DR1 DR2 DR3 DR6 DR7 DSEG DX EAX EBP EBX ECX EDI EDX ESEG ESI ESP FSEG GSEG MM0 MM1 MM2 MM3 MM4 MM5 MM6 MM7 SI SP SSEG ST0 ST1 ST2 ST3 ST4 ST5 ST6 ST7 TR3 TR4 TR5 TR6 TR7 XMM0 XMM1 XMM2 XMM3  
    ;  
    ;Instructions  
    ;  
    C1=AAA AAD AAM AAS ADC ADD AND ARPL BOUND BSF BSR BSWAP BT BTC BTR BTS CALL CBW CDQ CLC CLD CLI CLTS CMC CMOVA CMOVAE CMOVB CMOVBE CMOVC CMOVE CMOVG CMOVGE CMOVL CMOVLE CMOVNA CMOVNAE CMOVNB CMOVNBE CMOVNC CMOVNE CMOVNG CMOVNGE CMOVNL CMOVNLE CMOVNO CMOVNP CMOVNS CMOVNZ CMOVO CMOVP CMOVPE CMOVPO CMOVS CMOVZ CMP CMPSB CMPSD CMPSW CMPXCHG CMPXCHG8B CPUID CWD CWDE DAS DEC DIV DAA DAS ENTER HLT IDIV IMOD IMUL IN INC INSB INSD INSW INT INTMUL INTO INVD INVLPG IRET IRETD JA JAE JB JBE JC JCXZ JE JECXZ JG JGE JL JLE JMP JNA JNAE JNB JNBE JNC JNE JNG JNGE JNL JNLE JNO JNP JNS JNZ JO JP JPE JPO JS JZ LAHF LAR LDS LEA LEAVE LES LFS LGDT LGS LIDT LLDT LOCK.ADC LOCK.ADD LOCK.AND LOCK.BTC LOCK.BTR LOCK.BTS LOCK.CMPXCHG LOCK.DEC LOCK.INC LOCK.NEG LOCK.NOT LOCK.OR LOCK.SBB LOCK.SUB LOCK.XADD LOCK.XCHG LOCK.XOR LODSB LODSD LODSW LOOP LOOPE LOOPNE LOOPNZ LOOPZ LSL LSS LTREG MOV MOVSB MOVSD MOVSW MOVSX MOVZX MUL NEG NOP NOT OR OUT OUTSB OUTSD OUTSW POP POPA POPAD POPF POPFD PUSH PUSHA PUSHAD PUSHD PUSHF PUSHFD PUSHW RCL RCR RDMSR RDPMC RDTSC REP.INSB REP.INSD REP.INSW REP.MOVSB REP.MOVSD REP.MOVSW REP.OUTSB REP.OUTSD REP.OUTSW REP.STOSB REP.STOSD REP.STOSW REPE.CMPSB REPE.CMPSD REPE.CMPSW REPE.SCASB REPE.SCASD REPE.SCASW REPNE.CMPSB REPNE.CMPSD REPNE.CMPSW REPNE.SCASB REPNE.SCASD REPNE.SCASW REPNZ.CMPSB REPNZ.CMPSD REPNZ.CMPSW REPNZ.SCASB REPNZ.SCASD REPNZ.SCASW REPZ.CMPSB REPZ.CMPSD REPZ.CMPSW REPZ.SCASB REPZ.SCASD REPZ.SCASW RET ROL ROR RSM SAHF SAL SAR SBB SCASB SCASD SCASW SETA SETAE SETB SETBE SETC SETE SETG SETGE SETL SETLE SETNA SETNAE SETNB SETNBE SETNC SETNE SETNG SETNGE SETNL SETNLE SETNO SETNP SETNS SETNZ SETO SETP SETPE SETPO SETS SETZ SGDT SHL SHLD SHR SHRD SIDT SLDT STC STD STI STOSB STOSD STOSW STREG SUB TEST UD2 VERR VERW WAIT WBINVD WRMSR XADD XCHG XLAT XOR  
    ;  
    ;FPU Instructions  
    ;  
    C2=F2XM1 FABS FADD FADDP FBLD FBSTP FCHS FCLEX FCMOVA FCMOVAE FCMOVB FCMOVBE FCMOVE FCMOVNA FCMOVNAE FCMOVNB FCMOVNBE FCMOVNE FCMOVNU FCMOVU FCOM FCOMI FCOMIP FCOMP FCOMPP FCOS FDECSTP FDIV FDIVP FDIVR FDIVRP FFREE FIADD FICOM FICOMP FIDIV FIDIVR FILD FIMUL FINCSTP FINIT FIST FISTP FISUB FISUBR FLD FLD1 FLDCW FLDENV FLDL2E FLDL2T FLDLG2 FLDLN2 FLDPI FLDZ FMUL FMULP FNOP FPATAN FPREM FPREM1 FPTAN FRNDINT FRSTOR FSAVE FSCALE FSIN FSINCOS FSQRT FST FSTCW FSTENV FSTP FSTSW FSUB FSUBP FSUBR FSUBRP FTST FUCOM FUCOMI FUCOMIP FUCOMP FUCOMPP FWAIT FXAM FXCH FXTRACT FYL2X FYL2XP1  
    ;  
    ;MMX Instructions  
    ;  
    C3=EMMS MOVD MOVQ PACKSSDW PACKSSWB PACKUSWB PADDB PADDD PADDSB PADDSW PADDUSB PADDUSW PADDW PAND PANDN PAVGB PAVGW PCMPEQB PCMPEQD PCMPEQW PCMPGTB PCMPGTD PCMPGTW PEXTRW PINSRW PMADDWD PMAXSW PMAXUB PMINSW PMINUB PMOVMSKB PMULHUW PMULHW PMULLW POR PSADBW PSHUFW PSLLD PSLLQ PSLLW PSRAD PSRAW PSRLD PSRLQ PSRLW PSUBB PSUBD PSUBSB PSUBSW PSUBUSB PSUBUSW PSUBW PUNPCKHBW PUNPCKHDQ PUNPCKHWD PUNPCKLBW PUNPCKLDQ PUNPCKLWD PXOR  
    ;  
    ; HLA built-in data types  
    ;  
    C4=BOOLEAN BYTE CHAR CSET DWORD INT128 INT16 INT32 INT64 INT8 LWORD QWORD REAL32 REAL64 REAL80 STRING TBYTE TEXT TEXT THUNK UNS128 UNS16 UNS32 UNS64 UNS8 WCHAR WORD WSTRING  
    ;  
    ;Compile-time language statements, variables, and functions  
    ;  
    C5=#ASM #CLOSEREAD #CLOSEWRITE #ELSE #ELSEIF #EMIT #ENDASM #ENDFOR #ENDIF #ENDMACRO #ENDTEXT #ENDWHILE #ERROR #FOR #IF #INCLUDE #INCLUDEONCE #KEYWORD #MACRO #OPENREAD #OPENWRITE #PRINT #SYSTEM #TERMINATOR #TEXT #WHILE #WRITE @ABS @ARITY @BASEREG @BOOLEAN @BOUND @BYTE @CEIL @CHAR @CLASS @COS @CSET @CURDIR @CURLEX @CUROBJECT @CUROFFSET @DATE @DEFINED @DELETE @DIM @DWORD @ELEMENTS @ELEMENTSIZE @ENTER @ENUMSIZE @EOS @EVAL @EXACTLYNCHAR @EXACTLYNCSET @EXACTLYNICHAR @EXACTLYNTOMCHAR @EXACTLYNTOMCSET @EXACTLYNTOMICHAR @EXCEPTIONS @EXP @EXTRACT @FILENAME @FIRSTNCHAR @FIRSTNCSET @FIRSTNICHAR @FLOOR @INDEX @INSERT @INT128 @INT16 @INT32 @INT64 @INT8 @INTO @ISALPHANUM @ISCLASS @ISCONST @ISDIGIT @ISEXTERNAL @ISFREG @ISLOWER @ISMEM @ISREG @ISREG16 @ISREG32 @ISREG8 @ISSPACE @ISTYPE   @ISALPHA @ISUPPER @ISXDIGIT @LASTOBJECT @LEAVE @LENGTH @LEX @LINENUMBER @LOCALOFFSET @LOCALSYMS @LOG @LOG10 @LOWERCASE @LWORD @MATCHID @MATCHINTCONST @MATCHISTR @MATCHNUMERICCONST @MATCHREALCONST @MATCHSTR @MATCHSTRCONST @MATCHTOISTR @MATCHTOSTR @MAX @MIN @NAME @NORLESSCHAR @NORLESSCSET @NORLESSICHAR @NORMORECHAR @NORMORECSET @NORMOREICHAR @NTOMCHAR @NTOMCSET @NTOMICHAR @ODD @OFFSET @ONECHAR @ONECSET @ONEICHAR @ONEORMORECHAR @ONEORMORECSET @ONEORMOREICHAR @ONEORMOREWS @OPTSTRINGS @PARMOFFSET @PARMS @PCLASS @PEEKCHAR @PEEKCSET @PEEKICHAR @PEEKWS @PTYPE @QWORD @RANDOM @RANDOMIZE @READ @REAL32 @REAL64 @REAL80 @REG @REG16 @REG32 @REG8 @RINDEX @SECTION @SIN @SIZE @SQRT @STATICNAME @STRBRK @STRING @STRSET @STRSPAN @SUBSTR @TAN @TBYTE @TEXT @TEXT @TIME @TOKENIZE @TRACE @TRIM @TYPE @TYPENAME @UNS128 @UNS16 @UNS32 @UNS64 @UNS8 @UPPERCASE @UPTOCHAR @UPTOCSET @UPTOICHAR @UPTOISTR @UPTOSTR @WCHAR @WORD @WSOREOS @WSTHENEOS @WSTRING @ZEROORMORECHAR @ZEROORMORECSET @ZEROORMOREICHAR @ZEROORMOREWS @ZEROORONECHAR @ZEROORONECSET @ZEROORONEICHAR DOWNTO  
    ;  
    ; HLA Declaration Keywords:  
    ;  
    C6=BEGIN CLASS CONST DUP END ENDCLASS ENDREADONLY ENDRECORD ENDSTATIC ENDSTORAGE ENDUNION ENUM EXTERNAL FORWARD INHERITS ITERATOR LABEL METHOD NAMESPACE POINTER PROCEDURE PROGRAM READONLY RECORD SEGMENT STATIC STORAGE TYPE UNION UNIT VAL VAR  
    ;  
    ;HLA Keywords (non-code producing)  
    ;  
    C7=@ABSTRACT @ALIGN @ALIGNSTACK @CDECL @DISPLAY @EXTERNAL @FAST @FORWARD @FRAME @NOALIGNSTACK @NODISPLAY @NOENTER @NOFRAME @NOLEAVE @NOSTACKALIGN @NOSTORAGE @PASCAL @RETURNS @STACKALIGN @STDCALL @USE @VOLATILE ABSTRACT ALIGN LAZY NAME OVERRIDE OVERRIDES RESULT RETURNS VALRES VMT  
    ;  
    ;HLA High Level Statement Key Words (code producing)  
    ;  
    C8=ANYEXCEPTION BREAK BREAKIF CONTINUE CONTINUEIF DO ELSE ELSEIF ENDFOR ENDIF ENDTRY ENDWHILE EXCEPTION EXIT EXITIF FELSE FOR FOREACH FOREVER IF JF JT RAISE REPEAT THEN TO TRY UNPROTECTED UNTIL WELSE WHILE  
    ;  
    ;Special reserved words  
    ;  
    C9=@A @AE @B @BE @C @E @G @GE @GLOBAL @L @LE @NA @NAE @NB @NBE @NC @NE @NG @NGE @NL @NLE @NO @NP @NS @NZ @O @P @PE @PO @S @TOSTRING @Z THIS  
    ;  
    ;RC file  
    C10=#define #include ACCELERATORS ALT AUTOCHECKBOX AUTORADIOBUTTON BEGIN BITMAP BLOCK CAPTION CLASS COMBOBOX CONTROL CURSOR DIALOGEX DISCARDABLE EDITTEXT EXSTYLE FALSE FILEOS FILETYPE FILEVERSION FONT GROUPBOX ICON LISTBOX LTEXT MENU MENUITEM NOINVERT NULL POPUP PRODUCTVERSION PUSHBUTTON SEPARATOR SHIFT STRINGTABLE STYLE TRUE VALUE VERSIONINFO VIRTKEY  
    C11=  
    C12=  
    C13=  
    C14=  
    C15=  
    ;On Hold  
    C16=  
      
    \[ReallyRad\]  
    AddDlg=1  
    AddMnu=0  
    AddRes=0  
    AddVer=1  
    CtlClk=1  
    MnuSel=1  
    AddFile=0  
    AddFolder=0  
    CtlNme=1  
    AddStr=1  
    AddStrRc=0  
    AddResRc=0  
    AddVerRc=0  
    \[MenuTools\]  
    1=&Notepad,0,T,NOTEPAD.EXE  
    2=&Calculator,0,T,CALC.EXE  
    3=Note&pad (.rap),0,T,NOTEPAD.EXE,0  
    4=-,0,T,  
    5=Command &Line,0,T,cmd.exe  
    6=-,0,T,  
    7=Toolbar creator,0,T,$D\\TbrCreate.exe  
    Ver=100  
    \[MenuMacro\]  
    Ver=100