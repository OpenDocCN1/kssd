
```
int readJS(char *path, char *data[]) {

    char *reciever = (char*)malloc(sizeof(char)*1024*1024*200);

    struct stat StatInfo; //当前文件或目录的信息

    if((stat(path,&StatInfo)) ==  -1)

    {

        Mylog_ERROR(__FILE__, __LINE__, "Can't Get File Info !!");

    }

    if(StatInfo.st_size/1024/1024> 1)

    {

        Mylog_ERROR(__FILE__, __LINE__, "Too Large To Parse !!");

        //return 0;

    }

    FILE *pf = fopen(path, "rt");

    if (NULL == pf) {

        Mylog_ERROR(__FILE__, __LINE__, "Read JS file falied!!");

        return 0;

    }



    printf("open js ok\n");



    int length = 0;

    int num = 0;

    char ch='K';//需要初始化

    reciever[0]='/';

    reciever[1]='*';

    reciever[2]='*';

    reciever[3]='/';

    length=4;

    while (ch != EOF)

    {

        ch = fgetc(pf);

        level1:

        if(ch == '\'')

        {

            reciever[length] = ch;

            length += 1;

            eee:

            ch = fgetc(pf);

            while(ch != '\'')

            {

                reciever[length] = ch;

                length += 1;

                ch = fgetc(pf);

            }

            if (reciever[length - 1] == '\\' ) //上个字符是转译字符，当前字符是/

            {

                if(reciever[length - 2]== '\\' && reciever[length - 3] != '\\')

                {

                    goto end1;

                }

                else

                {

                    reciever[length] = ch;

                    length += 1;

                    goto eee;

                }

            }

            end1:

            reciever[length] = ch;

            length += 1;

            continue;

        }

        if(ch == '\"')

        {

            reciever[length] = ch;

            length += 1;

            fff:

            ch = fgetc(pf);

            while(ch != '\"')

            {

                reciever[length] = ch;

                length += 1;

                ch = fgetc(pf);

            }

            if (reciever[length - 1] == '\\' ) //上个字符是转译字符，当前字符是/

            {

                if(reciever[length - 2]== '\\'&& reciever[length - 3] != '\\')

                {

                    goto end2;

                }

                else

                {

                    reciever[length] = ch;

                    length += 1;

                    goto fff;

                }

            }

            end2:

            reciever[length] = ch;

            length += 1;

            continue;

        }

        if (ch == ' ')

        {

            reciever[length] = ch;

            length += 1;

            ch = fgetc(pf);

            while(ch == ' ')

            {

                ch = fgetc(pf);

            }

            goto level1;

        }



        level2:

        if (ch == '{') {

            if(reciever[length-1]=='\a')

            {

                reciever[length] = ch;

                length += 1;

                reciever[length] = '\a';

                length += 1;

                continue;

            }

            reciever[length] = '\a';

            length += 1;

            reciever[length] = ch;

            length += 1;

            reciever[length] = '\a';

            length += 1;

            continue;

        }

        if (ch == '}') {

            if(reciever[length-1]=='\a')

            {

                reciever[length] = ch;

                length += 1;

                reciever[length] = '\a';

                length += 1;

                continue;

            }

            reciever[length] = '\a';

            length += 1;

            reciever[length] = ch;

            length += 1;

            reciever[length] = '\a';

            length += 1;

            continue;

        }

        if (ch == '\t') //|| ch == '\v' || ch == '\n'|| ch == '\r'

        {

            reciever[length] = ' ';

            length += 1;

            continue;

        }

//        if (ch == '\n'|| ch == '\r')

//        {

//            int a =1;

//            while (reciever[length - a] == ' ' || reciever[length - a] == '\n'|| reciever[length - a] == '\r'|| reciever[length - a] == '\t'||reciever[length - a] == '\a') { a+=1; }

//            if(reciever[length - a] == '}')

//            {

//                reciever[length] = '\n';

//                length += 1;

//                continue;

//            }

//            reciever[length] = ' ';

//            length += 1;

//            continue;

//        }

        if (ch == '/')

        {

            ch = fgetc(pf);

            int a = 1;

            while (reciever[length - a] == ' ' || reciever[length - a] == '\n'|| reciever[length - a] == '\r') { a+=1; }

            if ((reciever[length - a] == '=' && ch != '/'&& ch != '*') || (reciever[length - a] == '(' && ch != '/'&& ch != '*') || (reciever[length - a] == '&' && ch != '/'&& ch != '*')|| (reciever[length - a] == '|' && ch != '/'&& ch != '*')|| (reciever[length - a] == '!' && ch != '/'&& ch != '*')|| (reciever[length - a] == ':' && ch != '/'&& ch != '*')|| (reciever[length - a] == '[' && ch != '/'&& ch != '*')|| (reciever[length - a] == ',' && ch != '/'&& ch != '*')|| (reciever[length - a] == ';' && ch != '/'&& ch != '*'))

            {

                    reciever[length] = '/';

                    length += 1;

                    ddd:

                    while (ch != '/')

                    {

                        reciever[length] = ch;

                        length += 1;

                        ch = fgetc(pf);

                    }

                    if (reciever[length-1] == '\\' && reciever[length-2] != '\\')

                    {

                        reciever[length] = ch;

                        length += 1;

                        ch = fgetc(pf);

                        goto ddd;

                    }

                    reciever[length] = ch;

                    length += 1;

                    continue;

            }

            if(ch == '*')

            {

                ch = fgetc(pf);

                if(ch =='/')

                {

                    continue;

                }

                else

                {

                    bbb:

                    if (ch == '*')

                    {

                        ch = fgetc(pf);

                        if (ch == '/')

                        {

                            ch = fgetc(pf);

                            if (ch == EOF) { break; }

                            goto level1;

                        } else

                        {

                            goto bbb;

                        }

                    }

                    else

                    {

                        ch = fgetc(pf);

                        goto bbb;

                    }

                }

            }

            if (ch == '=')

            {

                reciever[length] = '/';

                length += 1;

                reciever[length] = ch;

                length += 1;

                continue;

            }

            if(ch == '/')

             {

                aaa:

                ch = fgetc(pf);

                if(ch == '\r' || ch =='\n')

                {

                    ch = fgetc(pf);

                    goto level1;

                }

                else if(ch ==EOF)

                {

                    break;

                }

                {

                    goto aaa;

                }

             }

            reciever[length] = '/';

            length += 1;

            goto level1;

        }



        reciever[length] = ch;

        length += 1;

        continue;

    }

    //从length-1开始需要用0填充，不填充的话length-1是一个？。

//    printf("HHHHHHHHHHHHH%c\n",reciever[length - 1]);

//    printf("HHHHHHHHHHHHH%c\n",reciever[length]);

//    printf("HHHHHHHHHHHHH%c\n",reciever[length + 1]);

    //加一个不影响语义的结束符,防止最后一个结点不存在

    reciever[length - 1] = '/';

    reciever[length] = '*';

    reciever[length + 1] = '*';

    reciever[length + 2] = '/';



    char *tmp = NULL;

    int i;

    for (i = 0; i < strlen(reciever); i++) //i= seqList->length -1 ; i >=index ; i --

    {

        tmp = &reciever[i];

        if (strncmp(tmp, "function", 8) == 0) {



            int a = 1;

            while (reciever[length - a] == ' ' || reciever[length - a] == '\n'|| reciever[length - a] == '\r') { a+=1; }

            if(reciever[length - a] == '(')

            {

                memmove(&reciever[length - a + 1], &reciever[length - a], strlen(&reciever[length - a]));

                reciever[length - a] = '\a';

                i = i-a+2;

                printf("warning:%c",reciever[i]);

                tmp = NULL;

                continue;

            }

//            if(reciever[length - a] == '=')

//            {

//                char *  backwordtoken1 = 0;

//            }

            memmove(&reciever[i + 1], &reciever[i], strlen(&reciever[i]));

            reciever[i] = '\a';

            i += 2;

            tmp = NULL;

            continue;

        }

//        if(reciever[i]== ' ')

//        {

//            while(reciever[i+1] ==' ')

//            {

//                memmove(&reciever[i + 1], &reciever[i+2], strlen(&reciever[i+2]));

//                //int end =;

//                reciever[ i+strlen(&reciever[i+1])]='\0';

//            }

//        }

//        if (reciever[i] == '\n'|| reciever[i] == '\r'|| reciever[i] == '\t' || reciever[i] == '\v')

//        {

//            reciever[length] = ' ';

//            length += 1;

//        }

        tmp = NULL;



    }

    //Mylog_INFO(__FILE__, __LINE__, "Before splite the string is:");

    //Mylog_INFO(__FILE__, __LINE__, reciever);



    char *result;

    result = strtok(reciever, "\a");



    while (result != NULL) {

        data[num] = result;

        result = strtok(NULL, "\a");

        num += 1;

    }

    fclose(pf);



    return num;



}
```
