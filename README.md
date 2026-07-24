<div align="center">

# 🚀 i.MX6ULL Embedded Linux & Qt Multifunctional Platform

**基于 i.MX6ULL 的嵌入式 Linux 驱动调试与多媒体应用综合测试平台**

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Platform](https://img.shields.io/badge/platform-i.MX6ULL%20%7C%20Linux%20%7C%20Ubuntu-orange.svg)
![Language](https://img.shields.io/badge/language-C%2B%2B%20%7C%20Qt5-green.svg)
![Build](https://img.shields.io/badge/build-passing-brightgreen.svg)

[功能特性](#-features) • [效果演示](#-demo--preview) • [安装指南](#-installation) • [快速开始](#-quick-start--usage) • [架构设计](#%EF%B8%8F-tech-stack--architecture) • [路线图](#-roadmap)

---

</div>

## 📌 简介 (Introduction)

**i.MX6ULL Embedded Platform** 是一个基于 NXP i.MX6ULL 芯片与 Qt 5 框架开发的综合性嵌入式 Linux 测试与应用平台。

本项目整合了常见的嵌入式硬件驱动测试（GPIO、I2C、SPI、CAN、UART、ADC）与多媒体应用（UVC 摄像头、音乐/视频播放器、画板、姿态仪表）。同时，项目创新性地引入了 **PC 端软件虚拟设备解耦技术**，在无需真实硬件开发板的情况下，开发者也可在 Ubuntu/Linux 环境下进行跨平台的驱动逻辑与 UI 交互调试。

### 🌟 核心优势

- **⚡ 统一调试界面**：将分散的驱动节点集成于 GUI，提供直观的可视化操控与数据监控。
- **💻 跨平台虚拟模拟**：支持在 PC 端（Ubuntu/WSL）通过虚拟文件、`socat`、`vcan` 及模拟算法对硬件解耦验证。
- **🎵 丰富多媒体支持**：内置硬件加速多媒体播放器与 V4L2 摄像头实时图像处理算法。
- **✈️ 专业仪表组件**：内置自定义姿态指引仪 (ADI) 控件，精准可视化展现传感器姿态角变化。

---

## ✨ Features (核心特性)

| 模块分类 | 功能特性 | 硬件 / 协议支持 | 软件模拟方案 (PC端) |
| :--- | :--- | :--- | :--- |
| **基础外设** | **GPIO 控制** (LED / 蜂鸣器) | `sysfs` (`/sys/class/gpio`) | 文件读写重定向 + UI 状态灯同步 |
| **低速通信** | **UART / CAN 消息收发** | `/dev/ttyS*`, `SocketCAN` | `socat` 虚拟串口对 / `vcan` 虚拟总线 |
| **环境传感** | **AP3216C / ICM20608** | I2C (`ioctl`), SPI 接口 | JSON/CSV 数据驱动，算法模拟数值波动 |
| **模拟采集** | **ADC 电压实时绘制** | IIO 驱动 (`/sys/bus/iio`) | `QTimer` + 正弦波算法模拟实时电压 |
| **视觉影像** | **UVC 摄像头 & 相册** | V4L2 协议, `mmap` | `v4l2loopback` 推流或应用级 Mock |
| **多媒体** | **音乐 / MP4 视频播放器** | Qt Multimedia / MPlayer | MPlayer 后端重定向与窗口句柄 (`WId`) 嵌入 |
| **交互与设置**| **画板 & 屏幕亮度调节** | Touchscreen / Sysfs | QPainter Canvas 自由绘制 & 导出本地 |

---

## 🖼️ Demo / Preview (效果演示)
<img width="1220" height="976" alt="image" src="https://github.com/user-attachments/assets/9c2edad7-7899-4a27-b115-8fcd503efcde" />

---

## 📦 Installation (安装指南)

### 1. 环境要求 (Prerequisites)

- **操作系统**：Ubuntu 18.04/20.04/22.04 LTS 或 WSL2
- **构建工具**：GCC/G++ (>= 7.5), Make
- **框架依赖**：Qt 5.12+ (Qt Creator, Qt Base, Qt Multimedia)
- **辅助工具**：MPlayer, socat, can-utils (用于虚拟模拟环境)

### 2. 依赖项安装

```bash
# 更新系统软件源
sudo apt-get update

# 安装 Qt 5 开发环境与交叉编译基础依赖
sudo apt-get install -y build-essential qtcreator qtbase5-dev qtmultimedia5-dev libqt5multimedia5-plugins

# 安装多媒体播放后端与虚拟调试工具（可选，用于模拟模式）
sudo apt-get install -y mplayer socat can-utils
```

## 🚀 Quick Start / Usage (快速开始)
### 1. 获取源码并编译
```bash
# 克隆仓库
git clone [https://github.com/your-username/imx6ull-qt-platform.git](https://github.com/your-username/imx6ull-qt-platform.git)
cd imx6ull-qt-platform

# 使用 qmake 生成 Makefile 并编译
qmake IMx6ULL.pro
make -j$(nproc)
```

### 2. 启动应用程序
#### 模式 A：PC 虚拟模拟运行 (默认)
```bash
# 启动平台程序
./IMx6ULL
```

#### 模式 B：虚拟通信总线搭建 (用于调试 CAN/UART)
```bash
# 创建虚拟串口对
socat -d -d pty,raw,echo=0 pty,raw,echo=0 &

# 创建虚拟 CAN 总线
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

#### 模式 C：交叉编译至 i.MX6ULL 开发板
```bash
# 加载交叉编译器环境变量 (依据实际 Toolchain 路径)
source /opt/fsl-imx-fb/4.1.15-2.0.0/environment-setup-cortexa7hf-neon-poky-linux-gnueabi

# 编译生成 Target 可执行文件
qmake IMx6ULL.pro
make clean && make -j4

# 将生成的文件传输至开发板
scp IMx6ULL root@<DEVELOPMENT_BOARD_IP>:/home/root/
```

## 🛠️ Tech Stack & Architecture (技术栈与架构)
### 1. 源码组织结构
```bash
.
├── IMx6ULL.pro         # Qt 项目主管理文件
├── Headers/            # 头文件目录 (*.h)
├── Sources/            # C++ 源码目录 (*.cpp)
├── Forms/              # Qt Designer UI 布局文件 (*.ui)
└── Resources/          # 静态资源 (图片、图标、QSS 样式表)
```
### 2. 模块分层设计
```text
┌─────────────────────────────────────────────────────────────┐
|                     Qt GUI Layer (用户交互层)                 |
|   [MainWindow]  [Music/Video]  [SketchPad]  [WidgetADI]     |
└──────────────────────────────┬──────────────────────────────┘
                               │ Signals / Slots
┌──────────────────────────────▼──────────────────────────────┐
|                   Hardware Abstraction Layer                |
|           (硬件接口层 & 虚拟适配层 #ifdef PC_SIM)              |
├───┬───────────┬────────────┬───────────┬───────────┬────────┤
|ADC| GPIO/sysfs| UART/Socket| CAN/Socket| I2C/SPI   | V4L2   |
└───┴─────┬─────┴──────┬─────┴─────┬─────┴─────┬─────┴───┬────┘
          │            │           │           │         │
┌─────────▼────────────▼───────────▼───────────▼─────────▼────┐
|                     Linux Kernel / Device Drivers           |
|         /sys/class  /dev/ttyS*   vcan0       /dev/i2c-*     |
└─────────────────────────────────────────────────────────────┘
```
### 3. 核心功能类说明
**核心控制与 UI 框架**
- main.cpp：程序入口，初始化 QApplication 并加载全局 QSS 样式。
- mainwindow.cpp：主导航中心，管理各子模块的实例化与路由切换。
- config.cpp：全局配置中心（路径映射、分辨率适应、虚拟模式开关）。
- LayoutSquare.cpp：网格响应式布局助手。

**硬件驱动接口 (HAL)**
- gpio.cpp：通过 sysfs 读写 /sys/class/gpio 实现 LED / 蜂鸣器控制。
- can.cpp / uart.cpp：基于 SocketCAN 与 POSIX TTY 的通信接口封装。
- ap3216c.cpp / icm20608.cpp：I2C/SPI 驱动封装，实现光照度与六轴姿态数据解析。
- adc.cpp：Linux IIO 子系统采集与波形数据转换。

**多媒体与可视化**
- music_player.cpp / video.cpp：基于 QProcess + MPlayer 的跨平台播放控制。
- camera.cpp：基于 V4L2 架构的摄像头预览、捕获与帧率管理。
- qfi_ADI.cpp / WidgetADI.cpp：自定义姿态指引仪 (Attitude Director Indicator) 绘图控件。

## 🗺️ Roadmap (未来规划)
- [x] 完成基础 GPIO、UART、CAN、ADC 硬件控制与 UI 绑定
- [x] 实现 AP3216C / ICM20608 传感器数据的图表与仪表可视化
- [x] 完成基于 PC 端的跨平台虚拟软件模拟层重构
- [x] 集成 UVC 摄像头实时图像处理与拍照预览
- [ ] [In Progress] 优化视频播放器在嵌入式端 Framebuffer 的硬解码支持
- [ ] [Planned] 引入 MQTT 协议，将采集的传感器数据同步至云端物联网平台
- [ ] [Planned] 增加 OTA 远程固件与应用程序更新功能

## 🤝 Contributing (贡献指南)
我们非常欢迎社区开发者为本项目贡献代码、报告 Issue 或提出改进建议！
1. Fork 本仓库
2. 创建您的特性分支 (git checkout -b feature/AmazingFeature)
3. 提交您的修改 (git commit -m 'Add some AmazingFeature')
4. 推送到分支 (git push origin feature/AmazingFeature)
5. 提交 Pull Request

## 📄 License & Acknowledgments (开源协议与致谢)
**开源协议**
本项目基于 MIT License 许可协议开源 - 详情请参阅 [LICENSE](https://www.google.com/search?q=LICENSE) 文件。

## 致谢
- [NXP i.MX6ULL](https://www.nxp.com/) 官方 Linux SDK 支持
- [Qt Project](https://www.qt.io/) 提供优秀的跨平台 GUI 框架
- [MPlayer](http://www.mplayerhq.hu/) 多媒体播放引擎
