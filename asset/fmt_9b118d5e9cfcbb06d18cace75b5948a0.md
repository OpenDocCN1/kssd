
```
004F5995      FF75 EC                 push dword ptr ss:[ebp-14]
004F5998      FF75 E8                 push dword ptr ss:[ebp-18]
004F599B      FF75 E4                 push dword ptr ss:[ebp-1C]
004F599E      FF75 E0                 push dword ptr ss:[ebp-20]
004F59A1      FF75 DC                 push dword ptr ss:[ebp-24]
004F59A4      8D45 D8                 lea eax,dword ptr ss:[ebp-28]
004F59A7      BA 05000000             mov edx,5
004F59AC      E8 9FEFF0FF             call DiskMedi.00404950
004F59B1      8D45 88                 lea eax,dword ptr ss:[ebp-78]
004F59B4      8B0D E02E5300           mov ecx,dword ptr ds:[532EE0]      ;s6
004F59BA      8B55 F0                 mov edx,dword ptr ss:[ebp-10]      ;md5str5
004F59BD      E8 1AEFF0FF             call DiskMedi.004048DC             ;s=md5str5+s6
004F59C2      8B45 88                 mov eax,dword ptr ss:[ebp-78]
004F59C5      8D55 B4                 lea edx,dword ptr ss:[ebp-4C]
004F59C8      E8 03FBFFFF             call DiskMedi.004F54D0
004F59CD      8D45 B4                 lea eax,dword ptr ss:[ebp-4C]
004F59D0      8D55 EC                 lea edx,dword ptr ss:[ebp-14]
004F59D3      E8 6CFBFFFF             call DiskMedi.004F5544             ;md5str6=md5(s)
004F59D8      8D45 84                 lea eax,dword ptr ss:[ebp-7C]
004F59DB      8B0D 88305300           mov ecx,dword ptr ds:[533088]      ;s7
004F59E1      8B55 EC                 mov edx,dword ptr ss:[ebp-14]      ;md5str6
004F59E4      E8 F3EEF0FF             call DiskMedi.004048DC             ;s=md5str6+s7
004F59E9      8B45 84                 mov eax,dword ptr ss:[ebp-7C]
004F59EC      8D55 B4                 lea edx,dword ptr ss:[ebp-4C]
004F59EF      E8 DCFAFFFF             call DiskMedi.004F54D0
004F59F4      8D45 B4                 lea eax,dword ptr ss:[ebp-4C]
004F59F7      8D55 E8                 lea edx,dword ptr ss:[ebp-18]
004F59FA      E8 45FBFFFF             call DiskMedi.004F5544             ;md5str7=md5(s)
004F59FF      8D4D 80                 lea ecx,dword ptr ss:[ebp-80]
004F5A02      BA 01000000             mov edx,1
004F5A07      8B45 EC                 mov eax,dword ptr ss:[ebp-14]      ;md5str6
004F5A0A      E8 69FCFFFF             call DiskMedi.004F5678             ;subsn6=mid(md5str6,1,1)  md5str6 的第一个字符
004F5A0F      8B55 80                 mov edx,dword ptr ss:[ebp-80]
004F5A12      8D45 EC                 lea eax,dword ptr ss:[ebp-14]
004F5A15      E8 56ECF0FF             call DiskMedi.00404670
004F5A1A      8D8D 7CFFFFFF           lea ecx,dword ptr ss:[ebp-84]
004F5A20      BA 01000000             mov edx,1
004F5A25      8B45 E8                 mov eax,dword ptr ss:[ebp-18]      ;md5str7
004F5A28      E8 4BFCFFFF             call DiskMedi.004F5678             ;subsn7=mid(md5str7,1,1)  md5str7 的第一个字符
004F5A2D      8B95 7CFFFFFF           mov edx,dword ptr ss:[ebp-84]
004F5A33      8D45 E8                 lea eax,dword ptr ss:[ebp-18]
004F5A36      E8 35ECF0FF             call DiskMedi.00404670
004F5A3B      FF75 D8                 push dword ptr ss:[ebp-28]
004F5A3E      FF75 EC                 push dword ptr ss:[ebp-14]
004F5A41      FF75 E8                 push dword ptr ss:[ebp-18]
004F5A44      8D45 D8                 lea eax,dword ptr ss:[ebp-28]
004F5A47      BA 03000000             mov edx,3
004F5A4C      E8 FFEEF0FF             call DiskMedi.00404950
004F5A51      8D4D CC                 lea ecx,dword ptr ss:[ebp-34]
004F5A54      BA 07000000             mov edx,7
004F5A59      8B45 C8                 mov eax,dword ptr ss:[ebp-38]
004F5A5C      E8 17FCFFFF             call DiskMedi.004F5678
004F5A61      8D4D D4                 lea ecx,dword ptr ss:[ebp-2C]
004F5A64      BA 03000000             mov edx,3
004F5A69      8B45 C8                 mov eax,dword ptr ss:[ebp-38]
004F5A6C      E8 5FFCFFFF             call DiskMedi.004F56D0
004F5A71      8D8D 74FFFFFF           lea ecx,dword ptr ss:[ebp-8C]
004F5A77      BA 1E000000             mov edx,1E
004F5A7C      8B45 C8                 mov eax,dword ptr ss:[ebp-38]
004F5A7F      E8 F4FBFFFF             call DiskMedi.004F5678
004F5A84      8B85 74FFFFFF           mov eax,dword ptr ss:[ebp-8C]
004F5A8A      8D55 B4                 lea edx,dword ptr ss:[ebp-4C]
004F5A8D      E8 3EFAFFFF             call DiskMedi.004F54D0
004F5A92      8D45 B4                 lea eax,dword ptr ss:[ebp-4C]
004F5A95      8D95 78FFFFFF           lea edx,dword ptr ss:[ebp-88]
004F5A9B      E8 A4FAFFFF             call DiskMedi.004F5544               ;md5strchk=md5(mid(sn,1,len(sn)-3)))
004F5AA0      8B85 78FFFFFF           mov eax,dword ptr ss:[ebp-88]
004F5AA6      8D4D D0                 lea ecx,dword ptr ss:[ebp-30]       ;left(md5strchk,3)
004F5AA9      BA 03000000             mov edx,3
004F5AAE      E8 C5FBFFFF             call DiskMedi.004F5678
004F5AB3      8B45 D8                 mov eax,dword ptr ss:[ebp-28]       ;判断 left(sn,7)=subsn1+subsn2+subsn3+subsn4+subsn5+subsn6+subsn7
004F5AB6      8B55 CC                 mov edx,dword ptr ss:[ebp-34]
004F5AB9      E8 16EFF0FF             call DiskMedi.004049D4
004F5ABE      75 11                   jnz short DiskMedi.004F5AD1
004F5AC0      8B45 D4                 mov eax,dword ptr ss:[ebp-2C]       ;判断 right(sn,3)=left(md5strchk,3)
004F5AC3      8B55 D0                 mov edx,dword ptr ss:[ebp-30]
004F5AC6      E8 09EFF0FF             call DiskMedi.004049D4
004F5ACB      75 04                   jnz short DiskMedi.004F5AD1
004F5ACD      C645 F7 01              mov byte ptr ss:[ebp-9],1
004F5AD1      33C0                    xor eax,eax
004F5AD3      5A                      pop edx
004F5AD4      59                      pop ecx
004F5AD5      59                      pop ecx
004F5AD6      64:8910                 mov dword ptr fs:[eax],edx
004F5AD9      68 EB5A4F00             push DiskMedi.004F5AEB
004F5ADE      E8 4DFBFFFF             call DiskMedi.004F5630
004F5AE3      C3                      retn
4dd466d0e1e0cc123afbe504da2462ca
```
