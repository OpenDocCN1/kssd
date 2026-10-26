
```c++
// 发送登录请求
int __stdcall login_request_412A20(char *passport, int a2)
{
LPSTR v2; // eax
__time64_t v3; // ST2C_8
int v4; // eax
int v5; // eax
int v6; // eax
int v7; // eax
int v8; // eax
int v9; // eax
int v10; // eax
int v11; // eax
int v12; // eax
int v13; // eax
LPSTR v14; // eax
int *v15; // edi
int *v16; // esi
int *v17; // ebx
int *v18; // eax
__time64_t v19; // ST28_8
int *v20; // ST20_4
int *v21; // ST08_4
int v22; // eax
int v23; // eax
int v24; // eax
int v25; // eax
int v26; // eax
int v27; // eax
int v28; // eax
int v29; // eax
int v30; // eax
int v31; // eax
int v32; // eax
_DWORD *v33; // eax
signed int v34; // edi
int v35; // eax
_DWORD *v36; // eax
int v37; // ebx
int v38; // eax
_DWORD *v39; // eax
int v40; // eax
_DWORD *v41; // eax
int v43; // [esp+0h] [ebp-38Ch]
char v44; // [esp+10h] [ebp-37Ch]
char v45; // [esp+20h] [ebp-36Ch]
char v46; // [esp+B8h] [ebp-2D4h]
char v47; // [esp+C8h] [ebp-2C4h]
int *v48; // [esp+160h] [ebp-22Ch]
int v49; // [esp+164h] [ebp-228h]
__time64_t v50; // [esp+168h] [ebp-224h]
char v51; // [esp+170h] [ebp-21Ch]
char v52; // [esp+178h] [ebp-214h]
int v53; // [esp+180h] [ebp-20Ch]
char v54; // [esp+184h] [ebp-208h]
char v55; // [esp+204h] [ebp-188h]
int v56; // [esp+220h] [ebp-16Ch]
char v57; // [esp+23Ch] [ebp-150h]
char v58; // [esp+258h] [ebp-134h]
CHAR v59; // [esp+274h] [ebp-118h]
char v60; // [esp+290h] [ebp-FCh]
char v61; // [esp+2ACh] [ebp-E0h]
int v62[7]; // [esp+2ECh] [ebp-A0h]
int MultiByteStr[7]; // [esp+308h] [ebp-84h]
char v64; // [esp+324h] [ebp-68h]
char v65; // [esp+340h] [ebp-4Ch]
int v66; // [esp+350h] [ebp-3Ch]
int v67; // [esp+354h] [ebp-38h]
char v68; // [esp+35Ch] [ebp-30h]
int v69; // [esp+36Ch] [ebp-20h]
int v70; // [esp+370h] [ebp-1Ch]
int *v71; // [esp+37Ch] [ebp-10h]
int v72; // [esp+388h] [ebp-4h]
v71 = &v43;
v49 = a2;
v53 = 1;
v72 = 0;
sub_4096D0(&v44);
LOBYTE(v72) = 1;
sub_4096D0(&v46);
v70 = 15;
v69 = 0;
v68 = 0;
LOBYTE(v72) = 3;
v50 = 1000 * _time64(0);
sub_51BA80(v62); // 获取各种信息
LOBYTE(v72) = 4;
v2 = W2A_51CF00((LPSTR)MultiByteStr);
LOBYTE(v72) = 5;
sub_40A7F0(v2, (int)&v68);
LOBYTE(v72) = 4;
sub_407400((int)MultiByteStr);
LOBYTE(v72) = 3;
sub_438100((int)v62);
v3 = v50;
v4 = sub_410AD0((int)&v45, "app_id=");
v5 = sub_410AD0(v4, "d0beea3fe1a1423392690dfd4d3713fd");
v6 = sub_410AD0(v5, "&passport=");
v7 = sub_4351B0(v6, passport);
v8 = sub_410AD0(v7, "&password=");
v9 = sub_4351B0(v8, (_DWORD *)passport + 7);
v10 = sub_410AD0(v9, "&platform=");
v11 = sub_4351B0(v10, &v68);
v12 = sub_410AD0(v11, "&timestamp=");
v13 = sub_434CA0(v12, v3, SHIDWORD(v3));
sub_410AD0(v13, "ZWtVdHB0WjBocFIxdmNzZlg3cUQ5SFhEYU5pWGsv");
sub_4073A0(MultiByteStr, "d0beea3fe1a1423392690dfd4d3713fd");
LOBYTE(v72) = 6;
sub_409840(&v44, (int)&v64); // 获取拼接完成待签名的字符串 [edx]
LOBYTE(v72) = 7;
sub_516850((int)&v56); // 数据签名
LOBYTE(v72) = 8;
v14 = W2A_51CF00(&v59);
LOBYTE(v72) = 9;
v15 = sub_51D080((int *)&v55, v14);
LOBYTE(v72) = 10;
v48 = sub_51D080((int *)&v58, &v68);
LOBYTE(v72) = 11;
v16 = sub_51D080((int *)&v57, (_DWORD *)passport + 7);
LOBYTE(v72) = 12;
v17 = sub_51D080((int *)&v60, passport);
LOBYTE(v72) = 13;
v18 = sub_51D080(v62, MultiByteStr);
LOBYTE(v72) = 14;
v19 = v50;
v20 = v48;
v21 = v18;
v22 = sub_410AD0((int)&v47, "app_id=");
v23 = sub_4351B0(v22, v21);
v24 = sub_410AD0(v23, "&passport=");
v25 = sub_4351B0(v24, v17);
v26 = sub_410AD0(v25, "&password=");
v27 = sub_4351B0(v26, v16);
v28 = sub_410AD0(v27, "&platform=");
v29 = sub_4351B0(v28, v20);
v30 = sub_410AD0(v29, "&timestamp=");
v31 = sub_434CA0(v30, v19, SHIDWORD(v19));
v32 = sub_410AD0(v31, "&sign=");
sub_4351B0(v32, v15);
LOBYTE(v72) = 13;
sub_407400((int)v62);
LOBYTE(v72) = 12;
sub_407400((int)&v60);
LOBYTE(v72) = 11;
sub_407400((int)&v57);
LOBYTE(v72) = 10;
sub_407400((int)&v58);
LOBYTE(v72) = 9;
sub_407400((int)&v55);
LOBYTE(v72) = 8;
sub_407400((int)&v59);
LOBYTE(v72) = 7;
sub_438100((int)&v56);
LOBYTE(v72) = 6;
sub_407400((int)&v64);
LOBYTE(v72) = 3;
sub_407400((int)MultiByteStr);
sub_4AE570(&v61);
v67 = 15;
v66 = 0;
v65 = 0;
LOBYTE(v72) = 16;
sub_4073A0(MultiByteStr, "https://pan-api.bitqiu.com/loginServer/login");
LOBYTE(v72) = 17;
v33 = (_DWORD *)sub_409840(&v46, (int)&v64);
LOBYTE(v72) = 18;
v34 = sub_4AEB40((int)&v61, MultiByteStr, v33, (unsigned int)&v65);
LOBYTE(v72) = 17;
sub_407400((int)&v64);
LOBYTE(v72) = 16;
sub_407400((int)MultiByteStr);
if ( !v34 )
{
sub_521840(0);
LOBYTE(v72) = 19;
sub_524230(&v54);
LOBYTE(v72) = 20;
if ( !(unsigned __int8)sub_524310((int)&v65, &v54, (int)&v51, 1) )
{
v53 = 0;
LABEL_19:
LOBYTE(v72) = 19;
sub_402AC0(&v54);
goto LABEL_20;
}
if ( (unsigned __int8)sub_522200(&v51) )
{
v53 = 0;
LOBYTE(v72) = 19;
sub_402AC0(&v54);
}
else
{
if ( v52 == 7 )
{
if ( (unsigned __int8)sub_522A70("message") && *(_BYTE *)(sub_522700(&v51, "message") + 8) == 4 )
{
v35 = sub_522700(&v51, "message");
v36 = (_DWORD *)sub_521E20(v35);
LOBYTE(v72) = 21;
v37 = v49;
sub_40A7F0(v36, v49 + 28);
LOBYTE(v72) = 20;
sub_407400((int)&v64);
}
else
{
v37 = v49;
}
if ( (unsigned __int8)sub_522A70("code") && *(_BYTE *)(sub_522700(&v51, "code") + 8) == 4 )
{
v38 = sub_522700(&v51, "code");
v39 = (_DWORD *)sub_521E20(v38);
LOBYTE(v72) = 22;
sub_40A7F0(v39, v37);
LOBYTE(v72) = 20;
sub_407400((int)&v64);
}
if ( (unsigned __int8)sub_522A70("data") && *(_BYTE *)(sub_522700(&v51, "data") + 8) == 7 )
{
v40 = sub_522700(&v51, "data");
v41 = (_DWORD *)sub_522D00(v40);
LOBYTE(v72) = 23;
sub_40A7F0(v41, v37 + 56);
LOBYTE(v72) = 20;
sub_407400((int)&v64);
}
goto LABEL_19;
}
v53 = 0;
LOBYTE(v72) = 19;
sub_402AC0(&v54);
}
LABEL_20:
LOBYTE(v72) = 16;
sub_521BF0(&v51);
goto LABEL_21;
}
v53 = 256;
LABEL_21:
LOBYTE(v72) = 15;
sub_407400((int)&v65);
LOBYTE(v72) = 3;
sub_4AE810(&v61);
LOBYTE(v72) = 2;
sub_407400((int)&v68);
LOBYTE(v72) = 1;
sub_403F30(&v46);
LOBYTE(v72) = 0;
sub_403F30(&v44);
return v53;
}
```

在充满各种`v`开头的符号中间，字符串总是显得如此亲切。都是`&k=v`的形式，很明显这里是拼接http请求参数的地方，大多数参数我们已经知道了，我们只重点看`platform`和`sign`参数即可。跟到这里，就没有捷径可走了，只能耐下性子一点点分析。

### `platform`参数解析

```c++
// 获取系统信息并拼接为字符串
// [app_id];[exe_version];[mac];pc;[computer_name];[sys_version]
//
_DWORD *__thiscall sub_51BA80(_DWORD *this)
{
_DWORD *v1; // edi
int v2; // ecx
_DWORD *v3; // eax
int v4; // ecx
int v5; // ecx
_DWORD *v6; // eax
int v7; // ecx
int v8; // ecx
int v9; // ecx
int v10; // ecx
_DWORD *v11; // eax
int v12; // ecx
int v13; // ecx
_DWORD *v14; // eax
int v15; // ecx
int v17; // [esp+0h] [ebp-48h]
int v18; // [esp+10h] [ebp-38h]
_DWORD *v19; // [esp+14h] [ebp-34h]
char v20; // [esp+18h] [ebp-30h]
int *v21; // [esp+38h] [ebp-10h]
int v22; // [esp+44h] [ebp-4h]
v21 = &v17;
v1 = this;
v19 = this;
this[5] = 7;
this[4] = 0;
*(_WORD *)this = 0;
v18 = 1;
v22 = 1;
sub_408120(L"d0beea3fe1a1423392690dfd4d3713fd", (int)this);
sub_411600(v2, L";", (int)v1);
v3 = get_exe_version_51B7B0(&v20);
LOBYTE(v22) = 2;
if ( v3[5] >= 8u )
v3 = (_DWORD *)*v3;
sub_411600(v4, v3, (int)v1);
LOBYTE(v22) = 1;
sub_438100((int)&v20); // 重置缓冲区
sub_411600(v5, L";", (int)v1);
v6 = (_DWORD *)get_mac_51B1F0((int)&v20);
LOBYTE(v22) = 3;
if ( v6[5] >= 8u )
v6 = (_DWORD *)*v6;
sub_411600(v7, v6, (int)v1);
LOBYTE(v22) = 1;
sub_438100((int)&v20);
sub_411600(v8, L";", (int)v1);
sub_411600(v9, L"pc", (int)v1);
sub_411600(v10, L";", (int)v1);
v11 = (_DWORD *)get_computer_name_51B390(&v20);
LOBYTE(v22) = 4;
if ( v11[5] >= 8u )
v11 = (_DWORD *)*v11;
sub_411600(v12, v11, (int)v1); // 增加计算机名
LOBYTE(v22) = 1;
sub_438100((int)&v20);
sub_411600(v13, L";", (int)v1);
v14 = (_DWORD *)get_sysver_string_51B460(&v20);
LOBYTE(v22) = 5;
if ( v14[5] >= 8u )
v14 = (_DWORD *)*v14;
sub_411600(v15, v14, (int)v1);
LOBYTE(v22) = 1;
sub_438100((int)&v20);
return v1;
}
```

都是体力活，没啥好说的，一步步跟就好了，只要找准了缓冲区地址，就很开心了，使用内存窗口能看到字符串逐渐累加，感觉自己离胜利越来越近。

![系统信息拼接](upload/attach/202009/903028_37ZK4R234AXUHEH.png)
