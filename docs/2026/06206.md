
# [分享]修复`IAT_Execryptor`脚本

在看雪论坛下载了ASProtect.2.3.6.26.Modified汉化版，光看了一个PE节，用壳用PEid扫了一下，不知是什么壳，可能是PEid的Sign库太老了，我OD原来是启动着的，后来运行了这家伙，可恶被强暴了，激动着要用OD载入调试之，就单步异常，又不是入口，很快又被踢开了，原来有`Tls`小刀！没接触过 execryptor（后来种种表现有点象simonzh2000兄文章提的execryptor），Hide了OD，太多单步了，糊涂地死了，又发现修改`XXXXX`

``` text
mov     dword ptr [ecx], 10001
mov     dword ptr [ecx], 10013
```

将其所有`NOP`之，保留`Int3`异常项，一路shit，`Code`解压之，待`VM`（后来才知叫`VM`）解开，同样`NOP`之，`Code`节F2内存访问断点，shit了几下中断，来到真正入口。看了一下是Delphi，再F4到第一个API调用，盲的都知道是`GetModuleHandleA()`，返回处下了断点跟之，死得难看，几次跟踪后，发现`IAT`有这样的处理：1.未到`OEP`入口时`IAT`存放的是固定的`VM`地址和模块序号，序号的将会自动解出API替换之；2.存入`VM`地址的，有的会运行时解之替换，有时雷打不动。所以要在`OEP`处解之，必定要用`VM`解之！

于是在论坛上找了些文章和脚本，抄起脚本器调试之，发现没有脚本可顺利运行到结束，Crash之或没什么反映，看来看去还是 okdodo 的脚本处理得还可以（呵呵，加壳者弄的水印），便找起Bug来，发现扫描判断有些不保险的安排！改了改它，便发现可以顺利地运行，报告可观点！

Dump后修复`OEP`、`IAT`和`Tls`，`kqunredf`节名改回`.idata`，Cut掉壳的3节节省了`185000h`，Cut掉空的`reloc`节少了`12000h`，重新调整了资源节又少了`1000h`，改回对节数，保存运行之OK了！

``` text
/*

Script written by okdodo  2007/03

Tested for execryptor v2.24/v2.25



Ollyice: Ignore all exceptions (add 0EEDFADE,C0000005,C000001E)

HideOD : Check HideNtDebugBit and ZwQueryInformationProcess(method2)



Test Environment : Ollyice 1.1 + HideOD    

                   ODBGScript 1.51 under WINXP

Thanks :

         kanxue     - author of HideOD       

         hnhuqiong  - author of ODbgScript 1.51

*/

/*

  Test Environment : Ollyice 1.1 + HideOD    

                   ODBGScript 1.52 under WINXP

  test only by ASProtect.2.3.6.26.Modified by KuNgBiM && Supply: shoooo^_^(I don't konow what Packer on it, but sure can do by this script)

  fixed some bug in crash condition

  Modify by NewHand

*/

data:

    var hInstance
    var codeseg
    var vmseg
    var ep
    var oep
    var esptmp
    var _esp
    var iat_start
    var iat_end
    var iat_cur
    var addr
    var c_gpa
    var ibase
    var iend
    var temp
    var tmp
    var SBM
    var TOA
    var mbase
    var msize

code:

    bphwcall
    gpa "SetBkMode","GDI32.dll"
    mov SBM,$RESULT
    REV SBM
    mov SBM,$RESULT
    itoa SBM
    gpa "TextOutA","GDI32.dll"
    mov TOA,$RESULT
    REV TOA
    mov TOA,$RESULT
    itoa TOA

    gpa "VirtualFree","kernel32.dll"
    bphws $RESULT,"x"
    run
    bphwc $RESULT
    rtu
    gmi eip,MODULEBASE
    mov hInstance,$RESULT
    mov temp,$RESULT
    add temp,3c
    mov temp,[temp]
    add temp,hInstance
    add temp,28
    mov temp,[temp]
    add temp,hInstance
    mov ep,temp // Clear Deubuger CC on EP

    bc ep

    gmemi eip,MEMORYBASE
    mov codeseg,$RESULT
/*
    find $RESULT,#2ECC9D#
    cmp $RESULT,0
    jne @error
    mov [$RESULT],#2ECC90# // what??? Clear comment if your Target need 
*/
    gpa "EnumWindows","user32.dll"
    mov [$RESULT],#8BC09C85C09D0578563412C20800# // Enum Fail

    gpa "CreateThread","kernel32.dll"
    find $RESULT,#FF7518#
    mov [$RESULT],#6A0490#  // Suspend Child Thread

    gpa "ZwCreateThread","ntdll.dll"
    bp $RESULT

loop1:
    esto
    cmp eip,$RESULT
    jne loop1
    bc $RESULT 
    bp ep // set breakpoint on src ep

bpep:
    run // skip Load DLL
    cmp eip,ep
    je loop2
    jmp bpep

loop2:
    bc ep
    mov esptmp,esp
    sub esptmp,4    

    mov temp,codeseg
    sub temp,1
    gmemi temp,MEMORYBASE // GetPreBlockBase
    mov vmseg,$RESULT
    gmemi temp,MEMORYSIZE
    bprm vmseg,$RESULT

loop3:    
    esto
    mov tmp,eip // handle IAT API Entry
    mov tmp,[tmp]
    cmp tmp,992C008A
    jne loop5
    mov oep,eax // Get pseudo OEP
    sti
    bprm oep,1

loop4:
    esto
    cmp eip,oep
    jne loop4
    jmp iat

loop5:
    cmp esp,esptmp
    jne loop3

iat:
    bpmc
    mov oep,eip
    cmt eip,"OEP?" 
    gmi eip, MODULEBASE
    mov ibase, $RESULT
    mov temp,ibase
    add temp,3C
    mov temp,[temp]
    add temp,ibase
    add temp,50
    mov iend,[temp]
    add iend,ibase

    mov count,0
    mov iatbase,0
    mov mbase,codeseg

hwloop: 
    sub mbase,1 
    cmp mbase,ibase
    jb regnext
    gmemi mbase,MEMORYBASE
    mov mbase,$RESULT
    gmemi msize,MEMORYSIZE
    mov msize,$RESULT
    mov temp,mbase

    cmp iatbase,0
    jne vmsegloop
    eval #{SBM}#
    find temp,$RESULT
    cmp 0,$RESULT
    je findTextOutA
    gmemi $RESULT,MEMORYBASE
    mov iatbase,$RESULT
    jmp vmsegloop

findTextOutA:
    cmp iatbase,0
    jne vmsegloop
    eval #{TOA}#
    find temp,$RESULT
    cmp 0,$RESULT
    je vmsegloop
    gmemi $RESULT,MEMORYBASE
    mov iatbase,$RESULT

vmsegloop:
    find temp,#0355FC03C28B000345FC#
    mov tmp, $RESULT
    cmp tmp,0
    je regged
    add tmp,0A
    bphws tmp,"x"
    mov temp,tmp
    mov c_gpa,tmp 
    inc count
    jmp vmsegloop

regged:
    cmp count,0
    jne hwloop

regnext:
    mov mbase,codeseg

hwloop1: 
    sub mbase,1 
    cmp mbase,ibase
    jb @iatinit
    gmemi mbase,MEMORYBASE
    mov mbase,$RESULT
    mov temp,mbase

    cmp iatbase,0
    jne vmsegloop1
    eval #{SBM}#
    find temp,$RESULT
    cmp 0,$RESULT
    je findTextOutA1
    gmemi $RESULT,MEMORYBASE
    mov iatbase,$RESULT
    jmp vmsegloop1

findTextOutA1:
    cmp iatbase,0
    jne vmsegloop1
    eval #{TOA}#
    find temp,$RESULT
    cmp 0,$RESULT
    je vmsegloop1
    gmemi $RESULT,MEMORYBASE
    mov iatbase,$RESULT

vmsegloop1:
    find temp,#0345FC8945F48B45F4#
    mov tmp, $RESULT
    cmp tmp,0
    je hwloop1
    add tmp,3
    bphws tmp,"x"
    mov temp,tmp
    mov c_gpa,tmp
    inc count 
    jmp vmsegloop1

@iatinit:
    cmp iatbase,0
    je @error
    cmp count,0
    je @error
    gmemi iatbase,MEMORYSIZE
    mov iat_end,$RESULT
    add iat_end,iatbase
    sub iat_end,4
    mov _esp,esp
    mov iat_cur,iatbase
    sub iat_cur,4
    mov count,0    
    pause
@ImpInc:  
    add iat_cur,4
    cmp iat_cur,iat_end
    ja @end
    mov addr,[iat_cur]
    cmp addr,0
    je @ImpInc
    gn addr
    cmp $RESULT,0 // There is Real Api!
    jne @ImpInc
    cmp count,0
    jne @next
    mov iat_start,iat_cur
    log iat_start

@next:
    cmp addr,iatbase
    jb @error2
    cmp addr,iend
    jae @error2
    cmp addr,iat_end
    je @end
    inc count
    mov temp,iat_cur
    mov esp,_esp
    mov eip,addr
    mov [esp],eip  
    esto    
    mov [iat_cur],eax    
    jmp @ImpInc

@end:
    mov iat_end,temp
    add temp,8
    cmp [temp],0
    je @exit
    sub temp,4
@IsApi:
    add temp,4
    gn [temp]
    cmp $RESULT,0
    jne @IsApi
    cmp [temp],0
    jne @error
    add temp,4
@exit:
    sub temp, iat_start
    bphwcall
    log iat_end
    mov eip,oep
    eval "IAT Begin: {iat_start}   IAT End: {iat_end}   Size: {temp} "
    msg $RESULT   
    msg "Script ends ok!  Find the OEP manually and dump it~"
    ret

@error:
    bphwcall
    msg "ERROR!"
    ret 
@error2:
    eval "May be [{iat_cur}]-> {addr}: IAT has some problem or Other data!"
    msg $RESULT
    mov iat_end, iat_cur
    mov temp, iat_end 
    jmp @exit

@MayEnd:
    mov iat_end, iat_cur
    sub iat_end, 4
    mov temp, iat_end
    jmp @exit  
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
