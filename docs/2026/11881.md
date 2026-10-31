
# [原创]高强度随机密码生成C++实现

看到有人贴了一个随机密码生成的应用，我贴下以前自己写的高强度随机密码生成C++版本的源代码吧，这垃圾玩意写的戳，放在微云上好久了，没啥用，今天看到有人贴了（[分享一个随机密码生成应用](http://bbs.pediy.com/showthread.php?p=1399609#post1399609)），也贴出来吧。

## `password.h`

```
#ifndef PASSWORD_PASSWORD_GEN_H_

#define PASSWORD_PASSWORD_GEN_H_

//////////////////////////////////////////////////////////////////////////

#include "password/basictypes.h"

//////////////////////////////////////////////////////////////////////////

namespace password{

	enum PasswordGenLength{

		kPassword32Bit = 32,

		kPassword16Bit = 16

	};

	class WIN_DLL_API PasswordGen

	{

	public:

		static PasswordGen* GetInterface(bool free_exit = false);

		virtual void DIAllocate();

		virtual void DIRelease();

		char* PasswordGenerator(const unsigned long length);

		bool IsValidPassword(const unsigned long length);

	private:

		PasswordGen(void);

		~PasswordGen(void);

		char* password_;

		DISALLOW_EVIL_CONSTRUCTORS(PasswordGen);

	};

}



#endif
```

## `password.cc`

```
#include "password/password_gen.h"



namespace password{

	static const unsigned char password_table[] = {

		0x61, 0x62, 0x63, 0x64, 0x65, 0x66, 0x3C, 0x3E, 0x40, 0x6A, 

		0x6B, 0x6C, 0x6D, 0x6E, 0x6F, 0x70, 0x71, 0x72, 0x73, 0x74, 

		0x75, 0x76, 0x77, 0x78, 0x79, 0x7A, 0x41, 0x42, 0x43, 0x44, 

		0x45, 0x46, 0x47, 0x48, 0x25, 0x26, 0x2F, 0x4B, 0x4C, 0x4D, 

		0x4F, 0x50, 0x51, 0x52, 0x53, 0x54, 0x55, 0x56, 0x57, 0x58, 

		0x59, 0x5A, 0x30, 0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37, 

		0x38, 0x39, 0x5E, 0x21, 0x24, 0x49, 0x4A, 0x28, 0x22, 0x4E,

		0x29, 0x3D, 0x3F, 0x3F, 0x5C, 0x2A, 0x2B, 0x27, 0x23, 0x3B, 

		0x2C, 0x3A, 0x2E, 0x5F, 0x2D, 0x67, 0x68, 0x69

	};

	PasswordGen* PasswordGen::GetInterface(bool free_exit){

		static PasswordGen* ref_instance;

		if(!ref_instance){

			PasswordGen* new_instance = new PasswordGen;

			if(InterlockedCompareExchangePointer(reinterpret_cast<PVOID*>(&ref_instance),new_instance,NULL)){

				delete new_instance;

			}

		}

		if(free_exit){

			delete ref_instance;

			ref_instance = NULL;

		}

		return ref_instance;

	}

	void PasswordGen::DIAllocate(){

		password_ = new char[kMaxTempLength];

		memset(password_,0,kMaxTempLength);

	}

	void PasswordGen::DIRelease(){

		if(password_){

			delete[] password_;

			password_ = NULL;

		}

	}

	char* PasswordGen::PasswordGenerator(const unsigned long length){

		srand(static_cast<unsigned int>(time(NULL)));

		for(unsigned int i=0;i<length;i++){

			int rand_position = (static_cast<int>(rand()%sizeof(password_table)-1));

			password_[i] = password_table[static_cast<int>(floor(static_cast<long double>(rand_position)))];

			while(i>0&&i<length&&isupper(password_[i-1])&&isupper(password_[i])){

				rand_position = (static_cast<int>(rand()%sizeof(password_table)-1));

				password_[i] = password_table[static_cast<int>(floor(static_cast<long double>(rand_position)))];

			}

			while(i>0&&i<length&&islower(password_[i-1])&&islower(password_[i])){

				rand_position = (static_cast<int>(rand()%sizeof(password_table)-2));

				password_[i] = password_table[static_cast<int>(floor(static_cast<long double>(rand_position)))];

			}

			while(i>0&&i<length&&isdigit(password_[i-1])&&isdigit(password_[i])){

				rand_position = (static_cast<int>(rand()%sizeof(password_table)-3));

				password_[i] = password_table[static_cast<int>(floor(static_cast<long double>(rand_position)))];

			}

			while(i>0&&i<length&&ispunct(password_[i-1])&&ispunct(password_[i])){

				rand_position = (static_cast<int>(rand()%sizeof(password_table)-4));

				password_[i] = password_table[static_cast<int>(floor(static_cast<long double>(rand_position)))];

			}

		}

		return password_;

	}

	bool PasswordGen::IsValidPassword(const unsigned long length){

		return (strlen(password_)==length);

	}

	PasswordGen::PasswordGen(void):password_(NULL){

		DIRelease();

	}

	PasswordGen::~PasswordGen(void){

		DIRelease();

	}

}
```

BTW:这垃圾玩意真的没啥用，掖着藏着干啥啊。。。。。。没搞明白。有点直接，抱歉。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
