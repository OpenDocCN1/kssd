
# FLEXlm9.2 的破解教学二

> 原文：[`www.pediy.com/kssd/pediy06/pediy6934.htm`](https://www.pediy.com/kssd/pediy06/pediy6934.htm)

On Software Reverse Engineering - 2 选择自 zxg32 的 Blog  
Reverse Engineering, FLEXlm, IMSL  

翻译第二篇 FLEXlm 结构  

在配备有 `VS+FLEXLM` 源代码、`W32Dasm+cmath.exe` 以及 `IDA+cmath.exe`(具签名)后，现在我们将能够揭示 FLEXLM 的核心了。下面是我们的一些发现，在这里 `lm_ckout.c!lc_checkout()` 表示”在 `lm_ckout.c` 模块/文件中的函数 `lc_checkout()`“，箭头符号指示函数调用。注意，由于程序分支的原因，仅部分代码被追踪和展示，但通常情况下，我们感兴趣的仅是这些分支。  

```
0047D0C6: push 00000018 ;程序入口点

0047D22D: call 00401000 ;调用 cmath.exe!main()

0047D240: call 0047F20D ;调用链接到 ntdll.dll!NtTerminateProcess()

sub_401000: cmath.exe!main()

0040101D: call 004033B0 ;调用 vc++\flexlm.obj!imsl_f_lin_sol_gen()

00401039: call 00401050 ;调用 vc++\fwrimat.obj!imsl_f_write_matrix()

sub_004033B0: vc++\flexlm.obj!imsl_f_lin_sol_gen()

004033C8: call 00408F24 ;调用 vc++\error.obj!imsl_e1psh()

0040342C: call 004034A0 ;调用 vc++\flinslg.obj!l_lin_sol_gen()以进行真正的工作

sub_00408F24: vc++\error.obj!imsl_e1psh()

00408F3A: call 0040A850 ;调用链接 vc++\single.obj!imsl_once() ® vc++\error.obj!l_error_init()

;® vc++\flexlm.obj!imsl_flexlm()

00408F76: call 00414AFD ;调用 imsl_highland_check() ® l_check.c!lc_timer() ® l_timer_heart()

;® l_check() ® l_reconnect() ® lm_ckout.c!l_checkout() as heartbeat

sub_00413290: vc++\flexlm.obj!imsl_flexlm()

004132EF: call 004294A0 ;调用 lm_njob.c!lc_new_job()

004133A4: call 00426380 ;设置 LM_A_DISABLE_ENV 为 1

004133DE: call 00426380 ;设置 LM_A_LICENSE_FILE_PTR 为许可证文件位置

00413486: call 00426380 ;设置 LM_A_CHECK_INTERVAL 为 -1

004134C0: call 00426380 ;设置 LM_A_RETRY_INTERVAL 为 -1

004134FB: call 00426380 ;设置 LM_A_RETRY_COUNT 为 -1

004135A7: call 00426380 ;设置 LM_A_LINGER 为 0

004136A6: call 0042420C ;调用 l_check.c!lc_status(), 返回 LM_NEVERCHECKOUT

004138CD: call 0041A010 ;调用 lm_ckout.c!lc_checkout()

00414099: call 0047FA9B ;返回当前日期和时间

004141C6: call 0042563D ;调用 lm_config.c!lc_get_config()

0041434E: call 0047F8F0 ;检查许可证十分过期，如果没有返回 0

sub_0041A010: lm_ckout.c!lc_checkout()

0041A093: call 0041A14B ;调用 lm_ckout.c!l_checkout(),返回 FFFFFFF8

sub_0041A14B: lm_ckout.c!l_checkout()

0041A2E8: call [004AA01C] ;调用 lm_ckout.c!lm_start_real(),返回 FFFFFFF8

sub_0041A875: lm_ckout.c!lm_start_real()

0041AA47: call 0041B4A5 ;调用 lm_ckout.c!l_local_verify_conf(), 返回 1=成功

0041AC01: call 0041BD89 ;调用 lm_ckout.c!l_good_lic_key(), 返回 0=失败

sub_0041BD89: lm_ckout.c!l_good_lic_key()

0041BE30: call 00433D15 ;调用 l_getattr.c!l_xorname()

0041BE4D: call 0041DB9E ;调用 lm_ckout.c!l_sg()

0041C202: call 0041EBE3 ;调用 vc++\lm_ckout.obj!l_crypt_private(), 返回 0

sub_0041DB9E: lm_ckout.c!l_sg()

0041DBF7: call [004AD064] ;调用 lm_new.c!l_n36_buff()

0041DC16: call 00443283 ;调用 l_key.c!l_key()

sub_0041EBE3: vc++\lm_ckout.obj!l_crypt_private()

0041EC07: call 0041EE44 ;调用 vc++\lm_ckout.obj!real_crypt(), 返回 0

sub_0041EE44: vc++\lm_ckout.obj!real_crypt()

0041F9B6: call 00420AF6 ;调用 vc++\lm_ckout.obj!l_string_key(), 返回 0

sub_00420AF6: vc++\lm_ckout.obj!l_string_key()

00420E94 - 00421156 ;调用宏 XOR_SEEDS_INIT_ARRAY(xor_arr)

00421247: call 0047F250 ;调用 strcpy(lkey, license_key)

0042145E: call 004803A0 ;调用 memcpy(newinput, input, inputlen)

0042191D: call 00421D66 ;调用 l_strkey.c!our_encrypt()

00421A13 - 00421B27 ;for{}循环许可证密码匹配

00421B34: call 00421C22 ;调用 l_strkey.c!atox()以转换二进制串为 ASCII 文本

sub_00426380: lm_set_attr.c!lc_set_attr()

004263E4: call 0042641E ;调用 lm_set_attr.c!l_set_attr()以设置配置结构中的属性

sub_0042641E: lm_set_attr.c!l_set_attr()

00427045: call 00427DBC ;如果设置 LM_A_LICENSE_FILE_PTR,调用 lm_set_attr.c!l_set_license_path()

;® lm_config.c!l_flush_config() ® l_init_file.c!l_init_file()

;® l_allfeat.c!l_allfeat() ® l_allfeat.c!l_parse_feature_line()

;® l_allfeat.c!oldkey() ® vc++\l_allfeat.obj!l_crypt_private()

sub_004294A0: lm_njob.c!lc_new_job()

004294C0: call [004A5A98] ;调用 lm_new.c!l_n36_buf(), 返回 1

004294D2: call [004A5A98] ;调用具有所有的 O 参数的 lm_new.c!l_n36_buf(),返回 0

004294E8: call 0044357F ;调用链接 lm_init.c!lc_init() ® lm_init.c!l_init()

004294FD – 00429516 ;打开 LM_OPTFLAG_CUSTOM_KEY5 标志

sub_0043873B: vc++\l_allfeat.obj!l_crypt_private()

0043875F: call 00438771 ;调用 vc++\l_allfeat.obj!real_crypt(), 返回 21D5B6E8572E

sub_00438771: vc++\l_allfeat.obj!real_crypt()

004392DC: call 0043A41C ;调用 vc++\l_allfeat.obj!l_string_key(), 返回 21D5B6E8572E

sub_0043A41C: vc++\l_allfeat.obj!l_string_key()

sub_0044359E: lm_init.c!l_init() ;接收 VENDORCODE 和 VENDORNAME 以初始化 job 结构

00443E8F – 00444354 ;一些有效性测试，可能报告错误

004441F9: call 0041DB9E ;调用 lm_ckout.c!l_sg()

sub_0044A110: lm_new.c!l_n36_buf() ;初始化 VENDORCODE 结构和 VENDORNAME

0044B503: push 00450BE0 ;压入 lm_new.c!l_n36_buff() 地址

0044B508: call 00444B11 ;调用 lm_init.c!l_x77_buf() 以设置 L_UNIQ_KEY5_FUNC

sub_00450BE0: lm_new.c!l_n36_buff()

00450C34 – 00450EB0 ;模糊化在 job 中的 mem_ptr2_bytes[]

00450EB5 – 00450FF8 ;模糊化在 VENDORCODE 中的 data[0]和 data[1]

sub_77F8DD80: ntdll.dll!NtTerminateProcess()

77F8DD8B: ret 00000008 ;程序退出
```

正如我们所看到的，我们先前提及的子程序 `0041A010` 实际上是 `lm_ckout.c!lc_checkout()`，它是一个真正的核心函数。如果许可证检查成功，它将返回 `0`；否则，它将返回错误代码(在 `lmclient.h` 中 `FFFFFFF8` 被定义为 `LM_BADCODE` ![](img/e45fac309594d4f9580d902b0ed40a73.jpg)。由于是核心，`l_checkout()` 可能将被调用几次，但我们不关心（它）。记住，我们的目标是找到 `checksum` 对比代码的位置和恢复真正的签名。  

一个快速的搜索告诉我们：`STRNCMP`(在 `l_privat.h` 中定义的宏， 如果串匹配则设置结果=`0`)仅在 `lm_ckout.c!l_good_lic_key()` 和 `l_crypt.c!l_crypt_private()` 中出现。注意 `l_crypt.c` 和 `l_strkey.c` 并不直接被编译为目标文件，而是被包含进模块 `lm_ckout.obj` 和 `l_allfeat.obj` 中。除了这两个之外，`lm_crypt.obj` 也包含了 `l_crypt.c` 并揭示其函数到外部作为了 API，其下的许多(函数)具有不同的别名，比如 `lc_crypt()`。但是，在 `cmath.exe` 中并未用该（代码）拷贝：`lc_crypt()` 的地址是 `00469960`，但我们在该处设置断点却什么也没有发生。  

`lm_ckout.c`

```
... ...

#define l_crypt_private l_ckout_crypt

#define l_string_key l_ckout_string_key

... ...

/* Include l_crypt.c, so that these functions won’t be global. */

#define LM_CKOUT

#include "l_crypt.c"

l_allfeat.c

... ...

#define LM_CRYPT_HASH

#include "l_crypt.c"

l_crypt.c

#include "l_strkey.c"

l_strkey.c

#include "l_strkey.h"

lm_crypt.c

#define l_crypt_private lc_crypt

... ...

#define LM_CRYPT

... ...

#include "l_crypt.c"
```

由于包含了超过一次，因为编译指令的原因，`l_crypt/l_strkey` 函数的多个拷贝将不再相同。下面的 `l_crypt_private()` 代码说明了这点（在这里，`STRNCMP` 不是真正的问题）。取决于 `LM_CKOUT` 是否被定义，`0041EBE3` (长版本)和 `0043873B` (短版本)在两个模块中将有所不同。`IDA FLAIR` 认知后者而非前者－实践中，我们必须在 `lm_ckout.obj` 中手工识别这些函数。  

```
ret = real_crypt(job, conf, sdate, code);

#ifdef LM_CKOUT

if (!(job->user_crypt_filter) && !(job->lc_this_keylist) && valid_code(conf->code))

{

if (job->flags & LM_FLAG_MAKE_OLD_KEY)

{

STRNCMP(conf->code, ret, MAX_CRYPT_LEN, not_eq);

}

else

{

STRNCMP(conf->lc_sign, ret, MAX_CRYPT_LEN, not_eq);

}

if (not_eq && !(job->options->flags & LM_OPTFLAG_STRINGS_CASE_SENSITIVE))

{

job->options->flags |= LM_OPTFLAG_STRINGS_CASE_SENSITIVE;

ret = real_crypt(job, conf, sdate, code);

job->options->flags &= ~LM_OPTFLAG_STRINGS_CASE_SENSITIVE;

}

}

#endif /* LM_CKOUT */

return ret;
```

在上面的代码注释中，我们看到 `l_crypt_private()` 是计算 `SIGN` 杂乱信息的关键函数。当分别设置 `LM_A_LICENSE_FILE_PTR` 和 `lc_checkout()` 时，将调用两个链接并从 `lc_set_attr()` 初始化。第一次追踪 `l_crypt_private()` 的结果是 `21D5B6E8572E`，第二次为 `0`。它看起来就好像是签名代码，因此我们立即尝试用于许可证文件。很不幸的是，它并不工作。  

好了，那并不令人吃惊；令人吃惊的事情是，为什么 `lc_set_attr()` 将计算 `checksum`（总和检查）?（其实）设置属性与授权无关。但是，我们必须指出，在链接调用中，许可证文件的每行是被逐字解析的！在我们的例子中，在 `license.dat` 中有 3 行但仅有一个 feature（特征）行，因此 `l_parse_feature_line()` 被调用了三次而 `oldkey()` 仅被调用一次。我们认为，最大的可能解释是：它仅仅是填充一个杂乱信息代码到 `config` 结构中作为初始化，正如其 `oldkey()` 名字所暗示的那样。或者，你可以说这是一个花招来引诱破解者从真相到伪装得很好的（假）代码上去，以此来浪费他们的时间。
