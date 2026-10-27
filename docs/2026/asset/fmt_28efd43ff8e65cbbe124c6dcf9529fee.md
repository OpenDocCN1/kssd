
```
_QWORD * __fastcall sub_70EE90(_QWORD * a1) {
    int v2; // eax
    int v3; // ebp
    unsigned __int64 v4; // rax
    __int64 v5; // rdx
    __m128i v6[2]; // [rsp+0h] [rbp-1C8h] BYREF
    struct stat stat_buf; // [rsp+20h] [rbp-1A8h] BYREF
    char buf[280]; // [rsp+B0h] [rbp-118h] BYREF

    *a1 = &qword_BB1640[3];
    strcpy(v6[0].m128i_i8, "0qspd0tzt0lfsofm/hostname");
    v6[0] = _mm_add_epi8((__m128i)-1LL, v6[0]);
    if ( __xstat(1, v6[0].m128i_i8, &stat_buf) )
        return a1;
    v2 = open(v6[0].m128i_i8, 0);
    v3 = v2;
    if ( v2 < 0 )
        return a1;
    v4 = read(v2, buf, 0x100uLL);
    v5 = 255LL;
    if ( v4 <= 0xFE )
    {
        LOBYTE(v5) = 0;
        if ( v4 )
            v5 = v4 - 1;
    }
    buf[v5] = 0;
    sub_80BCF0(a1, buf, strlen(buf));
    close(v3);
    return a1;
}
```

下面总结了不同编号对应的功能

| 编号 | 功能 |
| --- | --- |
| 5001 | `/proc/sys/kernel/hostname` |
| 5002 | `/sys/bus/i2c/drivers/i2c_adapter/module/srcversion` |
| 5003 |  |
| 6001 |  |
| 6101 | `/sys/class/dmi/id/product_name` |
| 6102 | `/sys/class/dmi/id/product_version` |
| 6103 | `/sys/class/dmi/id/sys_vendor` |
| 6104 | `/sys/class/dmi/id/board_version` |
| 6105 | `/sys/class/dmi/id/board_vendor` |
| 6106 | `/sys/class/dmi/id/board_name` |
| 6107 | `/sys/class/dmi/id/bios_date` |
| 6108 | `/sys/class/dmi/id/bios_vendor` |
| 6109 | `/sys/class/dmi/id/bios_version` |
| 6110 | 运行 `/usr/bin/codemeter-info` 命令 |
| 7001 | 硬盘相关 |
| 7002 | 硬盘相关 |
| 7003 | 硬盘相关 |
| 8001 |  |
| 8002 | `/sys/class/power_supply` |
| 8003 | `/sys/class/drm` |
| 8004 |  |
| 8005 | 运行 `/usr/bin/codemeter-info` 命令 |

未完待续

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
