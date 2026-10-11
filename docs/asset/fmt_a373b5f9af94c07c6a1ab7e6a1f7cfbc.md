
```
case   R\_WH\_FOREGROUNDIDLE: printf("Handle:%x; Type:WH\_FOREGROUNDIDLE, FunAddr:%x\\n", `pHook->tshead.ThreadObjHead.headinfo.h`, `pHook->offPfn`);break;

case   R\_WH\_CALLWNDPROCRET: printf("Handle:%x; Type:WH\_KEYBOARD\_LL, FunAddr:%x\\n", `pHook->tshead.ThreadObjHead.headinfo.h`, `pHook->offPfn`);break;

case   R\_WH\_KEYBOARD\_LL: printf("Handle:%x; Type:WH\_MSGFILTER, FunAddr:%x\\n", `pHook->tshead.ThreadObjHead.headinfo.h`, `pHook->offPfn`);break;

case   R\_WH\_MOUSE\_LL: printf("Handle:%x; Type:WH\_MOUSE\_LL, FunAddr:%x\\n", `pHook->tshead.ThreadObjHead.headinfo.h`, `pHook->offPfn`);break;

default: printf("Unknown\\n");

}

}//if

}

//////////////////////////////////////////////////////////////////////////

`VirtualFree`(`pHook`, `sizeof(HOOK)`, `MEM\_DECOMMIT`);

`VirtualFree`(`pHandleEntryR`, `sizeof(HANDLEENTRY)` * `ServerInfo.cHandleEntries`, `MEM\_DECOMMIT`);

`\_getch`();

return 0;

}
```
