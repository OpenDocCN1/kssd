
5. `C:\WINDOWS\system32\driveset.dll`：保存程序的运行次数（未注册时）以及注册的用户名和注册码。

6. `X:\recycle\S-1-5-21-1060284298-811497611-11778920086-500 \com1.{21ec2020-3aea-1069-a2dd-08002b30309d}\data\panfile2.dll`：保存当前分区中的密盘信息，以及经过加密处理的密盘打开密码。

## 四、程序的运行密码和密盘打开密码

调试程序可以发现程序的运行密码及密盘打开密码都只是简单的字符替换处理，于是设置一个所有字符的密码后就能得到对应的字符替换表。

运行密码替换表为：

```
0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz
89:;<=>?@AIJKLMNOPQRSTUVWXYZ[\]^\_`abijklmnopqrstuvwxyz{|}~€亗
```

调试程序发现加密算法只是将各位ASC码加8后保存。

密盘打开密码替换表与上相同。

## 五、建立特殊文件夹

```
004945FE  push dump\_.00494A60                         ; ASCII "recycle"
00494603  lea eax,dword ptr ss:[ebp-C]
00494606  mov edx,3
0049460B  call dump\_.0040444C
00494610  mov eax,dword ptr ds:[4B6114]
00494615  mov eax,dword ptr ds:[eax]
00494617  mov edx,dword ptr ss:[ebp-C]
0049461A  call dump\_.004AEE5C                         ; 建立recycle
0049461F  push dword ptr ss:[ebp-20]
00494622  push dump\_.00494A54
00494627  push dump\_.00494A60                         ; ASCII "recycle"
0049462C  push dump\_.00494A70
00494631  push dump\_.00494AAC                         ; ASCII "com1.{21ec2020-3aea-1069-a2dd-08002b30309d}"
00494636  lea eax,dword ptr ss:[ebp-18]
00494639  mov edx,5
0049463E  call dump\_.0040444C
00494643  lea edx,dword ptr ss:[ebp-44]
00494646  mov eax,dword ptr ss:[ebp-18]
00494649  call dump\_.00408DE4
0049464E  mov eax,dword ptr ss:[ebp-44]
00494651  call dump\_.00408AF0                         ; 建立S-1-5-21-1060284298-811497611-11778920086-500
00494656  xor eax,eax
00494658  push ebp
00494659  push dump\_.00494705
0049465E  push dword ptr fs:[eax]
00494661  mov dword ptr fs:[eax],esp
00494664  lea eax,dword ptr ss:[ebp-48]
00494667  mov ecx,dump\_.00494A54
0049466C  mov edx,dword ptr ss:[ebp-18]
0049466F  call dump\_.004043D8
00494674  mov eax,dword ptr ss:[ebp-48]
00494677  call dump\_.00408ACC
0049467C  test al,al
0049467E  jnz short dump\_.0049469D                    ; 判断com1.是否存在
00494680  lea eax,dword ptr ss:[ebp-4C]
00494683  mov ecx,dump\_.00494A54
00494688  mov edx,dword ptr ss:[ebp-18]
0049468B  call dump\_.004043D8
00494690  mov eax,dword ptr ss:[ebp-4C]
00494693  call <dump\_.CreateSpecialDir>               ; 建立com1.{21ec2020-3aea-1069-a2dd-08002b30309d}
00494698  call dump\_.0040284C
0049469D  push dword ptr ss:[ebp-20]
004946A0  push dump\_.00494A54
004946A5  push dump\_.00494A60                         ; ASCII "recycle"
004946AA  push dump\_.00494AE0
004946AF  lea eax,dword ptr ss:[ebp-50]
004946B2  mov edx,4
004946B7  call dump\_.0040444C
004946BC  mov eax,dword ptr ss:[ebp-50]
004946BF  call dump\_.00408ACC
004946C4  test al,al
004946C6  jnz short dump\_.004946F4                    ; 判断mp...\是否存在
004946C8  push dword ptr ss:[ebp-20]
004946CB  push dump\_.00494A54
004946D0  push dump\_.00494A60                         ; ASCII "recycle"
004946D5  push dump\_.00494AE0
004946DA  lea eax,dword ptr ss:[ebp-54]
004946DD  mov edx,4
004946E2  call dump\_.0040444C
004946E7  mov eax,dword ptr ss:[ebp-54]
004946EA  call <dump\_.CreateSpecialDir>               ; 建立mp...\
004946EF  call dump\_.0040284C
004946F4  push 64
004946F6  call <jmp.&kernel32.Sleep>
004946FB  xor eax,eax
004946FD  pop edx
004946FE  pop ecx
004946FF  pop ecx
00494700  mov dword ptr fs:[eax],edx
00494703  jmp short dump\_.00494728
00494705  jmp dump\_.0040381C
0049470A  mov eax,dump\_.00494B20
0049470F  call dump\_.004312DC
00494714  call dump\_.00403B84
00494719  call dump\_.00403BB4
0049471E  jmp dump\_.00494929
00494723  call dump\_.00403B84
00494728  lea eax,dword ptr ss:[ebp-14]
0049472B  mov ecx,dump\_.00494B4C                      ; ASCII "\data"
00494730  mov edx,dword ptr ss:[ebp-18]
00494733  call dump\_.004043D8
00494738  mov eax,dword ptr ss:[ebp-14]
0049473B  call dump\_.00408ACC
00494740  test al,al
00494742  jnz short dump\_.0049474C                    ; 判断data是否存在
00494744  mov eax,dword ptr ss:[ebp-14]
00494747  call <dump\_.CreateDir>                      ; 建立data
0049474C  push dword ptr ss:[ebp-14]
0049474F  push dump\_.00494A54
00494754  lea edx,dword ptr ss:[ebp-60]
00494757  mov eax,dword ptr ss:[ebp-4]
0049475A  mov eax,dword ptr ds:[eax+32C]
00494760  call dump\_.004573B4
00494765  mov eax,dword ptr ss:[ebp-60]
00494768  mov dl,byte ptr ds:[eax]
0049476A  lea eax,dword ptr ss:[ebp-5C]
0049476D  call dump\_.004042B4
00494772  push dword ptr ss:[ebp-5C]
00494775  lea eax,dword ptr ss:[ebp-58]
00494778  mov edx,3
0049477D  call dump\_.0040444C
00494782  mov eax,dword ptr ss:[ebp-58]
00494785  call dump\_.00408ACC
0049478A  test al,al
0049478C  jnz short dump\_.004947CC                    ; 判断盘符文件夹是否存在
0049478E  push dword ptr ss:[ebp-14]
00494791  push dump\_.00494A54
00494796  lea edx,dword ptr ss:[ebp-6C]
00494799  mov eax,dword ptr ss:[ebp-4]
0049479C  mov eax,dword ptr ds:[eax+32C]
004947A2  call dump\_.004573B4
004947A7  mov eax,dword ptr ss:[ebp-6C]
004947AA  mov dl,byte ptr ds:[eax]
004947AC  lea eax,dword ptr ss:[ebp-68]
004947AF  call dump\_.004042B4
004947B4  push dword ptr ss:[ebp-68]
004947B7  lea eax,dword ptr ss:[ebp-64]
004947BA  mov edx,3
004947BF  call dump\_.0040444C
004947C4  mov eax,dword ptr ss:[ebp-64]
004947C7  call <dump\_.CreateDir>                      ; 建立盘符文件夹
004947CC  lea eax,dword ptr ss:[ebp-8]
004947CF  mov ecx,dump\_.00494B5C                      ; ASCII "\panfile2.dll"
004947D4  mov edx,dword ptr ss:[ebp-14]               ; 保存密盘的设置
004947D7  call dump\_.004043D8
004947DC  xor eax,eax
004947DE  push ebp
004947DF  push dump\_.004948CB
004947E4  push dword ptr fs:[eax]
004947E7  mov dword ptr fs:[eax],esp
004947EA  mov ecx,dword ptr ss:[ebp-8]
004947ED  mov dl,1
004947EF  mov eax,dword ptr ds:[436CC4]
004947F4  call dump\_.00436D74
004947F9  mov dword ptr ss:[ebp-10],eax
004947FC  lea edx,dword ptr ss:[ebp-70]
004947FF  mov eax,dword ptr ss:[ebp-4]
00494802  mov eax,dword ptr ds:[eax+304]
00494808  call dump\_.004573B4
0049480D  mov eax,dword ptr ss:[ebp-70]
00494810  push eax
00494811  lea edx,dword ptr ss:[ebp-78]
00494814  mov eax,dword ptr ss:[ebp-4]
00494817  mov eax,dword ptr ds:[eax+32C]
0049481D  call dump\_.004573B4
00494822  mov eax,dword ptr ss:[ebp-78]
00494825  mov dl,byte ptr ds:[eax]
00494827  lea eax,dword ptr ss:[ebp-74]
0049482A  call dump\_.004042B4
0049482F  mov edx,dword ptr ss:[ebp-74]
00494832  mov ecx,dump\_.00494B74                      ; ASCII "name"，密盘名称
00494837  mov eax,dword ptr ss:[ebp-10]
0049483A  mov ebx,dword ptr ds:[eax]
0049483C  call dword ptr ds:[ebx+4]
0049483F  mov eax,dword ptr ss:[ebp-4]
00494842  mov eax,dword ptr ds:[eax+314]
00494848  mov edx,dword ptr ds:[eax]
0049484A  call dword ptr ds:[edx+C8]
00494850  test al,al
00494852  je short dump\_.004948B5
00494854  lea edx,dword ptr ss:[ebp-80]
00494857  mov eax,dword ptr ss:[ebp-4]
0049485A  mov eax,dword ptr ds:[eax+318]
00494860  call dump\_.004573B4
00494865  mov edx,dword ptr ss:[ebp-80]
00494868  lea ecx,dword ptr ss:[ebp-7C]
0049486B  mov eax,dword ptr ds:[4B6114]
00494870  mov eax,dword ptr ds:[eax]
00494872  call dump\_.004AFCA8
00494877  mov eax,dword ptr ss:[ebp-7C]
0049487A  push eax
0049487B  lea edx,dword ptr ss:[ebp-88]
00494881  mov eax,dword ptr ss:[ebp-4]
00494884  mov eax,dword ptr ds:[eax+32C]
0049488A  call dump\_.004573B4
0049488F  mov eax,dword ptr ss:[ebp-88]
00494895  mov dl,byte ptr ds:[eax]
00494897  lea eax,dword ptr ss:[ebp-84]
0049489D  call dump\_.004042B4
004948A2  mov edx,dword ptr ss:[ebp-84]
004948A8  mov ecx,dump\_.00494B84                      ; ASCII "ps"，密盘打开密码
004948AD  mov eax,dword ptr ss:[ebp-10]
004948B0  mov ebx,dword ptr ds:[eax]
004948B2  call dword ptr ds:[ebx+4]
```

经测试文件夹都是用`CreateDirectoryA`创建的，用`RemoveDirectoryA`可以直接删除该非法文件夹。

删除2个非法文件夹的代码：

```
RemoveDirectory("E:\recycle\S-1-5-21-1060284298-811497611-11778920086-500\com1.{21ec2020-3aea-1069-a2dd-08002b30309d}\")
```
