
```
:0044C40B E89052FEFF    call 004316A0

:0044C410 83C404        add esp, 00000004

:0044C413 898534FFFFFF  mov dword ptr [ebp+FFFFFF34], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0044C404(U)

|

:0044C419 6A00          push 00000000

:0044C41B 6A00          push 00000000

* Possible StringData Ref from Data Obj ->"Error Saving Registration Codes, "

                                            ->"..

Registration is incomplete"

                                            |

:0044C41D 6808644700    push 00476408

* Reference To: MFC42.Ordinal:04B0, Ord:04B0h

                                            |

:0044C422 E821920000    Call 00455648

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0044C3EF(U)

|

:0044C427 6A00          push 00000000

:0044C429 6A00          push 00000000

* Possible StringData Ref from Data Obj ->"Thank you for Registering SWF "

                                            ->"Scanner, Please Close the program "

                                            ->"and open it again to complete "

                                            ->"the process"

                                            |

:0044C42B 6848644700    push 00476448

* Reference To: MFC42.Ordinal:04B0, Ord:04B0h

                                            |

:0044C430 E813920000    Call 00455648……注册成功的消息窗！

:0044C435 8B8D60FFFFFF  mov ecx, dword ptr [ebp+FFFFFF60]

* Reference To: MFC42.Ordinal:12F5, Ord:12F5h

                                            |

:0044C43B E838950000    Call 00455978

:0044C440 C645FC00      mov [ebp-04], 00

:0044C444 8D8D78FFFFFF  lea ecx, dword ptr [ebp+FFFFFF78]

:0044C44A E8C71A0000    call 0044DF16

:0044C44F EB0E          jmp 0044C45F…………成功的跳转噢！：〉

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0044C2B7(C)

|

:0044C451 6A00          push 00000000

:0044C453 6A00          push 00000000

* Possible StringData Ref from Data Obj ->"Sorry, The codes you have entered "

                                            ->"are invalid. Please Register at "

                                            ->"www.livetronix.com"

                                            |

:0044C455 68B4644700    push 004764B4

* Reference To: MFC42.Ordinal:04B0, Ord:04B0h

                                            |

:0044C45A E8E9910000    Call 00455648…………注册失败的消息窗！

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0044C2AD(C), :0044C44F(U)……看看这里！

|

:0044C45F 8B8D60FFFFFF  mov ecx, dword ptr [ebp+FFFFFF60]……跳到这里还有戏吗？当然有！

只要你看看它的第二个跳转源就知道了！

* Reference To: MFC42.Ordinal:1118, Ord:1118h

                                            |

:0044C465 E8CC940000    Call 00455936

:0044C46A C745FCFFFFFFFF mov [ebp-04], FFFFFFFF

:0044C471 8D4D88         lea ecx, dword ptr [ebp-78]

:0044C474 E84708FFFF    call 0043CCC0

:0044C479 8B4DF4         mov ecx, dword ptr [ebp-0C]

:0044C47C 64890D00000000 mov dword ptr fs:[00000000], ecx

:0044C483 8BE5           mov esp, ebp

:0044C485 5D             pop ebp

:0044C486 C3             ret
```

好了，反正咱们已经达到目的了，俺就不管了。有时间再写注册机吧。

还多亏了好友 `fwnl` 的帮助，不然，俺又该继续郁闷了！

MD，我怎么有一点比破解女生还要来的兴奋？

同志们有嘛？

顺便给一个注册：

注册名：`ericluom`

注册码：`SCN-114116122856980-z107z98-0000`
`4dd466d0e1e0cc123afbe504da2462ca`
