# 设置CMake的系统名称为Generic
set(CMAKE_SYSTEM_NAME Generic)

# 设置CMake的系统版本为1
set(CMAKE_SYSTEM_VERSION 1)

# 设置CMake尝试编译的目标类型为静态库
set(CMAKE_TRY_COMPILE_TARGET_TYPE "STATIC_LIBRARY")

# 设置CMake的最低版本要求为3.20
cmake_minimum_required(VERSION 3.20)

# 设置工具链路径
set(TOOLPATH "D:/MountRiver/MounRiver/MounRiver_Studio/toolchain/RISC-V Embedded GCC12/bin/riscv-none-elf-")

# 如果当前系统是Windows
if (WIN32)
    # 输出消息：当前是Windows系统
    MESSAGE(STATUS "Now is windows!")

    # 设置C编译器路径
    set(CMAKE_C_COMPILER ${TOOLPATH}gcc.exe)

    # 设置C++编译器路径
    set(CMAKE_CXX_COMPILER ${TOOLPATH}g++.exe)

    # 设置汇编编译器路径
    set(CMAKE_ASM_COMPILER ${TOOLPATH}gcc.exe)

    # 设置AR工具路径
    set(CMAKE_AR ${TOOLPATH}ar.exe)

    # 设置OBJCOPY工具路径
    set(CMAKE_OBJCOPY ${TOOLPATH}objcopy.exe)

    # 设置OBJDUMP工具路径
    set(CMAKE_OBJDUMP ${TOOLPATH}objdump.exe)

    # 设置SIZE工具路径
    set(SIZE ${TOOLPATH}size.exe)

# 如果当前系统是UNIX或类UNIX系统
elseif (UNIX)
    # 输出消息：当前是UNIX或类UNIX系统
    MESSAGE(STATUS "Now is UNIX-like OS!")

    # 设置C编译器路径
    set(CMAKE_C_COMPILER ${TOOLPATH}gcc)

    # 设置C++编译器路径
    set(CMAKE_CXX_COMPILER ${TOOLPATH}g++)

    # 设置汇编编译器路径
    set(CMAKE_ASM_COMPILER ${TOOLPATH}gcc)

    # 设置AR工具路径
    set(CMAKE_AR ${TOOLPATH}ar)

    # 设置OBJCOPY工具路径
    set(CMAKE_OBJCOPY ${TOOLPATH}objcopy)

    # 设置OBJDUMP工具路径
    set(CMAKE_OBJDUMP ${TOOLPATH}objdump)

    # 设置SIZE工具路径
    set(SIZE ${TOOLPATH}size)

# 如果当前系统不是Windows也不是UNIX或类UNIX系统
else ()
    # 输出消息：不支持的系统
    MESSAGE(STATUS "Unsupported system!")
endif ()

# 设置项目名称和使用的语言
project(ch32v203-FreeRTOS C CXX ASM)

# 设置C++标准为C++17
set(CMAKE_CXX_STANDARD 17)

# 设置C标准为C99
set(CMAKE_C_STANDARD 99)

# 添加编译选项
add_compile_options(
        -march=rv32imac  # 设置目标架构为RISC-V，具有RV32I基础整数指令集，M标准乘法和除法扩展，A标准原子扩展，C标准压缩扩展
        -mabi=ilp32     # 设置应用二进制接口（ABI）为ilp32，即整数、长整数和指针都是32位
        -mcmodel=medany  # 设置代码模型为medany，适用于任何大小的代码和数据
        -msmall-data-limit=8  # 设置小数据区的大小限制为8字节
        -mno-save-restore  # 禁用在函数调用前后保存和恢复寄存器的优化
)

add_compile_options(
        -fmessage-length=0  # 设置诊断消息的最大长度为0，即没有最大长度
        -fsigned-char  # 设置char类型为有符号类型
        -ffunction-sections  # 将每个函数放入自己的section中，以便链接器优化
        -fdata-sections  # 将数据放入自己的section中，以便链接器优化
        -fno-common  # 不允许多个文件共享同一个全局变量
)

# 编译等级
if ("${CMAKE_BUILD_TYPE}" STREQUAL "Release")
    message(STATUS "Maximum optimization for speed")
    add_compile_options(-Ofast -flto)
elseif ("${CMAKE_BUILD_TYPE}" STREQUAL "Debug")
    message(STATUS "Debug mode")
    add_compile_options(-Og -g)
else ()
    message(STATUS "Minimal optimization, debug info included")
    add_compile_options(-Og -g)
endif ()

# 编译信息等级
add_compile_options(-Wall -Wextra -Wpedantic -Wstack-usage=2048)


# 头文件路径
include_directories(
        User
        Core
        FreeRTOS/include
        FreeRTOS/portable/GCC/RISC-V
        FreeRTOS/portable/GCC/RISC-V/chip_specific_extensions/RV32I_PFIC_no_extensions
        Debug
        Peripheral/inc
)

# 宏定义
# add_definitions(-DDEBUG=1)

# 源码文件
file(GLOB_RECURSE SOURCES
        "User/*.c"
        "Core/core_riscv.c"
        "Debug/debug.c"
        "FreeRTOS/*.c"
        "FreeRTOS/portable/GCC/RISC-V/*.c"
        "FreeRTOS/portable/MemMang/heap_4.c"
        "FreeRTOS/portable/GCC/RISC-V/portASM.S"
        "Peripheral/src/*.c"
        "Startup/startup_ch32v20x_D6.S"
)
list(REMOVE_ITEM SOURCES ${CMAKE_SOURCE_DIR}/FreeRTOS/portable/Common/mpu_wrappers.c)


# 设置链接脚本的路径
set(LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/Ld/Link.ld)

# 添加链接选项
add_link_options(
    -march=rv32imac  # 设置目标架构为RISC-V，具有RV32I基础整数指令集，M标准乘法和除法扩展，A标准原子扩展，C标准压缩扩展
    -mabi=ilp32  # 设置应用二进制接口（ABI）为ilp32，即整数、长整数和指针都是32位
    -nostartfiles  # 不使用标准系统启动文件
    -Xlinker --gc-sections  # 通过链接器选项--gc-sections，删除未使用的代码和数据块
    -Wl,--print-memory-usage  # 通过链接器选项--print-memory-usage，打印内存使用情况
    -Wl,-Map,${PROJECT_NAME}.map  # 通过链接器选项-Map，生成map文件，显示符号的分布情况
    --specs=nano.specs  # 使用nano.specs规格文件，这会链接更小的C库
    --specs=nosys.specs  # 使用nosys.specs规格文件，这会链接一个不包含系统调用的C库
)

# 添加链接脚本选项
add_link_options(-T ${LINKER_SCRIPT})

# 编译可执行文件
add_executable(${PROJECT_NAME}.elf ${SOURCES} ${LINKER_SCRIPT})

# 链接静态库
# target_link_libraries(${PROJECT_NAME}.elf printfloat)

# 设置输出文件的变量
set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)
set(LST_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.lst)

# 添加自定义命令以在构建目标后生成输出文件
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -Oihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${HEX_FILE} # 生成 .hex 文件
    COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:${PROJECT_NAME}.elf> ${BIN_FILE} # 生成 .bin 文件
    COMMAND ${CMAKE_OBJDUMP} --all-headers --demangle --disassemble $<TARGET_FILE:${PROJECT_NAME}.elf> > ${LST_FILE} # 生成 .lst 文件
    COMMENT "Building ${HEX_FILE}\nBuilding ${BIN_FILE}" # 在生成输出文件后显示消息
)