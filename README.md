# JD JZ-V3N Gamepad to Xbox 360 Controller Mapper

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE.txt)
[![Platform](https://img.shields.io/badge/platform-Windows-brightgreen)](https://github.com/your-repo)
[![.NET](https://img.shields.io/badge/.NET-4.5.2-blue)](https://dotnet.microsoft.com/)

将京东 JZ-V3N 游戏手柄映射为 Xbox 360 手柄的 Windows 工具。支持震动反馈，无延迟，即插即用。

## 功能特点

- **Xbox 360 映射** — 将 JZ-V3N 手柄模拟为标准的 Xbox 360 手柄，可在任何支持 Xbox 手柄的游戏中直接使用
- **震动反馈** — 完整支持双马达可变强度震动
- **低延迟** — 独立高优先级输入线程，实测无感知延迟
- **系统托盘运行** — 最小化到系统托盘，后台静默运行
- **多手柄支持** — 支持同时连接最多 4 个手柄
- **自动重连** — 自动检测手柄插入/拔出

## 系统要求

| 要求 | 说明 |
|---|---|
| 操作系统 | Windows 7/8/10/11 |
| .NET 版本 | .NET Framework 4.5.2 或更高 |
| 权限 | **需要以管理员身份运行** |
| 驱动 | 需要安装 [SCP Virtual Bus Driver](https://github.com/mogzol/ScpDriverInterface) |
| 手柄 | 京东 JZ-V3N 游戏手柄（VID: `0x20BC`, PID: `0x505E`） |

## 快速开始

### 1. 安装 SCP 驱动

SCP Virtual Bus Driver 是让 Windows 识别虚拟 Xbox 360 手柄的关键驱动。

- 下载并安装：[ScpDriverInterface](https://github.com/mogzol/ScpDriverInterface/releases)
- 运行 `ScpDriver.exe` 并点击 "Install"

### 2. 运行程序

1. 从 [Releases](../../releases) 下载最新版 `jzv3n.exe`
2. **右键 → 以管理员身份运行**（程序需要管理员权限访问 HID 设备和 SCP 总线）
3. 程序最小化到系统托盘，图标为手柄图案
4. 连接 JZ-V3N 手柄（USB 或 2.4G 接收器）
5. 程序自动检测并映射，Windows 中会显示新增了一个 Xbox 360 手柄
6. 打开游戏即可正常使用

### 3. 退出程序

右键系统托盘手柄图标 → 选择 **Exit**

## 手动构建

### 使用 Visual Studio

```bash
# 打开解决方案
mi/mi.sln

# 选择 Release 配置 → 生成解决方案
```

### 使用 MSBuild 命令行

```bash
msbuild mi/mi.csproj /p:Configuration=Release
```

编译产物位于 `mi/bin/Release/jzv3n.exe`

## 按钮映射

| JZ-V3N 手柄 | Xbox 360 对应 |
|---|---|
| A | A |
| B | B |
| X | X |
| Y | Y |
| LB (左肩键) | Left Bumper |
| RB (右肩键) | Right Bumper |
| 左摇杆按下 | Left Stick |
| 右摇杆按下 | Right Stick |
| Start | Start |
| Back | Back |
| 左摇杆 | 左摇杆 (X/Y 轴) |
| 右摇杆 | 右摇杆 (X/Y 轴) |
| 左扳机 (L2) | Left Trigger |
| 右扳机 (R2) | Right Trigger |
| 十字键 | D-Pad (上/下/左/右) |

## 项目结构

```
/
├── mi/                          # 主程序 (C# Windows Forms)
│   ├── Program.cs               # 入口 + 手柄管理
│   ├── Xiaomi_gamepad.cs        # 核心：输入读取 + 按键映射 + 震动
│   ├── ProcessIcon.cs           # 系统托盘图标
│   ├── ContextMenus.cs          # 托盘右键菜单
│   ├── AboutBox.cs              # 关于对话框
│   ├── HidLibrary/              # HID 通信库 (第三方，经修改)
│   └── ScpDriverInterface/      # SCP 总线接口 (Xbox 360 模拟)
├── accelerometer_print/         # Linux 加速度计读取工具 (C)
├── readme.txt                   # HID 协议逆向工程文档
└── CODE_WIKI.md                 # 完整代码 Wiki 文档
```

## 技术细节

- 使用 `HidLibrary` 通过 USB HID 协议读取手柄原始输入数据
- 使用 `ScpDriverInterface` 通过 SCP 虚拟总线驱动模拟 Xbox 360 手柄
- 输入线程以最高优先级独立运行，确保最低延迟
- 震动反馈通过独立的监听线程实现，实时响应游戏震动指令

## 鸣谢

- [ScpDriverInterface](https://github.com/mogzol/ScpDriverInterface) — SCP 虚拟总线驱动接口
- [HidLibrary](https://github.com/mikeobrien/HidLibrary/) — HID 设备通信库
- [MiControllerPluginForInputMapper](https://github.com/aadfPT/MiControllerPluginForInputMapper) — 参考实现
- [EAll4Windows](https://github.com/aadfPT/EAll4Windows) — 参考实现
- 原 Xiaomi Gamepad 逆向工程作者 — HID 协议分析基础

## 许可

本项目基于 [MIT 许可证](mi/LICENSE.txt) 发布。