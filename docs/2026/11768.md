
# [原创]windows下的tail，附代码-添加 f选项，看5楼

作者：Aker Lee  mailtoljm#163。com  
标题：windows下的tail  
日期：2007-6-16 7:16

这几天，总要查看大文件，每次用ultraedit打开文件，但是还是很慢，  
非常怀念linux下的tail，到网上搜索了一下，发现有unixutils里面有tail.  
但是从sourceforge上下了n久，都连不上；另外发现codeproject上面有个gui版的  
不喜欢，而且那个是mfc写得。  
想想还是自己动手，丰衣足食  
所以就有了这个小工具，希望大家喜欢。有时候应该比较有用:)

思路很简单，打开文件读到最后，往回读，读到`'\n'`，行计数加一，  
直到满足要读的行数，确定位置，输出.......

编译环境：winxpsp2 + `cl 13.10.3077` + `link 7.10`

问题：读大文件(2G)不支持,不过一般应该够用了,如果要用的话,自己改下  
如果还有其他问题，请mailme:)

改进：如果想要多功能的tail，如从文件某个位置开始读一定行数  
以16进制，字符等显示.......

使用方法：`tail` 文件名 [最后的行数]  
最后的行数为可选项，默认最后10行

代码随便修改，最好给我发一份:)

## 代码

```c
//----------------------------------------------------------------------------
//                                                                           |
//  作者：Aker Lee  mailtoljm#163。com                                       |
//  标题：windows下的tail                                                    |
//  日期：2007-6-16 7:16                                                     |
//                                                                           |
//  这几天，总要查看大文件，每次用ultraedit打开文件，但是还是很慢，          |
//  非常怀念linux下的tail，到网上搜索了一下，发现有unixutils里面有tail.      |
//  但是从sourceforge上下了n久，都连不上；另外发现codeproject上面有个gui版的 |
//  不喜欢，而且那个是mfc写得。                                              |
//  想想还是自己动手，丰衣足食                                               |
//  所以就有了这个小工具，希望大家喜欢。有时候应该比较有用:)                 |
//                                                                           |
//  思路很简单，打开文件读到最后，往回读，读到'\n'，行计数加一，             |
//  直到满足要读的行数，确定位置，输出........                               |
//                                                                           |
//  编译环境：winxpsp2 + cl 13.10.3077 + link 7.10                           |
//                                                                           |
//  问题：读大文件(2G)不支持,不过一般应该够用了,如果要用的话,自己改下        |
//        如果还有其他问题，请mailme:)                                       |
//                                                                           |
//  改进：如果想要多功能的tail，如从文件某个位置开始读一定行数               |
//        以16进制，字符等显示........                                       |
//                                                                           |
//  使用方法：tail 文件名 [最后的行数]                                       |
//            最后的行数为可选项，默认最后10行                               |
//                                                                           |
//  代码随便修改，最好给我发一份:)                                           |
//                                                                           |
//----------------------------------------------------------------------------

//////////////////////////////////////////////////////////////////////////////
// 头文件
#include <stdio.h>
#include <assert.h>
#include <windows.h>

////////////////////////////////////////////////////////////////////////////////////////////////
#pragma comment(lib,"advapi32")
#pragma comment(linker, "/align:0x1000") 
// 如果觉得文件大的话，谁把printf换成wsprintf和writeconsole
////////////////////////////////////////////////////////////////////////////////////////////////
// 全局变量
////////////////////////////////////////////////////////////////////////////////////////////////

////////////////////////////////////////////////////////////////////////////////////////////////
// 函数定义
// taken from msdn SetFilePointer :)
__int64 myFileSeek (HANDLE hf, __int64 distance, DWORD MoveMethod)
{
   LARGE_INTEGER li;
   li.QuadPart = distance;
   li.LowPart = SetFilePointer (hf, li.LowPart, &li.HighPart, MoveMethod);
   if (li.LowPart == INVALID_SET_FILE_POINTER && GetLastError() != NO_ERROR)
   {    // vc6 要修改INVALID_SET_FILE_POINTER为((DWORD)-1)
        // WinBase.h:#define INVALID_SET_FILE_POINTER ((DWORD)-1)
      li.QuadPart = -1;
   }
   return li.QuadPart;
}

void MyCloseFile(HANDLE hFile)
{
    BOOL    bFlag;                  // a result holder
    bFlag = CloseHandle(hFile);     // close the file

    if(!bFlag) 
    {
        printf("[-] Error # %ld occurred closing the file!\n",
           GetLastError());
    }    
}
////////////////////////////////////////////////////////////////////////////////////////////////
int gettail(char *filename,DWORD nlastlines )
{
    // 局部变量
	HANDLE  hFile;
    DWORD   dwFileSize,iterator,bytes_read,lines = 0;
#define BUFFSIZE 1024                // the size of the memory to examine at any one time
	char		buf[BUFFSIZE];

    // 函数参数分析
	assert(filename != NULL);
    assert(nlastlines > 0);
    
    // 函数动作
    // 打开文件
	hFile = CreateFile(	filename, GENERIC_READ,
		FILE_SHARE_READ | FILE_SHARE_WRITE | FILE_SHARE_DELETE ,	NULL,
			OPEN_EXISTING,	FILE_ATTRIBUTE_NORMAL,	NULL);
	if(hFile == INVALID_HANDLE_VALUE)
	{
        printf("[-] Error # %ld occurred opening %s!\n", GetLastError(),filename);
		return -1;
	}

    // 获取文件大小
    dwFileSize = iterator = GetFileSize(hFile,  NULL);
	
    // 寻找最后nlastlines行的初始位置
	while(lines < nlastlines && iterator > 0)
	{
		iterator--;
		myFileSeek(hFile, iterator, FILE_BEGIN);
		if(!ReadFile(hFile, buf, 1, &bytes_read, NULL))
		{
            printf("[-] Error # %ld occurred reading %s!\n", GetLastError(),filename);
			MyCloseFile(hFile);
			return -1;
		}

		if(buf[0] == '\n')  lines++;
	}
	// 如果已经遍历到文件头，重设文件开始位置
	if(iterator == 0) myFileSeek(hFile, 0, FILE_BEGIN);
    
    // 读取到文件末尾并输出
	do{
		if(!ReadFile(hFile, buf, BUFFSIZE-1, &bytes_read, NULL))
		{
            printf("[-] Error # %ld occurred reading %s!\n", GetLastError(),filename);
			MyCloseFile(hFile);
			return -1;
		}
		iterator += bytes_read;
		buf[bytes_read] = '\0';

		printf("%s", buf);
	}while(bytes_read >0 );
	
    MyCloseFile(hFile);
    // 返回值
    return 0;
}

//////////////////////////////////////////////////////////////////////////////
// 入口点
int main(int argc, char* argv[])
{
    // 局部变量
    DWORD nlastlines = 10;          // 默认最后10行
    char *szFileName = strdup(argv[1]);
    // 参数分析
    // 1命令行分析
    if (argc < 2)
    {
    	    printf("[+] Tail 0.90 created by Aker Lee <mailtoljm#163.com>\n");
            printf("[-] Usage: tail {file name} [number of lines]\n"
                   "[-]        Display last 10 lines by default\n");
            return 1;
    }
    else if(argc == 3)
    {
        nlastlines = (DWORD)atoi(argv[2]);
    }
    // 函数动作
    gettail(szFileName,nlastlines);
    
    free(szFileName);
    // 返回值
	return 0;
}
//////////////////////////////////////////////////////////////////////////////
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [tail.zip](javascript:void(0);)

  （19.68kb，38次下载）
