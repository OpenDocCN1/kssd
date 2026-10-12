
# 无视 DebugPort 清零

> 原文：[`www.pediy.com/kssd/pediy13/133382.html`](https://www.pediy.com/kssd/pediy13/133382.html)

可行否。

实现太琐碎了，不行不写了 ![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")

代码:

```c
DEBUG_OBJECT g_DebugObject; //only Debugger! just for test.

//
//HookRoutine of DbgkForwardException
//
BOOLEAN
DbgForwardException(
    IN PEXCEPTION_RECORD ExceptionRecord,
    IN BOOLEAN DebugException,      
    IN BOOLEAN SecondChance
    )
{
  BOOLEAN  Handled = FALSE;

  if(!DebugException)
    return OrgDbgForwardException(ExceptionRecord,DebugException,SecondChange);
  //
  //Queue the DebugEvent  and Wakeup the Debugger
  //

  //
  //Wait For DebugEvent->ContinueEvent
  //

  //
  //return whether the Exception was Handled
  //TRUE  -  handled
  //

  return Handled;
}

//
//HookRoutine of NtWaitForDebugEvent
//
NTSTATUS
DbgWaitForDebugEvent (
    IN HANDLE DebugObjectHandle,
    IN BOOLEAN Alertable,
    IN PLARGE_INTEGER Timeout OPTIONAL,
    OUT PDBGU_WAIT_STATE_CHANGE WaitStateChange
    )
{
  NTSTATUS Status;
    KPROCESSOR_MODE PreviousMode = ExGetPreviousMode();
     PDEBUG_OBJECT DebugObject = &g_DebugObject;

  //
  //Waiting For wakeup
  //

  //
  //Get DebugEvent and Convert to WaitStateChange
  //

  return  Status;
}

//
//HookRoutine of NtDebugContinue
//
NTSTATUS
DbgDebugContinue (
    IN HANDLE DebugObjectHandle,
    IN PCLIENT_ID ClientId,
    IN NTSTATUS ContinueStatus
    )
{
  //
  //wakeup the Debuggee
  //
}

//
//HookRoutine of NtCreateDebugObject
//
NTSTATUS
DbgCreateDebugObject (
    OUT PHANDLE DebugObjectHandle,
    IN ACCESS_MASK DesiredAccess,
    IN POBJECT_ATTRIBUTES ObjectAttributes,
    IN ULONG Flags
    )
{
  //
  //initialize the g_DebugObject and just return Success
  //
}

//
//HookRoutine of NtDebugActiveProcess
//
NTSTATUS
DbgDebugActiveProcess (
    IN HANDLE ProcessHandle,
    IN HANDLE DebugObjectHandle
    )
{
  //
  //initialize the g_DebugObject and just return Success
  //
}
```
