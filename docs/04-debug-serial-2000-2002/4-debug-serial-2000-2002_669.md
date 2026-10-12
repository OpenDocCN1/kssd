
# 评语编辑室 v5.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1058.htm`](https://www.pediy.com/kssd/pediy03/forum1058.htm)

{评语编辑室 v5.0 注册机,delhpi5.0 编译通过；

以下为 `key.pas` 代码，你需要自己构造 `key.frm` 文件}

```delphi
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Classes, Graphics, Controls, Forms, Dialogs,
  StdCtrls;

type

  TForm1 = class(TForm)
    Edit1: TEdit;{其中输入用户名}
    Edit2: TEdit;{其中输入待注册码}
    Edit3: TEdit;{显示生成的注册码}

    Label1: TLabel;
    Label2: TLabel;
    Button1: TButton;{caption:='GetCode'}
    Button2: TButton;{caption:='exit'}
    Label3: TLabel;
    Label4: TLabel;
    procedure Button1Click(Sender: TObject);
    procedure Button2Click(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;
  a:string;
  sum,i,b,len:integer;

implementation

{$R \*.DFM}

procedure TForm1.Button1Click(Sender: TObject);

begin
if length(edit1.text)<6 then edit1.text:='用户名至少 6 个字符！'
else
    begin
    i:=1;  sum:=0;
    a:=edit1.text + edit2.text;
    len:=length(a);
    while i<=len do
                      begin
                      b:=ord(a\[i\])\*i;
                      sum:=b+sum;
                      i:=i+1;
                      end;
    sum:=sum\*$2A1F3+$1DACA0F;
    edit3.text:=inttostr(sum);
    sum:=0;
    end;
end;

procedure TForm1.Button2Click(Sender: TObject);

begin
close;
end;

end.
```

```c
#include "stdio.h"

main()
{
unsigned char string\[81\];
char sn\[10\];
long d=0;
int i,c;

end:printf("\*\*\*\*\*\*pyEDITOR 5.0\*\*\*\*\*\*\*\*\\nThis keygen is made by Pretender\\nPlease input your name  : ");

gets(string);

d=strlen(string);

if(d<6) {printf("Sorry,your username must longer than 5 letters!\\n"); goto end;}

printf("Please input your ID    : ");

gets(sn);

strcat(string,sn);

d=0;

for(i=0;(c=string\[i\])!='\\0';i++)
{c\*=(i+1);d+=c;}

d\*=0x2a1f3;d+=0x1daca0f;

/*printf("Your name is           : ");
puts(string);*/

printf("Your Register code is  : %ld\\n",d);

printf(" ************... (truncated for brevity in thought, but full in output below)
```

Wait, I need to output the exact content from the input but formatted correctly. The input had some escaped characters like `\[` and `\*` which likely represent `[` and `*` in the original source or just formatting artifacts. In standard C/Delphi code inside markdown blocks, I should keep the code as is unless it's clearly broken syntax due to markdown escaping in the prompt. However, looking at the prompt's input: `b:=ord(a\[i\])\*i;`. This looks like the backslashes were added to prevent markdown formatting in the *input* text provided to me. When outputting inside a code block, I should probably strip the unnecessary backslashes that were likely added to escape brackets/mults in the *plain text* context, OR keep them if they are part of the specific syntax (e.g., in Delphi `[i]` is index, `*` is multiply).

Actually, `a\[i\]` in the input looks like an artifact of how the text was copy-pasted into the prompt (escaping square brackets). In valid Delphi, it is `a[i]`. In valid C, it is `string[i]`.
Let's look at the C part: `string\[81\]`. This is definitely `string[81]`.
Let's look at the Delphi part: `a\[i\]`. This is `a[i]`.
`\*$2A1F3` -> `*$2A1F3` (Hue integer).
So, I will clean up the backslashes that are clearly escaping brackets and stars for markdown display in the input source, to restore the valid code syntax inside the code blocks.

Re-evaluating the instruction: "Do not modify the semantic content of the body text". Code blocks are part of the document. If the code has typos/escapes from copying, should I fix them? The prompt asks to "layout" (排版). Usually, this implies formatting. If the code is broken by backslashes, it's not valid code. However, strictly speaking, I should preserve the content. But `a\[i\]` is not valid Delphi. `a[i]` is. The backslashes are likely artifacts of the user's input method (escaping markdown special chars). I will remove the backslashes preceding `[`, `]`, `*` inside the code blocks to make them valid code, as this is part of "optimizing paragraph format" and ensuring standard Markdown/Code.

Let's look at the last line: `4dd466d0e1e0cc123afbe504da2462ca`. This looks like an MD5 hash or a key. It is not a command or variable name in the context of a sentence. It's just a hex string. The rule says "variable name, function name, class name, path name, command name". A raw hex string isn't any of these unless it's assigned to something. It's just text. I will leave it as is or maybe treat it as a `cmdname`? No, it's likely the result of the keygen or a checksum. I will leave it plain.

Structure:
1. Header `# 评语编辑室 v5.0`
2. Blockquote with link.
3. Note in braces `{...}`.
4. Delphi code block.
5. C code block.
6. The hex string at the end.

Refining the code blocks:
Delphi:
```delphi
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Classes, Graphics, Controls, Forms, Dialogs,
  StdCtrls;

type

  TForm1 = class(TForm)
    Edit1: TEdit;{其中输入用户名}
    Edit2: TEdit;{其中输入待注册码}
    Edit3: TEdit;{显示生成的注册码}

    Label1: TLabel;
    Label2: TLabel;
    Button1: TButton;{caption:='GetCode'}
    Button2: TButton;{caption:='exit'}
    Label3: TLabel;
    Label4: TLabel;
    procedure Button1Click(Sender: TObject);
    procedure Button2Click(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;
  a:string;
  sum,i,b,len:integer;

implementation

{$R \*.DFM}

procedure TForm1.Button1Click(Sender: TObject);

begin
if length(edit1.text)<6 then edit1.text:='用户名至少 6 个字符！'
else
    begin
    i:=1;  sum:=0;
    a:=edit1.text + edit2.text;
    len:=length(a);
    while i<=len do
                      begin
                      b:=ord(a[i])*i;
                      sum:=b+sum;
                      i:=i+1;
                      end;
    sum:=sum*$2A1F3+$1DACA0F;
    edit3.text:=inttostr(sum);
    sum:=0;
    end;
end;

procedure TForm1.Button2Click(Sender: TObject);

begin
close;
end;

end.
```

C:
```c
#include "stdio.h"

main()
{
unsigned char string[81];
char sn[10];
long d=0;
int i,c;

end:printf("******pyEDITOR 5.0******\nThis keygen is made by Pretender\nPlease input your name  : ");

gets(string);

d=strlen(string);

if(d<6) {printf("Sorry,your username must longer than 5 letters!\n"); goto end;}

printf("Please input your ID    : ");

gets(sn);

strcat(string,sn);

d=0;

for(i=0;(c=string[i])!='\0';i++)
{c*=(i+1);d+=c;}

d*=0x2a1f3;d+=0x1daca0f;

/*printf("Your name is           : ");
puts(string);*/

printf("Your Register code is  : %ld\n",d);

printf(" ******************************     ---     ---     ---\n *Welcome to WWW.CRACKNOW.COM*     /     /     / --\n ******************************     ---     ---     --/\n");
}
```

Note on the C string literals: The input had `\*\*\*` which becomes `***` in the C string. `\n` stays `\n`. `\\0` becomes `\0`.
The last printf in the input:
`printf(" \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*       ---     ---     ---\\n \*Welcome to WWW.CRACKNOW.COM\*       /       /       / --\\n \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*       ---     ---     --/\\n");`

Cleaning this up:
`printf(" ******************************     ---     ---     ---\n *Welcome to WWW.CRACKNOW.COM*     /     /     / --\n ******************************     ---     ---     --/\n");`

The trailing hex string: `4dd466d0e1e0cc123afbe504da2462ca`. I will leave it as plain text.

Let's double check rule 4: "All standalone variable names... need backticks".
Inside the code blocks, do I need to put backticks around variables?
No, code blocks already preserve formatting. The rule about backticks applies to *inline* code in regular text paragraphs.
"Code blocks add three backticks before and after".
So inside the ``` block, I do *not* add backticks to every variable. I just output the code.

The text `{评语编辑室 v5.0 注册机,delhpi5.0 编译通过； ...}` is a note. It contains `key.pas` and `key.frm`. These are filenames/paths. They should be backticked in the note text.
`key.pas` -> `` `key.pas` ``
`key.frm` -> `` `key.frm` ``
`delhpi5.0` -> This is a compiler name/ID. Should it be backticked? "Command name...". It's a proper noun/ID. I'll backtick it to be safe as a tool name. Or just `delhpi5.0`.
"v5.0" is version.

Let's draft the final output.

[content]
# 评语编辑室 v5.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1058.htm`](https://www.pediy.com/kssd/pediy03/forum1058.htm)

{评语编辑室 v5.0 注册机, `delhpi5.0` 编译通过；

以下为 `key.pas` 代码，你需要自己构造 `key.frm` 文件}

```delphi
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Classes, Graphics, Controls, Forms, Dialogs,
  StdCtrls;

type

  TForm1 = class(TForm)
    Edit1: TEdit;{其中输入用户名}
    Edit2: TEdit;{其中输入待注册码}
    Edit3: TEdit;{显示生成的注册码}

    Label1: TLabel;
    Label2: TLabel;
    Button1: TButton;{caption:='GetCode'}
    Button2: TButton;{caption:='exit'}
    Label3: TLabel;
    Label4: TLabel;
    procedure Button1Click(Sender: TObject);
    procedure Button2Click(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;
  a:string;
  sum,i,b,len:integer;

implementation

{$R \*.DFM}

procedure TForm1.Button1Click(Sender: TObject);

begin
if length(edit1.text)<6 then edit1.text:='用户名至少 6 个字符！'
else
    begin
    i:=1;  sum:=0;
    a:=edit1.text + edit2.text;
    len:=length(a);
    while i<=len do
                      begin
                      b:=ord(a[i])*i;
                      sum:=b+sum;
                      i:=i+1;
                      end;
    sum:=sum*$2A1F3+$1DACA0F;
    edit3.text:=inttostr(sum);
    sum:=0;
    end;
end;

procedure TForm1.Button2Click(Sender: TObject);

begin
close;
end;

end.
```

```c
#include "stdio.h"

main()
{
unsigned char string[81];
char sn[10];
long d=0;
int i,c;

end:printf("******pyEDITOR 5.0******\nThis keygen is made by Pretender\nPlease input your name  : ");

gets(string);

d=strlen(string);

if(d<6) {printf("Sorry,your username must longer than 5 letters!\n"); goto end;}

printf("Please input your ID    : ");

gets(sn);

strcat(string,sn);

d=0;

for(i=0;(c=string[i])!='\0';i++)
{c*=(i+1);d+=c;}

d*=0x2a1f3;d+=0x1daca0f;

/*printf("Your name is           : ");
puts(string);*/

printf("Your Register code is  : %ld\n",d);

printf(" ******************************     ---     ---     ---\n *Welcome to WWW.CRACKNOW.COM*     /     /     / --\n ******************************     ---     ---     --/\n");
}
```

4dd466d0e1e0cc123afbe504da2462ca
