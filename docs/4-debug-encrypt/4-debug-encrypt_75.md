

# Disk Medic v3.0 算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-227.htm`](https://www.pediy.com/kssd/pediy07/pediy7-227.htm)

核心算法 md5  
  
S1 = "6ec4d389b8fb2ca9750c4d89924d8536"  
S2 = "e6828c86c19b461798db8aa8ef3674de"  
S3 = "21cc86789c11d769a127e60448ccdc0b"  
S4 = "5e9e28323bebf08eef264cfc1d2d34f5"  
S5 = "51c9f3f1bd344b8c29e3802a89a0613e"  
S6 = "eca60ae8611369fe28a02e2ab8c5d12e"  
S7 = "539afe13b5657b5d2cea99b64ec01f68"  
  
004F580B     64:8920                mov dword ptr fs:\[eax\],esp  
004F580E     E8 15FEFFFF            call DiskMedi.004F5628  
004F5813     8D55 F0                lea edx,dword ptr ss:\[ebp-10\]  
004F5816     8B45 FC                mov eax,dword ptr ss:\[ebp-4\]  
004F5819     E8 9A160000            call DiskMedi.004F6EB8  
004F581E     8D55 C4                lea edx,dword ptr ss:\[ebp-3C\]  
004F5821     8B45 F0                mov eax,dword ptr ss:\[ebp-10\]  
004F5824     E8 AB35F1FF            call DiskMedi.00408DD4  
004F5829     8B45 C4                mov eax,dword ptr ss:\[ebp-3C\]  
004F582C     E8 5FF0F0FF            call DiskMedi.00404890  
004F5831     83F8 06                cmp eax,6                         ;email 长度  
004F5834     7D 0A                  jge short DiskMedi.004F5840  
004F5836     E8 A9E8F0FF            call DiskMedi.004040E4  
004F583B     E9 AB020000            jmp DiskMedi.004F5AEB  
004F5840     8D55 C8                lea edx,dword ptr ss:\[ebp-38\]  
004F5843     8B45 F8                mov eax,dword ptr ss:\[ebp-8\]      ;sn  
004F5846     E8 7533F1FF            call DiskMedi.00408BC0  
004F584B     8D45 B0                lea eax,dword ptr ss:\[ebp-50\]  
004F584E     8B0D E82F5300          mov ecx,dword ptr ds:\[532FE8\]     ;s1  
004F5854     8B55 F0                mov edx,dword ptr ss:\[ebp-10\]     ;emailname  
004F5857     E8 80F0F0FF            call DiskMedi.004048DC            ;s=emailname+s1  
004F585C     8B45 B0                mov eax,dword ptr ss:\[ebp-50\]  
004F585F     8D55 B4                lea edx,dword ptr ss:\[ebp-4C\]  
004F5862     E8 69FCFFFF            call DiskMedi.004F54D0  
004F5867     8D45 B4                lea eax,dword ptr ss:\[ebp-4C\]  
004F586A     8D55 EC                lea edx,dword ptr ss:\[ebp-14\]  
004F586D     E8 D2FCFFFF            call DiskMedi.004F5544            ;md5str1=md5(s)  
004F5872     8D45 AC                lea eax,dword ptr ss:\[ebp-54\]  
004F5875     8B0D 0C2E5300          mov ecx,dword ptr ds:\[532E0C\]     ;s2  
004F587B     8B55 EC                mov edx,dword ptr ss:\[ebp-14\]     ;md5str1  
004F587E     E8 59F0F0FF            call DiskMedi.004048DC            ;s=md5str1+s2  
004F5883     8B45 AC                mov eax,dword ptr ss:\[ebp-54\]  
004F5886     8D55 B4                lea edx,dword ptr ss:\[ebp-4C\]  
004F5889     E8 42FCFFFF            call DiskMedi.004F54D0  
004F588E     8D45 B4                lea eax,dword ptr ss:\[ebp-4C\]  
004F5891     8D55 E8                lea edx,dword ptr ss:\[ebp-18\]  
004F5894     E8 ABFCFFFF            call DiskMedi.004F5544            ;md5str2=md5(s)  
004F5899     8D45 A8                lea eax,dword ptr ss:\[ebp-58\]  
004F589C     8B0D D8305300          mov ecx,dword ptr ds:\[5330D8\]     ;s3  
004F58A2     8B55 E8                mov edx,dword ptr ss:\[ebp-18\]     ;md5str2  
004F58A5     E8 32F0F0FF            call DiskMedi.004048DC            ;s=md5str2+s3  
004F58AA     8B45 A8                mov eax,dword ptr ss:\[ebp-58\]  
004F58AD     8D55 B4                lea edx,dword ptr ss:\[ebp-4C\]  
004F58B0     E8 1BFCFFFF            call DiskMedi.004F54D0  
004F58B5     8D45 B4                lea eax,dword ptr ss:\[ebp-4C\]  
004F58B8     8D55 E4                lea edx,dword ptr ss:\[ebp-1C\]  
004F58BB     E8 84FCFFFF            call DiskMedi.004F5544            ;md5str3=md5(s)  
004F58C0     8D45 A4                lea eax,dword ptr ss:\[ebp-5C\]  
004F58C3     8B0D 2C2E5300          mov ecx,dword ptr ds:\[532E2C\]     ;s4  
004F58C9     8B55 E4                mov edx,dword ptr ss:\[ebp-1C\]     ;md5str3  
004F58CC     E8 0BF0F0FF            call DiskMedi.004048DC            ;s=md5str3+s4  
004F58D1     8B45 A4                mov eax,dword ptr ss:\[ebp-5C\]  
004F58D4     8D55 B4                lea edx,dword ptr ss:\[ebp-4C\]  
004F58D7     E8 F4FBFFFF            call DiskMedi.004F54D0  
004F58DC     8D45 B4                lea eax,dword ptr ss:\[ebp-4C\]  
004F58DF     8D55 E0                lea edx,dword ptr ss:\[ebp-20\]  
004F58E2     E8 5DFCFFFF            call DiskMedi.004F5544            ;md5str4=md5(s)  
004F58E7     8D45 A0                lea eax,dword ptr ss:\[ebp-60\]  
004F58EA     8B0D 242E5300          mov ecx,dword ptr ds:\[532E24\]     ;s5  
004F58F0     8B55 E0                mov edx,dword ptr ss:\[ebp-20\]     ;md5str4  
004F58F3     E8 E4EFF0FF            call DiskMedi.004048DC            ;s=md5str4+s5  
004F58F8     8B45 A0                mov eax,dword ptr ss:\[ebp-60\]  
004F58FB     8D55 B4                lea edx,dword ptr ss:\[ebp-4C\]  
004F58FE     E8 CDFBFFFF            call DiskMedi.004F54D0  
004F5903     8D45 B4                lea eax,dword ptr ss:\[ebp-4C\]  
004F5906     8D55 DC                lea edx,dword ptr ss:\[ebp-24\]  
004F5909     E8 36FCFFFF            call DiskMedi.004F5544            ;md5str5=md5(s)  
004F590E     8D4D 9C                lea ecx,dword ptr ss:\[ebp-64\]  
004F5911     BA 01000000            mov edx,1  
004F5916     8B45 EC                mov eax,dword ptr ss:\[ebp-14\]     ;md5str1  
004F5919     E8 5AFDFFFF            call DiskMedi.004F5678            ;subsn1=mid(md5str1,1,1)  md5str1 的第一个字符  
004F591E     8B55 9C                mov edx,dword ptr ss:\[ebp-64\]  
004F5921     8D45 EC                lea eax,dword ptr ss:\[ebp-14\]       
004F5924     E8 47EDF0FF            call DiskMedi.00404670  
004F5929     8D4D 98                lea ecx,dword ptr ss:\[ebp-68\]  
004F592C     BA 01000000            mov edx,1  
004F5931     8B45 E8                mov eax,dword ptr ss:\[ebp-18\]     ;md5str2  
004F5934     E8 3FFDFFFF            call DiskMedi.004F5678            ;subsn2=mid(md5str2,1,1)  md5str2 的第一个字符  
004F5939     8B55 98                mov edx,dword ptr ss:\[ebp-68\]  
004F593C     8D45 E8                lea eax,dword ptr ss:\[ebp-18\]  
004F593F     E8 2CEDF0FF            call DiskMedi.00404670  
004F5944     8D4D 94                lea ecx,dword ptr ss:\[ebp-6C\]  
004F5947     BA 01000000            mov edx,1  
004F594C     8B45 E4                mov eax,dword ptr ss:\[ebp-1C\]     ;md5str3  
004F594F     E8 24FDFFFF            call DiskMedi.004F5678            ;subsn3=mid(md5str3,1,1)  md5str3 的第一个字符  
004F5954     8B55 94                mov edx,dword ptr ss:\[ebp-6C\]  
004F5957     8D45 E4                lea eax,dword ptr ss:\[ebp-1C\]  
004F595A     E8 11EDF0FF            call DiskMedi.00404670  
004F595F     8D4D 90                lea ecx,dword ptr ss:\[ebp-70\]  
004F5962     BA 01000000            mov edx,1  
004F5967     8B45 E0                mov eax,dword ptr ss:\[ebp-20\]     ;md5str4  
004F596A     E8 09FDFFFF            call DiskMedi.004F5678            ;subsn4=mid(md5str4,1,1)  md5str4 的第一个字符  
004F596F     8B55 90                mov edx,dword ptr ss:\[ebp-70\]  
004F5972     8D45 E0                lea eax,dword ptr ss:\[ebp-20\]  
004F5975     E8 F6ECF0FF            call DiskMedi.00404670  
004F597A     8D4D 8C                lea ecx,dword ptr ss:\[ebp-74\]  
004F597D     BA 01000000            mov edx,1  
004F5982     8B45 DC                mov eax,dword ptr ss:\[ebp-24\]     ;md5str5  
004F5985     E8 EEFCFFFF            call DiskMedi.004F5678            ;subsn5=mid(md5str5,1,1)  md5str5 的第一个字符  
004F598A     8B55 8C                mov edx,dword ptr ss:\[ebp-74\]  
004F598D     8D45 DC                lea eax,dword ptr ss:\[ebp-24\]  
004F5990     E8 DBECF0FF            call DiskMedi.00404670  
004F5995     FF75 EC                push dword ptr ss:\[ebp-14\]  
004F5998     FF75 E8                push dword ptr ss:\[ebp-18\]  
004F599B     FF75 E4                push dword ptr ss:\[ebp-1C\]  
004F599E     FF75 E0                push dword ptr ss:\[ebp-20\]  
004F59A1     FF75 DC                push dword ptr ss:\[ebp-24\]  
004F59A4     8D45 D8                lea eax,dword ptr ss:\[ebp-28\]  
004F59A7     BA 05000000            mov edx,5  
004F59AC     E8 9FEFF0FF            call DiskMedi.00404950  
004F59B1     8D45 88                lea eax,dword ptr ss:\[ebp-78\]  
004F59B4     8B0D E02E5300          mov ecx,dword ptr ds:\[532EE0\]     ;s6  
004F59BA     8B55 F0                mov edx,dword ptr ss:\[ebp-10\]     ;md5str5  
004F59BD     E8 1AEFF0FF            call DiskMedi.004048DC            ;s=md5str5+s6  
004F59C2     8B45 88                mov eax,dword ptr ss:\[ebp-78\]  
004F59C5     8D55 B4                lea edx,dword ptr ss:\[ebp-4C\]  
004F59C8     E8 03FBFFFF            call DiskMedi.004F54D0  
004F59CD     8D45 B4                lea eax,dword ptr ss:\[ebp-4C\]  
004F59D0     8D55 EC                lea edx,dword ptr ss:\[ebp-14\]  
004F59D3     E8 6CFBFFFF            call DiskMedi.004F5544            ;md5str6=md5(s)  
004F59D8     8D45 84                lea eax,dword ptr ss:\[ebp-7C\]  
004F59DB     8B0D 88305300          mov ecx,dword ptr ds:\[533088\]     ;s7  
004F59E1     8B55 EC                mov edx,dword ptr ss:\[ebp-14\]     ;md5str6  
004F59E4     E8 F3EEF0FF            call DiskMedi.004048DC            ;s=md5str6+s7  
004F59E9     8B45 84                mov eax,dword ptr ss:\[ebp-7C\]  
004F59EC     8D55 B4                lea edx,dword ptr ss:\[ebp-4C\]  
004F59EF     E8 DCFAFFFF            call DiskMedi.004F54D0  
004F59F4     8D45 B4                lea eax,dword ptr ss:\[ebp-4C\]  
004F59F7     8D55 E8                lea edx,dword ptr ss:\[ebp-18\]  
004F59FA     E8 45FBFFFF            call DiskMedi.004F5544            ;md5str7=md5(s)  
004F59FF     8D4D 80                lea ecx,dword ptr ss:\[ebp-80\]  
004F5A02     BA 01000000            mov edx,1  
004F5A07     8B45 EC                mov eax,dword ptr ss:\[ebp-14\]     ;md5str6  
004F5A0A     E8 69FCFFFF            call DiskMedi.004F5678            ;subsn6=mid(md5str6,1,1)  md5str6 的第一个字符  
004F5A0F     8B55 80                mov edx,dword ptr ss:\[ebp-80\]  
004F5A12     8D45 EC                lea eax,dword ptr ss:\[ebp-14\]  
004F5A15     E8 56ECF0FF            call DiskMedi.00404670  
004F5A1A     8D8D 7CFFFFFF          lea ecx,dword ptr ss:\[ebp-84\]  
004F5A20     BA 01000000            mov edx,1  
004F5A25     8B45 E8                mov eax,dword ptr ss:\[ebp-18\]     ;md5str7  
004F5A28     E8 4BFCFFFF            call DiskMedi.004F5678            ;subsn7=mid(md5str7,1,1)  md5str7 的第一个字符  
004F5A2D     8B95 7CFFFFFF          mov edx,dword ptr ss:\[ebp-84\]  
004F5A33     8D45 E8                lea eax,dword ptr ss:\[ebp-18\]  
004F5A36     E8 35ECF0FF            call DiskMedi.00404670  
004F5A3B     FF75 D8                push dword ptr ss:\[ebp-28\]  
004F5A3E     FF75 EC                push dword ptr ss:\[ebp-14\]  
004F5A41     FF75 E8                push dword ptr ss:\[ebp-18\]  
004F5A44     8D45 D8                lea eax,dword ptr ss:\[ebp-28\]  
004F5A47     BA 03000000            mov edx,3  
004F5A4C     E8 FFEEF0FF            call DiskMedi.00404950  
004F5A51     8D4D CC                lea ecx,dword ptr ss:\[ebp-34\]  
004F5A54     BA 07000000            mov edx,7  
004F5A59     8B45 C8                mov eax,dword ptr ss:\[ebp-38\]  
004F5A5C     E8 17FCFFFF            call DiskMedi.004F5678  
004F5A61     8D4D D4                lea ecx,dword ptr ss:\[ebp-2C\]  
004F5A64     BA 03000000            mov edx,3  
004F5A69     8B45 C8                mov eax,dword ptr ss:\[ebp-38\]  
004F5A6C     E8 5FFCFFFF            call DiskMedi.004F56D0  
004F5A71     8D8D 74FFFFFF          lea ecx,dword ptr ss:\[ebp-8C\]  
004F5A77     BA 1E000000            mov edx,1E  
004F5A7C     8B45 C8                mov eax,dword ptr ss:\[ebp-38\]  
004F5A7F     E8 F4FBFFFF            call DiskMedi.004F5678  
004F5A84     8B85 74FFFFFF          mov eax,dword ptr ss:\[ebp-8C\]  
004F5A8A     8D55 B4                lea edx,dword ptr ss:\[ebp-4C\]  
004F5A8D     E8 3EFAFFFF            call DiskMedi.004F54D0  
004F5A92     8D45 B4                lea eax,dword ptr ss:\[ebp-4C\]  
004F5A95     8D95 78FFFFFF          lea edx,dword ptr ss:\[ebp-88\]  
004F5A9B     E8 A4FAFFFF            call DiskMedi.004F5544              ;md5strchk=md5(mid(sn,1,len(sn)-3)))  
004F5AA0     8B85 78FFFFFF          mov eax,dword ptr ss:\[ebp-88\]  
004F5AA6     8D4D D0                lea ecx,dword ptr ss:\[ebp-30\]       ;left(md5strchk,3)  
004F5AA9     BA 03000000            mov edx,3  
004F5AAE     E8 C5FBFFFF            call DiskMedi.004F5678  
004F5AB3     8B45 D8                mov eax,dword ptr ss:\[ebp-28\]       ;判断 left(sn,7)=subsn1+subsn2+subsn3+subsn4+subsn5+subsn6+subsn7  
004F5AB6     8B55 CC                mov edx,dword ptr ss:\[ebp-34\]  
004F5AB9     E8 16EFF0FF            call DiskMedi.004049D4  
004F5ABE     75 11                  jnz short DiskMedi.004F5AD1  
004F5AC0     8B45 D4                mov eax,dword ptr ss:\[ebp-2C\]       ;判断 right(sn,3)=left(md5strchk,3)  
004F5AC3     8B55 D0                mov edx,dword ptr ss:\[ebp-30\]        
004F5AC6     E8 09EFF0FF            call DiskMedi.004049D4  
004F5ACB     75 04                  jnz short DiskMedi.004F5AD1  
004F5ACD     C645 F7 01             mov byte ptr ss:\[ebp-9\],1  
004F5AD1     33C0                   xor eax,eax  
004F5AD3     5A                     pop edx  
004F5AD4     59                     pop ecx  
004F5AD5     59                     pop ecx  
004F5AD6     64:8910                mov dword ptr fs:\[eax\],edx  
004F5AD9     68 EB5A4F00            push DiskMedi.004F5AEB  
004F5ADE     E8 4DFBFFFF            call DiskMedi.004F5630  
004F5AE3     C3                     retn 

 4dd466d0e1e0cc123afbe504da2462ca