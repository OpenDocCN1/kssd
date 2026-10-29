
```python
from struct import *

import binascii

import base64

import string



Offst_xor_func = 0x00

Offst_func_size = 0x08

Offst_s_start_addr = 0x0c

Offst_s_len = 0x10

Offst_dst_pos = 0x14

Offst_xor_data_pos = 0x18

Offst_x_data = 0x20



fib_list = [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987, 1597, 2584, 4181, 6765, 10946, 17711, 28657, 46368, 75025, 121393, 196418, 317811, 514229, 832040, 1346269, 2178309, 3524578, 5702887, 9227465, 14930352, 24157817, 39088169, 63245986, 102334155, 165580141, 267914296, 433494437, 701408733, 1134903170, 1836311903, 2971215073, 4807526976, 7778742049, 12586269025, 20365011074, 32951280099, 53316291173, 86267571272, 139583862445, 225851433717, 365435296162, 591286729879, 956722026041, 1548008755920, 2504730781961, 4052739537881, 6557470319842, 10610209857723, 17167680177565, 27777890035288, 44945570212853, 72723460248141, 117669030460994, 190392490709135, 308061521170129, 498454011879264, 806515533049393, 1304969544928657, 2111485077978050, 3416454622906707, 5527939700884757, 8944394323791464, 14472334024676221, 23416728348467685, 37889062373143906, 61305790721611591, 99194853094755497, 160500643816367088, 259695496911122585, 420196140727489673, 679891637638612258, 1100087778366101931, 1779979416004714189, 2880067194370816120, 4660046610375530309, 7540113804746346429, 12200160415121876738, 1293530146158671551, 13493690561280548289, 14787220707439219840, 9834167195010216513, 6174643828739884737, 16008811023750101250, 3736710778780434371, 1298777728820984005, 5035488507601418376, 6334266236422402381, 11369754744023820757, 17704020980446223138, 10627031650760492279, 9884308557497163801, 2064596134548104464, 11948904692045268265, 14013500826593372729, 7515661444929089378, 3082418197812910491, 10598079642741999869, 13680497840554910360, 5831833409587358613, 1065587176432717357, 6897420586020075970, 7963007762452793327, 14860428348472869297, 4376692037216111008, 790376311979428689, 5167068349195539697, 5957444661174968386, 11124513010370508083, 17081957671545476469, 9759726608206432936, 8394940206042357789, 18154666814248790725, 8102862946581596898, 7810785687120836007]

orig_bs_table = [0x2A, 0x39, 0x5F, 0x64, 0xC2, 0xA7, 0x46, 0x23,
0x53, 0x6B, 0x74, 0x47, 0x28, 0x4D, 0x70, 0x42,
0x49, 0x25, 0x52, 0x6A, 0x62, 0x38, 0x40, 0x4A,
0x69, 0x45, 0x44, 0x59, 0x2D, 0x31, 0x24, 0x50,
0x67, 0x79, 0x54, 0x21, 0x4C, 0x76, 0x71, 0x66,
0x2B, 0x63, 0x68, 0x6D, 0x51, 0x57, 0x4F, 0x30,
0x65, 0x4E, 0x5A, 0x34, 0x75, 0x6E, 0x33, 0x6C,
0x37, 0x48, 0x26, 0x32, 0x77, 0x61, 0x7A, 0x4B]

for i in range(len(orig_bs_table)):
    orig_bs_table[i] = chr(orig_bs_table[i])



def dump_data_core(filename, data_addr, base_addr, size):
    fp= open(filename, "rb")
    offset = data_addr - base_addr
    fp.seek(offset,0)
    f = fp.read()
    type_format = '<'
    if size == 1:
        type_format += 'B'
    elif size == 2:
        type_format += 'H'
    elif size == 4:
        type_format += 'I'
    elif size == 8:
        type_format += 'Q'
    else:
        print "[warn]"
        type_format += 'x'
    target_data, = unpack(type_format, f[:size])
    return target_data



def dump_data_specific(filename, start_addr, base_addr, order, species, size, count):
    arr_data = []
    for i in range(count):
        data_addr = start_addr + 0x120*order + species + i*size
        arr_data.append(hex(dump_data_core(filename, data_addr, base_addr, size)))
    return arr_data



def explode_fib_num(dst):
    for i in range(len(fib_list)):
        if(dst == fib_list[i]):
            return i-1
    return "Error"



def spec_fib(key_1, key_pos, filename, start_addr, base_addr, order, species, size, count):
    res_arr = dump_data_specific(filename, start_addr, base_addr, order, species, size, count)
    print 'order:'+str(order)+' '+str(res_arr)
    for i in range(len(res_arr)):
        if(res_arr[i][-1] == 'L'):
            c = chr(explode_fib_num(int(res_arr[i][:-1] ,16)))
            key_1[key_pos+i] = c
        else:
            c = chr(explode_fib_num(int(res_arr[i],16)))
            key_1[key_pos+i] = c
    return key_1



def explode_crc32_c1(dst):
    if dst == 0x0:
        return ord('X')
    for i in range(31, 127):
        if(dst == binascii.crc32(chr(i)) &0xffffffff):
            return chr(i)
    return 'E1ROR'



def explode_crc32_c2(dst):
    if dst == 0x0:
        return ord('X')
    for x0 in range(31, 127):
        for x1 in range(31, 127):
            s = chr(x0) + chr(x1)
            if(binascii.crc32(s) &0xffffffff) == dst:
                return s
    return 'E2ROR'



def explode_crc32_c3(dst):
    if dst == 0x0:
        return ord('X')
    for x0 in range(31, 127):
        for x1 in range(31, 127):
            for x2 in range(31, 127):
                s = chr(x0) + chr(x1) + chr(x2)
                if(binascii.crc32(s) &0xffffffff) == dst:
                    return s
    return 'E3ROR'



def spec_crc32(key_1, key_pos, filename, start_addr, base_addr, order, species, size, count):
    res_arr = dump_data_specific(filename, start_addr, base_addr, order, species, size, 1)
    print 'order:'+str(order)+' '+str(res_arr)
    if count == 1:
        p = res_arr[0]
        if res_arr[0][-1] == 'L':
            p = res_arr[0][:-1]
        dec = explode_crc32_c1(int(p,16))
    elif count == 2:
        p = res_arr[0]
        if res_arr[0][-1] == 'L':
            p = res_arr[0][:-1]
        dec = explode_crc32_c2(int(p,16))
    elif count == 3:
        p = res_arr[0]
        if res_arr[0][-1] == 'L':
            p = res_arr[0][:-1]
        dec = explode_crc32_c3(int(p,16))
    else:
        print "Can not reach explode num"
        return "Fail"
    for i in range(count):
        key_1[key_pos+i] = dec[i]
    return key_1



def xor_swap(count):
    arr_mess = [0xDC, 0x48, 0x8F, 0x56, 0xBC, 0x36, 0xB0, 0x64, 0x40, 0x27, 0xE6, 0x2C, 0xD2, 0x3F, 0xC2, 0x34,
    0x5D, 0x52, 0xEE, 0xCD, 0xAA, 0xCA, 0x81, 0x8D, 0x71, 0x23, 0x28, 0xD7, 0x96, 0x4E, 0x7F, 0x6B,
    0xA1, 0x3E, 0xA3, 0x12, 0x91, 0x26, 0xE4, 0x03, 0x60, 0x8C, 0x01, 0x44, 0x79, 0xE3, 0x84, 0x35,
    0xB8, 0x4A, 0xC1, 0x55, 0x1A, 0x9D, 0x11, 0xE7, 0x92, 0xA4, 0xD4, 0x68, 0x37, 0x85, 0x62, 0x66,
    0xFC, 0xBF, 0xD8, 0x98, 0x9A, 0x8E, 0x32, 0x20, 0x16, 0x38, 0x57, 0x0E, 0x18, 0x5B, 0xF4, 0x17,
    0x1E, 0xA0, 0xDD, 0x53, 0x5F, 0x06, 0xF8, 0xF6, 0xE1, 0x0C, 0x02, 0x74, 0xF3, 0xC0, 0xC3, 0xF0,
    0x3C, 0x94, 0x10, 0xDB, 0x04, 0xF9, 0x08, 0xD6, 0x1F, 0xBE, 0xEF, 0x95, 0xE5, 0x50, 0xB6, 0xAB,
    0x5A, 0x19, 0xAD, 0x24, 0x99, 0x43, 0xCB, 0xA5, 0xEB, 0x39, 0xCC, 0x67, 0xB9, 0xC5, 0xC9, 0xA6,
    0x6A, 0x90, 0x7A, 0x0F, 0xD9, 0x3B, 0x1C, 0xE8, 0xA8, 0x7E, 0xC4, 0x72, 0x8B, 0x63, 0x1B, 0x59,
    0x07, 0x49, 0xAE, 0x05, 0xA9, 0xEC, 0x00, 0xC6, 0x31, 0x14, 0x69, 0xBA, 0x82, 0x1D, 0x65, 0x46,
    0x70, 0x29, 0xAF, 0xC8, 0xDE, 0xD0, 0xFD, 0x77, 0x73, 0x2B, 0x6D, 0xDF, 0x6C, 0xFE, 0x30, 0x2F,
    0xF1, 0x78, 0x21, 0x7C, 0x4D, 0xCE, 0x13, 0xB5, 0x97, 0x2A, 0x9E, 0xFF, 0xEA, 0xCF, 0x25, 0xBD,
    0xAC, 0x4C, 0x0D, 0x80, 0xE9, 0xB2, 0x4B, 0xFA, 0x54, 0x41, 0x89, 0x3A, 0x51, 0xF7, 0x00, 0x8A,
    0x2D, 0xDA, 0x33, 0xF2, 0x22, 0x9B, 0xD5, 0xA2, 0x45, 0x6E, 0x4F, 0xD3, 0x76, 0x3D, 0x86, 0x2E,
    0xBB, 0x7B, 0x0A, 0x42, 0xB1, 0x5E, 0xED, 0x9C, 0xE0, 0x88, 0x7D, 0x15, 0x93, 0xD1, 0x83, 0x6F,
    0xFB, 0xB4, 0x9F, 0x47, 0x09, 0xA7, 0x87, 0xF5, 0x61, 0xE2, 0xC7, 0xB3, 0x75, 0x0B, 0x58, 0x5C]
    add = 0
    res = []
    for i_ in range(count):
        i_ += 1
        add = (add +arr_mess[i_]) & 0xff
        arr_mess[i_] ^= arr_mess[add]
        arr_mess[add] ^= arr_mess[i_]
        arr_mess[i_] ^= arr_mess[add]
        pos = (arr_mess[i_] + arr_mess[add])&0xff
        res.append(arr_mess[pos])
    return res



def spec_xor_swap(key_1, key_pos, filename, start_addr, base_addr, order, species, size, count):
    res_arr = dump_data_specific(filename, start_addr, base_addr, order, species, size, count)
    print 'order:'+str(order)+' '+str(res_arr)
    dst = xor_swap(count)
    for i in range(len(res_arr)):
        if(res_arr[i][-1] == 'L'):
            key_1[key_pos+i] = chr(dst[i] ^ int(res_arr[i][:-1] ,16))
        else:
            key_1[key_pos+i] = chr(dst[i] ^ int(res_arr[i],16))
    return key_1



def dec_base64(dst, read_size):
    for i in range(read_size):
        if(dst[i] == '\x00'):
            dst[i] = '='
    dst = ''.join(dst)
    this_table = ''.join(orig_bs_table)
    orig_table = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
    transtab = string.maketrans(this_table, orig_table)
    dec_1 = dst.translate(transtab).decode('base64')
    return dec_1



def spec_base64(key_1, key_pos, filename, start_addr, base_addr, order, species, size, count):
    if(count % 3 !=0):
        read_size = (count/3+1)*4
    else:
        read_size = (count/3)*4
    res_arr = dump_data_specific(filename, start_addr, base_addr, order, species, size, read_size)
    print 'order:'+str(order)+' '+str(res_arr)
    dst = []
    for i in range(len(res_arr)):
        dst.append(chr(int(res_arr[i], 16)))
    dec_1 = dec_base64(dst, read_size)
    for i in range(len(dec_1)):
        key_1[key_pos+i] = dec_1[i]
    return key_1



def dec_value_shift(dst, count):
    for i in range(count):
        dst[i] = chr(dst[i]-13)
    return dst



def spec_value_shift(key_1, key_pos, filename, start_addr, base_addr, order, species, size, count):
    res_arr = dump_data_specific(filename, start_addr, base_addr, order, species, size, count)
    print 'order:'+str(order)+' '+str(res_arr)
    dst = []
    for i in range(len(res_arr)):
        dst.append((int(res_arr[i], 16)))
    dec_1 = dec_value_shift(dst, count)
    for i in range(count):
        key_1[key_pos+i] = dec_1[i]
    return key_1



def dec_value_equal(dst, count):
    for i in range(count):
        dst[i] = chr(dst[i])
    return dst



def spec_value_equal(key_1, key_pos, filename, start_addr, base_addr, order, species, size, count):
    res_arr = dump_data_specific(filename, start_addr, base_addr, order, species, size, count)
    print 'order:'+str(order)+' '+str(res_arr)
    dst = []
    for i in range(len(res_arr)):
        dst.append((int(res_arr[i], 16)))
    dec_1 = dec_value_equal(dst, count)
    for i in range(count):
        key_1[key_pos+i] = dec_1[i]
    return key_1



def dec_value_xor(dst, count):
    for i in range(count):
        dst[i] = chr(dst[i] ^ 0x2A)
    return dst



def spec_value_xor(key_1, key_pos, filename, start_addr, base_addr, order, species, size, count):
    res_arr = dump_data_specific(filename, start_addr, base_addr, order, species, size, count)
    print 'order:'+str(order)+' '+str(res_arr)
    dst = []
    for i in range(len(res_arr)):
        dst.append((int(res_arr[i], 16)))
    dec_1 = dec_value_xor(dst, count)
    for i in range(count):
        key_1[key_pos+i] = dec_1[i]
    return key_1



key_1 = ['x'] * 69

key_1 = spec_fib(key_1, 2, "magic", 0x605100, 0x600000, 0, Offst_x_data, 8, 3)
key_1 = spec_fib(key_1, 0x2c, "magic", 0x605100, 0x600000, 1, Offst_x_data, 8, 2)
key_1 = spec_crc32(key_1, 0x10, "magic", 0x605100, 0x600000, 2, Offst_x_data, 4, 1)
key_1 = spec_xor_swap(key_1, 0x7, "magic", 0x605100, 0x600000, 3, Offst_x_data, 1, 3)
key_1 = spec_base64(key_1, 0x3f, "magic", 0x605100, 0x600000, 4, Offst_x_data, 1, 1)
key_1 = spec_fib(key_1, 0x39, "magic", 0x605100, 0x600000, 5, Offst_x_data, 8, 3)
key_1 = spec_fib(key_1, 0x30, "magic", 0x605100, 0x600000, 6, Offst_x_data, 8, 2)
key_1 = spec_value_shift(key_1, 0x1e, "magic", 0x605100, 0x600000, 7, Offst_x_data, 1, 3)
key_1 = spec_value_equal(key_1, 0x0a, "magic", 0x605100, 0x600000, 8, Offst_x_data, 1, 1)
key_1 = spec_value_equal(key_1, 0x3c, "magic", 0x605100, 0x600000, 9, Offst_x_data, 1, 3)
key_1 = spec_value_shift(key_1, 0x0d, "magic", 0x605100, 0x600000, 10, Offst_x_data, 1, 1)
key_1 = spec_fib(key_1, 0x2b, "magic", 0x605100, 0x600000, 11, Offst_x_data, 8, 1)
key_1 = spec_value_shift(key_1, 0x38, "magic", 0x605100, 0x600000, 12, Offst_x_data, 1, 1)
key_1 = spec_value_xor(key_1, 0x12, "magic", 0x605100, 0x600000, 13, Offst_x_data, 1, 2)
key_1 = spec_value_equal(key_1, 0x11, "magic", 0x605100, 0x600000, 14, Offst_x_data, 1, 1)
key_1 = spec_crc32(key_1, 0x42, "magic", 0x605100, 0x600000, 15, Offst_x_data, 4, 3)
key_1 = spec_value_equal(key_1, 0x28, "magic", 0x605100, 0x600000, 16, Offst_x_data, 1, 3)
key_1 = spec_value_xor(key_1, 0x2e, "magic", 0x605100, 0x600000, 17, Offst_x_data, 1, 2)
key_1 = spec_value_shift(key_1, 0x0, "magic", 0x605100, 0x600000, 18, Offst_x_data, 1, 2)
key_1 = spec_xor_swap(key_1, 0xB, "magic", 0x605100, 0x600000, 19, Offst_x_data, 1, 2)
key_1 = spec_value_xor(key_1, 0x26, "magic", 0x605100, 0x600000, 20, Offst_x_data, 1, 2)
key_1 = spec_value_xor(key_1, 0x14, "magic", 0x605100, 0x600000, 21, Offst_x_data, 1, 2)
key_1 = spec_base64(key_1, 0x32, "magic", 0x605100, 0x600000, 22, Offst_x_data, 1, 3)
key_1 = spec_value_shift(key_1, 0x24, "magic", 0x605100, 0x600000, 23, Offst_x_data, 1, 2)
key_1 = spec_value_equal(key_1, 0x1c, "magic", 0x605100, 0x600000, 24, Offst_x_data, 1, 2)
key_1 = spec_value_equal(key_1, 0x2f, "magic", 0x605100, 0x600000, 25, Offst_x_data, 1, 1)
key_1 = spec_value_xor(key_1, 0x05, "magic", 0x605100, 0x600000, 26, Offst_x_data, 1, 2)
key_1 = spec_value_xor(key_1, 0x35, "magic", 0x605100, 0x600000, 27, Offst_x_data, 1, 3)
key_1 = spec_fib(key_1, 0x40, "magic", 0x605100, 0x600000, 28, Offst_x_data, 8, 2)
key_1 = spec_value_xor(key_1, 0x16, "magic", 0x605100, 0x600000, 29, Offst_x_data, 1, 3)
key_1 = spec_fib(key_1, 0x0e, "magic", 0x605100, 0x600000, 30, Offst_x_data, 8, 2)
key_1 = spec_value_shift(key_1, 0x19, "magic", 0x605100, 0x600000, 31, Offst_x_data, 1, 3)
key_1 = spec_value_xor(key_1, 0x21, "magic", 0x605100, 0x600000, 32, Offst_x_data, 1, 3)

print ''.join(key_1)
```
