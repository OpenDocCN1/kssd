
# PSAPI 和 ToolHelpAPI 学习笔记

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-730.htm`](https://www.pediy.com/kssd/pediy07/pediy7-730.htm)

关键字：`Tool Help Library`

## 一、所有的 Tool Help API

- `CreateToolhelp32Snapshot`
- `Heap32First`
- `Heap32ListFirst`
- `Heap32ListNext`
- `Heap32Next`
- `Module32First`
- `Module32Next`
- `Process32First`
- `Process32Next`
- `Thread32First`
- `Thread32Next`
- `Toolhelp32ReadProcessMemory`

## 二、编程实践

```cpp
#include <windows.h>
#include <tlhelp32.h>//be sure before include tlhelp32.h, you should contain windows.h
#include <fstream>
#include <iostream>

using namespace std ;

//use to storing process id of "EXPLORER.EXE" 

static DWORD dwExplorerProcessId = 0 ;

ofstream fout ;

/************************************************************************
 *    sub function pre-defined
 ************************************************************************/

void EnumProcessList ( ) ;

void EnumThreadOfExplorer () ;

void EnumModuleOfExplorer () ;

void EnumHeapListOfExplorer () ;

void EnumHeapOfExplorer ( DWORD ) ;

/************************************************************************
 *   main function
 ************************************************************************/

int main()
{
	EnumProcessList () ;

	EnumThreadOfExplorer () ;

	EnumModuleOfExplorer () ;

	EnumHeapListOfExplorer () ;

	return 0 ;
}


void EnumProcessList()
{
	fout.open ( "ProcessList.txt" ) ;

	//Be careful : if ( dwFlags == TH32CS_SNAPPROCESS ) ,the second parament of 
	//CreateToolhelp32Snapshot will be ignored

	HANDLE hProcessShot = CreateToolhelp32Snapshot ( TH32CS_SNAPPROCESS, 0 ) ;

	if ( hProcessShot == INVALID_HANDLE_VALUE )
	{
		fout << "CreateToolhelp32Snapshot failed!" << endl ;
	}
	else
	{
		PROCESSENTRY32 pe32 ;

		pe32.dwSize = sizeof(pe32) ;

		if ( Process32First ( hProcessShot, &pe32 ) )
		{
			do {
				fout << "************************************************************************" << endl ;
				fout << "*Process : " << pe32.szExeFile << endl ;
				fout << "************************************************************************" << endl ;

				fout << "Usage : "			<< pe32.cntUsage				<< endl ;
				fout << "ProcessID : "		<< pe32.th32ProcessID			<< endl ;
				fout <<	"DefaultHeapID : "		<< (ULONG_PTR)pe32.th32DefaultHeapID		<< endl ;
				fout << "ModuleID : "		<< pe32.th32ModuleID			<< endl ;
				fout << "ThreadNum : "		<< pe32.cntThreads				<< endl ;
				fout << "ParentProcessID : "	<< pe32.th32ParentProcessID			<< endl ;
				fout << "PriClassBase : "		<< pe32.pcPriClassBase			<< endl ;
				
				if ( !strcmp( pe32.szExeFile, "EXPLORER.EXE" ) )
				{
					//get Process ID of "EXPLORER.EXE" for addation use

					dwExplorerProcessId = pe32.th32ProcessID ;
				}

				fout << endl << endl ;

			}while ( Process32Next ( hProcessShot, &pe32 ) ) ;
		}

	}

	CloseHandle ( hProcessShot ) ;

	fout.close() ;
}


void EnumThreadOfExplorer ( )
{
	fout.open ( "ThreadListOfExplorer.txt" ) ;

	if ( dwExplorerProcessId == 0 )
	{
		fout << "Get Explorer Process ID failed!" << endl ;
		return ;
	}

	THREADENTRY32 te32 ;

	te32.dwSize = sizeof(THREADENTRY32) ;

	//Be careful : if ( dwFlags == TH32CS_SNAPPROCESS ) ,the second parament of 
	//CreateToolhelp32Snapshot will be ignored

	HANDLE hThreadSnap = CreateToolhelp32Snapshot ( TH32CS_SNAPTHREAD, 0 ) ;

	if ( hThreadSnap == INVALID_HANDLE_VALUE ) 
        return ; 

	if ( Thread32First ( hThreadSnap, &te32 ) )
	{
		do{
			if ( te32.th32OwnerProcessID == dwExplorerProcessId )
			{
				fout << "************************************************************************" << endl ;
				fout << "*ThreadId : "   << te32.th32ThreadID	      << endl ;
				fout << "************************************************************************" << endl ; 
				fout << "Usage    : "	 << te32.cntUsage	      << endl ;
				fout << "Delta Priority : " << te32.tpDeltaPri      << endl ;
				fout << "Base Priority  : " << te32.tpBasePri	      << endl ; 
				fout << endl << endl ;
            } 

		}while ( Thread32Next ( hThreadSnap, &te32 ) ) ;
	}

	CloseHandle ( hThreadSnap ) ;
	
	fout.close() ;
}


void EnumModuleOfExplorer()
{
	fout.open ( "ModuleListOfExplorer.txt" ) ;

	if ( dwExplorerProcessId == 0 )
	{
		fout << "Get Explorer Process ID failed!" << endl ;
		return ;
	}

	MODULEENTRY32 me32 ;

	me32.dwSize = sizeof(MODULEENTRY32) ;


	HANDLE hModuleSnap = CreateToolhelp32Snapshot ( TH32CS_SNAPMODULE, dwExplorerProcessId ) ;

	if ( hModuleSnap == INVALID_HANDLE_VALUE ) 
        return ; 

	if ( Module32First ( hModuleSnap, &me32 ) )
	{
		do{
			fout << "************************************************************************"	<< endl ;
			fout << "*Module        : "	<< me32.szModule		<< endl ;
			fout << "************************************************************************"	<< endl ; 
			fout << "ModulePath     : " << me32.szExePath		<< endl ;
			fout << "ProcessID      : "	<< (PVOID)me32.th32ProcessID<< endl ;
			fout << "ModuleHandle   : "	<< me32.hModule		<< endl ;
			fout << "ModuleBaseAddr : " << (PVOID)me32.modBaseAddr	<< endl ;
			fout << "ModuleBaseSize : " << (PVOID)me32.modBaseSize	<< endl ;
			fout << "GlobalUsage    : "	<< me32.GlblcntUsage	<< endl ;
			fout << "LocalUsage     : "	<< me32.ProccntUsage	<< endl ;
 
			fout << endl << endl ;
		
		}while ( Module32Next ( hModuleSnap, &me32 ) ) ;
	}

	CloseHandle ( hModuleSnap ) ;

	fout.close() ;
}


void EnumHeapListOfExplorer ()
{
	fout.open ( "HeapListOfExplorer.txt" ) ;

	if ( dwExplorerProcessId == 0 )
	{
		fout << "Get Explorer Process ID failed!" << endl ;
		return ;
	}

	HEAPLIST32 hl32 ;

	hl32.dwSize = sizeof(HEAPLIST32) ;


	HANDLE hHeapListSnap = CreateToolhelp32Snapshot ( TH32CS_SNAPALL, dwExplorerProcessId ) ;

	if ( hHeapListSnap == INVALID_HANDLE_VALUE ) 
        return ; 

	if ( Heap32ListFirst ( hHeapListSnap, &hl32 ) )
	{
		cout << "Wait for a few moment..." << endl ;

		do{
			fout << "********************************************************************************" << endl ;
			fout << "* HeapId        : " << (PVOID)hl32.th32HeapID	<< endl ;
			fout << "********************************************************************************" << endl ;

			// In this step will enum all heaps of explorer ;
			// It will take about half of minute to finish 
			// Keep a little patient
			EnumHeapOfExplorer ( hl32.th32HeapID ) ;	
 
			fout << endl << endl ;
		
		}while ( Heap32ListNext ( hHeapListSnap, &hl32 ) ) ;
	}

	CloseHandle ( hHeapListSnap ) ;

	fout.close() ;
}

void EnumHeapOfExplorer ( DWORD dwHeapId )
{
	HEAPENTRY32 he32 ;

	he32.dwSize = sizeof(HEAPENTRY32) ;

	if ( Heap32First ( &he32, dwExplorerProcessId, dwHeapId ) )
	{
		do{
			fout << '\t' << "HeapHandle			: " << he32.hHandle				<< endl ;
			fout << '\t' << "ProcessId			: " << he32.th32ProcessID		<< endl ;
			fout << '\t' << "HeapId			: " << (PVOID)he32.th32HeapID	<< endl ;
			fout << '\t' << "HeapAddress		: " << (PVOID)he32.dwAddress	<< endl ;
			fout << '\t' << "HeapSize			: " << (PVOID)he32.dwBlockSize	<< endl ;
			fout << '\t' << "Flags			: " << he32.dwFlags				<< endl ;
			fout << '\t' << "LockCount			; " << he32.dwLockCount			<< endl ;
 
			fout << endl << endl ;
		
		}while ( Heap32Next ( &he32 ) ) ;
	}
}
```

---

**作者**：北极星 2003

**邮箱**：`zhangjingsheng_nbu@yahoo.com.cn`

`4dd466d0e1e0cc123afbe504da2462ca`
