
```
0x7ffff7f908f0 <_IO_2_1_stdin_+16>: 0x7ffff7f90900 <_IO_2_1_stdin_+32> 0x7ffff7f90900 <_IO_2_1_stdin_+32>
0x7ffff7f90900 <_IO_2_1_stdin_+32>: 0x7ffff7f90900 <_IO_2_1_stdin_+32> 0x7ffff7f90900 <_IO_2_1_stdin_+32>
0x7ffff7f90910 <_IO_2_1_stdin_+48>: 0x7ffff7f90900 <_IO_2_1_stdin_+32> 0x7ffff7f90900 <_IO_2_1_stdin_+32>
0x7ffff7f90920 <_IO_2_1_stdin_+64>: 0x7ffff7f90964 <_IO_2_1_stdin_+132> 0x0
0x7ffff7f90930 <_IO_2_1_stdin_+80>: 0x0     0x0
0x7ffff7f90940 <_IO_2_1_stdin_+96>: 0x0     0x0
0x7ffff7f90950 <_IO_2_1_stdin_+112>: 0x0     0xffffffffffffffff
0x7ffff7f90960 <_IO_2_1_stdin_+128>: 0xa000000       0x7ffff7f92720 <_IO_stdfile_0_lock>
0x7ffff7f90970 <_IO_2_1_stdin_+144>: 0xffffffffffffffff      0x0
0x7ffff7f90980 <_IO_2_1_stdin_+160>: 0x7ffff7f909c0 <_IO_wide_data_0> 0x0
0x7ffff7f90990 <_IO_2_1_stdin_+176>: 0x0     0x0
0x7ffff7f909a0 <_IO_2_1_stdin_+192>: 0xffffffff      0x0
0x7ffff7f909b0 <_IO_2_1_stdin_+208>: 0x0     0x7ffff7f8f030 <_IO_file_jumps>

pwndbg> x/40xga &_IO_2_1_stdin_

0x7ffff7f908e0 <_IO_2_1_stdin_>: 0xfbad20ab      0x7ffff7f90900 <_IO_2_1_stdin_+32>
0x7ffff7f908f0 <_IO_2_1_stdin_+16>: 0x7ffff7f90900 <_IO_2_1_stdin_+32> 0x7ffff7f90900 <_IO_2_1_stdin_+32>
0x7ffff7f90900 <_IO_2_1_stdin_+32>: 0x7ffff7f90900 <_IO_2_1_stdin_+32> 0x7ffff7f90900 <_IO_2_1_stdin_+32>
0x7ffff7f90910 <_IO_2_1_stdin_+48>: 0x7ffff7f90900 <_IO_2_1_stdin_+32> 0x7ffff7f90900 <_IO_2_1_stdin_+32>
0x7ffff7f90920 <_IO_2_1_stdin_+64>: 0x7ffff7f90964 <_IO_2_1_stdin_+132> 0x0
0x7ffff7f90930 <_IO_2_1_stdin_+80>: 0x0     0x0
0x7ffff7f90940 <_IO_2_1_stdin_+96>: 0x0     0x0
```

[回复或点赞可查看完整内容](https://bbs.kanxue.com/#quick_reply_form)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#基础知识](https://bbs.kanxue.com/forum-171-1-180.htm)

[#漏洞机制](https://bbs.kanxue.com/forum-171-1-181.htm)

[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)
