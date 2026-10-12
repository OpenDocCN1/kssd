
# Pegtop XFader 4.02

> 原文：[`www.pediy.com/kssd/pediy02/forum119.htm`](https://www.pediy.com/kssd/pediy02/forum119.htm)

`Pegtop XFader`

Version 4.02 - 08 May 2000

`XFader` creates seamless textures from any image you want. It's easy to manipulate wallpapers and other images. Useful for backgrounds of websites, your Windows desktop or to generate textures for 3D rendering.

Visit our website at `www.pegtop.de`

## 注册码

设断点 `bpx hmemcpy`

程序被中断后,按 `F12` 键 12 次,在此处用 `d eax` 可看到输入的注册码.

再设断点 `bpx eax`,当程序再次被中断时,是一比较语句,用 `d esi` 和 `d edi` 可看到输入的注册码和实际的注册码.

下面是注册码生成过程:

`LiuTong`

程序首先将用户名转成大写:`LIUTONG`(对应 HEX 为 `4C 49 55 54 4F 4E 47`)

再将用户名补足为 8 个字符:`LIUTONGL`(对应 HEX 为 `4C 49 55 54 4F 4E 47 4C`)

此 8 个子符与系统内部的 8 个子符组合成:

| 序号 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | A | B | C | D | E | F | 10 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 字符 | P | X | F | 1 | 2 | O | K | T | L | I | U | T | O | N | G | L |
| HEX | 50 | 58 | 46 | 31 | 32 | 4F | 4B | 54 | 4C | 49 | 55 | 54 | 4F | 4E | 47 | 4C |

系统内部字符 `<--------` `------->` 用户名

系统内部另有 8 组数据:

- `1 3 5 7` 根据此行数据计算得到第 1 位注册码
- `2 4 A C` 根据此行数据计算得到第 2 位注册码
- `9 B D F` 根据此行数据计算得到第 3 位注册码
- `6 8 E 10` 根据此行数据计算得到第 4 位注册码
- `1 10 2 F` 根据此行数据计算得到第 5 位注册码
- `3 E 4 D` 根据此行数据计算得到第 6 位注册码
- `5 C 6 B` 根据此行数据计算得到第 7 位注册码
- `7 A 8 9` 根据此行数据计算得到第 8 位注册码

举例说明:

- `50 XOR 46 --> 16 ROL 32 --> 58 XOR 4B --> 13` 除 `0A` 余数 `9`
- `58 XOR 31 --> 69 ROL 49 --> D2 XOR 54 --> 86` 除 `0A` 余数 `4`
- `4C XOR 55 --> 19 ROL 4F --> 8C XOR 47 --> CB` 除 `0A` 余数 `3`
- `4F XOR 54 --> 1B ROL 4E --> C6 XOR 4C --> 8A` 除 `0A` 余数 `8`
- `50 XOR 4C --> 1C ROL 58 --> 1C XOR 47 --> 5B` 除 `0A` 余数 `1`
- `46 XOR 4E --> 08 ROL 31 --> 10 XOR 4F --> 5F` 除 `0A` 余数 `5`
- `32 XOR 54 --> 66 ROL 4F --> 33 XOR 55 --> 66` 除 `0A` 余数 `2`
- `4B XOR 49 --> 02 ROL 54 --> 20 XOR 4C --> 6C` 除 `0A` 余数 `8`

整理结果:

- 用户名: `LiuTong`
- 注册码: `94381528`

# Pegtop WaterWall

`Pegtop WaterWall`

Version 4.00 - 20 Oct 1999

`WaterWall` is a wallpaper manager for you desktop including various style effects, a slide show, sounds and more.

Visit our website at `www.pegtop.de`

## 注册码

设断点 `bpx hmemcpy`

程序被中断后,按 `F12` 键 12 次,在此处用 `d eax` 可看到输入的注册码.

再设断点 `bpx eax`,当程序再次被中断时,是一比较语句,用 `d esi` 和 `d edi` 可看到输入的注册码和实际的注册码.

下面是注册码生成过程:

`LiuTong`

程序首先将用户名转成大写:`LIUTONG`(对应 HEX 为 `4C 49 55 54 4F 4E 47`)

再将用户名补足为 8 个字符:`LIUTONGL`(对应 HEX 为 `4C 49 55 54 4F 4E 47 4C`)

此 8 个子符与系统内部的 8 个子符组合成:

| 序号 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | A | B | C | D | E | F | 10 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 字符 | P | W | W | 2 | 4 | J | A | N | L | I | U | T | O | N | G | L |
| HEX | 50 | 57 | 57 | 32 | 34 | 4A | 41 | 4E | 4C | 49 | 55 | 54 | 4F | 4E | 47 | 4C |

系统内部字符 `<--------` `------->` 用户名

系统内部另有 8 组数据:

- `1 3 5 7` 根据此行数据计算得到第 1 位注册码
- `2 4 A C` 根据此行数据计算得到第 2 位注册码
- `9 B D F` 根据此行数据计算得到第 3 位注册码
- `6 8 E 10` 根据此行数据计算得到第 4 位注册码
- `1 10 2 F` 根据此行数据计算得到第 5 位注册码
- `3 E 4 D` 根据此行数据计算得到第 6 位注册码
- `5 C 6 B` 根据此行数据计算得到第 7 位注册码
- `7 A 8 9` 根据此行数据计算得到第 8 位注册码

整理结果:

- 用户名: `LiuTong`
- 注册码: `98373328`

# Pegtop Flagimation

`Pegtop Flagimation`

Version 1.05 - 11 Dec 1999

`Flagimation` creates animated 3D flags you can save as `gif` images. Just copy `Flagimation.exe` to your harddisk and execute it.

Visit our homepage at `www.pegtop.de`

挺有意思的程序

## 注册码

设断点 `bpx hmemcpy`

程序被中断后,按 `F12` 键 12 次,在此处用 `d eax` 可看到输入的注册码.

再设断点 `bpx eax`,当程序再次被中断时,是一比较语句,用 `d esi` 和 `d edi` 可看到输入的注册码和实际的注册码.

下面是注册码生成过程:

`LiuTong`

程序首先将用户名转成大写:`LIUTONG`(对应 HEX 为 `4C 49 55 54 4F 4E 47`)

再将用户名补足为 8 个字符:`LIUTONGL`(对应 HEX 为 `4C 49 55 54 4F 4E 47 4C`)

此 8 个子符与系统内部的 8 个子符组合成:

| 序号 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | A | B | C | D | E | F | 10 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 字符 | P | F | L | 2 | 8 | J | U | L | L | I | U | T | O | N | G | L |
| HEX | 50 | 46 | 4C | 32 | 38 | 4A | 55 | 4C | 4C | 49 | 55 | 54 | 4F | 4E | 47 | 4C |

系统内部字符 `<--------` `------->` 用户名

系统内部另有 8 组数据:

- `1 3 5 7` 根据此行数据计算得到第 1 位注册码
- `2 4 A C` 根据此行数据计算得到第 2 位注册码
- `9 B D F` 根据此行数据计算得到第 3 位注册码
- `6 8 E 10` 根据此行数据计算得到第 4 位注册码
- `1 10 2 F` 根据此行数据计算得到第 5 位注册码
- `3 E 4 D` 根据此行数据计算得到第 6 位注册码
- `5 C 6 B` 根据此行数据计算得到第 7 位注册码
- `7 A 8 9` 根据此行数据计算得到第 8 位注册码

计算方法同初学者(18)

整理结果:

- 用户名: `LiuTong`
- 注册码: `38354181`
- 校验码: `4dd466d0e1e0cc123afbe504da2462ca`
