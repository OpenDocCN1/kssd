
```
******Possible String Ref To->"29"  

                      |  

:00402F94  1B6F00              LitStr               ;Push ptr_00402658  

:00402F97  2A                  ConcatStr            ;vbaStrCat  

************************************************************  

    Case "q"  

      strCode = strCode & "29"  

************************************************************  

:00402F98  3170FF              FStStr               ;SysFreeString [LOCAL_0090]; [LOCAL_0090]=Pop  

:00402F9B  1EAA08              Branch               ;ESI=0040308E  

:00402F9E  04A4FE              FLdRfVar             ;Push LOCAL_015C  

******Possible String Ref To->"r"  

                      |  

:00402FA1  3A48FF7000          LitVarStr            ;PushVarString ptr_00402664  

:00402FA6  5D                  HardType             ;  

:00402FA7  FB33                EqVarBool            ;  

:00402FA9  1CD507              BranchF              ;If Pop=0 then ESI=00402FB9  

:00402FAC  6C70FF              ILdRf                ;Push DWORD [LOCAL_0090]  

******Possible String Ref To->"57"  

                      |  

:00402FAF  1B7100              LitStr               ;Push ptr_0040266C  

:00402FB2  2A                  ConcatStr            ;vbaStrCat  

************************************************************  

    Case "r"  

      strCode = strCode & "57"  

************************************************************  

:00402FB3  3170FF              FStStr               ;SysFreeString [LOCAL_0090]; [LOCAL_0090]=Pop  

:00402FB6  1EAA08              Branch               ;ESI=0040308E  

:00402FB9  04A4FE              FLdRfVar             ;Push LOCAL_015C  

******Possible String Ref To->"s"  

                      |  

:00402FBC  3A48FF7200          LitVarStr            ;PushVarString ptr_00402678  

:00402FC1  5D                  HardType             ;  

:00402FC2  FB33                EqVarBool            ;  

:00402FC4  1CF007              BranchF              ;If Pop=0 then ESI=00402FD4  

:00402FC7  6C70FF              ILdRf                ;Push DWORD [LOCAL_0090]  

******Possible String Ref To->"50"  

                      |  

:00402FCA  1B7300              LitStr               ;Push ptr_00402680  

:00402FCD  2A                  ConcatStr            ;vbaStrCat  

************************************************************  

    Case "s"  

      strCode = strCode & "50"  

************************************************************  

:00402FCE  3170FF              FStStr               ;SysFreeString [LOCAL_0090]; [LOCAL_0090]=Pop  

:00402FD1  1EAA08              Branch               ;ESI=0040308E  

:00402FD4  04A4FE              FLdRfVar             ;Push LOCAL_015C  

******Possible String Ref To->"t"  

                      |  

:00402FD7  3A48FF7400          LitVarStr            ;PushVarString ptr_0040268C  

:00402FDC  5D                  HardType             ;  

:00402FDD  FB33                EqVarBool            ;  

:00402FDF  1C0B08              BranchF              ;If Pop=0 then ESI=00402FEF  

:00402FE2  6C70FF              ILdRf                ;Push DWORD [LOCAL_0090]  

******Possible String Ref To->"59"  

                      |  

:00402FE5  1B7500              LitStr               ;Push ptr_00402694  

:00402FE8  2A                  ConcatStr            ;vbaStrCat  

************************************************************  

    Case "t"  

      strCode = strCode & "59"  

************************************************************  

:00402FE9  3170FF              FStStr               ;SysFreeString [LOCAL_0090]; [LOCAL_0090]=Pop  

:00402FEC  1EAA08              Branch               ;ESI=0040308E  

:00402FEF  04A4FE              FLdRfVar             ;Push LOCAL_015C  

******Possible String Ref To->"u"  

                      |  

:00402FF2  3A48FF7600          LitVarStr            ;PushVarString ptr_004026A0  

:00402FF7  5D                  HardType             ;  

:00402FF8  FB33                EqVarBool            ;  

:00402FFA  1C2608              BranchF              ;If Pop=0 then ESI=0040300A  

:00402FFD  6C70FF              ILdRf                ;Push DWORD [LOCAL_0090]  

******Possible String Ref To->"38"  

                      |  

:00403000  1B7700              LitStr               ;Push ptr_004026A8  

:00403003  2A                  ConcatStr            ;vbaStrCat  

************************************************************  

    Case "u"  

      strCode = strCode & "38"  

************************************************************  

:00403004  3170FF              FStStr               ;SysFreeString [LOCAL_0090]; [LOCAL_0090]=Pop  

:00403007  1EAA08              Branch               ;ESI=0040308E  

:0040300A  04A4FE              FLdRfVar             ;Push LOCAL_015C  

******Possible String Ref To->"v"  

                      |  

:0040300D  3A48FF7800          LitVarStr            ;PushVarString ptr_004026B4  

:00403012  5D                  HardType             ;  

:00403013  FB33                EqVarBool            ;  

:00403015  1C4108              BranchF              ;If Pop=0 then ESI=00403025  

:00403018  6C70FF              ILdRf                ;Push DWORD [LOCAL_0090]  

******Possible String Ref To->"46"  

                      |  

:0040301B  1B7900              LitStr               ;Push ptr_004026BC  

:0040301E  2A                  ConcatStr            ;vbaStrCat  

************************************************************  

    Case "v"  

      strCode = strCode & "46"  

************************************************************  

:0040301F  3170FF              FStStr               ;SysFreeString [LOCAL_0090]; [LOCAL_0090]=Pop  

:00403022  1EAA08              Branch               ;ESI=0040308E  

:00403025  04A4FE              FLdRfVar             ;Push LOCAL_015C  

******Possible String Ref To->"w"  

                      |  

:00403028  3A48FF7A00          LitVarStr            ;PushVarString ptr_004026C8  

:0040302D  5D                  HardType             ;  

:0040302E  FB33                EqVarBool            ;  

:00403030  1C5C08              BranchF              ;If Pop=0 then ESI=00403040  

:00403033  6C70FF              ILdRf                ;Push DWORD [LOCAL_0090]  

******Possible String Ref To->"58"  

                      |  

:00403036  1B7B00              LitStr               ;Push ptr_004026D0  

:00403039  2A                  ConcatStr            ;vbaStrCat  

************************************************************  

    Case "w"  

      strCode = strCode & "58"  

************************************************************  

:0040303A  3170FF              FStStr               ;SysFreeString [LOCAL_0090]; [LOCAL_0090]=Pop  

:0040303D  1EAA08              Branch               ;ESI=0040308E  

:00403040  04A4FE              FLdRfVar             ;Push LOCAL_015C  

******Possible String Ref To->"x"  

                      |  

:00403043  3A48FF7C00          LitVarStr            ;PushVarString ptr_004026DC  

:00403048  5D                  HardType             ;  

:00403049  FB33                EqVarBool            ;  

:0040304B  1C7708              BranchF              ;If Pop=0 then ESI=0040305B  

:0040304E  6C70FF              ILdRf                ;Push DWORD [LOCAL_0090]  
```
