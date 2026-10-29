
## Hook 脚本

```javascript
function hook_main__libso_encrypt() {
  let base = Module.findBaseAddress("libsojm.so");
  Interceptor.attach(base.add("0xFFD0C"), {
    onEnter: function (args) {
      this.arg0_len = findRangeByAddress(args[1])
      if (this.arg0_len == 64) {
        // 过滤输出, 感觉其他值都是无用的
        console.log(`onEnter encrypt arg0:${args[0]} arg1:${args[1]} arg2:${args[2]} arg3:${args[3]} arg4:${args[4]} arg5:${args[5]} arg6:${args[6]} arg7:${args[7]} arg8:${args[8]} arg9:${args[9]}`);
        console.error(`[ * ] libso_encrypt.args[${0}] onEnter :==> ${args[0].readCString()}`)
        // BwcnBzRjN2U/MmZhYjRmND4xPjI+NWQwZWU0YmI2MWQ3YjAzKw8cEywsIS4BIg==
        console.error(`[ * ] libso_encrypt.args[${1}] onEnter :==> ${findRangeByAddress(args[1])}`)
        // arg1 是 args[0] 字符串的长度
        // console.error(`[ * ] libso_encrypt.args[${1}] onEnter :==> ${findRangeByAddress(args[1])}`)
        // arg1 是 args[0] 字符串的长度
        // console.error(`[ * ] libso_encrypt.args[${2}] onEnter :==> ${findRangeByAddress(args[2].readPointer())}`)
        // arg2 感觉也是字符串长度 go 的字符串可能是字符串+长度 | slice 是长度+容量
        // arg3 似乎是函数执行次数
        // console.error(`[ * ] libso_encrypt.args[${4}] onEnter :==> ${findRangeByAddress(args[4].readPointer().readPointer())}`)
        // 867740 固定值
        // console.error(`[ * ] libso_encrypt.args[${5}] onEnter :==> ${findRangeByAddress(args[5].readPointer().readPointer())}`)
        // 16个0
        console.error(`[ * ] libso_encrypt.args[${5}] onEnter :==> ${ab2Hex(args[5].readPointer().readPointer().readByteArray(16))}`)
        // 16个0
        console.error(`[ * ] libso_encrypt.args[${6}] onEnter :==> ${args[6].readCString()}`)
        // data 明文数据⭐
        console.error(`[ * ] libso_encrypt.args[${7}] onEnter :==> ${findRangeByAddress(args[7])}`)
        // 明文数据长度
        console.error(`[ * ] libso_encrypt.args[${8}] onEnter :==> ${findRangeByAddress(args[8])}`)
        // 也是明文数据长度
        console.error(`[ * ] libso_encrypt.args[${9}] onEnter :==> ${args[9].readCString()}`)
        // 又是key
        console.error(`[ * ] libso_encrypt.args[${10}] onEnter :==> ${findRangeByAddress(args[10])}`)
        // key 长度
        console.error(`[ * ] libso_encrypt.args[${11}] onEnter :==> ${findRangeByAddress(args[11])}`)
        // key 长度
        console.error("------------------------------------------------------------");
      }
    }, onLeave: function (retval) {
      // console.log(`onLeave encrypt ${retval}`);
      // console.error("------------------------------------------------------------");
    }
  });
}

function call() {
  Java.perform(() => {
    let ret = Java.use("com.qq.lib.EncryptUtil").encrypt({"system_build_id":"a1000","system_iid":"923868eb5f543afa55f1f33cfac37d35","app_status":"9A5C6BDC62AD1CFE45A6578F84E858F9CA1A4F76:2","system_version":"5.7.1","system_build_aff":"","bundle_id":"tv.iytqy.cvhaca","system_app_type":"local","new_player":"fx","system_oauth_id":"02274477773a9ac8a2cf3605400ece4b","system_oauth_type":"android","system_token":"023FAC3AFE2285DC98E50CA4C638E0845277DC8CB62CCCC088DA1E3CB679180C97F2688C0DBE197F03950D84900C2A6B050334216C4CC604021DCC624543AFDDEF8D19363E43F9C0B8EEC79BBF269E7DE77B0304FF1034FED885ACA4C74DAC768D26D9FADD"}, "BwcnBzRjN2U/MmZhYjRmND4xPjI+NWQwZWU0YmI2MWQ3YjAzKw8cEywsIS4BIg==")
    console.warn(`encrypt("abcdef0123456789", "BwcnBzRjN2U/MmZhYjRmND4xPjI+NWQwZWU0YmI2MWQ3YjAzKw8cEywsIS4BIg==") :==> ${ret}`)
  })
}

function hook_dlopen(addr, soName, callback) {
  Interceptor.attach(addr, {
    onEnter: function (args) {
      const name = args[0].readCString(); // 输出so路径
      if (name.indexOf(soName) !== -1) this.hook = true;
    }, onLeave: function (retval) {
      if (this.hook) callback();
    }
  })
}

const android_dlopen_ext = Module.findExportByName(null, "android_dlopen_ext");
hook_dlopen(android_dlopen_ext, "libsojm.so", so);

so()
```

## 日志输出

```text
// 输出日志
onEnter encrypt arg0:0x86902940 arg1:0x40 arg2:0x40 arg3:0x22d arg4:0xc6a44ba0 arg5:0x86965f28 arg6:0x86944b40 arg7:0x22d arg8:0x22d arg9:0x86902940
[ * ] libso_encrypt.args[0] onEnter :==> BwcnBzRjN2U/MmZhYjRmND4xPjI+NWQwZWU0YmI2MWQ3YjAzKw8cEywsIS4BIg==
[ * ] libso_encrypt.args[1] onEnter :==> 64
[ * ] libso_encrypt.args[5] onEnter :==> [00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00]
[ * ] libso_encrypt.args[6] onEnter :==> {"system_build_id":"a1000","system_iid":"923868eb5f543afa55f1f33cfac37d35","app_status":"9A5C6BDC62AD1CFE45A6578F84E858F9CA1A4F76:2","system_version":"5.7.1","system_build_aff":"","bundle_id":"tv.iytqy.cvhaca","system_app_type":"local","new_player":"fx","system_oauth_id":"02274477773a9ac8a2cf3605400ece4b","system_oauth_type":"android","system_token":"023FAC3AFE2285DC98E50CA4C638E0845277DC8CB62CCCC088DA1E3CB679180C97F2688C0DBE197F03950D84900C2A6B050334216C4CC604021DCC624543AFDDEF8D19363E43F9C0B8EEC79BBF269E7DE77B0304FF1034FED885ACA4C74DAC768D26D9FADD"}
[ * ] libso_encrypt.args[7] onEnter :==> 557
[ * ] libso_encrypt.args[8] onEnter :==> 557
[ * ] libso_encrypt.args[9] onEnter :==> BwcnBzRjN2U/MmZhYjRmND4xPjI+NWQwZWU0YmI2MWQ3YjAzKw8cEywsIS4BIg==
[ * ] libso_encrypt.args[10] onEnter :==> 64
[ * ] libso_encrypt.args[11] onEnter :==> 64
------------------------------------------------------------------
encrypt("abcdef0123456789", "BwcnBzRjN2U/MmZhYjRmND4xPjI+NWQwZWU0YmI2MWQ3YjAzKw8cEywsIS4BIg==") :==> 6841646c536444435553467a65706946f933c2cae8a2a199fb1e46d1d60a1da52bd51cf0bdf4cae032f95454abdbfed529ccfcf252fa7581ef7504c5cd3c3465940bcbc59c629a029459dc10585f7426ab652646f2d2e4fc5861572e9b6ee65320ac4190fc6ffe3d41487fbb365e98dd906f9c3b13cbc46e422e75b0a28b947ede0dfe4acc98a851ec029f3d74f556f31cc2b82fafe70669fd0177ab1e2194e919c1c86789978beb1a46c2b4b14fb199596b46050fa85bf667d599fb31fd56e00de19b90b154c73a418c0ca7f4b9a2fadb3d5f912551c255681893db2e4260e3982743c8dfa5f3ba1010b58ae876c3361348f45cdfbd11be8a65e036e7c638834aa787c08417102f84c612020587e5518db749cd5616394e2eee544ca4febfa386e9befb9e8634ef17f34cb5afb8f1cbe45bb00706ef2547a8009b8faa7e7c4702dc9061e0143478d413b4d0477720c4c9779cc7b4f2ae60cc6b3fe5191e3c8a4948a344c92fdff1bd291b79d7148f933771199b6383bfa05c51f73d059673e5a1459213d2ef2bb858c134093aa44bb58f139a6870a853e31b9590c216f70e739fcbaef17d95550e61d8c3ca5ffa257f7f2534e6ed069e51f355a2b03e9f786fd33060b39f17721acb449bec0fce25cb9fdfce67be2475b06eba9ab0005a68bace1918ce4a2e61a726f286f775963f424c22dbcffaefef7d575fa868fb7a5111f36967a3537fd3d0c24e8e86196df7885624620fed4f085a07d205f19d201536391a1687fd7781491a60fe27f11d3ff3b03bba1ec20175ba4533ac8e2d3fd3a13dd20721706da02e5107745436ebd97425b1b7550f68e55d65a021e3467d1fa48a44912669286f4915ed84dbe276e53059f632d98cf2154831857d23c7d88f3f54f5050d7e97290b5f5a91a976c2ff4aec897c84ccc6ebad
```

## `main__libso_encrypt` 伪代码

```c
void __golang main__libso_encrypt(
        int a1,
        int a2,
        int key1,
        int a4,
        int a5,
        int a6,
        int a7,
        int a8,
        int data,
        int data_len,
        int a11,
        int key2)
{
  // .. 省略一些参数命名

  while ( (unsigned int)&a1 <= *(_DWORD *)(v12 + 8) )
    runtime_morestack_noctxt();
  data = 0;
  data_len = 0;
  v96 = &off_11ED34;
  main_logger(v13, (int)"in encrypt", 10); // 应该是个C的打印函数
  main_parsePassphrase(v24, a6, a7, a8); // 将 base64 编码的字符串解码并进行格式校验
  // 1.检查栈空间是否足够。
  //
  // 2.将传入的 base64 编码的密码字符串解码为二进制数据。
  //
  // 3.如果解码出错或者解码后的二进制数据长度小于 30，则返回相应的错误信息。
  //
  // 4.根据特定规则对解密后的二进制数据进行处理（异或操作等）。
  //
  // 5.将处理后的数据切片成若干个固定长度的子切片，并分别记录日志。
  //
  // 6.对其中一个子切片进行进一步的处理，并检查其格式是否符合特定要求。
  //
  // 7.如果格式不符合要求，则返回相应的错误信息；否则返回空指针表示处理成功。
  v14 = v58;
  if ( v83 )
  {
    data = 0;
    data_len = 0;
    main__libso_encrypt_func1(v37, v41); // 异常捕获的函数,用于恢复发生在goroutine中的panic
  }
  else
  {
    v92 = v69;
    v87 = v75;
    v88 = v78;
    v93 = v79;
    runtime_newobject(v25, (int)&map_string_interface_, v41);// malloc 分配一段内存并返回指向该内存地址的指针
    v95 = v42;
    runtime_makemap_small(v26, v38);
    if ( dword_1B9460 )
    {
      runtime_gcWriteBarrier(); // 垃圾回收
      v15 = v16;
    }
    else
    {
      v15 = v95;
      *(_DWORD *)v95 = v39;
    }
    encoding_json_Unmarshal(v27, key1, a4, a5, (int)&map_string_interface__ptr, v15, v75, v78);// 将 JSON 数据解析为 Go 的结构体
    if ( v76 )
    {
      v17 = a5;
      v18 = a4;
      v19 = key1;
    }
    else
    {
      v20 = v89;
      do
        *v20++ = 0; // 内存清零
      while ( (int)v20 <= (int)&v89[15] );
      qmemcpy(v89, "__package_name__", sizeof(v89));// 字符串拷贝
      runtime_slicebytetostring(v28, 0, (int)v89, 16, v59, v70);// 将字节切片转换为字符串
      v91 = v60;
      v86 = v71;
      v21 = v89;
      do
        *v21++ = 0; // 内存清零
      while ( (int)v21 <= (int)&v89[15] );
      qmemcpy(v89, "__package_hash__", sizeof(v89));// 字符串拷贝
      runtime_slicebytetostring(v29, 0, (int)v89, 16, v60, v71);// 将字节切片转换为字符串
      v90 = v61;
      v85 = v72;
      main__libso_getPackageName(v30, a2, v44, v48, v61);// 反射调用 Java 层方法获取包名
      runtime_slicebytetostring(v31, 0, v45, v49, v62, v72);// 将字节切片转换为字符串
      runtime_convTstring(v32, v63, (int)v73, v50);// 将指向任意类型的指针转换为字符串
      v22 = *(_DWORD *)v95;
      v94 = v51;
      runtime_mapassign_faststr((int)&map_string_interface_, v22, v91, v86);// 将指针指向的数据转为字符串返回
      *v73 = &string;
      if ( dword_1B9460 )
        runtime_gcWriteBarrier(); // 垃圾回收
      else
        v73[1] = v94;
      main__libso_getPackageCodePath(v33, a2, v46, v52, v64);// 获取应用路径
      runtime_slicebytetostring(v34, 0, v47, v53, v65, (int)v73);// 将字节切片转换为字符串
      main_md5File(v35, v66, (int)v74, v54, v66);
      runtime_convTstring(v36, (int)v55, v67, v55);// 将指向任意类型的指针转换为字符串
      v23 = *(_DWORD *)v95;
      v94 = v56;
      v77 = runtime_mapassign_faststr((int)&map_string_interface_, v23, v90, v85);// 将指针指向的数据转为字符串返回
      *v74 = &string;
      if ( dword_1B9460 )
        runtime_gcWriteBarrier(); // 垃圾回收
      else
        v74[1] = v94;
      v68 = encoding_json_Marshal((int)&map_string_interface_, *(_DWORD *)v95);// 转json数据
      v19 = v57;
      v18 = v68;
      v17 = (int)v74;
      if ( v77 )
      {
        v17 = a5;
        v18 = a4;
        v19 = key1;
      }
    }
    if ( v14 )
    {
      data = 0;
      data_len = 0;
    }
    else
    {
      main_oldEncrypt(v28, v19, v18, v17, v93, v80, v81, v92, v87, v88, v81, 0);// 开始加密
      data = v82;
      data_len = v84;
    }
    main__libso_encrypt_func1(v40, v43);
  }
}
```

## `main_parsePassphrase`

- 该函数主要做的事：将 base64 编码的字符串解码并进行格式校验

1. 检查栈空间是否足够 `runtime_morestack_noctxt`
2. 将传入的 base64 编码的密码字符串解码为二进制数据
3. 如果解码出错或者解码后的二进制数据长度小于 30，则返回相应的错误信息
4. 根据特定规则对解密后的二进制数据进行处理（异或操作等）
5. 将处理后的数据切片成若干个固定长度的子切片，并分别记录日志
6. 对其中一个子切片进行进一步的处理，并检查其格式是否符合特定要求
7. 如果格式不符合要求，则返回相应的错误信息；否则返回空指针表示处理成功

```javascript
function hook_main_parsePassphrase_ret() {
  let base = Module.findBaseAddress("libsojm.so");
  Interceptor.attach(base.add("0xFFD9C"), {
    onEnter(args) {
      console.log(`call 0xFFD9C ${JSON.stringify(this.context)}`);
      console.log(`call 0xFFD9C ${findRangeByAddress(this.context.r0)}`);
      console.log(`call 0xFFD9C ${findRangeByAddress(this.context.r1)}`);
    }
  });
}
```

```text
// BwcnBzRjN2U/MmZhYjRmND4xPjI+NWQwZWU0YmI2MWQ3YjAzKw8cEywsIS4BIg==
// ..'.4c7e?2fab4f4>1>2>5d0ee4bb61d7b03+...,,!.." 这是直接解码的字符串

// 得到了两个字符串（其实这里就是对我们传入的 PWD 解码对后半部分进行了一些类的运算
8692ece4 6d 49 5a 55 6a 6a 67 68 47 64 00 00 00 00 00 00
mIZUjjghGd......

8692ecc4 34 63 37 65 3f 32 66 61 62 34 66 34 3e 31 3e 32
4c7e?2fab4f4>1>2

8692ecd4 3e 35 64 30 65 65 34 62 62 36 31 64 37 62 30 33
>5d0ee4bb61d7b03

// 关于这里为什么是从栈上取的，是因为它的调用约定并不是 fastcall，分析汇编可知都是栈传递参数跟接受返回值
.text:000FFD64 5B 0D 00 EB                   BL              main_logger
.text:000FFD64
.text:000FFD68 90 00 9D E5                   LDR             R0, [SP,#0x7C+arg_14]
.text:000FFD6C 04 00 8D E5  STR             R0, [SP,#0x7C+var_78]   ; int
.text:000FFD70 94 00 9D E5                   LDR             R0, [SP,#0x7C+data]
.text:000FFD74 08 00 8D E5  STR             R0, [SP,#0x7C+var_74]   ; int
.text:000FFD78 98 00 9D E5                   LDR             R0, [SP,#0x7C+data_len]
.text:000FFD7C 0C 00 8D E5  STR             R0, [SP,#0x7C+var_70]   ; int
.text:000FFD80 1C 05 00 EB                   BL              main_parsePassphrase
.text:000FFD80
.text:000FFD84 14 00 9D E5                   LDR             R0, [SP,#0x7C+var_68]
.text:000FFD88 20 10 9D E5                   LDR             R1, [SP,#0x7C+var_5C]
.text:000FFD8C 18 20 9D E5                   LDR             R2, [SP,#0x7C+var_64]
.text:000FFD90 1C 30 9D E5                   LDR             R3, [SP,#0x7C+var_60]
.text:000FFD94 24 40 9D E5                   LDR             R4, [SP,#0x7C+var_58]
.text:000FFD98 28 50 9D E5                   LDR             R5, [SP,#0x7C+var_54]
.text:000FFD9C 2C 60 9D E5                   LDR             R6, [SP,#0x7C+var_50]
.text:000FFDA0 10 70 DD E5                   LDRB            R7, [SP,#0x7C+var_6C]
.text:000FFDA4 11 80 DD E5                   LDRB            R8, [SP,#0x7C+var_6C+1]
.text:000FFDA8 00 00 56 E3                   CMP             R6, #0
.text:000FFDAC 32 01 00 1A                   BNE             loc_10027C
```

## `main_md5File`

- 这里就是传入了一个文件路径，得到了一个文件的 MD5 值

```javascript
function hook_main_md5File() {
  let base = Module.findBaseAddress("libsojm.so");

  Interceptor.attach(base.add("0xFF94C"), {
    onEnter: function (args) {
      this.arg1 = args[1]
      if (this.arg1 == 0x3b) {
        console.log(`onEnter main_md5File arg0:${args[0]} arg1:${args[1]} arg2:${args[2]} arg3:${args[3]} arg4:${args[4]}`);
        console.log(`onEnter main_md5File arg0:${args[0].readCString(0x3b)}`); // 包路径 /data/app/tv.iytqy.cvhaca-gmGCryB_O0HztLC7rqkhRQ==/base.apk
        // console.log(`onEnter main_md5File arg1:${findRangeByAddress(args[1])}`); // arg1 参数0 长度
        // console.log(`onEnter main_md5File arg2:${findRangeByAddress(args[2])}`); // 地址内容为0
        // console.log(`onEnter main_md5File arg3:${findRangeByAddress(args[3])}`); // 地址内容为0
        // console.log(`onEnter main_md5File arg4:${findRangeByAddress(args[4])}`); // 不知道这个是啥
      }

    }, onLeave: function (retval) {
      if (this.arg1 == 0x3b) {
        console.log(`onLeave main_md5File ${retval.readCString(32)}`); // 678c7f3bf3584a2079295d8834928146
      }
    }
  });
}
```
