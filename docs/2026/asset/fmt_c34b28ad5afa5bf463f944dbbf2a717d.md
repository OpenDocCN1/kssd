
# [原创]从最小的`LoadPic`开始（`IPICTURE`接口）

参考原型：

微软的`loadpic.exe`，下载地址：`585K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4#2M7s2m8G2M7Y4c8Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6C8j5W2)9J5c8U0t1I4z5o6V1%4x3R3.`

我能做到的最小的`LoadPic`:

```cpp
#include <windows.h>

#include <olectl.h>

LPPICTURE                        gpPicture;

HWND                                hWnd;

HDC                                        hDC;

long                                hmWidth,hmHeight;

int                                        SrcX,SrcY;

LRESULT CALLBACK MainWndProc(HWND, UINT,WPARAM, LPARAM);

void LoadPictureFile(char* point);

int WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)

{

        MSG                        msg;

        WNDCLASS        wndclass;

        wndclass.style = 0;

        wndclass.cbClsExtra = 0;

        wndclass.cbWndExtra = 0;

        wndclass.hCursor = LoadCursor(NULL, IDC_ARROW);

        wndclass.lpszMenuName =  NULL;

    hInstance=GetModuleHandle(NULL);

        wndclass.hInstance = hInstance;

        wndclass.hIcon = NULL;

        wndclass.lpfnWndProc = MainWndProc;

        wndclass.hbrBackground = (HBRUSH)(COLOR_WINDOW);

        wndclass.lpszClassName = "MyWindowClass";

        RegisterClass (&wndclass);

        hWnd = CreateWindow ("MyWindowClass","LoadPic",

                                                 WS_MAXIMIZEBOX|WS_MINIMIZEBOX|WS_SYSMENU|WS_SIZEBOX,

                                                 CW_USEDEFAULT, 0, CW_USEDEFAULT, 0, NULL,NULL,hInstance,NULL);

        hDC = GetDC(hWnd);

        ShowWindow(hWnd, SW_SHOWNORMAL);

        while (GetMessage(&msg, NULL, 0, 0))

        {

                DispatchMessage(&msg);

        }

        return 0;

}

void LoadPictureFile(char* point)

{

        char                szPath[255];

        OLECHAR                wszPath[255];

        int                        i,j;

        if (gpPicture)

        {

                gpPicture->Release();

        }

        i=0;

        j=0;

        do

        {

                szPath[j]=point[i];

                if(szPath[j]=='\\')

                {

                        j++;

                        szPath[j]='\\';

                }

                i++;

                j++;

        }while(point[i-1]!='\0');

        MultiByteToWideChar(CP_ACP, 0, szPath, -1, wszPath, 255);

        OleLoadPicturePath(wszPath, 0, 0, 0, IID_IPicture, (LPVOID *)&gpPicture);

        gpPicture->get_Width(&hmWidth);

        gpPicture->get_Height(&hmHeight);

        SrcX=hmWidth*GetDeviceCaps(hDC, LOGPIXELSX)/2540;

        SrcY=hmHeight*GetDeviceCaps(hDC, LOGPIXELSY)/2540;

        InvalidateRect(hWnd,NULL,1);

}

LRESULT CALLBACK MainWndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)

{

        PAINTSTRUCT                ps;

        OPENFILENAME        ofn;

    char                        szFileName[255];

        switch (message)

        {

                case WM_RBUTTONUP:

                        szFileName[0]='\0';

                        ofn.lStructSize=sizeof(OPENFILENAME);

                        ofn.hwndOwner=hWnd;

                        ofn.hInstance=0;

                        ofn.lpstrFilter="JPG Files (*.jpg)\0*.jpg\0";

                        ofn.lpstrCustomFilter=0;

                        ofn.nMaxCustFilter=0;

                        ofn.nFilterIndex=0;

                        ofn.lpstrFile=szFileName;

                        ofn.nMaxFile=MAX_PATH;

                        ofn.lpstrFileTitle=0;

                        ofn.nMaxFileTitle=MAX_PATH;

                        ofn.lpstrInitialDir=0;

                        ofn.lpstrTitle=0;

                        ofn.Flags=OFN_HIDEREADONLY|OFN_PATHMUSTEXIST;

                        ofn.nFileOffset=0;

                        ofn.nFileExtension=0;

                        ofn.lpstrDefExt=NULL;

                        ofn.lCustData=0;

                        ofn.lpfnHook=0;

                        ofn.lpTemplateName=0;

                        if (GetOpenFileName(&ofn))

                        {

                                LoadPictureFile(szFileName);

                        }

                        break;

                case WM_PAINT:

                        BeginPaint(hWnd, &ps);

                        if (gpPicture)

                        {

                                gpPicture->Render(hDC, 0, 0, SrcX, SrcY, 0, hmHeight, hmWidth, -hmHeight, NULL);

                        }

                        EndPaint(hWnd, &ps);

                        break;

                case WM_DESTROY:

                        ExitProcess(0);

                        break;

                default:

                        return DefWindowProc (hWnd, message, wParam, lParam);

   }

   return 0;

}
```

VC6编译后大小：2032字节，连2K都不到。

源程序下载：

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`LoadPic.zip`](javascript:void(0);)（12.92kb，53次下载）
