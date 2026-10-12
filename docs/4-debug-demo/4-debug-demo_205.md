
# ZIP PASSWORD FINDER

> 原文：[`www.pediy.com/kssd/pediy03/forum899.htm`](https://www.pediy.com/kssd/pediy03/forum899.htm)

最近在华军下载了一个 `ZIP PASSWORD FINDER`，是一个破解 `WinZIP` 密码的软件。虽然是免费软件不用注册，但每次运行都会出现一个讨厌的广告窗口。经过脱壳后，在 `TRW2000` 下我已经找到了关键 `Call`，但改成 `NOP` 后马上就非法操作，而且也没有发现可以跳过此 `Call` 的地方，请各位大侠赐教!!!!

娃娃(NYDoll)

## ZIP Password Finder v 1

用 `CASPR` 脱壳后，`EXESCOPE` 打开脱壳的 `EXE` 文件，在 `RCDATA\taboutform` 下，找 `object Timer1: TTimer`，将 `Interval = 100000` 改为 `1`，勉强解决。`4dd466d0e1e0cc123afbe504da2462ca`
