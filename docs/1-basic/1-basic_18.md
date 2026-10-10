

# 常用转移指令整理

> 原文：[`www.pediy.com/kssd/pediy11/125047.html`](https://www.pediy.com/kssd/pediy11/125047.html)

转移分类：  
1. 段内转移：只修改 IP  
    a. 短转移：-128~127  
    b. 近转移：-32768~32767  
2. 段间转移：修改 CS 和 IP  
  
  
1. offset 标号  
   取得标号的偏移地址  
  
2. 段内短转移  
   jmp short 标号 (IP)=(IP)+8 位位移  
   转到标号处执行（并不包含转移的目的地址而是转移的位移）  
     
   段内近转移  
   jmp near ptr 标号 (IP)=(IP)+16 位位移  
  
3. 段间转移（远转移）  
   jmp far ptr 标号  
   (CS)=标号所在段的段地址；(IP)=标号在段中的偏移地址  
  
4. jmp 16 位 reg  
   (IP)=(16 位 reg)  
  
5. jmp word ptr 内存单元地址（段内转移）  
   (IP)=内存单元开头一个字  
  
   jmp dword ptr 内存单元地址（段间转移）  
   (CS)=(内存单元地址+2)  
   (IP)=(内存单元地址)  
  
6. jcxz 标号  
   if((cx)==0) jmp short 标号  
  
7. loop 标号  
   (cx)--;  
   if((cx)!=0) jmp short 标号  
  
8. ret  
   相当于：pop IP  
   retf  
   相当于：pop IP  
           pop CS  
  
9. call 标号  
   相当于：push IP  
           jmp near ptr 标号  
  
   call far ptr 标号  
   相当于： push CS  
            push IP  
            jmp far ptr 标号  
  
10. call 16 位 reg  
    相当于：push IP  
            jmp 16 位 reg  
  
11. call word ptr 内存单元地址  
    相当于：push IP  
            jmp word ptr 内存单元地址  
  
    call dword ptr 内存单元地址  
    相当于：push CS  
            push IP  
            jmp dword ptr 内存单元地址