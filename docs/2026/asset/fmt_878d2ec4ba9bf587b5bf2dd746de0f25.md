
```
def huffman_decompress(module_contents, compressed_size, decompressed_size, print_msg):

    chunk_count = int(decompressed_size / CHUNK_SIZE)

    header_size = chunk_count * 0x4



    module_buffer = bytearray(module_contents)

    header_buffer = module_buffer[0:header_size]

    compressed_buffer = module_buffer[header_size:compressed_size]

    decompressed_array = []



    header_entries = struct.unpack('<{:d}I'.format(chunk_count), header_buffer)

    start_offsets, flags = zip(*[(x & 0x1FFFFFF, (x >> 25) & 0x7F) for x in header_entries])

    end_offsets = itertools.chain(start_offsets[1:], [compressed_size - header_size])



    for index, dictionary_type, compressed_position, compressed_limit in zip(range(chunk_count), flags, start_offsets, end_offsets):

        if print_msg == 'all' :

            print('==Processing chunk 0x{:X} at compressed offset 0x{:X} with dictionary 0x{:X}=='.format(index, compressed_position, dictionary_type))



        dictionary = HUFFMAN_SYMBOLS[dictionary_type]



        decompressed_position, decompressed_limit = index * CHUNK_SIZE, (index + 1) * CHUNK_SIZE



        bit_buffer = 0

        available_bits = 0



        while decompressed_position < decompressed_limit:

            while available_bits <= 24 and compressed_position < compressed_limit:

                bit_buffer = bit_buffer | compressed_buffer[compressed_position] << (24 - available_bits)

                compressed_position = compressed_position + 1

                available_bits = available_bits + 8



            codeword_length, base_codeword = 0, 0

            for length, shape, base in HUFFMAN_SHAPE:

                if bit_buffer >= shape:

                    codeword_length, base_codeword = length, base

                    break



            if available_bits >= codeword_length:

                codeword = bit_buffer >> (32 - codeword_length)

                bit_buffer = (bit_buffer << codeword_length) & 0xFFFFFFFF

                available_bits = available_bits - codeword_length



                symbol = dictionary[codeword_length][base_codeword - codeword]

                symbol_length = len(symbol)



                if decompressed_limit - decompressed_position >= symbol_length:

                    decompressed_array.extend(symbol)

                    decompressed_position += symbol_length

                else:

                    # Shouldn't ever happen now that the dictionaries are complete

                    if print_msg in ['all','error'] :

                        print('Skipping overflowing codeword {: <15s} (dictionary 0x{:X}, codeword length {: >2d}, codeword {: >5s}, symbol length {:d}) at decompressed offset 0x{:X}'.format(

                        ('{:0>' + str(codeword_length) + 'b}').format(codeword), dictionary_type, codeword_length, '0x{:X}'.format(codeword), symbol_length, decompressed_position))

                    filler = itertools.repeat(0x7F, decompressed_limit - decompressed_position)

                    decompressed_array.extend(filler)

                    decompressed_position = decompressed_limit

            else:

                # Shouldn't ever happen now that the dictionaries are complete

                if print_msg in ['all','error'] :

                    print('Reached end of compressed stream early at decompressed offset 0x{:X}'.format(decompressed_position))

                filler = itertools.repeat(0x7F, decompressed_limit - decompressed_position)

                decompressed_array.extend(filler)

                decompressed_position = decompressed_limit

    

    return bytes(decompressed_array)



def main():

    if len(sys.argv) in [5, 6] :

        with open(sys.argv[1], 'rb') as file_input:

            decompressed = huffman_decompress(file_input.read(), int(sys.argv[3], 0), int(sys.argv[4], 0), (sys.argv[5] if len(sys.argv) == 6 else 'all'))

        with open(sys.argv[2], 'wb') as file_output:

            file_output.write(decompressed)

    else :

        print('\nUsage: HuffmanDecompress <compressed file path> <decompressed file path> <compressed file size> <decompressed file size> <info all|error|none>')

        print('\nExample: HuffmanDecompress bup.huff bup.mod 0x28E00 0x31000 all')



if __name__ == "__main__":

    main()
```

字典参考：`e54K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3N6s2u0W2M7$3g2S2M7X3y4Z5i4K6u0r3N6h3&6y4c8e0p5I4`这个项目。

## 项目进展

这个项目实际上研究时间大概在一个月，并且中间烧掉了两块主板，包括文中写的这个`8700k`平台以及废掉了。写文章的时候是两周前，最近才把剩下的坑填完。也是在不停的价钱在咸鱼收集符合版本的设备。

**感谢您看到这里了，如果你看到这里觉得好玩，或者对你有帮助请点个赞，有啥建议或者想说的可以留言，文章不做任何限制要求。**

### 剧透

已经对ME的启动流程进行完整分析。从`rbe` -> `bup`.以及kaokao 所说的漏洞的定位（分析主要依靠我构建的ai并行逆向框架与手工校对，感兴趣可以看一下我其他的文章，感谢感谢）。

![图片描述](https://bbs.kanxue.com/upload/attach/202605/992284_N4H34QKP6Y6T8PW.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202605/992284_7BYFTZ9GHQMX3NX.webp)

其实酷睿也存在一模一样的洞hhh，不仅仅是低功耗平台。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
