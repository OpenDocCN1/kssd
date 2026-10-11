
```asm
        test eax, eax  

        jnz  skip_unregistered_part               ;if there is registration key, jump  

;if you want to do special action for unregistered version, You can do it there.  

        push 0  

        push offset special_capture2  

        push offset special_message2  

        push 0  

        iWin32 MessageBoxA  

;end of part for non registered users  

skip_unregistered_part:  

        push -1  

        iWin32 ExitProcess  

end Start
```

**VC:**

```c
#include <windows.h>  

#include <io.h>  

#include "...&Lib.h"  

#pragma comment (lib,"SPECIAL")  

int WINAPI WinMain (HINSTANCE hInstance, HINSTANCE hPrevInstance,  

                    PSTR szCmdLine, int iCmdShow)  

{  

  char *message = "Encryption blocks example!";  

  char* registration_information=(char*)malloc(255*sizeof(char));  

  int flag;  

  MessageBox(0,message,"",0);  

  SVKP_ENCRYPTED_BLOCK_START    //Start of encrypted block  

// This code will encrypted and it will decrypted only if user have correct registration key  

    message = "Registered version!";  

    MessageBox(0,message,"",0);    // display message box for registered users  

  SVKP_ENCRYPTED_BLOCK_END      // End of encrypted block   

// if was registered flag=1   

  if(flag = SVKP_GetRegistrationInformation(registration_information)==0) {  

// This is part for unregistered users     

    message = "Unregistered version!";     

    MessageBox(0,message,"",0);  

  }  

  return 0;  

}
```

**Delphi:**

```pascal
unit main;  

interface  

procedure Start;  

implementation  

uses  

  Registry,  

  Classes,  

  Windows,  

  SysUtils,  

  special;  

var  

  RegMessage : String;  

// This procedure will encrypted and it will decrypted only if user have correct registration key  

procedure ActionForRegisteredUsers;  

begin  

    {$I include\delphi_svkp_block_begin.inc}  

    RegMessage := 'Registered version!';  

    MessageBox(0,PChar (RegMessage),'',0);  

    {$I include\delphi_svkp_block_end.inc}  

end;  

procedure Start;  

begin  

  try  

    RegMessage := 'Encryption blocks example';  

    MessageBox(0,PChar (RegMessage),'',0);  

    ActionForRegisteredUsers; // this procedure is only for registered users  

  except  

  end;  

end;  

end.
```

**警告:** 不要在 `VB` 中使用这个方法!

**说明:** 在 `SVKP` 的 `Examples/Encrypted_blocks_RSA` 目录下有使用这个方法的例子.

### 用特征保护

    在选择这个选项之前需要先在菜单中选择 "Use registration keys(使用注册密钥)",因为这个函数是建立在注册密钥使用的基础之上.在选择了这个选项后加密密钥就可用于单独的特性了.别忘了这个函数的使用需要对被保护程序的源代码做一些调整.

    可以在程序中设置 32 个保护特性.对于每个特性都可以选择多个代码块,这些代码块被标记为 `SVKP_ENCRYPTED_BLOCK_TYPEX_START` 和 `SVKP_ENCRYPTED_BLOCK_TYPEX_END` ,X 代表特性的号码.举例说明对于特性 1#可以有 100 个代码块,这些代码块只能在注册文件中进行解锁.这看起来很复杂，但是使用这个函数相当容易(请看例子).

    因此,标记单独特性的开始和结束并且编译代码足够了.在 `SVK Protector` 中选择 "Use features for license keys(给授权密钥使用特性)" 选项并且按下按钮 "Protect!!",这些可以保护这个应用程序.

    当你为你的客户生成注册密钥的时候,选择特性,将在特性菜单中解锁(例如我们选择特性 1 和 3)选择的特性将解锁但其它的特性将被锁定。这使得处理消费者的区别相当容易.

    **例子：**

**ASM:**

```asm
        .586  
        .MMX  
        .model flat,stdcall  
include ..\..\Inc&Lib\apimacro.mac  
include ..\..\Inc&Lib\windows.inc  

INCLUDELIB      ..\..\Inc&Lib.lib  

INCLUDELIB      ..\..\Inc&Lib\iUSER32.lib  

;there are defined include file and library for SVKP's special functions  

include         ..\..\Inc&Lib\special.inc  

INCLUDELIB      ..\..\Inc&Lib\ispecial.lib  

.DATA?  

registration_information db 255 dup (?)         ;buffer for registration information  

.DATA  

capture db "Features example.",0  

message db "Now I will try run some features!",0  

capture_feature1 db "Feature 1",0  

message_feature1 db "Feature 1 is not enabled in license!",0  

special_capture2 db "Message for unregistered users!",0  

special_message2 db "You can buy our product at [www....](http://www....)",0  

feature1_unlocked_flag db 0  

.code  

assume          fs      :flat  

Start:  

;int 3                                   ;only for debug version  

        push 0  

        push offset capture  

        push offset message  

        push 0  

        iWin32 MessageBoxA  

SVKP_ENCRYPTED_FEATURE1_START                 ;start of feature 1  

;this code will encrypted and it will decrypted only if user have correct registration key with this feature unlocked  

        push 0  

        push offset special_capture1  

        push offset special_message1  

        push 0  

        iWin32 MessageBoxA                  ;display message box for registered users  

        jmp  skip_text  

special_capture1 db "Message for registered users!",0  

special_message1 db "Feature 1 is unlocked in license!",0  

skip_text:  

        mov  [feature1_unlocked_flag], 1        ;set flag, because feature is unlocked in license  

SVKP_ENCRYPTED_FEATURE1_END                 ;end of feature 1  

        cmp  [feature1_unlocked_flag], 1  

        jz   skip_unregistered_feature1  

;display text for users with feature 1 not unlocked  

        push 0  

        push offset capture_feature1  

        push offset message_feature1  

        push 0  

        iWin32 MessageBoxA  

skip_unregistered_feature1:  

;now we will read registration information  

        iWin32 SVKP_GetRegistrationInformation, offset registration_information  

        test eax, eax  

        jnz  skip_unregistered_part               ;if there is registration key, jump  

;if you want to do special action for unregistered version, You can do it there.  

        push 0  

        push offset special_capture2  

        push offset special_message2  

        push 0  

        iWin32 MessageBoxA  

;end of part for non registered users  

skip_unregistered_part:  

;action for users with Feature 2 activated  

SVKP_ENCRYPTED_FEATURE2_START  

        push 0  

        push offset special_capture3  

        push offset special_message3  

        push 0  

        iWin32 MessageBoxA                  ;display message box for registered users  

        jmp  skip_text2  

special_capture3 db "Message for registered users!",0  

special_message3 db "Feature 2 is unlocked in license!",0  

skip_text2:  

SVKP_ENCRYPTED_FEATURE2_END  

;is possible select more than one part of code for one feature  

SVKP_ENCRYPTED_FEATURE1_START  

        push 0  

        push offset special_capture4  

        push offset special_message4  

        push 0  

        iWin32 MessageBoxA                  ;display message box for registered users  

        jmp  skip_text3  

special_capture4 db "Message for registered users!",0  

special_message4 db "Feature 1 second part of code!",0  

skip_text3:  

SVKP_ENCRYPTED_FEATURE1_END  

        push -1  

        iWin32 ExitProcess  

end Start
```

**VC:**

```c
#include <windows.h>  

#include <io.h>  

#include "..\\..\\Inc&Lib\\special.h"  

#pragma comment (lib,"SPECIAL")  

int WINAPI WinMain (HINSTANCE hInstance, HINSTANCE hPrevInstance,  

                    PSTR szCmdLine, int iCmdShow)  

{  

  char *message = "Features example!";  

  char* registration_information=(char*)malloc(255*sizeof(char));  

  int flag;  

  MessageBox(0,message,"Now I will try run some features!",0);  

  SVKP_ENCRYPTED_FEATURE1_START     // Start of feature 1  

// this code will encrypted and it will decrypted only if user have correct registration key with this feature unlocked  

    MessageBox(0,"Feature 1 is unlocked in license!","Message for registered users!",0);    // display message box for registered users  

  SVKP_ENCRYPTED_FEATURE1_END         // end of feature 1  

// if was registered flag=1   

  if(flag = SVKP_GetRegistrationInformation(registration_information)==0) {  

// This is part for unregistered users     

    message = "Unregistered version!";     

    MessageBox(0,message,"",0);  

  }  

// action for users with Feature 2 activated  

  SVKP_ENCRYPTED_FEATURE2_START  
```
