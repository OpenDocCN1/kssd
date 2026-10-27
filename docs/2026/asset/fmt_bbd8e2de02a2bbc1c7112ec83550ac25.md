
```

/******************************************************

* FileName   :   1.cpp

* Author     :   sudami

* FinishTime :   2007/07/10

* Comment    :   converse a verb to

******************************************************/

#include <iostream.h>
#include <string.h>
#include <afx.h>
#include <stdlib.h>
#include <stdio.h>
#include <conio.h>
#include <fstream.h>

char g_buffer[MAX_PATH];     // globle array in order to preserve strings
TCHAR szBuffer[MAX_PATH+1];  // current directory
HANDLE hFile;

////////////////////////////////////////////////////////////////////////////

int ShowMessage ();        // 显示提示信息
int Convert (char *str);   // 负责动词的查找、添加和显示
int Dispart (char *s);     // 将数据写入文件

////////////////////////////////////////////////////////////////////////////

int main ()
{
    ShowMessage ();
    Convert (g_buffer);

    return 0;
}

/***********************************************************************
*                                                                      *
*           Module: Convert ()                                         *
*           Function : convert some vertb past tense to Prototype      *
************************************************************************/

int Convert (char *s)
{
    int i = 0;
    int j = 0;
    int k = 0;
    char temp[MAX_PATH]; // array intend to preserve letters

    // filters the character string,only left with letters 
    do
    {
        while (*s >= 'A' && *s <= 'Z' || *s >= 'a' && *s <= 'z')
        {
            temp[i] = *s;
            ++i;
            ++s;
        }
    }while (*s++);

    // check if it iterminates with 'd'(ignore the '\0' character)
    if (temp[i-1] == 'd')
    {
        if (temp[i-2] == 'e')
        {
            if (temp[i-3] == temp[i-4])
            {
                i = i - 3;
                j = i - 1;
                cout << "---->Result:" ;
                while (i--)
                {
                    cout << temp[j-i];
                }
                cout << endl;

                cout << "Input:";
                cin.getline (g_buffer, MAX_PATH);
                Convert (g_buffer);    
            } 
            else
            {
                cout << "---->Result:" ;
                i = i - 2;
                j = i - 1;
                while (i--)
                {
                    cout << temp[j-i];
                }
                cout << endl;
                cout << "Input:";
                cin.getline (g_buffer, MAX_PATH);
                Convert (g_buffer);    
            } // if
        } 
    } 

    /*********************************************************************
    **** 下面的else是判断的重点，如果是不规则动词就按照下面的方式处理 ****
    *********************************************************************/

    else
    {
        cout << "this is not a regular verb,if it is not find int test.txt,you could build it." << endl;

        // 取得当前目录
        ::GetCurrentDirectory (MAX_PATH + 1, szBuffer);
        if (szBuffer[lstrlen(szBuffer) - 1] != '\\')
            lstrcat (szBuffer, TEXT("\\"));

        lstrcat (szBuffer, "text.txt");

        // 打开文件
        CStdioFile   fText;  
        CFileException FileExc;
        UINT nOpenFlags;
        CString str;

        nOpenFlags = CFile::modeReadWrite | CFile::modeCreate | CFile::modeNoTruncate;
        if (!fText.Open(szBuffer, nOpenFlags, &FileExc))
        {
            FileExc.ReportError ();
            return 0;
        }
        DWORD dwBytes = fText.GetLength ();
        fText.Close ();  

        // 判断文件是否为空
        if (dwBytes == 0)  // 是空文件
        {
            char choice;
            cout << "There are no characters macth the current string,do you want to create it? y/n" <<endl;
            choice = getchar ();
            if (choice == 'y')
            {
                char szBuild[1024];
                cout << "\t* [+] Usage: intput 'broke@break' *\n"
                     << "\t* [-] Example:  broke@break *\n"
                     << "\t* [-] Result: Your input is now preserved in test.txt *\n"
                     << endl;
                cout << "Build:";
                cin.getline (szBuild, 1024);

                // 写到文件中
                Dispart (szBuild);
                cout << "Your input is now preserved in " << szBuffer <<endl;

                cout << "Input:";
                cin.getline (g_buffer, MAX_PATH);
                Convert (g_buffer);
            }
            else
            {
                cout << "Input:";
                cin.getline (g_buffer, MAX_PATH);
                Convert (g_buffer);
            } // if
        }

        else  // 非空文件
        {
            // 打开文件
            CStdioFile   Text;  
            CFileException FileExc;
            UINT nOpenFlags;
            CString str;

            nOpenFlags = CFile::modeReadWrite | CFile::modeCreate | CFile::modeNoTruncate;

            if (!Text.Open(szBuffer, nOpenFlags, &FileExc))
            {
                MessageBox(NULL, "不能打开非空文件", "sudami", MB_OK);//显示出来
                FileExc.ReportError ();
                return 0;
            }

            BOOL bCheck = FALSE;
            CString szstr;

            // 一行一行的读取文件中的内容
            while (Text.ReadString (szstr))
            { 
                //  MessageBox(NULL, szstr, "sudami", MB_OK);//显示出来
                UINT t = szstr.Find("@");
                UINT n = szstr.GetLength ();
                int l = n - t - 1;
                CString szLeft = szstr.Left (t);

                // 如果此行中@前面的值与用户输入的匹配,则输出@后面的值
                if (strcmp (g_buffer, szLeft) == 0)
                {
                    bCheck = TRUE;
                    CString szRight = szstr.Right (l);
                    cout << "---->Result:" << szRight <<endl;
                //    cout << "Input:";
                //    cin.getline (g_buffer, MAX_PATH);
                //    Convert (g_buffer);
                //    break;
                }    
            }
            Text.Close ();

            if (!bCheck) // 在文件中没有找到可以匹配的, 新建
            {
                cout << "There are no characters macth the current string,do you want to create it? y/n" <<endl;

                char szBuild[1024];
                cout << "\t* [+] Usage: intput 'broke@break' *\n"
                    << "\t* [-] Example:  broke@break *\n"
                    << "\t* [-] Result: Your input is now preserved in test.txt *\n"
                    << endl;
                cout << "Build:";
                cin.getline (szBuild, 1024);

                // 写入文件
                Dispart (szBuild);
                cout << "Your input is now preserved in " << szBuffer <<endl;

                cout << "Input:";
                cin.getline (g_buffer, MAX_PATH);
                Convert (g_buffer);
            }

        } // if

    } // if

    return 0;
}

/***********************************************************************
*                                                                      *
*        Module: Dispart ()                                            *
*        Function: Read data from test.txt,dispart into two characters *
************************************************************************/

int Dispart (char *s)
{
    int i = 0;
    int j = 0;
    int k = 0;
    int r = 0;
    char szBuildBuffer1[20];
    char szBuildBuffer2[20];

    /************************************************************
    * 获取用户输入的对应字符串，将其分开分别保存在2个数组中
    * eg：用户输入---> broke@break,则szBuildBuffer1[1024]保存broke
    * szBuildBuffer2[1024]保存break 
    ************************************************************/
    do
    {
        if (*s == '@')
        {
            char szBuild[1024];
            cout << "Error,please input again!\n";
            cout << "Build:";
            cin.getline (szBuild, 1024);
            Dispart (szBuild);
        }

        while (*s >= '@' && *s <= 'Z' || *s >= 'a' && *s <= 'z')
        {
            if (*s == '@')
            {
                s++;
                do
                {

                    while (*s >= 'A' && *s <= 'Z' || *s >= 'a' && *s <= 'z')
                    {
                        szBuildBuffer2[j] = *s; 
                        j++;
                        s++;
                    }
                }while (*s++);
            }

            else
            {
                if (*s == '\0')
                    break;

                szBuildBuffer1[i] = *s;
                i++;
                s++;
            }
        }
    }while (*s++);

    /*******************************************************************
    * 将所得到的2个数组内容写入到test.txt文件中
    * 用到fstream库中的函数 fputc
    *******************************************************************/

    char a = '@';
    char b = '\n';
    FILE *fp;
    if ((fp = fopen (szBuffer, "at+")) == NULL)
    {
        ::MessageBox (NULL, "Can not open it!", "sudami", MB_OK);
        exit (1);
    }

    k = i - 1;
    while (--i)
    {
        fputc (szBuildBuffer1[k - i], fp);
    }

    fputc (a,fp);
    r = j - 1;
    while (j--)
    {
        fputc (szBuildBuffer2[r - j], fp);
    }
    fputc (b,fp);
    fclose (fp);
    return 0;
}

/***********************************************************************
*                                                                      *
*           Module: ShowMessage ()                                     *
*           Function : Display some Information to the user            *
************************************************************************/

int ShowMessage ()
{
    system ("color F");
    system ("title sudami");
    cout << "--- a little subject made by sudami ---\n"
        << "--- You may contact with me through the mail: <[email]xiao_rui_119@163.com[/email]> ---\n\n"
        << "\t* Please input some verb past tense here... *\n"
        << "\t* [+] Usage: intput '1r+%e3a@4d5e6d',return 'read' *\n"
        << "\t* [-] Example:  1r+%e3a@4d5e6d *\n"
        << "\t* [-] ---->Result: read *\n"
        << endl;
    cout << "Input:";
    cin.getline (g_buffer, MAX_PATH);

    return 0;
}
```
