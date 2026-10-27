
```
00674190    .  51     push ecx                                   ; /Arg4

00674191    .  8D95 E>lea edx,dword ptr ss:[ebp-11C]             ; |

00674197    .  6A 06  push 6                                     ; |Arg3 = 00000006

00674199    .  8D45 8>lea eax,dword ptr ss:[ebp-7C]              ; |

0067419C    .  BE 54D>mov esi,pkgs910.0042D154                   ; |0042d154是“-”

006741A1    .  52     push edx                                   ; |Arg2

006741A2    .  50     push eax                                   ; |Arg1

006741A3    .  89B5 0>mov dword ptr ss:[ebp-F4],esi              ; |

006741A9    .  89BD 0>mov dword ptr ss:[ebp-FC],edi              ; |

006741AF    .  C785 F>mov dword ptr ss:[ebp-104],pkgs910.0043EDA>; |0043edac是字母s

006741B9    .  89BD F>mov dword ptr ss:[ebp-10C],edi             ; |

006741BF    .  C745 9>mov dword ptr ss:[ebp-64],5                ; |

006741C6    .  C745 9>mov dword ptr ss:[ebp-6C],2                ; |

006741CD    .  899D E>mov dword ptr ss:[ebp-11C],ebx             ; |

006741D3    .  FF15 F>call dword ptr ds:[<&MSVBVM60.#632>]       ; \取中间5位

006741D9    .  8B0D B>mov ecx,dword ptr ds:[7394B0]

006741DF    .  8D95 A>lea edx,dword ptr ss:[ebp-15C]

006741E5    .  6A 04  push 4                                     ; /Arg3 = 00000004

006741E7    .  8D85 4>lea eax,dword ptr ss:[ebp-BC]              ; |

006741ED    .  52     push edx                                   ; |Arg2

006741EE    .  50     push eax                                   ; |Arg1

006741EF    .  89B5 C>mov dword ptr ss:[ebp-134],esi             ; |

006741F5    .  89BD C>mov dword ptr ss:[ebp-13C],edi             ; |

006741FB    .  C785 B>mov dword ptr ss:[ebp-144],pkgs910.0043EDB>; |43EDB4的值35

00674205    .  89BD B>mov dword ptr ss:[ebp-14C],edi             ; |

0067420B    .  898D A>mov dword ptr ss:[ebp-154],ecx             ; |

00674211    .  899D A>mov dword ptr ss:[ebp-15C],ebx             ; |

00674217    .  FF15 5>call dword ptr ds:[<&MSVBVM60.#619>]       ; \取最后4位都在堆栈中有

0067421D    .  8B35 9>mov esi,dword ptr ds:[<&MSVBVM60.\_\_vbaVarC>;  MSVBVM60.\_\_vbaVarCat

00674223    .  8D8D 1>lea ecx,dword ptr ss:[ebp-EC]

00674229    .  8D55 D>lea edx,dword ptr ss:[ebp-2C]

0067422C    .  51     push ecx

0067422D    .  8D45 C>lea eax,dword ptr ss:[ebp-3C]

00674230    .  52     push edx

00674231    .  50     push eax

00674232    .  FFD6   call esi                                   ;  组合 前5位前加了一个1变成1hxhgg; <&MSVBVM60.\_\_vbaVarCat>

00674234    .  8D8D 0>lea ecx,dword ptr ss:[ebp-FC]

0067423A    .  50     push eax

0067423B    .  8D55 B>lea edx,dword ptr ss:[ebp-4C]

0067423E    .  51     push ecx

0067423F    .  52     push edx

00674240    .  FFD6   call esi                                   ;  组合  1hxhgg-

00674242    .  50     push eax

00674243    .  8D85 F>lea eax,dword ptr ss:[ebp-10C]

00674249    .  8D4D A>lea ecx,dword ptr ss:[ebp-5C]

0067424C    .  50     push eax

0067424D    .  51     push ecx

0067424E    .  FFD6   call esi                                   ;  组合1hxhgg-s与S组合

00674250    .  50     push eax

00674251    .  8D55 8>lea edx,dword ptr ss:[ebp-7C]

00674254    .  8D85 7>lea eax,dword ptr ss:[ebp-8C]

0067425A    .  52     push edx

0067425B    .  50     push eax

0067425C    .  FFD6   call esi                                   ;  与中间5位组合1hxhgg-s2x3d9

0067425E    .  8D8D C>lea ecx,dword ptr ss:[ebp-13C]

00674264    .  50     push eax

00674265    .  8D95 6>lea edx,dword ptr ss:[ebp-9C]

0067426B    .  51     push ecx

0067426C    .  52     push edx

0067426D    .  FFD6   call esi                                   ;  与“-”组合1hxhgg-s2x3d9-

0067426F    .  50     push eax

00674270    .  8D85 B>lea eax,dword ptr ss:[ebp-14C]

00674276    .  8D8D 5>lea ecx,dword ptr ss:[ebp-AC]

0067427C    .  50     push eax

0067427D    .  51     push ecx

0067427E    .  FFD6   call esi                                   ;  再与35组合1hxhgg-s2x3d9-35

00674280    .  50     push eax

00674281    .  8D95 4>lea edx,dword ptr ss:[ebp-BC]

00674287    .  8D85 3>lea eax,dword ptr ss:[ebp-CC]

0067428D    .  52     push edx

0067428E    .  50     push eax

0067428F    .  FFD6   call esi                                   ;  再组合后4位1hxhgg-s2x3d9-352gdx

00674291    .  50     push eax

00674292    .  FF15 3>call dword ptr ds:[<&MSVBVM60.\_\_vbaStrVarM>;  MSVBVM60.\_\_vbaStrVarMove

00674298    .  8BD0   mov edx,eax

0067429A    .  8D4D E>lea ecx,dword ptr ss:[ebp-1C]

0067429D    .  FF15 4>call dword ptr ds:[<&MSVBVM60.\_\_vbaStrMove>;  MSVBVM60.\_\_vbaStrMove

006742A3    .  8B0D B>mov ecx,dword ptr ds:[7394B0]

006742A9    .  8BD0   mov edx,eax

006742AB    .  FF15 C>call dword ptr ds:[<&MSVBVM60.\_\_vbaStrCopy>;  MSVBVM60.\_\_vbaStrCopy

006742B1    .  8D4D E>lea ecx,dword ptr ss:[ebp-1C]

006742B4    .  FF15 9>call dword ptr ds:[<&MSVBVM60.\_\_vbaFreeStr>;  MSVBVM60.\_\_vbaFreeStr

006742BA    .  8D8D 3>lea ecx,dword ptr ss:[ebp-CC]

006742C0    .  8D95 4>lea edx,dword ptr ss:[ebp-BC]

006742C6    .  51     push ecx

006742C7    .  8D85 5>lea eax,dword ptr ss:[ebp-AC]

006742CD    .  52     push edx

006742CE    .  8D8D 6>lea ecx,dword ptr ss:[ebp-9C]

006742D4    .  50     push eax

006742D5    .  8D95 7>lea edx,dword ptr ss:[ebp-8C]

006742DB    .  51     push ecx

006742DC    .  8D45 8>lea eax,dword ptr ss:[ebp-7C]

006742DF    .  52     push edx

006742E0    .  8D4D A>lea ecx,dword ptr ss:[ebp-5C]

006742E3    .  50     push eax

006742E4    .  8D55 9>lea edx,dword ptr ss:[ebp-6C]

006742E7    .  51     push ecx

006742E8    .  8D45 B>lea eax,dword ptr ss:[ebp-4C]

006742EB    .  52     push edx

006742EC    .  8D4D C>lea ecx,dword ptr ss:[ebp-3C]

006742EF    .  50     push eax

006742F0    .  8D55 D>lea edx,dword ptr ss:[ebp-2C]

006742F3    .  51     push ecx

006742F4    .  52     push edx

006742F5    .  6A 0B  push 0B

006742F7    .  FF15 4>call dword ptr ds:[<&MSVBVM60.\_\_vbaFreeVar>;  MSVBVM60.\_\_vbaFreeVarList

006742FD    .  A1 B09>mov eax,dword ptr ds:[7394B0]

00674302    .  83C4 3>add esp,30

00674305    .  8D8D 2>lea ecx,dword ptr ss:[ebp-DC]

0067430B    .  8D55 D>lea edx,dword ptr ss:[ebp-2C]

0067430E    .  6A 05  push 5                                     ; /Arg3 = 00000005

00674310    .  83C0 0>add eax,4                                  ; |

00674313    .  51     push ecx                                   ; |Arg2

00674314    .  52     push edx                                   ; |Arg1

00674315    .  C785 1>mov dword ptr ss:[ebp-E4],pkgs910.00433218 ; |

0067431F    .  89BD 1>mov dword ptr ss:[ebp-EC],edi              ; |

00674325    .  8985 2>mov dword ptr ss:[ebp-D4],eax              ; |

0067432B    .  899D 2>mov dword ptr ss:[ebp-DC],ebx              ; |

00674331    .  FF15 3>call dword ptr ds:[<&MSVBVM60.#617>]       ; \再取机器码第2次计算出来的值的前5位

00674337    .  A1 B09>mov eax,dword ptr ds:[7394B0]

0067433C    .  8D4D 9>lea ecx,dword ptr ss:[ebp-6C]

0067433F    .  83C0 0>add eax,4

00674342    .  51     push ecx                                   ; /Arg4

00674343    .  8985 E>mov dword ptr ss:[ebp-114],eax             ; |

00674349    .  8D95 E>lea edx,dword ptr ss:[ebp-11C]             ; |

0067434F    .  6A 06  push 6                                     ; |Arg3 = 00000006

00674351    .  8D45 8>lea eax,dword ptr ss:[ebp-7C]              ; |

00674354    .  52     push edx                                   ; |Arg2

00674355    .  50     push eax                                   ; |Arg1

00674356    .  C785 0>mov dword ptr ss:[ebp-F4],pkgs910.0042D154 ; |0042D154的值是”-“

00674360    .  89BD 0>mov dword ptr ss:[ebp-FC],edi              ; |

00674366    .  C785 F>mov dword ptr ss:[ebp-104],pkgs910.0042E86>; |0042e860的值”y"下面要与他们组合

00674370    .  89BD F>mov dword ptr ss:[ebp-10C],edi             ; |

00674376    .  C745 9>mov dword ptr ss:[ebp-64],5                ; |

0067437D    .  C745 9>mov dword ptr ss:[ebp-6C],2                ; |

00674384    .  899D E>mov dword ptr ss:[ebp-11C],ebx             ; |
```
