
```
@@ -0,0 +1,158 @@

print("hello world!")



local debug_level = {

    DISABLED = 0,

    LEVEL_1  = 1,

    LEVEL_2  = 2

}

local DEBUG = debug_level.LEVEL_1



local dprint = function() end

local dprint2 = function() end

local function resetDebugLevel()

    current_debug_level = 2

    if current_debug_level > debug_level.DISABLED then

        dprint = function(...)

            info(table.concat({"Lua: ", ...}," "))

        end



        if current_debug_level > debug_level.LEVEL_1 then

            dprint2 = dprint

        end

    else

        dprint = function() end

        dprint2 = dprint

    end

end

-- call it now

resetDebugLevel()





local bmw_proto = Proto("bmw", "BMW BCL")



local hdr_fields =

{

    val1 = ProtoField.uint16 ("bmw.val1", "Val1", base.HEX),

    val2 = ProtoField.uint16 ("bmw.val2", "Val2", base.HEX),

    val3 = ProtoField.uint16 ("bmw.val3", "Val3", base.HEX),

    len = ProtoField.uint16 ("bmw.len", "Length", base.DEC)

}

bmw_proto.fields = hdr_fields

dprint2("bmw_proto ProtoFields registered")



local dissect_data = Dissector.get("data")



function bmw_proto.init()

end



local BMW_MSG_HDR_LEN = 8



-- mention future helper methods

local check_bmw_length



function bmw_proto.dissector(tvbuf, pktinfo, root)

    dprint2("bmw_proto.dissector called")



    -- check packet length

    local pktlen = tvbuf:len()

    local bytes_consumed = 0



    while bytes_consumed < pktlen do

        -- call dissect_packet for this single packet

        -- it will return a positive number for the amount of bytes consumed

        -- or a negative number for a request for more bytes

        -- or 0 for an error

        local result = dissect_packet(tvbuf, pktinfo, root, bytes_consumed)



        if result > 0 then

            -- successfully parsed packet

            bytes_consumed = bytes_consumed + result

        elseif result == 0 then

            -- not a valid packet

            return 0

        else

            -- need more data to finish parsing

            pktinfo.desegment_offset = bytes_consumed

            pktinfo.desegment_len = 0 - result

            return pktlen

        end

    end

    return bytes_consumed

end



local function heuristic(tvbuf, pktinfo, root)

    ETCH_MAGIC = ByteArray.new("de ad be ef")



    if tvbuf:len() < BMW_MSG_HDR_LEN + 4 then

        return false

    end



    local etch_magic = tvbuf:range(BMW_MSG_HDR_LEN, 4):bytes()

    return etch_magic == ETCH_MAGIC

end



function dissect_packet(tvbuf, pktinfo, root, offset)

    dprint2("dissect_packet function called")

    local length_val, length_tvbf = check_bmw_length(tvbuf, offset)



    if length_val <= 0 then

        -- not enough data to get the header

        return length_val

    end



    -- update the packet list info

    pktinfo.cols.protocol:set("BMW BCL")

    if string.find(tostring(pktinfo.cols.info), "^BMW") == nil then

        pktinfo.cols.info:set("BMW BCL")

    end



    -- create the protocol tree field

    local tree = root:add(bmw_proto, tvbuf:range(offset, BMW_MSG_HDR_LEN + length_val))



    -- get the vals

    tree:add(hdr_fields.val1, tvbuf:range(offset, 2))

    tree:add(hdr_fields.val2, tvbuf:range(offset+2, 2))

    tree:add(hdr_fields.val3, tvbuf:range(offset+4, 2))

    tree:add(hdr_fields.len, length_tvbf)



    remaining_tvb = tvbuf(offset + BMW_MSG_HDR_LEN, length_val):tvb()

    dissect_data:call(remaining_tvb, pktinfo, root)



    return BMW_MSG_HDR_LEN + length_val

end



function check_bmw_length(tvbuf, offset)

    -- remaining bytes in the packet to look through

    local msglen = tvbuf:len() - offset



    -- check if capture was only capturing partial packet size

    if msglen ~= tvbuf:reported_length_remaining(offset) then

        -- captured packets are being sliced/cut-off, so don't try to desegment/reassemble

        dprint2("Captured packet was shorter than original, can't reassemble")

        return 0

    end



    if msglen < BMW_MSG_HDR_LEN then

        -- we need more bytes to parse the header

        dprint2("Need more bytes to look at the header")

        return -DESEGMENT_ONE_MORE_SEGMENT

    end



    -- we have enough to parse the length from the header

    local length_tvbr = tvbuf:range(offset+6, 2)

    local length_val = length_tvbr:uint()



    -- check if we have the whole packet somewhere

    if msglen < BMW_MSG_HDR_LEN + length_val then

        dprint2("Need more bytes to desegment full packet")

        return -(BMW_MSG_HDR_LEN + length_val - msglen)

    end

    return length_val, length_tvbr

end



function enable_dissector()

    DissectorTable.get("btrfcomm.dlci"):add_for_decode_as(bmw_proto)

end

enable_dissector()



--bmw_proto:register_heuristic("btspp", heuristic)

```
似乎该协议用于将连接多路复用到单个蓝牙串行套接字，而`Val2`是不同的连接ID。在Wireshark中解析了该字段之后，我可以使用显示过滤器来遵循单个的通信流程。

### 0x05 Apache Etch

经过一番研究，我发现[了一篇文章，](https://bbs.kanxue.com/elink@31fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1L8i4N6Q4x3X3c8U0j5i4u0A6N6q4)9J5k6h3c8W2i4K6u0r3L8%4m8W2L8W2)9J5k6s2y4G2N6i4u0U0k6g2)9J5c8X3g2@1j5$3S2Q4x3X3g2H3K9s2l9`.)解释了宝马将Apache Etch用作“用于BMW Apps的基本通信协议”。速浏览[Apache Etch文档可](https://bbs.kanxue.com/elink@8cfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6r3y4Z5i4K6u0W2j5i4m8S2j5$3S2W2i4K6u0W2L8%4u0Y4i4K6u0r3k6r3!0U0N6h3#2W2L8Y4c8S2N6r3W2G2L8W2)9J5k6h3S2@1L8h3H3`.)确认这`0xDEADBEEF`是每个Etch RPC调用开始时的魔术标识符。因此，这意味着我只需要将每个BCL流解码为Apache Etch数据包，Wireshark的内置Etch解析就会解析数据！

### 0x06 Apache Etch符号名

除了Apache Etch，每个函数名称和任何其他符号名称都编译为32位哈希值。Wireshark可以使用Etch调试组件将哈希值替换为适合的名称，但是我首先需要弄清楚这些名称并自己对它们进行哈希处理。该[散列算法](https://bbs.kanxue.com/elink@379K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6X3&6Q4x3X3g2S2M7r3q4U0K9r3g2Q4x3X3g2G2M7X3N6Q4x3V1k6J5k6i4m8G2M7#2)9J5c8X3q4K6k6W2)9J5c8X3g2@1j5$3S2Q4x3V1k6@1M7Y4g2F1K9#2)9J5c8Y4g2@1K9h3I4Q4x3V1k6K6M7X3y4Q4x3V1k6E0j5h3W2F1i4K6u0r3K9X3q4$3j5g2)9J5c8X3!0J5k6#2)9J5c8X3q4H3j5h3y4Z5k6g2)9J5c8X3g2@1j5$3S2Q4x3V1k6#2N6r3W2D9i4K6u0r3d9r3q4K6K9q4)9J5k6h3A6S2N6X3p5`.)是公开的，所以我写了[如下代码](https://bbs.kanxue.com/elink@f81K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5N6h3k6E0j5h3&6Q4x3V1k6W2N6r3y4Z5i4K6u0V1K9r3q4K6K9q4)9J5k6s2u0K6)，以帮助手动生成此调试名。
