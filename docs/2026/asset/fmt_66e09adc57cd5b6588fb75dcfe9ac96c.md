
```c
signed int __fastcall parse_dex(_JNIEnv *a1, __int64 *a2)

{

  int v2; // r7@3

  char *v3; // r0@4

  const char *v4; // r1@4

  char *v5; // r0@6

  char *v6; // r9@16

  _BYTE *v7; // r3@17

  int v8; // r2@17

  int v9; // t1@19

  int fd; // ST14_4@23

  unsigned int v11; // r8@24

  int v12; // r7@24

  int v13; // r5@26

  int v14; // r5@25

  int v15; // r7@25

  int v16; // r1@25

  int v17; // r5@25

  int v18; // ST04_4@27

  int (__fastcall *v19)(int, signed int); // r5@27

  int v20; // r5@27

  unsigned __int8 *v21; // r8@27

  const char *v22; // r3@27

  char *v23; // r0@29

  char *v24; // r0@28

  char *v25; // r6@30

  ali::EncFile *v26; // r7@30

  int v27; // r0@31

  int *v28; // r0@34

  char *v29; // r0@34

  int v30; // r10@35

  void *v31; // r7@36

  int (__fastcall *v32)(unsigned __int8 *, int, signed int *); // r9@36

  int (__fastcall *v33)(_DWORD); // r7@36

  const char *v34; // r1@37

  const char *v35; // r2@37

  int v36; // r9@38

  signed int v37; // r7@38

  _DWORD *v38; // r9@41

  _BYTE *v39; // r5@41

  unsigned __int8 *v40; // r3@41

  void *v41; // r0@42

  JNINativeMethod *v42; // r0@42

  unsigned __int8 *v43; // r3@42

  signed int v44; // r3@42

  signed int result; // r0@44

  _JNIEnv *env; // [sp+8h] [bp-2A0h]@1

  __int64 *v47; // [sp+10h] [bp-298h]@1

  int v48; // [sp+24h] [bp-284h]@30

  unsigned __int8 *v49; // [sp+28h] [bp-280h]@30

  unsigned __int8 *v50; // [sp+2Ch] [bp-27Ch]@30

  void (__cdecl *v51)(const unsigned int *, jvalue *); // [sp+30h] [bp-278h]@42

  char v52; // [sp+34h] [bp-274h]@3

  signed int v53[2]; // [sp+38h] [bp-270h]@17

  char s; // [sp+40h] [bp-268h]@23

  char v55; // [sp+54h] [bp-254h]@2

  int v56; // [sp+64h] [bp-244h]@23

  int v57; // [sp+68h] [bp-240h]@23

  char v58; // [sp+6Ch] [bp-23Ch]@2

  const char *v59; // [sp+80h] [bp-228h]@2

  char v60; // [sp+84h] [bp-224h]@2

  const char *v61; // [sp+98h] [bp-210h]@25

  char v62; // [sp+9Ch] [bp-20Ch]@27

  unsigned int v63; // [sp+B0h] [bp-1F8h]@27

  char v64; // [sp+B4h] [bp-1F4h]@3

  char v65; // [sp+CCh] [bp-1DCh]@6

  int v66; // [sp+E0h] [bp-1C8h]@23

  char v67; // [sp+E4h] [bp-1C4h]@23

  char v68; // [sp+FCh] [bp-1ACh]@23

  const char *v69; // [sp+110h] [bp-198h]@23

  char v70; // [sp+114h] [bp-194h]@23

  char v71; // [sp+12Ch] [bp-17Ch]@23

  const char *v72; // [sp+140h] [bp-168h]@23

  char v73; // [sp+144h] [bp-164h]@23

  char *v74; // [sp+154h] [bp-154h]@23

  char *v75; // [sp+158h] [bp-150h]@23

  char v76; // [sp+15Ch] [bp-14Ch]@23

  char v77; // [sp+174h] [bp-134h]@23

  char v78; // [sp+18Ch] [bp-11Ch]@23

  char v79; // [sp+1A4h] [bp-104h]@23

  char v80; // [sp+1BCh] [bp-ECh]@23

  char v81; // [sp+1D4h] [bp-D4h]@23

  char v82; // [sp+1ECh] [bp-BCh]@23

  char v83; // [sp+204h] [bp-A4h]@23

  char v84; // [sp+21Ch] [bp-8Ch]@23

  char v85; // [sp+234h] [bp-74h]@17

  int v86; // [sp+244h] [bp-64h]@17

  _BYTE *v87; // [sp+248h] [bp-60h]@17

  char v88; // [sp+24Ch] [bp-5Ch]@22

  char v89; // [sp+264h] [bp-44h]@22

  char *v90; // [sp+274h] [bp-34h]@22

  char *v91; // [sp+278h] [bp-30h]@22

  int v92; // [sp+27Ch] [bp-2Ch]@1



  env = a1;

  v47 = a2;

  v92 = _stack_chk_guard;

  _android_log_print(3, "debug", "enter parse_dex");

  if ( ali::isDalvik )

  {

    v48 = 0;

    std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v89, &ali::g_filePath, "/cls.jar");

    v25 = v91;

    v26 = operator new(0xCu);

    ali::EncFile::EncFile(v26, v25);

    v49 = 0;

    v50 = 0;

    if ( ali::sdk_int > 13 )

    {

      v27 = ali::EncFile::openWithHeader(v26, &v49, &v48, 0x10u);

      v50 = v49 + 16;

    }

    else

    {

      v27 = ali::EncFile::open(v26, &v50, &v48);

    }

    if ( v50 == -1 )

    {

      v28 = _errno(v27);

      v29 = strerror(*v28);

      _android_log_print(3, "debug", "mmap dex file :%s", v29);

LABEL_45:

      v24 = &v89;

      goto LABEL_46;

    }

    v30 = *(v50 + 8);

    if ( ali::sdk_int > 13 )

    {

      v41 = dlopen("libdvm.so", 1);

      v42 = dlsym(v41, "dvm_dalvik_system_DexFile");

      v51 = 0;

      lookup(v42, "openDexFile", "([B)I", &v51);

      v43 = v49;

      *(v49 + 2) = v48;

      *&v52 = v43;

      (v51)();

      v44 = v53[0];

      *v47 = v53[0];

      *(*(*(v44 + 8) + 4) + 32) = *(v44 + 16);

      *(*(*(v44 + 8) + 4) + 36) = v48;

      ali::EncFile::~EncFile(v26);

      operator delete(v26);

    }

    else

    {

      v31 = dlopen("libdvm.so", 1);

      v32 = dlsym(v31, "dvmDexFileOpenPartial");

      v33 = dlsym(v31, "dexCreateClassLookup");

      v53[0] = 0;

      if ( v32(v50, v30, v53) == -1 )

      {

        v34 = "debug";

        v35 = "dvmDexFileOpenPartial error";

LABEL_40:

        _android_log_print(3, v34, v35);

        goto LABEL_45;

      }

      v36 = *v53[0];

      *(v36 + 36) = v33(*v53[0]);

      v37 = v53[0];

      if ( !*(*v53[0] + 36) )

      {

        v34 = "debug";

        v35 = "dexCreateClassLookup error";

        goto LABEL_40;

      }

      v38 = malloc(0x2Cu);

      v39 = malloc(0x14u);

      strdup(&unk_4CEE9);

      v39[4] = 0;

      v39[5] = 0;

      *(v39 + 2) = 0;

      v40 = v50;

      *v39 = v39;

      *(v39 + 3) = v38;

      v38[10] = v37;

      *(v37 + 32) = v40;

      *(v37 + 36) = v48;

      *v47 = v39;

    }

    v23 = &v89;

    goto LABEL_44;

  }

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v55, &ali::g_filePath, "/cls.jar");

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v58, &ali::g_filePath, "/cls.dex");

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v60, &ali::g_filePath, "/fak.jar");

  _android_log_print(3, "debug", "before oat gen");

  if ( !access(v59, 0) )

    goto LABEL_24;

  v2 = android_getCpuFamily();

  std::string::string(&v64, "arm", &v52);

  if ( v2 == 1 )

  {

    v3 = &v64;

    v4 = "arm";

LABEL_5:

    std::string::operator=(v3, v4);

    goto LABEL_6;

  }

  if ( v2 == 2 )

  {

    v3 = &v64;

    v4 = "x86";

    goto LABEL_5;

  }

  if ( v2 == 3 || v2 == 6 )

  {

    v3 = &v64;

    v4 = "mips";

    goto LABEL_5;

  }

  if ( v2 == 4 )

  {

    v3 = &v64;

    v4 = "arm64";

    goto LABEL_5;

  }

  if ( v2 == 5 )

  {

    v3 = &v64;

    v4 = "x86_64";

    goto LABEL_5;

  }

LABEL_6:

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v65, &ali::g_libPath, "/libhack.so");

  v5 = getenv("LD_PRELOAD");

  v6 = v5;

  if ( v5 )

  {

    _android_log_print(3, "debug", "the system already define LD_PRELOAD=%s", v5);

    std::string::string(&v85, v6, v53);

    v7 = v87;

    v8 = v86;

    while ( v7 != v8 )

    {

      v9 = *v7++;

      if ( v9 == 32 )

        *(v7 - 1) = 58;

    }

    std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v88, &v85, ":");

    std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v89, &v88, &v65);

    std::string::_M_assign(&v65, v91, v90);

    std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v89);

    std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v88);

    std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v85);

  }

  _android_log_print(3, "debug", "the new LD_PRELOAD is %s", v66);

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v67, &ali::g_filePath, "/juice.data");

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v68, &ali::g_filePath, "/fak.jar");

  fd = open(v69, 0);

  memset(&s, 0, 0x14u);

  sprintf(&s, "%d", fd);

  std::string::string(&v70, &s, v53);

  v74 = &v73;

  v75 = &v73;

  std::priv::_String_base<char,std::allocator<char>>::_M_allocate_block(&v73, v56 - v57 + 10);

  *v74 = 0;

  std::string::_M_appendT<char const*>(&v73);

  std::string::append(&v73, &v55);

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v76, &v73, "                     JUICE_FILE=");

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v77, &v76, &v67);

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v78, &v77, "                     LD_PRELOAD=");

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v79, &v78, &v65);

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v80, &v79, "                     /system/bin/dex2oat \t\t\t\t  \t --runtime-arg -Xms64m \t\t\t\t\t --runtime-arg -Xmx64m \t\t\t\t" "\t --boot-image=/system/framework/boot.art                      --zip-fd=");

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v81, &v80, &v70);

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v82, &v81, "\t\t\t\t\t --zip-location=");

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v83, &v82, &v68);

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v84, &v83, "\t\t\t\t\t --oat-file=");

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v85, &v84, &v58);

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v71, &v85, "                     ");

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v85);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v84);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v83);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v82);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v81);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v80);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v79);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v78);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v77);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v76);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v73);

  _android_log_print(3, "debug", "cmd is %s", v72);

  system(v72);

  close(fd);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v71);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v70);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v68);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v67);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v65);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v64);

LABEL_24:

  _android_log_print(3, "debug", "after oat gen");

  v11 = ali::JDexFile;

  v12 = unk_54140;

  if ( ali::sdk_int <= 19 )

  {

    v13 = _JNIEnv::NewStringUTF(env, v61);

    _JNIEnv::NewStringUTF(env, v59);

    v15 = _JNIEnv::CallStaticIntMethod(env, v11, v12, v13);

    v17 = 0;

  }

  else

  {

    v14 = _JNIEnv::NewStringUTF(env, v61);

    _JNIEnv::NewStringUTF(env, v59);

    v15 = _JNIEnv::CallStaticLongMethod(env, v11, v12, v14);

    v17 = v16;

  }

  v18 = v17;

  _android_log_print(3, "debug", "cookie is %llx");

  *v47 = v15;

  *(v47 + 1) = v17;

  v19 = dlsym(0xFFFFFFFF, "_ZNK3art7DexFile12FindClassDefEt");

  _android_log_print(3, "debug", "DexFile::FindClassDefFn is %p", v19, v15, v18);

  v20 = v19(v15, 1);

  _android_log_print(3, "debug", "call FindClassDefFn(%p,%d) => %p", v15, 1, v20);

  _android_log_print(3, "debug", "dex position is %p", v20 - 572);

  _android_log_print(3, "debug", "dex head is %08x %08x", *(v20 - 572), *(v20 - 568));

  v21 = *(v20 - 540);

  _android_log_print(3, "debug", "dex size is %d", v21);

  MemEnableWrite((v20 - 572), &v21[v20 - 572]);

  std::operator+<char,std::char_traits<char>,std::allocator<char>>(&v62, &ali::g_filePath, "/juice.data");

  if ( !ali::dex_juicer_patch((v20 - 572), v21, v63, v22) )

  {

    std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v62);

    std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v60);

    std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v58);

    v23 = &v55;

LABEL_44:

    std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(v23);

    _android_log_print(3, "debug", "exit parse_dex");

    result = 0;

    goto LABEL_47;

  }

  _android_log_print(6, "debug", "fail to patch dex");

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v62);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v60);

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(&v58);

  v24 = &v55;

LABEL_46:

  std::priv::_String_base<char,std::allocator<char>>::_M_deallocate_block(v24);

  _android_log_print(3, "debug", "exit parse_dex error");

  result = -1;

LABEL_47:

  if ( v92 != _stack_chk_guard )

    _stack_chk_fail(result);

  return result;

}
```
