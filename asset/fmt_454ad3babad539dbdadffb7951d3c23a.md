
```
void G_0051C35E(int ,char*)
{
}
void Convert_Str(char * str)
{
  char *cp;    
  for ( cp = str ; *cp ; ++cp )
    if ( ('a' <= *cp) && (*cp <= 'z') )
      *cp -= 'a' - 'A';
}
```
谢谢各位大侠路过。。。。。。。。。
