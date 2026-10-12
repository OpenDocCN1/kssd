
```
{
    hr = pUrlHistoryStg2->ClearHistory();
    pUrlHistoryStg2->Release();
}*/

IEnumSTATURL* pEnumURL;
hr = pUrlHistoryStg2->EnumUrls(&pEnumURL);

STATURL suURL;
ULONG pceltFetched;
suURL.cbSize = sizeof(suURL);
hr = pEnumURL->Reset();

while((hr = pEnumURL->Next(1, &suURL, &pceltFetched)) == S_OK)
{
    //hr = pUrlHistoryStg2->DeleteUrl(suURL.pwcsUrl, 0);
    printf("%s ", W2T(suURL.pwcsUrl));
}

pEnumURL->Release();
```
