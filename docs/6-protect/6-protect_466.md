

# 无视 DebugPort 清零

> 原文：[`www.pediy.com/kssd/pediy12/133382.html`](https://www.pediy.com/kssd/pediy12/133382.html)

可行否。。  
  
实现太琐碎了，不行不写了 ![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")  
  

代码:

```
DEBUG\_OBJECT g\_DebugObject; //only Debugger! just for test.

//
//HookRoutine of DbgkForwardException
//
BOOLEAN
DbgForwardException(
    IN PEXCEPTION\_RECORD ExceptionRecord,
    IN BOOLEAN DebugException,      
    IN BOOLEAN SecondChance
    )
{
  BOOLEAN  Handled = FALSE;

  if(!DebugException)
    return OrgDbgForwardException(ExceptionRecord,DebugException,SecondChange);
  //
  //Queue the DebugEvent  and Wakeup the Debugger
  //

  //
  //Wait For DebugEvent->ContinueEvent
  //

  //
  //return whether the Exception was Handled
  //TRUE  -  handled
  //

  return Handled;
}


//
//HookRoutine of NtWaitForDebugEvent
//
NTSTATUS
DbgWaitForDebugEvent (
    IN HANDLE DebugObjectHandle,
    IN BOOLEAN Alertable,
    IN PLARGE\_INTEGER Timeout OPTIONAL,
    OUT PDBGUI\_WAIT\_STATE\_CHANGE WaitStateChange
    )
{
  NTSTATUS Status;
    KPROCESSOR\_MODE PreviousMode = ExGetPreviousMode();
     PDEBUG\_OBJECT DebugObject = &g\_DebugObject;

  //
  //Waiting For wakeup
  //

  //
  //Get DebugEvent and Convert to WaitStateChange
  //

  return  Status;
}

//
//HookRoutine of NtDebugContinue
//
NTSTATUS
DbgDebugContinue (
    IN HANDLE DebugObjectHandle,
    IN PCLIENT\_ID ClientId,
    IN NTSTATUS ContinueStatus
    )
{
  //
  //wakeup the Debuggee
  //
}

//
//HookRoutine of NtCreateDebugObject
//
NTSTATUS
DbgCreateDebugObject (
    OUT PHANDLE DebugObjectHandle,
    IN ACCESS\_MASK DesiredAccess,
    IN POBJECT\_ATTRIBUTES ObjectAttributes,
    IN ULONG Flags
    )
{
  //
  //initialize the g\_DebugObject and just return Success
  //
}

//
//HookRoutine of NtDebugActiveProcess
//
NTSTATUS
DbgDebugActiveProcess (
    IN HANDLE ProcessHandle,
    IN HANDLE DebugObjectHandle
    )
{
  //
  //initialize the g\_DebugObject and just return Success
  //
}

```