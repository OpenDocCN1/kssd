
# CMake 配置示例（Linux/Windows）

```cmake
cmake_minimum_required (VERSION 3.8)

	project (src)



	if(CMAKE_HOST_SYSTEM_NAME MATCHES "Linux")

		set(ThirdLib "/usr/lib/boost_1_57_0/")



		include_directories("${ThirdLib}/include")

		MESSAGE (STATUS "ThirdLib/include :" ${ThirdLib}/include)

		include_directories("./src")

		

		set(link_files 	dl

						pthread

						)



		add_compile_options(-std=c++11)



		

		set(CMAKE_CXX_FLAGS_DEBUG "-g")

		set(CMAKE_CXX_FLAGS_RELEASE "-O0 -g")

		set(CMAKE_CXX_FLAGS "-fpic ${CMAKE_CXX_FLAGS}")

		set(CMAKE_CXX_FLAGS "-lrt ${CMAKE_CXX_FLAGS}")



		set(SRC_CPP_FILES 

			${PROJECT_SOURCE_DIR}/src/main2.cpp



		)



		add_executable(Hello ${SRC_CPP_FILES}

								${VersionFilesOutputVariable})



		target_link_libraries(Hello ${link_files})



		#以下是设置可执行文件的输出路径

		set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/../bin/linux_debug)

		MESSAGE (STATUS "exe PROJECT_BINARY_DIR :" ${PROJECT_SOURCE_DIR})

		MESSAGE (STATUS "EXECUTABLE_OUTPUT_PATH :" ${EXECUTABLE_OUTPUT_PATH})

		#以上是设置可执行文件的输出路径



	elseif(CMAKE_HOST_SYSTEM_NAME MATCHES "Windows")

		set(ThirdLib ${PROJECT_SOURCE_DIR}/../../../../ThirdLib)





		add_definitions(-DUNICODE -D_UNICODE)

		add_definitions(-DGTEST_NOT_REAL_FILES)	

		add_definitions(-DUSER_WIN_H) 

		add_definitions(-D_CONSOLE)



		include_directories("${ThirdLib}/include")



		include_directories("./src")

		MESSAGE (STATUS "ThirdLib/include :" ${ThirdLib}/include)



		set(CMAKE_SHARED_LINKER_FLAGS_RELWITHDEBINFO "/NODEFAULTLIB:MSVCRT.lib ${CMAKE_SHARED_LINKER_FLAGS_RELWITHDEBINFO}")

		set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "/NODEFAULTLIB:MSVCRT.lib ${CMAKE_SHARED_LINKER_FLAGS_RELEASE}")

		set(CMAKE_SHARED_LINKER_FLAGS_DEBUG "/NODEFAULTLIB:MSVCRTD.lib ${CMAKE_SHARED_LINKER_FLAGS_DEBUG}")

	##[[	

		#设置C++运行时库为MT

		set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")

		set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} /MT")

		set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")

		#设置C运行时库为MT

		set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /MT")

		set(CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_C_FLAGS_RELWITHDEBINFO} /MT")

		set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /MTd")

	#]]

	#通过下面这几句设置为MT链接方式也可以

	#[[

		if (MSVC)

			set(CompilerFlags

				CMAKE_CXX_FLAGS

				CMAKE_CXX_FLAGS_DEBUG

				CMAKE_CXX_FLAGS_RELEASE

				CMAKE_C_FLAGS

				CMAKE_C_FLAGS_DEBUG

				CMAKE_C_FLAGS_RELEASE

				)

			foreach(CompilerFlag ${CompilerFlags})

				string(REPLACE "/MD" "/MT" ${CompilerFlag} "${${CompilerFlag}}")

			endforeach()

		endif(MSVC)

	]]

		set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /Zi")

		set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /DEBUG /OPT:REF /OPT:ICF")

		set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "${CMAKE_SHARED_LINKER_FLAGS_RELEASE} /DEBUG /OPT:REF /OPT:ICF")



		set(link_files 	kernel32

						user32

						gdi32

					)



		SET (CMAKE_MODULE_PATH  "${ThirdLib}/build/cmake_product_version;${CMAKE_MODULE_PATH}")

		MESSAGE (STATUS "CMAKE_MODULE_PATH :" ${CMAKE_MODULE_PATH})

		include(generate_product_version)

		generate_product_version(

		   VersionFilesOutputVariable

		   NAME "Hello.exe"

		   VERSION_MAJOR 1

		   VERSION_MINOR 0

		   VERSION_PATCH ${BUILD_COUNTER}

		   VERSION_REVISION 1001

		)





		set(SRC_CPP_FILES 

			${PROJECT_SOURCE_DIR}/src/getopt.c

			${PROJECT_SOURCE_DIR}/src/main.cpp

			)

		   

	add_executable(Hello ${SRC_CPP_FILES}

							${VersionFilesOutputVariable})





	target_link_libraries(Hello ${link_files})



	#以下是设置可执行文件的输出路径

	set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/../bin/win32)

	MESSAGE (STATUS "exe PROJECT_BINARY_DIR :" ${PROJECT_SOURCE_DIR})

	MESSAGE (STATUS "EXECUTABLE_OUTPUT_PATH :" ${EXECUTABLE_OUTPUT_PATH})

	#以上是设置可执行文件的输出路径



	endif()
```

# 15、另外一个demo

```cmake
# CMakeList.txt: CMakeProject 的 CMake 项目，在此处包括源代码并定义
# 项目特定的逻辑。

cmake_minimum_required (VERSION 3.8)
PROJECT(Hello)

# set env

if(CMAKE_COMPILER_IS_GNUCXX)

    add_compile_options(-std=c++11)

    message(STATUS "optional:-std=c++11")

    message(STATUS "use c17inject")

else()

    add_compile_options(-std=c++17)

    message(STATUS "optional:-std=c++17")  

endif(CMAKE_COMPILER_IS_GNUCXX)

# common env (win&linux)

MESSAGE(STATUS "COMMON_SRC_ROOT=$ENV{COMMON_SRC_ROOT}")

MESSAGE(STATUS "LINUX_LIBRARY_PATH=$ENV{LINUX_LIBRARY_PATH}")

MESSAGE(STATUS "3RDPARTY_LIBRARY_ROOT=$ENV{3RDPARTY_LIBRARY_ROOT}")

MESSAGE(STATUS "PUBLIC_LIBRARY_PATH=$ENV{PUBLIC_LIBRARY_PATH}")

IF (CMAKE_SYSTEM_NAME MATCHES "Linux")  
##################################Linux System Start#######################################
    MESSAGE(STATUS "current platform: Linux ")

# 预编译宏
add_definitions(-D__LINUX__)
add_definitions(-DUNICODE -D_UNICODE)

set(ENV{LINUX_LIBRARY_PATH} /usr/public/linux)
set(ENV{3RDPARTY_LIBRARY_ROOT} /usr/public/3rdparty)
set(ENV{COMMON_SRC_ROOT} /usr/public/common)

# include headers
include_directories("$ENV{LINUX_LIBRARY_PATH}/Include")
include_directories("$ENV{3RDPARTY_LIBRARY_ROOT}/")
include_directories("$ENV{COMMON_SRC_ROOT}")

set(HELLO_SRC 
    ${PROJECT_SOURCE_DIR}/src/Hello.cpp
    ${PROJECT_SOURCE_DIR}/src/pch.cpp
   )

# gene so
add_library(HelloTest SHARED 
    ${HELLO_SRC} 
    )

set(CMAKE_CXX_FLAGS "-fpic ${CMAKE_CXX_FLAGS}")
set(CMAKE_CXX_FLAGS "-lrt ${CMAKE_CXX_FLAGS}")

# ext op
SET(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")  # Debug模式下的编译指令
#SET(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")        # Release模式下的编译指令

# add link lib
TARGET_LINK_LIBRARIES(HelloTest dl)

ENDIF (CMAKE_SYSTEM_NAME MATCHES "Linux") 
##################################Linux System End#######################################

##################################Windows System Start#####################################  
IF (CMAKE_SYSTEM_NAME MATCHES "Windows")  
    MESSAGE(STATUS "current platform: Windows")  

add_definitions(-DUNICODE -D_UNICODE)
add_compile_options(-std=c++11)

# include headers GLOBAL_PUBLIC_LIBRARY_PATH和3RDPARTY_LIBRARY_ROOT和COMMON_SRC_ROOT是windows上设置的环境变量
include_directories("$ENV{PUBLIC_LIBRARY_PATH}/Include")
include_directories("$ENV{3RDPARTY_LIBRARY_ROOT}/")
include_directories("$ENV{COMMON_SRC_ROOT}")

# gene dll
set(HELLO_SRC 
    ${PROJECT_SOURCE_DIR}/src/dll.cpp HelloTest.def
    ${PROJECT_SOURCE_DIR}/src/hello.cpp
   )

# set product version cmake modules path: 
SET (CMAKE_MODULE_PATH "$ENV{COMMON_SRC_ROOT}/cmake_product_version/;${CMAKE_MODULE_PATH}")
MESSAGE (STATUS "CMAKE_MODULE_PATH :" ${CMAKE_MODULE_PATH})
include_directories("${CMAKE_MODULE_PATH}/")

include(generate_product_version)
generate_product_version(
   VersionFilesOutputVariable
   NAME "HelloTest.dll"
   VERSION_MAJOR 10
   VERSION_MINOR 0
   VERSION_PATCH ${BUILD_COUNTER}
   VERSION_REVISION 1030
)

add_library(HelloTest SHARED
    ${HELLO_SRC}
    ${VersionFilesOutputVariable}
)

set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} /MT")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")

SET(BASE_LIB "$ENV{PUBLIC_LIBRARY_PATH}/Lib/Windows")

if(CMAKE_CL_64)
target_link_libraries(HelloTest "${BASE_LIB}/mt_64.lib")
else()
target_link_libraries(HelloTest "${BASE_LIB}/mt_32.lib")
endif()

#SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)       #设置可执行文件的输出目录	

ENDIF (CMAKE_SYSTEM_NAME MATCHES "Windows") 
##################################Windows System End#####################################
```

# 16、vs cmake远程调试

右击项目->添加调试配置->修改`launch.vs.json`

window配置（修改带注释的5项），然后启动远程调试`msvsmon.exe`
