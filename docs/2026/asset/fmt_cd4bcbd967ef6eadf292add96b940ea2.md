
```cpp
        LONG BindShow=-1;

        LONG BindKey=-1;

        if (strShow.Find("普通图色")!=-1)

        {

                //AfxMessageBox("普通");

                BindShow=0;

        }

        if (strShow.Find("GDI后台图色")!=-1)

        {

                //AfxMessageBox("GDI后台");

                BindShow=1;

        }

        if (strShow.Find("DX后台图色")!=-1)

        {

                //AfxMessageBox("DX后台");

                BindShow=2;

        }

        //////////////////////////////////////////////////////////////////////////

        strShow.Empty();

        strShow=Key\_Bord;

        if (strShow.Find("DX1键盘")!=-1)

        {

                //AfxMessageBox("DX1");

                BindKey=1;

        }

        if (strShow.Find("普通后台键盘")!=-1)

        {

                //AfxMessageBox("普通后台键盘");

                BindKey=0;

        }

        if (        momo1[Index].BindWindow((HWND)Hwnd,BindShow,BindKey)==0)

        {

                return 0;

        }

        return 1;

}

LONG MOMO::LoadDic(LONG Index, char\* DicPath, LONG DicIndex)

{

if (strlen(DicPath)<1)

{

        return -1;

}

if (DicIndex>1  ||  DicIndex<0)

{

        return -2;

}

return 0;

}

LONG MOMO::ShowDic(LONG Index, LONG DicIndex, char\* StrName)//把字库指定文字显示为图片

{

        AFX\_MANAGE\_STATE(AfxGetStaticModuleState());

        if (DicIndex>1 || DicIndex <0)

        {

                AfxMessageBox("仅支持2个字库 0号以及1号");

                return 0;

        }

        return 1;

}

BOOL MOMO::ScreenShot\_DxForce(LPDIRECT3DDEVICE9 lpDevice, HWND hWnd, char\* fileName)

{

        HRESULT hr;

        return hr ;

}

IplImage\* MOMO::GetFroceDxPic\_Dnf(LPDIRECT3DDEVICE9 lpDevice,HWND hWnd)

{

        return NULL ;

}

bool GetDnf\_RECT(char\* WindowName,LPRECT RetXY)

{

        ::GetWindowRect(::FindWindowA(NULL,WindowName),RetXY);

        return TRUE;

}

void MyMOMO::GetWuPinArray()

{

        data->IsGetBeibao=TRUE;

        while(data->IsGetBeibao==TRUE)

        {

                OutputDebugStringA("\r\nGetWuPinArray");

                Sleep(100);

        }

}

void MyMOMO::GetCurZhungbeiArray()

{

        ::SendMessageA(Thehwnd,WM\_USER+75741,1,1);

}

DLL:

#include <windows.h>

#include "MyOcr.h"

#include <d3d9.h>

#include <d3dx9tex.h>

#include "GameFun.h"

//

typedef struct \_FindPicdx

{ char FindPicDx\_Path[1024];

        int FindPic\_simmin;

        int FindPic\_simmax;

        int FindPic\_x1;

        int FindPic\_x2;

        int FindPic\_y1;

        int FindPic\_y2;

        int FindPic\_RetX;

        int FindPic\_RetY;

        int FindPic\_Retsim;

}FindPicdx,\*PFindPicdx;

typedef struct \_ImeMessage

{

        bool IsSendIme;

        char SendStr[102400];

        LONG SendImeLenth;

}MyImeStr,\*PMyImeStr;

typedef struct \_SendKey

{

        int HasDownKey;

        bool IsSendKey;

        BYTE SendGameDxKeyDate[0xed];

}SendKey,\*PSendKey;

typedef struct \_SendMouse

{

        bool IsSendMouse;

        int x;

        int y;

}SendMouse,\*PSendMouse;

typedef struct \_KuoZan

{

        bool IsHookGetSelfWindow;

        bool IsHookGetCurSor;

}Kuozan,\*Pkuozan;

typedef struct \_MyDic

{

        char Dic1Path[256];

        char Dic2Path[256];

                bool IsloadOk;

}MyDic,\*PMyDic;

typedef struct \_MyShowDic

{

        char Strname[256];

        int DicIndex;

        bool IsshowDic;

}MyShowDic,\*PMyShowDic;

typedef struct \_MyFindDxStr

{

        int DicIndex;

        int X1;

        int Y1;

        int X2;

        int Y2;

        char StrnameS[256];

        char Colors[256];

        int RetFindHows;//返回的数据，找到多少字

        char RetStr[1024];

        bool IsFindOk;

}MyFindDxStr,\*PMyFindDxStr;

typedef struct \_Test\_XY

{

        int X;

        int Y;

        bool IsOk;

}TESTXY,\*PTESTXY;

typedef struct \_RENWU\_GOTO\_XY

{

        int X;

        int Y;

        int Z;

}RENWU\_GOTO\_XY,\*PRENWU\_GOTO\_XY;

typedef struct \_XiGuai\_GOTO\_XY

{

        int X;

        int Y;

}XiGuai\_GOTO\_XY,\*PXiGuai\_GOTO\_XY;

typedef struct \_3S

{

        int BIG;

        int SMALL;

}SSS3,\*P3S;

typedef struct \_ZhiYe

{

        BOOL  IsGetZhiYe;

        WCHAR Zhiye[256];

}ZhiYe,\*PZhiYe;

typedef struct \_DATA\_TO\_DX

{

        int Bind\_moshi;

        int Bind\_moshi\_KEY;

        bool NeedWait;

        bool iscpu;

        int cpu\_sleepTime;

        bool IsScreen;

        char ScreenPath[1024];

        HWND thisWindow;

        HWND  MYWINDOWS;

        bool IsFindPic\_QuanPing;

        char FindPci\_Path[1024];

        bool Begin\_CF;

        DWORD  D3D\_44;//SetTransform偏移量

        DWORD  D3D\_17;//Present偏移量

        DWORD  D3D\_81;

        DWORD  D3D\_82;//DrawIndexedPrimitive偏移量

                DWORD  D3D\_65;//DrawIndexedPrimitive偏移量

        bool   Is\_Bind2\_ok;

        bool   Is\_Bind1\_ok;

        bool   Is\_Bind\_KEY1\_ok;

        bool   IsScreenXY;

        int x1;

        int x2;

        int y1;

        int y2;

        bool IS\_FindPicDX;

        FindPicdx findPicdx\_struct;

        bool IS\_FindPicDX\_XY;

        FindPicdx findPicdx\_xy\_struct;

        MyImeStr myImeMessage;

        HWND ImeHwnd;

        DWORD  D3DKEY\_9;

        DWORD  D3DKEY\_10;

        DWORD Unacquire\_8;

        DWORD SetCooperativeLevel\_13;

        SendKey  DxKeyTogame;

        SendMouse DxMouseTogame;

        Kuozan  SuperKuozan;

        POINT MOUSE\_MOVE\_WINDOWS;

        MyDic  mydic;

        MyShowDic myshowdic;

        MyFindDxStr   myDxStr;

        BYTE  ASM\_CODE[1024];

        int Asm\_code\_len;

        TESTXY  TestXY;

        int GuaiwuShuliang;

        int WUPUN\_WULIANG;

        BOOL ISXIGUAI;

        BOOL ISXIWU;

        RENWU\_GOTO\_XY  RENWU\_XY;

        WupinAll GameBeiBao;

        GetLv\_Name\_Info Name\_Lv;

        bool IsChuShou;

        CurZhuangBei curzb;

        bool IsXiuli;

        bool Isadd\_Liliang;

        int  liliang;

        bool IsRetTili;

        int Rettili;

        int Tili;

        int Lv;

        bool IsGetName\_Lv\_Pilao;

        bool IsGetJinbi;

        int jinbi;

        BOOL IsGetBeibao;

        BOOL IsGetCurZhuangBei;

        BOOL IsEndXiuLiMaiWu;

        BOOL  IsBeginXiuliMaiWu;

        int  XiuliMaiwuWat;

        BOOL IsGetGuaiwuShuliang;

        BOOL  IsGetWupinShuLiang;

        BOOL IsShunyibefor;

        BOOL IsShunyi;

        int Shunyi\_Fangxiang;

        BOOL IsSetXiGuaiFangXiang;

        int XiGuaiFangXiang;

        BOOL  Is3S;

        XiGuai\_GOTO\_XY Guai\_add\_xy;

        BOOL ISGETFANGXIANG;

        int RetGetFangXiang;

        bool IsSet3s;

        SSS3 sss;

        bool IsAddDuli;

        int Duli\_Value;

        bool IsAddJingShen;

        int JingShen\_Value;

        bool IsAddZL;

        int ZL\_Value;

        bool IsRuoGuai;

        bool IsTest;

        bool  IsSY;

        DWORD sdjz;

        DWORD fx;

        DWORD CallBase;

        BOOL  Is\_GetCangku;

        SmallCangku GameCanuku;

        int PrintTest;

        BOOL  ISGOTOXY;

        int XIGUAI\_TYPE;

        ZhiYe zhiye;

}DATA\_TO\_DX,\*PDATA\_TO\_DX;

  typedef  HRESULT( \_\_stdcall\*\_SetCooperativeLevel)(LPVOID DEV, HWND,DWORD);

class DX\_FUNC

{

public:

        MyOcr myocr[2];

        void Get\_SendData();

        PDATA\_TO\_DX DATA2;

        LPDIRECT3DDEVICE9 GameDEV;

        DX\_FUNC();

        void Scrren(LPDIRECT3DDEVICE9 dev,char\* FilePath);

        void SendTo\_Game();

        void SaveSuf\_To\_File();

        int FindPic\_QuanPing(LPDIRECT3DDEVICE9 dev,char\* FilePath,int simMin,int simMax,int \*RetX,int\* RetY);

        int FindPic\_QuanPing\_xy(LPDIRECT3DDEVICE9 dev,char\* FilePath,int simMin,int simMax,int x1,int y1,int x2,int y2,int \*RetX,int\*RetY);

        LPDIRECT3DSURFACE9 Suf;

        bool  IS\_save\_tofile;

        DWORD SetClassVirtualFnAddress\_1(LPVOID pthis,int Index,DWORD Newadr,DWORD\* SaveOLD\_ADR); //用序号取其这个函数的地址
```
