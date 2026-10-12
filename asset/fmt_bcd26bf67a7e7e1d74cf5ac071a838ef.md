
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
