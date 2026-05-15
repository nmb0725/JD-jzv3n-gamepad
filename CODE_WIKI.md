# JD JZ-V3N Gamepad to Xbox 360 Controller Mapper — Code Wiki

## 1. 项目概述

| 元数据 | 内容 |
|---|---|
| **项目名称** | JD JZ-V3N Controller Interface Application |
| **程序文件名** | `jzv3n.exe` |
| **版本** | 1.1.0.0 |
| **编程语言** | C# |
| **目标框架** | .NET Framework 4.5.2 |
| **输出类型** | Windows 应用程序 (WinExe) |
| **目标平台** | Windows (需要管理员权限) |
| **核心功能** | 将京东 JZ-V3N 游戏手柄映射为 Xbox 360 手柄，支持震动反馈 |

### 项目目标
该项目基于原小米游戏手柄映射方案进行适配修改，将京东 JZ-V3N 游戏手柄的原始 HID 输入数据转换为 Xbox 360 控制器标准输入报告，并通过 SCP Virtual Bus Driver 让 Windows 系统将 JZ-V3N 手柄识别为 Xbox 360 手柄，从而在支持 Xbox 手柄的游戏中正常使用。

---

## 2. 项目目录结构

```
Xiaomi_gamepad14/
├── readme.txt                          # 根项目说明（硬件与协议分析）
├── accelerometer_print/                # Linux 加速度计读取工具
│   ├── readme.txt                      # 编译与运行说明
│   └── hidraw.c                        # C 语言实现，通过 hidraw 读取加速度计数据
├── bin/Debug/test/                     # 测试构建输出
├── .vscode/                            # VS Code 配置
│   ├── launch.json
│   ├── settings.json
│   └── tasks.json
└── mi/                                 # 主项目目录（C# Windows 应用）
    ├── mi.sln                          # 解决方案文件
    ├── mi.csproj                       # 项目文件
    ├── Program.cs                      # 程序入口 + 控制器管理
    ├── Xiaomi_gamepad.cs               # 核心：手柄输入读取 + 震动控制
    ├── ProcessIcon.cs                  # 系统托盘图标
    ├── ContextMenus.cs                 # 系统托盘右键菜单
    ├── AboutBox.cs                     # "关于" 对话框
    ├── AboutBox.Designer.cs            # 对话框设计器文件
    ├── App.config                      # 应用配置 (.NET 运行时版本)
    ├── app.manifest                    # 应用程序清单（声明管理员权限）
    ├── LICENSE.txt                     # 许可文件
    ├── readme.txt                      # 项目说明
    ├── Properties/
    │   ├── AssemblyInfo.cs             # 程序集元数据
    │   ├── Resources.resx              # 资源文件
    │   └── Resources.Designer.cs       # 资源访问代码
    ├── Resources/
    │   └── micontroller.ico            # 托盘图标
    ├── Images/
    │   └── micontroller.png            # 手柄图片
    ├── HidLibrary/                     # HID 设备通信库（第三方，经修改）
    │   ├── HidDevice.cs                # HID 设备核心操作类
    │   ├── HidDevices.cs               # HID 设备枚举与发现
    │   ├── IHidDevice.cs               # HID 设备接口
    │   ├── HidFastReadDevice.cs        # 快速读取模式（跳过连接检查）
    │   ├── HidDeviceData.cs            # 读取数据封装
    │   ├── HidDeviceAttributes.cs      # 设备属性（VID/PID/版本）
    │   ├── HidDeviceCapabilities.cs    # 设备能力描述
    │   ├── HidDeviceEventMonitor.cs    # 设备插拔事件监控
    │   ├── HidReport.cs                # HID 报告封装
    │   ├── HidEnumerator.cs            # HID 枚举器（实例封装）
    │   ├── IHidEnumerator.cs           # 枚举器接口
    │   ├── HidAsyncState.cs            # 异步操作状态封装
    │   ├── NativeMethods.cs            # Win32 P/Invoke 声明
    │   ├── DeviceMode.cs               # 设备模式枚举（同步/重叠I/O）
    │   ├── ShareMode.cs                # 共享模式枚举
    │   ├── Extensions.cs               # 字节数组扩展方法
    │   ├── InsertedEventHandler.cs     # 设备插入委托
    │   ├── RemovedEventHandler.cs      # 设备移除委托
    │   ├── ReadCallback.cs             # 读取回调委托
    │   ├── ReadReportCallback.cs       # 读取报告回调委托
    │   └── WriteCallback.cs            # 写入回调委托
    └── ScpDriverInterface/             # SCP 虚拟总线驱动接口（第三方）
        ├── ScpBus.cs                   # SCP 总线操作类
        ├── X360Controller.cs           # 虚拟 Xbox 360 控制器状态
        ├── X360Buttons.cs              # Xbox 360 按钮枚举（Flags）
        └── NativeMethods.cs            # Win32 P/Invoke 声明
```

---

## 3. 项目架构与数据流

### 3.1 整体架构

```
┌─────────────────────────────────────────────────────────────────────┐
│                     jzv3n.exe (Windows 应用)                        │
│                                                                     │
│  ┌─────────────┐   ┌──────────────────┐   ┌──────────────────────┐ │
│  │  Program.cs  │──▶│ Xiaomi_gamepad   │──▶│  ScpDriverInterface  │ │
│  │  (入口/管理) │   │  (手柄处理核心)   │   │  (Xbox 360 模拟)     │ │
│  └──────┬──────┘   └────────┬─────────┘   └──────────┬───────────┘ │
│         │                   │                         │              │
│         │                   ▼                         ▼              │
│         │          ┌──────────────────┐   ┌──────────────────────┐ │
│         │          │   HidLibrary     │   │  SCP Virtual Bus     │ │
│         │          │  (HID 通信层)    │   │  Driver (系统驱动)    │ │
│         │          └────────┬─────────┘   └──────────┬───────────┘ │
│         │                   │                         │              │
│         ▼                   ▼                         ▼              │
│  ┌─────────────┐   ┌──────────────────┐                              │
│  │ ProcessIcon │   │ JD JZ-V3N 手柄    │                              │
│  │ (系统托盘)   │   │  (USB HID 设备)   │                              │
│  └─────────────┘   └──────────────────┘                              │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 核心数据流

```
JD JZ-V3N 手柄 (USB HID)
    │
    ▼ 原始 HID 输入报告 (21 字节)
HidLibrary (Read)
    │
    ▼ HidDeviceData
Xiaomi_gamepad.input_thread()
    │
    ├── 解析按键状态 → X360Buttons 位标志
    ├── 解析摇杆/扳机值 → 数值映射/转换
    │
    ▼ X360Controller 对象
ScpBus.Report() / DeviceIoControl
    │
    ▼ 20 字节 Xbox 360 输入报告
SCP Virtual Bus Driver
    │
    ▼
Windows → 识别为 Xbox 360 手柄
    │
    ▼ (输出报告含震动数据)
ScpBus.Report() 返回 outputReport
    │
    ▼
Xiaomi_gamepad.rumble_thread() → HidDevice.Write() → 手柄震动
```

---

## 4. 主要模块详解

### 4.1 程序入口 — [Program.cs](file:///l:/git/Xiaomi_gamepad14/mi/Program.cs)

**职责**：应用程序入口点，控制器生命周期管理，系统托盘初始化

| 方法/成员 | 说明 |
|---|---|
| `Main()` | 程序入口。检查单实例运行 → 创建系统托盘 → 启动控制器管理线程 → 运行消息循环 |
| `ManageControllers(ScpBus)` | 后台线程，每 1 秒轮询枚举 HID 设备。发现新设备时尝试以独占模式打开，失败则尝试重新启用设备或使用共享模式 |
| `IsSingleInstance()` | 通过命名 Mutex 确保程序只运行一个实例 |
| `TryReEnableDevice(string)` | 通过 SetupAPI 禁用再启用 HID 设备，解决驱动占用问题 |
| `devicePathToInstanceId(string)` | 将设备路径转换为设备实例 ID |
| `ConsoleEventCallback(int)` | 控制台关闭事件回调，用于清理 SCP 总线连接 |
| `InformUser(string)` | 通过系统托盘气球提示通知用户 |
| `Gamepads` | 静态列表，保存所有已连接的手柄实例 |
| `NIcon` | 静态属性，系统托盘图标对象 |

**关键行为**：
- 使用 `HidDevices.Enumerate(0x20BC, 0x505E)` 枚举指定 VID/PID 的 HID 设备
- 通过 `devicePathToInstanceId` 提取设备实例 ID，配合 `SetupDiGetClassDevs` 等 API 实现设备重新启用
- 只选择路径中包含 `&col03#` 的设备（手柄的第三个 HID 集合，即游戏控制器接口）

### 4.2 核心手柄处理 — [Xiaomi_gamepad.cs](file:///l:/git/Xiaomi_gamepad14/mi/Xiaomi_gamepad.cs)

**职责**：单个手柄实例的输入读取、按键映射、震动控制

| 成员 | 说明 |
|---|---|
| `Device` | HID 设备对象 |
| `Index` | 手柄编号（1-based，用于 SCP 总线） |
| `rThread` | 震动线程 |
| `iThread` | 输入读取线程（最高优先级） |
| `ScpBus` | SCP 总线实例引用 |
| `Vibration` | 震动状态缓存（5 字节数组） |

| 方法 | 说明 |
|---|---|
| `Xiaomi_gamepad(HidDevice, ScpBus, int)` | 构造函数。发送初始震动数据，启动震动线程和输入线程 |
| `check_connected()` | 通过写入震动数据检测设备是否仍连接 |
| `unplug()` | 停止线程 → 从 SCP 总线拔出 → 关闭设备 |
| `input_thread(HidDevice, ScpBus, int)` | **核心方法**。循环读取 HID 输入报告，解析数据并映射到 Xbox 360 控制器状态 |
| `rumble_thread(HidDevice)` | 震动控制线程。监听震动状态变化，通过 HID 写入震动数据到手柄 |
| `convert_number(int)` | 将有符号字节值转换为无符号中心偏移值 |

**输入报告解析**（JD JZ-V3N 手柄协议）：

原始输入数据格式（`currentState`）：
```
[0]    = 报告 ID (0x03)
[2]    = 十字键方向值
[3]    = 按钮组 1 (位掩码: A/B/X/Y/LB/RB)
[4]    = 按钮组 2 (位掩码: LS/RS/Start/Back)
[5]    = 左摇杆 X 轴
[6]    = 左摇杆 Y 轴
[7]    = 右摇杆 X 轴
[8]    = 右摇杆 Y 轴
[9]    = 左扳机
[10]   = 右扳机
```

**按键映射表**（JZ-V3N 手柄 → Xbox 360）：

| JZ-V3N 手柄按键 | currentState 位检测 | X360Buttons 标志 |
|---|---|---|
| A | `[3] & 1` | `A` |
| B | `[3] & 2` | `B` |
| X | `[3] & 8` | `X` |
| Y | `[3] & 16` | `Y` |
| L1 (左肩键) | `[3] & 64` | `LeftBumper` |
| R1 (右肩键) | `[3] & 128` | `RightBumper` |
| 左摇杆按下 | `[4] & 32` | `LeftStick` |
| 右摇杆按下 | `[4] & 64` | `RightStick` |
| Start | `[4] & 8` | `Start` |
| Back | `[4] & 4` | `Back` |

**十字键映射**（`currentState[2]` 值 → 方向）：

| 原始值 | 映射方向 |
|---|---|
| 0, 1, 7 | Up |
| 3, 4, 5 | Down |
| 5, 6, 7 | Left |
| 1, 2, 3 | Right |
| 15 | 无方向（松开） |

**摇杆值转换**：

使用 `convert_number()` 方法将有符号字节值转换为 Xbox 360 标准的 16 位有符号值（-32768 到 32767）：
```
LeftStickX = (convert_number(raw[5]) - 128) / 127 * 32767
LeftStickY = (convert_number(raw[6]) - 128) / 127 * -32767  (Y 轴取反)
RightStickX = (convert_number(raw[7]) - 128) / 127 * 32767
RightStickY = (convert_number(raw[8]) - 128) / 127 * -32767  (Y 轴取反)
```

**震动反馈**：
- 震动数据通过 SCP 总线的输出报告获取
- `outputReport[1] == 0x08` 表示震动报告
- `outputReport[3]` = 大电机强度，`outputReport[4]` = 小电机强度
- 通过 `rumble_thread` 线程异步写入 HID 设备

### 4.3 系统托盘 — [ProcessIcon.cs](file:///l:/git/Xiaomi_gamepad14/mi/ProcessIcon.cs)

**职责**：管理系统托盘图标

| 方法 | 说明 |
|---|---|
| `Display()` | 设置托盘图标（手柄图标）、文本、绑定右键菜单 |
| `Dispose()` | 释放托盘图标资源 |

### 4.4 右键菜单 — [ContextMenus.cs](file:///l:/git/Xiaomi_gamepad14/mi/ContextMenus.cs)

**职责**：系统托盘右键菜单

| 菜单项 | 功能 |
|---|---|
| About | 显示关于对话框 |
| Exit | 退出应用程序 |

### 4.5 关于对话框 — [AboutBox.cs](file:///l:/git/Xiaomi_gamepad14/mi/AboutBox.cs)

**职责**：显示应用程序版本、版权等信息的标准 About 对话框，使用 TableLayoutPanel 布局。

---

## 5. 第三方依赖库

### 5.1 HidLibrary（[mi/HidLibrary/](file:///l:/git/Xiaomi_gamepad14/mi/HidLibrary/)）

**来源**：https://github.com/mikeobrien/HidLibrary（经修改）

**职责**：封装 Windows HID API，提供 HID 设备的枚举、打开、读写功能。

**核心类体系**：

| 类 | 说明 |
|---|---|
| `HidDevices` | 静态工具类。通过 SetupAPI 枚举系统中所有 HID 设备，支持按 VID/PID 筛选 |
| `HidDevice` | HID 设备核心类。封装设备的打开/关闭/读/写/Feature 报告操作 |
| `HidFastReadDevice` | 继承 `HidDevice`，提供快速读取模式（假设设备始终连接） |
| `HidDeviceData` | 读取结果封装，包含原始字节数据和读取状态 |
| `HidReport` | HID 报告封装，包含报告 ID 和报告数据 |
| `HidDeviceAttributes` | 设备属性（VendorID, ProductID, Version） |
| `HidDeviceCapabilities` | 设备能力（报告长度、按钮/值能力等） |
| `HidDeviceEventMonitor` | 设备插拔事件监控，通过轮询 `IsConnected` 实现 |
| `HidEnumerator` | 实现 `IHidEnumerator` 接口，提供实例化枚举功能 |
| `HidAsyncState` | 异步操作状态管理 |

**枚举设备流程**：
1. `HidD_GetHidGuid()` 获取 HID 类 GUID
2. `SetupDiGetClassDevs()` 获取设备信息集
3. `SetupDiEnumDeviceInterfaces()` 遍历设备接口
4. 为每个接口构造 `HidDevice` 对象

**读写模式**：
- 支持 `NonOverlapped`（同步）和 `Overlapped`（异步重叠 I/O）两种模式
- 重叠模式下使用事件（Event）等待读写完成，支持超时

**NativeMethods 关键 API**：
- `CreateFile` — 打开 HID 设备
- `ReadFile` / `WriteFile` — 读写 HID 报告
- `HidD_GetAttributes` — 获取 VID/PID
- `HidD_GetPreparsedData` / `HidP_GetCaps` — 获取设备能力
- `HidD_SetFeature` / `HidD_GetFeature` — 发送/接收 Feature 报告
- `SetupDiGetClassDevs` / `SetupDiEnumDeviceInfo` — 设备枚举
- `RegisterDeviceNotification` — 注册设备通知（供事件监控使用）

### 5.2 ScpDriverInterface（[mi/ScpDriverInterface/](file:///l:/git/Xiaomi_gamepad14/mi/ScpDriverInterface/)）

**来源**：https://github.com/mogzol/ScpDriverInterface

**前置条件**：需要安装 SCP Virtual Bus Driver

**职责**：通过 SCP 虚拟总线驱动在 Windows 中模拟 Xbox 360 控制器。

| 类 | 说明 |
|---|---|
| `ScpBus` | SCP 虚拟总线操作类。通过 DeviceIoControl 与驱动通信 |
| `X360Controller` | 虚拟 Xbox 360 控制器状态（按钮/摇杆/扳机），提供 `GetReport()` 生成标准输入报告 |
| `X360Buttons` | `[Flags]` 枚举，定义所有 Xbox 360 按钮位标志 |

**ScpBus 关键方法**：

| 方法 | IOCTL 码 | 功能 |
|---|---|---|
| `PlugIn(int)` | `0x2A4000` | 在 SCP 总线上插入指定编号的控制器 |
| `Unplug(int)` | `0x2A4004` | 拔出指定编号的控制器 |
| `UnplugAll()` | `0x2A4004` | 拔出所有控制器 |
| `Report(int, byte[], byte[])` | `0x2A400C` | 发送控制器状态报告，可选接收输出报告（含震动数据） |

**X360Controller 属性**：

| 属性 | 类型 | 范围 | 说明 |
|---|---|---|---|
| `Buttons` | `X360Buttons` | Flags | 当前按下的按钮组合 |
| `LeftTrigger` | `byte` | 0–255 | 左扳机模拟值 |
| `RightTrigger` | `byte` | 0–255 | 右扳机模拟值 |
| `LeftStickX` | `short` | -32768–32767 | 左摇杆 X 轴 |
| `LeftStickY` | `short` | -32768–32767 | 左摇杆 Y 轴 |
| `RightStickX` | `short` | -32768–32767 | 右摇杆 X 轴 |
| `RightStickY` | `short` | -32768–32767 | 右摇杆 Y 轴 |

**X360Buttons 枚举**（位标志）：

| 标志 | 位 | 对应按钮 |
|---|---|---|
| `Up` | 0 | 十字键上 |
| `Down` | 1 | 十字键下 |
| `Left` | 2 | 十字键左 |
| `Right` | 3 | 十字键右 |
| `Start` | 4 | 开始键 |
| `Back` | 5 | 返回键 |
| `LeftStick` | 6 | 左摇杆按下 |
| `RightStick` | 7 | 右摇杆按下 |
| `LeftBumper` | 8 | 左肩键 |
| `RightBumper` | 9 | 右肩键 |
| `Logo` | 10 | Xbox 徽标键 |
| `A` | 12 | A 键 |
| `B` | 13 | B 键 |
| `X` | 14 | X 键 |
| `Y` | 15 | Y 键 |

---

## 6. 加速度计读取工具（Linux）

目录：[accelerometer_print/](file:///l:/git/Xiaomi_gamepad14/accelerometer_print/)

**职责**：在 Linux 系统下通过 hidraw 接口读取小米手柄的加速度计数据。

**编译与运行**：
```bash
gcc hidraw.c -o hidraw
sudo chmod a+rw /dev/hidraw5   # 替换为实际设备
./hidraw /dev/hidraw5
```

**功能**：
1. 打开 hidraw 设备文件
2. 读取并打印 HID 报告描述符
3. 发送 Feature 报告 `[0x31, 0x01, 0x08]` 启用加速度计
4. 循环读取输入报告，提取加速度数据（12–17 字节，3 轴各 2 字节有符号小端整数）

**加速度计数据格式**（输入报告偏移）：
```
[12-13] = X 轴 (signed short, little endian)
[14-15] = Y 轴 (signed short, little endian)
[16-17] = Z 轴 (signed short, little endian)
```

---

## 7. JD JZ-V3N 手柄 HID 协议摘要

根据 [readme.txt](file:///l:/git/Xiaomi_gamepad14/readme.txt) 中的逆向工程分析（该协议源自小米手柄逆向工程，JD JZ-V3N 兼容此协议）：

### 输入报告（21 字节）
```
[0]    = 报告 ID (0x04)
[1-2]  = 按钮位 (每字节 1 位/按钮)
[3]    = 0
[4]    = 十字键方向
[5-8]  = 4 轴摇杆 (每轴 1 字节)
[9]    = 0
[10]   = 0
[11]   = 左扳机
[12]   = 右扳机
[13-18]= 加速度计 (3 轴 × 2 字节, signed LE)
[19]   = 电池电量
[20]   = MI 按钮
```

### Set Feature 报告
| 类型 | 数据 | 说明 |
|---|---|---|
| 震动 | `[0x20][小马达][大马达]` | 控制马达震动 |
| 校准 | `[0x22][标志][8字节×2][3字节×2]` | 摇杆/扳机校准（持久保存） |
| 停止输入 | `[0x23][0x01]` / `[0x23][0x00]` | 停止/恢复输入 |
| 启用加速度计 | `[0x31][0x01][灵敏度]` | 启用加速度计 |
| 禁用加速度计 | `[0x31][0x00][0x00]` | 禁用加速度计 |

---

## 8. 构建与运行

### 8.1 环境要求

- **操作系统**：Windows 7 及以上（需管理员权限运行）
- **.NET Framework**：4.5.2 或更高版本
- **SCP Virtual Bus Driver**：必须预先安装（[ScpDriverInterface 项目](https://github.com/mogzol/ScpDriverInterface)）
- **开发工具**：Visual Studio 2015 或更高版本，或 MSBuild

### 8.2 构建方法

**使用 Visual Studio**：
1. 打开 `mi/mi.sln`
2. 选择 `Release` 配置
3. 生成解决方案

**使用 MSBuild 命令行**：
```bash
msbuild mi/mi.csproj /p:Configuration=Release
```

### 8.3 运行

编译产物为 `mi/bin/Release/jzv3n.exe`。

- **必须以管理员权限运行**（程序清单声明 `requireAdministrator`）
- 运行后程序最小化到系统托盘，自动检测并连接小米手柄
- 连接成功后在 Windows 中显示为 Xbox 360 手柄
- 可通过托盘图标右键菜单退出程序

### 8.4 注意事项

1. 程序使用命名 Mutex 确保单实例运行
2. 设备 VID/PID 为 `0x20BC` / `0x505E`（JD JZ-V3N 控制器）
3. 只连接设备路径中包含 `&col03#` 的 HID 集合（游戏控制器接口）
4. 支持最多 4 个手柄同时连接（SCP 总线限制）

---

## 9. 依赖关系图

```
jzv3n.exe
├── System (BCL)
├── System.Core
├── System.Drawing (托盘图标)
├── System.Windows.Forms (托盘/对话框)
├── System.Data
├── System.Xml
├── System.Net.Http
├── HidLibrary (内嵌)
│   └── hid.dll (Windows HID API)
│   └── setupapi.dll (设备枚举 API)
│   └── kernel32.dll (文件/同步 API)
└── ScpDriverInterface (内嵌)
    └── SCP Virtual Bus Driver (系统驱动)
    └── kernel32.dll (DeviceIoControl)
    └── setupapi.dll (设备发现)
```

---

## 10. 项目版本历史

| 版本 | 说明 |
|---|---|
| 1.0.0.0 | 初始版本 |
| 1.1.0.0 | 当前版本（AssemblyInfo 中记录） |

---

*本文档基于对项目源码的完整分析生成，涵盖了项目架构、模块职责、关键类/函数、数据流、协议细节及运行方式等核心信息。*