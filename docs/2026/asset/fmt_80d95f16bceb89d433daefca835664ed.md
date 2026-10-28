
```asm
004A9E37    8385 50FFFFFF 0>add dword ptr ss:[ebp-0xB0],0x6      ; len3 += 6
004A9E3E    83BD 50FFFFFF 0>cmp dword ptr ss:[ebp-0xB0],0x8
004A9E45    7C 6D           jl short virus.004A9EB4              ; if(len3 <= 8) goto 004A9EB4
004A9E47    83AD 50FFFFFF 0>sub dword ptr ss:[ebp-0xB0],0x8      ; len3 -= 8
004A9E4E    8B8D 50FFFFFF   mov ecx,dword ptr ss:[ebp-0xB0]
004A9E54    8B85 48FFFFFF   mov eax,dword ptr ss:[ebp-0xB8]      ; lenshl = v383
004A9E5A    D3E8            shr eax,cl                           ; lenshl >>= (len3 & 0xFF)
004A9E5C    8985 4CFFFFFF   mov dword ptr ss:[ebp-0xB4],eax      ; len\_0123wxyz = lenshl
004A9E62    8B8D 50FFFFFF   mov ecx,dword ptr ss:[ebp-0xB0]      ; ecx = len3
004A9E68    B8 01000000     mov eax,0x1                          ; lenshl =1
004A9E6D    D3E0            shl eax,cl                           ; lenshl <<= (len3 & 0xFF)
004A9E6F    50              push eax                             ; what = eax
004A9E70    8B85 48FFFFFF   mov eax,dword ptr ss:[ebp-0xB8]      ; eax = len\_0123wxyz << 6
004A9E76    5A              pop edx                              ; edx = what
004A9E77    8BCA            mov ecx,edx                          ; ecx = what
004A9E79    99              cdq
004A9E7A    F7F9            idiv ecx                             ; eax = 商, edx = 余
004A9E7C    8995 48FFFFFF   mov dword ptr ss:[ebp-0xB8],edx      ; 余 = edx
004A9E82    8B85 4CFFFFFF   mov eax,dword ptr ss:[ebp-0xB4]      ; eax = shang
004A9E88    25 FF000080     and eax,0x800000FF                   ; eax &= 0x800000FF
004A9E8D    79 07           jns short virus.004A9E96             ; if(0 < eax) goto 004A9E96
004A9E8F    48              dec eax                              ; eax--
004A9E90    0D 00FFFFFF     or eax,-0x100                        ; eax |= -0x100
004A9E95    40              inc eax                              ; eax++
004A9E96    8985 4CFFFFFF   mov dword ptr ss:[ebp-0xB4],eax      ; shang = shang
004A9E9C    8B45 08         mov eax,dword ptr ss:[ebp+0x8]       ; eax = 加密数据
004A9E9F    8B95 44FFFFFF   mov edx,dword ptr ss:[ebp-0xBC]      ; edx = len\_0xbc
004A9EA5    8B8D 4CFFFFFF   mov ecx,dword ptr ss:[ebp-0xB4]      ; ecx = shang
004A9EAB    880C10          mov byte ptr ds:[eax+edx],cl         ; \*(加密数据+len\_0xbc) = shang
004A9EAE    FF85 44FFFFFF   inc dword ptr ss:[ebp-0xBC]          ; len\_0xbc++
004A9EB4    43              inc ebx                              ; i++
004A9EB5    4E              dec esi                              ; esi--
004A9EB6  ^ 0F85 32FFFFFF   jnz virus.004A9DEE                   ; if(0 < esi) goto 004A9DEE
```

## 尝试写的C语言代码3

```c
void main()
{
  int str[] = { 0x55,  0x4D,  0x35,  0x67,  0x53,  0x4E,  0x31,  0x61,  0x52,  0x4D,  0x6E,  0x74,  0x53...... }; // str = 测试数据3

  int str2[] = {0x30, 0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37, 0x38, 0x39, 0x41, 0x42, 0x43, 0x44, 0x45, 0x46, 
    0x47, 0x48, 0x49, 0x4A, 0x4B, 0x4C, 0x4D, 0x4E, 0x4F, 0x50, 0x51, 0x52, 0x53, 0x54, 0x55, 0x56, 0x57, 0x58, 
    0x59, 0x5A, 0x61, 0x62, 0x63, 0x64, 0x65, 0x66, 0x67, 0x68, 0x69, 0x6A, 0x6B, 0x6C, 0x6D, 0x6E, 0x6F, 0x70, 
    0x71, 0x72, 0x73, 0x74, 0x75, 0x76, 0x77, 0x78, 0x79, 0x7A, 0x2B, 0x2F};  // 0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz+/

  int i, len, len_0123wxyz, len3 = 0, len_BC = 0;
  int v383=0;
  int *p = str;
  int *p2 = str2;

  for (i = 0, len = 1344/*0x9FD70*/; 0 < len; i++, len--)
  {
    for( len_0123wxyz = 1; len_0123wxyz < 0x41; len_0123wxyz++)
    {
      int str3 = (*(p+i) & 0xFF);
      if (str3 == *(p2+len_0123wxyz))
      {
        break;
      }
    }

    if( 0 <= len_0123wxyz)
    {
      int v384 = v383 = len_0123wxyz + (v383<<6);
      len3 += 6;

      if(8 <= len3)
      {
        len3 -= 8;

        len_0123wxyz = v383 >> (len3 & 0xFF);
        v383 = (signed int)v383 % (1 << len3);
        int shang = v384/(1 << len3);
        shang &= 0x800000FF;

        if(0 < shang)
        {
          *(p+len_BC) = shang & 0xFF;
          if (len_BC%16==0)
          {
            printf("\n");
          }
          printf("%02X", *(p+len_BC));

          len_BC++;
        }
        else
        {
          int c = *p;
          c--;
          c |= -0x100;
          c++;
          //    (((*p)--) |= -0x100)++;
        }
      }
    }
  }
  printf("\n");
}
```

配图

![](https://bbs.kanxue.com/upload/attach/201405/520001_cmn3dxxx1k74mh1.png)

至此PE文件就解密出来了。可通过 OD脚本 `[DMA A71715, 9FD70, "C:\xxxx.bin"]` 将核心PE文件抓取出来

写C代码的时候还发现一个有趣的事情，应该是我看书太少了吧：

`IDA F5`：`v383 %= (1 << len3);`

我写的：`v383 &= (1 << (len3-1));`

两个结果是一样的。才知道 `X % 2^n = X & (2^n - 1)`《与运算和取余之间的关系》

---

解密出来的PE文件才是真正的病毒程序母体。。。我不太会分析，马马虎虎整理如下

## 1，样本的基本信息。

- 有无壳：有：ASPack v2.12
- 编写语言：Borland Delphi 6.0 - 7.0
- 样本MD5：`A4EA96485DA00D33BADDBCF87034DEAD`
- 样本大小：505 KB (517,133 字节)
- 样本编写国家：可能是俄罗斯[依据资源语言为俄语]

## 2，样本的行为。

1. 反调试，发现有调试或抓包软件不作任何事情直接自删除。
2. 使用`GetStartupInfo`检查自己是否被调试。
3. 动态加载DLL
4. 核心代码加密
5. 获取地区+语言
6. 读写注册表数据
7. 运行后自动删除
8. 获取系统信息
9. 修改`hosts`文件
10. 写入开机注册表
11. 网络连接收发数据
12. 向自身进程注入代码
13. 可能会下载其他程序
14. 修改DNS服务器主机名
15. 修改mozilla浏览器信息
16. 以挂起方式创建自身进程
17. 反OD调试`FindWindowA("OLLYDBG", 0)`
18. 搜索硬件信息`"IBM-"` `"MAXTOR"` `"Maxtor"` `"WDC "`
19. 通过`IsWow64Process`判断系统是否为64位
20. 查找文件夹`System Volume Information`
21. 创建文件`c:\cgvi5r6i\vgdgfd.72g`和`\\.\Ip`
22. 根据不同参数执行不同代码`"shortcut"` / `"opt"`
23. 复制自身到`%TEMP%`目录下[随机2位数内]`xx.tmp`
24. 写入`%systemroot%\system32\drivers\fastfat.sys`驱动文件
25. 创建桌面快捷方式`%USERPROFILE%\Desktop\Computer.lnk`
26. 创建一个命名的事件对象`EventName = "{9D723E3C-5DD2-43a4-A593-6C4327DA79DE}"`
27. 获得系统版本：`"wv95";"wv98";"wvNT";"wv2k";"wvME";"wvXP";"wv2k3";"wvVista";"wv2k8";"wv7"; "wvUnknown";`
28. 伪装adobe flash player升级程序`FindResource(a1, 0x71, "IMAGE")`; 资源为伪装的adobe flash player升级程序[英文界面]--------->>`adobe_flash.jpg`

    ![](https://bbs.kanxue.com/upload/attach/201405/520001_wo1207ne3uafu6t.jpg)

29. 连接以下可疑网址与IP：

    `94.23.116.81` / `217.23.15.124` / `69.57.173.222` / `212.117.176.187`

    `findgala.com` / `update1.randomstring.com` / `update1.randomstring.com/update_c1eec.exe`

## 3，样本的详细分析。

1. 判断有无以下程序和注册表信息，判断电脑用户是否为调试机，是调试机不做任何木马行为，停止运行并自动删除：--------->>`is调试机.jpg`

    ![](https://bbs.kanxue.com/upload/attach/201405/520001_2hmmh2w5zail6v7.jpg)

    - `cv.exe`
    - `irise.exe`
    - `IrisSvc.exe`
    - `wireshark.exe`
    - `dumpcap.exe`
    - `ZxSniffer.exe`
    - `Aircrack-ng Gui.exe`
    - `observer.exe`
    - `tcpdump.exe`
    - `SandboxieDcomLaunch.exe`
    - `SUPERAntiSpyware.exe`
    - `ERUNT.exe`
    - `ERDNT.exe`
    - `EtherD.exe`
    - `Sniffer.exe`
    - `CamtasiaStudio.exe`
    - `CamRecorder.exe`
    - `windbg.exe`
    - `DrvLoader.exe`
    - `SymRecv.exe`
    - `Syser.exe`
    - `apis32.exe`
    - `VBoxService.exe`
    - `VBoxTray.exe`
    - `SbieSvc.exe`
    - `SbieCtrl.exe`
    - `SandboxieRpcSs.exe`
    - `WinDump.exe`
    - `wspass.exe`
    - `Regshot.exe`
    - `ollydbg.exe`
    - `PEBrowseDbg.exe`

    注册表：

    - `Software\CommView`
    - `SYSTEM\CurrentControlSet\Services\IRIS5`
    - `Software\eEye Digital Security`
    - `SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\Wireshark`
    - `SOFTWARE\Microsoft\Windows\CurrentVersion\AppPaths\wireshark.exe`
    - `SOFTWARE\ZxSniffer`
    - `SOFTWARE\Cygwin`
    - `SOFTWARE\B Labs\Bopup Observer`
    - `AppEvents\Schemes\Apps\Bopup Observer`
    - `Software\B Labs\BopupObserver`
    - `SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\Win Sniffer_is1`
    - `Software\Win Sniffer`
    - `SOFTWARE\Classes\PEBrowseDotNETProfiler.DotNETProfiler`
    - `Software\Microsoft\Windows\CurrentVersion\Explorer\MenuOrder\Start Menu2\Programs\Debugging Tools for Windows (x86)`
    - `SYSTEM\CurrentControlSet\Services\SDbgMsg`
    - `Software\Microsoft\Windows\CurrentVersion\Explorer\MenuOrder\Start Menu2\Programs\APIS32`
    - `Software\Syser Soft`
    - `SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\APIS32`
    - `SOFTWARE\APIS32`
    - `SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\Oracle VM VirtualBox Guest Additions`
    - `SYSTEM\CurrentControlSet\Services\VBoxGuest`
    - `SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\Sandboxie`
