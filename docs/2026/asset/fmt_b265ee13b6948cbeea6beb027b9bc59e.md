
```
void DrawMenu()

{

  int v0; // eax

  LPVOID v1; // eax

  char *v2; // eax

  LPVOID v3; // eax

  LPVOID v4; // ST0C_4

  int v5; // ST08_4

  int v6; // eax

  LPVOID v7; // eax

  char *v8; // eax

  LPVOID v9; // eax

  LPVOID v10; // ST0C_4

  int v11; // ST08_4

  int v12; // eax

  LPVOID v13; // eax

  char *v14; // eax

  LPVOID v15; // eax

  LPVOID v16; // ST0C_4

  int v17; // ST08_4

  int v18; // eax

  LPVOID v19; // eax

  char *v20; // eax

  LPVOID v21; // eax

  LPVOID v22; // ST0C_4

  int v23; // ST08_4

  int v24; // eax

  LPVOID v25; // eax

  char *v26; // eax

  LPVOID v27; // eax

  LPVOID v28; // ST0C_4

  int v29; // ST08_4

  int v30; // eax

  LPVOID v31; // eax

  char *v32; // eax

  LPVOID v33; // eax

  LPVOID v34; // ST0C_4

  int v35; // ST08_4

  int v36; // eax

  LPVOID v37; // eax

  char *v38; // eax

  int v39; // eax

  int v40; // ST0C_4

  int v41; // ST08_4

  int v42; // eax

  LPVOID v43; // eax

  char *v44; // eax

  LPVOID v45; // eax

  LPVOID v46; // ST0C_4

  int v47; // ST08_4

  int v48; // eax

  LPVOID v49; // eax

  char *v50; // eax

  LPVOID v51; // eax

  LPVOID v52; // ST0C_4

  int v53; // ST08_4

  int v54; // eax

  LPVOID v55; // eax

  char *v56; // eax

  LPVOID v57; // eax

  LPVOID v58; // ST0C_4

  int v59; // ST08_4

  int v60; // eax

  int v61; // [esp+4h] [ebp-34h]

  int v62; // [esp+Ch] [ebp-2Ch]

  int v63; // [esp+Ch] [ebp-2Ch]

  int v64; // [esp+Ch] [ebp-2Ch]

  int v65; // [esp+Ch] [ebp-2Ch]

  int v66; // [esp+Ch] [ebp-2Ch]

  int v67; // [esp+Ch] [ebp-2Ch]

  int v68; // [esp+Ch] [ebp-2Ch]

  int v69; // [esp+Ch] [ebp-2Ch]

  int v70; // [esp+Ch] [ebp-2Ch]

  int v71; // [esp+Ch] [ebp-2Ch]

  LPVOID v72[2]; // [esp+14h] [ebp-24h]

  double v73; // [esp+1Ch] [ebp-1Ch]

  LPVOID lpMem[2]; // [esp+24h] [ebp-14h]

  LPVOID StrSwitch; // [esp+2Ch] [ebp-Ch]

  int v76; // [esp+30h] [ebp-8h]

  int v77; // [esp+34h] [ebp-4h]



  v77 = 10;

  v76 = 20;

  StrSwitch = 0;

  v0 = sub_4065DD(255, 0, 0);

  lpMem[0] = "    T s";

  DrawText(lpMem, 10, 20, v0);

  if ( lpMem[0] )

    FreeMem(lpMem[0]);

  v76 += 25;

  StrSwitch = 0;

  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);

  lpMem[0] = 0;

  HIDWORD(v73) = sub_4065DD(255, 0, 0);

  if ( Switch_Box )

    v1 = lpMem[1];

  else

    v1 = (LPVOID)HIDWORD(v73);

  v72[1] = "显示方框F1";

  DrawText(&v72[1], v77, v76, v1);              // 这里是直接把文字显示

  if ( v72[1] )

    FreeMem(v72[1]);

  if ( Switch_Box )                             // 判断控制该功能的变量是否为真

    v2 = "开";

  else

    v2 = "关";

  StrSwitch = v2;

  *(double *)lpMem = (double)v77;

  v73 = *(double *)lpMem + 75.0;

  v72[1] = 0;

  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);

  v62 = sub_4065DD(255, 0, 0);

  if ( Switch_Box )

    v3 = v72[0];

  else

    v3 = (LPVOID)v62;

  v4 = v3;

  v5 = v76;

  v6 = toInt64(v73);

  DrawText(&StrSwitch, v6, v5, v4);             // 把结果显示出去

  if ( StrSwitch )

    FreeMem(StrSwitch);

  v76 += 25;

  StrSwitch = 0;

  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);

  lpMem[0] = 0;

  HIDWORD(v73) = sub_4065DD(255, 0, 0);

  if ( Switch_Hp )

    v7 = lpMem[1];

  else

    v7 = (LPVOID)HIDWORD(v73);

  v72[1] = "显示血量F2";

  DrawText(&v72[1], v77, v76, v7);

  if ( v72[1] )

    FreeMem(v72[1]);

  if ( Switch_Hp )

    v8 = "开";

  else

    v8 = "关";

  StrSwitch = v8;

  *(double *)lpMem = (double)v77;

  v73 = *(double *)lpMem + 75.0;

  v72[1] = 0;

  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);

  v63 = sub_4065DD(255, 0, 0);

  if ( Switch_Hp )

    v9 = v72[0];

  else

    v9 = (LPVOID)v63;

  v10 = v9;

  v11 = v76;

  v12 = toInt64(v73);

  DrawText(&StrSwitch, v12, v11, v10);

  if ( StrSwitch )

    FreeMem(StrSwitch);

  v76 += 25;

  StrSwitch = 0;

  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);

  lpMem[0] = 0;

  HIDWORD(v73) = sub_4065DD(255, 0, 0);

  if ( Switch_Line )

    v13 = lpMem[1];

  else

    v13 = (LPVOID)HIDWORD(v73);

  v72[1] = "显示射线F3";

  DrawText(&v72[1], v77, v76, v13);

  if ( v72[1] )

    FreeMem(v72[1]);

  if ( Switch_Line )

    v14 = "开";

  else

    v14 = "关";

  StrSwitch = v14;

  *(double *)lpMem = (double)v77;

  v73 = *(double *)lpMem + 75.0;

  v72[1] = 0;

  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);

  v64 = sub_4065DD(255, 0, 0);

  if ( Switch_Line )

    v15 = v72[0];

  else

    v15 = (LPVOID)v64;

  v16 = v15;

  v17 = v76;

  v18 = toInt64(v73);

  DrawText(&StrSwitch, v18, v17, v16);

  if ( StrSwitch )

    FreeMem(StrSwitch);

  v76 += 25;

  StrSwitch = 0;

  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);

  lpMem[0] = 0;

  HIDWORD(v73) = sub_4065DD(255, 0, 0);

  if ( Switch_Aim )

    v19 = lpMem[1];

  else

    v19 = (LPVOID)HIDWORD(v73);

  v72[1] = "右键自瞄F4";

  DrawText(&v72[1], v77, v76, v19);

  if ( v72[1] )

    FreeMem(v72[1]);

  if ( Switch_Aim )

    v20 = "开";

  else

    v20 = "关";

  StrSwitch = v20;

  *(double *)lpMem = (double)v77;

  v73 = *(double *)lpMem + 75.0;

  v72[1] = 0;

  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);

  v65 = sub_4065DD(255, 0, 0);

  if ( Switch_Aim )

    v21 = v72[0];

  else

    v21 = (LPVOID)v65;

  v22 = v21;

  v23 = v76;

  v24 = toInt64(v73);

  DrawText(&StrSwitch, v24, v23, v22);

  if ( StrSwitch )

    FreeMem(StrSwitch);

  v76 += 25;

  StrSwitch = 0;

  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);

  lpMem[0] = 0;

  HIDWORD(v73) = sub_4065DD(255, 0, 0);

  if ( Switch_Breath )

    v25 = lpMem[1];

  else

    v25 = (LPVOID)HIDWORD(v73);

  v72[1] = "开启屏息F5";

  DrawText(&v72[1], v77, v76, v25);

  if ( v72[1] )

    FreeMem(v72[1]);

  if ( Switch_Breath )

    v26 = "开";

  else

    v26 = "关";

  StrSwitch = v26;

  *(double *)lpMem = (double)v77;               // v78=10.0f

  v73 = *(double *)lpMem + 75.0;

  v72[1] = 0;

  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);

  v66 = sub_4065DD(255, 0, 0);

  if ( Switch_Breath )

    v27 = v72[0];

  else

    v27 = (LPVOID)v66;

  v28 = v27;

  v29 = v76;

  v30 = toInt64(v73);

  DrawText(&StrSwitch, v30, v29, v28);

  if ( StrSwitch )

    FreeMem(StrSwitch);

  v76 += 25;

  StrSwitch = 0;

  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);

  lpMem[0] = 0;

  HIDWORD(v73) = sub_4065DD(255, 0, 0);

  if ( Switch_HightJump )

    v31 = lpMem[1];

  else

    v31 = (LPVOID)HIDWORD(v73);

  v72[1] = "人物高跳F6";

  DrawText(&v72[1], v77, v76, v31);

  if ( v72[1] )

    FreeMem(v72[1]);

  if ( Switch_HightJump )

    v32 = "开";

  else

    v32 = "关";

  StrSwitch = v32;

  *(double *)lpMem = (double)v77;

  v73 = *(double *)lpMem + 75.0;

  v72[1] = 0;

  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);

  v67 = sub_4065DD(255, 0, 0);

  if ( Switch_HightJump )

    v33 = v72[0];

  else

    v33 = (LPVOID)v67;

  v34 = v33;

  v35 = v76;

  v36 = toInt64(v73);

  DrawText(&StrSwitch, v36, v35, v34);

  if ( StrSwitch )

    FreeMem(StrSwitch);

  v76 += 25;

  StrSwitch = 0;

  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);

  lpMem[0] = 0;

  HIDWORD(v73) = sub_4065DD(255, 0, 0);

  if ( Switch_CarFly )

    v37 = lpMem[1];

  else

    v37 = (LPVOID)HIDWORD(v73);

  v72[1] = "车辆飞行F7";

  DrawText(&v72[1], v77, v76, v37);

  if ( v72[1] )

    FreeMem(v72[1]);

  if ( Switch_CarFly )

    v38 = "开";

  else

    v38 = "关";

  StrSwitch = v38;

  lpMem[1] = (LPVOID)sub_40D2AF(1, dword_4DF3E8);

  lpMem[0] = (LPVOID)sub_40204B((char)StrSwitch);

  if ( StrSwitch )

    FreeMem(StrSwitch);

  if ( lpMem[1] )

    FreeMem(lpMem[1]);

  v73 = (double)v77;

  *(double *)v72 = v73 + 75.0;

  v68 = sub_4065DD(255, 0, 0);

  v61 = sub_4065DD(255, 0, 0);

  if ( Switch_CarFly )

    v39 = v68;

  else

    v39 = v61;

  v40 = v39;

  v41 = v76;

  v42 = toInt64(*(double *)v72);

  DrawText(lpMem, v42, v41, v40);

  if ( lpMem[0] )

    FreeMem(lpMem[0]);

  v76 += 25;

  StrSwitch = 0;

  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);

  lpMem[0] = 0;

  HIDWORD(v73) = sub_4065DD(255, 0, 0);

  if ( Switch_SuperSpeed )

    v43 = lpMem[1];

  else

    v43 = (LPVOID)HIDWORD(v73);

  v72[1] = "人物加速F8";

  DrawText(&v72[1], v77, v76, v43);

  if ( v72[1] )

    FreeMem(v72[1]);

  if ( Switch_SuperSpeed )

    v44 = "开";

  else

    v44 = "关";

  StrSwitch = v44;

  *(double *)lpMem = (double)v77;

  v73 = *(double *)lpMem + 75.0;

  v72[1] = 0;

  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);

  v69 = sub_4065DD(255, 0, 0);

  if ( Switch_SuperSpeed )

    v45 = v72[0];

  else

    v45 = (LPVOID)v69;

  v46 = v45;

  v47 = v76;

  v48 = toInt64(v73);

  DrawText(&StrSwitch, v48, v47, v46);

  if ( StrSwitch )

    FreeMem(StrSwitch);

  v76 += 25;

  StrSwitch = 0;

  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);

  lpMem[0] = 0;

  HIDWORD(v73) = sub_4065DD(255, 0, 0);

  if ( Switch_NoRecoil )

    v49 = lpMem[1];

  else

    v49 = (LPVOID)HIDWORD(v73);

  v72[1] = "腰射午后F9";

  DrawText(&v72[1], v77, v76, v49);

  if ( v72[1] )

    FreeMem(v72[1]);

  if ( Switch_NoRecoil )

    v50 = "开";

  else

    v50 = "关";

  StrSwitch = v50;

  *(double *)lpMem = (double)v77;

  v73 = *(double *)lpMem + 75.0;

  v72[1] = 0;

  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);

  v70 = sub_4065DD(255, 0, 0);

  if ( Switch_NoRecoil )

    v51 = v72[0];

  else

    v51 = (LPVOID)v70;

  v52 = v51;

  v53 = v76;

  v54 = toInt64(v73);

  DrawText(&StrSwitch, v54, v53, v52);

  if ( StrSwitch )

    FreeMem(StrSwitch);

  v76 += 25;

  StrSwitch = 0;

  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);

  lpMem[0] = 0;

  HIDWORD(v73) = sub_4065DD(255, 0, 0);

  if ( Switch_AbsorbObject )

    v55 = lpMem[1];

  else

    v55 = (LPVOID)HIDWORD(v73);

  v72[1] = "超级吸人F10";

  DrawText(&v72[1], v77, v76, v55);

  if ( v72[1] )

    FreeMem(v72[1]);

  if ( Switch_AbsorbObject )

    v56 = "开";

  else

    v56 = "关";

  StrSwitch = v56;

  *(double *)lpMem = (double)v77;

  v73 = *(double *)lpMem + 75.0;

  v72[1] = 0;

  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);

  v71 = sub_4065DD(255, 0, 0);

  if ( Switch_AbsorbObject )

    v57 = v72[0];

  else

    v57 = (LPVOID)v71;

  v58 = v57;

  v59 = v76;

  v60 = toInt64(v73);

  DrawText(&StrSwitch, v60, v59, v58);

  if ( StrSwitch )

    FreeMem(StrSwitch);

}
```
