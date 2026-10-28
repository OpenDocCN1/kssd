
```
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;  Service Control Program  by nohacks
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
.386

.model flat, stdcall

option casemap:none

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;                             I N C L U D E   F I L E S
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

include windows.inc

include kernel32.inc

include user32.inc

include advapi32.inc

includelib kernel32.lib

includelib user32.lib

includelib advapi32.lib

include \RadASM\masm32\Macros\Strings.mac

includelib debug.lib

include debug.inc

include load.inc

; ----------------------debug-------------------------------
; PrintError
; PrintText<"debug message!">
; PrintString<driver>
; PrintStringByAddr<offset driver>
; PrintDword<hSCManager,driver>

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;                                       C O D E
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

 .code

 LoadDriver proc uses esi edi lpszDriverPath,lpszDriverName,lpDisplayName

local hSCManager:HANDLE,SvrSta,Error
local hService:HANDLE
local acDriverPath[MAX_PATH]:CHAR
LOCAL bRet:BOOL

   mov bRet,FALSE
   ;打开服务管理器
   invoke OpenSCManager, NULL, NULL, SC_MANAGER_CREATE_SERVICE

    mov hSCManager, eax
    .if ! hSCManager

          PrintText "打开服务管理器出错"
          PrintError

    .endif

    ;得到完整的驱动路径    
    invoke GetFullPathName, lpszDriverPath,sizeof acDriverPath,addr acDriverPath,NULL
     PrintString acDriverPath

    ;创建驱动所对应的服务       
       invoke CreateService, hSCManager, lpszDriverName, lpDisplayName, \
                SERVICE_START + DELETE, SERVICE_KERNEL_DRIVER, SERVICE_DEMAND_START, \
                SERVICE_ERROR_IGNORE, addr acDriverPath, NULL, NULL, NULL, NULL, NULL
        mov hService, eax

        ;创建成功就跳到运行服务处     
          .if  hService

                jmp RunServer      

           .endif

        ;没跳代表失败         
         invoke GetLastError
         mov Error,eax

         ;创建服务失败，不是因为其他原因
           .if  Error!=ERROR_IO_PENDING  && Error!=ERROR_SERVICE_EXISTS   && Error!=ERROR_SERVICE_MARKED_FOR_DELETE

                    PrintText "注册驱动时出错，出错信息如下："
                    PrintError                 
                    mov bRet,FALSE
                    jmp exit

         .endif

           ; 服务创建失败，是由于服务已经创立过

            invoke OpenService ,hSCManager,$CTA0("beeper"), SERVICE_ALL_ACCESS
            mov hService, eax

                   ;如果打开服务也失败，则意味错误 
                   .if eax==NULL
                             PrintText "注册驱动时出错，出错信息如下："
                             PrintError
                             mov bRet,FALSE
                             jmp exit 

                    .endif

            ;服务创建失败，是由于服务已经标记删除  
             .if  Error==ERROR_SERVICE_MARKED_FOR_DELETE
                  ;停止服务
                  invoke ControlService,hService,SERVICE_CONTROL_STOP, addr SvrSta
                  invoke CloseServiceHandle,hService

                   ;再次创建
                   invoke CreateService, hSCManager, lpszDriverName, lpDisplayName, \
                          SERVICE_START + DELETE, SERVICE_KERNEL_DRIVER, SERVICE_DEMAND_START, \
                            SERVICE_ERROR_IGNORE, addr acDriverPath, NULL, NULL, NULL, NULL, NULL

                          ;如果打创建务再失败，则意味着错误                       
                            .if eax==NULL
                                PrintText "注册驱动时出错，出错信息如下："
                              PrintError
                                mov bRet,FALSE
                                jmp exit 

                            .endif

              .endif       

       ;运行服务

       RunServer:

            PrintText "CreateService OK!"  

            invoke StartService, hService, 0, NULL

            .if eax==NULL

                invoke GetLastError

                 ;其他错误
                  .if(eax!=ERROR_IO_PENDING && eax!=ERROR_SERVICE_ALREADY_RUNNING) 

                    PrintText<"StartService error ,other error!"> 
                    PrintError  

                       mov bRet,FALSE
                   jmp exit

             .elseif  eax==ERROR_IO_PENDING

                 ; 设备被挂起 

                 PrintText<"StartService error,设备被挂起 !"> 
                 PrintError                 
                 mov bRet,FALSE
                 jmp exit

                  .else

                   PrintText<"设备已经运行!"> 

                  mov bRet,TRUE

                  jmp exit

                 .endif

            .endif

    mov bRet,TRUE

    exit:

      .if  hService

               invoke CloseServiceHandle ,hService ; 服务句柄
     .endif

      .if hSCManager

             invoke CloseServiceHandle ,hSCManager; SCM句柄

      .endif

     mov eax,bRet

     ret

LoadDriver endp

 UnloadDriver proc uses esi edi  lpszSvrName

   LOCAL bRet:BOOL
   local hSCManager:HANDLE
   local hService:HANDLE
    LOCAL SrvStatus:SERVICE_STATUS

   mov bRet,FALSE

    ; 打开SCM管理器
     invoke OpenSCManager, NULL, NULL, SC_MANAGER_ALL_ACCESS

      mov hSCManager,eax

       ;打开SCM管理器失败
       .if  !hSCManager

            PrintText<"OpenSCManager err!"> 
          invoke MessageBox, NULL, CTEXT("OpenSCManager err!"), NULL, MB_OK + MB_ICONSTOP

          mov bRet,FALSE

          jmp exit

       .endif

    ;打开驱动服务

    invoke OpenService,hSCManager,lpszSvrName,SERVICE_ALL_ACCESS ;

                  mov hService,eax

                ;打开驱动服务失败 
                .if !hService

                      PrintText<"OpenService error!"> 

                      PrintError  

                         mov bRet,FALSE
                      jmp exit
                .endif

    ; 停止驱动服务

      invoke ControlService, hService, SERVICE_CONTROL_STOP, addr SrvStatus

          ; 停止驱动服务失败 只有重新启动才能，再动态加载
           .if !eax

                      PrintText<"StopService error!"> 
                   PrintError

                .endif

          ;动态卸载服务,只做标记，服务停止时才被删除
       invoke DeleteService, hService
           mov bRet,eax
               ;卸载服务失败
           .if !eax

               PrintText<"DeleteService error!"> 
                   PrintError
                      mov bRet,FALSE
                   jmp exit
                .else

                    PrintText<"DeleteService OK!"> 

                .endif

    mov bRet,TRUE

    exit:

        .if   hService

               invoke CloseServiceHandle, hService

    .endif

        .if  hSCManager

            invoke CloseServiceHandle, hSCManager

       .endif

    mov eax,bRet

    ret

UnloadDriver endp

start proc

   invoke LoadDriver,$CTA0("beeper.sys"),$CTA0("beeper"),$CTA0("beeper")

    invoke UnloadDriver,$CTA0("beeper")

    invoke ExitProcess, 0

start endp
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
end start
```
