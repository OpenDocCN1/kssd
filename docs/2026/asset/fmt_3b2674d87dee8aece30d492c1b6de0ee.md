
`/data/Project/Base_RevC_AEMvA_pkg/models/Linux64_GCC-9.3/FVP_Base_RevC-2xAEMvA` \
`-C pctl.startup=0.0.0.0` \
`-C bp.secure_memory=1` \
`-C bp.tzc_400.diagnostics=1` \
`-C cluster0.NUM_CORES=4` \
`-C cluster1.NUM_CORES=4` \
`-C cache_state_modelled=1` \
`-C bp.secureflashloader.fname="./bl1.bin"` \
`-C bp.flashloader0.fname="./fip.bin"` # 如果需要运行到rootfs请添加下方参数，
`--data cluster0.cpu0="<path-to>/<kernel-binary>"@0x80080000` \
`--data cluster0.cpu0="<path-to>/<ramdisk>"@0x84000000`

![image-20240422190557018](https://bbs.kanxue.com/upload/attach/202405/749234_TND2FDTTEASUFW2.png)

## TF-A Tests构建并运行

```bash
export CROSS_COMPILE=/data/toolchains/SYS_PUBLIC_TOOLS/.toolchain/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu-linux-5.10/bin/aarch64-none-linux-gnu-
git clone https://review.trustedfirmware.orgTF-A/tf-a-tests.git
cd tf-a-tests
make PLAT=fvp tftf
```

![image-20240422192642093](https://bbs.kanxue.com/upload/attach/202405/749234_ZNWY2Q2EKCMZNWA.png)

重编译ATF，指定bl33.bin为tftf.bin

```bash
cd /data/Project/arm-trusted-firmware-lts-v2.8.4/
export CROSS_COMPILE=/data/toolchains/SYS_PUBLIC_TOOLS/.toolchain/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu-linux-5.10/bin/aarch64-none-linux-gnu-
make PLAT=fvp BL33=/data/Project/tf-a-tests/build/fvp/release/tftf.bin all fip
```

![image-20240422192823075](https://bbs.kanxue.com/upload/attach/202405/749234_5H3AMJFJBS4WACJ.png)

重新使用FVP运行，成功引导进入tftf中

![image-20240422193006244](https://bbs.kanxue.com/upload/attach/202405/749234_25G52N4QH5FT8EQ.png)

运行完成后会输出测试结果并提示退出

![image-20240422193154845](https://bbs.kanxue.com/upload/attach/202405/749234_AAPUCDR3J3RVK6P.png)

## SMC Fuzz

推荐阅读：[ab5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1M7Y4g2K6N6r3g2V1k6X3W2J5L8i4N6S2M7X3g2Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8$3y4K6i4K6u0r3c8r3W2J5k6h3y4@1k6h3c8Q4y4h3k6d9j5h3c8G2L8h3W2*7k6h3c8Q4y4h3k6e0e0f1y4Q4y4h3k6b7M7X3g2K6k6h3&6@1j5i4c8A6L8$3&6Q4x3X3g2H3k6r3j5`.](https://bbs.kanxue.com/elink@98bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1M7Y4g2K6N6r3g2V1k6X3W2J5L8i4N6S2M7X3g2Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8$3y4K6i4K6u0r3c8r3W2J5k6h3y4@1k6h3c8Q4y4h3k6d9j5h3c8G2L8h3W2*7k6h3c8Q4y4h3k6e0e0f1y4Q4y4h3k6b7M7X3g2K6k6h3&6@1j5i4c8A6L8$3&6Q4x3X3g2H3k6r3j5`.)

### 默认配置运行

```bash
export CROSS_COMPILE=/data/toolchains/SYS_PUBLIC_TOOLS/.toolchain/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu-linux-5.10/bin/aarch64-none-linux-gnu-
make PLAT=fvp SMC_FUZZING=1 SMC_FUZZ_DTS=/data/Project/tf-a-tests/smc_fuzz/dts/top.dts TESTS=smcfuzzing tftf
```

**注意这里的`SMC_FUZZ_DTS`是可以自定义的，这里使用了官方提供的`top.dts`**

```dts
/*
 * Copyright (c) 2023, Arm Limited.
 * All rights reserved.
 *
 * SPDX-License-Identifier: BSD-3-Clause
 */

/*
 * Top level device tree file to bias the SMC calls. T
 * he biases are arbitrary and can be any value.
 * They are only significant when weighted against the
 * other biases. 30 was chosen arbitrarily.
 */

/dts-v1/;

/ {
    sdei {
        bias = <30>;
        sdei_version {
            bias = <30>;
            functionname = "sdei_version_funcid";
        };
        sdei_pe_unmask {
            bias = <30>;
            functionname = "sdei_pe_unmask_funcid";
        };
        sdei_pe_mask {
            bias = <30>;
            functionname = "sdei_pe_mask_funcid";
        };
        sdei_event_status {
            bias = <30>;
            functionname = "sdei_event_status_funcid";
        };
        sdei_event_signal {
            bias = <30>;
            functionname = "sdei_event_signal_funcid";
        };
        sdei_private_reset {
            bias = <30>;
            functionname = "sdei_private_reset_funcid";
        };
        sdei_shared_reset {
            bias = <30>;
            functionname = "sdei_shared_reset_funcid";
        };
    };
    tsp {
        bias = <30>;
        tsp_add_op {
            bias = <30>;
            functionname = "tsp_add_op_funcid";
        };
        tsp_sub_op {
            bias = <30>;
            functionname = "tsp_sub_op_funcid";
        };
        tsp_mul_op {
            bias = <30>;
            functionname = "tsp_mul_op_funcid";
        };
        tsp_div_op {
            bias = <30>;
            functionname = "tsp_div_op_funcid";
        };
    };
};
```

![image-20240423103107010](https://bbs.kanxue.com/upload/attach/202405/749234_H9D8UPWP64GE8TP.png)

重编译ATF，并替换tftf.bin

```bash
cd /data/Project/arm-trusted-firmware-lts-v2.8.4/
export CROSS_COMPILE=/data/toolchains/SYS_PUBLIC_TOOLS/.toolchain/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu-linux-5.10/bin/aarch64-none-linux-gnu-
make PLAT=fvp BL33=/data/Project/tf-a-tests/build/fvp/release/tftf.bin all fip
```

![image-20240423102154343](https://bbs.kanxue.com/upload/attach/202405/749234_9TTMFNPM7WNS2JA.png)

再次运行

```bash
cd /data/Project/arm-trusted-firmware-lts-v2.8.4/build/fvp/release/
 /data/Project/Base_RevC_AEMvA_pkg/models/Linux64_GCC-9.3/FVP_Base_RevC-2xAEMvA \
 -C pctl.startup=0.0.0.0 \
 -C bp.secure_memory=1 \
 -C bp.tzc_400.diagnostics=1 \
 -C cluster0.NUM_CORES=4 \
 -C cluster1.NUM_CORES=4 \
 -C cache_state_modelled=1 \
 -C bp.secureflashloader.fname="./bl1.bin" \
 -C bp.flashloader0.fname="./fip.bin"
```

![image-20240423153625590](https://bbs.kanxue.com/upload/attach/202405/749234_RQJ9MZV86R7E6V6.png)

### 扩展SMC fuzz

先来通过目录结构确定需要扩展的文件1. `Dts` 2. `fuzz helper`

![image-20240423162629887](https://bbs.kanxue.com/upload/attach/202405/749234_UUXXVRWNDH5S2CW.png)

1. 首先创建`test_fuzz_helper.h` ，引用上述头文件(tftf框架), 并且定义与dts中function_name对应的常量funcid。最后在底部申明使用到的函数入口run_test_fuzz和具体的handler函数tftf_test_smc

```c
//
// Created by ios on 24-4-23.
//

#include <fuzz_helper.h>
#include <power_management.h>
#include <sdei.h>
#include <test_helpers.h>
#include <tftf_lib.h>
#include <timer.h>

#ifndef test_funcid
#define test_funcid 0
#endif

void tftf_test_smc(uint64_t tsp_id, char *funcstr);
void run_test_fuzz(int funcid);
```

1. 完善具体的test_fuzz_helper.c,具体功能为打印固定的字符串`ios-test` 并输出测试信息。
