
```c
int _tmain(int argc, _TCHAR* argv[])  
{  
  void *PEB = NULL,  
    *Ldr = NULL,  
    *Flink = NULL,  
    *p = NULL,  
    *BaseAddress = NULL,  
    *FullDllName = NULL;  
   
  __asm  
  {  
    mov eax,fs:[0x30]  
    mov PEB,eax  
  }  
  Ldr = * ( ( void ** ) ( ( unsigned char * )PEB+0x0c ) );  
  Flink = * ( ( void ** ) ( ( unsigned char * )Ldr+ 0x1c ) );  
  p = Flink;  
  do  
  {  
    BaseAddress = * ( ( void ** ) ( ( unsigned char * )p+ 0x08 ) );  
    FullDllName = * ( ( void ** ) ( ( unsigned char * )p+ 0x18) );  
    wprintf(L"FullDllName is %s\n",FullDllName);  
    printf("BaseAddress is %x\n",BaseAddress);  
    p = * ( ( void ** )p);  
  }  
  while ( Flink != p );  
  return 0;  
}
```

我是个菜鸟，以上是我的一点儿愚见，如有错误，希望能善意指出，谅解。

`4dd466d0e1e0cc123afbe504da2462ca`
