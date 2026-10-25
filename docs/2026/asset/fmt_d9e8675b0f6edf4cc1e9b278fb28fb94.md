
## 解码过程

```
00490507  |. 42               INC EDX                                  |EDX=EDX+1=2
00490508  |. 48               DEC EAX                                  |EAX=EAX-1=B
00490509  |.^75 E9            JNZ SHORT 局域网查.004904F4              |循环
```

`004904E6`～`00490509`代码计算`((((((((((((X1*A)+X2)*A+X3)*A+X4)*A+X5)*A+X6)*A+X7)*A+X8)*A+X9)*A+X10)*A+X11)*A+X12)*A`

`Xn`代表对应各位的`ASCII`值，计算结果取后8位

本例中 `30B970AEEADE`→`EBX`= `70AEEADE`

```
0049050B  |> F7D3             NOT EBX                                  |EBX=NOT EBX=8F511521
0049050D  |. 81F3 80275082    XOR EBX,82502780                         |EBX=EBX XOR 82502780=0D0132A1
00490513  |. 8BC3             MOV EAX,EBX                              |EAX=EBX=0D0132A1
00490515  |. 25 000000FF      AND EAX,FF000000                         |EAX=EAX AND FF000000=0D000000
0049051A  |. C1E8 15          SHR EAX,15                               |EAX=EAX SHR 15=00000068
0049051D  |. 8906             MOV DWORD PTR DS:[ESI],EAX               |入ESI=00000068
0049051F  |. 8BC3             MOV EAX,EBX                              |EAX=EBX=0D0132A1
00490521  |. 25 0000FF00      AND EAX,FF0000                           |EAX=EAX AND FF0000=00010000
00490526  |. C1E8 11          SHR EAX,11                               |EAX=EAX SHR 11=00000000
00490529  |. 8946 04          MOV DWORD PTR DS:[ESI+4],EAX             |入ESI+4=00000000
0049052C  |. 8BC3             MOV EAX,EBX                              |EAX=EBX=0D0132A1
0049052E  |. 25 00FF0000      AND EAX,FF00                             |EAX=EAX AND FF00=00003200
00490533  |. C1E8 07          SHR EAX,7                                |EAX=EAX SHR 7=00000064
00490536  |. 8946 08          MOV DWORD PTR DS:[ESI+8],EAX             |入ESI+8=00000064
00490539  |. 8BC3             MOV EAX,EBX                              |EAX=EBX=0D0132A1
0049053B  |. 25 FF000000      AND EAX,FF                               |EAX=EAX AND FF=000000A1
00490540  |. 8946 0C          MOV DWORD PTR DS:[ESI+C],EAX             |入ESI+C=000000A1
```

`ESI`= `00000068`、`ESI+4`= `00000000`、`ESI+8`= `00000064`、`ESI+C`= `000000A1`

```
00490543  |. 8B16             MOV EDX,DWORD PTR DS:[ESI]               |EDX=ESI=00000068
00490545  |. 81E2 C0000000    AND EDX,C0                               |EDX=EDX AND CO=00000040
0049054B  |. 8B4E 04          MOV ECX,DWORD PTR DS:[ESI+4]             |ECX=ESI+4=00000000
0049054E  |. 81E1 C0000000    AND ECX,C0                               |ECX=ECX AND CO=00000000
00490554  |. C1E9 02          SHR ECX,2                                |ECX=ECX SHR 2=00000000
00490557  |. 03D1             ADD EDX,ECX                              |EDX=EDX+ECX=00000040
00490559  |. 8B4E 08          MOV ECX,DWORD PTR DS:[ESI+8]             |ECX=ESI+8=00000064
0049055C  |. 81E1 C0000000    AND ECX,C0                               |ECX=ECX AND CO=00000040
00490562  |. C1E9 04          SHR ECX,4                                |ECX=ECX SHR 4=00000004
00490565  |. 03D1             ADD EDX,ECX                              |EDX=EDX+ECX=00000044
00490567  |. 25 C0000000      AND EAX,C0                               |ECX=ECX AND CO=00000080
0049056C  |. C1E8 05          SHR EAX,5                                |EAX=EAX SHR 5=00000004
0049056F  |. 03D0             ADD EDX,EAX                              |EDX=EDX+EAX=00000048
00490571  |. 8955 D4          MOV DWORD PTR SS:[EBP-2C],EDX            |入EBP-2C=00000048
00490574  |. 8B06             MOV EAX,DWORD PTR DS:[ESI]               |EAX=ESI=00000068
00490576  |. 83E0 30          AND EAX,30                               |EAX=EAX AND 30=00000020
00490579  |. C1E0 02          SHL EAX,2                                |EAX=EAX SHL 2=00000080
0049057C  |. 8B56 04          MOV EDX,DWORD PTR DS:[ESI+4]             |EDX=ESI+4=00000000
0049057F  |. 83E2 30          AND EDX,30                               |EDX=EDX AND 30=00000000
00490582  |. 03C2             ADD EAX,EDX                              |EAX=EAX+EDX=00000080
00490584  |. 8B56 08          MOV EDX,DWORD PTR DS:[ESI+8]             |EDX=ESI+8=00000064
00490587  |. 83E2 30          AND EDX,30                               |EDX=EDX AND 3O=00000020
0049058A  |. C1EA 02          SHR EDX,2                                |EDX=EDX SHR 2=00000008
0049058D  |. 03C2             ADD EAX,EDX                              |EAX=EAX+EDX=00000088
0049058F  |. 8B56 0C          MOV EDX,DWORD PTR DS:[ESI+C]             |EDX=ESI+C=000000A1
00490592  |. 83E2 30          AND EDX,30                               |EDX=EDX AND 3O=00000020
00490595  |. C1EA 06          SHR EDX,6                                |EDX=EDX SHR 6=00000000
00490598  |. 03C2             ADD EAX,EDX                              |EAX=EAX+EDX=00000088
0049059A  |. 8945 D8          MOV DWORD PTR SS:[EBP-28],EAX            |入EBP-28=00000088
0049059D  |. 8B06             MOV EAX,DWORD PTR DS:[ESI]               |EAX=ESI=00000068
0049059F  |. 83E0 0C          AND EAX,C                                |EAX=EAX AND C=00000008
004905A2  |. C1E0 04          SHL EAX,4                                |EAX=EAX SHL 4=00000080
004905A5  |. 8B56 04          MOV EDX,DWORD PTR DS:[ESI+4]             |EDX=ESI+4=00000000
004905A8  |. 83E2 0C          AND EDX,C                                |EDX=EDX AND C=00000000
004905AB  |. C1E2 02          SHL EDX,2                                |EDX=EDX SHL 2=00000000
004905AE  |. 03C2             ADD EAX,EDX                              |EAX=EAX+EDX=00000080
004905B0  |. 8B56 08          MOV EDX,DWORD PTR DS:[ESI+8]             |EDX=ESI+8=00000064
004905B3  |. 83E2 0C          AND EDX,C                                |EDX=EDX AND C=00000004
004905B6  |. 03C2             ADD EAX,EDX                              |EAX=EAX+EDX=00000084
004905B8  |. 8B56 0C          MOV EDX,DWORD PTR DS:[ESI+C]             |EDX=ESI+C=000000A1
004905BB  |. 83E2 0C          AND EDX,C                                |EDX=EDX AND C=00000000
004905BE  |. C1EA 03          SHR EDX,3                                |EDX=EDX SHR 3=00000000
004905C1  |. 03C2             ADD EAX,EDX                              |EAX=EAX+EDX=00000084
004905C3  |. 8945 DC          MOV DWORD PTR SS:[EBP-24],EAX            |入EBP-24=00000084
004905C6  |. 8B06             MOV EAX,DWORD PTR DS:[ESI]               |EAX=ESI=00000068
004905C8  |. 83E0 03          AND EAX,3                                |EAX=EAX AND 3=00000000
004905CB  |. C1E0 06          SHL EAX,6                                |EAX=EAX SHL 6=00000000
004905CE  |. 8B56 04          MOV EDX,DWORD PTR DS:[ESI+4]             |EDX=ESI+4=00000000
004905D1  |. 83E2 03          AND EDX,3                                |EDX=EDX AND 3=00000000
004905D4  |. C1E2 04          SHL EDX,4                                |EDX=EDX SHL 4=00000000
004905D7  |. 03C2             ADD EAX,EDX                              |EAX=EAX+EDX=00000000
004905D9  |. 8B56 08          MOV EDX,DWORD PTR DS:[ESI+8]             |EDX=ESI+8=00000064
004905DC  |. 83E2 03          AND EDX,3                                |EDX=EDX AND 3=00000000
004905DF  |. C1E2 02          SHL EDX,2                                |EDX=EDX SHL 2=00000000
004905E2  |. 03C2             ADD EAX,EDX                              |EAX=EAX+EDX=00000000
004905E4  |. 8B56 0C          MOV EDX,DWORD PTR DS:[ESI+C]             |EDX=ESI+C=000000A1
004905E7  |. 83E2 03          AND EDX,3                                |EDX=EDX AND 3=00000001
004905EA  |. 03C2             ADD EAX,EDX                              |EAX=EAX+EDX=00000001
004905EC  |. 8945 E0          MOV DWORD PTR SS:[EBP-20],EAX            |入EBP-20=00000001
```

`EBP-2C`= `00000048`、`EBP-28`= `00000088`、`EBP-24`= `00000084`、`EBP-20`= `00000001`

```
004905EF  |. 8B5D D4          MOV EBX,DWORD PTR SS:[EBP-2C]             |EBX= EBP-2C=00000048
004905F2  |. C1E3 18          SHL EBX,18                                |EBX=EBX SHL 18=48000000
004905F5  |. 8B45 D8          MOV EAX,DWORD PTR SS:[EBP-28]             |EAX= EBP-28=00000088
```
