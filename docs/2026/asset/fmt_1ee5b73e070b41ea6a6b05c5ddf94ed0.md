
# coding: utf-8

# ====================================================== #

#                                                        #

#      FLOKIBOT BOT32 DEOBFUSCATION IDA SCRIPT           #

#                                                        #

#       ef1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4V1k6h3I4E0j5i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3k6X3I4G2K9$3W2T1L8%4c8Q4x3X3g2H3K9s2m8Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6t1K6

#                                                        #

# ====================================================== #

# IDAPython script to deobfuscate statically the bot32 payload of the banking malware FlokiBot. 
# Imports are fully resolved, hooks are identified and named and strings are decrypted and added in comments, without using any debugger. 
# May take a few minutes to resolve imports. 
# Works with FlokiBot dropper with some small changes.

import sys

# sys.path.append("/usr/local/lib/python2.7/dist-packages")
# idaapi.enable_extlang_python(True)

import pefile

# RunPlugin("python", 3)

`CRC_POLY` 	= 0xEDB88320	# Depending on sample
`XOR_KEY` 	= 0x34ED 	# Depending on sample
`ARRAY_ADDR`	= 0x41B350 	# Depending on sample
`ARRAY_ITER` 	= 12 		# Size of a triplet (3*sizeof(DWORD))

i = 0

# ----------------------------------------------------
# Generating CRC polynoms
# ----------------------------------------------------

poly = []

while i < 256:
	size = 8
	b = i
	while size != 0:
		if b & 1:
			b = (b >> 1) ^ `CRC_POLY`
		else:
			b >>= 1
		size -= 1
	poly.insert(i, b)
	i += 1

# ----------------------------------------------------
# FlokiBot CRC32
# ----------------------------------------------------

def `crc32`(name):
	name_len = len(name)
	i = 0
	crc = 0xFFFFFFFF
	while i < name_len:
		crc = poly[(crc ^ ord(name[i])) & 0xFF] ^ (crc >> 8)
		i += 1
	crc = (~crc) & 0xFFFFFFFF
	return crc

# ----------------------------------------------------
# DEOBFUSCATING API CALLS
# ----------------------------------------------------

array_dll = ['ntdll', 'kernel32', 'wininet', 'ws2_32', 'advapi32', 'secur32', 'crypt32', 
			'shlwapi', 'ole32', 'gdi32', 'shell32', 'user32', 'urlmon' #, 'nss3', 'nspr4', 'chrome'
			]

dll_hash = {}

for dll in array_dll:
	h = `crc32`(dll + '.dll') ^ `XOR_KEY`
 	dll_hash[h] = dll
	print "[+] %s.dll (%X) : Parsing..." % (dll, h)
	pe = pefile.PE("C:\\Windows\\System32\\" + dll + ".dll")
	api_hash = {}
	pe.parse_data_directories()
	for exp in pe.DIRECTORY_ENTRY_EXPORT.symbols:
		if exp.name:
			api_crc = `crc32`(exp.name) ^ `XOR_KEY`
			api_hash[api_crc] = exp.name
	nb = 0
	for i in range(0, 287):
		ea_name = (`ARRAY_ADDR` + i*`ARRAY_ITER`)
		ea_func = Dword(ea_name)
		ea_crc = ea_name + 4
		MakeDword(ea_crc)
		crc = Dword(ea_crc)
		if crc in api_hash:
			if MakeName(ea_func, api_hash[crc]+"_wrap&quot![](https://bbs.kanxue.com/view/img/face/41.gif)
				nb += 1
	print "[+] %s : Resolved %d API names" % (dll, nb)

# ----------------------------------------------------
# PARSING HOOK STRUCT
# ----------------------------------------------------

sid = AddStruc(-1, 'HOOKWINAPI')
AddStrucMember(sid, 'functionForHook', 0, FF_DWRD|FF_DATA, -1, 4)
AddStrucMember(sid, 'hookerFunction', 4, FF_DWRD|FF_DATA, -1, 4)
AddStrucMember(sid, 'originalFunction', 8, FF_DWRD|FF_DATA, -1, 4)
AddStrucMember(sid, 'originalFunctionSize', 12, FF_DWRD|FF_DATA, -1, 4)
AddStrucMember(sid, 'dllHash', 16, FF_DWRD|FF_DATA, -1, 4)
AddStrucMember(sid, 'apiHash', 20, FF_DWRD|FF_DATA, -1, 4)

`HOOKWINAPI_EA` = 0x41B000
`HOOKWINAPI_SIZE` = 0x18
ea = `HOOKWINAPI_EA`
MakeName(`HOOKWINAPI_EA`, "hookWinApi_array")
print "Parsing hook table @ 0x%X" % `HOOKWINAPI_EA`
for i in range(0, 25):
	for field in range(0, 6):
		MakeDword(ea+4*field)
	fn_name = Name(Dword(ea))
	hook_ea = Dword(ea+4)
	MakeName(hook_ea, "hook_" + fn_name)
	hook_name = Name(Dword(ea+4))
	ori_ea = ea+8
	MakeName(ori_ea, "ori_" + fn_name)
	print "[+] Hook on %s \t--> %s" % (fn_name, hook_name)
	ea += `HOOKWINAPI_SIZE`

# ----------------------------------------------------
# STRING DEOBFUSCATION
# ----------------------------------------------------

`DECRYPT_FN_EA` = 0x403948			# Depending on sample
`ENCRYPTED_STRINGS_EA` = 0x402278 		# Depending on sample
`DECRYPT_FN` = "decrypt_string"
`ENCRYPTED_STRINGS` = "encrypted_strings"
`ARRAY_SIZE` = 0x77 				# Depending on sample
decrypted_strings = {}

def backwardSearch(ea, instr):
	while True:
		ea = PrevHead(ea)
		if GetMnem(ea) == instr:
			return ea

def `decrypt_string`(index, ea_encrypted):
	string = ""
	if index == -1:
		string = "Invalid index"
		return string
	encr_array = LocByName(ea_encrypted)
	if encr_array == 0xFFFFFFFF:
		string = "Invalid array for encrypted strings"
		return string
	ea_item = encr_array + index*2*4
	xor_k = Byte(ea_item)
	size = Word(ea_item+2)
	ptr_string = Dword(ea_item + 4)
	MakeByte(ptr_string)
	MakeArray(ptr_string, size)
	i = 0
	if size <= 0:
		string = "Size <= 0"
		return string
	while i < size:
		ichr = i
		string += str(unichr((i ^ xor_k ^ Byte(ptr_string + i)) & 0xFF))
		i += 1
	MakeComm(ptr_string, string)
	return string

# ----------------------------------------------------
# Decrypting and commenting whole array
# ----------------------------------------------------

MakeName(`DECRYPT_FN_EA`, `DECRYPT_FN`) 
MakeName(`ENCRYPTED_STRINGS_EA`, `ENCRYPTED_STRINGS`)

i = 0
loc = LocByName(`DECRYPT_FN`)
for ea in range(loc, loc+`ARRAY_SIZE`):
	decrypted_strings[i] = `decrypt_string`(i, `ENCRYPTED_STRINGS`)
	i += 1

print "[+] Decrypted %d strings :" % (i)
print decrypted_strings

# ----------------------------------------------------
# Commenting calls to decryption function with decrypted strings
# ----------------------------------------------------

i = 0
for xref in XrefsTo(LocByName(`DECRYPT_FN`)):
	ea = xref.frm
	mnem = GetMnem(PrevHead(ea))
	index = 0
	if mnem == "xor":
		index = 0
	elif mnem == "pop":
		ea = backwardSearch(ea, "push")
		index = GetOperandValue(ea, 0)
	elif mnem == "inc":
		index = 1
	elif mnem == "mov":
		index = GetOperandValue(ea, 1)
	if index in decrypted_strings:
		MakeComm(xref.frm, decrypted_strings[index])
		i += 1

print "[+] Commented %d strings with decrypted values" % (i)
print "[+] Script is done."

[flokibot32_deobf.py](https://bbs.kanxue.com/elink@e3eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3j5h3c8W2L8r3#2S2M7#2)9J5c8U0S2U0z5o6j5@1x3K6p5#2y4U0b7^5j5e0t1I4k6r3c8S2j5X3u0V1y4X3u0U0y4$3f1H3j5U0j5@1x3e0p5&6i4K6t1K6k6X3W2D9k6g2)9J5k6r3k6D9L8$3E0A6j5X3!0@1x3K6u0Q4y4h3k6V1k6h3!0T1k6W2)9J5k6s2m8&6)

## 持久性

bot用一个伪随机名字把自己复制到 `C:\Documents and Settings\[username]\Application Data` 并通过在Windows的启动文件夹创建一个.lnk来获得持久性。

```c
int startup_lnk() {

  int v0; // edi@1

  _WORD *v1; // ecx@1

  int v2; // eax@2

  _WORD *v3; // ecx@2

  const void *v4; // eax@2

  const void *v5; // esi@3

  int strStartupFolder; // [sp+8h] [bp-20Ch]@1

  int v8; // [sp+210h] [bp-4h]@6

  v0 = 0;

  SHGetFolderPathW_wrap(0, 7, 0, 0, &strStartupFolder); // 7 = CSIDL_STARTUP

  v1 = (_WORD *)PathFindFileNameW_wrap(&pFilename);

  if ( v1 && (v2 = cstm_strlen(v1), sub_40FECB(v2 - 4, v3), v4) )
    v5 = v4;
  else
    v5 = 0;

  if ( v5 ) {
    v8 = 0;
    if ( build_lnk((int)&v8, (const char *)L"%s\\%s.lnk", &strStartupFolder, v5) > 0 )
      v0 = v8;
    cstm_FreeHeap(v5);
  }

  return v0;

}
```

挂钩API

## 概述

基于ZeuS，FlokiBot用了同一种但又有些许不同的结构数组来存储它的钩子：
