
# W32DSM 黄金版

> 原文：[`www.pediy.com/kssd/pediy03/forum341.htm`](https://www.pediy.com/kssd/pediy03/forum341.htm)

你的 `W32DSM` 黄金版已过期？先搞定它吧，fire your `regmon` and run the program，

what? `regmon` closed but `W32DSM` is still running? ah Anti `regmon`!

ok, let's go, replace all `"regvxd"` in `regmon.exe` and `regvxd.vxd` with `"iegvxd"`(just what you like!:). then rename the `regvxd.vxd` file into `iegvxd.vxd`, well, launch `regmon` again and run `W32DSM`, do you find the key? like `HKCR\CLSID\{943BE8AD-85F9-0EC3-C5E4-F122FDB46E6A}` (may by not the same to yours), rename it and run `W32DSM` once again, did you pass now?

this can be used in some anti `trw2000` protector, common anti debug routine is `CreateFile \\\.\trw2000` and `CreateFile \\\.\trwdebug`, so you can change... (i don't want talk more about this! you can do that).

now, no one can catch you(`dbpe` v1.5b3 is not the exception, not our target this time), ok go on!

this is sure not your will, well, let's shell(unshell?) the program.

first, we must know the entry point, `Blast Wave2000` is our good choose, but unfortunately, it not works on my computer this time, what else?

One way is debug `W32DSM`, My God! let's look at `W32DSM`'s section table, `.CODE`! we know the entry point `401000h`! load `W32DSM` in `trw200`, do a command `bpx 401000`, and let it go, we break at `401000`, dump it full!

the rest thing is rebuild import table, run `imp_list`, select `W32DSM` and click Rebuild import button, ignore the error message, you can get four files: `Import0.bin`, `Import1.bin`, `Import2.bin` and `Address.txt`.

Load `Import1.bin` and `W32DSM` into `Hex Workshop`, copy all `Import1.bin`'s data and goto `W32DSM` OFFSET `3C200h`(`.idata` section), high light `0xb38h` bytes, `ctrl + v`, save it, that's all! forgive my poor english, good luck!

大头成   4dd466d0e1e0cc123afbe504da2462ca
