
```
\>>>sdis.dis("MyWindow.get\_reg\_no\_true")
\*\*\*\*\*\*\*\*   byte code for MyWindow.get\_reg\_no\_true  \*\*\*\*\*\*\*\*
2117           0 SETUP\_EXCEPT           638 (to 641)

2118           3 SETUP\_EXCEPT           173 (to 179)

2119           6 LOAD\_CONST               1 ('')
             9 STORE\_FAST               9 (mac\_address)

2120          12 LOAD\_CONST               2 ('.')
            15 STORE\_FAST               6 (strComputer)

2121          18 LOAD\_GLOBAL              2 (win32com)
            21 LOAD\_ATTR                3 (client)
            24 LOAD\_ATTR                4 (Dispatch)
            27 LOAD\_CONST               3 ('WbemScripting.SWbemLocator')
            30 CALL\_FUNCTION            1
            33 STORE\_FAST               3 (objWMIService)

2122          36 LOAD\_FAST               3 (objWMIService)
            39 LOAD\_ATTR                6 (ConnectServer)
            42 LOAD\_FAST               6 (strComputer)
            45 LOAD\_CONST               4 ('root\\cimv2')
            48 CALL\_FUNCTION            2
            51 STORE\_FAST               8 (objSWbemServices)

2123          54 LOAD\_FAST               8 (objSWbemServices)
            57 LOAD\_ATTR                8 (ExecQuery)
            60 LOAD\_CONST               5 ('Select \* from Win32\_NetworkAdapter')
            63 CALL\_FUNCTION            1
            66 STORE\_FAST              10 (colItems)

2124          69 SETUP\_LOOP              80 (to 152)
             72 LOAD\_FAST              10 (colItems)
             75 GET\_ITER
        >>   76 FOR\_ITER               72 (to 151)
             79 STORE\_FAST              4 (objItem)

2125          82 LOAD\_FAST               4 (objItem)
             85 LOAD\_ATTR              11 (MACAddress)
             88 LOAD\_CONST              0 (None)
             91 COMPARE\_OP              3 (!=)
             94 JUMP\_IF\_FALSE          50 (to 147)
             97 POP\_TOP

2142          98 LOAD\_CONST               6 ('VEN\_')
            101 LOAD\_FAST               4 (objItem)
            104 LOAD\_ATTR              13 (PNPDeviceID)
            107 COMPARE\_OP              6 (in)
            110 JUMP\_IF\_FALSE          30 (to 143)
            113 POP\_TOP
            114 LOAD\_CONST               7 ('DEV\_')
            117 LOAD\_FAST               4 (objItem)
            120 LOAD\_ATTR              13 (PNPDeviceID)
            123 COMPARE\_OP              6 (in)
            126 JUMP\_IF\_FALSE          14 (to 143)
            129 POP\_TOP

2143         130 LOAD\_FAST               4 (objItem)
            133 LOAD\_ATTR              11 (MACAddress)
            136 STORE\_FAST              9 (mac\_address)

2145         139 BREAK\_LOOP
            140 JUMP\_ABSOLUTE          148
        >>  143 POP\_TOP
            144 JUMP\_ABSOLUTE           76
        >>  147 POP\_TOP
        >>  148 JUMP\_ABSOLUTE           76
        >>  151 POP\_BLOCK

2146     >>  152 LOAD\_FAST               9 (mac\_address)
            155 LOAD\_CONST               1 ('')
            158 COMPARE\_OP              2 (==)
            161 JUMP\_IF\_FALSE          10 (to 174)
            164 POP\_TOP

2147         165 LOAD\_CONST               8 ('00:0A:EB:F5:D4:14')
            168 STORE\_FAST              9 (mac\_address)
            171 JUMP\_FORWARD            1 (to 175)
        >>  174 POP\_TOP
        >>  175 POP\_BLOCK
            176 JUMP\_FORWARD           13 (to 192)

2148     >>  179 POP\_TOP
            180 POP\_TOP
            181 POP\_TOP

2149         182 LOAD\_CONST               8 ('00:0A:EB:F5:D4:14')
            185 STORE\_FAST              9 (mac\_address)
            188 JUMP\_FORWARD            1 (to 192)
            191 END\_FINALLY

2150     >>  192 SETUP\_EXCEPT             68 (to 263)

2151         195 LOAD\_GLOBAL             14 (md5)
            198 LOAD\_ATTR               15 (new)
            201 LOAD\_FAST               9 (mac\_address)
            204 CALL\_FUNCTION            1
            207 LOAD\_ATTR               16 (hexdigest)
            210 CALL\_FUNCTION            0
            213 STORE\_FAST              7 (s)

2152         216 LOAD\_GLOBAL             14 (md5)
            219 LOAD\_ATTR               15 (new)
            222 LOAD\_FAST               7 (s)
            225 LOAD\_CONST              9 (16)
            228 SLICE+1
            229 LOAD\_FAST               7 (s)
            232 LOAD\_CONST              9 (16)
            235 SLICE+2
            236 BINARY\_ADD
            237 CALL\_FUNCTION            1
            240 LOAD\_ATTR               16 (hexdigest)
            243 CALL\_FUNCTION            0
            246 LOAD\_CONST             10 (6)
            249 LOAD\_CONST             11 (-6)
            252 SLICE+3
            253 LOAD\_FAST               0 (self)
            256 STORE\_ATTR             19 (regno)
            259 POP\_BLOCK
            260 JUMP\_FORWARD            16 (to 279)

2153     >>  263 POP\_TOP
            264 POP\_TOP
            265 POP\_TOP

2154         266 LOAD\_CONST             12 ('00e6e95aff213b8e40ff')
            269 LOAD\_FAST               0 (self)
            272 STORE\_ATTR             19 (regno)

2155         275 JUMP\_FORWARD            1 (to 279)
            278 END\_FINALLY

2156     >>  279 SETUP\_EXCEPT           323 (to 605)

2157         282 LOAD\_CONST               1 ('')
            285 STORE\_FAST              2 (tmp\_result)

2158         288 LOAD\_FAST               0 (self)
            291 LOAD\_ATTR              21 (conf\_user)
            294 LOAD\_ATTR              22 (config)
            297 LOAD\_ATTR              23 (regnum)
            300 LOAD\_FAST               0 (self)
            303 STORE\_ATTR             24 (regno2)

2159         306 LOAD\_GLOBAL             14 (md5)
            309 LOAD\_ATTR               15 (new)
            312 LOAD\_FAST               0 (self)
            315 LOAD\_ATTR               19 (regno)
            318 CALL\_FUNCTION            1
            321 LOAD\_ATTR               16 (hexdigest)
            324 CALL\_FUNCTION            0
            327 STORE\_FAST              7 (s)

2160         330 LOAD\_GLOBAL             14 (md5)
            333 LOAD\_ATTR               15 (new)
            336 LOAD\_FAST               7 (s)
            339 LOAD\_CONST             13 (14)
            342 SLICE+1
            343 LOAD\_CONST             14 ('bb2')
            346 BINARY\_ADD
            347 LOAD\_FAST               7 (s)
            350 LOAD\_CONST             13 (14)
            353 SLICE+2
            354 BINARY\_ADD
            355 CALL\_FUNCTION            1
            358 LOAD\_ATTR               16 (hexdigest)
            361 CALL\_FUNCTION            0
            364 STORE\_FAST              1 (md5\_tmp)

2161         367 LOAD\_FAST               1 (md5\_tmp)
            370 LOAD\_CONST             15 (28)
            373 SLICE+1
            374 LOAD\_FAST               1 (md5\_tmp)
            377 LOAD\_CONST              9 (16)
            380 LOAD\_CONST             16 (20)
            383 SLICE+3
            384 BINARY\_ADD
            385 LOAD\_FAST               1 (md5\_tmp)
            388 LOAD\_CONST              18 (0)
            391 LOAD\_CONST              18 (5)
            394 SLICE+3
            395 BINARY\_ADD
            396 LOAD\_FAST               1 (md5\_tmp)
            399 LOAD\_CONST              19 (7)
            402 LOAD\_CONST              20 (9)
            405 SLICE+3
            406 BINARY\_ADD
            407 LOAD\_FAST               1 (md5\_tmp)
            410 LOAD\_CONST              18 (5)
            413 LOAD\_CONST              19 (7)
            416 SLICE+3
            417 BINARY\_ADD
            418 LOAD\_FAST               1 (md5\_tmp)
            421 LOAD\_CONST              20 (9)
            424 LOAD\_CONST              9 (16)
            427 SLICE+3
            428 BINARY\_ADD
            429 LOAD\_FAST               1 (md5\_tmp)
            432 LOAD\_CONST              16 (20)
            435 LOAD\_CONST              15 (28)
            438 SLICE+3
            439 BINARY\_ADD
            440 LOAD\_FAST               1 (md5\_tmp)
            443 LOAD\_CONST              21 (8)
            446 LOAD\_CONST              22 (12)
            449 SLICE+3
            450 BINARY\_ADD
            451 LOAD\_FAST               1 (md5\_tmp)
            454 LOAD\_CONST              23 (17)
            457 LOAD\_CONST              24 (21)
            460 SLICE+3
            461 BINARY\_ADD
            462 STORE\_FAST              5 (reg\_tmp)

2162         465 LOAD\_FAST               0 (self)
            468 LOAD\_ATTR               24 (regno2)
            471 LOAD\_CONST              17 (0)
            474 LOAD\_CONST              25 (4)
            477 SLICE+3
            478 LOAD\_FAST               5 (reg\_tmp)
            481 LOAD\_CONST              17 (0)
            484 LOAD\_CONST              25 (4)
            487 SLICE+3
            488 COMPARE\_OP              2 (==)
            491 JUMP\_IF\_FALSE         106 (to 600)
            494 POP\_TOP

2163         495 LOAD\_FAST               0 (self)
            498 LOAD\_ATTR               24 (regno2)
            501 LOAD\_CONST              25 (4)
            504 LOAD\_CONST              21 (8)
            507 SLICE+3
            508 LOAD\_FAST               5 (reg\_tmp)
            511 LOAD\_CONST              25 (4)
            514 LOAD\_CONST              21 (8)
            517 SLICE+3
            518 COMPARE\_OP              2 (==)
            521 JUMP\_IF\_FALSE          72 (to 596)
            524 POP\_TOP

2164         525 LOAD\_FAST               0 (self)
            528 LOAD\_ATTR               24 (regno2)
            531 LOAD\_CONST              19 (7)
            534 LOAD\_CONST              26 (15)
            537 SLICE+3
            538 LOAD\_FAST               5 (reg\_tmp)
            541 LOAD\_CONST              19 (7)
            544 LOAD\_CONST              26 (15)
            547 SLICE+3
            548 COMPARE\_OP              2 (==)
            551 JUMP\_IF\_FALSE          38 (to 592)
            554 POP\_TOP

2165         555 LOAD\_FAST               0 (self)
            558 LOAD\_ATTR               24 (regno2)
            561 LOAD\_CONST              13 (14)
            564 SLICE+1
            565 LOAD\_FAST               5 (reg\_tmp)
            568 LOAD\_CONST              13 (14)
            571 SLICE+1
            572 COMPARE\_OP              2 (==)
            575 JUMP\_IF\_FALSE          10 (to 588)
            578 POP\_TOP

2166         579 LOAD\_CONST              27 ('ok')
            582 STORE\_FAST              2 (tmp\_result)
            585 JUMP\_ABSOLUTE          593
        >>  588 POP\_TOP
            589 JUMP\_ABSOLUTE          597
        >>  592 POP\_TOP
        >>  593 JUMP\_ABSOLUTE          601
        >>  596 POP\_TOP
        >>  597 JUMP\_FORWARD            1 (to 601)
        >>  600 POP\_TOP
        >>  601 POP\_BLOCK
            602 JUMP\_FORWARD            7 (to 612)

2167     >>  605 POP\_TOP
            606 POP\_TOP
            607 POP\_TOP

2168         608 JUMP\_FORWARD            1 (to 612)
            611 END\_FINALLY

2169     >>  612 LOAD\_FAST               2 (tmp\_result)
            615 LOAD\_CONST              27 ('ok')
            618 COMPARE\_OP              2 (==)
            621 JUMP\_IF\_FALSE           8 (to 632)
            624 POP\_TOP

2170         625 LOAD\_CONST              28 ('YES')
            628 RETURN\_VALUE
            629 JUMP\_FORWARD            5 (to 637)
        >>  632 POP\_TOP

2172         633 LOA
```
