
# 浪漫情书 v3.0

> 原文：`www.pediy.com/kssd/pediy04/kanxue065.htm`

主程序文件：`main.cpp`

```cpp
//---------------------------------------------------------------------------
#include <vcl.h>
#pragma hdrstop
#include "main.h"
//---------------------------------------------------------------------------
#pragma package(smart_init)
#pragma resource "*.dfm"

TMainForm *MainForm;

//---------------------------------------------------------------------------
__fastcall TMainForm::TMainForm(TComponent* Owner)
    : TForm(Owner)
{
}
//---------------------------------------------------------------------------

void __fastcall TMainForm::Exit_BtnClick(TObject *Sender)
{
Application->Terminate();
}
//---------------------------------------------------------------------------

void __fastcall TMainForm::ID_EditKeyPress(TObject *Sender, char &Key)
{
if(Key<'0' | Key>'9')
 {
 if(Key=='\b')
   {return;}
 Application->MessageBoxA("请输入一个数字，否则你将得不到正确的结果！","错误",MB_OK);
 Key=NULL;
 return;
 }
}
//---------------------------------------------------------------------------

void __fastcall TMainForm::ID_EditChange(TObject *Sender)
{
AnsiString REG_Temp;
ULONG NUM_Temp;
DWORD ASC_Temp;

NUM_Temp=0;

if(User_Edit->Text.Length()<6)
 {
 Key_Edit->Text="用户名的长度不能少于 6 位";
 return;
 }
if(ID_Edit->Text=="")
 {
 Key_Edit->Text="待注册码不能为空";
 return;
 }

REG_Temp=ID_Edit->Text+User_Edit->Text;

for(int i=1;i<=REG_Temp.Length();i++)
 {
   ASC_Temp=REG_Temp[i]*(i+3);
   NUM_Temp=NUM_Temp+ASC_Temp;
 }

NUM_Temp=NUM_Temp*0x43C9;
NUM_Temp=NUM_Temp+0x595EFBB;

Key_Edit->Text=AnsiString(NUM_Temp);   ;这里就是注册码了
}
//---------------------------------------------------------------------------

void __fastcall TMainForm::Web_LabelClick(TObject *Sender)
{
ShellExecute(Handle,"open","http://WWW.DOWNCRACK.COM/",NULL,NULL,SW_MAXIMIZE);
}
//---------------------------------------------------------------------------
```

头文件：`main.h`

```cpp
//---------------------------------------------------------------------------
#ifndef mainH
#define mainH
//---------------------------------------------------------------------------
#include <Classes.hpp>
#include <Controls.hpp>
#include <StdCtrls.hpp>
#include <Forms.hpp>
#include <Buttons.hpp>
//---------------------------------------------------------------------------

class TMainForm : public TForm
{
__published:   // IDE-managed Components
    TLabel *ID_Label;
    TLabel *Reg_Label;
    TEdit *ID_Edit;
    TEdit *Key_Edit;
    TBitBtn *Exit_Btn;
    TLabel *Web_Label;
    TLabel *Label2;
    TLabel *Label3;
    TLabel *Label1;
    TLabel *User_Label;
    TEdit *User_Edit;
    void __fastcall Exit_BtnClick(TObject *Sender);
    void __fastcall ID_EditKeyPress(TObject *Sender, char &Key);
    void __fastcall ID_EditChange(TObject *Sender);
    void __fastcall Web_LabelClick(TObject *Sender);

private:   // User declarations
public:     // User declarations
    __fastcall TMainForm(TComponent* Owner);
};
//---------------------------------------------------------------------------
extern PACKAGE TMainForm *MainForm;
//---------------------------------------------------------------------------
#endif
```

```c
#include "stdio.h"
main()
{
unsigned char string[81];
char sn[10];
long d=0;
int i,c;
end:printf("******LoveLetter 1.0******\nThis keygen is made by Pretender\nPlease input your name  : ");
gets(string);
d=strlen(string);
if(d<6) {printf("Sorry,your username must longer than 5 letters!\n"); goto end;}
printf("Please input your ID     : ");
gets(sn);
strcat(string,sn);
d=0;
for(i=0;(c=string[i])!='\0';i++)
{c*=(i+2);d+=c;}
d*=0x3505;d+=0x3686c61;
/*printf("Your name is            : ");
puts(string);*/
printf("Your Register code is  : %ld\n",d);
printf(" *******************************      ---    ---    ---\n *Welcome to WWW.CRACKNOW.COM*      /      /      / --\n *******************************      ---    ---    --/\n");}
```

`4dd466d0e1e0cc123afbe504da2462ca`
