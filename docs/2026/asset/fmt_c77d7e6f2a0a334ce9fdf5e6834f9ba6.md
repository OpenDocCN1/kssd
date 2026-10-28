
```cpp
class AndroidManifestEditor {
public:
    struct sAndroidManifestHead {
        int magic;
        int file_size;
    };
    char* mAndroidManifestData;
    sAndroidManifestHead* mAndroidManifestHead;
    StringChunk* mStringChunk;
    StartNamespaceChunk* mStartNamespaceChunk;
    EndNameSpaceChunk* mEndNameSpaceChunk;

    std::vector<Chunk*> chunk_list;
    std::vector<StartTagChunk*> mStartTagChunkList;
    std::vector<EndTagChunk*> mEndTagChunkList;
    std::vector<StartTagChunk*> tStartTagChunkList; // 临时列表，用来处理 StartTagChunk 和 EndTagChunk 的对应关系

    AndroidManifestEditor(const char* AndroidManifestPath) {
        mAndroidManifestData = read_file(AndroidManifestPath);
        mAndroidManifestHead = (sAndroidManifestHead*)mAndroidManifestData;

        Chunk* chunk = new Chunk(((char*)mAndroidManifestHead + sizeof(sAndroidManifestHead)));
        while (chunk->type != "Unknown") {
            if (chunk->type == "StartNameSpaceChunk") {
                mStartNamespaceChunk = new StartNamespaceChunk(chunk->addr, &mStringChunk);
            }
            else if (chunk->type == "StringChunk") {
                mStringChunk = new StringChunk(chunk->addr);
            }
            else if (chunk->type == "StartTagChunk") {
                StartTagChunk* mStartTagChunk = new StartTagChunk(chunk->addr, &mStringChunk, &mStartNamespaceChunk);
                mStartTagChunkList.push_back(mStartTagChunk);
                tStartTagChunkList.push_back(mStartTagChunk);
            }
            else if (chunk->type == "EndTagChunk") {
                EndTagChunk* mEndTagChunk = new EndTagChunk(chunk->addr, &mStringChunk);
                mEndTagChunkList.push_back(mEndTagChunk);
                tStartTagChunkList.back()->mEndTagChunk = mEndTagChunk;
                tStartTagChunkList.pop_back();
            }
            else if (chunk->type == "EndNameSpaceChunk") {
                mEndNameSpaceChunk = new EndNameSpaceChunk(chunk->addr, &mStringChunk);
            }
            chunk_list.push_back(chunk);
            chunk = chunk->NextChunk();
        }
    }

    void show() {
        printf("AndroidManifest 魔数\t\t\t\t0x%x\n", mAndroidManifestHead->magic);
        printf("AndroidManifest 文件大小\t\t\t0x%x\n", mAndroidManifestHead->file_size);

        int start_tag_id = 0;
        int end_tag_id = 0;

        for (int i = 0; i < chunk_list.size(); i++) {
            Chunk* chunk = chunk_list[i];
            printf("[%s] \n", chunk->type.c_str());
            if (chunk->type == "StringChunk") {
                printf("\n");
            }
            else if (chunk->type == "ResourceChunk") {
                printf("\n");
            }
            else if (chunk->type == "StartNameSpaceChunk") {
                std::wstring wstr = mStartNamespaceChunk->get_string();
                printf("%S\n\n", wstr.c_str());
            }
            else if (chunk->type == "StartTagChunk") {
                StartTagChunk* mStartTagChunk = mStartTagChunkList[start_tag_id];
                printf("%S\n", mStartTagChunk->mEndTagChunk->get_string().c_str());
                printf("%S\n\n", mStartTagChunk->get_string().c_str());
                start_tag_id++;
            }
            else if (chunk->type == "EndTagChunk") {
                EndTagChunk* mEndTagChunk = mEndTagChunkList[end_tag_id];
                std::wstring wstr = mEndTagChunk->get_string();
                end_tag_id++;
                printf("%S\n\n", wstr.c_str());
            }
            else if (chunk->type == "EndNameSpaceChunk") {
                std::wstring wstr = mEndNameSpaceChunk->get_string();
                printf("%S\n\n", wstr.c_str());
                break;
            }
        }
    }
};
```

## 添加字符串

现在 `AndroidManifest` 已经可以任由我们搓扁揉圆了，重新组装 `StringChunk` 一下，就可以向其中添加任意的字符串

```cpp
class StringChunk {
    char* build_raw() {
        int raw_size = sizeof(sStringChunk);
        for (int i = 0; i < StringOffset_list.size(); i++) {
            raw_size += 4;
        }
        for (int i = 0; i < StringItem_list.size(); i++) {
            raw_size += StringItem_list[i]->size;
        }

        // 4字节对齐
        if (raw_size % 4 != 0) {
            raw_size += 2;
        }

        char* new_raw = (char*)malloc(raw_size);
        memset(new_raw, 0, raw_size);
        char* cp_point = new_raw;

        ((sStringChunk*)cp_point)->scSignature = scSignature;
        ((sStringChunk*)cp_point)->scSize = raw_size;
        ((sStringChunk*)cp_point)->scStringCount = StringItem_list.size();
        ((sStringChunk*)cp_point)->scStyleCount = scStyleCount;
        ((sStringChunk*)cp_point)->scUNKNOWN = scUNKNOWN;
        ((sStringChunk*)cp_point)->scStringPoolOffset = scStringPoolOffset + 4;
        ((sStringChunk*)cp_point)->scStylePoolOffset = scStylePoolOffset;
        cp_point += sizeof(sStringChunk);

        for (int i = 0; i < StringOffset_list.size(); i++) {
            *(int*)cp_point = StringOffset_list[i];
            cp_point += 4;
        }

        for (int i = 0; i < StringItem_list.size(); i++) {
            memcpy(cp_point, StringItem_list[i]->raw, StringItem_list[i]->size);
            cp_point += StringItem_list[i]->size;
        }

        return new_raw;
    }

    StringChunk* add_string(std::wstring wstr) {

        StringItem* mStringItem = new StringItem(wstr);

        int new_StringOffset = StringOffset_list[scStringCount - 1] + StringItem_list[scStringCount - 1]->size;
        StringOffset_list.push_back(new_StringOffset);
        StringItem_list.push_back(mStringItem);

        char* new_raw = build_raw();
        return new StringChunk(new_raw);
    }

};

class AndroidManifestEditor {
    void add_string(std::wstring wstr) {
        for (int i = 0; i < chunk_list.size(); i++) {
            if (chunk_list[i]->type == "StringChunk") {
                StringChunk* newStringChunk = mStringChunk->add_string(wstr);
                mAndroidManifestHead->file_size += (newStringChunk->scSize - mStringChunk->scSize);
                delete mStringChunk;
                mStringChunk = newStringChunk;
                delete chunk_list[i];
                chunk_list[i] = new Chunk(newStringChunk->addr);
            }
        }
    }
};
```

## 修改启动项

当 `AndroidManifest` 中已存在 `ApplicationName` 时直接修改其字符串偏移即可

```cpp
class AndroidManifestEditor {
    void change_application_name(std::wstring application_name) {
        int id = mStringChunk->get_id(application_name);
        if (id == -1) {
            add_string(application_name);
            id = mStringChunk->get_id(application_name);
        }
        for (int i = 0; i < mStartTagChunkList.size(); i++) {
            if (mStartTagChunkList[i]->mEndTagChunk->get_string() == L"application") {
                std::vector<AttributeChunk*> tAttributeChunkList = mStartTagChunkList[i]->mAttributeChunkList;
                for (int i = 0; i < tAttributeChunkList.size(); i++) {
                    if (tAttributeChunkList[i]->get_srting().find(L"android:name=") != std::wstring::npos) {
                        tAttributeChunkList[i]->change_value(id);
                    }
                }
            }
        }
    }
};
```

## 总结

`AndroidManifest` 的数据结构总体来讲不算过于复杂，和PE/ELF比算是小巫见大巫了，不过由于搞得不是很精细，后续还需要多做一些测试

## 参考资料

[Android逆向之旅—解析编译之后的AndroidManifest文件格式](<https://bbs.kanxue.com/elink@a2bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0f1J5x3r3#2G2L8X3E0W2P5g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3y4e0M7#2)>)

[AndroidManifest二进制文件格式分析](https://bbs.kanxue.com/thread-194206.htm)

[Android逆向笔记 —— AndroidManifest.xml 文件格式解析](<https://bbs.kanxue.com/elink@0b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0j5$3z5o6l9H3y4e0M7@1i4K6y4r3N6i4c8E0i4K6g2X3K9h3c8Q4x3@1b7H3)>)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2024-1-18 22:07

被简单的简单编辑

，原因：
