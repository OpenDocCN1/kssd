
```
                dd      101h  

                ; single step  

                dd      80000004h  

                dd      except_4  

                dd      0  

                dd      0  

                dd      0  

                dd      0  

                dd      0  

                dd      0  

                ; trap  

                dd      80000004h  

                dd      origin_3  

                dd      0  

                dd      0  

                dd      0  

                dd      0  

                dd      0  

                dd      0  

                ; integer division by zero  

                dd      0c0000094h  

                dd      origin_4  

                dd      0  

                dd      0  

                dd      0  

                dd      0  

                dd      0  

                dd      0  

                ; integer overflow  

                dd      0c0000095h  

                dd      except_8  

                dd      0  

                dd      0  

                dd      0  

                dd      0  

                dd      0  

                dd      0  

                ; array bounds exceeded  

                dd      0c000008ch  

                dd      sehchain_done  

                dd      0  

                dd      0  

                dd      0  

                dd      0  

boundlimit          dd      0                       ; a little optimization X-D  

                dd      0  

end     start
```

`4dd466d0e1e0cc123afbe504da2462ca`
