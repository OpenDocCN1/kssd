

# EncryptPE V2.2007.4.11S 方式固定机器码的脚本

> 原文：[`www.pediy.com/kssd/pediy10/59696.html`](https://www.pediy.com/kssd/pediy10/59696.html)

///////////////////////////////////////////////////////////////////////////////////  
// FileName    :  Modify\_MecjineID.oSc  
// Comment     :  EncryptPE V2.2007.4.11. Modify\_MechinID   
// Environment :  WinXP SP2,TheODBG\_fly + HideOD181+ OllyScript1.65   
// Author      :  d\_h  
// Date        :  2008-2-15  
// WebSite     :  http://www.pediy.com  
///////////////////////////////////////////////////////////////////////////////////  
var ID  
var ID0  
var ID2  
var ID3  
var ID4  
var addr1  
var addr2  
var RegSet  
var patch1  
var patch2  
var patch3  
var patch4  
var patch5  
var patch6  
var RandInt  
var count  
var tmp  
  
  
  
findID:  
    cmp $VERSION, "1.48"   
    jb version  
  
    bphwcall  
    bpmc  
  
    gpa "IsDebuggerPresent","kernel32.dll"  
    bp $RESULT  
    esto  
    bc $RESULT  
  
//    gpa "ReadFile", "kernel32.dll"       
//    gpa "VirtualAlloc", "kernel32.dll"  
//    gpa "RegSetValueExA", "advapi32.dll"       
//    mov RegSet, $RESULT  
      
/\*  
    lbl 711F54AC, "GetBIOS"  
    lbl 711F55D0, "GetHD"  
    lbl 711F5684, "GetMAC"  
    lbl 711F5758, "GetCPUID"  
    lbl 71123170, "GetRandomValue"  
  
    mov ID,711F57BD  
    mov ID1,711FC408  
    mov ID2,711FC0D0  
    mov ID3,711F5864  
    mov ID4,711F5B98  
  
    CMT ID1+250 , "Gen MechineID1"  
    CMT ID1+193 , "call GetCPUID1"  
    CMT ID1+170 , "call GetMAC1"  
    CMT ID1+14D , "call GetHD1"  
    CMT ID1+12A , "call GetBIOS1"  
  
    CMT ID2+24F , "Gen MechineID2"  
    CMT ID2+192 , "call GetCPUID2"  
    CMT ID2+16F , "call GetMAC2"  
    CMT ID2+14C , "call GetHD2"  
    CMT ID2+129 , "call GetBIOS2"  
  
    CMT ID3+24B , "Gen MechineID3"  
    CMT ID3+192 , "call GetCPUID3"  
    CMT ID3+16F , "call GetMAC3"  
    CMT ID3+14C , "call GetHD3"  
    CMT ID3+129 , "call GetBIOS3"  
  
    CMT ID4+24C , "Gen MechineID4"  
    CMT ID4+193 , "call GetCPUID4"  
    CMT ID4+170 , "call GetMAC4"  
    CMT ID4+14D , "call GetHD4"  
    CMT ID4+12A , "call GetBIOS4"  
  
    CMT 712010FB, "call GetRandomValue"  
    CMT 71201649, "call GetRandomValue"  
    CMT 711F6CFF, "call GetRandomValue"  
\*/  
  
    mov  patch1 , 7120350A  
    mov \[patch1\], #7400#  
    mov  patch2 , 71201642  
    mov \[patch2\], #EB5E#  
    mov  patch3 , 71128294  
    mov \[patch3\], #33C0C39090#  
    mov  patch4,  711F217E  
    mov \[patch4\], #909090909090#  
    mov  patch5,  711F9561  
    mov \[patch5\], #7400#  
    mov  patch6,  7118C274     //RegSetVelueExA  
//  mov \[patch6\], #EB24#       //#7424#  
    mov  patch7,  71123170+16  //GetRandomValue  
//    mov \[patch7\], #33C0#     //#89D0#  
    mov RandInt, patch7+3  
  
    mov ID1,711FC679  
    mov ID2,711FC340  
    mov ID3,711F5AD0  
    mov ID4,711F5E05  
  
    bphws ID1,"x"  
    bphws ID2,"x"  
    bphws ID3,"x"  
    bphws ID4,"x"  
  
    bp RandInt  
      
next:  
    esto  
  
    cmp eip ,ID1  
    je continue  
  
    cmp eip ,ID2  
    je continue  
  
    cmp eip ,ID3  
    je continue  
  
    cmp eip ,ID4  
    je continue  
  
    cmp eip, RandInt  
    je setzero  
  
    jmp next  
  
continue:  
    mov tmp,eip  
    eval "-----SetID At: {tmp}--------"  
    log $RESULT     
//  机器 ID ：6B79A7CB B6F7ECD7 49081328 ，后两个反序   
    mov ebx,6B79A7CB   
    mov addr1,ebp-08   
    mov \[addr1\],#D7ECF7B6#  
    mov addr2,ebp-0C   
    mov \[addr2\],#28130849#  
  
    jmp next  
  
setzero:  
  
    eval "RandInt--{count}: {eax}"  
    log $RESULT  
    mov eax,0  
  
    inc count  
    cmp count,85  
    jb next   
    
    bc RandInt  
    mov count,0  
    jmp next  
          
version:  
    msg "插件版本过低"    
    ret