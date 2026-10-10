

# ff15 方式的 call 怎么计算地址

> 原文：[`www.pediy.com/kssd/pediy11/119396.html`](https://www.pediy.com/kssd/pediy11/119396.html)

比如  
b3d80908 ff15089ed9b3    call    dword ptr ds:\[0B3D99E08h\]  
它调到哪里去了,怎么算的,谢谢! 

if ( 0xE8 == code\_instr.opcode ) // 0xE8 call 类型  
{  
dwJumpTo = (DWORD)p + \*(DWORD\*)(p+1) + 5 ;  
}  
else if ( 0xFF == \*p && 0x15 == \*(p+1) ) // 0xFF15 call 类型  
{   
dwJumpTo = \*( DWORD\* )(\*( DWORD\* )( p + 2 ));  
}  
else if ( 0xFF == \*p && 0x14 == \*(p+1) ) // 0xFF14 call 数组\[Index\]  
{  
dwJumpTo = \*(DWORD\*)(p + 3) ;  
   
ProbeForRead( (PVOID)dwJumpTo, 4, 4 );  
dwJumpTo = \*( DWORD\* ) dwJumpTo ;  
}  
else if ( 0xFF == \*p && 0xD2 == \*(p+1) ) // 0xFFD2 类型, 不清楚是什么指令  
{  
dwJumpTo = (DWORD)(p + 2) ;   
}  
else if ( 0xFF == \*p && 0x12 == \*(p+1) && 0xba == \*(p-5) ) // 0xFF12 类型 call \[寄存器\]  
dwJumpTo = \*( DWORD\* )( p - 4 );  
ProbeForRead( (PVOID)dwJumpTo, 4, 4 );  
dwJumpTo = \*( DWORD\* ) dwJumpTo ;  
} 

Intel 64 and IA-32 Architectures Software Developer's Manuals(2008)![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")