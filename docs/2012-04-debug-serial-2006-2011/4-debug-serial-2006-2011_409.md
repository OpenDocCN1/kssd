
# Error Repair Pro 3.8.9 简单分析

> 原文：[`www.pediy.com/kssd/pediy10/78220.html`](https://www.pediy.com/kssd/pediy10/78220.html)

**【破文标题】** Error Repair Pro 3.8.9 简单分析  
**【破文作者】** `tianxj`  
**【作者邮箱】** `tianxj_2007@126.com`  
**【作者主页】** `WwW.ChiNaPYG.CoM`  
**【破解工具】** `PEiD`, `OD`  
**【破解平台】** Windows XP  
**【软件名称】** Error Repair Pro 3.8.9  
**【软件大小】** 1297KB  
**【软件类别】** 国外软件/系统设置  
**【软件授权】** 共享版  
**【软件语言】** 英文  
**【运行环境】** `Win9x/Me/NT/2000/XP/2003`  
**【更新时间】** 2008-12-2  
**【原版下载】** 华军软件园  
**【保护方式】** 注册码  
**【软件简介】** 一款 Windows 注册表修复工具，可以检查注册表的错误并且进行修复。  
**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 【破解内容】

---

### 一、运行程序，进行注册，输入错误的注册信息进行检测，有提示信息

> "The Information you have entered is invalid. Please check the information you have entered and try again. If it still doesnot work then please contact our customer service."

---

### 二、用 `PEiD` 对 `ErrorRepairProfessional.exe` 查壳

为 `ASProtect 1.2x - 1.3x [Registered] -> Alexey Solodovnikov`

---

### 三、带壳调试

运行 `OD`，打开 `ErrorRepairProfessional.exe`，输入注册信息后，下万能断点

---

```
代码:
```



```asm
77D33566      F3:A5             rep movs dword ptr es:[edi],dword p>;  //在这里断下,alt+F9 返回后,用 F8 一路狂奔
==============================================================
005153B8    /$  55             push ebp
005153B9    |.  8BEC          mov ebp,esp
005153BB    |.  B9 C5000000   mov ecx,0C5
005153C0    |>  6A 00         /push 0
005153C2    |.  6A 00         |push 0
005153C4    |.  49            |dec ecx
005153C5    |.^ 75 F9         \jnz short ErrorRep.005153C0
005153C7    |.  51            push ecx
005153C8    |.  53            push ebx
005153C9    |.  8955 F4       mov dword ptr ss:[ebp-C],edx
005153CC    |.  8945 F8       mov dword ptr ss:[ebp-8],eax
005153CF    |.  33C0          xor eax,eax
005153D1    |.  55            push ebp
005153D2    |.  68 9A805100   push ErrorRep.0051809A
005153D7    |.  64:FF30       push dword ptr fs:[eax]
005153DA    |.  64:8920       mov dword ptr fs:[eax],esp
005153DD    |.  8D95 98FDFFFF lea edx,dword ptr ss:[ebp-268]
005153E3    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
005153E8    |.  8B00          mov eax,dword ptr ds:[eax]
005153EA    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
005153F0    |.  E8 B32AF3FF   call ErrorRep.00447EA8
005153F5    |.  8B85 98FDFFFF mov eax,dword ptr ss:[ebp-268]       ;  //来到到这里,试炼码
005153FB    |.  8D95 9CFDFFFF lea edx,dword ptr ss:[ebp-264]
00515401    |.  E8 AEFDFFFF   call ErrorRep.005151B4               ;  //Base64 加密运算
00515406    |.  8B85 9CFDFFFF mov eax,dword ptr ss:[ebp-264]       ;  //试炼码 Base64 加密字符串
0051540C    |.  BA B4805100   mov edx,ErrorRep.005180B4            ;  ASCII "M0syRVBIWlhSQlVXS0IyWTdVWEM4WkI3"
00515411    |.  E8 82F8EEFF   call ErrorRep.00404C98               ;  //比较 CALL
00515416    |.  75 06         jnz short ErrorRep.0051541E          ;  //不等则跳
00515418    |.  C645 F3 01    mov byte ptr ss:[ebp-D],1
0051541C    |.  EB 04         jmp short ErrorRep.00515422
0051541E    |>  C645 F3 00    mov byte ptr ss:[ebp-D],0
00515422    |>  8D95 90FDFFFF lea edx,dword ptr ss:[ebp-270]
00515428    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
0051542D    |.  8B00          mov eax,dword ptr ds:[eax]
0051542F    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
00515435    |.  E8 6E2AF3FF   call ErrorRep.00447EA8
0051543A    |.  8B85 90FDFFFF mov eax,dword ptr ss:[ebp-270]       ;  //试炼码
00515440    |.  8D95 94FDFFFF lea edx,dword ptr ss:[ebp-26C]
00515446    |.  E8 69FDFFFF   call ErrorRep.005151B4               ;  //Base64 加密运算
0051544B    |.  8B85 94FDFFFF mov eax,dword ptr ss:[ebp-26C]       ;  //试炼码 Base64 加密字符串
00515451    |.  BA E0805100   mov edx,ErrorRep.005180E0            ;  ASCII "RVRLNkpOQlRGSzZES0JBRFRQRVRWUUsz"
00515456    |.  E8 3DF8EEFF   call ErrorRep.00404C98               ;  //比较 CALL
0051545B    |.  75 06         jnz short ErrorRep.00515463          ;  //不等则跳
0051545D    |.  C645 F2 01    mov byte ptr ss:[ebp-E],1
00515461    |.  EB 04         jmp short ErrorRep.00515467
00515463    |>  C645 F2 00    mov byte ptr ss:[ebp-E],0
00515467    |>  8D95 88FDFFFF lea edx,dword ptr ss:[ebp-278]
0051546D    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
00515472    |.  8B00          mov eax,dword ptr ds:[eax]
00515474    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
0051547A    |.  E8 292AF3FF   call ErrorRep.00447EA8
0051547F    |.  8B85 88FDFFFF mov eax,dword ptr ss:[ebp-278]       ;  //试炼码
00515485    |.  8D95 8CFDFFFF lea edx,dword ptr ss:[ebp-274]
0051548B    |.  E8 24FDFFFF   call ErrorRep.005151B4               ;  //试炼码 Base64 加密字符串
00515490    |.  8B85 8CFDFFFF mov eax,dword ptr ss:[ebp-274]       ;  //试炼码 Base64 加密字符串
00515496    |.  BA 0C815100   mov edx,ErrorRep.0051810C            ;  ASCII "WE1RQThLTkRUQVBXWFZRNEhVRTZSNEIz"
0051549B    |.  E8 F8F7EEFF   call ErrorRep.00404C98               ;  //比较 CALL
005154A0    |.  75 06         jnz short ErrorRep.005154A8          ;  //不等则跳
005154A2    |.  C645 F1 01    mov byte ptr ss:[ebp-F],1
005154A6    |.  EB 04         jmp short ErrorRep.005154AC
005154A8    |>  C645 F1 00    mov byte ptr ss:[ebp-F],0
005154AC    |>  8D95 80FDFFFF lea edx,dword ptr ss:[ebp-280]
005154B2    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
005154B7    |.  8B00          mov eax,dword ptr ds:[eax]
005154B9    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
005154BF    |.  E8 E429F3FF   call ErrorRep.00447EA8
005154C4    |.  8B85 80FDFFFF mov eax,dword ptr ss:[ebp-280]
005154CA    |.  8D95 84FDFFFF lea edx,dword ptr ss:[ebp-27C]
005154D0    |.  E8 DFFCFFFF   call ErrorRep.005151B4
005154D5    |.  8B85 84FDFFFF mov eax,dword ptr ss:[ebp-27C]
005154DB    |.  BA 38815100   mov edx,ErrorRep.00518138            ;  ASCII "WllMTllHRVVGUVJOWENMQlk2UUZUSFhC"
005154E0    |.  E8 B3F7EEFF   call ErrorRep.00404C98
005154E5    |.  75 06         jnz short ErrorRep.005154ED
005154E7    |.  C645 F0 01    mov byte ptr ss:[ebp-10],1
005154EB    |.  EB 04         jmp short ErrorRep.005154F1
005154ED    |>  C645 F0 00    mov byte ptr ss:[ebp-10],0
005154F1    |>  8D95 78FDFFFF lea edx,dword ptr ss:[ebp-288]
005154F7    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
005154FC    |.  8B00          mov eax,dword ptr ds:[eax]
005154FE    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
00515504    |.  E8 9F29F3FF   call ErrorRep.00447EA8
00515509    |.  8B85 78FDFFFF mov eax,dword ptr ss:[ebp-288]
0051550F    |.  8D95 7CFDFFFF lea edx,dword ptr ss:[ebp-284]
00515515    |.  E8 9AFCFFFF   call ErrorRep.005151B4
0051551A    |.  8B85 7CFDFFFF mov eax,dword ptr ss:[ebp-284]
00515520    |.  BA 64815100   mov edx,ErrorRep.00518164            ;  ASCII "TlJUQUhSWUQyVjhSWVhVR01ROE5QWkxE"
00515525    |.  E8 6EF7EEFF   call ErrorRep.00404C98
0051552A    |.  75 06         jnz short ErrorRep.00515532
0051552C    |.  C645 EF 01    mov byte ptr ss:[ebp-11],1
00515530    |.  EB 04         jmp short ErrorRep.00515536
00515532    |>  C645 EF 00    mov byte ptr ss:[ebp-11],0
00515536    |>  8D95 70FDFFFF lea edx,dword ptr ss:[ebp-290]
0051553C    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
00515541    |.  8B00          mov eax,dword ptr ds:[eax]
00515543    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
00515549    |.  E8 5A29F3FF   call ErrorRep.00447EA8
0051554E    |.  8B85 70FDFFFF mov eax,dword ptr ss:[ebp-290]
00515554    |.  8D95 74FDFFFF lea edx,dword ptr ss:[ebp-28C]
0051555A    |.  E8 55FCFFFF   call ErrorRep.005151B4
0051555F    |.  8B85 74FDFFFF mov eax,dword ptr ss:[ebp-28C]
00515565    |.  BA 90815100   mov edx,ErrorRep.00518190            ;  ASCII "UFlEN1hDVlJBR0xNWEVSV1FGVVJNQ1VF"
0051556A    |.  E8 29F7EEFF   call ErrorRep.00404C98
0051556F    |.  75 06         jnz short ErrorRep.00515577
00515571    |.  C645 EE 01    mov byte ptr ss:[ebp-12],1
00515575    |.  EB 04         jmp short ErrorRep.0051557B
00515577    |>  C645 EE 00    mov byte ptr ss:[ebp-12],0
0051557B    |>  8D95 68FDFFFF lea edx,dword ptr ss:[ebp-298]
00515581    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
00515586    |.  8B00          mov eax,dword ptr ds:[eax]
00515588    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
0051558E    |.  E8 1529F3FF   call ErrorRep.00447EA8
00515593    |.  8B85 68FDFFFF mov eax,dword ptr ss:[ebp-298]
00515599    |.  8D95 6CFDFFFF lea edx,dword ptr ss:[ebp-294]
0051559F    |.  E8 10FCFFFF   call ErrorRep.005151B4
005155A4    |.  8B85 6CFDFFFF mov eax,dword ptr ss:[ebp-294]
005155AA    |.  BA BC815100   mov edx,ErrorRep.005181BC            ;  ASCII "UERCOVdBUVVXNExNTksyRDJONjNVSlBB"
005155AF    |.  E8 E4F6EEFF   call ErrorRep.00404C98
005155B4    |.  75 06         jnz short ErrorRep.005155BC
005155B6    |.  C645 ED 01    mov byte ptr ss:[ebp-13],1
005155BA    |.  EB 04         jmp short ErrorRep.005155C0
005155BC    |>  C645 ED 00    mov byte ptr ss:[ebp-13],0
005155C0    |>  8D95 60FDFFFF lea edx,dword ptr ss:[ebp-2A0]
005155C6    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
005155CB    |.  8B00          mov eax,dword ptr ds:[eax]
005155CD    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
005155D3    |.  E8 D028F3FF   call ErrorRep.00447EA8
005155D8    |.  8B85 60FDFFFF mov eax,dword ptr ss:[ebp-2A0]
005155DE    |.  8D95 64FDFFFF lea edx,dword ptr ss:[ebp-29C]
005155E4    |.  E8 CBFBFFFF   call ErrorRep.005151B4
005155E9    |.  8B85 64FDFFFF mov eax,dword ptr ss:[ebp-29C]
005155EF    |.  BA E8815100   mov edx,ErrorRep.005181E8            ;  ASCII "NzdBUE1WRFU5R0xCRFQ3Q1I2M0NDVVg5"
005155F4    |.  E8 9FF6EEFF   call ErrorRep.00404C98
005155F9    |.  75 06         jnz short ErrorRep.00515601
005155FB    |.  C645 EC 01    mov byte ptr ss:[ebp-14],1
005155FF    |.  EB 04         jmp short ErrorRep.00515605
00515601    |>  C645 EC 00    mov byte ptr ss:[ebp-14],0
00515605    |>  8D95 58FDFFFF lea edx,dword ptr ss:[ebp-2A8]
0051560B    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
00515610    |.  8B00          mov eax,dword ptr ds:[eax]
00515612    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
00515618    |.  E8 8B28F3FF   call ErrorRep.00447EA8
0051561D    |.  8B85 58FDFFFF mov eax,dword ptr ss:[ebp-2A8]
00515623    |.  8D95 5CFDFFFF lea edx,dword ptr ss:[ebp-2A4]
00515629    |.  E8 86FBFFFF   call ErrorRep.005151B4
0051562E    |.  8B85 5CFDFFFF mov eax,dword ptr ss:[ebp-2A4]
00515634    |.  BA 14825100   mov edx,ErrorRep.00518214            ;  ASCII "RDZZQTNSRDY5V1RUMzNNS1FaNlJLN0VO"
00515639    |.  E8 5AF6EEFF   call ErrorRep.00404C98
0051563E    |.  75 06         jnz short ErrorRep.00515646
00515640    |.  C645 EB 01    mov byte ptr ss:[ebp-15],1
00515644    |.  EB 04         jmp short ErrorRep.0051564A
00515646    |>  C645 EB 00    mov byte ptr ss:[ebp-15],0
0051564A    |>  8D95 50FDFFFF lea edx,dword ptr ss:[ebp-2B0]
00515650    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
00515655    |.  8B00          mov eax,dword ptr ds:[eax]
00515657    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
0051565D    |.  E8 4628F3FF   call ErrorRep.00447EA8
00515662    |.  8B85 50FDFFFF mov eax,dword ptr ss:[ebp-2B0]
00515668    |.  8D95 54FDFFFF lea edx,dword ptr ss:[ebp-2AC]
0051566E    |.  E8 41FBFFFF   call ErrorRep.005151B4
00515673    |.  8B85 54FDFFFF mov eax,dword ptr ss:[ebp-2AC]
00515679    |.  BA 40825100   mov edx,ErrorRep.00518240            ;  ASCII "S1dUNzIzQVdIR0w4RFJRNEEzVlBBV1VR"
0051567E    |.  E8 15F6EEFF   call ErrorRep.00404C98
00515683    |.  75 06         jnz short ErrorRep.0051568B
00515685    |.  C645 EA 01    mov byte ptr ss:[ebp-16],1
00515689    |.  EB 04         jmp short ErrorRep.0051568F
0051568B    |>  C645 EA 00    mov byte ptr ss:[ebp-16],0
0051568F    |>  8D95 48FDFFFF lea edx,dword ptr ss:[ebp-2B8]
00515695    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
0051569A    |.  8B00          mov eax,dword ptr ds:[eax]
0051569C    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
005156A2    |.  E8 0128F3FF   call ErrorRep.00447EA8
005156A7    |.  8B85 48FDFFFF mov eax,dword ptr ss:[ebp-2B8]
005156AD    |.  8D95 4CFDFFFF lea edx,dword ptr ss:[ebp-2B4]
005156B3    |.  E8 FCFAFFFF   call ErrorRep.005151B4
005156B8    |.  8B85 4CFDFFFF mov eax,dword ptr ss:[ebp-2B4]
005156BE    |.  BA 6C825100   mov edx,ErrorRep.0051826C            ;  ASCII "S1pRVEVFQkRIVzZZTjdHVEY2TEtZVlc0"
005156C3    |.  E8 D0F5EEFF   call ErrorRep.00404C98
005156C8    |.  75 06         jnz short ErrorRep.005156D0
005156CA    |.  C645 E9 01    mov byte ptr ss:[ebp-17],1
005156CE    |.  EB 04         jmp short ErrorRep.005156D4
005156D0    |>  C645 E9 00    mov byte ptr ss:[ebp-17],0
005156D4    |>  8D95 40FDFFFF lea edx,dword ptr ss:[ebp-2C0]
005156DA    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
005156DF    |.  8B00          mov eax,dword ptr ds:[eax]
005156E1    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
005156E7    |.  E8 BC27F3FF   call ErrorRep.00447EA8
005156EC    |.  8B85 40FDFFFF mov eax,dword ptr ss:[ebp-2C0]
005156F2    |.  8D95 44FDFFFF lea edx,dword ptr ss:[ebp-2BC]
005156F8    |.  E8 B7FAFFFF   call ErrorRep.005151B4
005156FD    |.  8B85 44FDFFFF mov eax,dword ptr ss:[ebp-2BC]
00515703    |.  BA 98825100   mov edx,ErrorRep.00518298            ;  ASCII "QUtRQ05IVEdMRVpBUEw3WVBKWkVaMzJa"
00515708    |.  E8 8BF5EEFF   call ErrorRep.00404C98
0051570D    |.  75 06         jnz short ErrorRep.00515715
0051570F    |.  C645 E8 01    mov byte ptr ss:[ebp-18],1
00515713    |.  EB 04         jmp short ErrorRep.00515719
00515715    |>  C645 E8 00    mov byte ptr ss:[ebp-18],0
00515719    |>  8D95 38FDFFFF lea edx,dword ptr ss:[ebp-2C8]
0051571F    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
00515724    |.  8B00          mov eax,dword ptr ds:[eax]
00515726    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
0051572C    |.  E8 7727F3FF   call ErrorRep.00447EA8
00515731    |.  8B85 38FDFFFF mov eax,dword ptr ss:[ebp-2C8]
00515737    |.  8D95 3CFDFFFF lea edx,dword ptr ss:[ebp-2C4]
0051573D    |.  E8 72FAFFFF   call ErrorRep.005151B4
00515742    |.  8B85 3CFDFFFF mov eax,dword ptr ss:[ebp-2C4]
00515748    |.  BA C4825100   mov edx,ErrorRep.005182C4            ;  ASCII "NzNNS0U4QlgzSE1BTEo4UDhNQjI4TUU2"
0051574D    |.  E8 46F5EEFF   call ErrorRep.00404C98
00515752    |.  75 06         jnz short ErrorRep.0051575A
00515754    |.  C645 E7 01    mov byte ptr ss:[ebp-19],1
00515758    |.  EB 04         jmp short ErrorRep.0051575E
0051575A    |>  C645 E7 00    mov byte ptr ss:[ebp-19],0
0051575E    |>  8D95 30FDFFFF lea edx,dword ptr ss:[ebp-2D0]
00515764    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
00515769    |.  8B00          mov eax,dword ptr ds:[eax]
0051576B    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
00515771    |.  E8 3227F3FF   call ErrorRep.00447EA8
00515776    |.  8B85 30FDFFFF mov eax,dword ptr ss:[ebp-2D0]
0051577C    |.  8D95 34FDFFFF lea edx,dword ptr ss:[ebp-2CC]
00515782    |.  E8 2DFAFFFF   call ErrorRep.005151B4
00515787    |.  8B85 34FDFFFF mov eax,dword ptr ss:[ebp-2CC]
0051578D    |.  BA F0825100   mov edx,ErrorRep.005182F0            ;  ASCII "UFk5OVJMSFpYQVI4RE5VNE43NDJIWUJM"
00515792    |.  E8 01F5EEFF   call ErrorRep.00404C98
00515797    |.  75 06         jnz short ErrorRep.0051579F
00515799    |.  C645 E6 01    mov byte ptr ss:[ebp-1A],1
0051579D    |.  EB 04         jmp short ErrorRep.005157A3
0051579F    |>  C645 E6 00    mov byte ptr ss:[ebp-1A],0
005157A3    |>  8D95 28FDFFFF lea edx,dword ptr ss:[ebp-2D8]
005157A9    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
005157AE    |.  8B00          mov eax,dword ptr ds:[eax]
005157B0    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
005157B6    |.  E8 ED26F3FF   call ErrorRep.00447EA8
005157BB    |.  8B85 28FDFFFF mov eax,dword ptr ss:[ebp-2D8]
005157C1    |.  8D95 2CFDFFFF lea edx,dword ptr ss:[ebp-2D4]
005157C7    |.  E8 E8F9FFFF   call ErrorRep.005151B4
005157CC    |.  8B85 2CFDFFFF mov eax,dword ptr ss:[ebp-2D4]
005157D2    |.  BA 1C835100   mov edx,ErrorRep.0051831C            ;  ASCII "RjZWVEVKWERBRlFVTEJWWlBXOFhYOE5U"
005157D7    |.  E8 BCF4EEFF   call ErrorRep.00404C98
005157DC    |.  75 06         jnz short ErrorRep.005157E4
005157DE    |.  C645 E5 01    mov byte ptr ss:[ebp-1B],1
005157E2    |.  EB 04         jmp short ErrorRep.005157E8
005157E4    |>  C645 E5 00    mov byte ptr ss:[ebp-1B],0
005157E8    |>  8D95 20FDFFFF lea edx,dword ptr ss:[ebp-2E0]
005157EE    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
005157F3    |.  8B00          mov eax,dword ptr ds:[eax]
005157F5    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
005157FB    |.  E8 A826F3FF   call ErrorRep.00447EA8
00515800    |.  8B85 20FDFFFF mov eax,dword ptr ss:[ebp-2E0]
00515806    |.  8D95 24FDFFFF lea edx,dword ptr ss:[ebp-2DC]
0051580C    |.  E8 A3F9FFFF   call ErrorRep.005151B4
00515811    |.  8B85 24FDFFFF mov eax,dword ptr ss:[ebp-2DC]
00515817    |.  BA 48835100   mov edx,ErrorRep.00518348            ;  ASCII "M1pWVDJSUDhFV044S05YTDQzUTZCVzI4"
0051581C    |.  E8 77F4EEFF   call ErrorRep.00404C98
00515821    |.  75 06         jnz short ErrorRep.00515829
00515823    |.  C645 E4 01    mov byte ptr ss:[ebp-1C],1
00515827    |.  EB 04         jmp short ErrorRep.0051582D
00515829    |>  C645 E4 00    mov byte ptr ss:[ebp-1C],0
0051582D    |>  8D95 18FDFFFF lea edx,dword ptr ss:[ebp-2E8]
00515833    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
00515838    |.  8B00          mov eax,dword ptr ds:[eax]
0051583A    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
00515840    |.  E8 6326F3FF   call ErrorRep.00447EA8
00515845    |.  8B85 18FDFFFF mov eax,dword ptr ss:[ebp-2E8]
0051584B    |.  8D95 1CFDFFFF lea edx,dword ptr ss:[ebp-2E4]
00515851    |.  E8 5EF9FFFF   call ErrorRep.005151B4
00515856    |.  8B85 1CFDFFFF mov eax,dword ptr ss:[ebp-2E4]
0051585C    |.  BA 74835100   mov edx,ErrorRep.00518374            ;  ASCII "VktIRUVYTlVKTjRNMjdNQjdBUFo2WTlC"
00515861    |.  E8 32F4EEFF   call ErrorRep.00404C98
00515866    |.  75 06         jnz short ErrorRep.0051586E
00515868    |.  C645 E3 01    mov byte ptr ss:[ebp-1D],1
0051586C    |.  EB 04         jmp short ErrorRep.00515872
0051586E    |>  C645 E3 00    mov byte ptr ss:[ebp-1D],0
00515872    |>  8D95 10FDFFFF lea edx,dword ptr ss:[ebp-2F0]
00515878    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
0051587D    |.  8B00          mov eax,dword ptr ds:[eax]
0051587F    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
00515885    |.  E8 1E26F3FF   call ErrorRep.00447EA8
0051588A    |.  8B85 10FDFFFF mov eax,dword ptr ss:[ebp-2F0]
00515890    |.  8D95 14FDFFFF lea edx,dword ptr ss:[ebp-2EC]
00515896    |.  E8 19F9FFFF   call ErrorRep.005151B4
0051589B    |.  8B85 14FDFFFF mov eax,dword ptr ss:[ebp-2EC]
005158A1    |.  BA A0835100   mov edx,ErrorRep.005183A0            ;  ASCII "NlVIOFpYQVVVMzdORzNXSlRBTDkyMzNR"
005158A6    |.  E8 EDF3EEFF   call ErrorRep.00404C98
005158AB    |.  75 06         jnz short ErrorRep.005158B3
005158AD    |.  C645 E2 01    mov byte ptr ss:[ebp-1E],1
005158B1    |.  EB 04         jmp short ErrorRep.005158B7
005158B3    |>  C645 E2 00    mov byte ptr ss:[ebp-1E],0
005158B7    |>  8D95 08FDFFFF lea edx,dword ptr ss:[ebp-2F8]
005158BD    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
005158C2    |.  8B00          mov eax,dword ptr ds:[eax]
005158C4    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
005158CA    |.  E8 D925F3FF   call ErrorRep.00447EA8
005158CF    |.  8B85 08FDFFFF mov eax,dword ptr ss:[ebp-2F8]
005158D5    |.  8D95 0CFDFFFF lea edx,dword ptr ss:[ebp-2F4]
005158DB    |.  E8 D4F8FFFF   call ErrorRep.005151B4
005158E0    |.  8B85 0CFDFFFF mov eax,dword ptr ss:[ebp-2F4]
005158E6    |.  BA CC835100   mov edx,ErrorRep.005183CC            ;  ASCII "OEpNV0VQWFg4UFEzWUdWMkUyQ0YyOEo3"
005158EB    |.  E8 A8F3EEFF   call ErrorRep.00404C98
005158F0    |.  75 06         jnz short ErrorRep.005158F8
005158F2    |.  C645 E1 01    mov byte ptr ss:[ebp-1F],1
005158F6    |.  EB 04         jmp short ErrorRep.005158FC
005158F8    |>  C645 E1 00    mov byte ptr ss:[ebp-1F],0
005158FC    |>  8D95 00FDFFFF lea edx,dword ptr ss:[ebp-300]
00515902    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
00515907    |.  8B00          mov eax,dword ptr ds:[eax]
00515909    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
0051590F    |.  E8 9425F3FF   call ErrorRep.00447EA8
00515914    |.  8B85 00FDFFFF mov eax,dword ptr ss:[ebp-300]
0051591A    |.  8D95 04FDFFFF lea edx,dword ptr ss:[ebp-2FC]
00515920    |.  E8 8FF8FFFF   call ErrorRep.005151B4
00515925    |.  8B85 04FDFFFF mov eax,dword ptr ss:[ebp-2FC]
0051592B    |.  BA F8835100   mov edx,ErrorRep.005183F8            ;  ASCII "UDlSVjZFR1Q3SjNMQllORDJHUEMyUFBG"
00515930    |.  E8 63F3EEFF   call ErrorRep.00404C98
00515935    |.  75 06         jnz short ErrorRep.0051593D
00515937    |.  C645 E0 01    mov byte ptr ss:[ebp-20],1
0051593B    |.  EB 04         jmp short ErrorRep.00515941
0051593D    |>  C645 E0 00    mov byte ptr ss:[ebp-20],0
00515941    |>  8D95 F8FCFFFF lea edx,dword ptr ss:[ebp-308]
00515947    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
0051594C    |.  8B00          mov eax,dword ptr ds:[eax]
0051594E    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
00515954    |.  E8 4F25F3FF   call ErrorRep.00447EA8
00515959    |.  8B85 F8FCFFFF mov eax,dword ptr ss:[ebp-308]
0051595F    |.  8D95 FCFCFFFF lea edx,dword ptr ss:[ebp-304]
00515965    |.  E8 4AF8FFFF   call ErrorRep.005151B4
0051596A    |.  8B85 FCFCFFFF mov eax,dword ptr ss:[ebp-304]
00515970    |.  BA 24845100   mov edx,ErrorRep.00518424            ;  ASCII "NDhEOEpFRFFMMkhVVVoySlkyS1VDSDI3"
00515975    |.  E8 1EF3EEFF   call ErrorRep.00404C98
0051597A    |.  75 06         jnz short ErrorRep.00515982
0051597C    |.  C645 DF 01    mov byte ptr ss:[ebp-21],1
00515980    |.  EB 04         jmp short ErrorRep.00515986
00515982    |>  C645 DF 00    mov byte ptr ss:[ebp-21],0
00515986    |>  8D95 F0FCFFFF lea edx,dword ptr ss:[ebp-310]
0051598C    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
00515991    |.  8B00          mov eax,dword ptr ds:[eax]
00515993    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
00515999    |.  E8 0A25F3FF   call ErrorRep.00447EA8
0051599E    |.  8B85 F0FCFFFF mov eax,dword ptr ss:[ebp-310]
005159A4    |.  8D95 F4FCFFFF lea edx,dword ptr ss:[ebp-30C]
005159AA    |.  E8 05F8FFFF   call ErrorRep.005151B4
005159AF    |.  8B85 F4FCFFFF mov eax,dword ptr ss:[ebp-30C]
005159B5    |.  BA 50845100   mov edx,ErrorRep.00518450            ;  ASCII "UEJQWFZMREw2MjRWTDM0Wkg2UEtGQ0xL"
005159BA    |.  E8 D9F2EEFF   call ErrorRep.00404C98
005159BF    |.  75 06         jnz short ErrorRep.005159C7
005159C1    |.  C645 DE 01    mov byte ptr ss:[ebp-22],1
005159C5    |.  EB 04         jmp short ErrorRep.005159CB
005159C7    |>  C645 DE 00    mov byte ptr ss:[ebp-22],0
005159CB    |>  8D95 E8FCFFFF lea edx,dword ptr ss:[ebp-318]
005159D1    |.  A1 AC325500   mov eax,dword ptr ds:[5532AC]
005159D6    |.  8B00          mov eax,dword ptr ds:[eax]
005159D8    |.  8B80 48030000 mov eax,dword ptr ds:[eax+348]
005159DE    |.  E8 C524F3FF   call ErrorRep.00447EA8
005159E3    |.  8B85 E8FCFFFF mov eax,dword ptr ss:[ebp-318]
005159E9    |.  8D95 ECF...
```

（注：由于原文本极其冗长且为汇编代码，上述代码块保留了核心结构，末尾省略部分已根据原文“下面省略部分代码”进行处理，保持代码块完整性。）

```asm
。下面省略部分代码
```



***

【破解总结】

----------------------------------------------------------------

【算法总结】

采用 `Base64` 加密算法

1. 注册码与用户名无关
2. 注册码的 `Base64` 加密字符串与内置的字符串相同则注册成功

找个密码学工具随便找上面一组字符串解码，解码后的字符串就是一组注册码

----------------------------------------------------------------

【注册信息】

一组可用的注册码

用户名：`tianxj`

注册码：`3K2EPHZXRBUWKB2Y7UXC8ZB7`

保存在注册表`[HKEY_CURRENT_USER\Software\ErrorRepairPro]`和文件`Dataprogs.dat`里

----------------------------------------------------------------

感谢飘云老大、猫老大、`Nisy`老大以及很多前辈们的学习教程以及徐超等所有帮助过我的论坛兄弟姐妹们！谢谢

----------------------------------------------------------------

【版权声明】破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流，转载请注明作者并保持文章的完整，谢谢!
