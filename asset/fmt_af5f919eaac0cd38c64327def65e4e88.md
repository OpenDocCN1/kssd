
```c
#include "..\\key\_object.h"
#include <stdlib.h>
#include <stdio.h>

#define TEST\_NUM        1024

int keys\[TEST\_NUM\];

int main(int argc, char\*\* argv)
{
        void\* hash\_pool = alloc\_object\_hash\_pool(TEST\_NUM, 512);
        int i,obj,err\_num = 0, ok\_num = 0, fail\_num = 0;
        
        for (i=0; i<TEST\_NUM; i++){
                keys\[i\] = rand();
                //printf("\[%d\] key:%d\\n", i, keys\[i\]);
                if (add\_object\_hash\_pool(hash\_pool, keys\[i\], keys + i) < 0)
                        keys\[i\] = -1;
        }

        for (i=0; i<TEST\_NUM; i++){
                obj = (int)get\_object\_hash\_pool(hash\_pool, keys\[i\]);
                if ((int)(keys + i) == obj){
                        printf("\[%d\] key:%08x \\tobj:%08x ok\\n", i, keys\[i\], obj);
                        ok\_num++;
                }else if (keys\[i\] == -1){
                        printf("\[%d\] insert fail!\\n", i);
                        fail\_num++;
                }else{
                        printf("\[%d\] key:%08x \\tobj:%08x err!!!%08x\\n", i, keys\[i\], obj, keys + i);
                        err\_num++;
                }
        }

        free\_object\_hash\_pool(hash\_pool);

        printf("Total %d:ok %d fail %d err %d\\n", TEST\_NUM, ok\_num, fail\_num, err\_num);
        return 0;
}
```
