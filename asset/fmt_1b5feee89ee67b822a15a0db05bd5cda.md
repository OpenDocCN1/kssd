
......(省略).........

```
51CA1A: f5 LitI4:                 0x2d0  720  (....)

51CA1F: aa AddI4

51CA20: 71 FStR4                  local_0090

51CA23: 00 LargeBos

51CA25: 6c ILdRf                  local_008C

51CA28: f5 LitI4:                 0x438  1080  (...8)

51CA2D: aa AddI4

51CA2E: 71 FStR4                  local_0090

51CA31: 00 LargeBos

51CA33: 00 LargeBos

51CA35: 04 FLdRfVar               local_00D0

51CA38: 21 FLdPrThis

51CA39: 0f VCallAd                menu

51CA3C: 19 FStAdFunc              local_00CC

51CA3F: 08 FLdPr                  local_00CC

51CA42: 0d VCallHresult           get__ipropMDIWINDOWMENU

51CA47: 6c ILdRf                  local_00D0＝＞取得注册码

51CA4A: 0b ImpAdCallI2            ＝＞注册码串反倒

51CA4F: 23 FStStrNoPop            local_00D4

51CA52: 1b LitStr:                wbregfilename

51CA55: 1b LitStr:                wbregfile

51CA58: 1b LitStr:                wbreg

51CA5B: 0a ImpAdCallFPR4:         衊帬＝＞写入到注册表中

51CA60: 32 FFreeStr

51CA67: 1a FFree1Ad               local_00CC

51CA6A: 00 LargeBos

51CA6C: 27 LitVar_Missing

51CA6F: 27 LitVar_Missing

51CA72: 3a LitVarStr:             ( local_0130 )

51CA77: 4e FStVarCopyObj          local_00F0

51CA7A: 04 FLdRfVar               local_00F0

51CA7D: f5 LitI4:                 0x40  64  (...@)

51CA82: 3a LitVarStr:             ( local_00A0 )

51CA87: 4e FStVarCopyObj          local_00B0

51CA8A: 04 FLdRfVar               local_00B0

51CA8D: 0a ImpAdCallFPR4:         ＝＞注册成功标志框

51CA92: 36 FFreeVar

51CA9D: 00 LargeBos

51CA9F: 27 LitVar_Missing

51CAA2: 0a ImpAdCallFPR4:

51CAA7: 73 FStFPR4

51CAAA: f4 LitI2_Byte:            0x18  24  (.)

51CAAC: eb CR8I2

51CAAD: 6e FLdFPR4

........(省略)..........

51D0AC: f5 LitI4:                 0x2d0  720  (....)

51D0B1: aa AddI4

51D0B2: 71 FStR4                  local_0090

51D0B5: 00 LargeBos

51D0B7: 6c ILdRf                  local_008C

51D0BA: f5 LitI4:                 0x438  1080  (...8)

51D0BF: aa AddI4

51D0C0: 71 FStR4                  local_0090

51D0C3: 00 LargeBos

51D0C5: 00 LargeBos

51D0C7: 00 LargeBos

51D0C9: 21 FLdPrThis

51D0CA: 0f VCallAd                menu

51D0CD: 19 FStAdFunc              local_00CC

51D0D0: 08 FLdPr                  local_00CC

51D0D3: 0d VCallHresult

51D0D8: 1a FFree1Ad               local_00CC

51D0DB: 00 LargeBos

51D0DD: 13 ExitProcHresult ＝＞注册码不正确则直接跳出
```

此外还还一个暗桩：

用 `rtcMsg` 下断，在弹出的注册窗口时，会断到这里，代码如下：

`Proc: 50598c`

```
5058D0: f5 LitI4:                 0x1  1  (....)

5058D5: 3a LitVarStr:             ( local_0094 ) 110

5058DA: 25 PopAdLdVar

5058DB: 1b LitStr:                wbregfilename

5058DE: 1b LitStr:                wbregfile

5058E1: 1b LitStr:                wbreg

5058E4: 0b ImpAdCallI2            衊帬＝＝＝＝＞从注册表中读取注册码

5058E9: 23 FStStrNoPop            local_0098

5058EC: 0b ImpAdCallI2

5058F1: 23 FStStrNoPop            local_00D0

5058F4: 05 ImpAdLdRf:             526030

5058F7: 28 LitVarI2:              ( local_00A8 ) 0x3fe  (1022)

5058FC: Lead0/ac IDvVar

505900: Lead1/4b FnCStrVar

505902: 23 FStStrNoPop            local_00BC

505905: 0b ImpAdCallI2

50590A: 23 FStStrNoPop            local_00C4

50590D: 76 ImpAdLdI4

505910: 50 CI4Str

505911: f5 LitI4:                 0x7  7  (....)

505916: c0 IDvI4                  ＝＞值除以 7

505917: Lead0/fe CStrI4

505919: 23 FStStrNoPop            local_00C0

50591C: 0b ImpAdCallI2            ＝＞反转为注册码

505921: 23 FStStrNoPop            local_00C8

505924: 2a ConcatStr

505925: 23 FStStrNoPop            local_00CC

505928: f5 LitI4:                 0x1  1  (....)

50592D: Lead3/fd FnInStr4

50592F: f5 LitI4:                 0x1  1  (....)

505934: d1 LtI4

505935: 32 FFreeStr

505946: 1c BranchF:               505988

505949: 04 FLdRfVar               local_0098

50594C: f5 LitI4:                 0xd3  211  (....)

505951: 05 ImpAdLdRf:             5267dc

505954: 24 NewIfNullPr            40bfe4

505957: 0d VCallHresult

50595C: 27 LitVar_Missing

50595F: 27 LitVar_Missing

505962: 3a LitVarStr:             ( local_0094 )

505967: 4e FStVarCopyObj          local_00E0

50596A: 04 FLdRfVar               local_00E0

50596D: f5 LitI4:                 0x40  64  (...@)

505972: 3e FLdZeroAd              local_0098

505975: 46 CVarStr                local_00B8

505978: 0a ImpAdCallFPR4:

50597D: 36 FFreeVar

505988: 13 ExitProcHresult
```

－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－

注册码总结，共为 8 位，分两部分，左边 5 位，及右边 3 位，在输入时先检验左边的 5 位，如正确则存　注册表中，在运行时，从注册表中取得右边 3 位进行检验。

左边 5 位注册码：

走到 `51c33e` 时会产生 `40208`，反过来就是注册码左边 5 位了，即为：`08021`

右边 3 位注册码：

走到 `50591C` 处会得到。

总结：

机器码：`818314132231985`

注册码：`08021716`

注册信息保存在：

`[HKEY_CURRENT_USER\\Software\\VB and VBA Program Settings\\wbreg\\wbregfile]`

`"wbregfilename"="61712080"`

cracked by lordor[BCG]

03.5.21

`4dd466d0e1e0cc123afbe504da2462ca`
