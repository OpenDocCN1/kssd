
```c
static const u8* trampoline_fmt_64 =
"\n"
"/* --- AFL TRAMPOLINE (64-BIT) --- */\n"
"\n"
".align 4\n"
"\n"
"leaq -(128+24)(%%rsp), %%rsp\n"
"movq %%rdx,  0(%%rsp)\n"
"movq %%rcx,  8(%%rsp)\n"
"movq %%rax, 16(%%rsp)\n"
"movq $0x%08x, %%rcx\n"
"call __afl_maybe_log\n"
"movq 16(%%rsp), %%rax\n"
"movq  8(%%rsp), %%rcx\n"
"movq  0(%%rsp), %%rdx\n"
"leaq (128+24)(%%rsp), %%rsp\n"
"\n"
"/* --- END --- */\n"
;
```

它首先保存了一部分将要被破坏的寄存器，然后调用了`__afl_maybe_log`来记录路径的发现。该函数同样是由汇编编写的，但我们可以用一些其他工具来反编译它：

```c
char __fastcall _afl_maybe_log(__int64 a1, __int64 a2, __int64 a3, __int64 a4)
{
  char v4; // of
  char v5; // al
  __int64 v6; // rdx
  __int64 v7; // rcx
  char *v9; // rax
  int v10; // eax
  void *v11; // rax
  int v12; // edi
  __int64 v13; // rax
  __int64 v14; // rax
  __int64 v15; // [rsp-10h] [rbp-180h]
  char v16; // [rsp+10h] [rbp-160h]
  __int64 v17; // [rsp+18h] [rbp-158h]

  v5 = v4;
  v6 = _afl_area_ptr;
  if ( !_afl_area_ptr )
  {
    if ( _afl_setup_failure )
      return v5 + 127;
    v6 = _afl_global_area_ptr;
    if ( _afl_global_area_ptr )
    {
      _afl_area_ptr = _afl_global_area_ptr;
    }
    else
    {
      v16 = v4;
      v17 = a4;
      v9 = getenv("__AFL_SHM_ID");
      if ( !v9 || (v10 = atoi(v9), v11 = shmat(v10, 0LL, 0), v11 == -1LL) )
      {
        ++_afl_setup_failure;
        v5 = v16;
        return v5 + 127;
      }
      _afl_area_ptr = v11;
      _afl_global_area_ptr = v11;
      v15 = v11;
      if ( write(199, &_afl_temp, 4uLL) == 4 )
      {
        while ( 1 )
        {
          v12 = 198;
          if ( read(198, &_afl_temp, 4uLL) != 4 )
            break;
          LODWORD(v13) = fork();
          if ( v13 < 0 )
            break;
          if ( !v13 )
            goto __afl_fork_resume;
          _afl_fork_pid = v13;
          write(199, &_afl_fork_pid, 4uLL);
          v12 = _afl_fork_pid;
          LODWORD(v14) = waitpid(_afl_fork_pid, &_afl_temp, 0);
          if ( v14 <= 0 )
            break;
          write(199, &_afl_temp, 4uLL);
        }
        _exit(v12);
      }
      __afl_fork_resume:
      close(198);
      close(199);
      v6 = v15;
      v5 = v16;
      a4 = v17;
    }
  }
  v7 = _afl_prev_loc ^ a4;
  _afl_prev_loc ^= v7;
  _afl_prev_loc = _afl_prev_loc >> 1;
  ++*(v6 + v7);
  return v5 + 127;
}
```

前面的一大段代码其实都是为了去建立我们在上文所说的“共享内存”，在完成初始化后调用最后这么一小段代码进行记录：

```c
v7 = _afl_prev_loc ^ a4;
_afl_prev_loc ^= v7;
_afl_prev_loc = _afl_prev_loc >> 1;
++*(v6 + v7);
```

此处`v6`即为共享内存的地址，而`a4`为`cur_location`，因此`v7=cur_location ^ prev_location`，它将作为索引，使得共享内存中的对应偏移处的值增加。而在`fuzzer`部分就可以通过检查这块内存来发现是否有新路径被得到了。

另外，`_afl_prev_loc = _afl_prev_loc >> 1;`的目的是为了避开`A->A`和`B->B`以及`A->B`和`B->A`被识别为相同路径的情况。

## 其他阅读材料

> sakuraのAFL源码全注释
>
> 912K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6r3g2J5L8X3q4D9M7$3q4C8N6i4u0S2x3e0y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6S2Q4x3V1j5J5x3#2)9J5c8X3q4X3L8q4)9J5c8R3`.`.

> fuzzer AFL 源码分析
>
> c11K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6s2c8S2L8X3N6Q4x3X3g2U0L8$3#2Q4x3V1k6#2M7$3g2J5i4K6u0r3k6U0q4@1j5h3)9`.

> AFL二三事——源码分析
>
> 9ecK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4m8W2M7W2)9J5k6i4y4W2k6h3u0#2k6#2)9J5k6h3!0J5k6#2)9J5c8U0p5%4x3K6u0Q4x3V1k6Q4x3U0y4S2k6X3I4Q4x3X3c8S2k6X3I4Q4x3X3c8S2M7$3x3`.

> AFL漏洞挖掘技术漫谈（一）：用AFL开始你的第一次Fuzzing
>
> 5dbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4m8W2M7W2)9J5k6i4y4W2k6h3u0#2k6#2)9J5k6h3!0J5k6#2)9J5c8U0R3@1x3g2)9J5c8R3`.`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#自动化挖掘](https://bbs.kanxue.com/forum-150-1-155.htm)

[#Fuzz](https://bbs.kanxue.com/forum-150-1-157.htm)

[#Linux](https://bbs.kanxue.com/forum-150-1-161.htm)
