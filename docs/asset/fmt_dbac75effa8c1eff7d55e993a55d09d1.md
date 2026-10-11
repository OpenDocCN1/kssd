
```c
char * keygen(char *name)
{
        return step2(step1(name));
}

void Banner()
{
        fprintf(stderr, "Keygen for ASPMaker v2.2 by CyGwIN@sMtH.\n\n");
}

int main()
{
        char name[0x100];
        char * key;
        Banner();
        do
        {
                printf("Enter Your Name: ");
                scanf("%[^\\n]",name);getchar();
                key = keygen(name);
                if(*key=='\0')fprintf(stderr," Invalid Name!!\n");
        }while(*key=='\0');
        printf("Your RegCode Is:\n\t%s\n",key);
        printf("Press Enter to exit.");
        scanf("%[^\\n]",name);
        return 0;
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
