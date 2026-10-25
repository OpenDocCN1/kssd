
`:004080CC` `83C9FF`
`or ecx, FFFFFFFF`

`:004080CF` `33C0`
`xor eax, eax`

`:004080D1` `F2`
`repnz`

`:004080D2` `AE`
`scasb`

`:004080D3` `F7D1`
`not ecx`

`:004080D5` `2BF9`
`sub edi, ecx`

`:004080D7` `8BD1`
`mov edx, ecx`

`:004080D9` `8BF7`
`mov esi, edi`

`:004080DB` `8BFB`
`mov edi, ebx`

`:004080DD` `C1E902`
`shr ecx, 02`

`:004080E0` `F3`
`repz`

`:004080E1` `A5`
`movsd`

`:004080E2` `8BCA`
`mov ecx, edx`

`:004080E4` `83E103`
`and ecx, 00000003`

`:004080E7` `F3`
`repz`

`:004080E8` `A4`
`movsb`

// copy "Guest" to `EBX`

`:004080E9` `8B742410` `mov esi, dword ptr [esp+10]`

`:004080ED` `B905000000`
`mov ecx, 00000005`

//  }

// Referenced by a (U)nconditional or (C)onditional Jump at Address:
// |:004080C5(C)
// |

`:004080F2` `8BC5`
`mov eax, ebp`

// set    (0X0F - IdxCodeFirstCharInStr) to EAX

`:004080F4`
`99` `cdq`

`:004080F5`
`F7F9` `idiv ecx`

// EDX = EAX % ECX
// ECX = (NameIdLen % 7) ? (NameIdLen % 7) : 5 ;
// EBX = (NameIdLen % 7) ? NameId : "Guest"
// SET    (0X0F - IdxCodeFirstCharInStr) % ECX to EDX

`:004080F7` `0FBE041A`
`movsx eax, byte ptr [edx+ebx]`

// SET    EAX = (DWORD)[EBX+EDX]

`:004080FB` `250F000080`
`and eax, 8000000F`

`:00408100` `7905`
`jns 00408107`

`:00408102` `48`
`dec eax`

`:00408103` `83C8F0`
`or eax, FFFFFFF0`

`:00408106` `40`
`inc eax`

//  ???
// support portion of chinese
// or bug

// Referenced by a (U)nconditional or (C)onditional Jump at Address:
// |:00408100(C)
// |

`:00408107` `8A8828D54100`
`mov cl, byte ptr [eax+0041D528]`

`:0040810D` `8A442415`
`mov al, byte ptr [esp+15]`

`:00408111`
`3AC8` `cmp cl, al`

`:00408113` `740A`
`je 0040811F`      // must jmp
// sCode[1] = str[eax]

// Possible StringData Ref from Data Obj ->"99#SDFG^\*&"

`:00408115` `BF5CD54100`
`mov edi, 0041D55C`

`:0040811A` `E9C5000000`
`jmp 004081E4`

// Referenced by a (U)nconditional or (C)onditional Jump at Address:
// |:00408113(C)
// |

`:0040811F` `8D46FD` `lea eax, dword ptr [esi-03]`
// eax = CodeLen - 3

`:00408122` `33FF`
`xor edi, edi`

`:00408124` `99`
`cdq`

`:00408125` `2BC2`
`sub eax, edx`

`:00408127` `D1F8`
`sar eax, 1`

`:00408129` `85C0`
`test eax, eax`

`:0040812B` `0F8E8F000000`
`jle 004081C0`

`:00408131` `8D742416`
`lea esi, dword ptr [esp+16]`
// ESI = &Code[2];
// for(;;)    //  004081B6
// Referenced by a (U)nconditional or (C)onditional Jump at Address:
// |:004081B6(C)
// |

`:00408135` `8A5601` `mov dl, byte ptr [esi+01]`
// DL = Code[2+2*EAX+1]

`:00408138` `33C9`
`xor ecx, ecx`

// Referenced by a (U)nconditional or (C)onditional Jump at Address:
// |:00408146(C)
// |

`:0040813A` `389128D54100` `cmp byte ptr [ecx+0041D528], dl`

`:00408140` `741C`
`je 0040815E`        // must jmp (ecx <0x10)

`:00408142` `41`
`inc ecx`

`:00408143` `83F910` `cmp ecx, 00000010`

`:00408146` `7CF2`
`jl 0040813A`

// Referenced by a (U)nconditional or (C)onditional Jump at Address:
// |:00408161(C)
// |

`:00408148` `32D2`
`xor dl, dl`

// Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
// |:00408173(C),
// :0040817A(U)
// |

`:0040814A` `8A1E`
`mov bl, byte ptr [esi]`
// BL = Code[2+2*EAX+1]

`:0040814C` `33C9`
`xor ecx, ecx`

// Referenced by a (U)nconditional or (C)onditional Jump at Address:
// |:0040815A(C)
// |

`:0040814E` `389928D54100`
`cmp byte ptr [ecx+0041D528], bl`

`:00408154` `7426`
`je 0040817C`      // must jmp (ecx<0x10)

`:00408156` `41`
`inc ecx`

`:00408157` `83F910`
`cmp ecx, 00000010`

`:0040815A` `7CF2`
`jl 0040814E`

`:0040815C` `EB3A`
`jmp 00408198`

// Referenced by a (U)nconditional or (C)onditional Jump at Address:
// |:00408140(C)
// |

`:0040815E` `83F910`
`cmp ecx, 00000010`

`:00408161` `7DE5`
`jge 00408148`

`:00408163`
`2BCD` `sub ecx, ebp`

`:00408165` `81C1803E0000` `add ecx, 00003E80`

`:0040816B` `8BD1`
`mov edx, ecx`

`:0040816D` `81E20F000080`
`and edx, 8000000F`
//  edx = (ecx - ebp + 0x3e80) & 0x0f

`:00408173` `79D5`
`jns 0040814A`

`:00408175` `4A`
`dec edx`

`:00408176` `83CAF0` `or edx, FFFFFFF0`

`:00408179` `42`
`inc edx`

`:0040817A` `EBCE`
`jmp 0040814A`

// Referenced by a (U)nconditional or (C)onditional Jump at Address:
// |:00408154(C)
// |

`:0040817C` `83F910`
`cmp ecx, 00000010`

`:0040817F` `7D17`
`jge 00408198`

`:00408181` `2BCD`
`sub ecx, ebp`

`:00408183` `81C1803E0000`
`add ecx, 00003E80`

`:00408189` `81E10F000080`
`and ecx, 8000000F`
// ecx = (ecx - ebp + 0x3e80) & 0x0f

`:0040818F` `7909`
`jns 0040819A`

`:00408191` `49`
`dec ecx`

`:00408192` `83C9F0`
`or ecx, FFFFFFF0`

`:00408195` `41`
`inc ecx`

`:00408196` `EB02`
`jmp 0040819A`

// Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
// |:0040815C(U), :0040817F(C)
// |

`:00408198` `32C9`
`xor cl, cl`

// Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
// |:0040818F(C), :00408196(U)
// |

`:0040819A` `8AD9`
`mov bl, cl`

`:0040819C` `83C602`
`add esi, 00000002`

`:0040819F` `C0E304`
`shl bl, 04`

`:004081A2` `02DA`
`add bl, dl`

`:004081A4` `8B9424E0000000`
`mov edx, dword ptr [esp+000000E0]`

`:004081AB` `0FBEC9`
`movsx ecx, cl`

`:004081AE` `881C17`
`mov byte ptr [edi+edx], bl`
// pCode[edx] = (CL << 4) + DL
// pCode must be "Clayman".

`:004081B1` `03E9`
`add ebp, ecx`
// important
// ebp += ecx

`:004081B3` `47`
`inc edi`

`:004081B4` `3BF8`
`cmp edi, eax`

`:004081B6` `0F8C79FFFFFF`
`jl 00408135`

`:004081BC` `8B742410`
`mov esi, dword ptr [esp+10]`

// Referenced by a (U)nconditional or (C)onditional Jump at Address:
// |:0040812B(C)
// |

`:004081C0` `8D542EFD`
`lea edx, dword ptr [esi+ebp-03]`

`:004081C4` `81E20F000080`
`and edx, 8000000F`
// edx =  (esi + ebp - 3) & 0x8000000f

`:004081CA` `7905`
`jns 004081D1`

`:004081CC` `4A`
`dec edx`

`:004081CD` `83CAF0`
`or edx, FFFFFFF0`

`:004081D0` `42`
`inc edx`

// Referenced by a (U)nconditional or (C)onditional Jump at Address:
// |:004081CA(C)
// |

`:004081D1` `8A4C3413`
`mov cl, byte ptr [esp+esi+13]`

`:004081D5`
`8A9A28D54100` `mov bl, byte ptr [edx+0041D528]`

`:004081DB` `3ACB`
`cmp cl, bl`

`:004081DD` `7432`
`je 00408211`     // must jmp
// sCode last char
// sCode[16] = str[edx]

// Possible StringData Ref from Data Obj ->"45#SDFG^\*&"

`:004081DF` `BF50D54100` `mov edi, 0041D550`

// Referenced by a (U)nconditional or (C)onditional Jump at Address:
// |:0040811A(U)
// |

`:004081E4` `83C9FF`
`or ecx, FFFFFFFF`

`:004081E7` `33C0`
`xor eax, eax`

`:004081E9` `F2`
`repnz`

`:004081EA` `AE`
`scasb`

`:004081EB` `F7D1`
`not ecx`

`:004081ED` `2BF9`
`sub edi, ecx`

`:004081EF` `8BD1`
`mov edx, ecx`

`:004081F1` `8BF7`
`mov esi, edi`

`:004081F3` `8BBC24E0000000`
`mov edi, dword ptr [esp+000000E0]`

`:004081FA` `C1E902`
`shr ecx, 02`

`:004081FD` `F3`
`repz`

`:004081FE` `A5`
`movsd`

`:004081FF` `8BCA`
`mov ecx, edx`

`:00408201` `83E103`
`and ecx, 00000003`

`:00408204` `F3`
`repz`

`:00408205` `A4`
`movsb`

`:00408206`
`5F` `pop edi`

`:00408207` `5E` `pop esi`

`:00408208` `5D`
`pop ebp`

`:00408209` `5B`
`pop ebx`

`:0040820A` `81C4CC000000`
`add esp, 000000CC`

`:00408210` `C3`
`ret`

// Referenced by a (U)nconditional or (C)onditional Jump at Address:
// |:004081DD(C)
// |

`:00408211` `8B8C24E0000000`
`mov ecx, dword ptr [esp+000000E0]`

`:00408218` `5F`
`pop edi`

`:00408219` `5E`
`pop esi`

`:0040821A` `5D`
`pop ebp`

`:0040821B` `C6040800`
`mov byte ptr [eax+ecx], 00`
// pCode[strlen(pCode)] = '\0'

`:0040821F` `5B`
`pop ebx`

`:00408220` `81C4CC000000`
`add esp, 000000CC`

`:00408226` `C3`
`ret`
