
```c
for(j=0;j<0x0168;j++) /* FROM A98H TO BFFH, TOTAL 168H BYTES */
{
    if(fputc(0x00,fp)==EOF)
    {
        printf("\nWrite file %s error !\n",file_name);
        fclose(fp);
        exit(0);
    }
}

for(j=0;j<0x0010;j++) /* FROM C00H TO C0FH, TOTAL 010H BYTES */
{
    if(fputc(data_C00h_to_C0Fh[j],fp)==EOF)
    {
        printf("\nWrite file %s error !\n",file_name);
        fclose(fp);
        exit(0);
    }
}

for(j=0;j<0x01F0;j++) /* FROM C10H TO DFFH, TOTAL 1F0H BYTES */
{
    if(fputc(0x00,fp)==EOF)
    {
        printf("\nWrite file %s error !\n",file_name);
        fclose(fp);
        exit(0);
    }
}

fclose(fp);
printf("\n%s has been created successfully !\n\n",file_name);
}
```
