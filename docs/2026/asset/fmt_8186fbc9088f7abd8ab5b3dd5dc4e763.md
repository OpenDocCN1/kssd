
``` 
        char D3D9X_DLL_NAME[1024];

        DWORD OrgD3DXSaveSurfaceToFileInMemory_adr;

        void ScrrenXY(LPDIRECT3DDEVICE9 dev,char* FilePath,int x1,int y1,int x2,int y2);

        bool IsCur();

        int Test1(int DicIndex,char* COLOR,char* StrNameS,int X1,int Y1,int X2,int Y2,CAtlString& RetStr);

protected:

private:

};

void  Hook_dx();

void  Hook_dx1();

bool  IsCur();

.h

#include "HOOK_DX.h"

#include <d3d9.h>

#include <d3dx9tex.h>

#include <fstream>

#include "MyEat.h"

#ifndef MEM_H

#define MEM_H

#include "MemLoadDll.h"

#endif

#include "MyInlineHook.h"

#include "hook_d3d3x_.h"

#include <highgui.h>

#include "CvvImage.h"

#include "Find.h"

#include <dinput.h>

#include "User32_Hook.h"

#include "MyBmp.h"

#include "HextoBinary.h"

#include "DNF_CODE.h"

#include <atlstr.h>

#include <Mmsystem.h>

#include <winbase.h>

#include "GameFun.h"

#include "HookSystemCall.h"

#include "Common.h"

#include <Shlwapi.h>

//#include "TestLib.h"

#include "NTFUNCTION.h"

MyGame* Gamefu=NULL;

//#include <dinput.h>

CMemLoadDll* RELOAD2=NULL;

char* BUFER;

typedef  void  ( __stdcall* _MyDllInit)();

typedef  void ( __stdcall* _My_CALL)(int nRoleAdd,int shanghai,int nCode,int MaxNum);

typedef void ( __stdcall* _My3s)(ULONG sssBaseAddr,int sssVar);

_MyDllInit  MyDllInit=NULL;

_My_CALL    My_CALL =NULL;

_My3s My3s1=NULL;

DWORD JIA_D3D9_82=0;

int cout_test=0;

LPDIRECT3D9             g_pD3D = NULL; // Used to create the D3DDevice

LPDIRECT3DDEVICE9       g_pd3dDevice = NULL; // Our rendering device

bool isFree=FALSE;

DX_FUNC Game_dx;

DATA_TO_DX my_dx_data;

bool Is_XiWU=FALSE;

bool Is_XiGuai=FALSE;

bool Is_END_XiGuai=TRUE;

LRESULT CALLBACK WindowProc( HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam );

void HookPrsent();

void HookDrawPrimitive();

WNDPROC OldWindowProc=0;

bool is_hook_user=FALSE;

time_t Begintime=0;

time_t EndTime;

BOOL Is_In=FALSE;

long   __stdcall   callback(_EXCEPTION_POINTERS*   excp)

{

        OutputDebugStringA("ERRO  GAME  23");

        Game_dx.DATA2->IsGetBeibao=FALSE;

        Is_In=FALSE;

        //        return   EXCEPTION_EXECUTE_HANDLER;//悄悄退出

        return EXCEPTION_CONTINUE_EXECUTION;//继续执行

}

BOOL Type_xiguai=FALSE;

int  Type_xiguai_4=0;

void SetGame()

{

         SetUnhandledExceptionFilter(callback);

         try{

        if (Is_In==TRUE)

        {

                return;

        }

        Is_In=TRUE;

        if (is_hook_user==FALSE)

        {
        }

        if (Gamefu==NULL)

        {
                Gamefu=new MyGame;
        }

        if (Game_dx.DATA2!=NULL)

        {
                //设置绑定成功
                PDATA_TO_DX DATA2=Game_dx.DATA2;

                if (DATA2->Is_Bind1_ok==FALSE)

                {
                        DATA2->Is_Bind1_ok=TRUE;

                        DATA2->Is_Bind2_ok=TRUE;

                        DATA2->Is_Bind_KEY1_ok=TRUE;
                }

                //////////////////////////////////////////////////////////////////////////

                 if    (Game_dx.DATA2->IsGetName_Lv_Pilao==TRUE)//获取疲劳负重 名字

                {
                        Gamefu->GetRenwu_Lv_Name(RenwuJizhi, Game_dx.DATA2->Name_Lv);

                        DWORD CurPilao=Gamefu->GetPiLao();

                        Game_dx.DATA2->Name_Lv.Pilao=CurPilao;

                        DWORD AllFuzhong=Gamefu->GetAllFuZhong();

                        Game_dx.DATA2->Name_Lv.MaxFuzhong=AllFuzhong;

                        DWORD  CurFuzhong=Gamefu->GetFuZhong();

                        Game_dx.DATA2->Name_Lv.CurFuzhong=CurFuzhong;

                        Game_dx.DATA2->IsGetName_Lv_Pilao=FALSE;

                        Game_dx.SendTo_Game();
                }

                //////////////////////////////////////////////////////////////////////////

                if (DATA2->IsGetJinbi==TRUE)//获取金币

                {
                        DATA2->jinbi= Gamefu->GetJinBi(_BeiBaoJizhi);

                        DATA2->IsGetJinbi=FALSE;
                }

                //////////////////////////////////////////////////////////////////////////

                //背包

                if (Game_dx.DATA2->IsGetBeibao==TRUE)

                {
                        OutputDebugStringA("get beibao  gm");

                        Gamefu->GetBeiBaoToArray(_BeiBaoJizhi,&Game_dx.DATA2->GameBeiBao);

                        Game_dx.DATA2->IsGetBeibao=FALSE;
                }

                //仓库

                if (Game_dx.DATA2->Is_GetCangku==TRUE)

                {
                        OutputDebugStringA("get beibao  gm");

                        Gamefu->GetCangKuToArray(_BeiBaoJizhi,&Game_dx.DATA2->GameCanuku);

                        Game_dx.DATA2->Is_GetCangku=FALSE;
                }

                //装备

                if (Game_dx.DATA2->IsGetCurZhuangBei==TRUE)

                {
                        Gamefu->GetCurZhuangBeiToArray(&Game_dx.DATA2->curzb);

                        DATA2->IsGetCurZhuangBei=FALSE;
                }

                //结束修理卖屋状态

                if (DATA2->IsEndXiuLiMaiWu==TRUE)

                {
                        OutputDebugStringA("gm endxiulimaiwu");

                        Gamefu->ChuShou(0);

                        Game_dx.DATA2->IsChuShou=FALSE;

                        Game_dx.DATA2->IsXiuli=FALSE;

                        DATA2->IsEndXiuLiMaiWu=FALSE;
                }

                //开启修理卖屋

                if (DATA2->IsBeginXiuliMaiWu==TRUE)

                {
                        DATA2->IsBeginXiuliMaiWu=FALSE;

                        Gamefu->ChuShou(DATA2->XiuliMaiwuWat);//开启出售或修理

                        if (DATA2->XiuliMaiwuWat==5)

                        {
                                Game_dx.DATA2->IsChuShou=TRUE;
                        }

                        if (DATA2->XiuliMaiwuWat==6)

                        {
                                Game_dx.DATA2->IsXiuli=TRUE;
                        }
                }

                //获取怪物数目

                if (DATA2->IsGetGuaiwuShuliang==TRUE)

                {
                        Game_dx.DATA2->GuaiwuShuliang=Gamefu->Get_GuaiWuShuLiang(0);

                        DATA2->IsGetGuaiwuShuliang=FALSE;
                }

                //获取地上物品数目

                if (DATA2->IsGetWupinShuLiang==TRUE)

                {
                        Game_dx.DATA2->WUPUN_WULIANG=Gamefu->Get_WuPinShuLiang(0);

                        DATA2->IsGetWupinShuLiang=FALSE;
                }

                //修改力量

                if (DATA2->Isadd_Liliang==TRUE)

                {
                                Gamefu->SetJiami_Hot_BeiGong_Liliang(Game_dx.DATA2->liliang);

                if (Game_dx.DATA2->Tili!=0)

                        {
                                Gamefu->Add_TiLi(Game_dx.DATA2->Tili);

                                Gamefu->Add_JingShen(Game_dx.DATA2->Tili);

                                //Game_dx.DATA2->Tili=0;
                        }

                /*                if (Game_dx.DATA2->Lv!=0)

                        {
                                Gamefu->Add_Lv(Game_dx.DATA2->Lv);

                                Game_dx.DATA2->Lv=0;
                        }*/

                        Game_dx.DATA2->Isadd_Liliang=FALSE;
                }

         //SY

                if (DATA2->IsSY==TRUE)

                {
                        Gamefu->SY(DATA2->sdjz,DATA2->fx,DATA2->CallBase);

                        DATA2->IsSY=FALSE;
                }

                if (DATA2->IsShunyibefor==TRUE)

                {
                        Gamefu->Xi_Men_Befor(DATA2->Shunyi_Fangxiang);

                        DATA2->IsShunyibefor=FALSE;
                }

                //瞬移后
``` 
