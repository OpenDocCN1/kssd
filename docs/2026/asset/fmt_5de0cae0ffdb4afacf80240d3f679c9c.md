
```cpp
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        jmp OldPresend_Add5

        }

}

clock_t Begin2=0;

clock_t End2=0;

HRESULT __stdcall MyuPresent (LPDIRECT3DDEVICE9 pdev, RECT* pSourceRect,CONST RECT* pDestRect,HWND hDestWindowOverride,CONST RGNDATA* pDirtyRegion)
{
        End2= clock();
        if (End2-Begin2>100)
        {
                SetGame();
        }
        return Temp_Present(pdev,pSourceRect,pDestRect,hDestWindowOverride,pDirtyRegion);
}

void HookPrsent()
{
        BYTE ReSet[5]={0x8b,0xff,0x55,0x8b,0xec};
        HMODULE HD3d9=::GetModuleHandleA("d3d9.dll");
        DWORD OldProtect,JmpCode;
        DWORD DX_17=Game_dx.DATA2->D3D_17+(DWORD)GetModuleHandleA("D3D9.DLL");
        VirtualProtect((LPVOID)DX_17,5,PAGE_EXECUTE_READWRITE,&OldProtect);
        VirtualProtect(Temp_Present,5,PAGE_EXECUTE_READWRITE,&OldProtect);
        char out11[256]={0};
        sprintf(out11,"dx_17=  %x",DX_17);
        OutputDebugStringA(out11);
        memcpy((LPVOID)DX_17,ReSet,5);
        OldPresend_Add5=DX_17+5;
        memcpy(Temp_Present ,(LPVOID)DX_17,5);
        *((BYTE*)DX_17)=0xe9;
        JmpCode=(DWORD)MyuPresent-DX_17-5;
        *((DWORD*)(DX_17+1))=JmpCode;
}

ULONG_PTR SetTexture_add5;
__declspec(naked)  HRESULT  Temp_SetTexture(LPDIRECT3DDEVICE9 pdev, DWORD Stage,IDirect3DBaseTexture9* pTexture) //65
{
        __asm{
                nop
                nop
                nop
                nop
                nop
                nop
                nop
                nop
                nop
                nop
                jmp SetTexture_add5
        }
}

char writea[256]={0};

HRESULT __stdcall  MySetTexture(LPDIRECT3DDEVICE9 pdev, DWORD Stage,IDirect3DBaseTexture9* pTexture) //65
{
        if (pTexture!=NULL)
        {
                memset(writea,0,256);
                sprintf(writea,"Stage %d pTexture %d ",Stage,pTexture);
                OutputDebugStringA(writea);
                ::WritePrivateProfileStringA("test",writea,"1","C:\\TT.TXT");
                return  S_FALSE;
                if ((DWORD)472450720==(DWORD)pTexture)
                {
                                return S_OK;
                }
        }
/*
        IDirect3DVertexBuffer9* Pstreamdata=NULL;
        UINT ioffsetInbytes,iStride;
        if (pdev->GetStreamSource(0,&Pstreamdata,&ioffsetInbytes,&iStride)==D3D_OK)
        {
                Pstreamdata->Release();
                if (Game_dx.DATA2!=NULL)
                {
                        if (   ( PrimitiveCount>(Game_dx.DATA2->PrintTest-100 )|| PrimitiveCount<(Game_dx.DATA2->PrintTest+100 ) )  && iStride==28 /* && (type!=Game_dx.DATA2->PrintTest)*/ )
                        {
                        }
}
                IDirect3DBaseTexture9*  T1;        */
                                return Temp_SetTexture(pdev,Stage,pTexture);
        return S_OK;
}

void HookDrawPrimitive()
{
/*
        BYTE ReSet[5]={0x8b,0xff,0x55,0x8b,0xec};
        HMODULE HD3d9=::GetModuleHandleA("d3d9.dll");
        DWORD OldProtect,JmpCode;
        DWORD DX_65=Game_dx.DATA2->D3D_65+(DWORD)GetModuleHandleA("D3D9.DLL");
        VirtualProtect((LPVOID)DX_65,5,PAGE_EXECUTE_READWRITE,&OldProtect);
        VirtualProtect(Temp_SetTexture,5,PAGE_EXECUTE_READWRITE,&OldProtect);
        char out11[256]={0};
        sprintf(out11,"dx_81=  %x",DX_65);
        OutputDebugStringA(out11);
        memcpy((LPVOID)DX_65,ReSet,5);
        SetTexture_add5=DX_65+5;
        memcpy(Temp_SetTexture ,(LPVOID)DX_65,5);
        *((BYTE*)DX_65)=0xe9;
        JmpCode=(DWORD)MySetTexture-DX_65-5;
        *((DWORD*)(DX_65+1))=JmpCode;*/
}

BOOL isha_hookWM=FALSE;

time_t  time1=0;

time_t time2;

BOOL ISGOGO=FALSE;

VOID  MYTHREAD()
{
        while(1)
        {
        /*        if (is_hook_user==FALSE )
                {
                        if (Game_dx.IsCur()==TRUE)
                        {
                                Hook_dx();
                                HookPrsent();
                                is_hook_user=TRUE;
                        }
                }*/
                OutputDebugStringA("11");
                if (isFree==FALSE)
                {
                        HMODULE aa=GetModuleHandleA("爱冰的沫.DLL");
                        ::FreeLibrary(GetModuleHandleA("爱冰的沫.DLL"));
                        isFree=TRUE;
                }
                SetGame();
        }
}

#include <boost/archive/BINARY_iarchive.hpp>
#include <boost/archive/binary_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/XML_iarchive.hpp>
#include <boost/archive/XML_oarchive.hpp>
#include <boost/serialization/base_object.hpp>
#include <iostream>
#include <fstream>

typedef struct _sYY{
        int a;
        int b;
        int c;
}sYY1,*PSS;

sYY1 sYY;

void  Hook_dx1()
{
        OutputDebugStringA("ininininininin________________________________________");
        //OutputDebugStringA("进入");
        /*if (isFree==FALSE)
        {
                HMODULE aa=GetModuleHandleA("爱冰的沫.DLL");
                ::FreeLibrary(GetModuleHandleA("爱冰的沫.DLL"));
                isFree=TRUE;
        }*/
                if (Ishook==FALSE)
                {
                                Hook32();
                                char filename[256];
                                ::GetModuleFileNameA(NULL,filename,256);
                        //        OutputDebugStringA(filename);
                                if (strstr(filename,"DNF.exe")!=NULL  || strstr(filename,"dnf.exe")!=NULL )
                                {
HookInputTime();
        HookOle32();
/*
        FILE*stream;
        stream=fopen("C:\\showBase","r");
        int a,b,c;
        fscanf(stream,"%d,%d,%d",&a,&b,&c);
        fclose(stream);*/
                                }
/*
                                int allSafe[3];
                        using namespace  std;
                        std::ifstream file("C:\\archive");
                        boost::archive::text_iarchive   ia(file);
                        ia>>allSafe;
                        if(allSafe[0]==1)
                        {
                                HookInputTime();
                        }
                        if (allSafe[2]==1)
                        {
                                HookOle32();
                        }*/
                //
                //
                        Ishook=TRUE;
                }
        if (ISGOGO==TRUE)
        {
                return ;
        }
        //SetMyTop();
        ISGOGO=TRUE;
        return ;
        OldWindowProc=(WNDPROC)::SetWindowLongA(::FindWindowA(NULL,"地下城与勇士"),GWL_WNDPROC,(LONG)::WindowProc);
        //OutputDebugStringA("进入");
        return ;
        //GetWindowThreadProcessId()
                extern BOOL ISOK;
//OutputDebugStringA("Hook_dx");
        if (ISOK==FALSE)
        {
                if (!Game_dx.IsCur())
                {
                        //OutputDebugStringA("!IsCur");
                        IsCur_Bool=FALSE;
                        return ;
                }
                        ISOK=TRUE;
                //OutputDebugStringA("IsCur");
                IsCur_Bool=TRUE;
                Game_dx.SendTo_Game();
                //////////////////////////////////////////////////////////////////////////
```
