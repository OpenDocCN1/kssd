
# 无名小兵 V1.45

> 原文：[`www.pediy.com/kssd/pediy05/pediy50191.htm`](https://www.pediy.com/kssd/pediy05/pediy50191.htm)

## 基本信息

* **破解对象**：`无名小兵 V1.45` `VB5` `PCODE`
* **使用工具**：`ONLY SOFTICE 4.05`
* **辅助分析**：可以用 `exdec` 辅助分析
* **加密形式**：注册后校验，在 `WINDOWS` 目录下形成一个 `FILE！`

## 断点设置

我下的断点是：

```
bpx 799b4390
```

其实在好多地方都可以在 `VB` 里看到假的注册码：

* `799ad9a7` 可以到到假注册码
* `799b5a2e` 可以到到假注册码

## exdec 辅助分析资料

以下是 `exdec` 的辅助分析资料：

```
4292FB: 6b FLdI2                    local_00CE
4292FE: Lead2/3d Close              
429300: 04 FLdRfVar                 local_00C8
429303: 33 LdFixedStr              此处对应 VB，可以看到你输入的注册码
429306: 23 FStStrNoPop             local_015C
429309: 21 FLdPrThis              
42930A: 0f VCallAd                 text
42930D: 19 FStAdFunc               local_00D4
429310: 08 FLdPr                   local_00D4
429313: 0d VCallHresult            put__ipropTEXTEDIT
429318: 6c ILdRf                   local_015C
42931B: 04 FLdRfVar                local_00C8
42931E: 47 StFixedStr             
429321: 2f FFree1Str               local_015C
429324: 1a FFree1Ad                local_00D4
429327: f5 LitI4:                  0xff 255 (....)
42932C: 04 FLdRfVar                local_00F8
42932F: 0a ImpAdCallFPR4:          _rtcSpaceVar
429334: 04 FLdRfVar                local_00F8
429337: 60 CStrVarTmp             
429338: 31 FStStr                  local_008C
42933B: 35 FFree1Var               local_00F8
42933E: f5 LitI4:                  0xff 255 (....)
429343: 04 FLdRfVar                local_00F8
429346: 0a ImpAdCallFPR4:          _rtcSpaceVar
42934B: 04 FLdRfVar                local_00F8
42934E: 60 CStrVarTmp             
42934F: 31 FStStr                  local_0090
429352: 35 FFree1Var               local_00F8
429355: 6c ILdRf                   local_0090
429358: 4a FnLenStr               
429359: 6c ILdRf                   local_0090
42935C: 04 FLdRfVar                local_0170
42935F: 34 CStr2Ansi              
429360: 6c ILdRf                   local_0170
429363: f5 LitI4:                  0x0 0 (....)
429368: 59 PopTmpLdAdStr   local_016C
42936B: f5 LitI4:                  0x0 0 (....)
429370: 59 PopTmpLdAdStr   local_0168
429373: 04 FLdRfVar                local_00A0
429376: Lead1/22 CI4Var           
429378: 59 PopTmpLdAdStr   local_0164
42937B: 6c ILdRf                   local_008C
42937E: 4a FnLenStr               
42937F: 6c ILdRf                   local_008C
429382: 04 FLdRfVar                local_0160
429385: 34 CStr2Ansi              
429386: 6c ILdRf                   local_0160
429389: 1b LitStr:                 c:\
42938C: 04 FLdRfVar                local_015C
42938F: 34 CStr2Ansi              
429390: 6c ILdRf                   local_015C
429393: 0a ImpAdCallFPR4:          SetWindowPos
429398: 3c SetLastSystemError     
429399: 6c ILdRf                   local_0160
42939C: 04 FLdRfVar                local_008C
42939F: Lead1/58 CStr2Uni         
4293A1: 6c ILdRf                   local_0164
4293A4: Lead2/69 CVarI4            local_00E8
4293A8: Lead2/00 FStVarCopy       
4293AC: 6c ILdRf                   local_0170
4293AF: 04 FLdRfVar                local_0090
4293B2: Lead1/58 CStr2Uni         
4293B4: 32 FFreeStr
4293BD: 04 FLdRfVar                local_00A0
4293C0: Lead2/fe CStrVarVal        local_015C
4293C4: 21 FLdPrThis              
4293C5: 0f VCallAd                 text
4293C8: 19 FStAdFunc               local_00D4
4293CB: 08 FLdPr                   local_00D4
4293CE: 0d VCallHresult            put__ipropTEXTEDIT
4293D3: 2f FFree1Str               local_015C
4293D6: 1a FFree1Ad                local_00D4
4293D9: 04 FLdRfVar                local_015C
4293DC: 21 FLdPrThis              
4293DD: 0f VCallAd                 text
4293E0: 19 FStAdFunc               local_00D4
4293E3: 08 FLdPr                   local_00D4
4293E6: 0d VCallHresult            get__ipropTEXTEDIT
4293EB: 3e FLdZeroAd               local_015C
4293EE: 46 CVarStr                 local_00F8 注册码 1？
4293F1: 04 FLdRfVar                local_0118
4293F4: 0a ImpAdCallFPR4:          _rtcTrimVar
4293F9: 04 FLdRfVar                local_0118
4293FC: Lead1/f6 FStVar           
429400: 1a FFree1Ad                local_00D4
429403: 35 FFree1Var               local_00F8
429406: f5 LitI4:                  0x2 2 (....)
42940B: 04 FLdRfVar                local_00A0
42940E: 04 FLdRfVar                local_00F8
429411: 0a ImpAdCallFPR4:          _rtcLeftCharVar
429416: 04 FLdRfVar                local_00F8
429419: Lead1/45 FnCIntVar        
42941B: f4 LitI2_Byte:             0x63 99 (c)
42941D: b1 MulI2                  
42941E: Lead0/fc CStrI2           
429420: 23 FStStrNoPop             local_015C 注册码 2？
429423: 04 FLdRfVar                local_00A0
429426: Lead1/4b FnCStrVar        
429428: 23 FStStrNoPop             local_0160 注册码 1？
42942B: 2a ConcatStr              
42942C: 46 CVarStr                 local_0118 注册码 2+注册码 1
42942F: Lead1/f6 FStVar           
429433: 32 FFreeStr
42943A: 36 FFreeVar
429441: 04 FLdRfVar                local_00A0
429444: 04 FLdRfVar                local_015C
429447: 21 FLdPrThis              
429448: 0f VCallAd                 text
42944B: 19 FStAdFunc               local_00D4
42944E: 08 FLdPr                   local_00D4
429451: 0d VCallHresult            get__ipropTEXTEDIT
429456: 3e FLdZeroAd               local_015C
429459: 46 CVarStr                 local_00F8 你输入的注册码
42945C: 04 FLdRfVar                local_0118
42945F: 0a ImpAdCallFPR4:          _rtcTrimVar
429464: 04 FLdRfVar                local_0118
429467: Lead1/4b FnCStrVar        
429469: 46 CVarStr                 local_0138 你输入的注册码
42946C: 5d HardType               
42946D: Lead0/33 EqVarBool        
42946F: 1a FFree1Ad                local_00D4
429472: 36 FFreeVar
42947D: 1c BranchF:                4294BC       如果真码和假码不同就跳走
429480: f4 LitI2_Byte:             0x0 0 (.)
429482: 21 FLdPrThis              
429483: 0f VCallAd                 (object 3 )
429486: 19 FStAdFunc               local_00D4
429489: 08 FLdPr                   local_00D4
42948C: 0d VCallHresult            禚 d
429491: 1a FFree1Ad                local_00D4
429494: f4 LitI2_Byte:             0x0 0 (.)
429496: 21 FLdPrThis              
429497: 0f VCallAd                 menu
42949A: 19 FStAdFunc               local_00D4
42949D: 08 FLdPr                   local_00D4
4294A0: 0d VCallHresult            put__ipropENABLEDMENU
4294A5: 1a FFree1Ad                local_00D4
4294A8: f4 LitI2_Byte:             0x0 0 (.)
4294AA: 21 FLdPrThis              
4294AB: 0f VCallAd                 menu
4294AE: 19 FStAdFunc               local_00D4
4294B1: 08 FLdPr                   local_00D4
4294B4: 0d VCallHresult            put__ipropENABLEDMENU
4294B9: 1a FFree1Ad                local_00D4
4294BC: 04 FLdRfVar                local_015C
4294BF: 04 FLdRfVar                local_00D4
4294C2: 05 ImpAdLdRf:              42c63c
4294C5: 24 NewIfNullPr             41e8f0
4294C8: 0d VCallHresult            CVBApplication::get_App
4294CD: 08 FLdPr                   local_00D4
4294D0: 0d VCallHresult            CVBApplication::geD4$?(id
4294D5: 6c ILdRf                   local_015C
4294D8: 4a FnLenStr               
4294D9: f5 LitI4:                  0x3 3 (....)
4294DE: db GtI4                   
4294DF: 2f FFree1Str               local_015C
4294E2: 1a FFree1Ad                local_00D4
```
