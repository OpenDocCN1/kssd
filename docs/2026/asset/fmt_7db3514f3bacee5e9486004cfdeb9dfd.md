
``` 
__int64 sub_8C38()
{
_QWORD *v0;
// x8
pthread_mutex_t *v1;
// x0
__int64 v2;
// x0
int v3;
// w20
char *v4;
// x0
char v5;
// w0
size_t v6;
// w1
pthread_mutex_t *v7;
// x0
__int64 v8;
// x0
unsigned int v11;
// [xsp+10h] [xbp-1A0h]
__int64 v13;
// [xsp+20h] [xbp-190h]
_QWORD *v14;
// [xsp+40h] [xbp-170h]
int v15;
// [xsp+4Ch] [xbp-164h]

_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2));
v13 = sub_11444();
if ( v13 )
{
if ( ((v13 + 4095) & 0xFFFFFFFFFFFFF000LL) - v13 >= 0x208 )
v6 = 4096;
else
v6 = 0x2000;
if ( mprotect((void *)(v13 & 0xFFFFFFFFFFFFF000LL), v6, 3) < 0 )
{
return 0;
}
else
{
v1 = (pthread_mutex_t *)sub_BCEC();
v2 = pthread_mutex_lock(v1);
v3 = ((__int64 (__fastcall *)(__int64))sub_BCEC)(v2) + 40;
v4 = sub_BDF0();
v8 = sub_CC10(v3, v4);
if ( v8 )
{
if ( *(_DWORD *)(v8 + 12) == 0xCDEF2387 )
v0 = *(_QWORD **)(v8 + 16);
else
v0 = 0LL;
v14 = v0;
v15 = *off_43FB8;
if ( v15 < 23 )
{
*(_QWORD *)(v13 + 152) = v0[5];
*(_QWORD *)(v13 + 160) = v0[6];
*(_QWORD *)(v13 + 392) = v0[7];
*(_QWORD *)(v13 + 192) = v0[15];
*(_QWORD *)(v13 + 200) = v0[14];
v8 = sub_1F0A4(v0 + 16);
if ( (v8 & 1) != 0 )
{
*(_QWORD *)(v13 + 208) = v14[17];
*(_QWORD *)(v13 + 216) = v14[19];
*(_QWORD *)(v13 + 224) = v14[16];
*(_QWORD *)(v13 + 232) = v14[18];
}
*(_QWORD *)(v13 + 240) = v14[26];
*(_QWORD *)(v13 + 256) = v14[28];
*(_QWORD *)(v13 + 248) = v14[27] / 0x18uLL;
*(_QWORD *)(v13 + 264) = v14[29] / 0x18uLL;
*(_QWORD *)(v13 + 168) = 0LL;
}
else if ( v15 > 22 && v15 < 26 )
{
*(_QWORD *)(v13 + 24) = v0[5];
*(_QWORD *)(v13 + 32) = v0[6];
*(_QWORD *)(v13 + 264) = v0[7];
*(_QWORD *)(v13 + 64) = v0[15];
*(_QWORD *)(v13 + 72) = v0[14];
if ( (sub_1F0A4(v0 + 16) & 1) != 0 )
{
*(_QWORD *)(v13 + 80) = v14[17];
*(_QWORD *)(v13 + 88) = v14[19];
*(_QWORD *)(v13 + 96) = v14[16];
*(_QWORD *)(v13 + 104) = v14[18];
}
*(_QWORD *)(v13 + 112) = v14[26];
*(_QWORD *)(v13 + 128) = v14[28];
*(_QWORD *)(v13 + 120) = v14[27] / 0x18uLL;
*(_QWORD *)(v13 + 136) = v14[29] / 0x18uLL;
*(_QWORD *)(v13 + 40) = 0LL;
v8 = sub_1F2EC(v14 + 20);
if ( (v8 & 1) != 0 )
{
*(_QWORD *)(v13 + 352) = *((unsigned int *)v14 + 40);
*(_QWORD *)(v13 + 360) = v14[24];
*(_QWORD *)(v13 + 368) = v14[25] - 4LL * *((unsigned int *)v14 + 41);
*(_QWORD *)(v13 + 384) = v14[23];
*(_DWORD *)(v13 + 376) = *((_DWORD *)v14 + 43);
*(_DWORD *)(v13 + 380) = *((_DWORD *)v14 + 44);
}
}
else if ( v15 > 25 && v15 < 32 )
{
*(_QWORD *)(v13 + 16) = v0[5];
*(_QWORD *)(v13 + 24) = v0[6];
*(_QWORD *)(v13 + 256) = v0[7];
*(_QWORD *)(v13 + 56) = v0[15];
*(_QWORD *)(v13 + 64) = v0[14];
if ( (sub_1F0A4(v0 + 16) & 1) != 0 )
{
*(_QWORD *)(v13 + 72) = v14[17];
*(_QWORD *)(v13 + 80) = v14[19];
*(_QWORD *)(v13 + 88) = v14[16];
*(_QWORD *)(v13 + 96) = v14[18];
}
*(_QWORD *)(v13 + 104) = v14[26];
*(_QWORD *)(v13 + 120) = v14[28];
*(_QWORD *)(v13 + 112) = v14[27] / 0x18uLL;
*(_QWORD *)(v13 + 128) = v14[29] / 0x18uLL;
*(_QWORD *)(v13 + 32) = 0LL;
v8 = sub_1F2EC(v14 + 20);
if ( (v8 & 1) != 0 )
{
*(_QWORD *)(v13 + 344) = *((unsigned int *)v14 + 40);
*(_QWORD *)(v13 + 352) = v14[24];
*(_QWORD *)(v13 + 360) = v14[25] - 4LL * *((unsigned int *)v14 + 41);
*(_QWORD *)(v13 + 376) = v14[23];
*(_DWORD *)(v13 + 368) = *((_DWORD *)v14 + 43);
*(_DWORD *)(v13 + 372) = *((_DWORD *)v14 + 44);
}
}
else
{
*(_QWORD *)(v13 + 16) = v0[5];
*(_QWORD *)(v13 + 24) = v0[6];
*(_QWORD *)(v13 + 256) = v0[7];
*(_QWORD *)(v13 + 56) = v0[15];
*(_QWORD *)(v13 + 64) = v0[14];
v5 = sub_1F0A4(v0 + 16);
if ( (v5 & 1) != 0 )
{
*(_QWORD *)(v13 + 72) = v14[17];
*(_QWORD *)(v13 + 80) = v14[19];
*(_QWORD *)(v13 + 88) = v14[16];
*(_QWORD *)(v13 + 96) = v14[18];
}
*(_QWORD *)(v13 + 104) = v14[26];
*(_QWORD *)(v13 + 120) = v14[28];
*(_QWORD *)(v13 + 112) = v14[27] / 0x18uLL;
*(_QWORD *)(v13 + 128) = v14[29] / 0x18uLL;
*(_QWORD *)(v13 + 32) = 0LL;
v8 = sub_1F2EC(v14 + 20);
if ( (v8 & 1) != 0 )
{
*(_QWORD *)(v13 + 328) = *((unsigned int *)v14 + 40);
*(_QWORD *)(v13 + 336) = v14[24];
*(_QWORD *)(v13 + 344) = v14[25] - 4LL * *((unsigned int *)v14 + 41);
*(_QWORD *)(v13 + 360) = v14[23];
*(_DWORD *)(v13 + 352) = *((_DWORD *)v14 + 43);
*(_DWORD *)(v13 + 356) = *((_DWORD *)v14 + 44);
}
}
v11 = 1;
}
else
{
v11 = 0;
}
v7 = (pthread_mutex_t *)((__int64 (__fastcall *)(__int64))sub_BCEC)(v8);
pthread_mutex_unlock(v7);
return v11;
}
}
else
{
return 0;
}
}
```

这种方式处理的优点个人感觉就是流程准确度更高，缺点就是每碰到一个新的指令就需要增加对新指令的处理代码，如果指令处理出错也会出现问题，属于有点体力活的感觉，再则，当流程跑出来后如果有兴趣可以增加更加智能化的处理，比如自动识别虚假块，自动Patch等
