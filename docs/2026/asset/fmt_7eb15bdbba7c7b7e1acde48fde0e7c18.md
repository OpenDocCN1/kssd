
`FALSE,` // Set handle inheritance to `FALSE`.

`CREATE_SUSPENDED,` // No creation flags.

`NULL,` // Use parent's environment block.

`NULL,` // Use parent's starting directory.

`&si,` // Pointer to `STARTUPINFO` structure.

`&pi` )

)

{
`DWORD i = GetLastError();`
`::MessageBox(NULL, TEXT("CreateProcess failed."), TEXT("Error"), MB_OK);`
// return 0;
}
else
{
`WriteProcessMemory(pi.hProcess, (LPVOID)0x31030, destcode, 24, &fk);`
`ResumeThread(pi.hThread);`
}

记住 `ppc` 是没有

```c
DWORD GetCurrentDirectory(
  DWORD nBufferLength,
  LPTSTR lpBuffer
);
```

好的到此收工，注册机太麻烦。。。主要是懒得再看代码了-\_-''用 `loader` 就可以了。

---------------------------------------------------------------------------------------------

`WiNrOOt`

`winroot@gmail.com`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
