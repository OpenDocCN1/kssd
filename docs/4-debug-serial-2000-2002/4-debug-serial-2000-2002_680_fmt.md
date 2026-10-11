
# 神奇注册表 0.6

> 原文：[`www.pediy.com/kssd/pediy03/forum1177.htm`](https://www.pediy.com/kssd/pediy03/forum1177.htm)

```
:0041105D 8B3DA0454400             mov edi, dword ptr [004445A0]

:00411063 8D45B0                 lea eax, dword ptr [ebp-50]

:00411066 50                     push eax

:00411067 FFD7                   call edi

:00411069 FFB6D4000000           push dword ptr [esi+000000D4]

:0041106F 8D45D8                 lea eax, dword ptr [ebp-28]

:00411072 50                     push eax

:00411073 FFD7                   call edi

:00411075 83C410                 add esp, 00000010

:00411078 8D45D8                 lea eax, dword ptr [ebp-28]

:0041107B 8BCE                   mov ecx, esi

:0041107D 50                     push eax

:0041107E 8D45B0                 lea eax, dword ptr [ebp-50]

:00411081 50                     push eax

:00411082 E86B000000             call 004110F2<-关键，进去看看

:00411087 83F801                 cmp eax, 00000001<-比较

:0041108A 754F                   jne 004110DB<-跳就死拉

:0041108C 53                     push ebx

:0041108D 6A40                   push 00000040

* Possible Reference to Dialog:

:0041108F 6804754500             push 00457504

* Possible StringData Ref from Data Obj ->"感谢您注册本软件！"

:00411094 68F0744500             push 004574F0

:00411099 8BCE                   mov ecx, esi

:0041109B E885F00100             call 00430125

_______________________________________________________________

:004110F2 55                     push ebp

:004110F3 8BEC                   mov ebp, esp

:004110F5 81EC80000000           sub esp, 00000080

:004110FB 804D9FFF               or byte ptr [ebp-61], FF

:004110FF 8065A000               and byte ptr [ebp-60], 00

:00411103 56                     push esi

:00411104 C6458048               mov [ebp-80], 48 <-48,ee,76....装入

:00411108 C64581EE               mov [ebp-7F], EE

:0041110C C6458276               mov [ebp-7E], 76

:00411110 C645831D               mov [ebp-7D], 1D

:00411114 C6458467               mov [ebp-7C], 67

:00411118 C6458569               mov [ebp-7B], 69

:0041111C C64586A1               mov [ebp-7A], A1

:00411120 C645871B               mov [ebp-79], 1B

:00411124 C645887A               mov [ebp-78], 7A

:00411128 C645898C               mov [ebp-77], 8C

:0041112C C6458A47               mov [ebp-76], 47

:00411130 C6458BF8               mov [ebp-75], F8

:00411134 C6458C54               mov [ebp-74], 54

:00411138 C6458D95               mov [ebp-73], 95

:0041113C C6458E97               mov [ebp-72], 97

:00411140 C6458F5F               mov [ebp-71], 5F

:00411144 C6459078               mov [ebp-70], 78

:00411148 C64591D9               mov [ebp-6F], D9

:0041114C C64592DA               mov [ebp-6E], DA

:00411150 C645936C               mov [ebp-6D], 6C

:00411154 C6459459               mov [ebp-6C], 59

:00411158 C64595D7               mov [ebp-6B], D7

:0041115C C645966B               mov [ebp-6A], 6B

:00411160 C6459735               mov [ebp-69], 35

:00411164 C64598C5               mov [ebp-68], C5

:00411168 C6459977               mov [ebp-67], 77

:0041116C C6459A85               mov [ebp-66], 85

:00411170 C6459B18               mov [ebp-65], 18

:00411174 C6459C2A               mov [ebp-64], 2A

:00411178 C6459D0E               mov [ebp-63], 0E

:0041117C C6459E52               mov [ebp-62], 52

:00411180 C645A1E3               mov [ebp-5F], E3

:00411184 C645A21B               mov [ebp-5E], 1B

:00411188 C645A371               mov [ebp-5D], 71

:0041118C C645A48D               mov [ebp-5C], 8D

:00411190 C645A534               mov [ebp-5B], 34

:00411194 C645A663               mov [ebp-5A], 63

:00411198 C645A7EB               mov [ebp-59], EB

:0041119C C645A891               mov [ebp-58], 91

:004111A0 C645A9C3               mov [ebp-57], C3

:004111A4 C645AA24               mov [ebp-56], 24

:004111A8 C645AB0F               mov [ebp-55], 0F

:004111AC C645ACB7               mov [ebp-54], B7

:004111B0 C645ADC2               mov [ebp-53], C2

:004111B4 C645AEF8               mov [ebp-52], F8

:004111B8 C645AFE3               mov [ebp-51], E3

:004111BC C645B0B6               mov [ebp-50], B6

:004111C0 C645B154               mov [ebp-4F], 54

:004111C4 C645B24C               mov [ebp-4E], 4C

:004111C8 C645B335               mov [ebp-4D], 35

:004111CC C645B454               mov [ebp-4C], 54

:004111D0 C645B5D7               mov [ebp-4B], D7

:004111D4 C645B6C9               mov [ebp-4A], C9

:004111D8 C645B749               mov [ebp-49], 49

:004111DC C645B828               mov [ebp-48], 28

:004111E0 C645B9A3               mov [ebp-47], A3

:004111E4 C645BA85               mov [ebp-46], 85

:004111E8 C645BB11               mov [ebp-45], 11

:004111EC C645BC0B               mov [ebp-44], 0B

:004111F0 C645BD2C               mov [ebp-43], 2C

:004111F4 C645BE68               mov [ebp-42], 68

:004111F8 C645BFFB               mov [ebp-41], FB

:004111FC C645C0EE               mov [ebp-40], EE

:00411200 C645C17D               mov [ebp-3F], 7D

:00411204 C645C2F6               mov [ebp-3E], F6

:00411208 C645C36C               mov [ebp-3D], 6C

:0041120C C645C4E3               mov [ebp-3C], E3

:00411210 C645C59C               mov [ebp-3B], 9C

:00411214 C645C62D               mov [ebp-3A], 2D

:00411218 C645C7E4               mov [ebp-39], E4

:0041121C C645C872               mov [ebp-38], 72

:00411220 C645C9C3               mov [ebp-37], C3

:00411224 C645CABB               mov [ebp-36], BB

:00411228 C645CB85               mov [ebp-35], 85

:0041122C C645CC1A               mov [ebp-34], 1A

:00411230 C645CD12               mov [ebp-33], 12

:00411234 C645CE3C               mov [ebp-32], 3C

:00411238 C645CF32               mov [ebp-31], 32

:0041123C 8B7508                 mov esi, dword ptr [ebp+08]

:0041123F C645D0E3               mov [ebp-30], E3

:00411243 56                     push esi

:00411244 C645D16B               mov [ebp-2F], 6B

:00411248 C645D24F               mov [ebp-2E], 4F

:0041124C C645D34D               mov [ebp-2D], 4D

:00411250 C645D4F4               mov [ebp-2C], F4

:00411254 C645D5A9               mov [ebp-2B], A9

:00411258 C645D624               mov [ebp-2A], 24

:0041125C C645D7C8               mov [ebp-29], C8

:00411260 C645D8FA               mov [ebp-28], FA

:00411264 C645D978               mov [ebp-27], 78

:00411268 C645DAAD               mov [ebp-26], AD

:0041126C C645DB23               mov [ebp-25], 23

:00411270 C645DCA1               mov [ebp-24], A1

:00411274 C645DDE4               mov [ebp-23], E4

:00411278 C645DE6D               mov [ebp-22], 6D

:0041127C C645DF9A               mov [ebp-21], 9A

:00411280 C645E004               mov [ebp-20], 04

:00411284 C645E1CE               mov [ebp-1F], CE

:00411288 C645E22B               mov [ebp-1E], 2B

:0041128C C645E3C5               mov [ebp-1D], C5

:00411290 C645E4B6               mov [ebp-1C], B6

:00411294 C645E5C5               mov [ebp-1B], C5
```



:00411298 C645E6EF   mov `[ebp-1A]`, EF   

:0041129C C645E793   mov `[ebp-19]`, 93   

:004112A0 C645E85C   mov `[ebp-18]`, 5C   

:004112A4 C645E9A8   mov `[ebp-17]`, A8   

:004112A8 C645EA85   mov `[ebp-16]`, 85   

:004112AC C645EB2B   mov `[ebp-15]`, 2B   

:004112B0 C645EC41   mov `[ebp-14]`, 41   

:004112B4 C645ED37   mov `[ebp-13]`, 37   

:004112B8 C645EE72   mov `[ebp-12]`, 72   

:004112BC C645EFFA   mov `[ebp-11]`, FA   

:004112C0 C645F057   mov `[ebp-10]`, 57   

:004112C4 C645F145   mov `[ebp-0F]`, 45   

:004112C8 C645F241   mov `[ebp-0E]`, 41   

:004112CC C645F3A1   mov `[ebp-0D]`, A1   

:004112D0 C645F420   mov `[ebp-0C]`, 20   

:004112D4 C645F54F   mov `[ebp-0B]`, 4F   

:004112D8 C645F680   mov `[ebp-0A]`, 80   

:004112DC C645F7B3   mov `[ebp-09]`, B3   

:004112E0 C645F8D5   mov `[ebp-08]`, D5   

:004112E4 C645F923   mov `[ebp-07]`, 23   

:004112E8 C645FA02   mov `[ebp-06]`, 02   

:004112EC C645FB64   mov `[ebp-05]`, 64   

:004112F0 C645FC3F   mov `[ebp-04]`, 3F   

:004112F4 C645FD6C   mov `[ebp-03]`, 6C   

:004112F8 C645FEF1   mov `[ebp-02]`, F1   

:004112FC C645FF0F   mov `[ebp-01]`, 0F   

:00411300 E8DBCA0000   call 0041DDE0   

:00411305 85C0   test `eax`, `eax`   

:00411307 59   pop `ecx`   

:00411308 7E48   jle 00411352   

:0041130A 57   push `edi`   

:0041130B 8D7D80   lea `edi`, dword ptr `[ebp-80]`   

:0041130E 8BCE   mov `ecx`, `esi`   

:00411310 2BFE   sub `edi`, `esi`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0041134F(C)   

|   

:00411312 8A11   mov `dl`, byte ptr `[ecx]`<-依次取注册名   

:00411314 84D2   test `dl`, `dl`   

:00411316 7D04   jge 0041131C   

:00411318 F6DA   neg `dl`   

:0041131A 8811   mov byte ptr `[ecx]`, `dl`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00411316(C)   

|   

:0041131C 8A140F   mov `dl`, byte ptr `[edi+ecx]`<-<-依次取 `48,ee,76.`   

:0041131F 3211   xor `dl`, byte ptr `[ecx]`<-与注册名的 ASC 码 XOR   

:00411321 80FA30   cmp `dl`, 30<-比较   

:00411324 8811   mov byte ptr `[ecx]`, `dl`   

:00411326 7D0F   jge 00411337 <-大于或等于就跳   

:00411328 0FBED2   movsx `edx`, `dl`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00411335(U)   

|   

:0041132B 83FA30   cmp `edx`, 00000030<-比较   

:0041132E 7D07   jge 00411337<-大于或等于就跳   

:00411330 83C203   add `edx`, 00000003<-加 3   

:00411333 8811   mov byte ptr `[ecx]`, `dl`   

:00411335 EBF4   jmp 0041132B<-循环   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:00411326(C), :0041132E(C)   

|   

:00411337 8A11   mov `dl`, byte ptr `[ecx]`   

:00411339 80FA39   cmp `dl`, 39<-比较   

:0041133C 7E0F   jle 0041134D<-小于或等于就跳   

:0041133E 0FBED2   movsx `edx`, `dl`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0041134B(U)   

|   

:00411341 83FA39   cmp `edx`, 00000039<-比较   

:00411344 7E07   jle 0041134D<-小于或等于就跳   

:00411346 83EA03   sub `edx`, 00000003<-减 3   

:00411349 8811   mov byte ptr `[ecx]`, `dl`   

:0041134B EBF4   jmp 00411341<-循环   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:0041133C(C), :00411344(C)   

|   

:0041134D 41   inc `ecx`   

:0041134E 48   dec `eax`<-`EAX` 减一（`EAX` 注册名长度）   

:0041134F 75C1   jne 00411312<-循环（到注册名取完）   

:00411351 5F   pop `edi`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00411308(C)   

|   

:00411352 FF750C   push `[ebp+0C]`   

:00411355 56   push `esi`   <-真正的注册码(D `ESI`)   

:00411356 E815C10000   call 0041D470   

:0041135B F7D8   neg `eax`   

:0041135D 59   pop `ecx`   

:0041135E 1BC0   sbb `eax`, `eax`   

:00411360 59   pop `ecx`   

:00411361 40   inc `eax`   

:00411362 5E   pop `esi`   

:00411363 C9   leave   

:00411364 C20800   ret 0008   

总结：依次取注册名于 `48,ee,76.` 序列进行 XOR，其结果通过加 3 或减 3，使之界于 30~39 之间（0~9）   

`NAME`: `jieao[CCG]`   

`CODE`: `1217220771`   

```cpp
//---------------------------------------------------------------------------    
#include <vcl.h>   

#pragma hdrstop   

#include "main.h"   

//---------------------------------------------------------------------------    
#pragma package(smart_init)   

#pragma resource "*.dfm"   

TfrmMain *frmMain;   

//---------------------------------------------------------------------------    
__fastcall TfrmMain::TfrmMain(TComponent* Owner)   
    : TForm(Owner)   
{   
}   

//---------------------------------------------------------------------------    
void __fastcall TfrmMain::sbExitClick(TObject *Sender)   
{   
    Close();   
}   

//---------------------------------------------------------------------------    
void __fastcall TfrmMain::Image1DblClick(TObject *Sender)   
{   
    Application->MessageBoxA("Http://******", "Welcome", NULL);   
}   

//---------------------------------------------------------------------------    
void __fastcall TfrmMain::sbCopyClick(TObject *Sender)   
{   
    if(edtCode->Text.IsEmpty())   
        return;   
    // copy to clipboard   
    Clipboard()->SetTextBuf(edtCode->Text.c_str());   
}   

//---------------------------------------------------------------------------    
void __fastcall TfrmMain::sbComputeClick(TObject *Sender)   
{   
    char cMixer[31]={0x48,0xEE,0x76,0x1D,0x67,0x69,0xA1,0x1B   
                    ,0x7A,0x8C,0x47,0xF8,0x54,0x95,0x97,0x5F   
                    ,0x78,0xD9,0xDA,0x6C,0x59,0xD7,0x6B,0x35   
                    ,0xC5,0x77,0x85,0x18,0x2A,0x0E,0x52};   
    char *pName;       //point to edtName->Text    
    int i, nLen;       //Counter, Length of Name    
    nLen=edtName->Text.Length();   
    // check if name leagle   
    if(nLen==0)   
        return;   
    if(nLen<4)   
    {   
        Application->MessageBoxA("ah!!Name should be at least 4 characters.", "Dennied", NULL);   
        return;   
    }   
    if(nLen>31)   
    {   
        Application->MessageBoxA("Don't u think your Name is too long?", "Dennied", NULL);   
        return;   
    }   
    pName=edtName->Text.c_str();   
    // generate RegCode here   
    for(i=0; i<nLen; i++)   
    {   
        cMixer[i]^=*(pName+i);   
        while(cMixer[i]<0x30)   
            cMixer[i]+=3;   
        while(cMixer[i]>0x39)   
            cMixer[i]-=3;   
    }   
    cMixer[i]='\0';   
    // show it now   
    edtCode->Text=cMixer;   
    // select name for another input   
    edtName->SelectAll();   
}   

//--------------------------------------------------------------------------- 
```

4dd466d0e1e0cc123afbe504da2462ca
