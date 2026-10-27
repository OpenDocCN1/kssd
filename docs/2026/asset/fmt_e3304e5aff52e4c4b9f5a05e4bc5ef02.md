
# [原创]Minlink Engine V1.0

RT.顺便妹子.

简介一下几个函数.没使用反汇编引擎,因此遇到特定的指令序列的时候会出错.测试了10个程序左右.我运气好,只有一个出错了.

## 计算对其值

```
unsigned long alignment(unsigned long size,unsigned long align)

{

	if(size%align!=0)

		return  (size/align+1)*align;

	return size;

}
```

## 变形 `push xxxxxxxx`

```
void _link_push_imm(void * dst,size_t _new_push_rva,void * src,size_t _code_push_rva,size_t _image_base_offset)

{

	/*

	    jmp     xxxxxxxx

        sub     esp, 4

        mov     dword ptr [esp], -1

        jmp     FFFFFFFF

	*/

	unsigned char _rand_reg[] = {0x50, 0x51, 0x52, 0x53, 0x53, 0x54, 0x55, 0x56, 0x57};

	unsigned char push[] = {0x50, 

		                    0xC7, 0x04, 0x24, 0x00, 0x00, 0x00, 0x00, 

                            0xE9, 0x00, 0x00, 0x00, 0x00};

	unsigned char * pdst = (unsigned char *) dst;

	unsigned char * psrc = (unsigned char *) src;

	unsigned long _next_opcode_va = ((unsigned long)psrc + 5 + _code_push_rva + _image_base_offset);

	if(*psrc == 0x68)

	{

		/*mov     dword ptr [esp], xxxxxxxx*/

		memmove(push + 4,psrc + 1,sizeof(unsigned long));

		int index = clock() % 9;

		memset(push,_rand_reg[index],sizeof(unsigned char));

		/*jmp     xxxxxxxx*/

		memset(psrc,0xE9,sizeof(unsigned char));

	    *(unsigned long *) (psrc + 1)  = (unsigned long) (pdst + _new_push_rva + _image_base_offset - _next_opcode_va);

		/*jmp     xxxxxxxx*/

		*(unsigned long *)(push + 9) = (_next_opcode_va - ((unsigned long)pdst + 8 + _new_push_rva  + _image_base_offset + 5));

		memmove(pdst,push,0xD);

	}

}
```

## 变形`call [xxxxxxxx] /*FF15 XXXXXXXX*/`

```
void _link_call_mem(void * dst,size_t _new_push_rva,void * src,size_t _code_push_rva,size_t _image_base_offset)

{

	unsigned long _next_address = 0;

	unsigned char _rand_reg[] = {0x50, 0x51, 0x52, 0x53, 0x53, 0x54, 0x55, 0x56, 0x57};

	unsigned char call[] = {0x50, 

		                    0xC7, 0x04, 0x24, 0x00, 0x00, 0x00, 0x00, 

		                    0xFF, 0x25, 0x00, 0x00, 0x00, 0x00, 

		                    0xE9, 0x00, 0x00, 0x00, 0x00};

	unsigned char * pdst = (unsigned char *) dst;

	unsigned char * psrc = (unsigned char *) src;

	unsigned long _next_opcode_va = ((unsigned long)psrc + 5 + _code_push_rva + _image_base_offset);



	if(*(unsigned short *)psrc == 0x15ff)

	{

		Sleep(1);

		int index = rand() % 9;

		memset(call,_rand_reg[index],sizeof(unsigned char));

		memmove(call + 10,psrc + 2,sizeof(unsigned long));



		memset(psrc,0xE9,sizeof(unsigned char));

		*(unsigned long *) (psrc + 1)  = (unsigned long) (pdst + _new_push_rva + _image_base_offset - _next_opcode_va);

		memset(psrc + 5,0x90,sizeof(unsigned char));

		/*mov     dword ptr [esp], -1*/

		_next_address = (unsigned long) ((unsigned long)pdst + 14 + _new_push_rva  + _image_base_offset);

		memmove((void *)(call + 4),(void *)&_next_address,sizeof(unsigned long));

		_next_address = (unsigned long) (_next_opcode_va - ((unsigned long)pdst + 14 + _new_push_rva  + _image_base_offset + 5));

		memmove((void *)(call + 15),(void *)&_next_address,sizeof(unsigned long));

		memmove(pdst,call,19);

	}

}
```

## 变形 `call [xxxxxxxx]./*call......jmp*/`

```
void _link_call_rva(void * dst,size_t _new_push_rva,void * src,size_t _code_push_rva,size_t _image_base_offset)

{

	/*

	    jmp     _new_adderss

        push    exx

	    mov     dword ptr [esp], -1

        jmp     dword ptr[xxxxxxxx]

_next:

        jmp     _ret_address

	*/

	unsigned char _rand_reg[] = {0x50, 0x51, 0x52, 0x53, 0x53, 0x54, 0x55, 0x56, 0x57};

	unsigned char call[] = {0x50, 

		                    0xC7, 0x04, 0x24, 0x00, 0x00, 0x00, 0x00,

		                    0xE9, 0x00, 0x00, 0x00, 0x00, 

                            0xE9, 0x00, 0x00, 0x00, 0x00};

	unsigned char * pdst = (unsigned char *) dst;

	unsigned char * psrc = (unsigned char *) src;

	unsigned long _next_opcode_va = ((unsigned long)psrc + 5 + _code_push_rva + _image_base_offset);

	if(*psrc == 0xE8 && *(unsigned short *)(psrc + 3) == 0)

	{

		/*jmp     _new_adderss*/

		unsigned long _jmp_va = ((unsigned long)psrc + (*(unsigned long*)(psrc + 1)) + 6 + _code_push_rva  + _image_base_offset); 

		memset(psrc,0xE9,sizeof(unsigned char));

		*(unsigned long *) (psrc + 1)  = (unsigned long) (pdst + _new_push_rva + _image_base_offset - _next_opcode_va);

		/*push    exx*/

		Sleep(1);

		int index = clock() % 9;

		memset(call,_rand_reg[index],sizeof(unsigned char));

		/*mov     dword ptr [esp], -1*/

		unsigned long _ret_address = (unsigned long) ((unsigned long)pdst + 13 + _new_push_rva  + _image_base_offset);

		memmove((void *)(call + 4),(void *)&_ret_address,sizeof(unsigned long));

		/*jmp     dword ptr[xxxxxxxx]*/

		_jmp_va -= ((unsigned long)pdst + 9 + _new_push_rva  + _image_base_offset + 5);

		memmove((void *)(call + 9),(void *)&_jmp_va,sizeof(unsigned long));

		/*jmp     _ret_address*/

		*(unsigned long *)(call + 14) = (_next_opcode_va - ((unsigned long)pdst + 13 + _new_push_rva  + _image_base_offset + 5));

		memmove(pdst,call,18);

	}

}
```

## 主函数
