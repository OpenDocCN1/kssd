
```c
  srand16((seed3 ^seed4) & 0xffff, 
   ((seed3 ^ seed4) >> 16) & 0xffff, lmrands); 
  rand16(lmrands, 1); 
  rand16(lmrands, 1); 
  rand16(lmrands, 1); 
  i[0] = rand16(lmrands, 1); 
  i[1] = rand16(lmrands, 1); 
  i[2] = rand16(lmrands, 1); 
  i[3] = rand16(lmrands, 1); 
  fprintf(ofp, 
 "  if (%s == %s) %s\\n\\ 
  {\\n\\ 
   unsigned char %s = *%s;\\n\\ 
   unsigned int %s = %s/%d; %s\\n\\ 
 \\n\\ 
   if ((%s == %s) && ((%s ^ %d) & 0xff)) %s ^= %d;\\n\\ 
   if ((%s == (%s + 1)) && ((%s ^ %d) & 0xff)) %s ^= %d;\\n\\ 
   if ((%s == (%s + 3)) && ((%s ^ %d) & 0xff)) %s ^= %d;\\n\\ 
   if ((%s == (%s + 2)) && ((%s ^ %d) & 0xff)) %s ^= %d;\\n\\ 
   *%s = %s;\\n\\ 
   return 0;\\n\\ 
  }\\n", switchname, bytenames[1], 
  DEVCOMMENT("/* == 1 for handshake */"), 
  charname, charptrname, 
  zeroname, charname, rand16(lmrands, 1)%400 + 257, 
  DEVCOMMENT("/* The result of the divide is guaranteed to be zero */"), 
  idxname, zeroname, charname, i[0], charname, i[0], 
  idxname, zeroname, charname, i[1], charname, i[1], 
  idxname, zeroname, charname, i[2], charname, i[2], 
  idxname, zeroname, charname, i[3], charname, i[3], 
  charptrname, charname); 
} 

 

 

static 
int 
decode(job, d, buf, key, junk, rcvmsg) 
LM_HANDLE *job; 
long d; 
char *buf; 
long key; 
char *junk; 
int rcvmsg; 
{ 
  int verrev; 

#ifndef RELEASE_VERSION  
  which_failed++; 
#endif 
  verrev = (job->daemon->ver * 100) + job->daemon->rev; 
  memset(junk, 0, MAX_HAND_DATA_LEN+1); 
  l_encode_long_hex(junk, l_modify((long)d)); 
/* 
 *  Handle the case where it's a pre-v6 server, and 
 *  it's the extra handshake tests 
*/ 
  if ((rcvmsg == -1 && (verrev < 579) )) return 1; 

  if (rcvmsg != LM_HANDSHAKE && rcvmsg != -1) 
  { 
    LM_SET_ERRNO(job, LM_BADHANDSHAKE, 375, 0); 
    return 0; 
  } 

  l_str_crypt(junk, MAX_HAND_DATA_LEN, key, 1); 
  if (!memcmp( buf, junk, MAX_HAND_DATA_LEN)) return 1; 

  if (rcvmsg == -1)  LM_SET_ERRNO(job, LM_BADHANDSHAKE, 376, 0) 
  else       LM_SET_ERRNO(job, LM_BADHANDSHAKE, 377, 0) 

  return 0; 
}
```
`4dd466d0e1e0cc123afbe504da2462ca`
