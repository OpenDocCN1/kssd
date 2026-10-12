
# Readbook

> 原文：[`www.pediy.com/kssd/pediy04/kanxue396.htm`](https://www.pediy.com/kssd/pediy04/kanxue396.htm)

```c
#include <vcl.h>
#pragma hdrstop
#include "readbook_RC.h"

//---------------------------------------------------------------------------
#pragma package(smart_init)
#pragma resource "*.dfm"

Tform1 *form1;

//---------------------------------------------------------------------------
__fastcall Tform1::Tform1(TComponent* Owner)
    : Tform(Owner)
{
}

//---------------------------------------------------------------------------
void __fastcall Tform1::calcClick(TObject *Sender)
{
   char name[255],code[255] = "";
   int i;
   long values = 0;
   for(i = 0; i < 255; i++)
      name *= '\0x0';
   if(edtName->Text.IsEmpty()) {
      edtCode->Text = "";
      return;
   }
   strcpy(name, edtName->Text.c_str());
   _asm
   {
      lea esi, name
      mov eax, dword ptr [esi]
      mov esi, eax
      mov ecx, 0x60606040
      imul esi, esi, 0x7531
      imul ecx, ecx, 0x7531
      sub esi, ecx
      mov eax, esi
      mov values, eax
   }
   wsprintf(code, "%lu", values);
   edtCode->Text = code;
}

//-------------------------------------**\---------------------------------------
```

`4dd466d0e1e0cc123afbe504da2462ca`
