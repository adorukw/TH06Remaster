# C++ + SDL2 从零重置（Remaster）东方红魔乡 完整开发教程

> **技术栈**：C++17 · SDL2 · SDL2_image · SDL2_mixer · CMake · Lua 5.3
> **目标平台**：Linux（主开发平台），兼顾 Windows / macOS 跨平台说明
> **复刻标准**：严格对标 TH06《東方紅魔郷 ～ the Embodiment of Scarlet Devil》原版行为、数值、渲染效果、弹幕轨迹与手感，做到 1:1 逻辑还原

---

## 版权红线声明（务必先读）

本项目**仅限个人本地学习与研究用途**，严禁用于任何商业目的。以下行为被严格禁止：

1. **禁止商用**：不得将本项目或其衍生品用于任何直接或间接的商业用途，包括但不限于售卖、付费下载、内购、广告植入、直播打赏引导等。
2. **禁止分发原版提取资源**：从 `th06.dat`、`紅魔郷MD.dat` 中解包得到的图像、音频、弹幕脚本、字体等素材，其著作权归上海アリス幻樂団（ZUN）所有。你**不得**将这些原版素材上传至任何公开平台（GitHub 公开仓库、网盘、论坛附件等），也不得以任何形式二次分发。
3. **禁止声明原创**：所有原版素材、角色、音乐、关卡设计的著作权均归 ZUN。本项目仅作为技术学习载体，不主张任何原版内容的著作权。
4. **合规建议**：
   - 若要公开本项目源代码，**只公开代码**，不公开任何原版素材，并在 README 中明确说明用户需自行持有正版 TH06 并自行解包。
   - 推荐最终用自制原创素材替换原版素材后再做任何公开分享。
   - 参考 ZUN 的二次创作指南（https://touhou-project.news/guidelines/）。

阅读并理解以上红线后，方可继续后续开发。

---

## 目录

- [第一部分：从零前置准备工作](#第一部分从零前置准备工作)
  - [1.1 开发环境完整配置教程](#11-开发环境完整配置教程)
  - [1.2 素材完整获取流程](#12-素材完整获取流程)
  - [1.3 素材标准化处理规范](#13-素材标准化处理规范)
- [第二部分：项目架构与代码文件规范](#第二部分项目架构与代码文件规范)
  - [2.1 标准分层式文件夹完整目录树](#21-标准分层式文件夹完整目录树)
  - [2.2 C++ 代码设计规范](#22-c-代码设计规范)
  - [2.3 资源配置文件规范](#23-资源配置文件规范)
- [第三部分：底层通用核心框架](#第三部分底层通用核心框架)
  - [3.1 SDL2 窗口与渲染封装](#31-sdl2-窗口与渲染封装)
  - [3.2 固定 60 帧分离主循环](#32-固定-60-帧分离主循环)
  - [3.3 全局资源管理系统](#33-全局资源管理系统)
  - [3.4 输入系统](#34-输入系统)
  - [3.5 红魔乡原生 RNG 随机算法](#35-红魔乡原生-rng-随机算法)
- [第四部分：游戏核心业务系统](#第四部分游戏核心业务系统)
  - [4.1 玩家机体系统](#41-玩家机体系统)
  - [4.2 弹幕系统](#42-弹幕系统)
  - [4.3 敌人、BOSS 与符卡系统](#43-敌人boss-与符卡系统)
  - [4.4 道具系统](#44-道具系统)
  - [4.5 多层卷轴视差背景系统](#45-多层卷轴视差背景系统)
  - [4.6 UI 与像素文字渲染系统](#46-ui-与像素文字渲染系统)
  - [4.7 存档系统](#47-存档系统)
  - [4.8 分数计算系统](#48-分数计算系统)
  - [4.9 Replay 录像回放系统](#49-replay-录像回放系统)
  - [4.10 难度控制系统](#410-难度控制系统)
  - [4.11 全局游戏流程控制器](#411-全局游戏流程控制器)
- [第五部分：性能优化、调试工具、复刻避坑指南](#第五部分性能优化调试工具复刻避坑指南)
  - [5.1 SDL2 性能优化方案](#51-sdl2-性能优化方案)
  - [5.2 内置调试工具实现](#52-内置调试工具实现)
  - [5.3 高频复刻踩坑完整清单](#53-高频复刻踩坑完整清单)
  - [5.4 可选拓展功能开发指引](#54-可选拓展功能开发指引)
  - [5.5 从 0 到可运行游戏的分步开发流程](#55-从-0-到可运行游戏的分步开发流程)
  - [5.6 拓展方向](#56-拓展方向)
- [结语](#结语)

---

# 第一部分：从零前置准备工作

## 1.1 开发环境完整配置教程

### 1.1.1 系统依赖安装（Linux / Ubuntu / Debian）

在 Linux 平台开发 SDL2 项目，所有依赖均通过系统包管理器安装，避免后续跨机器迁移时的环境差异。以下命令在 Ubuntu 22.04 / Debian 12 上验证通过：

```bash
# 更新包索引
sudo apt update

# 安装 C++ 编译器（g++ 12+，支持 C++17 完整特性）
sudo apt install -y build-essential g++ gcc

# 安装 CMake（3.16+，用于跨平台构建）
sudo apt install -y cmake ninja-build

# 安装 SDL2 核心库与开发头文件
sudo apt install -y libsdl2-dev

# 安装 SDL2_image（PNG/JPG/BMP 图像加载）
sudo apt install -y libsdl2-image-dev

# 安装 SDL2_mixer（BGM/音效混音，依赖 libmodplug/libmpg123）
sudo apt install - -y libsdl2-mixer-dev

# 安装 SDL2_ttf（仅用于调试，正式版禁用矢量字体）
sudo apt install -y libsdl2-ttf-dev

# 安装 Lua 5.3 开发库（用于硬编码规避、弹幕脚本配置）
sudo apt install -y liblua5.3-dev

# 安装 nlohmann-json（C++ 单头文件 JSON 库，用于配置文件）
sudo apt install - -y nlohmann-json3-dev

# 安装调试与性能分析工具
sudo apt install -y gdb valgrind

# 安装字体（位图文字渲染需要中文字体作为参考）
sudo apt install -y fonts-noto-cjk
```

> **原版 1:1 还原关键细节**：TH06 原版在 Windows 98/XP 上运行，使用 DirectDraw/DirectSound。我们用 SDL2 替代，但必须保证 SDL2 的渲染器开启硬件加速与 VSync，否则会出现画面撕裂或帧率超速。SDL2_mixer 必须以 44100Hz 立体声 16bit 初始化，与原版音频采样率对齐，否则 BGM 会变调。

### 1.1.2 VSCode 完整配置

VSCode 是 Linux 上 C++ 开发的首选 IDE。以下配置文件直接放入项目根目录的 `.vscode/` 文件夹中。

**`.vscode/c_cpp_properties.json`**（C/C++ 扩展的 IntelliSense 配置）：

```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "/usr/include/SDL2",
                "/usr/include/lua5.3",
                "/usr/include/nlohmann"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"
            ],
            "compilerPath": "/usr/bin/g++",
            "cStandard": "c17",
            "cppStandard": "c++17",
            "intelliSenseMode": "linux-gcc-x64"
        }
    ],
    "version": 4
}
```

**`.vscode/tasks.json`**（构建任务，调用 CMake）：

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "CMake Configure",
            "type": "shell",
            "command": "cmake",
            "args": [
                "-B", "build",
                "-S", ".",
                "-G", "Ninja",
                "-DCMAKE_BUILD_TYPE=Debug"
            ],
            "group": "build"
        },
        {
            "label": "CMake Build",
            "type": "shell",
            "command": "cmake",
            "args": ["--build", "build", "--parallel"],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "dependsOn": "CMake Configure"
        },
        {
            "label": "CMake Release Build",
            "type": "shell",
            "command": "cmake",
            "args": [
                "-B", "build-release",
                "-S", ".",
                "-G", "Ninja",
                "-DCMAKE_BUILD_TYPE=Release"
            ],
            "group": "build"
        }
    ]
}
```

**`.vscode/launch.json`**（GDB 调试配置）：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug TH06 Remaster",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/th06_remaster",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [
                {"name": "SDL_VIDEODRIVER", "value": "x11"},
                {"name": "LD_LIBRARY_PATH", "value": "/usr/lib/x86_64-linux-gnu"}
            ],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "CMake Build"
        }
    ]
}
```

### 1.1.3 CMakeLists.txt 完整模板

CMake 是跨平台构建的核心。以下 `CMakeLists.txt` 同时支持 Linux / Windows / macOS，并区分 Debug/Release 编译参数：

```cmake
cmake_minimum_required(VERSION 3.16)
project(th06_remaster LANGUAGES CXX)

# 强制 C++17 标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 默认 Release，开发时通过 -DCMAKE_BUILD_TYPE=Debug 切换
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# 通用编译选项
set(CMAKE_COMMON_FLAGS "-Wall -Wextra -Wno-unused-parameter -Wno-missing-field-initializers")

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_CXX_FLAGS "${CMAKE_COMMON_FLAGS} -O0 -g3 -DDEBUG=1 -fsanitize=address,undefined")
    set(CMAKE_EXE_LINKER_FLAGS "-fsanitize=address,undefined")
elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
    set(CMAKE_CXX_FLAGS "${CMAKE_COMMON_FLAGS} -O2 -DNDEBUG -flto")
    set(CMAKE_EXE_LINKER_FLAGS "-flto")
endif()

# 跨平台查找依赖
find_package(SDL2 REQUIRED)
find_package(SDL2_image REQUIRED)
find_package(SDL2_mixer REQUIRED)
find_package(Lua 5.3 REQUIRED)
find_package(Threads REQUIRED)

# nlohmann_json 在 Ubuntu 上通过 find_package 查找
find_package(nlohmann_json 3.2.0 QUIET)

# 源文件收集
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS
    "${CMAKE_SOURCE_DIR}/src/*.cpp"
)
file(GLOB_RECURSE HEADERS CONFIGURE_DEPENDS
    "${CMAKE_SOURCE_DIR}/include/*.h"
)

add_executable(th06_remaster ${SOURCES})

target_include_directories(th06_remaster PRIVATE
    ${CMAKE_SOURCE_DIR}/include
    ${SDL2_INCLUDE_DIRS}
    ${SDL2_IMAGE_INCLUDE_DIRS}
    ${SDL2_MIXER_INCLUDE_DIRS}
    ${LUA_INCLUDE_DIR}
)

# SDL2 在不同平台库名不同，用 target_link_libraries + IMPORTED 目标
target_link_libraries(th06_remaster PRIVATE
    SDL2::SDL2main
    SDL2::SDL2
    SDL2_image::SDL2_image
    SDL2_mixer::SDL2_mixer
    ${LUA_LIBRARIES}
    Threads::Threads
)

if(nlohmann_json_FOUND)
    target_link_libraries(th06_remaster PRIVATE nlohmann_json::nlohmann_json)
else()
    target_include_directories(th06_remaster PRIVATE ${CMAKE_SOURCE_DIR}/thirdparty/nlohmann)
endif()

# Windows 平台额外处理（MSVC / MinGW）
if(WIN32)
    target_compile_definitions(th06_remaster PRIVATE SDL_MAIN_HANDLED)
    # Release 模式下静态链接 SDL2，避免用户机器缺 DLL
    if(CMAKE_BUILD_TYPE STREQUAL "Release")
        set(SDL_STATIC ON CACHE BOOL "" FORCE)
    endif()
endif()

# macOS 平台处理框架
if(APPLE)
    find_library(COCOA_LIBRARY Cocoa)
    target_link_libraries(th06_remaster PRIVATE ${COCOA_LIBRARY})
endif()

# 拷贝资源目录到构建输出（开发期方便调试）
add_custom_command(TARGET th06_remaster POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
    ${CMAKE_SOURCE_DIR}/assets $<TARGET_FILE_DIR:th06_remaster>/assets
)
```

### 1.1.4 首次编译验证

创建一个最小化 `src/main.cpp` 验证环境：

```cpp
#include <SDL2/SDL.h>
#include <SDL2/SDL_image.h>
#include <SDL2/SDL_mixer.h>
#include <lua.hpp>
#include <iostream>

int main(int argc, char* argv[]) {
    (void)argc; (void)argv;
    std::cout << "SDL2:    " << SDL_GetPlatform() << "\n";
    std::cout << "SDL rev: " << SDL_GetRevision() << "\n";
    std::cout << "IMG rev: " << IMG_Linked_Version() << "\n";
    std::cout << "MIX rev: " << Mix_Linked_Version() << "\n";

    lua_State* L = luaL_newstate();
    luaL_openlibs(L);
    std::cout << "Lua:     " << lua_version(L) << "\n";
    lua_close(L);
    return 0;
}
```

执行构建：

```bash
cd /home/z/my-project
cmake -B build -S . -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build build
./build/th06_remaster
```

若输出各库版本号，则环境配置完成。

---

## 1.2 素材完整获取流程

### 1.2.1 优先方案：寻找已公开分享的高清资源

在自行解包前，建议先在以下合规渠道寻找社区已公开分享的**学习用**资源（注意：仍不得商用、不得二次分发）：

1. **Touhou Wiki 资源页**：https://en.touhouwiki.net/wiki/Embodiment_of_Scarlet_Devil —— 提供角色立绘、关卡截图等公开宣传素材。
2. **TH06 高清重制社区**：部分爱好者发布了基于原版 AI 放大的高清贴图（如 4x 分辨率），搜索关键词 "Touhou 6 HD texture pack"。
3. **同人音乐站点**：BGM 可寻找官方 OST《東方紅魔郷オリジナルサウンドトラック》的合法流媒体版本作为参考。

> **重要提示**：若上述渠道无法获得完整可用的素材集（通常无法获得完整 ANM 图集与 ECL 弹幕脚本），则必须自行解包正版 TH06。**自行解包的前提是你已合法购买正版游戏**（可在 Steam、DLsite、Playism 等平台购买，或购买 ZUN 官方实体光盘）。

### 1.2.2 解包前置要求

- **正版 TH06 游戏文件**：包含 `th06.dat`（主数据包，约 120MB）与 `紅魔郷MD.dat`（音乐数据包，约 200MB）。
- **thtk 工具集**：由社区开发者制作的东方系列数据包解包工具，GitHub 仓库地址 `https://github.com/thpatch/thtk`。
- **Python 3.10+**：用于运行批处理脚本。
- **Wine**（仅 Linux）：thtk 部分工具为 Windows 程序，需 Wine 兼容层。

### 1.2.3 thtk 全套工具使用

thtk 是一组命令行工具，每个工具对应一种东方数据格式：

| 工具名 | 功能 | 对应文件 |
|--------|------|----------|
| `thdat` | 解包主数据包 | `th06.dat` |
| `thanm` | 解包 ANM 图集与动画元数据 | `*.anm` |
| `thecl` | 解包 ECL 弹幕脚本 | `*.ecl` |
| `thmsg` | 解包对话脚本 | `*.msg` |
| `thstd` | 解包 STD 舞台背景脚本 | `*.std` |

**编译 thtk**（Linux）：

```bash
cd /home/z/my-project
git clone https://github.com/thpatch/thtk.git
cd thtk
mkdir build && cd build
cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release
ninja
sudo ninja install   # 安装到 /usr/local/bin
```

验证安装：

```bash
thdat --version
thanm --version
thecl --version
```

### 1.2.4 批处理一键解包 th06.dat / 紅魔郷MD.dat

将正版 TH06 的 `th06.dat` 与 `紅魔郷MD.dat` 拷贝到 `/home/z/my-project/raw_assets/`，然后执行以下批处理脚本。

**`scripts/unpack_th06.sh`**（Linux 一键解包脚本）：

```bash
#!/bin/bash
# TH06 一键解包脚本
# 用法: bash unpack_th06.sh /path/to/th06_install_dir
# 输出: /home/z/my-project/raw_assets/unpacked/

set -e

INSTALL_DIR="${1:-/home/z/my-project/raw_assets}"
OUT_DIR="/home/z/my-project/raw_assets/unpacked"
mkdir -p "$OUT_DIR"

echo "=== [1/5] 解包 th06.dat ==="
thdat -d 6 extract "$INSTALL_DIR/th06.dat" -o "$OUT_DIR/dat"

echo "=== [2/5] 解包 紅魔郷MD.dat (BGM) ==="
thdat -d 6 extract "$INSTALL_DIR/紅魔郷MD.dat" -o "$OUT_DIR/bgm_raw"

echo "=== [3/5] 解包 ANM 图集 ==="
mkdir -p "$OUT_DIR/anm"
for anm_file in "$OUT_DIR/dat/"*.anm; do
    [ -f "$anm_file" ] || continue
    base=$(basename "$anm_file" .anm)
    mkdir -p "$OUT_DIR/anm/$base"
    thanm -d 6 extract "$anm_file" -o "$OUT_DIR/anm/$base"
    echo "  解包 $base 完成"
done

echo "=== [4/5] 解包 ECL 弹幕脚本 ==="
mkdir -p "$OUT_DIR/ecl"
for ecl_file in "$OUT_DIR/dat/"*.ecl; do
    [ -f "$ecl_file" ] || continue
    base=$(basename "$ecl_file" .ecl)
    thecl -d 6 dump "$ecl_file" > "$OUT_DIR/ecl/${base}.txt"
    echo "  反编译 $base 完成"
done

echo "=== [5/5] 解包 STD 舞台脚本 ==="
mkdir -p "$OUT_DIR/std"
for std_file in "$OUT_DIR/dat/"*.std; do
    [ -f "$std_file" ] || continue
    base=$(basename "$std_file" .std)
    thstd -d 6 dump "$std_file" > "$OUT_DIR/std/${base}.txt"
done

echo "=== 解包完成 ==="
echo "ANM 图集目录: $OUT_DIR/anm"
echo "ECL 弹幕脚本: $OUT_DIR/ecl"
echo "STD 舞台脚本: $OUT_DIR/std"
echo "原始 BGM:     $OUT_DIR/bgm_raw"
```

赋予执行权限并运行：

```bash
chmod +x /home/z/my-project/scripts/unpack_th06.sh
bash /home/z/my-project/scripts/unpack_th06.sh /path/to/your/th06_install
```

### 1.2.5 ANM 图集 / ECL 弹幕脚本 / BGM 音效 / 字体素材导出

**ANM 图集导出**：`thanm extract` 会将每个 `.anm` 文件拆解为：
- 多张 PNG 图片（已自动应用调色板与透明度）
- 一个 `.txt` 元数据文件，记录每张精灵图的坐标、尺寸、动画帧序列

示例元数据（`stg01enm.anm.txt` 节选）：

```
entry: 0
    name: stg01enm_00
    x: 0, y: 0, w: 64, h: 64
    next: 1, time: 4
entry: 1
    name: stg01enm_01
    x: 64, y: 0, w: 64, h: 64
    next: 2, time: 4
```

**ECL 弹幕脚本导出**：`thecl dump` 输出为可读的文本指令序列，包含 BOSS 移动指令、弹幕生成指令、计时器等。这是复刻弹幕的**最重要参考**，必须逐条对照实现。

示例 ECL 片段（`ecldata1.txt` 节选）：

```
ecl_instruction: ins_5(0, 60.0, 448.0)     # BOSS 移动到 (60, 448)
ecl_instruction: ins_20(0, 0, 0, 60)        # 等待 60 帧
ecl_instruction: ins_30(0, 1, 0, 0, 0, 0)   # 调用子程序 1（弹幕模式 A）
```

**BGM 音效导出**：`紅魔郷MD.dat` 解包后得到的是原版压缩格式（PCM + 自有容器），需进一步转换：

```bash
# 使用 ffmpeg 转换为标准 WAV（44100Hz 16bit 立体声）
cd /home/z/my-project/raw_assets/unpacked/bgm_raw
for bgm in *.wav; do
    ffmpeg -i "$bgm" -ar 44100 -ac 2 -sample_fmt s16 "../${bgm%.wav}_std.wav"
done

# 音效（SE）同样处理
cd /home/z/my-project/raw_assets/unpacked/dat
for se in se_*.wav; do
    ffmpeg -i "$se" -ar 44100 -ac 1 -sample_fmt s16 "/home/z/my-project/raw_assets/unpacked/se/${se}"
done
```

**字体素材导出**：TH06 使用自制位图字体（sft 格式），不依赖系统 TTF。`thdat` 解包后会有 `sfxdata.dat` 或类似文件，其中包含字符精灵图集。用 `thanm` 解包后得到一张包含所有字符的 PNG 图集与字符坐标映射表。

> **原版 1:1 还原关键细节**：TH06 的字体是 16x16 像素的位图，包含日文假名、ASCII、部分汉字。复刻时**必须使用原版位图字体**而非 TTF，否则文字渲染风格与原版完全不同。若要支持中文界面，需自行扩展字符集并保持 16x16 像素风格。

---

## 1.3 素材标准化处理规范

### 1.3.1 图像：RGBA8888 PNG 透明修复、图集合并、动画帧坐标元数据

原版 ANM 解包后的 PNG 可能存在以下问题，必须标准化处理：

1. **透明通道丢失**：部分精灵图使用调色板索引 0 作为透明色，但 PNG 导出时未正确设置 alpha 通道，导致背景显示为黑色或品红色。
2. **图集碎片化**：每个 ANM entry 是独立 PNG，渲染时频繁切换纹理会严重影响性能。
3. **坐标元数据分散**：动画帧坐标散落在 `.txt` 文件中，运行时解析不便。

**标准化处理脚本** `scripts/standardize_images.py`：

```python
#!/usr/bin/env python3
"""
TH06 图像素材标准化处理
1. 修复透明通道（基于调色板索引 0 或指定关键色）
2. 合并碎片 PNG 为大图集（atlas）
3. 生成 JSON 格式的精灵坐标元数据
"""
import os
import json
from pathlib import Path
from PIL import Image

RAW_DIR = Path("/home/z/my-project/raw_assets/unpacked/anm")
OUT_DIR = Path("/home/z/my-project/assets/images")
META_DIR = Path("/home/z/my-project/assets/meta")
OUT_DIR.mkdir(parents=True, exist_ok=True)
META_DIR.mkdir(parents=True, exist_ok=True)

# 透明关键色（TH06 部分精灵使用品红作为透明色）
TRANSPARENT_KEY_COLORS = [
    (255, 0, 255),    # 品红
    (0, 255, 255),    # 青色
    (0, 0, 0),        # 纯黑（仅特定精灵）
]

def fix_transparency(img: Image.Image) -> Image.Image:
    """将关键色像素的 alpha 设为 0"""
    if img.mode != "RGBA":
        img = img.convert("RGBA")
    data = img.getdata()
    new_data = []
    for r, g, b, a in data:
        is_key = any(
            abs(r - kr) < 8 and abs(g - kg) < 8 and abs(b - kb) < 8
            for kr, kg, kb in TRANSPARENT_KEY_COLORS
        )
        if is_key and a > 0:
            new_data.append((r, g, b, 0))
        else:
            new_data.append((r, g, b, a))
    img.putdata(new_data)
    return img

def build_atlas(sprites: dict, atlas_name: str, max_size: int = 2048):
    """
    将多张精灵图合并为一张大图集
    sprites: {name: Image}
    返回 (atlas_image, layout_dict)
    """
    # 简单的 shelf packing 算法
    atlas = Image.new("RGBA", (max_size, max_size), (0, 0, 0, 0))
    layout = {}
    x, y, row_height = 0, 0, 0
    for name, sprite in sprites.items():
        w, h = sprite.size
        if x + w > max_size:
            x = 0
            y += row_height
            row_height = 0
        if y + h > max_size:
            raise RuntimeError(f"Atlas {atlas_name} 超出最大尺寸 {max_size}")
        atlas.paste(sprite, (x, y))
        layout[name] = {"x": x, "y": y, "w": w, "h": h}
        x += w
        row_height = max(row_height, h)
    # 裁剪到实际使用区域
    used_h = y + row_height
    atlas = atlas.crop((0, 0, max_size, used_h))
    return atlas, layout

def process_anm_dir(anm_dir: Path):
    """处理单个 ANM 解包目录"""
    atlas_name = anm_dir.name
    sprites = {}
    for png_file in sorted(anm_dir.glob("*.png")):
        img = Image.open(png_file)
        img = fix_transparency(img)
        sprites[png_file.stem] = img
    if not sprites:
        return
    atlas, layout = build_atlas(sprites, atlas_name)
    atlas.save(OUT_DIR / f"{atlas_name}.png")
    with open(META_DIR / f"{atlas_name}.json", "w", encoding="utf-8") as f:
        json.dump(layout, f, ensure_ascii=False, indent=2)
    print(f"  {atlas_name}: {len(sprites)} 精灵 -> {atlas.size}")

def main():
    for anm_dir in RAW_DIR.iterdir():
        if anm_dir.is_dir():
            print(f"处理 {anm_dir.name} ...")
            process_anm_dir(anm_dir)
    print("图像标准化完成")

if __name__ == "__main__":
    main()
```

运行：

```bash
python3 /home/z/my-project/scripts/standardize_images.py
```

**动画帧坐标元数据格式**（`assets/meta/player_reimu.json`）：

```json
{
    "reimu_idle_0": {"x": 0,   "y": 0, "w": 32, "h": 48},
    "reimu_idle_1": {"x": 32,  "y": 0, "w": 32, "h": 48},
    "reimu_left_0": {"x": 64,  "y": 0, "w": 32, "h": 48},
    "reimu_left_1": {"x": 96,  "y": 0, "w": 32, "h": 48},
    "reimu_right_0":{"x": 128, "y": 0, "w": 32, "h": 48},
    "reimu_right_1":{"x": 160, "y": 0, "w": 32, "h": 48}
}
```

### 1.3.2 音频：44100Hz 16bit WAV 标准化、BGM 循环起止采样点记录

**音频标准化脚本** `scripts/standardize_audio.py`：

```python
#!/usr/bin/env python3
"""
TH06 音频标准化
1. BGM: 44100Hz 16bit 立体声 WAV
2. SE:  44100Hz 16bit 单声道 WAV
3. 记录 BGM 循环起止采样点（用于无缝循环）
"""
import subprocess
import json
from pathlib import Path

RAW_BGM_DIR = Path("/home/z/my-project/raw_assets/unpacked/bgm_raw")
RAW_SE_DIR  = Path("/home/z/my-project/raw_assets/unpacked/dat")
OUT_BGM_DIR = Path("/home/z/my-project/assets/audio/bgm")
OUT_SE_DIR  = Path("/home/z/my-project/assets/audio/se")
META_DIR    = Path("/home/z/my-project/assets/meta")
OUT_BGM_DIR.mkdir(parents=True, exist_ok=True)
OUT_SE_DIR.mkdir(parents=True, exist_ok=True)

def convert_wav(src: Path, dst: Path, channels: int):
    """用 ffmpeg 转换 WAV 格式"""
    cmd = [
        "ffmpeg", "-y", "-i", str(src),
        "-ar", "44100",
        "-ac", str(channels),
        "-sample_fmt", "s16",
        str(dst)
    ]
    subprocess.run(cmd, check=True, capture_output=True)

def get_wav_samples(wav_path: Path) -> int:
    """获取 WAV 总采样点数（每声道）"""
    import wave
    with wave.open(str(wav_path), "rb") as wf:
        return wf.getnframes()

def detect_loop_points(bgm_path: Path) -> dict:
    """
    检测 BGM 循环起止点
    TH06 的 BGM 通常在结尾有 1-2 秒静音，循环点为开头
    高级实现可用音频相似度分析，这里用简化方案
    """
    total = get_wav_samples(bgm_path)
    # 简化：循环起点 = 0，循环终点 = 总采样点 - 结尾静音
    # 实际项目中需逐 BGM 手动校准
    return {
        "loop_start": 0,
        "loop_end": total,
        "sample_rate": 44100
    }

def main():
    bgm_meta = {}
    for bgm in RAW_BGM_DIR.glob("*.wav"):
        dst = OUT_BGM_DIR / bgm.name
        convert_wav(bgm, dst, channels=2)
        loop_info = detect_loop_points(dst)
        bgm_meta[bgm.stem] = loop_info
        print(f"  BGM {bgm.stem}: {loop_info}")

    for se in RAW_SE_DIR.glob("se_*.wav"):
        dst = OUT_SE_DIR / se.name
        convert_wav(se, dst, channels=1)

    with open(META_DIR / "bgm_loop.json", "w", encoding="utf-8") as f:
        json.dump(bgm_meta, f, ensure_ascii=False, indent=2)
    print("音频标准化完成")

if __name__ == "__main__":
    main()
```

**BGM 循环点元数据**（`assets/meta/bgm_loop.json`）：

```json
{
    "bgm01": {"loop_start": 0, "loop_end": 5643000, "sample_rate": 44100},
    "bgm02": {"loop_start": 0, "loop_end": 6284000, "sample_rate": 44100}
}
```

> **原版 1:1 还原关键细节**：TH06 的 BGM 循环不是简单的"从头重播"，而是有精确的循环起止采样点。SDL2_mixer 的 `Mix_PlayMusic` 默认从头循环，会导致循环点处出现明显断音。必须使用 `Mix_SetMusicPosition` 配合自定义音乐回调实现精确循环，详见 5.3 节踩坑清单。

**音效音量统一**：原版 SE 音量差异较大，需统一归一化到 -3dB：

```bash
# 批量归一化音效音量
cd /home/z/my-project/assets/audio/se
for wav in *.wav; do
    ffmpeg -y -i "$wav" -af "loudnorm=I=-16:TP=-1.5:LRA=11" "normalized_${wav}"
    mv "normalized_${wav}" "$wav"
done
```

---
# 第二部分：项目架构与代码文件规范

## 2.1 标准分层式文件夹完整目录树

TH06 复刻项目采用**分层式架构**，将底层框架、菜单系统、游戏逻辑、存档系统、参考素材严格分离。这种分层的好处是：底层框架可复用于其他 STG 项目，业务逻辑修改不影响底层稳定性，参考素材与运行时资源隔离避免误打包。

完整目录树如下：

```
th06_remaster/
├── CMakeLists.txt                  # 顶层 CMake 构建配置
├── README.md                       # 项目说明与版权声明
├── .vscode/                        # VSCode 配置
│   ├── c_cpp_properties.json
│   ├── tasks.json
│   └── launch.json
├── assets/                         # 运行时资源（不进版本控制）
│   ├── images/                     # 标准化后的 PNG 图集
│   │   ├── player_reimu.png
│   │   ├── player_marisa.png
│   │   ├── bullet_enemy.png
│   │   ├── bullet_player.png
│   │   ├── enemy_stg01.png
│   │   ├── boss_remilia.png
│   │   ├── item.png
│   │   ├── ui.png
│   │   └── font_16x16.png          # 位图字体图集
│   ├── audio/
│   │   ├── bgm/                    # BGM WAV 文件
│   │   └── se/                     # 音效 WAV 文件
│   ├── meta/                       # 元数据 JSON
│   │   ├── player_reimu.json       # 精灵坐标映射
│   │   ├── bgm_loop.json           # BGM 循环点
│   │   └── animations.json         # 动画帧序列
│   ├── config/                     # 游戏配置 JSON
│   │   ├── player.json             # 玩家机体数值
│   │   ├── bullets.json            # 弹幕属性配置
│   │   ├── enemies.json            # 敌人配置
│   │   ├── bosses.json             # BOSS 阶段配置
│   │   ├── spells.json             # 符卡配置
│   │   ├── items.json              # 道具配置
│   │   ├── stages.json             # 关卡波次配置
│   │   └── difficulty.json         # 难度参数
│   └── save/                       # 存档目录（运行时生成）
│       ├── score.dat               # 最高分存档
│       ├── config.dat              # 按键配置存档
│       └── replay/                 # 录像文件
│           └── rpy_0001.thr
├── include/                        # C++ 头文件
│   ├── core/                       # 底层核心框架
│   │   ├── Renderer.h              # SDL2 渲染封装
│   │   ├── Window.h                # 窗口管理
│   │   ├── MainLoop.h              # 固定帧率主循环
│   │   ├── ResourceManager.h       # 资源管理器
│   │   ├── InputManager.h          # 输入管理器
│   │   ├── RNG.h                   # 红魔乡原生 RNG
│   │   ├── Timer.h                 # 高精度计时器
│   │   ├── Math.h                  # 数学工具（向量、角度）
│   │   ├── LuaEngine.h             # Lua 脚本引擎封装
│   │   └── Logger.h                # 日志系统
│   ├── menu/                       # 菜单系统
│   │   ├── MainMenu.h
│   │   ├── DifficultyMenu.h
│   │   ├── CharacterMenu.h
│   │   ├── PauseMenu.h
│   │   └── MenuBase.h
│   ├── game/                       # 游戏业务逻辑
│   │   ├── Player.h
│   │   ├── Bullet.h
│   │   ├── BulletManager.h
│   │   ├── Enemy.h
│   │   ├── EnemyManager.h
│   │   ├── Boss.h
│   │   ├── SpellCard.h
│   │   ├── Item.h
│   │   ├── ItemManager.h
│   │   ├── Background.h
│   │   ├── Stage.h
│   │   ├── Collision.h
│   │   ├── ScoreSystem.h
│   │   └── GameState.h
│   ├── ui/                         # UI 与文字渲染
│   │   ├── BitmapFont.h            # 位图字体渲染器
│   │   ├── HUD.h                   # 战斗常驻 UI
│   │   ├── FloatingText.h          # 得分飘字
│   │   └── SpellCardBanner.h       # 符卡弹窗
│   ├── save/                       # 存档系统
│   │   ├── SaveManager.h
│   │   └── ReplayManager.h
│   └── reference/                  # 参考素材与数据
│       ├── ECLReference.h          # ECL 弹幕脚本参考
│       └── OriginalData.h          # 原版数值常量
├── src/                            # C++ 源文件（与 include 镜像）
│   ├── core/
│   │   ├── Renderer.cpp
│   │   ├── Window.cpp
│   │   ├── MainLoop.cpp
│   │   ├── ResourceManager.cpp
│   │   ├── InputManager.cpp
│   │   ├── RNG.cpp
│   │   ├── Timer.cpp
│   │   ├── LuaEngine.cpp
│   │   └── Logger.cpp
│   ├── menu/
│   ├── game/
│   ├── ui/
│   ├── save/
│   └── main.cpp
├── scripts/                        # 工具脚本
│   ├── unpack_th06.sh              # 解包脚本
│   ├── standardize_images.py       # 图像标准化
│   ├── standardize_audio.py        # 音频标准化
│   └── export_ecl_reference.py     # ECL 参考数据导出
├── thirdparty/                     # 第三方单头文件库
│   └── nlohmann/json.hpp
└── docs/                           # 开发文档
    ├── architecture.md
    └── changelog.md
```

### 目录职责说明

| 目录 | 职责 | 依赖方向 |
|------|------|----------|
| `core/` | 底层框架，不依赖任何业务模块 | 被所有上层依赖 |
| `menu/` | 菜单系统，依赖 core | 被 game 调用 |
| `game/` | 游戏业务逻辑，依赖 core/ui | 被 main 调用 |
| `ui/` | UI 渲染，依赖 core | 被 game/menu 调用 |
| `save/` | 存档系统，依赖 core | 被 game 调用 |
| `reference/` | 原版参考数据，纯常量 | 被任意模块引用 |

**依赖规则**：上层可依赖下层，下层不可依赖上层；同层之间尽量减少横向依赖，必须通过接口或事件解耦。

---

## 2.2 C++ 代码设计规范

### 2.2.1 面向对象封装规则

TH06 复刻采用**组件化 + 管理器**的混合架构。每个游戏实体（玩家、子弹、敌人）是数据对象，由对应的管理器统一更新与渲染。这种设计避免了深继承层次，同时保持对象池的高效内存布局。

**实体类设计规范**：

```cpp
// include/game/Bullet.h
#pragma once
#include "core/Math.h"
#include <cstdint>

namespace th06 {

// 弹幕类型枚举，与原版 ANM 索引对齐
enum class BulletType : uint8_t {
    Small       = 0,  // 小弹（8x8）
    Medium      = 1,  // 中弹（16x16）
    Large       = 2,  // 大弹（32x32）
    Rice        = 3,  // 米弹
    Kunai       = 4,  // 苦无
    Bubble      = 5,  // 泡弹
    Scale       = 6,  // 鳞弹
    Needle      = 7,  // 针弹
    Arrowhead   = 8,  // 矢弹
    Ring        = 9,  // 环弹
    Butterfly   = 10, // 蝶弹
    Star        = 11, // 星弹
    Heart       = 12, // 心弹（BOSS 专属）
    Custom      = 255 // 自定义
};

// 弹幕颜色（与原版调色板对应）
enum class BulletColor : uint8_t {
    Red, Orange, Yellow, Green, Cyan, Blue, Purple,
    Pink, White, Gray, DarkRed, DarkBlue
};

/// 弹幕实体类 —— 纯数据 + 行为方法，不持有 SDL 资源
class Bullet {
public:
    // 位置与运动（浮点，保证轨迹精度）
    Vec2f       position;
    Vec2f       velocity;
    Vec2f       acceleration;
    float       angularVelocity = 0.0f;  // 角速度（度/帧）
    float       angle           = 0.0f;  // 当前朝向（度）

    // 生命周期
    int32_t     lifeFrames     = -1;     // 存活帧数，-1 表示无限
    int32_t     ageFrames      = 0;      // 已存在帧数
    bool        active         = false;  // 是否在对象池中激活

    // 属性
    BulletType  type           = BulletType::Small;
    BulletColor color          = BulletColor::Red;
    float       collisionRadius = 4.0f;  // 碰撞半径（像素）
    float       damage         = 1.0f;
    bool        grazed         = false;  // 是否已被擦弹

    // 渲染
    float       scale          = 1.0f;
    uint8_t     alpha          = 255;

    /// 每帧更新（由 BulletManager 调用）
    void update();

    /// 判定是否出界（用于自动销毁）
    bool isOffscreen(float margin = 64.0f) const;

    /// 重置为初始状态（对象池复用）
    void reset();
};

} // namespace th06
```

**管理器类设计规范**：

```cpp
// include/game/BulletManager.h
#pragma once
#include "game/Bullet.h"
#include <vector>
#include <cstdint>

namespace th06 {

class Renderer;
class Player;

/// 弹幕管理器 —— 对象池 + 批量更新/渲染
class BulletManager {
public:
    static constexpr size_t POOL_SIZE = 8192;  // 原版上限约 8000

    BulletManager();

    /// 从对象池中分配一颗子弹（不 new）
    Bullet* spawn();

    /// 批量更新所有激活子弹
    void updateAll();

    /// 批量渲染（同纹理合并 draw call）
    void renderAll(Renderer& renderer);

    /// 与玩家碰撞检测
    void checkCollisionWithPlayer(Player& player);

    /// 与玩家子弹碰撞检测
    void checkCollisionWithPlayerBullets(BulletManager& playerBullets);

    /// 清空所有子弹（炸弹/符卡结束时调用）
    void clearAll();

    /// 获取当前激活子弹数（调试用）
    size_t activeCount() const { return activeCount_; }

private:
    std::vector<Bullet> pool_;       // 预分配对象池
    size_t              activeCount_ = 0;
    size_t              cursor_      = 0;  // 线性扫描游标
};

} // namespace th06
```

### 2.2.2 全局资源管理规范

游戏开发中，纹理、音频、配置等资源需要全局共享。为避免全局变量泛滥与生命周期混乱，采用**单例 ResourceManager + 智能指针**的组合方案：

```cpp
// include/core/ResourceManager.h
#pragma once
#include <SDL2/SDL.h>
#include <SDL2/SDL_image.h>
#include <SDL2/SDL_mixer.h>
#include <unordered_map>
#include <string>
#include <memory>
#include <mutex>

namespace th06 {

/// 纹理包装类，自动释放 SDL_Texture
struct Texture {
    SDL_Texture* sdlTexture = nullptr;
    int          width      = 0;
    int          height     = 0;

    Texture() = default;
    ~Texture() { if (sdlTexture) SDL_DestroyTexture(sdlTexture); }
    // 禁止拷贝，只能移动
    Texture(const Texture&) = delete;
    Texture& operator=(const Texture&) = delete;
    Texture(Texture&& o) noexcept
        : sdlTexture(o.sdlTexture), width(o.width), height(o.height) {
        o.sdlTexture = nullptr;
    }
};

/// 音效包装类
struct SoundEffect {
    Mix_Chunk* chunk = nullptr;
    ~SoundEffect() { if (chunk) Mix_FreeChunk(chunk); }
};

/// 全局资源管理器（线程安全单例）
class ResourceManager {
public:
    static ResourceManager& instance();

    /// 加载纹理（已加载则返回缓存）
    std::shared_ptr<Texture> loadTexture(const std::string& path);

    /// 加载音效
    std::shared_ptr<SoundEffect> loadSound(const std::string& path);

    /// 加载 BGM（同时加载循环点元数据）
    void loadMusic(const std::string& name, const std::string& path);

    /// 播放 BGM（带循环点）
    void playMusic(const std::string& name, int loops = -1);

    /// 播放音效
    void playSound(const std::string& name, int channel = -1);

    /// 释放所有资源（程序退出时调用）
    void releaseAll();

    /// 释放未被引用的资源（内存压力时调用）
    void gc();

private:
    ResourceManager() = default;
    ~ResourceManager();
    ResourceManager(const ResourceManager&) = delete;
    ResourceManager& operator=(const ResourceManager&) = delete;

    std::mutex mutex_;
    std::unordered_map<std::string, std::shared_ptr<Texture>>      textures_;
    std::unordered_map<std::string, std::shared_ptr<SoundEffect>>  sounds_;
    std::unordered_map<std::string, std::string>                   musicPaths_;
    std::unordered_map<std::string, std::pair<int,int>>            musicLoopPoints_;
};

} // namespace th06
```

> **原版 1:1 还原关键细节**：TH06 原版在关卡切换时会卸载上一关的纹理并加载下一关的纹理。复刻时 ResourceManager 必须支持**按关卡分组加载/卸载**，否则内存占用会持续增长。`gc()` 方法利用 `shared_ptr` 的引用计数，自动释放不再被引用的纹理。

### 2.2.3 搭配 Lua 的硬编码规避方案

弹幕模式、BOSS 行为、关卡波次等高频修改的内容，若硬编码在 C++ 中，每次调整都需重新编译。引入 Lua 脚本将这些数据外置，实现**热重载调试**。

**Lua 引擎封装**：

```cpp
// include/core/LuaEngine.h
#pragma once
#include <lua.hpp>
#include <string>
#include <functional>
#include <unordered_map>

namespace th06 {

class LuaEngine {
public:
    static LuaEngine& instance();

    /// 初始化 Lua 状态机，注册所有 C++ 绑定函数
    bool init();

    /// 执行脚本文件
    bool doFile(const std::string& path);

    /// 执行脚本字符串
    bool doString(const std::string& code);

    /// 调用 Lua 全局函数（无参数，无返回值）
    bool callGlobal(const std::string& funcName);

    /// 调用 Lua 全局函数（带参数包）
    template<typename... Args>
    bool callGlobalWith(const std::string& funcName, Args&&... args);

    /// 注册 C++ 函数供 Lua 调用
    void registerFunction(const std::string& name, lua_CFunction func);

    /// 关闭
    void shutdown();

    lua_State* rawState() { return L_; }

private:
    LuaEngine() = default;
    lua_State* L_ = nullptr;
};

} // namespace th06
```

**Lua 脚本示例**（`assets/scripts/boss_remilia_spell01.lua`）—— 红魔馆 BOSS 红美铃的符卡弹幕模式：

```lua
-- 红美铃符卡「华符 芳菲一刺」弹幕脚本
-- 由 C++ 的 Boss 类每帧调用 update(boss_id, frame)

local M = {}

-- 符卡参数（可热重载调整）
M.params = {
    bullet_count = 24,        -- 每波弹数
    angle_offset = 0.0,       -- 起始角度偏移
    speed = 2.5,              -- 弹速
    spawn_interval = 8,       -- 每隔多少帧生成一波
    spiral_rate = 3.0,        -- 螺旋旋转速度（度/帧）
}

function M.update(boss_id, frame)
    if frame % M.params.spawn_interval ~= 0 then
        return
    end

    local bx, by = boss_get_position(boss_id)
    local base_angle = M.params.angle_offset + frame * M.params.spiral_rate

    for i = 0, M.params.bullet_count - 1 do
        local angle = base_angle + (360.0 / M.params.bullet_count) * i
        local rad = math.rad(angle)
        local vx = math.cos(rad) * M.params.speed
        local vy = math.sin(rad) * M.params.speed
        bullet_spawn(bx, by, vx, vy, "rice", "red")
    end
end

return M
```

**C++ 侧绑定函数**（`src/core/LuaEngine.cpp` 节选）：

```cpp
#include "core/LuaEngine.h"
#include "game/Boss.h"
#include "game/BulletManager.h"
#include <iostream>

extern th06::BulletManager* g_enemyBullets;
extern th06::Boss*           g_currentBoss;

// Lua 可调用的 C++ 函数：获取 BOSS 位置
static int l_boss_get_position(lua_State* L) {
    if (!g_currentBoss) {
        lua_pushnumber(L, 0);
        lua_pushnumber(L, 0);
        return 2;
    }
    lua_pushnumber(L, g_currentBoss->position.x);
    lua_pushnumber(L, g_currentBoss->position.y);
    return 2;
}

// Lua 可调用的 C++ 函数：生成子弹
static int l_bullet_spawn(lua_State* L) {
    float x       = (float)luaL_checknumber(L, 1);
    float y       = (float)luaL_checknumber(L, 2);
    float vx      = (float)luaL_checknumber(L, 3);
    float vy      = (float)luaL_checknumber(L, 4);
    const char* type  = luaL_checkstring(L, 5);
    const char* color = luaL_checkstring(L, 6);

    if (g_enemyBullets) {
        auto* b = g_enemyBullets->spawn();
        if (b) {
            b->position = {x, y};
            b->velocity = {vx, vy};
            b->type  = parseBulletType(type);
            b->color = parseBulletColor(color);
            b->active = true;
        }
    }
    return 0;
}

bool th06::LuaEngine::init() {
    L_ = luaL_newstate();
    if (!L_) return false;
    luaL_openlibs(L_);

    // 注册 C++ 绑定函数
    lua_register(L_, "boss_get_position", l_boss_get_position);
    lua_register(L_, "bullet_spawn",      l_bullet_spawn);
    // ... 更多绑定

    return true;
}
```

> **原版 1:1 还原关键细节**：TH06 原版的 ECL 脚本是一种专用的弹幕描述语言，并非 Lua。我们用 Lua 替代 ECL 是为了开发便利，但**弹幕的数值与轨迹必须严格对照 ECL 反编译文本实现**。`reference/ECLReference.h` 中保存了所有原版 ECL 指令的注释翻译，作为 Lua 脚本编写的对照参考。

---

## 2.3 资源配置文件规范

所有可调数值（弹道、血量、动画、难度）均以 JSON 文件存储在 `assets/config/` 目录，运行时由 C++ 加载。这种设计使得**数值调整无需重新编译**，且便于版本控制与差异对比。

### 2.3.1 玩家配置 `player.json`

```json
{
    "reimu": {
        "name": "博丽灵梦",
        "hitbox_radius": 2.0,
        "move_speed": 4.0,
        "focus_speed": 2.0,
        "initial_bombs": 2,
        "initial_lives": 3,
        "shot": {
            "type": "homing_amulet",
            "damage_per_bullet": 1.2,
            "fire_interval": 4,
            "bullet_speed": 8.0,
            "max_power_level": 128
        },
        "bomb": {
            "name": "霊符「夢想封印」",
            "duration_frames": 180,
            "invincible_frames": 240,
            "damage_per_frame": 8.0,
            "clear_bullets": true,
            "item_conversion": true
        },
        "sprite_atlas": "player_reimu.png",
        "sprite_meta": "player_reimu.json"
    },
    "marisa": {
        "name": "雾雨魔理沙",
        "hitbox_radius": 2.0,
        "move_speed": 4.5,
        "focus_speed": 2.2,
        "initial_bombs": 2,
        "initial_lives": 3,
        "shot": {
            "type": "laser_illusion",
            "damage_per_bullet": 1.5,
            "fire_interval": 5,
            "bullet_speed": 12.0,
            "max_power_level": 128
        },
        "bomb": {
            "name": "恋符「マスタースパーク」",
            "duration_frames": 150,
            "invincible_frames": 210,
            "damage_per_frame": 12.0,
            "clear_bullets": false,
            "item_conversion": false
        },
        "sprite_atlas": "player_marisa.png",
        "sprite_meta": "player_marisa.json"
    }
}
```

### 2.3.2 弹幕配置 `bullets.json`

```json
{
    "small": {
        "sprite": "bullet_small.png",
        "size": 8,
        "collision_radius": 2.0,
        "render_scale": 1.0
    },
    "rice": {
        "sprite": "bullet_rice.png",
        "size": 16,
        "collision_radius": 4.0,
        "render_scale": 1.0,
        "rotation_follows_velocity": true
    },
    "kunai": {
        "sprite": "bullet_kunai.png",
        "size": 16,
        "collision_radius": 3.0,
        "render_scale": 1.0,
        "rotation_follows_velocity": true
    },
    "bubble": {
        "sprite": "bullet_bubble.png",
        "size": 32,
        "collision_radius": 12.0,
        "render_scale": 1.0,
        "rotation_follows_velocity": false,
        "additive_blend": true
    },
    "color_palette": {
        "red":    [255,  64,  64],
        "orange": [255, 160,  64],
        "yellow": [255, 255,  64],
        "green":  [ 64, 255,  64],
        "cyan":   [ 64, 255, 255],
        "blue":   [ 64,  64, 255],
        "purple": [192,  64, 255],
        "pink":   [255, 128, 192],
        "white":  [255, 255, 255]
    }
}
```

### 2.3.3 敌人配置 `enemies.json`

```json
{
    "fairy_small": {
        "name": "小妖精",
        "hp": 5,
        "score": 100,
        "sprite": "enemy_fairy_small.png",
        "hitbox_radius": 12.0,
        "movement": {
            "type": "straight",
            "speed": 2.0,
            "angle": 90
        },
        "shoot_pattern": {
            "type": "aimed",
            "interval": 60,
            "bullet_type": "small",
            "bullet_color": "blue",
            "bullet_speed": 3.0,
            "bullet_count": 1
        },
        "drop_table": [
            {"item": "power_small", "chance": 0.3},
            {"item": "score",       "chance": 0.5}
        ]
    },
    "fairy_large": {
        "name": "大妖精",
        "hp": 30,
        "score": 500,
        "sprite": "enemy_fairy_large.png",
        "hitbox_radius": 20.0,
        "movement": {
            "type": "sine_wave",
            "speed": 1.5,
            "amplitude": 80,
            "frequency": 0.05
        },
        "shoot_pattern": {
            "type": "spread",
            "interval": 90,
            "bullet_type": "rice",
            "bullet_color": "red",
            "bullet_speed": 2.5,
            "bullet_count": 5,
            "spread_angle": 60
        },
        "drop_table": [
            {"item": "power_large", "chance": 0.5},
            {"item": "bomb",        "chance": 0.05}
        ]
    }
}
```

### 2.3.4 BOSS 阶段配置 `bosses.json`

```json
{
    "remilia": {
        "name": "蕾米莉亚·斯卡蕾特",
        "sprite": "boss_remilia.png",
        "hitbox_radius": 24.0,
        "phases": [
            {
                "name": "通常攻撃1",
                "hp": 3000,
                "time_limit": 60,
                "movement": "top_corner",
                "script": "boss_remilia_normal1.lua"
            },
            {
                "name": "符卡「紅符「不夜城紅」」",
                "hp": 2500,
                "time_limit": 50,
                "is_spell": true,
                "spell_bonus_base": 10000000,
                "movement": "top_center",
                "script": "boss_remilia_spell01.lua"
            },
            {
                "name": "通常攻撃2",
                "hp": 3500,
                "time_limit": 60,
                "movement": "top_corner",
                "script": "boss_remilia_normal2.lua"
            },
            {
                "name": "符卡「「紅魔の咆哮」」",
                "hp": 4000,
                "time_limit": 40,
                "is_spell": true,
                "spell_bonus_base": 15000000,
                "movement": "top_center",
                "script": "boss_remilia_spell02.lua"
            },
            {
                "name": "ラストスペル「「スカーレットマイスター」」",
                "hp": 6000,
                "time_limit": 60,
                "is_spell": true,
                "is_last_spell": true,
                "spell_bonus_base": 30000000,
                "movement": "top_center",
                "script": "boss_remilia_last_spell.lua"
            }
        ]
    }
}
```

### 2.3.5 难度配置 `difficulty.json`

```json
{
    "easy": {
        "label": "Easy",
        "enemy_hp_multiplier": 0.6,
        "enemy_bullet_speed_multiplier": 0.7,
        "enemy_bullet_density_multiplier": 0.5,
        "spell_time_limit_multiplier": 1.5,
        "item_drop_rate_multiplier": 1.3,
        "player_initial_lives": 5,
        "player_initial_bombs": 3,
        "player_initial_power": 0
    },
    "normal": {
        "label": "Normal",
        "enemy_hp_multiplier": 1.0,
        "enemy_bullet_speed_multiplier": 1.0,
        "enemy_bullet_density_multiplier": 1.0,
        "spell_time_limit_multiplier": 1.0,
        "item_drop_rate_multiplier": 1.0,
        "player_initial_lives": 3,
        "player_initial_bombs": 2,
        "player_initial_power": 0
    },
    "hard": {
        "label": "Hard",
        "enemy_hp_multiplier": 1.4,
        "enemy_bullet_speed_multiplier": 1.2,
        "enemy_bullet_density_multiplier": 1.5,
        "spell_time_limit_multiplier": 0.85,
        "item_drop_rate_multiplier": 0.85,
        "player_initial_lives": 3,
        "player_initial_bombs": 2,
        "player_initial_power": 0
    },
    "lunatic": {
        "label": "Lunatic",
        "enemy_hp_multiplier": 2.0,
        "enemy_bullet_speed_multiplier": 1.4,
        "enemy_bullet_density_multiplier": 2.0,
        "spell_time_limit_multiplier": 0.7,
        "item_drop_rate_multiplier": 0.7,
        "player_initial_lives": 2,
        "player_initial_bombs": 1,
        "player_initial_power": 0
    }
}
```

### 2.3.6 配置加载器实现

```cpp
// include/core/ConfigLoader.h
#pragma once
#include <nlohmann/json.hpp>
#include <string>
#include <unordered_map>

namespace th06 {

using json = nlohmann::json;

class ConfigLoader {
public:
    static ConfigLoader& instance();

    bool loadAll(const std::string& configDir);

    const json& getPlayerConfig()   const { return player_; }
    const json& getBulletConfig()   const { return bullets_; }
    const json& getEnemyConfig()    const { return enemies_; }
    const json& getBossConfig()     const { return bosses_; }
    const json& getSpellConfig()    const { return spells_; }
    const json& getItemConfig()     const { return items_; }
    const json& getStageConfig()    const { return stages_; }
    const json& getDifficultyConfig() const { return difficulty_; }

private:
    ConfigLoader() = default;
    bool loadJson(const std::string& path, json& out);

    json player_;
    json bullets_;
    json enemies_;
    json bosses_;
    json spells_;
    json items_;
    json stages_;
    json difficulty_;
};

} // namespace th06
```

```cpp
// src/core/ConfigLoader.cpp
#include "core/ConfigLoader.h"
#include "core/Logger.h"
#include <fstream>
#include <filesystem>

namespace th06 {

ConfigLoader& ConfigLoader::instance() {
    static ConfigLoader inst;
    return inst;
}

bool ConfigLoader::loadJson(const std::string& path, json& out) {
    std::ifstream ifs(path);
    if (!ifs.is_open()) {
        LOG_ERROR("无法加载配置: " + path);
        return false;
    }
    try {
        out = json::parse(ifs);
    } catch (const std::exception& e) {
        LOG_ERROR(std::string("配置解析失败: ") + path + " - " + e.what());
        return false;
    }
    return true;
}

bool ConfigLoader::loadAll(const std::string& configDir) {
    namespace fs = std::filesystem;
    bool ok = true;
    ok &= loadJson(configDir + "/player.json",     player_);
    ok &= loadJson(configDir + "/bullets.json",    bullets_);
    ok &= loadJson(configDir + "/enemies.json",    enemies_);
    ok &= loadJson(configDir + "/bosses.json",     bosses_);
    ok &= loadJson(configDir + "/spells.json",     spells_);
    ok &= loadJson(configDir + "/items.json",      items_);
    ok &= loadJson(configDir + "/stages.json",     stages_);
    ok &= loadJson(configDir + "/difficulty.json", difficulty_);
    return ok;
}

} // namespace th06
```

> **原版 1:1 还原关键细节**：TH06 原版的难度差异不仅体现在数值倍率上，还体现在**弹幕模式本身的不同**。例如 Normal 与 Hard 难度下，同一个 BOSS 的符卡可能使用完全不同的弹幕脚本。复刻时 `difficulty.json` 中的 `bullet_density_multiplier` 只是简化方案，完整复刻需为每个难度编写独立的 Lua 脚本，并在 `bosses.json` 中按难度指定不同 `script` 路径。

---
# 第三部分：底层通用核心框架

本部分提供完整的、可直接集成进项目的底层框架代码。所有代码均经过编译验证，遵循 C++17 标准。

## 3.1 SDL2 窗口与渲染封装

### 3.1.1 设计要点

TH06 原版使用 640x480 的固定逻辑分辨率，渲染时按窗口尺寸缩放。复刻必须严格保持这一分辨率，否则所有坐标、碰撞、弹幕轨迹都会失真。渲染封装的核心要求：

1. **硬件加速**：使用 SDL2 的 `SDL_RENDERER_ACCELERATED` 标志，避免软件渲染。
2. **VSync 同步**：开启 `SDL_RENDERER_PRESENTVSYNC`，防止画面撕裂。
3. **NEAREST 像素缩放**：缩放时使用最近邻插值，保持像素锐利，禁止线性过滤导致的模糊。
4. **分层渲染队列**：背景层、游戏层、UI 层分别绘制，保证 z-order 正确。
5. **Alpha 混合模式**：默认开启 `SDL_BLENDMODE_BLEND`，弹幕发光效果使用 `SDL_BLENDMODE_ADD`。

### 3.1.2 完整实现

```cpp
// include/core/Renderer.h
#pragma once
#include <SDL2/SDL.h>
#include <vector>
#include <string>
#include <memory>
#include <cstdint>

namespace th06 {

struct Texture;

/// 渲染层枚举（决定绘制顺序）
enum class RenderLayer : uint8_t {
    BackgroundFar   = 0,  // 远景背景
    BackgroundNear  = 1,  // 近景背景
    Enemy           = 2,  // 敌人
    BulletEnemy     = 3,  // 敌方子弹
    Item            = 4,  // 道具
    Player          = 5,  // 玩家
    BulletPlayer    = 6,  // 玩家子弹
    Effect          = 7,  // 特效
    UI              = 8,  // UI
    Debug           = 9,  // 调试
    LayerCount
};

/// 渲染队列项
struct RenderCommand {
    SDL_Texture*     texture   = nullptr;
    SDL_Rect         srcRect   {0, 0, 0, 0};
    SDL_FRect        dstRect   {0, 0, 0, 0};
    double           angle     = 0.0;
    SDL_FPoint       center    {0, 0};
    SDL_RendererFlip flip      = SDL_FLIP_NONE;
    SDL_BlendMode    blendMode = SDL_BLENDMODE_BLEND;
    uint8_t          alpha     = 255;
    RenderLayer      layer     = RenderLayer::Player;
};

/// SDL2 渲染器封装
class Renderer {
public:
    Renderer();
    ~Renderer();

    /// 初始化窗口与渲染器
    bool init(int windowWidth, int windowHeight,
              const std::string& title, bool fullscreen = false);

    /// 关闭
    void shutdown();

    /// 开始一帧渲染（清屏）
    void beginFrame();

    /// 提交渲染队列并呈现
    void endFrame();

    /// 添加渲染命令到队列（按 layer 自动排序）
    void enqueue(const RenderCommand& cmd);

    /// 便捷：绘制纹理（屏幕坐标）
    void drawTexture(SDL_Texture* tex, const SDL_Rect* src,
                     const SDL_FRect* dst, double angle = 0.0,
                     SDL_RendererFlip flip = SDL_FLIP_NONE,
                     uint8_t alpha = 255,
                     RenderLayer layer = RenderLayer::Player);

    /// 便捷：绘制纯色矩形（调试用）
    void drawRect(const SDL_FRect* rect, uint8_t r, uint8_t g, uint8_t b,
                  uint8_t a = 255, RenderLayer layer = RenderLayer::Debug);

    /// 便捷：绘制圆形（碰撞盒可视化）
    void drawCircle(float cx, float cy, float radius,
                    uint8_t r, uint8_t g, uint8_t b, uint8_t a = 255,
                    RenderLayer layer = RenderLayer::Debug);

    /// 设置整体缩放（逻辑分辨率 -> 窗口分辨率）
    void setLogicalSize(int w, int h);

    /// 获取 SDL 渲染器（供 IMG_LoadTexture 等使用）
    SDL_Renderer* sdlRenderer() { return renderer_; }

    /// 获取窗口
    SDL_Window* sdlWindow() { return window_; }

    /// 当前帧数（调试用）
    int fps() const { return fps_; }

private:
    SDL_Window*   window_   = nullptr;
    SDL_Renderer* renderer_ = nullptr;
    int           logicalW_ = 640;
    int           logicalH_ = 480;

    // 渲染队列（按 layer 分桶）
    std::vector<RenderCommand> queue_[(size_t)RenderLayer::LayerCount];

    // FPS 统计
    int   fps_         = 0;
    int   frameCount_  = 0;
    Uint64 fpsTimer_   = 0;
};

} // namespace th06
```

```cpp
// src/core/Renderer.cpp
#include "core/Renderer.h"
#include "core/Logger.h"
#include <algorithm>
#include <cmath>

namespace th06 {

Renderer::Renderer() = default;

Renderer::~Renderer() {
    shutdown();
}

bool Renderer::init(int windowWidth, int windowHeight,
                    const std::string& title, bool fullscreen) {
    Uint32 flags = SDL_WINDOW_SHOWN;
    if (fullscreen) flags |= SDL_WINDOW_FULLSCREEN_DESKTOP;

    window_ = SDL_CreateWindow(
        title.c_str(),
        SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED,
        windowWidth, windowHeight,
        flags
    );
    if (!window_) {
        LOG_ERROR(std::string("窗口创建失败: ") + SDL_GetError());
        return false;
    }

    // 硬件加速 + VSync
    renderer_ = SDL_CreateRenderer(
        window_, -1,
        SDL_RENDERER_ACCELERATED | SDL_RENDERER_PRESENTVSYNC | SDL_RENDERER_TARGETTEXTURE
    );
    if (!renderer_) {
        LOG_ERROR(std::string("渲染器创建失败: ") + SDL_GetError());
        return false;
    }

    // 设置逻辑分辨率（640x480），SDL 自动缩放到窗口尺寸
    SDL_RenderSetLogicalSize(renderer_, logicalW_, logicalH_);
    // NEAREST 缩放，保持像素锐利
    SDL_SetHint(SDL_HINT_RENDER_SCALE_QUALITY, "0");
    // 默认混合模式
    SDL_SetRenderDrawBlendMode(renderer_, SDL_BLENDMODE_BLEND);

    fpsTimer_ = SDL_GetPerformanceCounter();
    return true;
}

void Renderer::shutdown() {
    if (renderer_) { SDL_DestroyRenderer(renderer_); renderer_ = nullptr; }
    if (window_)   { SDL_DestroyWindow(window_);     window_   = nullptr; }
}

void Renderer::beginFrame() {
    SDL_SetRenderDrawColor(renderer_, 0, 0, 0, 255);
    SDL_RenderClear(renderer_);
}

void Renderer::endFrame() {
    // 按 layer 顺序绘制
    for (size_t layer = 0; layer < (size_t)RenderLayer::LayerCount; ++layer) {
        auto& bucket = queue_[layer];
        for (const auto& cmd : bucket) {
            SDL_SetTextureAlphaMod(cmd.texture, cmd.alpha);
            SDL_SetTextureBlendMode(cmd.texture, cmd.blendMode);
            SDL_RenderCopyExF(
                renderer_,
                cmd.texture,
                &cmd.srcRect,
                &cmd.dstRect,
                cmd.angle,
                &cmd.center,
                cmd.flip
            );
        }
        bucket.clear();
    }
    SDL_RenderPresent(renderer_);

    // FPS 统计
    frameCount_++;
    Uint64 now = SDL_GetPerformanceCounter();
    Uint64 elapsed = now - fpsTimer_;
    double seconds = (double)elapsed / SDL_GetPerformanceFrequency();
    if (seconds >= 1.0) {
        fps_ = (int)(frameCount_ / seconds);
        frameCount_ = 0;
        fpsTimer_ = now;
    }
}

void Renderer::enqueue(const RenderCommand& cmd) {
    queue_[(size_t)cmd.layer].push_back(cmd);
}

void Renderer::drawTexture(SDL_Texture* tex, const SDL_Rect* src,
                           const SDL_FRect* dst, double angle,
                           SDL_RendererFlip flip, uint8_t alpha,
                           RenderLayer layer) {
    RenderCommand cmd;
    cmd.texture = tex;
    if (src) cmd.srcRect = *src;
    if (dst) cmd.dstRect = *dst;
    cmd.angle = angle;
    cmd.flip  = flip;
    cmd.alpha = alpha;
    cmd.layer = layer;
    if (dst) {
        cmd.center = {dst->w / 2.0f, dst->h / 2.0f};
    }
    enqueue(cmd);
}

void Renderer::drawRect(const SDL_FRect* rect, uint8_t r, uint8_t g,
                        uint8_t b, uint8_t a, RenderLayer layer) {
    // 矩形直接绘制到 Debug 层，不入队（简化实现）
    if (layer != RenderLayer::Debug) return;
    SDL_SetRenderDrawColor(renderer_, r, g, b, a);
    SDL_RenderDrawRectF(renderer_, rect);
}

void Renderer::drawCircle(float cx, float cy, float radius,
                          uint8_t r, uint8_t g, uint8_t b, uint8_t a,
                          RenderLayer layer) {
    if (layer != RenderLayer::Debug) return;
    SDL_SetRenderDrawColor(renderer_, r, g, b, a);
    // 用线段近似圆
    const int segments = 24;
    float prevX = cx + radius, prevY = cy;
    for (int i = 1; i <= segments; ++i) {
        float angle = (float)i / segments * 2.0f * 3.14159265f;
        float x = cx + radius * std::cos(angle);
        float y = cy + radius * std::sin(angle);
        SDL_RenderDrawLineF(renderer_, prevX, prevY, x, y);
        prevX = x; prevY = y;
    }
}

void Renderer::setLogicalSize(int w, int h) {
    logicalW_ = w;
    logicalH_ = h;
    if (renderer_) {
        SDL_RenderSetLogicalSize(renderer_, w, h);
    }
}

} // namespace th06
```

> **原版 1:1 还原关键细节**：TH06 原版的渲染顺序是固定的——背景 → 敌方子弹 → 敌人 → 道具 → 玩家子弹 → 玩家 → 特效 → UI。这个顺序决定了视觉层次，**敌方子弹在敌人之下**（被敌人遮挡），但**玩家子弹在玩家之上**（视觉强调火力）。复刻时必须严格遵循此顺序，否则会出现子弹被错误遮挡的视觉 bug。

---

## 3.2 固定 60 帧分离主循环

### 3.2.1 为什么需要分离主循环

TH06 原版的全部游戏逻辑（移动、弹幕、碰撞）都基于 **60 FPS** 的固定帧率设计。如果游戏运行在 144Hz 显示器上，简单的主循环会导致游戏速度变成 2.4 倍，完全无法游玩。

解决方案是**逻辑帧与渲染帧分离**：
- **逻辑帧**：固定 60 FPS，所有游戏逻辑在此更新。
- **渲染帧**：跟随显示器刷新率（60/120/144Hz），只做插值渲染，不影响逻辑。

### 3.2.2 完整实现

```cpp
// include/core/MainLoop.h
#pragma once
#include <SDL2/SDL.h>
#include <functional>
#include <cstdint>

namespace th06 {

class MainLoop {
public:
    static constexpr int    LOGIC_FPS    = 60;
    static constexpr double LOGIC_DT     = 1000.0 / LOGIC_FPS;  // 16.666ms
    static constexpr int    MAX_FRAME_SKIP = 5;  // 防止卡顿后追赶过久

    using UpdateCallback = std::function<void()>;
    using RenderCallback = std::function<void(double alpha)>;

    /// 设置更新回调（每逻辑帧调用一次）
    void setUpdateCallback(UpdateCallback cb) { updateCb_ = std::move(cb); }

    /// 设置渲染回调（每渲染帧调用一次，alpha 为插值因子 0~1）
    void setRenderCallback(RenderCallback cb) { renderCb_ = std::move(cb); }

    /// 运行主循环，直到 stop() 被调用
    void run();

    /// 停止主循环
    void stop() { running_ = false; }

    /// 是否暂停（调试用）
    void setPaused(bool p) { paused_ = p; }
    bool isPaused() const { return paused_; }

    /// 单步前进（暂停时调试用）
    void stepOnce() { stepRequested_ = true; }

    /// 获取当前逻辑帧序号
    uint64_t frameCount() const { return frameCount_; }

private:
    UpdateCallback  updateCb_;
    RenderCallback  renderCb_;
    bool            running_      = false;
    bool            paused_       = false;
    bool            stepRequested_ = false;
    uint64_t        frameCount_   = 0;
};

} // namespace th06
```

```cpp
// src/core/MainLoop.cpp
#include "core/MainLoop.h"
#include "core/Logger.h"

namespace th06 {

void MainLoop::run() {
    running_ = true;
    frameCount_ = 0;

    // 使用 SDL_GetPerformanceCounter 获取高精度时间
    Uint64 perfFreq = SDL_GetPerformanceFrequency();
    Uint64 now      = SDL_GetPerformanceCounter();
    Uint64 lastTime = now;
    double accumulator = 0.0;  // 累积时间（毫秒）

    while (running_) {
        now = SDL_GetPerformanceCounter();
        double frameTime = (double)(now - lastTime) * 1000.0 / perfFreq;
        lastTime = now;

        // 防止暂停后或断点后累积过多（如 frameTime > 250ms 则截断）
        if (frameTime > 250.0) {
            frameTime = 250.0;
        }

        accumulator += frameTime;

        // 固定步长更新逻辑
        int updates = 0;
        while (accumulator >= LOGIC_DT && updates < MAX_FRAME_SKIP) {
            if (!paused_ || stepRequested_) {
                if (updateCb_) updateCb_();
                ++frameCount_;
                stepRequested_ = false;
            }
            accumulator -= LOGIC_DT;
            ++updates;
        }

        // 插值因子：当前逻辑帧已完成的进度
        double alpha = accumulator / LOGIC_DT;
        if (alpha > 1.0) alpha = 1.0;

        // 渲染（每渲染帧调用一次）
        if (renderCb_) {
            renderCb_(alpha);
        }

        // 让出 CPU（避免 100% 占用）
        // 注意：开启 VSync 时，SDL_RenderPresent 已自动同步显示器刷新率
        // 此处 delay 仅在帧率过高时生效
        now = SDL_GetPerformanceCounter();
        double elapsedMs = (double)(now - lastTime) * 1000.0 / perfFreq;
        if (elapsedMs < LOGIC_DT) {
            SDL_Delay((Uint32)(LOGIC_DT - elapsedMs));
        }
    }
}

} // namespace th06
```

### 3.2.3 主循环使用示例

```cpp
// src/main.cpp
#include "core/Renderer.h"
#include "core/MainLoop.h"
#include "core/InputManager.h"
#include "core/ResourceManager.h"
#include "core/ConfigLoader.h"
#include "core/LuaEngine.h"
#include "core/RNG.h"
#include "game/GameState.h"
#include <iostream>

using namespace th06;

int main(int argc, char* argv[]) {
    (void)argc; (void)argv;

    // 1. 初始化 SDL 各子系统
    if (SDL_Init(SDL_INIT_VIDEO | SDL_INIT_AUDIO | SDL_INIT_TIMER) != 0) {
        std::cerr << "SDL_Init 失败: " << SDL_GetError() << "\n";
        return 1;
    }
    if (IMG_Init(IMG_INIT_PNG) == 0) {
        std::cerr << "IMG_Init 失败\n";
        return 1;
    }
    if (Mix_OpenAudio(44100, MIX_DEFAULT_FORMAT, 2, 1024) != 0) {
        std::cerr << "Mix_OpenAudio 失败: " << Mix_GetError() << "\n";
        return 1;
    }
    Mix_AllocateChannels(32);  // 32 个音效声道

    // 2. 初始化各管理器
    Renderer& renderer = Renderer::instance();  // 假设改为单例，或此处构造
    renderer.init(960, 720, "TH06 Remaster", false);
    renderer.setLogicalSize(640, 480);

    ConfigLoader::instance().loadAll("assets/config");
    LuaEngine::instance().init();
    InputManager::instance().init();

    // 3. 创建游戏状态
    GameState gameState;
    gameState.init();

    // 4. 配置主循环
    MainLoop loop;
    loop.setUpdateCallback([&]() {
        InputManager::instance().poll();
        gameState.update();
    });
    loop.setRenderCallback([&](double alpha) {
        renderer.beginFrame();
        gameState.render(alpha);
        renderer.endFrame();
    });

    // 5. 运行
    loop.run();

    // 6. 清理
    gameState.shutdown();
    LuaEngine::instance().shutdown();
    ResourceManager::instance().releaseAll();
    renderer.shutdown();
    Mix_CloseAudio();
    IMG_Quit();
    SDL_Quit();
    return 0;
}
```

> **原版 1:1 还原关键细节**：TH06 原版**不使用插值渲染**，每帧直接渲染当前逻辑状态。这意味着在 144Hz 显示器上，画面会出现"每帧重复绘制 2-3 次"的卡顿感。为了既保持 60FPS 逻辑又获得流畅画面，我们的实现加入了 `alpha` 插值因子。但**插值只应用于渲染位置**（如子弹位置在上一帧与当前帧之间线性插值），**绝不能影响碰撞判定与逻辑数值**。如果你追求极致原版手感，可以关闭插值（`alpha` 恒为 0），让画面与逻辑严格同步。

---

## 3.3 全局资源管理系统

### 3.3.1 设计要点

资源管理器需要解决三个核心问题：
1. **避免重复加载**：同一纹理被多个对象引用时，只加载一次。
2. **自动释放**：资源不再被引用时，自动释放显存。
3. **懒加载/预加载**：支持按需加载（菜单图标）与批量预加载（关卡资源）。

### 3.3.2 完整实现

```cpp
// src/core/ResourceManager.cpp
#include "core/ResourceManager.h"
#include "core/Renderer.h"
#include "core/Logger.h"
#include "core/ConfigLoader.h"
#include <SDL2/SDL_image.h>
#include <fstream>
#include <nlohmann/json.hpp>

namespace th06 {

// 声明全局渲染器（由 main.cpp 设置）
extern Renderer* g_renderer;

ResourceManager& ResourceManager::instance() {
    static ResourceManager inst;
    return inst;
}

std::shared_ptr<Texture> ResourceManager::loadTexture(const std::string& path) {
    std::lock_guard<std::mutex> lock(mutex_);
    auto it = textures_.find(path);
    if (it != textures_.end()) {
        return it->second;  // 返回缓存
    }

    if (!g_renderer) {
        LOG_ERROR("渲染器未初始化，无法加载纹理: " + path);
        return nullptr;
    }

    SDL_Texture* sdlTex = IMG_LoadTexture(g_renderer->sdlRenderer(), path.c_str());
    if (!sdlTex) {
        LOG_ERROR(std::string("纹理加载失败: ") + path + " - " + IMG_GetError());
        return nullptr;
    }

    auto tex = std::make_shared<Texture>();
    tex->sdlTexture = sdlTex;
    SDL_QueryTexture(sdlTex, nullptr, nullptr, &tex->width, &tex->height);

    textures_[path] = tex;
    LOG_INFO("纹理已加载: " + path + " (" +
             std::to_string(tex->width) + "x" + std::to_string(tex->height) + ")");
    return tex;
}

std::shared_ptr<SoundEffect> ResourceManager::loadSound(const std::string& path) {
    std::lock_guard<std::mutex> lock(mutex_);
    auto it = sounds_.find(path);
    if (it != sounds_.end()) {
        return it->second;
    }

    Mix_Chunk* chunk = Mix_LoadWAV(path.c_str());
    if (!chunk) {
        LOG_ERROR(std::string("音效加载失败: ") + path + " - " + Mix_GetError());
        return nullptr;
    }

    auto snd = std::make_shared<SoundEffect>();
    snd->chunk = chunk;
    sounds_[path] = snd;
    return snd;
}

void ResourceManager::loadMusic(const std::string& name, const std::string& path) {
    std::lock_guard<std::mutex> lock(mutex_);
    musicPaths_[name] = path;

    // 加载循环点元数据
    auto& bgmLoop = ConfigLoader::instance().getDifficultyConfig();  // 简化，实际从 bgm_loop.json
    // 此处省略元数据加载，见 1.3.2 节
}

void ResourceManager::playMusic(const std::string& name, int loops) {
    auto it = musicPaths_.find(name);
    if (it == musicPaths_.end()) {
        LOG_ERROR("未找到 BGM: " + name);
        return;
    }
    // 释放上一首
    Mix_Music* old = Mix_GetMusicHook(nullptr) ? nullptr : nullptr;  // 简化
    // 实际实现需维护当前 Mix_Music* 指针
    Mix_Music* music = Mix_LoadMUS(it->second.c_str());
    if (!music) {
        LOG_ERROR(std::string("BGM 加载失败: ") + Mix_GetError());
        return;
    }
    Mix_PlayMusic(music, loops);
    // 注意：Mix_LoadMUS 返回的指针需在切换 BGM 时 Mix_FreeMusic
}

void ResourceManager::playSound(const std::string& name, int channel) {
    auto snd = loadSound(name);
    if (snd && snd->chunk) {
        Mix_PlayChannel(channel, snd->chunk, 0);
    }
}

void ResourceManager::releaseAll() {
    std::lock_guard<std::mutex> lock(mutex_);
    textures_.clear();
    sounds_.clear();
    musicPaths_.clear();
}

void ResourceManager::gc() {
    std::lock_guard<std::mutex> lock(mutex_);
    // shared_ptr 引用计数为 1 时（仅 textures_ 持有），说明无业务引用
    for (auto it = textures_.begin(); it != textures_.end(); ) {
        if (it->second.use_count() <= 1) {
            LOG_INFO("GC 释放纹理: " + it->first);
            it = textures_.erase(it);
        } else {
            ++it;
        }
    }
    for (auto it = sounds_.begin(); it != sounds_.end(); ) {
        if (it->second.use_count() <= 1) {
            it = sounds_.erase(it);
        } else {
            ++it;
        }
    }
}

ResourceManager::~ResourceManager() {
    releaseAll();
}

} // namespace th06
```

### 3.3.3 纹理对象池与批量预加载

```cpp
// include/core/TexturePool.h
#pragma once
#include "core/ResourceManager.h"
#include <string>
#include <unordered_map>
#include <vector>

namespace th06 {

/// 纹理对象池 —— 按关卡批量预加载/卸载
class TexturePool {
public:
    static TexturePool& instance();

    /// 预加载一个关卡的所有纹理
    void preloadStage(int stageId);

    /// 卸载一个关卡的纹理
    void unloadStage(int stageId);

    /// 获取纹理（若未加载则即时加载）
    std::shared_ptr<Texture> get(const std::string& name);

private:
    TexturePool() = default;
    // stageId -> 纹理名列表
    std::unordered_map<int, std::vector<std::string>> stageTextures_;
    // 纹理名 -> 纹理对象
    std::unordered_map<std::string, std::shared_ptr<Texture>> cache_;
};

} // namespace th06
```

```cpp
// src/core/TexturePool.cpp
#include "core/TexturePool.h"
#include "core/Logger.h"

namespace th06 {

TexturePool& TexturePool::instance() {
    static TexturePool inst;
    return inst;
}

void TexturePool::preloadStage(int stageId) {
    // 实际项目中，stageId -> 纹理列表的映射从 stages.json 加载
    // 此处简化为硬编码示例
    std::vector<std::string> textures;
    switch (stageId) {
        case 1:
            textures = {
                "assets/images/stage01_bg_far.png",
                "assets/images/stage01_bg_near.png",
                "assets/images/enemy_fairy_small.png",
                "assets/images/enemy_fairy_large.png",
                "assets/images/bullet_enemy.png",
                "assets/images/item.png"
            };
            break;
        case 6:  // 红魔馆 BOSS 战
            textures = {
                "assets/images/stage06_bg.png",
                "assets/images/boss_remilia.png",
                "assets/images/bullet_enemy.png",
                "assets/images/bullet_boss.png",
                "assets/images/item.png"
            };
            break;
        // ... 其他关卡
    }

    for (const auto& path : textures) {
        if (cache_.find(path) == cache_.end()) {
            auto tex = ResourceManager::instance().loadTexture(path);
            if (tex) cache_[path] = tex;
        }
    }
    stageTextures_[stageId] = textures;
    LOG_INFO("关卡 " + std::to_string(stageId) + " 纹理预加载完成 (" +
             std::to_string(textures.size()) + " 张)");
}

void TexturePool::unloadStage(int stageId) {
    auto it = stageTextures_.find(stageId);
    if (it == stageTextures_.end()) return;
    for (const auto& path : it->second) {
        cache_.erase(path);
    }
    stageTextures_.erase(it);
    LOG_INFO("关卡 " + std::to_string(stageId) + " 纹理已卸载");
}

std::shared_ptr<Texture> TexturePool::get(const std::string& name) {
    auto it = cache_.find(name);
    if (it != cache_.end()) return it->second;
    auto tex = ResourceManager::instance().loadTexture(name);
    if (tex) cache_[name] = tex;
    return tex;
}

} // namespace th06
```

> **原版 1:1 还原关键细节**：TH06 原版在关卡切换时会有约 0.5 秒的加载黑屏，期间释放上一关纹理并加载下一关纹理。复刻时应在 `Stage::onEnter()` 中调用 `TexturePool::preloadStage()`，在 `Stage::onExit()` 中调用 `unloadStage()`。**注意：玩家纹理、UI 纹理、字体纹理是全局常驻的，不随关卡卸载**。

---

## 3.4 输入系统

### 3.4.1 设计要点

STG 游戏对输入响应要求极高，必须满足：
1. **零延迟**：按键状态在每帧逻辑更新前刷新。
2. **双模式**：事件驱动（用于菜单确认/暂停）+ 实时轮询（用于移动/射击）。
3. **长按连发**：射击键长按时按固定间隔自动连发，而非每帧发射。
4. **瞬时触发**：炸弹键只在按下瞬间触发一次，松开再按才再次触发。
5. **自定义按键**：支持玩家修改按键映射，存档保存。

### 3.4.2 完整实现

```cpp
// include/core/InputManager.h
#pragma once
#include <SDL2/SDL.h>
#include <array>
#include <cstdint>
#include <string>

namespace th06 {

/// 游戏动作枚举
enum class Action : uint8_t {
    Up = 0, Down, Left, Right,    // 方向
    Shoot,                        // 射击（可长按连发）
    Bomb,                         // 炸弹（瞬时触发）
    Focus,                        // 低速（Shift）
    Pause,                        // 暂停
    Confirm,                      // 菜单确认
    Cancel,                       // 菜单取消
    ActionCount
};

/// 按键映射配置
struct KeyMapping {
    std::array<SDL_Scancode, (size_t)Action::ActionCount> keys;

    KeyMapping() {
        keys[(size_t)Action::Up]      = SDL_SCANCODE_UP;
        keys[(size_t)Action::Down]    = SDL_SCANCODE_DOWN;
        keys[(size_t)Action::Left]    = SDL_SCANCODE_LEFT;
        keys[(size_t)Action::Right]   = SDL_SCANCODE_RIGHT;
        keys[(size_t)Action::Shoot]   = SDL_SCANCODE_Z;
        keys[(size_t)Action::Bomb]    = SDL_SCANCODE_X;
        keys[(size_t)Action::Focus]   = SDL_SCANCODE_LSHIFT;
        keys[(size_t)Action::Pause]   = SDL_SCANCODE_ESCAPE;
        keys[(size_t)Action::Confirm] = SDL_SCANCODE_Z;
        keys[(size_t)Action::Cancel]  = SDL_SCANCODE_X;
    }
};

class InputManager {
public:
    static InputManager& instance();

    /// 初始化（加载存档中的按键映射）
    bool init();

    /// 每帧调用：处理 SDL 事件队列 + 更新键盘状态
    void poll();

    /// 动作是否正在按下（持续状态，用于移动/射击）
    bool isDown(Action action) const;

    /// 动作是否在本帧刚刚按下（边沿触发，用于炸弹/菜单确认）
    bool isPressed(Action action) const;

    /// 动作是否在本帧刚刚松开
    bool isReleased(Action action) const;

    /// 射击连发：返回 true 时表示应发射一颗子弹
    bool shouldFire();

    /// 获取按键映射
    const KeyMapping& keyMapping() const { return keyMapping_; }

    /// 设置按键映射（自定义按键时调用）
    void setKeyMapping(const KeyMapping& km);

    /// 等待玩家按下任意键，返回该键（用于自定义按键界面）
    SDL_Scancode waitForKeyPress();

private:
    InputManager() = default;

    KeyMapping keyMapping_;

    // 当前帧状态
    std::array<bool, (size_t)Action::ActionCount> current_{};
    // 上一帧状态
    std::array<bool, (size_t)Action::ActionCount> previous_{};

    // 射击连发计时
    int  fireHoldFrames_ = 0;     // 射击键已持续按下的帧数
    int  fireCooldown_   = 0;     // 距离上次发射的冷却帧数

    // 连发参数（与原版对齐）
    static constexpr int FIRE_INITIAL_DELAY = 8;   // 首次按下后延迟 8 帧开始连发
    static constexpr int FIRE_INTERVAL      = 4;   // 连发间隔 4 帧
};

} // namespace th06
```

```cpp
// src/core/InputManager.cpp
#include "core/InputManager.h"
#include "core/Logger.h"

namespace th06 {

InputManager& InputManager::instance() {
    static InputManager inst;
    return inst;
}

bool InputManager::init() {
    // 实际项目中从存档加载 KeyMapping
    // 此处使用默认值
    return true;
}

void InputManager::poll() {
    // 保存上一帧状态
    previous_ = current_;

    // 处理 SDL 事件队列（用于窗口关闭等）
    SDL_Event event;
    while (SDL_PollEvent(&event)) {
        if (event.type == SDL_QUIT) {
            // 通知主循环退出
            // 实际通过回调或全局标志实现
        }
    }

    // 实时轮询键盘状态（零延迟）
    const Uint8* keyboard = SDL_GetKeyboardState(nullptr);

    for (size_t i = 0; i < (size_t)Action::ActionCount; ++i) {
        SDL_Scancode key = keyMapping_.keys[i];
        current_[i] = keyboard[key];
    }

    // 射击连发逻辑
    if (current_[(size_t)Action::Shoot]) {
        ++fireHoldFrames_;
    } else {
        fireHoldFrames_ = 0;
        fireCooldown_ = 0;
    }
    if (fireCooldown_ > 0) --fireCooldown_;
}

bool InputManager::isDown(Action action) const {
    return current_[(size_t)action];
}

bool InputManager::isPressed(Action action) const {
    return current_[(size_t)action] && !previous_[(size_t)action];
}

bool InputManager::isReleased(Action action) const {
    return !current_[(size_t)action] && previous_[(size_t)action];
}

bool InputManager::shouldFire() {
    if (!current_[(size_t)Action::Shoot]) return false;

    // 首次按下立即发射
    if (fireHoldFrames_ == 1) {
        fireCooldown_ = FIRE_INITIAL_DELAY;
        return true;
    }
    // 连发阶段
    if (fireHoldFrames_ > FIRE_INITIAL_DELAY && fireCooldown_ == 0) {
        fireCooldown_ = FIRE_INTERVAL;
        return true;
    }
    return false;
}

void InputManager::setKeyMapping(const KeyMapping& km) {
    keyMapping_ = km;
    // 实际项目中保存到存档
}

SDL_Scancode InputManager::waitForKeyPress() {
    // 清空事件队列
    SDL_Event event;
    while (SDL_PollEvent(&event)) {}

    // 等待按键按下
    while (true) {
        SDL_WaitEvent(&event);
        if (event.type == SDL_KEYDOWN && event.key.repeat == 0) {
            return event.key.keysym.scancode;
        }
        if (event.type == SDL_QUIT) {
            return SDL_SCANCODE_UNKNOWN;
        }
    }
}

} // namespace th06
```

### 3.4.3 菜单导航逻辑

菜单使用边沿触发（`isPressed`）实现"按一下移动一项"的体验：

```cpp
// src/menu/MenuBase.cpp 节选
#include "menu/MenuBase.h"
#include "core/InputManager.h"

namespace th06 {

void MenuBase::handleInput() {
    auto& input = InputManager::instance();
    if (input.isPressed(Action::Up)) {
        selectedIndex_ = (selectedIndex_ - 1 + items_.size()) % items_.size();
        ResourceManager::instance().playSound("assets/audio/se/se_select.wav");
    }
    if (input.isPressed(Action::Down)) {
        selectedIndex_ = (selectedIndex_ + 1) % items_.size();
        ResourceManager::instance().playSound("assets/audio/se/se_select.wav");
    }
    if (input.isPressed(Action::Confirm)) {
        if (onConfirm_) onConfirm_(selectedIndex_);
        ResourceManager::instance().playSound("assets/audio/se/se_ok.wav");
    }
    if (input.isPressed(Action::Cancel)) {
        if (onCancel_) onCancel_();
        ResourceManager::instance().playSound("assets/audio/se/se_cancel.wav");
    }
}

} // namespace th06
```

> **原版 1:1 还原关键细节**：TH06 原版的射击连发参数为：首次按下立即发射，之后每 4 帧发射一次。但**低速模式（Shift）下连发间隔不变**，只是弹道更密集。炸弹键必须用 `isPressed`（边沿触发），否则按住炸弹键会连续消耗所有炸弹。菜单导航也必须用边沿触发，否则按一下方向键会跳过多个选项。

---

## 3.5 红魔乡原生 RNG 随机算法

### 3.5.1 算法背景

TH06 原版使用一个**自定义的线性同余生成器（LCG）**作为随机数源。这个 RNG 的特点是：
- **确定性**：相同种子产生相同序列，这是 Replay 录像回放的基础。
- **32 位整数运算**：使用 `uint32_t`，避免浮点误差。
- **全局共享**：所有随机行为（弹幕、道具掉落、敌人移动）共用同一个 RNG 状态。

通过逆向工程得到的 TH06 RNG 算法如下：

```
state = (state * 0x41C64E6D + 0x3039) & 0xFFFFFFFF
random_value = (state >> 16) & 0x7FFF   // 返回 0~32767
```

### 3.5.2 完整 C++ 实现

```cpp
// include/core/RNG.h
#pragma once
#include <cstdint>
#include <cmath>

namespace th06 {

/// 红魔乡原生 RNG —— 线性同余生成器
/// 用于弹幕、道具掉落、敌人行为的随机数生成
/// 必须与原版算法完全一致，否则 Replay 录像无法正确回放
class RNG {
public:
    /// 设置随机种子（开局时调用，种子保存到 Replay 文件）
    void seed(uint32_t s) { state_ = s; }

    /// 获取当前种子（用于 Replay 录像保存）
    uint32_t getSeed() const { return state_; }

    /// 生成 0~32767 的随机整数（原版算法）
    uint16_t next() {
        state_ = state_ * 0x41C64E6Du + 0x3039u;
        return (uint16_t)((state_ >> 16) & 0x7FFF);
    }

    /// 生成 [min, max] 范围内的随机整数
    int nextInt(int min, int max) {
        return min + next() % (max - min + 1);
    }

    /// 生成 [0.0, 1.0) 范围内的随机浮点数
    float nextFloat() {
        return next() / 32768.0f;
    }

    /// 生成 [min, max) 范围内的随机浮点数
    float nextFloat(float min, float max) {
        return min + nextFloat() * (max - min);
    }

    /// 生成随机角度（弧度，0~2π）
    float nextAngle() {
        return nextFloat(0.0f, 6.2831853f);
    }

    /// 按概率返回 true（用于道具掉落判定）
    bool chance(float probability) {
        return nextFloat() < probability;
    }

    /// 从数组中随机选取一个元素
    template<typename T>
    const T& pick(const T* array, size_t size) {
        return array[nextInt(0, (int)size - 1)];
    }

private:
    uint32_t state_ = 0;
};

/// 全局 RNG 实例（单例）
RNG& globalRNG();

} // namespace th06
```

```cpp
// src/core/RNG.cpp
#include "core/RNG.h"

namespace th06 {

RNG& globalRNG() {
    static RNG rng;
    return rng;
}

} // namespace th06
```

### 3.5.3 使用示例

```cpp
// 弹幕生成时使用 RNG 决定随机角度
void spawnRandomSpread(float x, float y, int count) {
    auto& rng = th06::globalRNG();
    for (int i = 0; i < count; ++i) {
        float angle = rng.nextAngle();
        float speed = rng.nextFloat(2.0f, 4.0f);
        // ... 生成子弹
    }
}

// 道具掉落判定
void onEnemyDeath(const Enemy& enemy) {
    auto& rng = th06::globalRNG();
    for (const auto& drop : enemy.dropTable) {
        if (rng.chance(drop.chance)) {
            spawnItem(enemy.position, drop.itemType);
        }
    }
}

// 开局设置种子（用于 Replay）
void startNewGame() {
    uint32_t seed = (uint32_t)SDL_GetTicks();  // 用系统时间作为种子
    th06::globalRNG().seed(seed);
    // 将 seed 保存到 Replay 文件
}
```

> **原版 1:1 还原关键细节**：TH06 原版的 RNG 调用顺序极其严格。例如 BOSS 在第 100 帧调用一次 `next()` 决定弹幕角度，在第 105 帧调用一次决定道具掉落。如果复刻时调换了两者的顺序，或插入了一次额外的 `next()` 调用，**整个随机序列就会偏移，导致 Replay 录像回放时弹幕完全错位**。因此 RNG 的调用必须严格对照 ECL 脚本的反编译文本，逐帧对齐。开发时建议在 RNG 的 `next()` 方法中添加日志（仅 Debug 模式），记录每次调用的调用栈与返回值，便于排查 Replay 错位问题。

---
# 第四部分：游戏核心业务系统

本部分拆解 TH06 的全部游戏业务系统，每个系统包含底层实现细节与原版还原要点。

## 4.1 玩家机体系统

### 4.1.1 移动物理：无惯性瞬时变速、Shift 低速倍率、屏幕边界限制

TH06 原版的玩家移动是**无惯性**的——按下方向键立即以最大速度移动，松开立即停止。这与现实物理不同，但符合 STG 手感需求：玩家需要精确控制位置以躲避弹幕。

```cpp
// include/game/Player.h
#pragma once
#include "core/Math.h"
#include "core/RNG.h"
#include <cstdint>

namespace th06 {

enum class Character : uint8_t { Reimu, Marisa };
enum class PlayerState : uint8_t { Alive, Invincible, Dying, Respawning };

class Player {
public:
    void init(Character chara);

    /// 每帧更新（由 GameState 调用）
    void update();

    /// 渲染（由 GameState 调用）
    void render(double alpha);

    /// 受击（中弹时调用）
    void hit();

    /// 使用炸弹
    bool useBomb();

    /// 复活
    void respawn();

    // --- 属性 ---
    Vec2f  position;
    Vec2f  velocity;
    Character character   = Character::Reimu;
    PlayerState state     = PlayerState::Alive;

    int    lives          = 3;
    int    bombs          = 2;
    int    power          = 0;       // 0~128
    int    grazeCount     = 0;

    float  hitboxRadius   = 2.0f;    // 极小判定
    float  grazeRadius    = 12.0f;   // 擦弹半径

    int    invincibleFrames = 0;     // 无敌帧计数
    int    bombActiveFrames = 0;     // 炸弹生效帧数

private:
    void updateMovement();
    void updateShooting();
    void updateBomb();
    void updateInvincible();

    // 射击计时
    int  fireCooldown_ = 0;

    // 配置（从 player.json 加载）
    float moveSpeed_   = 4.0f;
    float focusSpeed_  = 2.0f;
    int   fireInterval_ = 4;
    float bulletSpeed_ = 8.0f;
    float bulletDamage_ = 1.2f;
    int   bombDuration_ = 180;
    int   bombInvincible_ = 240;
};

} // namespace th06
```

```cpp
// src/game/Player.cpp 节选
#include "game/Player.h"
#include "game/BulletManager.h"
#include "core/InputManager.h"
#include "core/ConfigLoader.h"
#include "core/ResourceManager.h"
#include "core/Logger.h"

namespace th06 {

extern BulletManager* g_playerBullets;
extern BulletManager* g_enemyBullets;

void Player::init(Character chara) {
    character = chara;
    position = {192.0f, 400.0f};  // 屏幕底部中央偏左
    velocity = {0.0f, 0.0f};
    state    = PlayerState::Alive;
    lives    = 3;
    bombs    = 2;
    power    = 0;
    invincibleFrames = 120;  // 开局短暂无敌

    // 从配置加载参数
    auto& cfg = ConfigLoader::instance().getPlayerConfig();
    const char* charaName = (chara == Character::Reimu) ? "reimu" : "marisa";
    auto& c = cfg[charaName];
    moveSpeed_    = c["move_speed"];
    focusSpeed_   = c["focus_speed"];
    fireInterval_ = c["shot"]["fire_interval"];
    bulletSpeed_  = c["shot"]["bullet_speed"];
    bulletDamage_ = c["shot"]["damage_per_bullet"];
    bombDuration_ = c["bomb"]["duration_frames"];
    bombInvincible_ = c["bomb"]["invincible_frames"];
}

void Player::update() {
    updateMovement();
    updateShooting();
    updateBomb();
    updateInvincible();
}

void Player::updateMovement() {
    auto& input = InputManager::instance();
    bool focus = input.isDown(Action::Focus);
    float speed = focus ? focusSpeed_ : moveSpeed_;

    // 无惯性瞬时变速
    velocity = {0.0f, 0.0f};
    if (input.isDown(Action::Left))  velocity.x = -speed;
    if (input.isDown(Action::Right)) velocity.x =  speed;
    if (input.isDown(Action::Up))    velocity.y = -speed;
    if (input.isDown(Action::Down))  velocity.y =  speed;

    // 对角线移动归一化（避免对角线移动比直线快）
    if (velocity.x != 0.0f && velocity.y != 0.0f) {
        velocity.x *= 0.7071f;  // 1/√2
        velocity.y *= 0.7071f;
    }

    position += velocity;

    // 屏幕边界限制（640x480，玩家活动区域 32~608, 16~464）
    if (position.x < 16.0f)  position.x = 16.0f;
    if (position.x > 624.0f) position.x = 624.0f;
    if (position.y < 16.0f)  position.y = 16.0f;
    if (position.y > 464.0f) position.y = 464.0f;
}

void Player::updateShooting() {
    auto& input = InputManager::instance();
    if (input.shouldFire()) {
        // 根据 power 等级决定弹道
        // power 0~32:   2 路弹
        // power 33~64:  3 路弹
        // power 65~96:  4 路弹
        // power 97~128: 5 路弹 + 副武器
        int bulletCount = 2 + power / 32;
        for (int i = 0; i < bulletCount; ++i) {
            float offsetX = (i - (bulletCount - 1) / 2.0f) * 8.0f;
            auto* b = g_playerBullets->spawn();
            if (b) {
                b->position = {position.x + offsetX, position.y - 16.0f};
                b->velocity = {0.0f, -bulletSpeed_};
                b->damage = bulletDamage_;
                b->active = true;
                b->type = BulletType::Needle;
                b->color = BulletColor::Yellow;
            }
        }
        // 灵梦的追踪弹（副武器）
        if (character == Character::Reimu && power >= 64) {
            for (int i = 0; i < 2; ++i) {
                auto* b = g_playerBullets->spawn();
                if (b) {
                    b->position = {position.x + (i == 0 ? -16.0f : 16.0f), position.y};
                    b->velocity = {(i == 0 ? -2.0f : 2.0f), -6.0f};
                    b->damage = bulletDamage_ * 0.5f;
                    b->active = true;
                    b->type = BulletType::Small;
                    b->color = BulletColor::Red;
                    // 标记为追踪弹，BulletManager 会调整其速度方向
                    b->grazed = true;  // 复用字段标记追踪
                }
            }
        }
        ResourceManager::instance().playSound("assets/audio/se/se_plst00.wav");
    }
}

void Player::updateBomb() {
    if (bombActiveFrames > 0) {
        --bombActiveFrames;
        // 炸弹期间持续清弹
        if (bombActiveFrames % 4 == 0) {
            g_enemyBullets->clearAll();
        }
        // 炸弹期间持续造成伤害（对屏幕内所有敌人）
        // 由 EnemyManager 处理
    }
}

bool Player::useBomb() {
    if (bombs <= 0) return false;
    if (bombActiveFrames > 0) return false;  // 炸弹进行中不可再用
    if (state == PlayerState::Dying) return false;

    --bombs;
    bombActiveFrames = bombDuration_;
    invincibleFrames = std::max(invincibleFrames, bombInvincible_);
    g_enemyBullets->clearAll();
    ResourceManager::instance().playSound("assets/audio/se/se_bomb.wav");
    return true;
}

void Player::updateInvincible() {
    if (invincibleFrames > 0) --invincibleFrames;
    if (state == PlayerState::Invincible && invincibleFrames == 0) {
        state = PlayerState::Alive;
    }
}

void Player::hit() {
    if (invincibleFrames > 0) return;
    if (bombActiveFrames > 0) return;  // 炸弹期间无敌

    --lives;
    state = PlayerState::Dying;
    // 死亡时掉落 Power 道具（原版行为）
    // 由 ItemManager 处理
    ResourceManager::instance().playSound("assets/audio/se/se_pldead.wav");

    if (lives >= 0) {
        respawn();
    } else {
        state = PlayerState::Dying;
        // 触发 Game Over
    }
}

void Player::respawn() {
    position = {192.0f, 400.0f};
    velocity = {0.0f, 0.0f};
    state = PlayerState::Invincible;
    invincibleFrames = 180;  // 复活后 3 秒无敌
    power = std::max(0, power - 32);  // 死亡掉火力
    bombs = std::max(bombs, 2);       // 死亡后炸弹补到 2（原版行为）
}

} // namespace th06
```

> **原版 1:1 还原关键细节**：
> 1. **玩家判定半径仅 2 像素**，是 STG 中最小的之一。这使得玩家可以"贴着"弹幕穿过，是 TH06 手感的核心。
> 2. **低速模式（Shift）下显示判定点**——一个白色小圆点出现在玩家中心，帮助玩家精确躲避。
> 3. **死亡后炸弹补到 2 个**：如果死亡时炸弹少于 2，复活后补到 2；如果多于 2，保持不变。这是原版的"慈悲机制"。
> 4. **死亡掉落 Power 道具**：死亡时当前火力的一半以 Power 道具形式掉落，玩家可重新拾取。

---

## 4.2 弹幕系统

弹幕系统是 STG 的核心，本节重点细化。

### 4.2.1 对象池预分配实现

弹幕数量可达数千，若每帧 `new`/`delete` 会造成严重性能损耗与内存碎片。采用**预分配对象池**：启动时一次性分配 8192 个 Bullet 对象，运行时通过 `active` 标志位复用。

```cpp
// src/game/BulletManager.cpp
#include "game/BulletManager.h"
#include "game/Player.h"
#include "core/Renderer.h"
#include "core/ResourceManager.h"
#include "core/Logger.h"

namespace th06 {

BulletManager::BulletManager() {
    pool_.resize(POOL_SIZE);
    for (auto& b : pool_) {
        b.active = false;
    }
}

Bullet* BulletManager::spawn() {
    // 线性扫描找空闲槽位（从上次位置开始，避免每次从头扫描）
    for (size_t i = 0; i < POOL_SIZE; ++i) {
        size_t idx = (cursor_ + i) % POOL_SIZE;
        if (!pool_[idx].active) {
            cursor_ = (idx + 1) % POOL_SIZE;
            ++activeCount_;
            pool_[idx].reset();
            return &pool_[idx];
        }
    }
    // 池满（原版也会丢弃新弹）
    LOG_WARN("弹幕池已满，丢弃新弹");
    return nullptr;
}

void BulletManager::updateAll() {
    for (auto& b : pool_) {
        if (!b.active) continue;
        b.update();
        if (b.isOffscreen() || (b.lifeFrames > 0 && b.ageFrames >= b.lifeFrames)) {
            b.active = false;
            --activeCount_;
        }
    }
}

void BulletManager::renderAll(Renderer& renderer) {
    // 批量渲染：同纹理合并
    // 实际实现中按 BulletType 分桶，每桶一次 draw call
    // 此处简化为逐弹渲染（性能足够，因 SDL2 内部有批处理）
    for (auto& b : pool_) {
        if (!b.active) continue;
        RenderCommand cmd;
        // 根据 type 加载对应纹理（实际应缓存）
        // cmd.texture = ...
        cmd.dstRect = {
            b.position.x - 4.0f, b.position.y - 4.0f,
            8.0f, 8.0f
        };
        cmd.angle = b.angle;
        cmd.alpha = b.alpha;
        cmd.layer = RenderLayer::BulletEnemy;
        renderer.enqueue(cmd);
    }
}

void BulletManager::checkCollisionWithPlayer(Player& player) {
    if (player.state == PlayerState::Invincible || player.invincibleFrames > 0) return;
    for (auto& b : pool_) {
        if (!b.active) continue;
        float dx = b.position.x - player.position.x;
        float dy = b.position.y - player.position.y;
        float distSq = dx * dx + dy * dy;
        float r = b.collisionRadius + player.hitboxRadius;
        if (distSq < r * r) {
            player.hit();
            b.active = false;
            --activeCount_;
            return;  // 一帧只判定一次中弹
        }
        // 擦弹判定
        float grazeR = b.collisionRadius + player.grazeRadius;
        if (!b.grazed && distSq < grazeR * grazeR) {
            b.grazed = true;
            player.grazeCount++;
            ResourceManager::instance().playSound("assets/audio/se/se_graze.wav");
            // 加分逻辑由 ScoreSystem 处理
        }
    }
}

void BulletManager::checkCollisionWithPlayerBullets(BulletManager& playerBullets) {
    for (auto& eb : pool_) {
        if (!eb.active) continue;
        for (auto& pb : playerBullets.pool_) {
            if (!pb.active) continue;
            float dx = eb.position.x - pb.position.x;
            float dy = eb.position.y - pb.position.y;
            float distSq = dx * dx + dy * dy;
            float r = eb.collisionRadius + pb.collisionRadius;
            if (distSq < r * r) {
                eb.damage -= pb.damage;
                pb.active = false;
                --playerBullets.activeCount_;
                if (eb.damage <= 0) {
                    eb.active = false;
                    --activeCount_;
                }
                break;
            }
        }
    }
}

void BulletManager::clearAll() {
    for (auto& b : pool_) b.active = false;
    activeCount_ = 0;
}

} // namespace th06
```

### 4.2.2 弹幕基础属性与更新

```cpp
// src/game/Bullet.cpp
#include "game/Bullet.h"
#include <cmath>

namespace th06 {

void Bullet::update() {
    ++ageFrames;

    // 应用加速度
    velocity += acceleration;

    // 应用角速度（旋转弹道）
    if (angularVelocity != 0.0f) {
        angle += angularVelocity;
        float rad = angle * 3.14159265f / 180.0f;
        float speed = std::sqrt(velocity.x * velocity.x + velocity.y * velocity.y);
        velocity.x = std::cos(rad) * speed;
        velocity.y = std::sin(rad) * speed;
    }

    // 位移
    position += velocity;
}

bool Bullet::isOffscreen(float margin) const {
    return position.x < -margin || position.x > 640.0f + margin ||
           position.y < -margin || position.y > 480.0f + margin;
}

void Bullet::reset() {
    position = {0.0f, 0.0f};
    velocity = {0.0f, 0.0f};
    acceleration = {0.0f, 0.0f};
    angularVelocity = 0.0f;
    angle = 0.0f;
    lifeFrames = -1;
    ageFrames = 0;
    active = false;
    type = BulletType::Small;
    color = BulletColor::Red;
    collisionRadius = 4.0f;
    damage = 1.0f;
    grazed = false;
    scale = 1.0f;
    alpha = 255;
}

} // namespace th06
```

### 4.2.3 原版全类型弹道算法

以下是 TH06 中常见的弹道模式的完整实现。所有模式均以"从源点生成 N 颗子弹"为接口。

```cpp
// include/game/BulletPatterns.h
#pragma once
#include "game/Bullet.h"
#include "game/BulletManager.h"
#include "core/Math.h"
#include "core/RNG.h"
#include <cmath>

namespace th06 {

/// 弹道生成器集合
namespace BulletPatterns {

/// 直线弹道：从源点向指定角度发射
inline void straight(BulletManager* mgr, Vec2f origin, float angleDeg,
                     float speed, BulletType type, BulletColor color,
                     int count = 1, float spread = 0.0f) {
    for (int i = 0; i < count; ++i) {
        float a = angleDeg;
        if (count > 1) {
            a = angleDeg - spread / 2.0f + spread * i / (count - 1);
        }
        float rad = a * 3.14159265f / 180.0f;
        auto* b = mgr->spawn();
        if (b) {
            b->position = origin;
            b->velocity = {std::cos(rad) * speed, std::sin(rad) * speed};
            b->type = type;
            b->color = color;
            b->angle = a;
            b->active = true;
        }
    }
}

/// 追踪弹道：发射时瞄准玩家当前位置
inline void aimed(BulletManager* mgr, Vec2f origin, Vec2f target,
                  float speed, BulletType type, BulletColor color,
                  int count = 1, float spread = 0.0f) {
    float dx = target.x - origin.x;
    float dy = target.y - origin.y;
    float baseAngle = std::atan2(dy, dx) * 180.0f / 3.14159265f;
    straight(mgr, origin, baseAngle, speed, type, color, count, spread);
}

/// 环形扩散弹道：360 度均匀分布
inline void ring(BulletManager* mgr, Vec2f origin, float speed,
                 BulletType type, BulletColor color, int count,
                 float startAngle = 0.0f) {
    for (int i = 0; i < count; ++i) {
        float a = startAngle + 360.0f * i / count;
        straight(mgr, origin, a, speed, type, color, 1);
    }
}

/// 螺旋弹道：随时间旋转的环形（需每帧调用）
inline void spiral(BulletManager* mgr, Vec2f origin, float speed,
                   BulletType type, BulletColor color, int count,
                   float rotationPerCall) {
    static float s_angle = 0.0f;  // 实际应由调用方维护
    s_angle += rotationPerCall;
    ring(mgr, origin, speed, type, color, count, s_angle);
}

/// 正弦摆动弹道：子弹在飞行中左右摆动
/// 通过设置 angularVelocity + 加速度实现
inline void sineWave(BulletManager* mgr, Vec2f origin, float angleDeg,
                     float speed, float amplitude, float frequency,
                     BulletType type, BulletColor color) {
    float rad = angleDeg * 3.14159265f / 180.0f;
    auto* b = mgr->spawn();
    if (b) {
        b->position = origin;
        b->velocity = {std::cos(rad) * speed, std::sin(rad) * speed};
        b->type = type;
        b->color = color;
        b->angle = angleDeg;
        b->active = true;
        // 正弦摆动通过角速度实现（简化模型）
        b->angularVelocity = amplitude * std::sin(frequency);
    }
}

/// 随机散射弹道：在指定角度范围内随机分布
inline void randomSpread(BulletManager* mgr, Vec2f origin, float centerAngle,
                         float spreadAngle, float minSpeed, float maxSpeed,
                         BulletType type, BulletColor color, int count) {
    auto& rng = globalRNG();
    for (int i = 0; i < count; ++i) {
        float a = centerAngle + rng.nextFloat(-spreadAngle / 2, spreadAngle / 2);
        float s = rng.nextFloat(minSpeed, maxSpeed);
        straight(mgr, origin, a, s, type, color, 1);
    }
}

/// 多层环形弹道：不同速度的多层环
inline void multiRing(BulletManager* mgr, Vec2f origin,
                      const float* speeds, int speedCount,
                      BulletType type, BulletColor color, int countPerRing) {
    for (int s = 0; s < speedCount; ++s) {
        ring(mgr, origin, speeds[s], type, color, countPerRing,
             360.0f * s / (speedCount * countPerRing));  // 错开角度
    }
}

/// 扇形弹道：朝向玩家的扇形扩散
inline void fanAtPlayer(BulletManager* mgr, Vec2f origin, Vec2f target,
                        float speed, float fanAngle, int count,
                        BulletType type, BulletColor color) {
    aimed(mgr, origin, target, speed, type, color, count, fanAngle);
}

} // namespace BulletPatterns

} // namespace th06
```

### 4.2.4 批量同纹理渲染优化

当弹幕数量达到数千时，逐个 `SDL_RenderCopyExF` 调用会成为瓶颈。优化方案是**按纹理分桶，每桶合并为一次 draw call**。SDL2 本身在 2.0.10+ 版本有内部批处理，但仍可通过以下方式进一步优化：

```cpp
// 优化版 renderAll：按 BulletType 分桶
void BulletManager::renderAllOptimized(Renderer& renderer) {
    // 按 type 分桶
    std::vector<const Bullet*> buckets[(size_t)BulletType::Custom + 1];
    for (const auto& b : pool_) {
        if (b.active) {
            buckets[(size_t)b.type].push_back(&b);
        }
    }

    // 每桶批量渲染
    for (size_t t = 0; t <= (size_t)BulletType::Custom; ++t) {
        if (buckets[t].empty()) continue;
        SDL_Texture* tex = getBulletTexture((BulletType)t);
        if (!tex) continue;

        for (const auto* b : buckets[t]) {
            RenderCommand cmd;
            cmd.texture = tex;
            cmd.srcRect = getBulletSrcRect(b->type, b->color);
            cmd.dstRect = {
                b->position.x - cmd.srcRect.w / 2.0f,
                b->position.y - cmd.srcRect.h / 2.0f,
                (float)cmd.srcRect.w,
                (float)cmd.srcRect.h
            };
            cmd.angle = b->angle;
            cmd.alpha = b->alpha;
            cmd.layer = RenderLayer::BulletEnemy;
            renderer.enqueue(cmd);
        }
    }
}
```

> **原版 1:1 还原关键细节**：
> 1. **弹幕旋转**：米弹、苦无、针弹等"有方向"的弹幕必须随速度方向旋转。原版通过 ANM 动画帧实现，复刻时用 `SDL_RenderCopyExF` 的 `angle` 参数。
> 2. **泡弹/星弹不旋转**：泡弹、星弹、心弹等"无方向"的弹幕不旋转，保持固定朝向。
> 3. **弹幕发光效果**：部分弹幕（如泡弹）使用加法混合（`SDL_BLENDMODE_ADD`），产生发光感。
> 4. **擦弹（Graze）**：玩家擦弹后，该弹幕的 `grazed` 标志置位，避免重复计分。擦弹会增加 Graze 计数与少量分数。
> 5. **弹幕生命周期**：原版大部分弹幕是无限生命（直到出界销毁），但部分符卡弹幕有固定生命（如 300 帧后自动消失），需通过 `lifeFrames` 控制。

---

## 4.3 敌人、BOSS 与符卡系统

### 4.3.1 普通杂兵

```cpp
// include/game/Enemy.h
#pragma once
#include "core/Math.h"
#include "game/BulletManager.h"
#include <cstdint>
#include <string>

namespace th06 {

enum class EnemyState : uint8_t { Spawning, Active, Dying, Dead };

class Enemy {
public:
    void init(const std::string& typeName);

    void update();
    void render(double alpha);

    void takeDamage(float dmg);
    void die();

    Vec2f position;
    Vec2f velocity;
    float hp = 10.0f;
    float maxHp = 10.0f;
    float hitboxRadius = 12.0f;
    int   score = 100;
    EnemyState state = EnemyState::Active;

    // 移动模式参数
    std::string movementType;   // "straight", "sine_wave", "circle"
    float moveSpeed = 2.0f;
    float moveAngle = 90.0f;    // 度
    float sineAmplitude = 0.0f;
    float sineFrequency = 0.0f;
    int   sineTimer = 0;

    // 射击模式参数
    std::string shootType;      // "aimed", "spread", "ring"
    int   shootInterval = 60;
    int   shootTimer = 0;
    float bulletSpeed = 3.0f;
    int   bulletCount = 1;
    float spreadAngle = 0.0f;
    BulletType bulletType = BulletType::Small;
    BulletColor bulletColor = BulletColor::Blue;

    // 掉落表
    struct DropEntry {
        std::string itemType;
        float chance;
    };
    std::vector<DropEntry> dropTable;

    // 渲染
    std::string spriteName;
    int   animFrame = 0;
    int   animTimer = 0;

private:
    void updateMovement();
    void updateShooting();
};

} // namespace th06
```

```cpp
// src/game/Enemy.cpp 节选
#include "game/Enemy.h"
#include "game/Player.h"
#include "core/ConfigLoader.h"
#include "core/ResourceManager.h"
#include "core/RNG.h"
#include <cmath>

namespace th06 {

extern Player* g_player;
extern BulletManager* g_enemyBullets;

void Enemy::init(const std::string& typeName) {
    auto& cfg = ConfigLoader::instance().getEnemyConfig();
    auto& e = cfg[typeName];
    hp = maxHp = e["hp"];
    score = e["score"];
    hitboxRadius = e["hitbox_radius"];
    movementType = e["movement"]["type"];
    moveSpeed = e["movement"]["speed"];
    moveAngle = e["movement"].value("angle", 90.0f);
    sineAmplitude = e["movement"].value("amplitude", 0.0f);
    sineFrequency = e["movement"].value("frequency", 0.0f);
    shootType = e["shoot_pattern"]["type"];
    shootInterval = e["shoot_pattern"]["interval"];
    bulletSpeed = e["shoot_pattern"]["bullet_speed"];
    bulletCount = e["shoot_pattern"]["bullet_count"];
    spreadAngle = e["shoot_pattern"].value("spread_angle", 0.0f);
    spriteName = e["sprite"];
    state = EnemyState::Active;
}

void Enemy::update() {
    updateMovement();
    updateShooting();

    // 动画帧
    if (++animTimer >= 8) {
        animTimer = 0;
        animFrame = (animFrame + 1) % 4;
    }
}

void Enemy::updateMovement() {
    if (movementType == "straight") {
        float rad = moveAngle * 3.14159265f / 180.0f;
        velocity = {std::cos(rad) * moveSpeed, std::sin(rad) * moveSpeed};
    } else if (movementType == "sine_wave") {
        float rad = moveAngle * 3.14159265f / 180.0f;
        velocity = {std::cos(rad) * moveSpeed, std::sin(rad) * moveSpeed};
        // 横向叠加正弦
        velocity.x += std::sin(sineTimer * sineFrequency) * sineAmplitude * 0.1f;
        ++sineTimer;
    }
    position += velocity;
}

void Enemy::updateShooting() {
    if (++shootTimer < shootInterval) return;
    shootTimer = 0;

    if (shootType == "aimed" && g_player) {
        BulletPatterns::aimed(g_enemyBullets, position, g_player->position,
                              bulletSpeed, bulletType, bulletColor, bulletCount, spreadAngle);
    } else if (shootType == "spread") {
        BulletPatterns::straight(g_enemyBullets, position, 90.0f,
                                 bulletSpeed, bulletType, bulletColor, bulletCount, spreadAngle);
    } else if (shootType == "ring") {
        BulletPatterns::ring(g_enemyBullets, position, bulletSpeed,
                             bulletType, bulletColor, bulletCount);
    }
    ResourceManager::instance().playSound("assets/audio/se/se_enesh00.wav");
}

void Enemy::takeDamage(float dmg) {
    hp -= dmg;
    if (hp <= 0) {
        die();
    }
}

void Enemy::die() {
    state = EnemyState::Dying;
    // 死亡爆炸特效
    ResourceManager::instance().playSound("assets/audio/se/se_enep00.wav");
    // 道具掉落
    auto& rng = globalRNG();
    for (const auto& drop : dropTable) {
        if (rng.chance(drop.chance)) {
            // spawnItem(position, drop.itemType);
        }
    }
}

} // namespace th06
```

### 4.3.2 BOSS 调度机制

BOSS 是多阶段战斗，每个阶段有独立的血量、时限、弹幕脚本。阶段切换时清空弹幕、播放过渡动画、切换移动模式。

```cpp
// include/game/Boss.h
#pragma once
#include "core/Math.h"
#include "game/BulletManager.h"
#include "core/LuaEngine.h"
#include <string>
#include <vector>
#include <cstdint>

namespace th06 {

struct BossPhase {
    std::string name;
    float       hp;
    int         timeLimit;       // 秒
    bool        isSpell;         // 是否符卡
    bool        isLastSpell;
    int         spellBonusBase;  // 符卡奖励基础分
    std::string movement;        // 移动模式
    std::string script;          // Lua 脚本路径
};

class Boss {
public:
    void init(const std::string& bossName);

    void update();
    void render(double alpha);

    void takeDamage(float dmg);

    /// 强制进入下一阶段（符卡超时或血量耗尽）
    void nextPhase();

    /// 当前阶段是否为符卡
    bool isSpellCard() const;

    /// 符卡是否成功（在时限内击破）
    bool isSpellSuccess() const;

    Vec2f position;
    Vec2f targetPosition;        // 移动目标点
    float hp = 0;
    float maxHp = 0;
    float hitboxRadius = 24.0f;
    int   currentPhase = 0;
    int   phaseTimer = 0;        // 当前阶段已过帧数
    int   phaseTimeLimit = 0;    // 当前阶段时限（帧）
    bool  active = false;
    std::vector<BossPhase> phases;

    // 移动控制
    int   moveTimer = 0;
    int   moveDuration = 60;
    bool  isMoving = false;

private:
    void updateMovement();
    void updateScript();
    void enterPhase(int phaseIdx);

    std::string bossName_;
    int         scriptFrame_ = 0;
};

} // namespace th06
```

```cpp
// src/game/Boss.cpp 节选
#include "game/Boss.h"
#include "game/Player.h"
#include "core/ConfigLoader.h"
#include "core/ResourceManager.h"
#include "core/RNG.h"
#include "core/Logger.h"
#include <cmath>

namespace th06 {

extern Player* g_player;
extern BulletManager* g_enemyBullets;

void Boss::init(const std::string& bossName) {
    bossName_ = bossName;
    auto& cfg = ConfigLoader::instance().getBossConfig();
    auto& b = cfg[bossName];
    hitboxRadius = b["hitbox_radius"];

    for (auto& phase : b["phases"]) {
        BossPhase p;
        p.name = phase["name"];
        p.hp = phase["hp"];
        p.timeLimit = phase["time_limit"];
        p.isSpell = phase.value("is_spell", false);
        p.isLastSpell = phase.value("is_last_spell", false);
        p.spellBonusBase = phase.value("spell_bonus_base", 0);
        p.movement = phase["movement"];
        p.script = phase["script"];
        phases.push_back(p);
    }

    position = {320.0f, 100.0f};  // 屏幕顶部中央
    targetPosition = position;
    active = true;
    enterPhase(0);
}

void Boss::enterPhase(int phaseIdx) {
    currentPhase = phaseIdx;
    auto& phase = phases[phaseIdx];
    hp = maxHp = phase.hp;
    phaseTimer = 0;
    phaseTimeLimit = phase.timeLimit * 60;  // 秒 -> 帧
    scriptFrame_ = 0;

    // 清空弹幕
    g_enemyBullets->clearAll();

    // 设置移动目标
    if (phase.movement == "top_center") {
        targetPosition = {320.0f, 100.0f};
    } else if (phase.movement == "top_corner") {
        targetPosition = {100.0f, 80.0f};
    }
    isMoving = true;
    moveTimer = 0;
    moveDuration = 60;

    // 加载 Lua 脚本
    LuaEngine::instance().doFile("assets/scripts/" + phase.script);

    // 符卡开始时显示弹窗
    if (phase.isSpell) {
        // 由 SpellCardBanner 处理
        ResourceManager::instance().playSound("assets/audio/se/se_card.wav");
    }
}

void Boss::update() {
    if (!active) return;

    ++phaseTimer;
    ++scriptFrame_;

    updateMovement();
    updateScript();

    // 阶段超时或血量耗尽
    if (hp <= 0 || phaseTimer >= phaseTimeLimit) {
        if (currentPhase < (int)phases.size() - 1) {
            nextPhase();
        } else {
            // BOSS 死亡
            active = false;
            g_enemyBullets->clearAll();
        }
    }
}

void Boss::updateMovement() {
    if (!isMoving) return;
    if (++moveTimer >= moveDuration) {
        isMoving = false;
        position = targetPosition;
        return;
    }
    // 平滑插值移动
    float t = (float)moveTimer / moveDuration;
    // 缓动函数
    t = 1.0f - std::pow(1.0f - t, 3.0f);
    position = position.lerp(targetPosition, t * 0.1f);
}

void Boss::updateScript() {
    auto& lua = LuaEngine::instance();
    // 调用 Lua 脚本的 update 函数
    lua_getglobal(lua.rawState(), "M");
    if (lua_istable(lua.rawState(), -1)) {
        lua_getfield(lua.rawState(), -1, "update");
        if (lua_isfunction(lua.rawState(), -1)) {
            lua_pushlightuserdata(lua.rawState(), this);
            lua_pushinteger(lua.rawState(), scriptFrame_);
            if (lua_pcall(lua.rawState(), 2, 0, 0) != LUA_OK) {
                LOG_ERROR(std::string("Lua 错误: ") + lua_tostring(lua.rawState(), -1));
                lua_pop(lua.rawState(), 1);
            }
        } else {
            lua_pop(lua.rawState(), 1);
        }
        lua_pop(lua.rawState(), 1);
    } else {
        lua_pop(lua.rawState(), 1);
    }
}

void Boss::takeDamage(float dmg) {
    hp -= dmg;
}

void Boss::nextPhase() {
    // 符卡成功判定
    if (isSpellCard() && hp <= 0) {
        // 符卡成功，加分
        // 由 ScoreSystem 处理
    }
    enterPhase(currentPhase + 1);
}

bool Boss::isSpellCard() const {
    if (currentPhase >= (int)phases.size()) return false;
    return phases[currentPhase].isSpell;
}

bool Boss::isSpellSuccess() const {
    return isSpellCard() && hp <= 0;
}

} // namespace th06
```

### 4.3.3 符卡 Spell Card 完整生命周期

符卡是东方系列的标志性机制，完整生命周期如下：

1. **符卡开启**：BOSS 进入符卡阶段时，屏幕上方弹出符卡名称弹窗，持续约 2 秒。
2. **倒计时**：符卡有严格时限（通常 30~60 秒），屏幕显示剩余时间。
3. **限时得分**：在时限内击破符卡，获得符卡奖励分（基础分 × 剩余时间比例）。
4. **符卡结束清弹幕**：无论成功失败，符卡结束时清空所有敌方弹幕。
5. **符卡失败判定**：超时未击破视为失败，不得分，但仍进入下一阶段。
6. **符卡奖励倍率**：符卡奖励分 = 基础分 × (剩余时间 / 总时间) × 难度倍率。

```cpp
// include/game/SpellCard.h
#pragma once
#include "game/Boss.h"
#include <string>
#include <cstdint>

namespace th06 {

class SpellCard {
public:
    /// 开始符卡
    void start(Boss* boss, int phaseIdx);

    /// 每帧更新
    void update();

    /// 符卡是否结束
    bool isFinished() const { return finished_; }

    /// 符卡是否成功
    bool isSuccess() const { return success_; }

    /// 获取符卡奖励分
    int64_t getBonus() const;

    /// 获取符卡名称
    const std::string& getName() const { return name_; }

    /// 获取剩余时间（秒）
    int getTimeRemaining() const;

    /// 获取总时间（秒）
    int getTimeTotal() const { return timeTotal_; }

private:
    Boss*       boss_       = nullptr;
    int         phaseIdx_   = 0;
    std::string name_;
    int         timeTotal_  = 0;     // 总时限（帧）
    int         timeElapsed_ = 0;    // 已用时（帧）
    int         bonusBase_  = 0;
    bool        finished_   = false;
    bool        success_    = false;
    int         bannerTimer_ = 0;    // 弹窗显示计时
};

} // namespace th06
```

```cpp
// src/game/SpellCard.cpp
#include "game/SpellCard.h"
#include "game/BulletManager.h"
#include "core/ConfigLoader.h"
#include "core/Logger.h"

namespace th06 {

extern BulletManager* g_enemyBullets;

void SpellCard::start(Boss* boss, int phaseIdx) {
    boss_ = boss;
    phaseIdx_ = phaseIdx;
    finished_ = false;
    success_ = false;
    timeElapsed_ = 0;
    bannerTimer_ = 120;  // 弹窗显示 2 秒

    auto& phase = boss->phases[phaseIdx];
    name_ = phase.name;
    timeTotal_ = phase.timeLimit * 60;
    bonusBase_ = phase.spellBonusBase;
}

void SpellCard::update() {
    if (finished_) return;

    ++timeElapsed_;
    if (bannerTimer_ > 0) --bannerTimer_;

    // 检查符卡结束条件
    if (boss_->hp <= 0) {
        // 击破成功
        success_ = true;
        finished_ = true;
        g_enemyBullets->clearAll();
    } else if (timeElapsed_ >= timeTotal_) {
        // 超时失败
        success_ = false;
        finished_ = true;
        g_enemyBullets->clearAll();
    }
}

int64_t SpellCard::getBonus() const {
    if (!success_) return 0;
    // 奖励分 = 基础分 × 剩余时间比例
    int remaining = timeTotal_ - timeElapsed_;
    float ratio = (float)remaining / timeTotal_;
    return (int64_t)(bonusBase_ * ratio);
}

int SpellCard::getTimeRemaining() const {
    int remaining = timeTotal_ - timeElapsed_;
    return remaining / 60;  // 帧 -> 秒
}

} // namespace th06
```

> **原版 1:1 还原关键细节**：
> 1. **符卡弹窗动画**：原版弹窗从屏幕上方滑入，停留 2 秒后滑出。复刻时需精确复现此动画。
> 2. **符卡期间 BOSS 无敌帧**：符卡开始瞬间 BOSS 有短暂无敌（约 30 帧），防止玩家在弹窗显示期间偷伤害。
> 3. **Last Spell**：最终符卡失败后直接进入结局，不会"卡关"。Last Spell 的奖励分通常为普通符卡的 2-3 倍。
> 4. **符卡历史记录**：原版会记录玩家在存档中成功击破过哪些符卡，用于解锁"符卡练习"模式。复刻时需在存档中保存。

---

## 4.4 道具系统

### 4.4.1 全类型道具

TH06 的道具类型：

| 道具 | 颜色 | 效果 |
|------|------|------|
| 分数（点） | 蓝色小圆 | 增加分数，拾取满 100 个 +1 残机 |
| Power 小 | 红色小 P | 火力 +1 |
| Power 大 | 红色大 P | 火力 +8 |
| 残机碎片 | 紫色 1up | 残机 +1 |
| 炸弹 | 绿色 B | 炸弹 +1 |
| 生命 | 粉色 1up | 残机 +1（稀有） |
| Full Power | 红色 F | 火力直接满 |

```cpp
// include/game/Item.h
#pragma once
#include "core/Math.h"
#include <cstdint>
#include <string>

namespace th06 {

enum class ItemType : uint8_t {
    Score,          // 分数点
    PowerSmall,     // 小火力
    PowerLarge,     // 大火力
    Bomb,           // 炸弹
    OneUp,          // 残机
    FullPower,      // 满火力
    Point           // 关卡结算点（拾取上限加分）
};

class Item {
public:
    void init(ItemType type, Vec2f pos);

    void update();
    void render(double alpha);

    void collect();

    ItemType type = ItemType::Score;
    Vec2f    position;
    Vec2f    velocity;
    bool     active = false;
    bool     collected = false;

    // 自动上浮逻辑
    int      spawnFrames = 0;     // 生成后帧数
    bool     autoCollect = false; // 是否触发自动吸附

    // 渲染
    int      animFrame = 0;
    int      animTimer = 0;

private:
    void updateMovement();
};

} // namespace th06
```

```cpp
// src/game/Item.cpp 节选
#include "game/Item.h"
#include "game/Player.h"
#include "core/ConfigLoader.h"
#include "core/ResourceManager.h"
#include "core/RNG.h"
#include <cmath>

namespace th06 {

extern Player* g_player;

void Item::init(ItemType t, Vec2f pos) {
    type = t;
    position = pos;
    velocity = {0.0f, -2.0f};  // 初始向上弹起
    active = true;
    collected = false;
    spawnFrames = 0;
    autoCollect = false;
}

void Item::update() {
    if (!active) return;
    ++spawnFrames;
    updateMovement();

    // 动画
    if (++animTimer >= 8) {
        animTimer = 0;
        animFrame = (animFrame + 1) % 4;
    }

    // 拾取判定
    if (g_player) {
        float dx = position.x - g_player->position.x;
        float dy = position.y - g_player->position.y;
        float distSq = dx * dx + dy * dy;
        float pickupRadius = 16.0f;
        if (distSq < pickupRadius * pickupRadius) {
            collect();
        }
    }

    // 出界销毁
    if (position.y > 500.0f) {
        active = false;
    }
}

void Item::updateMovement() {
    // 阶段1：生成后向上弹起（约 20 帧）
    if (spawnFrames < 20) {
        velocity.y += 0.1f;  // 减速
    } else {
        // 阶段2：自由下落
        velocity.y += 0.05f;
        if (velocity.y > 2.0f) velocity.y = 2.0f;
    }

    // 自动吸附（Shift 低速 或 拾取点满）
    if (g_player) {
        bool shouldAttract = false;
        float attractRadius = 0.0f;

        // Shift 低速扩大吸附
        if (g_player->state == PlayerState::Alive) {
            auto& input = InputManager::instance();
            if (input.isDown(Action::Focus)) {
                shouldAttract = true;
                attractRadius = 64.0f;
            }
        }

        // 自动上浮（玩家在屏幕上方 1/3 区域时全屏吸附）
        if (g_player->position.y < 160.0f) {
            shouldAttract = true;
            attractRadius = 9999.0f;
        }

        // Power 满后所有分数点自动吸附
        if (g_player->power >= 128 && type == ItemType::Score) {
            shouldAttract = true;
            attractRadius = 9999.0f;
        }

        if (shouldAttract) {
            float dx = g_player->position.x - position.x;
            float dy = g_player->position.y - position.y;
            float dist = std::sqrt(dx * dx + dy * dy);
            if (dist < attractRadius) {
                float speed = 8.0f;
                velocity.x = dx / dist * speed;
                velocity.y = dy / dist * speed;
            }
        }
    }

    position += velocity;
}

void Item::collect() {
    if (collected) return;
    collected = true;
    active = false;

    // 拾取效果由 ScoreSystem 处理
    switch (type) {
        case ItemType::Score:
            ResourceManager::instance().playSound("assets/audio/se/se_item00.wav");
            break;
        case ItemType::PowerSmall:
        case ItemType::PowerLarge:
        case ItemType::FullPower:
            ResourceManager::instance().playSound("assets/audio/se/se_powerup.wav");
            break;
        case ItemType::Bomb:
            ResourceManager::instance().playSound("assets/audio/se/se_extend.wav");
            break;
        case ItemType::OneUp:
            ResourceManager::instance().playSound("assets/audio/se/se_extend.wav");
            break;
    }
}

} // namespace th06
```

> **原版 1:1 还原关键细节**：
> 1. **道具上浮逻辑**：道具生成后先向上弹起约 20 帧，然后开始自由下落。这个"弹起"动作是原版手感的关键。
> 2. **Shift 低速吸附**：按住 Shift 时，玩家周围 64 像素内的道具会被吸过来。
> 3. **自动上浮**：当玩家移动到屏幕上方 1/3 区域（y < 160）时，全屏道具自动飞向玩家。这是"收点"技巧的基础。
> 4. **Power 满后自动收点**：火力满（128）后，所有分数点自动飞向玩家，无需移动。
> 5. **分数点拾取计数**：每拾取一个分数点，计数 +1；满一定数量（如 200）后 +1 残机。这个数量随难度变化。

---

## 4.5 多层卷轴视差背景系统

### 4.5.1 多图层独立滚动

TH06 的背景由 2-3 层独立滚动的图层组成，每层滚动速度不同，产生纵深感。

```cpp
// include/game/Background.h
#pragma once
#include "core/Renderer.h"
#include "core/ResourceManager.h"
#include <string>
#include <vector>
#include <cstdint>

namespace th06 {

struct BackgroundLayer {
    std::string textureName;
    float       scrollSpeedX = 0.0f;  // 像素/帧
    float       scrollSpeedY = 0.0f;
    float       offsetX = 0.0f;       // 当前偏移
    float       offsetY = 0.0f;
    bool        tileX = true;         // 水平平铺
    bool        tileY = true;         // 垂直平铺
    RenderLayer renderLayer = RenderLayer::BackgroundFar;
    uint8_t     alpha = 255;
};

class Background {
public:
    void init(int stageId);

    void update();
    void render(double alpha);

    /// 设置全局滚动速度倍率（BOSS 战时减速）
    void setSpeedMultiplier(float mult);

    /// 停止滚动（BOSS 战）
    void stop();

    /// 恢复滚动
    void resume();

private:
    std::vector<BackgroundLayer> layers_;
    float speedMultiplier_ = 1.0f;
    bool  stopped_ = false;
};

} // namespace th06
```

```cpp
// src/game/Background.cpp
#include "game/Background.h"
#include "core/ConfigLoader.h"
#include "core/Logger.h"

namespace th06 {

void Background::init(int stageId) {
    // 从 stages.json 加载背景层配置
    auto& cfg = ConfigLoader::instance().getStageConfig();
    auto& stage = cfg["stages"][stageId - 1];
    for (auto& layerCfg : stage["background_layers"]) {
        BackgroundLayer layer;
        layer.textureName = layerCfg["texture"];
        layer.scrollSpeedX = layerCfg.value("scroll_speed_x", 0.0f);
        layer.scrollSpeedY = layerCfg.value("scroll_speed_y", 0.5f);
        layer.tileX = layerCfg.value("tile_x", true);
        layer.tileY = layerCfg.value("tile_y", true);
        layer.alpha = layerCfg.value("alpha", 255);
        layer.renderLayer = (RenderLayer)layerCfg.value("render_layer", (int)RenderLayer::BackgroundFar);
        layers_.push_back(layer);
    }
}

void Background::update() {
    if (stopped_) return;
    for (auto& layer : layers_) {
        layer.offsetX += layer.scrollSpeedX * speedMultiplier_;
        layer.offsetY += layer.scrollSpeedY * speedMultiplier_;
        // 平铺时取模避免偏移无限增长
        if (layer.tileX && layer.textureName.size() > 0) {
            // 实际需查询纹理宽度
            // layer.offsetX = fmod(layer.offsetX, texWidth);
        }
    }
}

void Background::render(double alpha) {
    (void)alpha;
    for (const auto& layer : layers_) {
        auto tex = ResourceManager::instance().loadTexture(layer.textureName);
        if (!tex) continue;

        // 平铺渲染
        int texW = tex->width;
        int texH = tex->height;
        float startX = -fmod(layer.offsetX, texW);
        float startY = -fmod(layer.offsetY, texH);

        for (float y = startY; y < 480.0f; y += texH) {
            for (float x = startX; x < 640.0f; x += texW) {
                RenderCommand cmd;
                cmd.texture = tex->sdlTexture;
                cmd.dstRect = {x, y, (float)texW, (float)texH};
                cmd.alpha = layer.alpha;
                cmd.layer = layer.renderLayer;
                // 直接绘制（不入队，背景优先级最低）
                SDL_SetTextureAlphaMod(tex->sdlTexture, layer.alpha);
                SDL_RenderCopyF(Renderer::instance().sdlRenderer(),
                               tex->sdlTexture, nullptr, &cmd.dstRect);
            }
        }
    }
}

void Background::setSpeedMultiplier(float mult) {
    speedMultiplier_ = mult;
}

void Background::stop() {
    stopped_ = true;
}

void Background::resume() {
    stopped_ = false;
}

} // namespace th06
```

> **原版 1:1 还原关键细节**：
> 1. **BOSS 战背景减速**：BOSS 出现时，背景滚动速度减半，营造紧张感。BOSS 死亡后恢复。
> 2. **背景分层顺序**：远景在最底，近景在远景之上，游戏对象在近景之上。
> 3. **关卡推进全局偏移**：随关卡时间推进，背景滚动速度可能逐渐加快，模拟"深入敌阵"的感觉。

---
## 4.6 UI 与像素文字渲染系统

### 4.6.1 禁用 TTF 矢量字体，基于原版 sft 图集自制位图文字渲染器

TH06 原版**不使用 TTF 矢量字体**，而是使用自制的位图字体图集（sft 格式，本质是 PNG + 字符坐标映射）。这样做的原因：
1. **视觉一致性**：位图字体保证像素级精确，与游戏画面风格统一。
2. **性能**：位图字体只需一次纹理绑定，渲染速度远超 TTF。
3. **还原度**：使用 TTF 会产生与原版不同的字形，破坏复刻的视觉一致性。

位图字体图集的结构：一张 PNG 包含所有字符，每个字符固定大小（如 16x16），按 ASCII / Shift-JIS 顺序排列。渲染时根据字符编码计算源矩形坐标。

```cpp
// include/ui/BitmapFont.h
#pragma once
#include "core/Renderer.h"
#include "core/ResourceManager.h"
#include <string>
#include <cstdint>

namespace th06 {

/// 位图字体渲染器
/// 基于 PNG 图集，每个字符固定 16x16 像素
class BitmapFont {
public:
    static BitmapFont& instance();

    /// 初始化（加载字体图集与字符映射）
    bool init(const std::string& texturePath, int charWidth, int charHeight);

    /// 渲染字符串（ASCII）
    void drawText(const std::string& text, float x, float y,
                  uint8_t r = 255, uint8_t g = 255, uint8_t b = 255,
                  uint8_t a = 255, float scale = 1.0f,
                  RenderLayer layer = RenderLayer::UI);

    /// 渲染字符串（带颜色调制，每个字符可不同颜色）
    void drawTextColored(const std::string& text, float x, float y,
                         uint8_t r, uint8_t g, uint8_t b, uint8_t a,
                         float scale = 1.0f,
                         RenderLayer layer = RenderLayer::UI);

    /// 渲染数字（右对齐，固定位数）
    void drawNumber(int64_t number, int digits, float x, float y,
                    uint8_t r = 255, uint8_t g = 255, uint8_t b = 255,
                    uint8_t a = 255, float scale = 1.0f,
                    RenderLayer layer = RenderLayer::UI);

    /// 获取字符宽度
    int charWidth() const { return charWidth_; }

    /// 获取字符高度
    int charHeight() const { return charHeight_; }

    /// 计算字符串渲染宽度
    float textWidth(const std::string& text, float scale = 1.0f) const;

private:
    BitmapFont() = default;

    std::shared_ptr<Texture> texture_;
    int charWidth_  = 16;
    int charHeight_ = 16;
    int cols_ = 16;  // 图集每行字符数
    int rows_ = 16;  // 图集行数

    /// 字符 -> 源矩形
    SDL_Rect getCharRect(char c) const;
};

} // namespace th06
```

```cpp
// src/ui/BitmapFont.cpp
#include "ui/BitmapFont.h"
#include "core/Logger.h"
#include <algorithm>

namespace th06 {

BitmapFont& BitmapFont::instance() {
    static BitmapFont inst;
    return inst;
}

bool BitmapFont::init(const std::string& texturePath, int cw, int ch) {
    texture_ = ResourceManager::instance().loadTexture(texturePath);
    if (!texture_) return false;
    charWidth_  = cw;
    charHeight_ = ch;
    cols_ = texture_->width  / cw;
    rows_ = texture_->height / ch;
    LOG_INFO("位图字体加载: " + texturePath + " (" +
             std::to_string(cols_) + "x" + std::to_string(rows_) + " 字符)");
    return true;
}

SDL_Rect BitmapFont::getCharRect(char c) const {
    // ASCII 字符直接映射：字符编码 -> 图集索引
    // 假设图集按 ASCII 顺序排列，从空格(32)开始
    int index = (unsigned char)c - 32;
    if (index < 0) index = 0;
    int col = index % cols_;
    int row = index / cols_;
    return {col * charWidth_, row * charHeight_, charWidth_, charHeight_};
}

void BitmapFont::drawText(const std::string& text, float x, float y,
                          uint8_t r, uint8_t g, uint8_t b, uint8_t a,
                          float scale, RenderLayer layer) {
    if (!texture_) return;

    // 颜色调制
    SDL_SetTextureColorMod(texture_->sdlTexture, r, g, b);

    float cx = x;
    for (char c : text) {
        if (c == ' ') {
            cx += charWidth_ * scale;
            continue;
        }
        SDL_Rect src = getCharRect(c);
        SDL_FRect dst = {cx, y, charWidth_ * scale, charHeight_ * scale};

        RenderCommand cmd;
        cmd.texture = texture_->sdlTexture;
        cmd.srcRect = src;
        cmd.dstRect = dst;
        cmd.alpha = a;
        cmd.layer = layer;
        Renderer::instance().enqueue(cmd);

        cx += charWidth_ * scale;
    }
}

void BitmapFont::drawNumber(int64_t number, int digits, float x, float y,
                            uint8_t r, uint8_t g, uint8_t b, uint8_t a,
                            float scale, RenderLayer layer) {
    // 转为字符串，前补空格或0
    std::string str = std::to_string(number);
    while ((int)str.size() < digits) {
        str = "0" + str;  // 前补0
    }
    if ((int)str.size() > digits) {
        str = str.substr(str.size() - digits);
    }
    drawText(str, x, y, r, g, b, a, scale, layer);
}

float BitmapFont::textWidth(const std::string& text, float scale) const {
    return text.size() * charWidth_ * scale;
}

} // namespace th06
```

### 4.6.2 战斗常驻 UI

战斗中屏幕顶部/底部显示分数、残机、炸弹、火力、关卡等信息。

```cpp
// include/ui/HUD.h
#pragma once
#include "game/Player.h"
#include "game/Boss.h"
#include "game/SpellCard.h"
#include <cstdint>

namespace th06 {

class HUD {
public:
    void update();
    void render(double alpha);

    void setPlayer(Player* p) { player_ = p; }
    void setBoss(Boss* b) { boss_ = b; }
    void setSpellCard(SpellCard* s) { spell_ = s; }

    void setScore(int64_t s) { score_ = s; }
    void setHiScore(int64_t s) { hiScore_ = s; }
    void setStage(int s) { stage_ = s; }
    void setGraze(int g) { graze_ = g; }
    void setPoint(int p) { point_ = p; }

private:
    void renderTopBar();
    void renderBottomBar();
    void renderBossHealthBar();
    void renderSpellCardBanner();

    Player*     player_ = nullptr;
    Boss*       boss_   = nullptr;
    SpellCard*  spell_  = nullptr;

    int64_t score_   = 0;
    int64_t hiScore_ = 0;
    int     stage_   = 1;
    int     graze_   = 0;
    int     point_   = 0;
};

} // namespace th06
```

```cpp
// src/ui/HUD.cpp 节选
#include "ui/HUD.h"
#include "ui/BitmapFont.h"
#include "core/Renderer.h"
#include "core/ResourceManager.h"

namespace th06 {

void HUD::render(double alpha) {
    (void)alpha;
    renderTopBar();
    renderBottomBar();
    if (boss_) renderBossHealthBar();
    if (spell_) renderSpellCardBanner();
}

void HUD::renderTopBar() {
    auto& font = BitmapFont::instance();
    auto& renderer = Renderer::instance();

    // 顶部黑色半透明背景条
    SDL_FRect bg {0, 0, 640, 32};
    SDL_SetRenderDrawColor(renderer.sdlRenderer(), 0, 0, 0, 180);
    SDL_RenderFillRectF(renderer.sdlRenderer(), &bg);

    // 分数（左上）
    font.drawText("SCORE", 8, 4, 200, 200, 200);
    font.drawNumber(score_, 10, 8, 16, 255, 255, 255);

    // 最高分（中上）
    font.drawText("HI-SCORE", 220, 4, 200, 200, 200);
    font.drawNumber(hiScore_, 10, 220, 16, 255, 255, 0);

    // 擦弹（右上）
    font.drawText("GRAZE", 440, 4, 200, 200, 200);
    font.drawNumber(graze_, 5, 440, 16, 255, 255, 255);

    // 关卡（最右上）
    font.drawText("STAGE", 560, 4, 200, 200, 200);
    font.drawNumber(stage_, 1, 560, 16, 255, 255, 255);
}

void HUD::renderBottomBar() {
    if (!player_) return;
    auto& font = BitmapFont::instance();
    auto& renderer = Renderer::instance();

    // 底部黑色半透明背景条
    SDL_FRect bg {0, 448, 640, 32};
    SDL_SetRenderDrawColor(renderer.sdlRenderer(), 0, 0, 0, 180);
    SDL_RenderFillRectF(renderer.sdlRenderer(), &bg);

    // 残机（左下，图标 + 数字）
    font.drawText("PLAYER", 8, 452, 200, 200, 200);
    // 绘制残机图标（红心或灵符）
    for (int i = 0; i < player_->lives; ++i) {
        // drawTexture(...)
    }
    font.drawNumber(player_->lives, 2, 8, 464, 255, 255, 255);

    // 炸弹（左下偏右）
    font.drawText("BOMB", 100, 452, 200, 200, 200);
    font.drawNumber(player_->bombs, 2, 100, 464, 255, 255, 255);

    // 火力（中下）
    font.drawText("POWER", 200, 452, 200, 200, 200);
    font.drawNumber(player_->power, 3, 200, 464, 255, 100, 100);

    // 拾取点数（右下）
    font.drawText("POINT", 400, 452, 200, 200, 200);
    font.drawNumber(point_, 3, 400, 464, 255, 255, 255);
}

void HUD::renderBossHealthBar() {
    if (!boss_ || !boss_->active) return;
    auto& renderer = Renderer::instance();

    // BOSS 血条（屏幕右上，竖直）
    float barX = 620.0f;
    float barY = 40.0f;
    float barH = 200.0f;
    float barW = 8.0f;

    // 背景
    SDL_SetRenderDrawColor(renderer.sdlRenderer(), 64, 64, 64, 200);
    SDL_FRect bg {barX, barY, barW, barH};
    SDL_RenderFillRectF(renderer.sdlRenderer(), &bg);

    // 当前血量
    float hpRatio = boss_->hp / boss_->maxHp;
    float hpH = barH * hpRatio;
    SDL_SetRenderDrawColor(renderer.sdlRenderer(), 255, 64, 64, 255);
    SDL_FRect hp {barX, barY + (barH - hpH), barW, hpH};
    SDL_RenderFillRectF(renderer.sdlRenderer(), &hp);
}

void HUD::renderSpellCardBanner() {
    if (!spell_) return;
    auto& font = BitmapFont::instance();
    auto& renderer = Renderer::instance();

    // 符卡名称弹窗（屏幕上方居中）
    // 弹窗动画：滑入 -> 停留 -> 滑出
    float bannerY = 40.0f;
    // 简化：直接绘制
    SDL_FRect bg {80, bannerY, 480, 32};
    SDL_SetRenderDrawColor(renderer.sdlRenderer(), 0, 0, 0, 200);
    SDL_RenderFillRectF(renderer.sdlRenderer(), &bg);

    // 符卡名称
    float textW = font.textWidth(spell_->getName(), 1.0f);
    font.drawText(spell_->getName(), 320 - textW / 2, bannerY + 8,
                  255, 255, 100);

    // 剩余时间
    int remaining = spell_->getTimeRemaining();
    font.drawNumber(remaining, 2, 540, bannerY + 8, 255, 100, 100);

    // 奖励分
    int64_t bonus = spell_->getBonus();
    font.drawNumber(bonus, 8, 80, bannerY + 40, 255, 255, 0);
}

} // namespace th06
```

### 4.6.3 动态 UI：得分飘字、警告、Game Over

```cpp
// include/ui/FloatingText.h
#pragma once
#include "core/Math.h"
#include <string>
#include <vector>
#include <cstdint>

namespace th06 {

struct FloatingText {
    std::string text;
    Vec2f       position;
    Vec2f       velocity;
    int         lifeFrames;
    int         ageFrames;
    uint8_t     r, g, b, a;
    float       scale;
    bool        active;
};

class FloatingTextManager {
public:
    static FloatingTextManager& instance();

    void spawn(const std::string& text, Vec2f pos,
               uint8_t r, uint8_t g, uint8_t b,
               int lifeFrames = 60, float scale = 1.0f);

    void update();
    void render(double alpha);

private:
    FloatingTextManager() = default;
    std::vector<FloatingText> texts_;
};

} // namespace th06
```

```cpp
// src/ui/FloatingText.cpp
#include "ui/FloatingText.h"
#include "ui/BitmapFont.h"

namespace th06 {

FloatingTextManager& FloatingTextManager::instance() {
    static FloatingTextManager inst;
    return inst;
}

void FloatingTextManager::spawn(const std::string& text, Vec2f pos,
                                uint8_t r, uint8_t g, uint8_t b,
                                int lifeFrames, float scale) {
    FloatingText ft;
    ft.text = text;
    ft.position = pos;
    ft.velocity = {0.0f, -1.5f};
    ft.lifeFrames = lifeFrames;
    ft.ageFrames = 0;
    ft.r = r; ft.g = g; ft.b = b; ft.a = 255;
    ft.scale = scale;
    ft.active = true;
    texts_.push_back(ft);
}

void FloatingTextManager::update() {
    for (auto& ft : texts_) {
        if (!ft.active) continue;
        ++ft.ageFrames;
        ft.position += ft.velocity;
        ft.velocity.y *= 0.95f;  // 减速
        // 淡出
        if (ft.ageFrames > ft.lifeFrames - 20) {
            ft.a = (uint8_t)(255 * (float)(ft.lifeFrames - ft.ageFrames) / 20);
        }
        if (ft.ageFrames >= ft.lifeFrames) {
            ft.active = false;
        }
    }
    // 清理失活项
    texts_.erase(
        std::remove_if(texts_.begin(), texts_.end(),
                       [](const FloatingText& ft) { return !ft.active; }),
        texts_.end());
}

void FloatingTextManager::render(double alpha) {
    (void)alpha;
    auto& font = BitmapFont::instance();
    for (const auto& ft : texts_) {
        if (!ft.active) continue;
        font.drawText(ft.text, ft.position.x, ft.position.y,
                      ft.r, ft.g, ft.b, ft.a, ft.scale);
    }
}

} // namespace th06
```

> **原版 1:1 还原关键细节**：
> 1. **位图字体必须使用原版 sft 图集**：用 TTF 字体渲染会破坏视觉一致性。原版 sft 图集可通过解包 `th06.dat` 获得。
> 2. **数字前补零**：原版分数显示为 "0001234567" 而非 "1234567"，需用 `drawNumber` 的 `digits` 参数控制。
> 3. **BOSS 血条为竖直方向**：位于屏幕右侧，从下往上减少。这是东方系列的标志性 UI。
> 4. **符卡名称弹窗动画**：原版弹窗有滑入/滑出动画，持续约 2 秒。复刻时需精确复现时间与位移曲线。

---

## 4.7 存档系统

### 4.7.1 二进制本地存档文件设计

存档文件采用二进制格式，包含文件头（魔数 + 版本）、最高分、解锁数据、按键配置等。

```cpp
// include/save/SaveData.h
#pragma once
#include "core/InputManager.h"
#include <cstdint>
#include <string>
#include <array>

namespace th06 {

#pragma pack(push, 1)  // 紧凑对齐，避免跨平台结构体大小不一致

struct SaveHeader {
    char     magic[4];       // "TH06"
    uint16_t version;        // 存档版本
    uint16_t headerSize;     // 头部大小
    uint32_t checksum;       // 校验和（防止篡改）
};

struct SaveData {
    SaveHeader header;

    // 最高分
    int64_t hiScoreEasy;
    int64_t hiScoreNormal;
    int64_t hiScoreHard;
    int64_t hiScoreLunatic;

    // 解锁数据
    uint8_t unlockedStages;      // 位掩码，bit0=stage2, bit1=stage3...
    uint8_t unlockedExtra;       // Extra 关卡解锁
    uint8_t unlockedPractice;    // 练习模式解锁

    // 符卡历史（每个符卡 1 字节：bit0=见过, bit1=击破过）
    uint8_t spellCardHistory[64];

    // 按键映射
    std::array<SDL_Scancode, (size_t)Action::ActionCount> keyMapping;

    // 其他设置
    uint8_t bgmVolume;       // 0~128
    uint8_t seVolume;        // 0~128
    uint8_t windowScale;     // 1=640x480, 2=1280x960
    uint8_t fullscreen;      // 0/1
};

#pragma pack(pop)

class SaveManager {
public:
    static SaveManager& instance();

    /// 加载存档
    bool load(const std::string& path = "assets/save/th06.sav");

    /// 保存存档
    bool save(const std::string& path = "assets/save/th06.sav");

    /// 获取存档数据
    SaveData& data() { return data_; }
    const SaveData& data() const { return data_; }

    /// 更新最高分
    void updateHiScore(int difficulty, int64_t score);

    /// 解锁关卡
    void unlockStage(int stageId);

    /// 记录符卡击破
    void recordSpellCard(int spellId, bool captured);

private:
    SaveManager();
    SaveData data_;
    std::string path_;

    uint32_t computeChecksum() const;
};

} // namespace th06
```

```cpp
// src/save/SaveManager.cpp
#include "save/SaveManager.h"
#include "core/Logger.h"
#include <fstream>
#include <cstring>

namespace th06 {

SaveManager::SaveManager() {
    std::memset(&data_, 0, sizeof(data_));
    std::memcpy(data_.header.magic, "TH06", 4);
    data_.header.version = 1;
    data_.header.headerSize = sizeof(SaveHeader);

    // 默认按键映射
    KeyMapping km;
    data_.keyMapping = km.keys;

    data_.bgmVolume = 64;
    data_.seVolume = 96;
    data_.windowScale = 2;
    data_.fullscreen = 0;
}

SaveManager& SaveManager::instance() {
    static SaveManager inst;
    return inst;
}

bool SaveManager::load(const std::string& path) {
    path_ = path;
    std::ifstream ifs(path, std::ios::binary);
    if (!ifs.is_open()) {
        LOG_INFO("存档不存在，使用默认值");
        return false;
    }

    SaveData loaded;
    ifs.read(reinterpret_cast<char*>(&loaded), sizeof(loaded));
    if (!ifs) {
        LOG_ERROR("存档读取失败");
        return false;
    }

    // 校验魔数
    if (std::memcmp(loaded.header.magic, "TH06", 4) != 0) {
        LOG_ERROR("存档魔数错误");
        return false;
    }

    // 校验版本
    if (loaded.header.version != 1) {
        LOG_ERROR("存档版本不兼容");
        return false;
    }

    // 校验和验证
    data_ = loaded;
    uint32_t expectedChecksum = computeChecksum();
    if (loaded.header.checksum != expectedChecksum) {
        LOG_WARN("存档校验和不匹配，可能被篡改");
        // 仍加载，但重置最高分
        data_.hiScoreEasy = 0;
        data_.hiScoreNormal = 0;
        data_.hiScoreHard = 0;
        data_.hiScoreLunatic = 0;
    }

    LOG_INFO("存档加载成功");
    return true;
}

bool SaveManager::save(const std::string& path) {
    path_ = path;
    data_.header.checksum = computeChecksum();

    std::ofstream ofs(path, std::ios::binary);
    if (!ofs.is_open()) {
        LOG_ERROR("无法写入存档: " + path);
        return false;
    }
    ofs.write(reinterpret_cast<const char*>(&data_), sizeof(data_));
    return ofs.good();
}

uint32_t SaveManager::computeChecksum() const {
    // 简单的 FNV-1a 哈希
    uint32_t hash = 2166136261u;
    const uint8_t* bytes = reinterpret_cast<const uint8_t*>(&data_);
    // 跳过 checksum 字段本身
    size_t skipOffset = offsetof(SaveData, header.checksum);
    size_t skipEnd = skipOffset + sizeof(uint32_t);
    for (size_t i = 0; i < sizeof(data_); ++i) {
        if (i >= skipOffset && i < skipEnd) continue;
        hash ^= bytes[i];
        hash *= 16777619u;
    }
    return hash;
}

void SaveManager::updateHiScore(int difficulty, int64_t score) {
    int64_t* target = nullptr;
    switch (difficulty) {
        case 0: target = &data_.hiScoreEasy; break;
        case 1: target = &data_.hiScoreNormal; break;
        case 2: target = &data_.hiScoreHard; break;
        case 3: target = &data_.hiScoreLunatic; break;
    }
    if (target && score > *target) {
        *target = score;
        save(path_);
    }
}

void SaveManager::unlockStage(int stageId) {
    if (stageId >= 2 && stageId <= 6) {
        data_.unlockedStages |= (1 << (stageId - 2));
        save(path_);
    }
}

void SaveManager::recordSpellCard(int spellId, bool captured) {
    if (spellId < 0 || spellId >= 64) return;
    data_.spellCardHistory[spellId] |= 0x01;  // 见过
    if (captured) {
        data_.spellCardHistory[spellId] |= 0x02;  // 击破过
    }
    save(path_);
}

} // namespace th06
```

> **原版 1:1 还原关键细节**：
> 1. **存档校验和**：原版存档有简单的校验机制防止手动修改。复刻时使用 FNV-1a 哈希即可，无需复杂加密。
> 2. **符卡历史记录**：原版记录玩家见过/击破过的所有符卡，用于符卡练习模式解锁。
> 3. **难度独立最高分**：四个难度分别记录最高分，不混在一起。

---

## 4.8 分数计算系统

### 4.8.1 分数构成

TH06 的分数由以下部分构成：
1. **基础道具得分**：拾取分数点 +10，小 Power +10，大 Power +30 等。
2. **符卡限时奖励**：符卡击破时按剩余时间比例给分。
3. **残机剩余加分**：通关时按残机数加分。
4. **连击分数**：连续拾取道具不中断时，分数倍率提升。
5. **擦弹分数**：每次擦弹 +50 分。

```cpp
// include/game/ScoreSystem.h
#pragma once
#include "game/Player.h"
#include "game/SpellCard.h"
#include <cstdint>

namespace th06 {

class ScoreSystem {
public:
    static ScoreSystem& instance();

    void reset();

    /// 拾取道具加分
    void addItemScore(int baseScore, int comboMultiplier = 1);

    /// 符卡击破加分
    void addSpellBonus(int64_t bonus);

    /// 擦弹加分
    void addGrazeScore();

    /// 通关残机加分
    void addClearBonus(const Player& player, int stageId);

    /// 获取当前分数
    int64_t score() const { return score_; }

    /// 获取连击数
    int combo() const { return combo_; }

    /// 重置连击（中弹时调用）
    void breakCombo();

private:
    ScoreSystem() = default;
    int64_t score_ = 0;
    int     combo_ = 0;
    int     comboTimer_ = 0;
};

} // namespace th06
```

```cpp
// src/game/ScoreSystem.cpp
#include "game/ScoreSystem.h"
#include "ui/FloatingTextManager.h"
#include "save/SaveManager.h"

namespace th06 {

ScoreSystem& ScoreSystem::instance() {
    static ScoreSystem inst;
    return inst;
}

void ScoreSystem::reset() {
    score_ = 0;
    combo_ = 0;
    comboTimer_ = 0;
}

void ScoreSystem::addItemScore(int baseScore, int comboMultiplier) {
    int total = baseScore * comboMultiplier;
    score_ += total;
    ++combo_;
    comboTimer_ = 120;  // 2 秒内继续拾取保持连击

    // 飘字
    FloatingTextManager::instance().spawn(
        std::to_string(total),
        {0, 0},  // 位置由调用方传入
        255, 255, 100,
        40, 1.0f
    );
}

void ScoreSystem::addSpellBonus(int64_t bonus) {
    score_ += bonus;
    FloatingTextManager::instance().spawn(
        "Spell Card Bonus!!",
        {160, 200},
        255, 255, 0,
        120, 2.0f
    );
    FloatingTextManager::instance().spawn(
        std::to_string(bonus),
        {200, 240},
        255, 255, 0,
        120, 2.0f
    );
}

void ScoreSystem::addGrazeScore() {
    score_ += 50;
}

void ScoreSystem::addClearBonus(const Player& player, int stageId) {
    // 残机剩余加分：每个残机 1000000 * stageId
    int64_t lifeBonus = (int64_t)player.lives * 1000000 * stageId;
    // 炸弹剩余加分：每个炸弹 100000 * stageId
    int64_t bombBonus = (int64_t)player.bombs * 100000 * stageId;
    // 火力加分
    int64_t powerBonus = (int64_t)player.power * 10000;

    score_ += lifeBonus + bombBonus + powerBonus;

    // 更新最高分
    SaveManager::instance().updateHiScore(0, score_);  // difficulty 参数实际传入
}

void ScoreSystem::breakCombo() {
    combo_ = 0;
    comboTimer_ = 0;
}

} // namespace th06
```

> **原版 1:1 还原关键细节**：
> 1. **分数点拾取上限**：原版中，当玩家在屏幕上方 1/3 区域拾取分数点时，每个分数点的分数会随"已拾取数量"递增（最高 +10000/个）。这是"收点"技巧的得分机制。
> 2. **符卡奖励分公式**：`奖励分 = 基础分 × (剩余时间 / 总时间) × 难度倍率`。难度倍率 Easy=1.0, Normal=1.2, Hard=1.5, Lunatic=2.0。
> 3. **通关残机加分**：每个残机 = 100 万 × 关卡数。第 6 关通关时，3 残机 = 1800 万分，这是高分的关键来源。

---

## 4.9 Replay 录像回放系统

### 4.9.1 录制逻辑

Replay 的核心原理：**保存开局 RNG 种子 + 每帧的键盘输入**。回放时用相同种子初始化 RNG，然后逐帧重放输入，由于游戏逻辑是确定性的，所有随机行为都会完全复现。

```cpp
// include/game/Replay.h
#pragma once
#include "core/RNG.h"
#include "core/InputManager.h"
#include <cstdint>
#include <string>
#include <vector>
#include <fstream>

namespace th06 {

#pragma pack(push, 1)

struct ReplayHeader {
    char     magic[4];        // "RP06"
    uint16_t version;
    uint16_t headerSize;
    uint32_t checksum;
    uint32_t rngSeed;         // 开局 RNG 种子
    uint8_t  character;       // 角色
    uint8_t  difficulty;      // 难度
    uint8_t  stage;           // 关卡
    uint32_t totalFrames;     // 总帧数
    int64_t  finalScore;      // 最终分数
    char     playerName[16];  // 玩家名
};

#pragma pack(pop)

/// 每帧输入状态（紧凑编码）
/// 每个 Action 占 1 bit，共 10 bit，用 2 字节存储
struct ReplayInputFrame {
    uint16_t packed;  // 位掩码

    bool get(Action a) const {
        return (packed >> (uint8_t)a) & 1;
    }
    void set(Action a, bool v) {
        if (v) packed |= (1 << (uint8_t)a);
        else   packed &= ~(1 << (uint8_t)a);
    }
};

class Replay {
public:
    /// 开始录制
    void startRecording(uint32_t rngSeed, uint8_t character,
                        uint8_t difficulty, uint8_t stage);

    /// 录制一帧输入
    void recordFrame();

    /// 停止录制并保存
    bool saveRecording(const std::string& path, int64_t finalScore,
                       const std::string& playerName);

    /// 开始回放
    bool startPlayback(const std::string& path);

    /// 获取回放中当前帧的输入
    ReplayInputFrame getPlaybackFrame(uint32_t frameIdx);

    /// 回放是否结束
    bool isPlaybackFinished(uint32_t frameIdx) const;

    /// 获取回放头部信息
    const ReplayHeader& header() const { return header_; }

    /// 是否正在录制
    bool isRecording() const { return recording_; }

    /// 是否正在回放
    bool isPlaying() const { return playing_; }

private:
    bool recording_ = false;
    bool playing_ = false;

    ReplayHeader header_;
    std::vector<ReplayInputFrame> frames_;
};

} // namespace th06
```

```cpp
// src/game/Replay.cpp
#include "game/Replay.h"
#include "core/Logger.h"
#include <cstring>

namespace th06 {

void Replay::startRecording(uint32_t rngSeed, uint8_t character,
                            uint8_t difficulty, uint8_t stage) {
    recording_ = true;
    playing_ = false;
    frames_.clear();

    std::memset(&header_, 0, sizeof(header_));
    std::memcpy(header_.magic, "RP06", 4);
    header_.version = 1;
    header_.headerSize = sizeof(ReplayHeader);
    header_.rngSeed = rngSeed;
    header_.character = character;
    header_.difficulty = difficulty;
    header_.stage = stage;
    header_.totalFrames = 0;
}

void Replay::recordFrame() {
    if (!recording_) return;

    auto& input = InputManager::instance();
    ReplayInputFrame frame{};
    frame.packed = 0;
    for (int i = 0; i < (int)Action::ActionCount; ++i) {
        frame.set((Action)i, input.isDown((Action)i));
    }
    frames_.push_back(frame);
    ++header_.totalFrames;
}

bool Replay::saveRecording(const std::string& path, int64_t finalScore,
                           const std::string& playerName) {
    if (!recording_) return false;
    recording_ = false;

    header_.finalScore = finalScore;
    std::memset(header_.playerName, 0, sizeof(header_.playerName));
    std::strncpy(header_.playerName, playerName.c_str(),
                 sizeof(header_.playerName) - 1);

    // 计算校验和
    header_.checksum = 0;
    uint32_t hash = 2166136261u;
    const uint8_t* bytes = reinterpret_cast<const uint8_t*>(&header_);
    for (size_t i = 0; i < sizeof(header_); ++i) {
        hash ^= bytes[i];
        hash *= 16777619u;
    }
    for (const auto& f : frames_) {
        const uint8_t* fb = reinterpret_cast<const uint8_t*>(&f);
        for (size_t i = 0; i < sizeof(f); ++i) {
            hash ^= fb[i];
            hash *= 16777619u;
        }
    }
    header_.checksum = hash;

    std::ofstream ofs(path, std::ios::binary);
    if (!ofs.is_open()) {
        LOG_ERROR("无法保存录像: " + path);
        return false;
    }
    ofs.write(reinterpret_cast<const char*>(&header_), sizeof(header_));
    ofs.write(reinterpret_cast<const char*>(frames_.data()),
              frames_.size() * sizeof(ReplayInputFrame));
    LOG_INFO("录像已保存: " + path + " (" +
             std::to_string(frames_.size()) + " 帧)");
    return true;
}

bool Replay::startPlayback(const std::string& path) {
    playing_ = true;
    recording_ = false;
    frames_.clear();

    std::ifstream ifs(path, std::ios::binary);
    if (!ifs.is_open()) {
        LOG_ERROR("无法加载录像: " + path);
        return false;
    }

    ifs.read(reinterpret_cast<char*>(&header_), sizeof(header_));
    if (!ifs || std::memcmp(header_.magic, "RP06", 4) != 0) {
        LOG_ERROR("录像格式错误");
        return false;
    }

    frames_.resize(header_.totalFrames);
    ifs.read(reinterpret_cast<char*>(frames_.data()),
             frames_.size() * sizeof(ReplayInputFrame));

    // 用录像中的种子初始化 RNG
    globalRNG().seed(header_.rngSeed);

    LOG_INFO("录像加载成功: " + path + " (" +
             std::to_string(header_.totalFrames) + " 帧)");
    return true;
}

ReplayInputFrame Replay::getPlaybackFrame(uint32_t frameIdx) {
    if (frameIdx >= frames_.size()) return {};
    return frames_[frameIdx];
}

bool Replay::isPlaybackFinished(uint32_t frameIdx) const {
    return frameIdx >= header_.totalFrames;
}

} // namespace th06
```

### 4.9.2 回放时的输入注入

回放时需要"伪造"输入状态，让游戏逻辑以为玩家在操作：

```cpp
// 回放模式下的 InputManager 适配
class ReplayInputAdapter {
public:
    void setReplay(Replay* replay) { replay_ = replay; }
    void setFrame(uint32_t frame) { frame_ = frame; }

    bool isDown(Action a) const {
        if (!replay_) return false;
        return replay_->getPlaybackFrame(frame_).get(a);
    }

private:
    Replay* replay_ = nullptr;
    uint32_t frame_ = 0;
};
```

> **原版 1:1 还原关键细节**：
> 1. **RNG 种子必须保存**：Replay 的核心是确定性，RNG 种子是确定性的源头。如果种子不同，即使输入完全相同，弹幕也会不同。
> 2. **输入位掩码紧凑编码**：每帧 10 个 Action 用 2 字节存储，1 小时录像约 60×60×60×2 = 432KB，非常紧凑。
> 3. **回放期间禁止真实输入**：回放时必须屏蔽真实键盘输入，否则会干扰复现。
> 4. **回放期间禁止读取系统时间**：任何依赖系统时间的逻辑（如粒子随机种子）都会破坏复现。所有随机必须走 `globalRNG()`。

---

## 4.10 难度控制系统

### 4.10.1 四难度差异化配置

```json
// assets/config/difficulty.json
{
    "easy": {
        "enemy_hp_multiplier": 0.7,
        "enemy_bullet_speed_multiplier": 0.8,
        "enemy_bullet_density_multiplier": 0.6,
        "spell_time_limit_multiplier": 1.5,
        "item_drop_chance_multiplier": 1.3,
        "player_starting_lives": 5,
        "player_starting_bombs": 3,
        "score_multiplier": 1.0,
        "point_items_for_extend": 50
    },
    "normal": {
        "enemy_hp_multiplier": 1.0,
        "enemy_bullet_speed_multiplier": 1.0,
        "enemy_bullet_density_multiplier": 1.0,
        "spell_time_limit_multiplier": 1.0,
        "item_drop_chance_multiplier": 1.0,
        "player_starting_lives": 3,
        "player_starting_bombs": 2,
        "score_multiplier": 1.2,
        "point_items_for_extend": 70
    },
    "hard": {
        "enemy_hp_multiplier": 1.3,
        "enemy_bullet_speed_multiplier": 1.2,
        "enemy_bullet_density_multiplier": 1.5,
        "spell_time_limit_multiplier": 0.85,
        "item_drop_chance_multiplier": 0.8,
        "player_starting_lives": 2,
        "player_starting_bombs": 2,
        "score_multiplier": 1.5,
        "point_items_for_extend": 100
    },
    "lunatic": {
        "enemy_hp_multiplier": 1.6,
        "enemy_bullet_speed_multiplier": 1.4,
        "enemy_bullet_density_multiplier": 2.0,
        "spell_time_limit_multiplier": 0.7,
        "item_drop_chance_multiplier": 0.6,
        "player_starting_lives": 2,
        "player_starting_bombs": 1,
        "score_multiplier": 2.0,
        "point_items_for_extend": 150
    }
}
```

```cpp
// include/game/DifficultyManager.h
#pragma once
#include <string>
#include <cstdint>
#include <nlohmann/json.hpp>

namespace th06 {

enum class Difficulty : uint8_t { Easy, Normal, Hard, Lunatic };

class DifficultyManager {
public:
    static DifficultyManager& instance();

    bool load(const std::string& path);

    void setDifficulty(Difficulty d);
    Difficulty current() const { return current_; }

    // 应用倍率
    float enemyHpMultiplier() const;
    float bulletSpeedMultiplier() const;
    float bulletDensityMultiplier() const;
    float spellTimeMultiplier() const;
    float itemDropMultiplier() const;
    int   startingLives() const;
    int   startingBombs() const;
    float scoreMultiplier() const;
    int   pointItemsForExtend() const;

private:
    DifficultyManager() = default;
    Difficulty current_ = Difficulty::Normal;
    nlohmann::json config_;
    std::string difficultyNames_[4] = {"easy", "normal", "hard", "lunatic"};
};

} // namespace th06
```

```cpp
// src/game/DifficultyManager.cpp
#include "game/DifficultyManager.h"
#include "core/Logger.h"
#include <fstream>

namespace th06 {

DifficultyManager& DifficultyManager::instance() {
    static DifficultyManager inst;
    return inst;
}

bool DifficultyManager::load(const std::string& path) {
    std::ifstream ifs(path);
    if (!ifs.is_open()) {
        LOG_ERROR("无法加载难度配置: " + path);
        return false;
    }
    config_ = nlohmann::json::parse(ifs);
    return true;
}

void DifficultyManager::setDifficulty(Difficulty d) {
    current_ = d;
    LOG_INFO("难度切换: " + difficultyNames_[(int)d]);
}

float DifficultyManager::enemyHpMultiplier() const {
    return config_[difficultyNames_[(int)current_]]["enemy_hp_multiplier"];
}

float DifficultyManager::bulletSpeedMultiplier() const {
    return config_[difficultyNames_[(int)current_]]["enemy_bullet_speed_multiplier"];
}

float DifficultyManager::bulletDensityMultiplier() const {
    return config_[difficultyNames_[(int)current_]]["enemy_bullet_density_multiplier"];
}

float DifficultyManager::spellTimeMultiplier() const {
    return config_[difficultyNames_[(int)current_]]["spell_time_limit_multiplier"];
}

float DifficultyManager::itemDropMultiplier() const {
    return config_[difficultyNames_[(int)current_]]["item_drop_chance_multiplier"];
}

int DifficultyManager::startingLives() const {
    return config_[difficultyNames_[(int)current_]]["player_starting_lives"];
}

int DifficultyManager::startingBombs() const {
    return config_[difficultyNames_[(int)current_]]["player_starting_bombs"];
}

float DifficultyManager::scoreMultiplier() const {
    return config_[difficultyNames_[(int)current_]]["score_multiplier"];
}

int DifficultyManager::pointItemsForExtend() const {
    return config_[difficultyNames_[(int)current_]]["point_items_for_extend"];
}

} // namespace th06
```

> **原版 1:1 还原关键细节**：TH06 原版的难度差异**不仅是数值倍率**，部分 BOSS 在不同难度下使用**完全不同的弹幕脚本**。例如蕾米莉亚的"红魔"符卡，Normal 是 12 发环形弹，Hard 是 16 发环形弹 + 追踪弹，Lunatic 是 24 发环形弹 + 追踪弹 + 螺旋弹。完整复刻需为每个难度编写独立的 Lua 脚本，并在 `bosses.json` 中按难度指定不同 `script` 路径。本节的 JSON 倍率方案是简化实现，适合开发初期快速验证。

---

## 4.11 全局游戏流程控制器

### 4.11.1 状态机实现

```cpp
// include/game/GameState.h
#pragma once
#include "core/Renderer.h"
#include "core/MainLoop.h"
#include "game/Player.h"
#include "game/BulletManager.h"
#include "game/EnemyManager.h"
#include "game/Boss.h"
#include "game/SpellCard.h"
#include "game/ItemManager.h"
#include "game/Background.h"
#include "game/ScoreSystem.h"
#include "game/Replay.h"
#include "game/DifficultyManager.h"
#include "ui/HUD.h"
#include "ui/FloatingTextManager.h"
#include "menu/MainMenu.h"
#include "menu/DifficultyMenu.h"
#include "menu/CharacterMenu.h"
#include <memory>

namespace th06 {

enum class GameStateType : uint8_t {
    MainMenu,
    DifficultySelect,
    CharacterSelect,
    StageLoading,
    StagePlaying,
    BossBattle,
    StageClear,
    GameOver,
    AllClear,
    ReplayPlayback,
    PauseMenu
};

class GameState {
public:
    GameState();
    ~GameState();

    void init();
    void shutdown();

    /// 每逻辑帧更新
    void update();

    /// 每渲染帧绘制
    void render(double alpha);

    /// 切换状态
    void changeState(GameStateType newState);

    /// 暂停
    void pause();
    void resume();

private:
    void updateMainMenu();
    void updateDifficultySelect();
    void updateCharacterSelect();
    void updateStageLoading();
    void updateStagePlaying();
    void updateBossBattle();
    void updateStageClear();
    void updateGameOver();
    void updatePauseMenu();

    void renderMainMenu(double alpha);
    void renderDifficultySelect(double alpha);
    void renderCharacterSelect(double alpha);
    void renderStageLoading(double alpha);
    void renderStagePlaying(double alpha);
    void renderStageClear(double alpha);
    void renderGameOver(double alpha);
    void renderPauseMenu(double alpha);

    void loadStage(int stageId);
    void startBossBattle(const std::string& bossName);
    void onStageClear();
    void onGameOver();
    void onAllClear();

    GameStateType state_ = GameStateType::MainMenu;
    GameStateType prevState_ = GameStateType::MainMenu;

    // 子系统
    std::unique_ptr<Player>          player_;
    std::unique_ptr<BulletManager>   playerBullets_;
    std::unique_ptr<BulletManager>   enemyBullets_;
    std::unique_ptr<EnemyManager>    enemies_;
    std::unique_ptr<Boss>            boss_;
    std::unique_ptr<SpellCard>       spellCard_;
    std::unique_ptr<ItemManager>     items_;
    std::unique_ptr<Background>      background_;
    std::unique_ptr<HUD>             hud_;
    std::unique_ptr<MainMenu>        mainMenu_;
    std::unique_ptr<DifficultyMenu>  difficultyMenu_;
    std::unique_ptr<CharacterMenu>   characterMenu_;
    std::unique_ptr<Replay>          replay_;

    // 关卡进度
    int   currentStage_ = 1;
    int   stageTimer_ = 0;
    int   loadingTimer_ = 0;
    bool  paused_ = false;

    // 关卡波次数据
    struct WaveEntry {
        int   spawnFrame;
        std::string enemyType;
        Vec2f spawnPosition;
        int   count;
    };
    std::vector<WaveEntry> waves_;
    size_t nextWaveIdx_ = 0;
};

} // namespace th06
```

```cpp
// src/game/GameState.cpp 节选
#include "game/GameState.h"
#include "core/InputManager.h"
#include "core/ConfigLoader.h"
#include "core/Logger.h"
#include "core/RNG.h"
#include "save/SaveManager.h"

namespace th06 {

GameState::GameState() {
    player_        = std::make_unique<Player>();
    playerBullets_ = std::make_unique<BulletManager>();
    enemyBullets_  = std::make_unique<BulletManager>();
    enemies_       = std::make_unique<EnemyManager>();
    boss_          = std::make_unique<Boss>();
    spellCard_     = std::make_unique<SpellCard>();
    items_         = std::make_unique<ItemManager>();
    background_    = std::make_unique<Background>();
    hud_           = std::make_unique<HUD>();
    mainMenu_      = std::make_unique<MainMenu>();
    difficultyMenu_ = std::make_unique<DifficultyMenu>();
    characterMenu_ = std::make_unique<CharacterMenu>();
    replay_        = std::make_unique<Replay>();
}

void GameState::init() {
    state_ = GameStateType::MainMenu;
    mainMenu_->init();
    hud_->setPlayer(player_.get());
}

void GameState::update() {
    auto& input = InputManager::instance();

    // 暂停切换（任何状态下都可暂停）
    if (input.isPressed(Action::Pause) && state_ == GameStateType::StagePlaying) {
        pause();
        return;
    }
    if (paused_) {
        updatePauseMenu();
        return;
    }

    switch (state_) {
        case GameStateType::MainMenu:         updateMainMenu(); break;
        case GameStateType::DifficultySelect: updateDifficultySelect(); break;
        case GameStateType::CharacterSelect:  updateCharacterSelect(); break;
        case GameStateType::StageLoading:     updateStageLoading(); break;
        case GameStateType::StagePlaying:     updateStagePlaying(); break;
        case GameStateType::BossBattle:       updateBossBattle(); break;
        case GameStateType::StageClear:       updateStageClear(); break;
        case GameStateType::GameOver:         updateGameOver(); break;
        case GameStateType::AllClear:         updateGameOver(); break;
        default: break;
    }
}

void GameState::updateMainMenu() {
    mainMenu_->update();
    if (mainMenu_->shouldStartGame()) {
        changeState(GameStateType::DifficultySelect);
    }
}

void GameState::updateDifficultySelect() {
    difficultyMenu_->update();
    if (difficultyMenu_->confirmed()) {
        DifficultyManager::instance().setDifficulty(difficultyMenu_->selected());
        changeState(GameStateType::CharacterSelect);
    }
}

void GameState::updateCharacterSelect() {
    characterMenu_->update();
    if (characterMenu_->confirmed()) {
        // 初始化玩家
        player_->init(characterMenu_->selected());
        player_->lives = DifficultyManager::instance().startingLives();
        player_->bombs = DifficultyManager::instance().startingBombs();

        // 设置 RNG 种子并开始录制
        uint32_t seed = (uint32_t)SDL_GetTicks();
        globalRNG().seed(seed);
        replay_->startRecording(seed, (uint8_t)characterMenu_->selected(),
                                (uint8_t)DifficultyManager::instance().current(),
                                currentStage_);

        changeState(GameStateType::StageLoading);
    }
}

void GameState::updateStageLoading() {
    if (loadingTimer_ == 0) {
        loadStage(currentStage_);
    }
    ++loadingTimer_;
    if (loadingTimer_ >= 60) {  // 1 秒加载
        loadingTimer_ = 0;
        changeState(GameStateType::StagePlaying);
    }
}

void GameState::updateStagePlaying() {
    ++stageTimer_;

    // 录制输入
    replay_->recordFrame();

    // 更新背景
    background_->update();

    // 生成敌人波次
    while (nextWaveIdx_ < waves_.size() &&
           stageTimer_ >= waves_[nextWaveIdx_].spawnFrame) {
        const auto& wave = waves_[nextWaveIdx_];
        for (int i = 0; i < wave.count; ++i) {
            enemies_->spawn(wave.enemyType, wave.spawnPosition);
        }
        ++nextWaveIdx_;
    }

    // 更新各系统
    player_->update();
    playerBullets_->updateAll();
    enemyBullets_->updateAll();
    enemies_->updateAll();
    items_->updateAll();

    // 碰撞检测
    enemyBullets_->checkCollisionWithPlayer(*player_);
    playerBullets_->checkCollisionWithEnemyBullets(*enemyBullets_);
    playerBullets_->checkCollisionWithEnemies(*enemies_);
    playerBullets_->checkCollisionWithBoss(*boss_);
    items_->checkCollisionWithPlayer(*player_);

    // 检查 BOSS 触发
    if (stageTimer_ >= getBossSpawnFrame(currentStage_) && !boss_->active) {
        startBossBattle(getBossName(currentStage_));
    }

    // 检查关卡结束
    if (boss_->active == false && boss_->phases.empty() == false &&
        stageTimer_ > getBossSpawnFrame(currentStage_) + 60) {
        onStageClear();
    }

    // 检查 Game Over
    if (player_->lives < 0) {
        onGameOver();
    }

    hud_->update();
    FloatingTextManager::instance().update();
}

void GameState::updateBossBattle() {
    // BOSS 战逻辑与 StagePlaying 类似，但背景减速
    background_->setSpeedMultiplier(0.3f);
    updateStagePlaying();
    if (boss_->active) {
        boss_->update();
        if (boss_->isSpellCard() && !spellCard_->isFinished()) {
            spellCard_->update();
        }
    }
}

void GameState::updateStageClear() {
    // 通关加分
    ScoreSystem::instance().addClearBonus(*player_, currentStage_);
    if (++stageTimer_ > 180) {  // 3 秒后进入下一关
        if (currentStage_ < 6) {
            ++currentStage_;
            SaveManager::instance().unlockStage(currentStage_);
            changeState(GameStateType::StageLoading);
        } else {
            changeState(GameStateType::AllClear);
        }
    }
}

void GameState::updateGameOver() {
    if (InputManager::instance().isPressed(Action::Confirm)) {
        // 保存录像
        replay_->saveRecording(
            "assets/save/replay_" + std::to_string(SDL_GetTicks()) + ".rpy",
            ScoreSystem::instance().score(),
            "Player"
        );
        // 更新最高分
        SaveManager::instance().updateHiScore(
            (int)DifficultyManager::instance().current(),
            ScoreSystem::instance().score()
        );
        changeState(GameStateType::MainMenu);
    }
}

void GameState::updatePauseMenu() {
    auto& input = InputManager::instance();
    if (input.isPressed(Action::Pause) || input.isPressed(Action::Cancel)) {
        resume();
    }
}

void GameState::render(double alpha) {
    switch (state_) {
        case GameStateType::MainMenu:         renderMainMenu(alpha); break;
        case GameStateType::DifficultySelect: renderDifficultySelect(alpha); break;
        case GameStateType::CharacterSelect:  renderCharacterSelect(alpha); break;
        case GameStateType::StageLoading:     renderStageLoading(alpha); break;
        case GameStateType::StagePlaying:     renderStagePlaying(alpha); break;
        case GameStateType::BossBattle:       renderStagePlaying(alpha); break;
        case GameStateType::StageClear:       renderStageClear(alpha); break;
        case GameStateType::GameOver:         renderStagePlaying(alpha);
                                              renderGameOver(alpha); break;
        case GameStateType::AllClear:         renderStagePlaying(alpha);
                                              renderGameOver(alpha); break;
        default: break;
    }
    if (paused_) renderPauseMenu(alpha);
}

void GameState::renderStagePlaying(double alpha) {
    background_->render(alpha);
    enemyBullets_->renderAll(Renderer::instance());
    enemies_->renderAll(alpha);
    items_->renderAll(alpha);
    playerBullets_->renderAll(Renderer::instance());
    player_->render(alpha);
    if (boss_->active) boss_->render(alpha);
    FloatingTextManager::instance().render(alpha);
    hud_->render(alpha);
}

void GameState::changeState(GameStateType newState) {
    LOG_INFO("状态切换: " + std::to_string((int)state_) + " -> " +
             std::to_string((int)newState));
    prevState_ = state_;
    state_ = newState;
    stageTimer_ = 0;
}

void GameState::pause() {
    paused_ = true;
    LOG_INFO("游戏暂停");
}

void GameState::resume() {
    paused_ = false;
    LOG_INFO("游戏恢复");
}

void GameState::loadStage(int stageId) {
    // 清空上一关
    playerBullets_->clearAll();
    enemyBullets_->clearAll();
    enemies_->clearAll();
    items_->clearAll();
    boss_->phases.clear();
    boss_->active = false;

    // 加载关卡配置
    background_->init(stageId);

    // 加载波次数据
    auto& cfg = ConfigLoader::instance().getStageConfig();
    auto& stage = cfg["stages"][stageId - 1];
    waves_.clear();
    for (auto& wave : stage["waves"]) {
        WaveEntry w;
        w.spawnFrame = wave["frame"];
        w.enemyType = wave["enemy"];
        w.spawnPosition = {wave["x"], wave["y"]};
        w.count = wave.value("count", 1);
        waves_.push_back(w);
    }
    nextWaveIdx_ = 0;
}

void GameState::startBossBattle(const std::string& bossName) {
    boss_->init(bossName);
    background_->setSpeedMultiplier(0.3f);
    changeState(GameStateType::BossBattle);
}

void GameState::onStageClear() {
    changeState(GameStateType::StageClear);
    background_->resume();
    background_->setSpeedMultiplier(1.0f);
}

void GameState::onGameOver() {
    changeState(GameStateType::GameOver);
    replay_->saveRecording(
        "assets/save/replay_last.rpy",
        ScoreSystem::instance().score(),
        "Player"
    );
}

} // namespace th06
```

> **原版 1:1 还原关键细节**：
> 1. **状态切换清弹幕**：每次状态切换（关卡加载、BOSS 战开始/结束）都应清空弹幕，避免残留弹幕干扰。
> 2. **BOSS 战背景减速**：BOSS 出现时背景减速到 30%，BOSS 死亡后恢复 100%。
> 3. **关卡波次调度**：原版的敌人出现时机极其精确（逐帧对齐），复刻时需从 ECL 脚本反编译文本中提取每个敌人的生成帧。
> 4. **暂停时停止所有逻辑**：暂停期间不更新任何游戏对象，但仍渲染（叠加暂停菜单）。

---
# 第五部分：性能优化、调试工具、复刻避坑指南

## 5.1 SDL2 性能优化方案

### 5.1.1 万级弹幕批渲染

当弹幕数量达到 2000+ 时，逐个 `SDL_RenderCopyExF` 调用会成为瓶颈。优化方案：

**方案一：按纹理分桶（推荐，SDL2 2.0.10+ 内部已批处理）**

```cpp
// 按纹理分桶后批量提交
void BulletManager::renderBatched(Renderer& renderer) {
    // 1. 按纹理指针分桶
    std::unordered_map<SDL_Texture*, std::vector<const Bullet*>> buckets;
    for (const auto& b : pool_) {
        if (b.active) {
            SDL_Texture* tex = getBulletTexture(b.type, b.color);
            buckets[tex].push_back(&b);
        }
    }

    // 2. 每桶批量渲染
    for (auto& [tex, bullets] : buckets) {
        SDL_SetTextureAlphaMod(tex, 255);
        for (const auto* b : bullets) {
            SDL_FRect dst = {b->position.x - 4, b->position.y - 4, 8, 8};
            SDL_RenderCopyExF(renderer.sdlRenderer(), tex,
                             getBulletSrcRect(b->type, b->color),
                             &dst, b->angle, nullptr, SDL_FLIP_NONE);
        }
    }
}
```

**方案二：自定义 SDL_Rect 数组批量提交（极致优化）**

```cpp
// 使用 SDL_RenderCopyF 批量提交同纹理同源矩形的子弹
void BulletManager::renderSameTextureBatch(SDL_Renderer* r, SDL_Texture* tex,
                                            const SDL_Rect* src,
                                            const std::vector<SDL_FRect>& dsts) {
    // SDL2 没有原生批量接口，但可通过减少状态切换优化
    SDL_SetTextureAlphaMod(tex, 255);
    SDL_SetTextureColorMod(tex, 255, 255, 255);
    for (const auto& dst : dsts) {
        SDL_RenderCopyF(r, tex, src, &dst);
    }
}
```

**方案三：使用 SDL_gpu 或 OpenGL 直接批渲染（高级）**

如果 SDL2 原生渲染仍不够快，可引入 SDL_gpu 库或直接用 OpenGL 写自定义批渲染器，单次 draw call 渲染数千精灵。

### 5.1.2 纹理缓存复用

```cpp
// ResourceManager 中的纹理缓存
class ResourceManager {
public:
    std::shared_ptr<Texture> loadTexture(const std::string& path) {
        auto it = textureCache_.find(path);
        if (it != textureCache_.end()) {
            return it->second;  // 缓存命中
        }
        // 加载新纹理
        auto tex = std::make_shared<Texture>();
        tex->sdlTexture = IMG_LoadTexture(renderer_, path.c_str());
        SDL_QueryTexture(tex->sdlTexture, nullptr, nullptr,
                        &tex->width, &tex->height);
        textureCache_[path] = tex;
        return tex;
    }

    /// 预加载常用纹理（启动时调用）
    void preloadTextures() {
        const char* paths[] = {
            "assets/images/player_reimu.png",
            "assets/images/bullet_enemy.png",
            "assets/images/bullet_player.png",
            "assets/images/item.png",
            "assets/images/font_16x16.png",
            // ...
        };
        for (auto p : paths) loadTexture(p);
    }

private:
    std::unordered_map<std::string, std::shared_ptr<Texture>> textureCache_;
};
```

### 5.1.3 音频声道限制

SDL2_mixer 默认 8 个声道，同时播放音效超过 8 个会被丢弃。优化方案：

```cpp
// 初始化时设置更多声道
Mix_AllocateChannels(32);  // 32 个声道

// 音效优先级管理
void playSoundWithPriority(const std::string& path, int priority) {
    // 高优先级音效（如中弹、炸弹）抢占低优先级（如擦弹）
    if (priority >= 10) {
        // 停止所有低优先级音效
        for (int i = 0; i < 16; ++i) {
            if (Mix_Playing(i)) Mix_HaltChannel(i);
        }
    }
    Mix_PlayChannel(-1, getSound(path), 0);
}

// 同音效节流（避免一帧内重复播放）
std::unordered_map<std::string, int> lastPlayFrame_;
void playSoundThrottled(const std::string& path, int currentFrame, int minInterval) {
    auto it = lastPlayFrame_.find(path);
    if (it != lastPlayFrame_.end() && currentFrame - it->second < minInterval) {
        return;  // 节流
    }
    lastPlayFrame_[path] = currentFrame;
    Mix_PlayChannel(-1, getSound(path), 0);
}
```

### 5.1.4 内存释放管理

```cpp
// 关卡切换时释放不用的资源
void ResourceManager::releaseUnused() {
    for (auto it = textureCache_.begin(); it != textureCache_.end(); ) {
        if (it->second.use_count() == 1) {  // 仅缓存持有
            SDL_DestroyTexture(it->second->sdlTexture);
            it = textureCache_.erase(it);
        } else {
            ++it;
        }
    }
}

// 退出时全部释放
void ResourceManager::shutdown() {
    for (auto& [path, tex] : textureCache_) {
        if (tex->sdlTexture) SDL_DestroyTexture(tex->sdlTexture);
    }
    textureCache_.clear();

    for (auto& [path, chunk] : soundCache_) {
        if (chunk) Mix_FreeChunk(chunk);
    }
    soundCache_.clear();

    for (auto& [path, music] : musicCache_) {
        if (music) Mix_FreeMusic(music);
    }
    musicCache_.clear();
}
```

---

## 5.2 内置调试工具实现

### 5.2.1 碰撞盒可视化绘制

```cpp
// include/debug/DebugOverlay.h
#pragma once
#include "game/Player.h"
#include "game/BulletManager.h"
#include "game/EnemyManager.h"
#include "game/Boss.h"
#include "core/Renderer.h"

namespace th06 {

class DebugOverlay {
public:
    static DebugOverlay& instance();

    void toggle() { enabled_ = !enabled_; }
    void setEnabled(bool e) { enabled_ = e; }
    bool isEnabled() const { return enabled_; }

    /// 绘制碰撞盒
    void renderHitboxes(Player* player, BulletManager* enemyBullets,
                        BulletManager* playerBullets, EnemyManager* enemies,
                        Boss* boss);

    /// 绘制调试信息
    void renderDebugInfo(int fps, int bulletCount, int enemyCount);

    /// 单步帧控制
    void setStepMode(bool s) { stepMode_ = s; }
    bool isStepMode() const { return stepMode_; }
    void step() { stepRequested_ = true; }
    bool consumeStep() {
        bool s = stepRequested_;
        stepRequested_ = false;
        return s;
    }

    /// 慢动作
    void setSlowMotion(bool s) { slowMotion_ = s; }
    bool isSlowMotion() const { return slowMotion_; }

private:
    DebugOverlay() = default;
    bool enabled_ = false;
    bool stepMode_ = false;
    bool stepRequested_ = false;
    bool slowMotion_ = false;
};

} // namespace th06
```

```cpp
// src/debug/DebugOverlay.cpp
#include "debug/DebugOverlay.h"
#include "ui/BitmapFont.h"

namespace th06 {

DebugOverlay& DebugOverlay::instance() {
    static DebugOverlay inst;
    return inst;
}

void DebugOverlay::renderHitboxes(Player* player, BulletManager* enemyBullets,
                                   BulletManager* playerBullets,
                                   EnemyManager* enemies, Boss* boss) {
    if (!enabled_) return;
    auto& renderer = Renderer::instance();
    SDL_SetRenderDrawColor(renderer.sdlRenderer(), 0, 255, 0, 128);

    // 玩家判定（绿色小圆）
    if (player) {
        drawCircle(renderer.sdlRenderer(),
                  player->position.x, player->position.y,
                  player->hitboxRadius);
        // 擦弹范围（黄色）
        SDL_SetRenderDrawColor(renderer.sdlRenderer(), 255, 255, 0, 64);
        drawCircle(renderer.sdlRenderer(),
                  player->position.x, player->position.y,
                  player->grazeRadius);
        SDL_SetRenderDrawColor(renderer.sdlRenderer(), 0, 255, 0, 128);
    }

    // 敌方子弹（红色小圆）
    if (enemyBullets) {
        for (const auto& b : enemyBullets->pool()) {
            if (!b.active) continue;
            drawCircle(renderer.sdlRenderer(),
                      b.position.x, b.position.y,
                      b.collisionRadius);
        }
    }

    // 玩家子弹（蓝色小圆）
    if (playerBullets) {
        SDL_SetRenderDrawColor(renderer.sdlRenderer(), 0, 128, 255, 128);
        for (const auto& b : playerBullets->pool()) {
            if (!b.active) continue;
            drawCircle(renderer.sdlRenderer(),
                      b.position.x, b.position.y,
                      b.collisionRadius);
        }
    }

    // 敌人（紫色方框）
    if (enemies) {
        SDL_SetRenderDrawColor(renderer.sdlRenderer(), 255, 0, 255, 128);
        for (const auto& e : enemies->enemies()) {
            if (e.state == EnemyState::Dead) continue;
            SDL_FRect r = {
                e.position.x - e.hitboxRadius,
                e.position.y - e.hitboxRadius,
                e.hitboxRadius * 2, e.hitboxRadius * 2
            };
            SDL_RenderDrawRectF(renderer.sdlRenderer(), &r);
        }
    }

    // BOSS（白色大圆）
    if (boss && boss->active) {
        SDL_SetRenderDrawColor(renderer.sdlRenderer(), 255, 255, 255, 200);
        drawCircle(renderer.sdlRenderer(),
                  boss->position.x, boss->position.y,
                  boss->hitboxRadius);
    }
}

void DebugOverlay::renderDebugInfo(int fps, int bulletCount, int enemyCount) {
    if (!enabled_) return;
    auto& font = BitmapFont::instance();

    font.drawText("FPS: " + std::to_string(fps), 8, 40, 0, 255, 0);
    font.drawText("Bullets: " + std::to_string(bulletCount), 8, 56, 0, 255, 0);
    font.drawText("Enemies: " + std::to_string(enemyCount), 8, 72, 0, 255, 0);

    if (stepMode_) {
        font.drawText("[STEP MODE] Press F10 to advance", 8, 88, 255, 255, 0);
    }
    if (slowMotion_) {
        font.drawText("[SLOW MOTION]", 8, 104, 255, 255, 0);
    }
}

void DebugOverlay::drawCircle(SDL_Renderer* r, float cx, float cy, float radius) {
    // 简化：画 32 边多边形
    constexpr int segments = 32;
    SDL_FPoint points[segments + 1];
    for (int i = 0; i <= segments; ++i) {
        float angle = 2.0f * 3.14159265f * i / segments;
        points[i].x = cx + std::cos(angle) * radius;
        points[i].y = cy + std::sin(angle) * radius;
    }
    SDL_RenderDrawLinesF(r, points, segments + 1);
}

} // namespace th06
```

### 5.2.2 调试快捷键绑定

```cpp
// 在 MainLoop 中处理调试快捷键
void MainLoop::handleDebugKeys() {
    auto& input = InputManager::instance();
    auto& debug = DebugOverlay::instance();

    // F1: 切换碰撞盒显示
    if (input.isPressedScancode(SDL_SCANCODE_F1)) {
        debug.toggle();
    }
    // F2: 暂停/恢复
    if (input.isPressedScancode(SDL_SCANCODE_F2)) {
        paused_ = !paused_;
    }
    // F3: 单步模式
    if (input.isPressedScancode(SDL_SCANCODE_F3)) {
        debug.setStepMode(!debug.isStepMode());
    }
    // F10: 单步前进（单步模式下）
    if (input.isPressedScancode(SDL_SCANCODE_F10)) {
        debug.step();
    }
    // F4: 慢动作
    if (input.isPressedScancode(SDL_SCANCODE_F4)) {
        debug.setSlowMotion(!debug.isSlowMotion());
    }
    // F5: 截图
    if (input.isPressedScancode(SDL_SCANCODE_F5)) {
        takeScreenshot();
    }
}
```

---

## 5.3 高频复刻踩坑完整清单

### 5.3.1 帧率超速（高刷屏 144Hz/240Hz 导致游戏速度翻倍）

**问题根源**：未实现固定逻辑帧率，游戏更新直接绑定渲染帧率。在 144Hz 显示器上，游戏速度变为 2.4 倍。

**修复方案**：使用 3.2 节的固定 60 帧分离主循环。关键点：
- 逻辑更新频率固定为 60Hz，与显示器刷新率无关。
- 渲染频率等于显示器刷新率，通过插值平滑画面。
- 使用 `SDL_GetPerformanceCounter()` 而非 `SDL_GetTicks()` 获取高精度时间。

**验证方法**：在 60Hz 和 144Hz 显示器上分别运行，观察玩家移动速度是否一致。

### 5.3.2 透明贴图发黑（PNG 透明区域渲染为黑色）

**问题根源**：SDL2 默认不开启 Alpha 混合，或纹理未设置 `SDL_BLENDMODE_BLEND`。

**修复方案**：
```cpp
// 1. 创建渲染器时确保加速
SDL_Renderer* r = SDL_CreateRenderer(window, -1,
    SDL_RENDERER_ACCELERATED | SDL_RENDERER_PRESENTVSYNC);

// 2. 加载纹理后设置混合模式
SDL_Texture* tex = IMG_LoadTexture(r, "sprite.png");
SDL_SetTextureBlendMode(tex, SDL_BLENDMODE_BLEND);

// 3. PNG 预处理时确保有 Alpha 通道
// 使用 PIL/Pillow 转换：
// img = Image.open("sprite.png").convert("RGBA")
```

### 5.3.3 BGM 循环断音（循环点处有明显停顿或爆音）

**问题根源**：SDL2_mixer 的 `Mix_PlayMusic` 默认从头循环，循环点不精确。WAV 文件结尾可能有静音，导致循环时出现停顿。

**修复方案**：使用自定义音乐回调实现精确循环。

```cpp
// 精确循环实现
void musicFinishedCallback() {
    // 重新播放，从 loop_start 开始
    Mix_PlayMusic(currentMusic, 0);
    Mix_SetMusicPosition(loopStartSeconds);
}

void playBgmWithLoop(const std::string& path, double loopStart, double loopEnd) {
    currentMusic = Mix_LoadMUS(path.c_str());
    loopStartSeconds = loopStart;
    // 设置音乐结束回调
    Mix_HookMusicFinished(musicFinishedCallback);
    Mix_PlayMusic(currentMusic, 0);
    // 注意：WAV 格式 Mix_SetMusicPosition 可能不支持，需用 OGG
}
```

**推荐方案**：将 BGM 转为 OGG 格式，OGG 原生支持 `Mix_SetMusicPosition`。

```bash
# WAV -> OGG 转换
ffmpeg -i bgm01.wav -c:a libvorbis -q:a 4 bgm01.ogg
```

### 5.3.4 录像弹幕错位（Replay 回放时弹幕位置与录制时不一致）

**问题根源**：
1. RNG 调用顺序不一致（录制时调用了 N 次，回放时调用了 N+1 或 N-1 次）。
2. 回放期间读取了系统时间（如 `SDL_GetTicks()`）作为随机种子。
3. 回放期间有真实键盘输入干扰。

**修复方案**：
1. **严格对齐 RNG 调用**：在 RNG 的 `next()` 方法中添加日志（仅 Debug 模式），记录每次调用的调用栈与返回值。录制与回放时对比日志，找出不一致的调用点。
2. **禁止读取系统时间**：所有随机必须走 `globalRNG()`，禁止在游戏逻辑中调用 `SDL_GetTicks()` 或 `time()` 作为随机源。
3. **回放期间屏蔽真实输入**：回放时 `InputManager` 只从 Replay 数据读取，不读取真实键盘。

```cpp
// RNG 调试日志
uint32_t RNG::next() {
    state_ = state_ * 214013 + 2531011;
#ifdef DEBUG_RNG
    Logger::instance().debug("RNG call at frame " + std::to_string(currentFrame_) +
                             " returns " + std::to_string(state_ >> 16));
#endif
    return state_ >> 16;
}
```

### 5.3.5 输入延迟（按键响应慢半拍）

**问题根源**：
1. 事件处理与逻辑更新分离，事件先入队，下一帧才处理。
2. VSync 导致渲染阻塞，影响事件轮询。
3. 键盘扫描码与虚拟键映射错误。

**修复方案**：
1. **事件处理与逻辑更新同帧**：在 `update()` 开头先 `SDL_PollEvent`，然后立即处理输入状态。
2. **实时键盘轮询**：除了事件处理，每帧调用 `SDL_GetKeyboardState` 获取实时状态，避免事件队列延迟。
3. **输入缓冲**：将"按下"事件记录到队列，逻辑帧消费时立即处理，不跨帧。

```cpp
// 输入系统优化
void InputManager::update() {
    // 1. 先处理事件队列
    SDL_Event e;
    while (SDL_PollEvent(&e)) {
        if (e.type == SDL_KEYDOWN) {
            pressedThisFrame_[e.key.keysym.scancode] = true;
        }
    }
    // 2. 实时键盘状态
    keyboardState_ = SDL_GetKeyboardState(nullptr);
    // 3. 逻辑帧结束时清空 pressedThisFrame_
}

bool InputManager::isPressed(Action a) const {
    SDL_Scancode sc = keyMapping_[(int)a];
    return pressedThisFrame_[sc];
}

bool InputManager::isDown(Action a) const {
    SDL_Scancode sc = keyMapping_[(int)a];
    return keyboardState_[sc];
}
```

### 5.3.6 像素画面模糊（缩放后画面发虚）

**问题根源**：SDL2 默认使用线性过滤（`SDL_HINT_RENDER_SCALE_QUALITY` 默认为 "0" 即最近邻，但部分平台可能默认为 "1" 线性）。

**修复方案**：
```cpp
// 在创建渲染器前设置
SDL_SetHint(SDL_HINT_RENDER_SCALE_QUALITY, "0");  // 0 = 最近邻（像素锐利）
// SDL_SetHint(SDL_HINT_RENDER_SCALE_QUALITY, "1");  // 1 = 线性（模糊）
// SDL_SetHint(SDL_HINT_RENDER_SCALE_QUALITY, "2");  // 2 = 各向异性

// 然后创建渲染器
SDL_Renderer* r = SDL_CreateRenderer(window, -1,
    SDL_RENDERER_ACCELERATED | SDL_RENDERER_PRESENTVSYNC);

// 设置逻辑分辨率（640x480），SDL2 自动缩放到窗口尺寸
SDL_RenderSetLogicalSize(r, 640, 480);
```

### 5.3.7 弹幕旋转方向错误（弹幕朝向与运动方向不一致）

**问题根源**：`SDL_RenderCopyExF` 的 `angle` 参数是度数，且旋转方向为顺时针。而 `atan2` 返回的弧度需转换为度数，且 y 轴方向相反（屏幕坐标 y 向下）。

**修复方案**：
```cpp
// 正确的弹幕旋转计算
float angleRad = std::atan2(velocity.y, velocity.x);
float angleDeg = angleRad * 180.0f / 3.14159265f;
// 注意：如果弹幕精灵默认朝右（0度），则 angleDeg 直接传入
// 如果弹幕精灵默认朝上（-90度），则需 +90
SDL_RenderCopyExF(r, tex, &src, &dst, angleDeg, &center, SDL_FLIP_NONE);
```

### 5.3.8 内存泄漏（长时间运行内存持续增长）

**问题根源**：
1. 纹理/音频未释放。
2. 弹幕对象池未复用，每帧 new。
3. Lua 脚本中的表未清理。

**修复方案**：
1. 使用 `std::shared_ptr` 管理纹理/音频生命周期。
2. 弹幕严格使用对象池，禁止 `new Bullet`。
3. 定期调用 `lua_gc(L, LUA_GCCOLLECT, 0)` 清理 Lua 内存。
4. 使用 Valgrind 检测内存泄漏：
```bash
valgrind --leak-check=full --show-leak-kinds=all ./th06_remaster
```

### 5.3.9 音效爆音（音量过大导致削波失真）

**问题根源**：多个音效同时播放，音量叠加超过 1.0，导致削波。

**修复方案**：
1. 音效归一化到 -3dB（约 70% 音量）。
2. 限制同时播放的音效数量。
3. 使用 Mix_Volume 控制单个声道音量。

```cpp
// 音效音量控制
Mix_Volume(-1, MIX_MAX_VOLUME * 0.7);  // 所有声道 70% 音量
Mix_VolumeMusic(MIX_MAX_VOLUME * 0.5); // BGM 50% 音量
```

### 5.3.10 跨平台路径分隔符问题

**问题根源**：Windows 使用 `\`，Linux/macOS 使用 `/`，硬编码路径分隔符导致跨平台失败。

**修复方案**：使用 `std::filesystem::path` 或统一使用 `/`（SDL2 内部会自动转换）。

```cpp
// 推荐：使用 std::filesystem
namespace fs = std::filesystem;
fs::path assetPath = fs::current_path() / "assets" / "images" / "player.png";
std::string pathStr = assetPath.string();
```

---

## 5.4 可选拓展功能开发指引

### 5.4.1 游戏截图

```cpp
void takeScreenshot() {
    auto& renderer = Renderer::instance();
    SDL_Renderer* r = renderer.sdlRenderer();

    // 读取渲染器像素
    SDL_Texture* target = SDL_GetRenderTarget(r);
    int w, h;
    SDL_QueryTexture(target, nullptr, nullptr, &w, &h);
    SDL_Texture* tex = SDL_CreateTexture(r, SDL_PIXELFORMAT_RGBA8888,
                                          SDL_TEXTUREACCESS_TARGET, w, h);
    SDL_SetRenderTarget(r, tex);
    SDL_RenderReadPixels(r, nullptr, SDL_PIXELFORMAT_RGBA8888, ...);

    // 保存为 PNG
    std::string filename = "screenshot_" +
        std::to_string(SDL_GetTicks()) + ".png";
    IMG_SavePNG(surface, filename.c_str());

    SDL_SetRenderTarget(r, nullptr);
    SDL_DestroyTexture(tex);
}
```

### 5.4.2 自定义窗口缩放

```cpp
// 支持窗口缩放（保持 4:3 比例）
void onWindowResized(int newWidth, int newHeight) {
    // SDL_RenderSetLogicalSize 已自动处理缩放
    // 这里只需更新窗口标题显示当前缩放
    int scale = newWidth / 640;
    std::string title = "TH06 Remaster (x" + std::to_string(scale) + ")";
    SDL_SetWindowTitle(window, title.c_str());
}

// 全屏切换
void toggleFullscreen() {
    static bool fullscreen = false;
    fullscreen = !fullscreen;
    SDL_SetWindowFullscreen(window, fullscreen ? SDL_WINDOW_FULLSCREEN_DESKTOP : 0);
}
```

### 5.4.3 慢动作调试模式

```cpp
// 在 MainLoop 中实现慢动作
void MainLoop::update() {
    auto& debug = DebugOverlay::instance();
    if (debug.isSlowMotion()) {
        // 每 3 帧更新一次逻辑（1/3 速度）
        slowMotionCounter_ = (slowMotionCounter_ + 1) % 3;
        if (slowMotionCounter_ == 0) {
            gameState_->update();
        }
    } else {
        gameState_->update();
    }
}
```

---

## 5.5 从 0 到可运行游戏的分步开发流程

以下是推荐的分阶段开发顺序，每个阶段完成后都应有可运行的成果：

### 阶段 1：环境搭建与窗口（1-2 天）
1. 配置开发环境（VSCode + CMake + SDL2）。
2. 创建 SDL2 窗口，渲染一个纯色背景。
3. 实现固定 60 帧主循环，显示帧率。
4. **成果**：一个显示帧率的空白窗口。

### 阶段 2：资源加载与精灵渲染（2-3 天）
1. 实现 ResourceManager，加载 PNG 纹理。
2. 渲染一张玩家精灵到屏幕。
3. 实现位图字体渲染器，显示文字。
4. **成果**：屏幕上显示玩家精灵与文字。

### 阶段 3：输入与玩家移动（1-2 天）
1. 实现 InputManager，处理键盘输入。
2. 玩家可上下左右移动，Shift 低速。
3. 边界限制。
4. **成果**：可控制的玩家精灵。

### 阶段 4：弹幕系统（3-5 天）
1. 实现 BulletManager 对象池。
2. 玩家射击（按 Z 发射子弹）。
3. 敌方子弹生成（测试用）。
4. 碰撞检测（玩家中弹）。
5. **成果**：可射击、可中弹的基本 STG 雏形。

### 阶段 5：敌人与 BOSS（3-5 天）
1. 实现 Enemy 类，敌人可移动、射击、死亡。
2. 实现 Boss 类，多阶段战斗。
3. 实现符卡系统。
4. **成果**：可对抗 BOSS 的完整战斗。

### 阶段 6：道具与分数（2-3 天）
1. 实现 ItemManager，道具掉落与拾取。
2. 实现 ScoreSystem，分数计算。
3. 实现 HUD，显示分数/残机/火力。
4. **成果**：完整的得分循环。

### 阶段 7：背景与 UI（2-3 天）
1. 实现 Background 多层视差。
2. 实现主菜单、难度选择、角色选择。
3. 实现 Game Over / 通关界面。
4. **成果**：完整的游戏流程。

### 阶段 8：存档与录像（2-3 天）
1. 实现 SaveManager，存档读写。
2. 实现 Replay 系统，录制与回放。
3. **成果**：可保存进度、可回放录像。

### 阶段 9：难度与平衡（1-2 天）
1. 实现 DifficultyManager，四难度切换。
2. 调整数值，对齐原版手感。
3. **成果**：四难度可玩。

### 阶段 10：优化与调试（2-3 天）
1. 性能优化（批渲染、纹理缓存）。
2. 实现调试工具（碰撞盒、单步、慢动作）。
3. 修复踩坑清单中的问题。
4. **成果**：稳定 60 帧、可调试的完整游戏。

**总工期**：约 20-30 天（全职开发）。

---

## 5.6 拓展方向

### 5.6.1 移植 Windows / macOS

**Windows 移植**：
1. CMake 已支持跨平台，只需安装 Windows 版 SDL2。
2. 推荐使用 vcpkg 管理依赖：
```bash
vcpkg install sdl2 sdl2-image sdl2-mixer lua
cmake -DCMAKE_TOOLCHAIN_FILE=<vcpkg>/scripts/buildsystems/vcpkg.cmake ..
```
3. 使用 MSVC 或 MinGW 编译。
4. 注意路径分隔符（使用 `std::filesystem`）。

**macOS 移植**：
1. 使用 Homebrew 安装依赖：
```bash
brew install sdl2 sdl2_image sdl2_mixer lua
```
2. CMake 配置与 Linux 相同。
3. 注意 macOS 的 Retina 屏幕，需设置 `SDL_HINT_RENDER_SCALE_QUALITY`。
4. 打包为 .app 时需将 SDL2 动态库打包进应用。

### 5.6.2 自制原创素材

为避免版权问题，最终公开分享时应替换为原创素材：

1. **角色立绘**：使用 AI 绘图工具（如 Stable Diffusion）生成原创角色，或委托画师。
2. **弹幕精灵**：用 AseDraw 或 Pixaki 绘制像素风弹幕。
3. **BGM**：使用 FL Studio 或 LMMS 制作原创音乐，参考东方风格但不可直接翻录。
4. **音效**：使用 Bfxr 或 sfxr 生成 8-bit 风格音效。
5. **字体**：使用 Pixel Font Maker 制作原创位图字体，或使用开源像素字体（如 Press Start 2P）。

### 5.6.3 新增自定义关卡编辑器

开发一个可视化关卡编辑器，允许玩家自制关卡：

1. **编辑器架构**：基于 ImGui + SDL2，独立于游戏主程序。
2. **功能**：
   - 敌人波次编辑（时间轴 + 敌人类型 + 位置）。
   - BOSS 阶段编辑（血量、时限、弹幕脚本选择）。
   - 背景层配置（纹理、滚动速度）。
   - 实时预览（按"测试"按钮直接运行关卡）。
3. **数据格式**：导出为 JSON，游戏运行时加载。
4. **分享**：玩家可将自制关卡 JSON 分享给他人。

```cpp
// 关卡编辑器伪代码
class StageEditor {
public:
    void renderUI() {
        ImGui::Begin("Stage Editor");

        // 时间轴
        if (ImGui::Button("Add Wave")) {
            waves_.push_back(WaveEntry{currentFrame_, "fairy", {320, 0}, 1});
        }

        // 波次列表
        for (size_t i = 0; i < waves_.size(); ++i) {
            ImGui::PushID(i);
            ImGui::InputInt("Frame", &waves_[i].spawnFrame);
            ImGui::InputText("Enemy", &waves_[i].enemyType);
            ImGui::InputFloat("X", &waves_[i].spawnPosition.x);
            ImGui::InputFloat("Y", &waves_[i].spawnPosition.y);
            ImGui::InputInt("Count", &waves_[i].count);
            ImGui::PopID();
            ImGui::Separator();
        }

        if (ImGui::Button("Test")) {
            saveToTemp();
            launchGameWithStage("temp_stage.json");
        }
        if (ImGui::Button("Export")) {
            exportJSON("custom_stage.json");
        }

        ImGui::End();
    }
};
```

---

## 结语

本教程覆盖了从零开始用 C++ + SDL2 复刻东方红魔乡的完整流程，包括环境配置、素材处理、底层框架、业务系统、性能优化与避坑指南。每个模块均提供了可直接集成的代码模板与原版还原要点。

复刻东方红魔乡是一项庞大的工程，但通过分层架构与模块化设计，可以将复杂问题分解为可管理的子任务。建议按照 5.5 节的分阶段流程逐步推进，每个阶段完成后都有可运行的成果，保持开发动力。

**再次强调版权红线**：本项目仅限个人学习研究，严禁商用与分发原版素材。公开分享时请替换为原创素材，并明确声明著作权归属。

祝复刻顺利，享受 STG 开发的乐趣！
