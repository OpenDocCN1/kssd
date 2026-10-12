
# 你知道更简洁的创建桌面快捷键的程序吗？ ^\_^

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-610.htm`](https://www.pediy.com/kssd/pediy09/pediy09-610.htm)
>
> `Delphi 7`:
>
> ```delphi
> { Function: create a desktop shortcut to your application }
>
> { make sure you add these units in the uses clause. }
>
> uses ShellAPI, ComObj, ComCtrls, Shlobj, ActiveX;
>
> procedure CreateShortCut;
>
> var
>     psl    : IShellLink;
>     ppf    : IPersistFile;
>     ws     : WideString;
>     wss    : array[0..MAX_PATH] of AnsiChar;
>
> begin
>     psl := CreateComObject(CLSID_ShellLink) as IShellLink;
>     ppf := psl as IPersistFile;
>     if Assigned( psl ) then
>     try
>         psl.SetPath( PChar( ParamStr( 0 ) ) );
>         psl.SetWorkingDirectory( PChar( ExtractFilePath( Application.ExeName ) ) );
>         psl.SetShowCmd( SW_SHOWNORMAL );
>         psl.SetDescription( PChar( 'Your app name' ) );
>         GetEnvironmentVariable( PChar('USERPROFILE'), wss, sizeof(wss) - 1 );
>         ws :=  wss + '\\Desktop\\Your app name.lnk';
>         ppf.Save( PWChar( ws ), False );
>     finally
>     end;
> end;
> ```

`4dd466d0e1e0cc123afbe504da2462ca`
