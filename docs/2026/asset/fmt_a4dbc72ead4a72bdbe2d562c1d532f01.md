
# 汇编片段

```asm
.text:004005F4                 mov     ecx, 222420h
.text:004005F9                 mov     [esp+58h+var_48], ebx
.text:004005FD                 cmp     eax, ecx
.text:004005FF                 ja      loc_400969
```

## 片段1

```asm
.text:0040077C
.text:0040077C loc_40077C:                             ; CODE XREF: myDeviceIoControl+33j
.text:0040077C                 push    dword ptr [esi+8] ; dwIoControlCode:222407h
.text:0040077C                                         ; //这个块主要进行获取函数地址并把该地址传给某变量保存
.text:0040077C                                         ; 变量的名称以及修正为该函数实际的名称
.text:0040077C                                         ; 获取这些函数地址的作用是处理其它功能号时会使用到
.text:0040077C                                         ; (size_t)
.text:0040077F                 push    dword ptr [esi+10h] ; (void *)pIrpStack->Parameters.QueryInterface.InterfaceSpecificData
.text:00400782                 push    offset dword_4028C0 ; void *
.text:00400787                 call    memcpy
.text:0040078C                 mov     ecx, ds:KeServiceDescriptorTable
.text:00400792                 mov     ecx, [ecx]      ; 囧囧ServiceTableBase
.text:00400794                 mov     eax, [esi+10h]
.text:00400797                 mov     [eax+6Ch], ecx
.text:0040079A                 mov     ecx, ds:KeServiceDescriptorTable
.text:004007A0                 mov     ecx, [ecx+8]    ; 嘎嘎，NumberOfService
.text:004007A3                 mov     eax, [esi+10h]
.text:004007A6                 mov     esi, ds:RtlInitUnicodeString
.text:004007AC                 mov     [eax+70h], ecx
.text:004007AF                 mov     eax, dword_4028C0 ; 取第一个函数MmGetSystemRoutineAddress
.text:004007B4                 mov     MmGetSystemRoutineAddress, eax
.text:004007B9                 add     esp, 0Ch
.text:004007BC                 lea     eax, [esp+58h+SourceString]
.text:004007C0                 push    eax             ; SourceString
.text:004007C1                 lea     eax, [esp+5Ch+DestinationString]
.text:004007C5                 push    eax             ; DestinationString
.text:004007C6                 mov     [esp+60h+SourceString.Length], 6F0049h
.text:004007CE                 mov     [esp+60h+SourceString.RootDirectory], 720044h
.text:004007D6                 mov     [esp+60h+SourceString.ObjectName], 760069h
.text:004007DE                 mov     [esp+60h+SourceString.Attributes], 720065h
.text:004007E6                 mov     [esp+60h+SourceString.SecurityDescriptor], 62004Fh
.text:004007EE                 mov     [esp+60h+SourceString.SecurityQualityOfService], 65006Ah
.text:004007F6                 mov     dword ptr [esp+50h], 740063h
.text:004007FE                 mov     dword ptr [esp+54h], 790054h
.text:00400806                 mov     dword ptr [esp+58h], 650070h
.text:0040080E                 mov     [esp+5Ch], ebx  ; 0000000
.text:00400812                 call    esi ; RtlInitUnicodeString ; SourceString = L"IoDriverObjectType"
.text:00400814                 lea     eax, [esp+58h+DestinationString]
.text:00400818                 push    eax             ; PUNICODE_STRING
.text:00400819                 call    MmGetSystemRoutineAddress
.text:0040081F                 push    79h
.text:00400821                 mov     myIoDriverObjectType, eax
.text:00400826                 pop     ebx
.text:00400827                 lea     eax, [esp+58h+SourceString]
.text:0040082B                 push    eax             ; SourceString
.text:0040082C                 lea     eax, [esp+5Ch+DestinationString]
.text:00400830                 push    eax             ; DestinationString
.text:00400831                 mov     [esp+60h+SourceString.Length], 77005Ah
.text:00400839                 mov     [esp+60h+SourceString.RootDirectory], 720043h
.text:00400841                 mov     [esp+60h+SourceString.ObjectName], 610065h
.text:00400849                 mov     [esp+60h+SourceString.Attributes], 650074h
.text:00400851                 mov     [esp+60h+SourceString.SecurityDescriptor], 65004Bh
.text:00400859                 mov     [esp+60h+SourceString.SecurityQualityOfService], ebx ; 00000000
.text:0040085D                 call    esi ; RtlInitUnicodeString ; SourceString = L"ZwCreateMailslotFile"
.text:0040085F                 lea     eax, [esp+58h+DestinationString]
.text:00400863                 push    eax             ; PUNICODE_STRING
.text:00400864                 call    MmGetSystemRoutineAddress
.text:0040086A                 mov     myZwCreateMailslotFile, eax ; ZwCreateMailslotFile
.text:0040086F                 lea     eax, [esp+58h+SourceString]
.text:00400873                 push    eax             ; SourceString
.text:00400874                 lea     eax, [esp+5Ch+DestinationString]
.text:00400878                 push    eax             ; DestinationString
.text:00400879                 mov     [esp+60h+SourceString.Length], 77005Ah ; 这里7个mov指令是赋值一个字符串
.text:00400881                 mov     [esp+60h+SourceString.RootDirectory], 650053h
.text:00400889                 mov     [esp+60h+SourceString.ObjectName], 560074h
.text:00400891                 mov     [esp+60h+SourceString.Attributes], 6C0061h
.text:00400899                 mov     [esp+60h+SourceString.SecurityDescriptor], 650075h
.text:004008A1                 mov     [esp+60h+SourceString.SecurityQualityOfService], 65004Bh
.text:004008A9                 mov     [esp+50h], ebx
.text:004008AD                 call    esi ; RtlInitUnicodeString ; SourceString=L"ZwSetVolumeInformationFile"
.text:004008AF                 lea     eax, [esp+58h+DestinationString]
.text:004008B3                 push    eax             ; PUNICODE_STRING
.text:004008B4                 call    MmGetSystemRoutineAddress
.text:004008BA                 mov     myZwSetVolumeInformationFile, eax ; ZwSetVolumeInformationFile
.text:004008BF                 push    offset aObreferenceobj ; "ObReferenceObjectByName"
.text:004008C4                 lea     eax, [esp+5Ch+DestinationString]
.text:004008C8                 push    eax             ; DestinationString
.text:004008C9                 call    esi ; RtlInitUnicodeString
.text:004008CB                 lea     eax, [esp+58h+DestinationString]
.text:004008CF                 push    eax             ; PUNICODE_STRING
.text:004008D0                 call    MmGetSystemRoutineAddress
.text:004008D6                 mov     myObReferenceObjectByName, eax
.text:004008DB                 push    offset aPslookupproces ; "PsLookupProcessByProcessId"
.text:004008E0                 lea     eax, [esp+5Ch+DestinationString]
.text:004008E4                 push    eax             ; DestinationString
.text:004008E5                 call    esi ; RtlInitUnicodeString
.text:004008E7                 lea     eax, [esp+58h+DestinationString]
.text:004008EB                 push    eax             ; PUNICODE_STRING
.text:004008EC                 call    MmGetSystemRoutineAddress
.text:004008F2                 mov     myPsLookupProcessByProcessId, eax
.text:004008F7                 push    offset aZwopenkey ; "ZwOpenKey"
.text:004008FC                 lea     eax, [esp+5Ch+DestinationString]
.text:00400900                 push    eax             ; DestinationString
.text:00400901                 call    esi ; RtlInitUnicodeString
.text:00400903                 lea     eax, [esp+58h+DestinationString]
.text:00400907                 push    eax             ; PUNICODE_STRING
.text:00400908                 call    MmGetSystemRoutineAddress
.text:0040090E                 mov     myZwOpenKey, eax
.text:00400913                 mov     eax, ds:MmIsAddressValid
.text:00400918                 mov     myMmIsAddressValid, eax
.text:0040091D                 mov     eax, ds:KfLowerIrql
.text:00400922                 mov     myKfLowerIrql, eax
.text:00400927                 mov     eax, ds:_wcsnicmp
.text:0040092C                 mov     mywcsnicmp, eax
```

该片段主要进行获取函数地址并把该地址传给某变量保存，变量的名称以及修正为该函数实际的名称，获取这些函数地址的作用是处理其它功能号时会使用到。
