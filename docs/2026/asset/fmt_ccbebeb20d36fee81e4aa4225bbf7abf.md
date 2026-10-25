
```
8B4868                  mov ecx,

dword ptr [eax+68]

 :0044B320 51

          push ecx

 :0044B321 E818DA0100

          call 00468D3E　　　　<<------注册失败对话框

:0044B326 83C414

add esp, 00000014
```

 ＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

此软件的注册码算法很简单，它将用户名与公司名进行相同的运算后得出真正的注册码（有两个，不知有什么区别），其中用到了五个数字常量：｛`0xEB`, `0x7B`, `0x11`, `0x2A`｝和`0x186A0`，还有一个初始值为`0x3AC7`的变量。

 ＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

附：注册机源程序（偷了点懒的^\_^）

```
#include<iostream.h>

#include<string.h>

#include<math.h>

#include<stdlib.h>

void main()

{

    int N[4]={-21,123,17,42},temp1=0,temp2=0,Name,Company,RegistCode=15047,count1,count2,LengthOftempName,LengthOftempCompany;

    char tempName[80],tempCompany[80];

    cout<<"

               KeyGen for Acoustica V2.21(Made

by alpha)"<<endl;

    cout<<"Your Name:";

   while(cin.getline(tempName,80))

    {

       LengthOftempName=strlen(tempName);

       for(count1=0;count1<LengthOftempName;count1++)

        {

 if(temp1==4)

 temp1=0;

            Name=tempName[count1];

//            cout<<Name[count1]<<endl;

            Name\*=N[temp1];

//            cout<<N[temp1]<<endl;

            RegistCode+=Name;

            temp1++;

     }

        break;

    }

    cout<<endl;

 cout<<"Your Company:";

    while(cin.getline(tempCompany,80))

    {

        LengthOftempCompany=strlen(tempCompany);

        for(count2=0;count2<LengthOftempCompany;count2++)

        {

 if(temp2==4)

 temp2=0;

            Company=tempCompany[count2];

            Company\*=N[temp2];

            RegistCode+=Company;

            temp2++;

     }

        break;

    }

    cout<<endl;

 cout<<"Your Registration Code is:"<<"AC210-"<<RegistCode<<endl;

    cout<<"

         or:"<<"AC220-"<<RegistCode<<endl;

    cout<<"\nPress any key to terminate...";

 while(cin.get())

        exit(0);

}
```

 ＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

源程序在VC＋＋6.0下编译通过

CrAcKeD BY alphakk(alpha)
