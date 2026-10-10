

# 时间限制

> 原文：[`www.pediy.com/kssd/tutorial/chap6-1-31.htm`](https://www.pediy.com/kssd/tutorial/chap6-1-31.htm)

**时间限制**　**【习题】**

**1、习题一** chap6-1-3-01 153K 每次 20 秒使用时间 易

**2、习题二** chap6-1-3-02 15K 时间限制 ( 需 cw3220.dll ) 易

 

 1、习题一 答案         

| 这程序用上了 SetTimer 函数，设置断点：bpx setimer,将中断如下：   
  :0043D3DF  66837B2E00  CMP    WORD PTR \[EBX+2E\],00   
  :0043D3E4  7433        JZ      0043D419   
  :0043D3E6  6A00        PUSH    00                    ;处理定时过程的程序入口地址  
  :0043D3E8  56          PUSH    ESI                  ;暂停时间值 (21 seconds)   
  :0043D3E9  6A01        PUSH    01                    ; 定时器 ID 标识符  
  :0043D3EB  8B4328      MOV    EAX,\[EBX+28\]   
  :0043D3EE  50          PUSH    EAX                  ; 句柄   
  :0043D3EF  E8648FFCFF  CALL    USER32!SetTimer   
  :0043D3F4  85C0        TEST    EAX,EAX              ; 时间是否到期?   
  :0043D3F6  7521        JNZ    0043D419              ; 没到 JMP   
    ...      ...   
  :0043D419  33C0        XOR    EAX,EAX   
  
现在我们可修改:0043D3E4 一行，让其直接跳过 SetTimer 函数，这样就没时间限制了。   
将： JZ 0043D419 改成 JMP 0043D419   
66837B2E007433 改成 66837B2E00EB33 即可。  |
| --- |

 2、习题二 答案        

| 这是个时间限制程序。   
一运行程序，我注意到跳出一对话框，说：trial-period has ended，这对话是调用 messagebox 函数的，今天我们不用此函数设断，这程序肯定调用了取系统时间函数，这样才能判断当前时间是否过期。因此用 GetLocalTime 函数设断。   
bpx GetLocalTime   
中断如下：   
  
\* Reference To: cw3220.\_getdate, Ord:0000h   
                                  |   
:0040108A E83F040000              Call 004014CE  ->GetLocalTime 函数   
:0040108F 59                      pop ecx   
:00401090 817DF8CD070000          cmp dword ptr \[ebp-08\], 000007CD   
//这一行觉得熟吧，7CD 的十进制是 1997,而当前时间在 EAX 里，是 07D0   
//   
:00401097 7F0F                    jg 004010A8 ->如当前日期大于 1997 则出错。   
  
剩下的工作太简单了，结束。  |
| --- |

    

 

 4dd466d0e1e0cc123afbe504da2462ca