# AppHub 客户端集成平台 - 设计文档

> **版本**: v1.0  
> **日期**: 2026-06-03  
> **技术栈**: Qt 6 / C++17  
> **状态**: 设计阶段  

---

## 目录

- [1. 产品概述](#1-产品概述)
  - [1.1 产品定位](#11-产品定位)
  - [1.2 目标用户](#12-目标用户)
  - [1.3 核心价值](#13-核心价值)
  - [1.4 功能概述](#14-功能概述)
  - [1.5 用户流程](#15-用户流程)
- [2. 技术架构设计](#2-技术架构设计)
  - [2.1 技术选型](#21-技术选型)
  - [2.2 系统架构图](#22-系统架构图)
  - [2.3 模块划分](#23-模块划分)
  - [2.4 进程嵌入方案](#24-进程嵌入方案)
  - [2.5 进程间通信](#25-进程间通信)
  - [2.6 生命周期管理](#26-生命周期管理)
- [3. UI / 交互设计](#3-ui--交互设计)
  - [3.1 主题系统](#31-主题系统)
  - [3.2 页面结构](#32-页面结构)
  - [3.3 视图模式](#33-视图模式)
  - [3.4 侧边栏](#34-侧边栏)
  - [3.5 标签页系统](#35-标签页系统)
  - [3.6 嵌入工具栏](#36-嵌入工具栏)
  - [3.7 交互状态说明](#37-交互状态说明)
  - [3.8 键盘快捷键](#38-键盘快捷键)
- [4. 数据与接口设计](#4-数据与接口设计)
  - [4.1 应用配置数据结构](#41-应用配置数据结构)
  - [4.2 用户配置](#42-用户配置)
  - [4.3 应用注册机制](#43-应用注册机制)
  - [4.4 内部接口设计](#44-内部接口设计)
  - [4.5 持久化存储](#45-持久化存储)
- [5. 非功能性需求](#5-非功能性需求)
- [6. 里程碑与版本规划](#6-里程碑与版本规划)
- [附录](#附录)

---

## 1. 产品概述

### 1.1 产品定位

**AppHub** 是一个基于 Qt 的桌面端客户端集成平台，用于将多个独立的桌面应用程序（exe）统一集成到一个主界面中进行管理和使用。用户可以在一个窗口内同时打开、切换和管理多个外部应用程序，实现"一个平台，多个工具"的工作体验。

### 1.2 目标用户

| 用户类型 | 描述 | 核心诉求 |
|---------|------|---------|
| 开发者 | 需要同时使用多个开发工具（IDE、终端、数据库、API 测试等） | 减少窗口切换，统一管理开发工具链 |
| 运维人员 | 需要监控多个运维工具面板 | 多面板并排展示，快速切换 |
| 项目经理 | 需要同时查看项目管理、文档、沟通工具 | 一站式访问常用办公工具 |
| 企业内部用户 | 需要访问公司内部多个独立客户端系统 | 统一入口，减少桌面混乱 |

### 1.3 核心价值

```
┌─────────────────────────────────────────────────┐
│                  AppHub 核心价值                   │
├─────────────────────────────────────────────────┤
│                                                   │
│  ① 统一管理   → 一个入口管理所有 exe 应用         │
│  ② 多开并行   → 同时嵌入运行多个 exe              │
│  ③ 快速切换   → 标签页式管理，一键切换            │
│  ④ 分类导航   → 按类别组织应用，快速查找          │
│  ⑤ 双主题     → 暗色/亮色主题，适配不同场景       │
│                                                   │
└─────────────────────────────────────────────────┘
```

### 1.4 功能概述

#### P0 - 核心功能（首版必须）

| 功能 | 说明 |
|------|------|
| 应用注册 | 支持用户手动添加 exe 路径，注册到平台 |
| 应用启动 | 点击图标启动对应 exe，嵌入到平台窗口内 |
| 多开支持 | 支持同时打开多个 exe，以标签页形式管理 |
| 标签管理 | 标签页切换、关闭、排序 |
| 侧边栏导航 | 左侧分类列表，展示所有已注册应用 |
| 搜索过滤 | 支持按名称/描述模糊搜索应用 |
| 分类管理 | 应用按类别分组（开发、设计、办公、数据等） |
| 双主题 | 暗色/亮色主题切换 |

#### P1 - 增强功能（二期）

| 功能 | 说明 |
|------|------|
| 窗口独立弹出 | 将嵌入的 exe 弹出为独立窗口 |
| 窗口置顶 | 嵌入窗口置顶显示 |
| 应用收藏 | 标记常用应用到首页快捷区 |
| 拖拽排序 | 侧边栏和标签页支持拖拽排序 |
| 最近使用 | 记录最近打开的应用，快速访问 |

#### P2 - 扩展功能（三期）

| 功能 | 说明 |
|------|------|
| 应用分组布局 | 多个 exe 在同一视口内分屏显示 |
| 插件系统 | 支持第三方扩展插件 |
| 远程 exe | 支持连接远程桌面中的应用 |
| 团队协作 | 共享应用配置和布局 |

### 1.5 用户流程

#### 首次使用流程

```
启动 AppHub
    │
    ▼
进入首页（空状态/欢迎页）
    │
    ▼
点击「管理应用」或「添加」按钮
    │
    ▼
弹出「添加应用」对话框 (QFileDialog)
    │
    ├── 选择 exe 文件路径
    ├── 填写应用名称（自动读取 exe 信息）
    ├── 选择分类
    └── 确认添加
    │
    ▼
应用出现在侧边栏和首页网格中
    │
    ▼
点击应用图标 → 启动并嵌入 → 进入工作台视图
```

#### 日常使用流程

```
启动 AppHub
    │
    ▼
首页展示已注册应用网格
    │
    ├── 方式 A: 在首页点击卡片 → 启动应用 → 自动进入工作台
    ├── 方式 B: 在侧边栏点击列表项 → 启动应用 → 自动进入工作台
    └── 方式 C: Ctrl+K 搜索 → 回车启动
    │
    ▼
工作台视图：标签页管理已打开的应用
    │
    ├── 切换标签页 → 切换嵌入的 exe 窗口
    ├── 点击 + → 返回首页添加更多应用
    └── 关闭标签 → 终止对应 exe 进程
    │
    ▼
退出 AppHub → 关闭所有嵌入的 exe 进程
```

---

## 2. 技术架构设计

### 2.1 技术选型

| 层级 | 技术方案 | 选型理由 |
|------|---------|---------|
| **UI 框架** | Qt 6.5+ (Qt Widgets) | 成熟的跨平台 C++ GUI 框架，原生支持 Win32 API |
| **编程语言** | C++17 | 高性能，Qt 官方支持，现代语言特性 |
| **构建系统** | CMake 3.22+ | Qt 6 官方推荐，跨平台，IDE 集成良好 |
| **进程管理** | QProcess | Qt 原生子进程管理，信号槽通知 |
| **窗口嵌入** | QWindow::fromWinId() + QWidget::createWindowContainer() | Qt 官方方案，将外部 HWND 转为 QWidget |
| **数据存储** | QSettings + QJsonDocument | 轻量级配置存储，支持 JSON/INI 格式 |
| **信号通信** | Qt 信号槽机制 | 类型安全，线程安全，Qt 核心特性 |
| **样式系统** | QSS (Qt Style Sheets) + QPalette | 双主题支持，类似 CSS 的样式系统 |
| **Win32 桥接** | Windows.h (直接调用) | Qt 原生支持，无需额外 FFI 库 |

### 2.2 系统架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                        AppHub 主窗口 (QMainWindow)               │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────┐ │
│  │ ProcessManager│  │ WindowManager│  │ ConfigManager          │ │
│  │ (进程管理)    │  │ (窗口管理)    │  │ (配置管理)             │ │
│  └──────┬───────┘  └──────┬───────┘  └────────┬───────────────┘ │
│         │                 │                    │                  │
│         └─────────────────┴────────────────────┘                  │
│                           │                                       │
│                    信号槽通信 (Signals & Slots)                    │
│                           │                                       │
└───────────────────────────┼───────────────────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────────────────┐
│                      UI 层 (Qt Widgets)                            │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │  MainWindow (QMainWindow)                                    │  │
│  │  ├── TitleBar (QWidget)           标题栏                      │  │
│  │  ├── Sidebar (QDockWidget)        侧边栏                      │  │
│  │  │   ├── SearchBar (QLineEdit)                                │  │
│  │  │   ├── CategoryList (QListWidget)                           │  │
│  │  │   └── AppList (QListWidget)                                │  │
│  │  └── CentralWidget (QStackedWidget) 主内容区                  │  │
│  │      ├── HomePage (QWidget)        首页视图                   │  │
│  │      │   └── AppGrid (QGridLayout) 应用网格                   │  │
│  │      └── WorkspacePage (QWidget)   工作台视图                 │  │
│  │          ├── TabBar (QTabBar)      标签栏                     │  │
│  │          ├── EmbedToolbar (QWidget)嵌入工具栏                 │  │
│  │          └── EmbedArea (QWidget)   嵌入区域                   │  │
│  │              └── [QWindow Containers] 嵌入的 exe 窗口         │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                                                                     │
└───────────────────────────────────────────────────────────────────┘
        │
        │ Win32 API (SetParent / SetWindowLong)
        ▼
┌──────────┐  ┌──────────┐  ┌──────────┐
│  EXE-A   │  │  EXE-B   │  │  EXE-C   │  ← 外部子进程 (QProcess)
│ (HWND)   │  │ (HWND)   │  │ (HWND)   │
└──────────┘  └──────────┘  └──────────┘
```

### 2.3 模块划分

```
AppHub/
├── CMakeLists.txt                # 主 CMake 配置
├── src/
│   ├── main.cpp                  # 程序入口
│   │
│   ├── core/                     # 核心业务逻辑
│   │   ├── ProcessManager.h/cpp  # 进程生命周期管理
│   │   ├── WindowManager.h/cpp   # 窗口嵌入与切换
│   │   ├── ConfigManager.h/cpp   # 配置读写
│   │   └── AppRegistry.h/cpp     # 应用注册表
│   │
│   ├── ui/                       # UI 界面
│   │   ├── MainWindow.h/cpp      # 主窗口
│   │   ├── TitleBar.h/cpp        # 自定义标题栏
│   │   ├── Sidebar.h/cpp         # 侧边栏
│   │   ├── HomePage.h/cpp        # 首页视图
│   │   ├── WorkspacePage.h/cpp   # 工作台视图
│   │   ├── TabBar.h/cpp          # 自定义标签栏
│   │   ├── EmbedToolbar.h/cpp    # 嵌入工具栏
│   │   ├── AppCard.h/cpp         # 应用卡片组件
│   │   ├── AppItem.h/cpp         # 列表项组件
│   │   └── Dialogs/              # 对话框
│   │       ├── AddAppDialog.h/cpp
│   │       └── SettingsDialog.h/cpp
│   │
│   ├── models/                   # 数据模型
│   │   ├── AppModel.h/cpp        # 应用数据模型
│   │   ├── TabModel.h/cpp        # 标签页模型
│   │   └── ConfigModel.h/cpp     # 配置模型
│   │
│   ├── utils/                    # 工具类
│   │   ├── Win32Helper.h/cpp     # Win32 API 封装
│   │   ├── IconExtractor.h/cpp   # 图标提取工具
│   │   ├── ThemeManager.h/cpp    # 主题管理器
│   │   └── Logger.h/cpp          # 日志工具
│   │
│   └── resources/                # 资源文件
│       ├── resources.qrc         # Qt 资源文件
│       ├── icons/                # 图标资源
│       ├── themes/               # 主题 QSS 文件
│       │   ├── dark.qss
│       │   └── light.qss
│       └── translations/         # 国际化翻译
│
├── third_party/                  # 第三方库（如有）
│
└── docs/                         # 文档
    ├── DESIGN_DOC.md
    └── API.md
```

### 2.4 进程嵌入方案

AppHub 的核心技术难点是将外部 exe 窗口嵌入到 Qt 窗口内。方案如下：

#### Windows 平台 (Win32 API + Qt)

```
┌────────────────────────────────────────────────┐
│  AppHub (QMainWindow)                           │
│                                                  │
│  ┌───────────────────────────────────────────┐  │
│  │  EmbedArea (QWidget)                       │  │
│  │                                            │  │
│  │  ┌─────────────────────────────────────┐  │  │
│  │  │  Container (QWidget)                │  │  │
│  │  │  createWindowContainer(QWindow)     │  │  │
│  │  │                                      │  │  │
│  │  │  QWindow::fromWinId(HWND)           │  │  │
│  │  │         ↓                            │  │  │
│  │  │  ┌──────────────────────────────┐   │  │  │
│  │  │  │   外部 EXE 窗口              │   │  │  │
│  │  │  │   (子进程的 HWND)            │   │  │  │
│  │  │  └──────────────────────────────┘   │  │  │
│  │  │                                      │  │  │
│  │  └─────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────┘  │
│                                                  │
└────────────────────────────────────────────────┘
```

**嵌入流程**:

```cpp
// ProcessManager.cpp - 伪代码
bool ProcessManager::launchAndEmbed(const QString& exePath, QWidget* container) {
    // 1. 启动子进程
    QProcess* process = new QProcess(this);
    process->start(exePath, QStringList());
    
    if (!process->waitForStarted(5000)) {
        qWarning() << "Failed to start process:" << exePath;
        return false;
    }
    
    // 2. 等待子进程创建窗口（轮询获取 HWND）
    HWND childHwnd = nullptr;
    for (int i = 0; i < 50; ++i) {  // 最多等待 10 秒
        childHwnd = Win32Helper::findWindowByPid(process->processId());
        if (childHwnd) break;
        QThread::msleep(200);
    }
    
    if (!childHwnd) {
        qWarning() << "Failed to find window for PID:" << process->processId();
        process->kill();
        return false;
    }
    
    // 3. 修改子窗口样式（去除标题栏等）
    LONG style = GetWindowLong(childHwnd, GWL_STYLE);
    style &= ~(WS_POPUP | WS_CAPTION | WS_THICKFRAME);
    style |= WS_CHILD | WS_VISIBLE;
    SetWindowLong(childHwnd, GWL_STYLE, style);
    
    // 4. 将子窗口嵌入 Qt 容器
    QWindow* window = QWindow::fromWinId((WId)childHwnd);
    QWidget* embeddedWidget = QWidget::createWindowContainer(window, container);
    
    // 5. 调整大小以填充容器
    embeddedWidget->setGeometry(container->rect());
    embeddedWidget->show();
    
    // 6. 保存嵌入信息
    EmbeddedProcess info;
    info.process = process;
    info.hwnd = childHwnd;
    info.widget = embeddedWidget;
    info.appId = appId;
    
    m_embeddedProcesses.insert(appId, info);
    
    // 7. 连接信号
    connect(process, &QProcess::finished, this, &ProcessManager::onProcessFinished);
    connect(process, &QProcess::errorOccurred, this, &ProcessManager::onProcessError);
    
    emit processEmbedded(appId, process->processId());
    return true;
}
```

**Win32Helper 工具类**:

```cpp
// Win32Helper.h
#pragma once
#include <windows.h>
#include <QString>

class Win32Helper {
public:
    // 通过进程 ID 查找窗口句柄
    static HWND findWindowByPid(DWORD pid);
    
    // 修改窗口样式
    static void setChildWindowStyle(HWND hwnd);
    
    // 调整窗口大小
    static void resizeWindow(HWND hwnd, int width, int height);
    
    // 设置父窗口
    static void setParentWindow(HWND child, HWND parent);
    
    // 显示/隐藏窗口
    static void showWindow(HWND hwnd, bool visible);
    
private:
    // 枚举窗口回调
    struct EnumData {
        DWORD pid;
        HWND hwnd;
    };
    static BOOL CALLBACK enumWindowsProc(HWND hwnd, LPARAM lParam);
};
```

**Win32Helper 实现**:

```cpp
// Win32Helper.cpp
#include "Win32Helper.h"

HWND Win32Helper::findWindowByPid(DWORD pid) {
    EnumData data = { pid, nullptr };
    EnumWindows(enumWindowsProc, (LPARAM)&data);
    return data.hwnd;
}

BOOL CALLBACK Win32Helper::enumWindowsProc(HWND hwnd, LPARAM lParam) {
    EnumData* data = (EnumData*)lParam;
    DWORD windowPid = 0;
    GetWindowThreadProcessId(hwnd, &windowPid);
    
    if (windowPid == data->pid) {
        // 检查窗口是否可见（排除隐藏窗口）
        if (IsWindowVisible(hwnd)) {
            data->hwnd = hwnd;
            return FALSE;  // 停止枚举
        }
    }
    return TRUE;  // 继续枚举
}

void Win32Helper::setChildWindowStyle(HWND hwnd) {
    LONG style = GetWindowLong(hwnd, GWL_STYLE);
    style &= ~(WS_POPUP | WS_CAPTION | WS_THICKFRAME | WS_MINIMIZEBOX | WS_MAXIMIZEBOX);
    style |= WS_CHILD | WS_VISIBLE | WS_CLIPCHILDREN;
    SetWindowLong(hwnd, GWL_STYLE, style);
    
    LONG exStyle = GetWindowLong(hwnd, GWL_EXSTYLE);
    exStyle &= ~(WS_EX_DLGMODALFRAME | WS_EX_WINDOWEDGE | WS_EX_CLIENTEDGE | WS_EX_STATICEDGE);
    SetWindowLong(hwnd, GWL_EXSTYLE, exStyle);
}

void Win32Helper::resizeWindow(HWND hwnd, int width, int height) {
    SetWindowPos(hwnd, nullptr, 0, 0, width, height, 
                 SWP_NOZORDER | SWP_NOACTIVATE | SWP_FRAMECHANGED);
}

void Win32Helper::setParentWindow(HWND child, HWND parent) {
    SetParent(child, parent);
}

void Win32Helper::showWindow(HWND hwnd, bool visible) {
    ShowWindow(hwnd, visible ? SW_SHOW : SW_HIDE);
}
```

### 2.5 进程间通信

#### 主窗口 ↔ 子模块 (Qt 信号槽)

```cpp
// ProcessManager.h
class ProcessManager : public QObject {
    Q_OBJECT
public:
    bool launchAndEmbed(const QString& appId, const QString& exePath, QWidget* container);
    void closeProcess(const QString& appId);
    
signals:
    // 进程状态变化信号
    void processLaunched(const QString& appId, qint64 pid);
    void processEmbedded(const QString& appId, qint64 pid);
    void processFinished(const QString& appId, int exitCode);
    void processError(const QString& appId, const QString& error);
    
public slots:
    void onRefreshEmbed(const QString& appId);
    void onDetachWindow(const QString& appId);
    
private slots:
    void onProcessFinished(int exitCode, QProcess::ExitStatus status);
    void onProcessError(QProcess::ProcessError error);
};

// MainWindow.cpp - 连接信号槽
void MainWindow::setupConnections() {
    connect(m_processManager, &ProcessManager::processEmbedded,
            this, &MainWindow::onProcessEmbedded);
    connect(m_processManager, &ProcessManager::processFinished,
            this, &MainWindow::onProcessFinished);
    
    connect(m_tabBar, &TabBar::tabCloseRequested,
            this, &MainWindow::onTabCloseRequested);
    connect(m_tabBar, &TabBar::currentChanged,
            this, &MainWindow::onTabChanged);
}
```

#### 主窗口 ↔ 子进程 (QProcess)

```cpp
// 启动子进程
QProcess* process = new QProcess(this);
process->setProcessChannelMode(QProcess::MergedChannels);

// 读取子进程输出
connect(process, &QProcess::readyReadStandardOutput, this, [=]() {
    QByteArray output = process->readAllStandardOutput();
    qDebug() << "Process output:" << output;
});

// 监听进程结束
connect(process, QOverload<int, QProcess::ExitStatus>::of(&QProcess::finished),
        this, [=](int exitCode, QProcess::ExitStatus status) {
    qDebug() << "Process finished with code:" << exitCode;
    emit processFinished(appId, exitCode);
});

// 启动进程
process->start(exePath, args);
```

### 2.6 生命周期管理

```
┌──────────┐    launch     ┌──────────┐    embed     ┌──────────┐
│  IDLE    │ ────────────► │ LAUNCHING│ ───────────► │ EMBEDDED │
│ (未启动)  │              │ (启动中)  │              │ (已嵌入)  │
└──────────┘              └────┬─────┘              └─────┬────┘
                               │                          │
                          fail │                     close│
                               ▼                          ▼
                          ┌──────────┐             ┌──────────┐
                          │  ERROR   │             │ CLOSING  │
                          │ (错误)    │             │ (关闭中)  │
                          └──────────┘             └─────┬────┘
                                                         │
                                                    done │
                                                         ▼
                                                   ┌──────────┐
                                                   │  IDLE    │
                                                   │ (未启动)  │
                                                   └──────────┘
```

**进程状态枚举**:

```cpp
// ProcessManager.h
enum class ProcessState {
    Idle,       // 未启动
    Launching,  // 启动中（等待窗口创建）
    Embedded,   // 已嵌入（正常运行）
    Error,      // 启动失败
    Closing     // 关闭中（正在终止）
};

// 嵌入进程信息结构
struct EmbeddedProcess {
    QString appId;
    QProcess* process;
    HWND hwnd;
    QWidget* widget;
    ProcessState state;
    qint64 pid;
};
```

**退出策略**:

| 场景 | 行为 |
|------|------|
| 用户关闭标签页 | 调用 `process->terminate()`，等待 3s 后 `process->kill()` |
| 用户退出 AppHub | 遍历所有子进程，批量调用 `terminate()` |
| 子进程意外崩溃 | `QProcess::finished` 信号触发，更新状态，提示用户 |
| AppHub 意外崩溃 | 子进程失去父窗口，由 OS 回收或手动清理 |

```cpp
// ProcessManager.cpp
void ProcessManager::closeProcess(const QString& appId) {
    if (!m_embeddedProcesses.contains(appId)) return;
    
    EmbeddedProcess& info = m_embeddedProcesses[appId];
    info.state = ProcessState::Closing;
    
    // 尝试优雅关闭
    info.process->terminate();
    
    // 等待 3 秒
    if (!info.process->waitForFinished(3000)) {
        // 强制终止
        info.process->kill();
        info.process->waitForFinished(1000);
    }
    
    // 清理资源
    delete info.widget;
    m_embeddedProcesses.remove(appId);
    
    emit processFinished(appId, 0);
}

void ProcessManager::closeAllProcesses() {
    for (auto it = m_embeddedProcesses.begin(); it != m_embeddedProcesses.end(); ++it) {
        it->process->terminate();
    }
    
    // 等待所有进程结束（最多 5 秒）
    for (auto& info : m_embeddedProcesses) {
        if (!info.process->waitForFinished(1000)) {
            info.process->kill();
        }
    }
    
    m_embeddedProcesses.clear();
}
```

---

## 3. UI / 交互设计

> 详细视觉参数参见 `design-system.html` (亮色) / `apphub-spec.html` (暗色)

### 3.1 主题系统

AppHub 支持暗色 (Dark) 和亮色 (Light) 两套主题，通过 QSS 样式表实现切换。

#### 色彩体系对比

| 用途 | 暗色主题 | 亮色主题 | QSS 变量 |
|------|---------|---------|----------|
| 页面背景 | `#0f0f1a` | `#f5f5f7` | `@bg-color` |
| 侧边栏/卡片 | `#1e1e2e` | `#ffffff` | `@sidebar-bg` |
| 悬停背景 | `#313244` | `#f3f4f6` | `@hover-bg` |
| 边框 | `#313244` | `#e5e7eb` | `@border-color` |
| 主色 | `#89b4fa` | `#3b82f6` | `@accent-color` |
| 成功色 | `#a6e3a1` | `#10b981` | `@success-color` |
| 危险色 | `#f38ba8` | `#ef4444` | `@danger-color` |
| 主文本 | `#cdd6f4` | `#111827` | `@text-color` |
| 弱文本 | `#6c7086` | `#9ca3af` | `@text-dim` |

#### 主题切换实现

```cpp
// ThemeManager.h
class ThemeManager : public QObject {
    Q_OBJECT
public:
    enum Theme { Dark, Light };
    
    static ThemeManager& instance();
    
    void setTheme(Theme theme);
    Theme currentTheme() const { return m_currentTheme; }
    
signals:
    void themeChanged(Theme theme);
    
private:
    Theme m_currentTheme = Dark;
    QString loadQSS(Theme theme);
};

// ThemeManager.cpp
void ThemeManager::setTheme(Theme theme) {
    if (m_currentTheme == theme) return;
    
    m_currentTheme = theme;
    
    // 加载 QSS 文件
    QString qss = loadQSS(theme);
    qApp->setStyleSheet(qss);
    
    // 更新调色板
    QPalette palette = createPalette(theme);
    qApp->setPalette(palette);
    
    emit themeChanged(theme);
}

QString ThemeManager::loadQSS(Theme theme) {
    QFile file(theme == Dark ? ":/themes/dark.qss" : ":/themes/light.qss");
    if (file.open(QFile::ReadOnly | QFile::Text)) {
        return QString::fromUtf8(file.readAll());
    }
    return QString();
}
```

**dark.qss 示例**:

```css
/* dark.qss */
QWidget {
    background-color: #0f0f1a;
    color: #cdd6f4;
    font-family: "Microsoft YaHei", "Segoe UI", sans-serif;
}

QMainWindow {
    background-color: #0f0f1a;
}

/* 侧边栏 */
#sidebar {
    background-color: #1e1e2e;
    border-right: 1px solid #313244;
}

/* 按钮 */
QPushButton {
    background-color: transparent;
    color: #6c7086;
    border: none;
    border-radius: 6px;
    padding: 5px 14px;
}

QPushButton:hover {
    background-color: #313244;
    color: #cdd6f4;
}

QPushButton:checked {
    background-color: #89b4fa;
    color: #1e1e2e;
    font-weight: 600;
}

/* 输入框 */
QLineEdit {
    background-color: #313244;
    color: #cdd6f4;
    border: 1px solid transparent;
    border-radius: 8px;
    padding: 6px 10px;
}

QLineEdit:focus {
    border-color: #89b4fa;
}
```

### 3.2 页面结构

```
┌─────────────────────────────────────────────────────────────┐
│  TitleBar (44px) - QWidget                                   │
│  [Logo] AppHub  [首页] [工作台]  ──────────  [─] [□] [×]    │
├──────────┬──────────────────────────────────────────────────┤
│          │                                                    │
│ Sidebar  │            CentralWidget (QStackedWidget)         │
│ (240px)  │                                                    │
│QDockWidget│   ┌─────────────────────────────────────────┐    │
│          │   │  HomePage / WorkspacePage                  │    │
│ SearchBar│   │                                           │    │
│ Category │   │  (两种视图通过 setCurrentWidget 切换)     │    │
│ AppList  │   │                                           │    │
│          │   │                                           │    │
│          │   │                                           │    │
│          │   └─────────────────────────────────────────┘    │
│ Footer   │                                                    │
├──────────┴──────────────────────────────────────────────────┤
└─────────────────────────────────────────────────────────────┘
```

**MainWindow 类结构**:

```cpp
// MainWindow.h
class MainWindow : public QMainWindow {
    Q_OBJECT
public:
    explicit MainWindow(QWidget* parent = nullptr);
    ~MainWindow();
    
private:
    // UI 组件
    TitleBar* m_titleBar;
    Sidebar* m_sidebar;
    QStackedWidget* m_centralStack;
    HomePage* m_homePage;
    WorkspacePage* m_workspacePage;
    
    // 管理器
    ProcessManager* m_processManager;
    ConfigManager* m_configManager;
    ThemeManager* m_themeManager;
    
    void setupUI();
    void setupConnections();
    void loadSettings();
    
private slots:
    void onSwitchToHome();
    void onSwitchToWorkspace();
    void onAppLaunchRequested(const QString& appId);
    void onProcessEmbedded(const QString& appId, qint64 pid);
    void onProcessFinished(const QString& appId, int exitCode);
};
```

### 3.3 视图模式

#### 首页视图 (HomePage)

```cpp
// HomePage.h
class HomePage : public QWidget {
    Q_OBJECT
public:
    explicit HomePage(QWidget* parent = nullptr);
    
    void setApps(const QList<AppInfo>& apps);
    void filterApps(const QString& keyword);
    
signals:
    void appLaunchRequested(const QString& appId);
    
private:
    QWidget* m_headerWidget;
    QLabel* m_logoLabel;
    QLabel* m_titleLabel;
    QLabel* m_subtitleLabel;
    QLineEdit* m_searchBar;
    QScrollArea* m_scrollArea;
    QWidget* m_gridContainer;
    QGridLayout* m_gridLayout;
    
    QList<AppCard*> m_appCards;
    
    void setupUI();
    void rebuildGrid();
};

// AppCard.h - 应用卡片组件
class AppCard : public QWidget {
    Q_OBJECT
public:
    AppCard(const AppInfo& app, QWidget* parent = nullptr);
    
signals:
    void clicked(const QString& appId);
    
protected:
    void mousePressEvent(QMouseEvent* event) override;
    void paintEvent(QPaintEvent* event) override;
    void enterEvent(QEnterEvent* event) override;
    void leaveEvent(QEvent* event) override;
    
private:
    AppInfo m_app;
    bool m_hovered = false;
    
    // 绘制图标（渐变色 + 缩写文字）
    void paintIcon(QPainter& painter, const QRect& rect);
};
```

- 居中布局，最大宽度 720px
- 卡片网格: `QGridLayout`，每行自动填充，最小列宽 116px，间距 16px
- 卡片尺寸: 最小 116px 宽，图标 48×48px
- 点击卡片 → 发射 `appLaunchRequested` 信号 → 启动应用 → 切换到工作台

#### 工作台视图 (WorkspacePage)

```cpp
// WorkspacePage.h
class WorkspacePage : public QWidget {
    Q_OBJECT
public:
    explicit WorkspacePage(QWidget* parent = nullptr);
    
    int addTab(const QString& appId, const QString& title, const QIcon& icon);
    void removeTab(int index);
    void setActiveTab(int index);
    
    void setEmbedWidget(const QString& appId, QWidget* widget);
    void removeEmbedWidget(const QString& appId);
    
signals:
    void tabCloseRequested(int index);
    void tabChanged(int index);
    void newTabRequested();
    
private:
    QTabBar* m_tabBar;
    EmbedToolbar* m_toolbar;
    QStackedWidget* m_embedStack;
    QWidget* m_emptyState;
    
    QMap<QString, QWidget*> m_embedWidgets;
    
    void setupUI();
    void updateEmptyState();
};
```

- 标签栏: `QTabBar`，高度 38px，标签高 32px
- 工具栏: `QWidget`，高度 30px，显示连接状态和操作按钮
- 嵌入区域: `QStackedWidget`，占满剩余空间

### 3.4 侧边栏

```cpp
// Sidebar.h
class Sidebar : public QDockWidget {
    Q_OBJECT
public:
    explicit Sidebar(QWidget* parent = nullptr);
    
    void setApps(const QList<AppInfo>& apps);
    void filterApps(const QString& keyword);
    void setCategory(const QString& category);
    void setCollapsed(bool collapsed);
    bool isCollapsed() const { return m_collapsed; }
    
signals:
    void appLaunchRequested(const QString& appId);
    void categoryChanged(const QString& category);
    void searchTextChanged(const QString& text);
    void collapsedChanged(bool collapsed);
    
private:
    bool m_collapsed = false;
    
    // UI 组件
    QPushButton* m_collapseBtn;
    QLineEdit* m_searchBar;
    QListWidget* m_categoryList;
    QListWidget* m_appList;
    QWidget* m_footer;
    
    void setupUI();
    void rebuildAppList();
    
private slots:
    void onCollapseClicked();
    void onCategoryClicked(QListWidgetItem* item);
    void onAppItemClicked(QListWidgetItem* item);
};

// AppItem.h - 列表项组件
class AppItem : public QWidget {
    Q_OBJECT
public:
    AppItem(const AppInfo& app, bool isRunning, QWidget* parent = nullptr);
    
    void setRunning(bool running);
    
signals:
    void clicked(const QString& appId);
    
private:
    AppInfo m_app;
    bool m_running;
    
    QLabel* m_iconLabel;
    QLabel* m_nameLabel;
    QLabel* m_descLabel;
    QLabel* m_badgeLabel;  // "运行中" 徽章
};
```

**折叠态 (56px)**:

- 隐藏搜索框、分类标签、文字、徽章
- 仅显示图标，居中对齐
- 折叠按钮箭头旋转 180°

### 3.5 标签页系统

```cpp
// TabBar.h - 自定义标签栏
class TabBar : public QTabBar {
    Q_OBJECT
public:
    explicit TabBar(QWidget* parent = nullptr);
    
    int addClosableTab(const QString& title, const QIcon& icon);
    
protected:
    void paintEvent(QPaintEvent* event) override;
    void mousePressEvent(QMouseEvent* event) override;
    
private:
    // 绘制关闭按钮
    void paintCloseButton(QPainter& painter, const QRect& tabRect, bool active);
    
    // 检测点击是否在关闭按钮区域
    bool isCloseButtonClicked(int tabIndex, const QPoint& pos);
};
```

```
标签页状态:
┌─────────────────────────┐
│ 激活态: BG=#313244(Dark) / #fff(Light) │
│         底部 2px 主色指示线             │
│         Color = 主文本色               │
├─────────────────────────┤
│ 默认态: BG=transparent                 │
│         Color = 弱文本色               │
├─────────────────────────┤
│ Hover:  BG=rgba(255,255,255,0.03)     │
│         Color = 次文本色               │
└─────────────────────────┘

关闭按钮 ×:
  默认: 14×14px, Color=弱文本
  Hover: BG=rgba(pink,0.2), Color=粉色

新建标签 +:
  26×26px, 点击跳转首页
```

**标签页交互**:

| 操作 | 行为 |
|------|------|
| 单击标签 | 切换到对应嵌入窗口 (`m_embedStack->setCurrentWidget`) |
| 点击 × | 发射 `tabCloseRequested` 信号 → 关闭标签，终止 exe 进程 |
| 点击 + | 发射 `newTabRequested` 信号 → 返回首页 |
| 关闭最后一个标签 | 自动返回首页 |
| 标签过多 | 横向滚动，支持鼠标滚轮 |

### 3.6 嵌入工具栏

```cpp
// EmbedToolbar.h
class EmbedToolbar : public QWidget {
    Q_OBJECT
public:
    explicit EmbedToolbar(QWidget* parent = nullptr);
    
    void setAppInfo(const QString& appName, qint64 pid);
    
signals:
    void refreshRequested();
    void detachRequested();
    void closeRequested();
    
private:
    QLabel* m_statusDot;    // 绿色圆点
    QLabel* m_statusLabel;  // "VS Code - PID: 12584"
    QPushButton* m_refreshBtn;
    QPushButton* m_detachBtn;
    QPushButton* m_closeBtn;
    
    void setupUI();
    void setupAnimations();
};
```

```
┌────────────────────────────────────────────────┐
│ ● VS Code - PID: 12584    [刷新] [置顶] [独立] [关闭] │
└────────────────────────────────────────────────┘

左侧: 状态指示
  - 绿色圆点 5×5px + 脉冲动画 (QPropertyAnimation, 2s infinite)
  - 文字: "{应用名} - PID: {进程号}"

右侧: 操作按钮
  - 刷新: 重新调整嵌入窗口大小
  - 置顶: 嵌入窗口 z-index 提升
  - 独立窗口: 弹出为独立 OS 窗口
  - 关闭(danger): 终止进程并关闭标签
```

**脉冲动画实现**:

```cpp
// EmbedToolbar.cpp
void EmbedToolbar::setupAnimations() {
    QPropertyAnimation* animation = new QPropertyAnimation(m_statusDot, "opacity");
    animation->setDuration(2000);
    animation->setStartValue(1.0);
    animation->setEndValue(0.3);
    animation->setLoopCount(-1);  // 无限循环
    animation->setEasingCurve(QEasingCurve::InOutQuad);
    animation->start();
}
```

### 3.7 交互状态说明

#### 应用启动流程

```
用户点击应用图标
    │
    ▼
显示 Toast "正在启动 {应用名} ..." (QLabel + QPropertyAnimation)
    │
    ▼
状态变为 LAUNCHING (侧栏图标显示加载动画)
    │
    ├── 成功 (10s 内找到窗口)
    │   ├── QWindow::fromWinId() + createWindowContainer() 嵌入
    │   ├── 创建标签页
    │   ├── 切换到工作台视图
    │   ├── 状态变为 EMBEDDED
    │   └── Toast "已启动 {应用名}"
    │
    └── 失败 (超时)
        ├── 状态变为 ERROR
        └── Toast "启动失败，请检查应用路径"
```

#### 空状态

```cpp
// WorkspacePage.cpp - 创建空状态页面
void WorkspacePage::setupUI() {
    m_emptyState = new QWidget(this);
    QVBoxLayout* layout = new QVBoxLayout(m_emptyState);
    layout->setAlignment(Qt::AlignCenter);
    
    QLabel* iconLabel = new QLabel("▦");
    iconLabel->setStyleSheet("font-size: 48px; color: #6c7086; opacity: 0.3;");
    iconLabel->setAlignment(Qt::AlignCenter);
    
    QLabel* textLabel = new QLabel("暂无打开的应用");
    textLabel->setStyleSheet("font-size: 14px; color: #6c7086;");
    textLabel->setAlignment(Qt::AlignCenter);
    
    QLabel* hintLabel = new QLabel("从首页或侧边栏选择应用启动");
    hintLabel->setStyleSheet("font-size: 12px; color: #6c7086; "
                             "background: #1a1a2e; border: 1px solid #313244; "
                             "border-radius: 8px; padding: 6px 14px;");
    hintLabel->setAlignment(Qt::AlignCenter);
    
    layout->addWidget(iconLabel);
    layout->addWidget(textLabel);
    layout->addWidget(hintLabel);
    
    m_embedStack->addWidget(m_emptyState);
}

void WorkspacePage::updateEmptyState() {
    if (m_embedStack->count() == 1) {  // 只有空状态页
        m_embedStack->setCurrentWidget(m_emptyState);
    }
}
```

### 3.8 键盘快捷键

```cpp
// MainWindow.cpp
void MainWindow::setupShortcuts() {
    // Ctrl+K - 聚焦搜索框
    QShortcut* searchShortcut = new QShortcut(QKeySequence("Ctrl+K"), this);
    connect(searchShortcut, &QShortcut::activated, this, [this]() {
        if (m_centralStack->currentWidget() == m_homePage) {
            m_homePage->focusSearchBar();
        } else {
            m_sidebar->focusSearchBar();
        }
    });
    
    // Ctrl+1 - 切换到首页
    QShortcut* homeShortcut = new QShortcut(QKeySequence("Ctrl+1"), this);
    connect(homeShortcut, &QShortcut::activated, this, &MainWindow::onSwitchToHome);
    
    // Ctrl+2 - 切换到工作台
    QShortcut* workspaceShortcut = new QShortcut(QKeySequence("Ctrl+2"), this);
    connect(workspaceShortcut, &QShortcut::activated, this, &MainWindow::onSwitchToWorkspace);
    
    // Ctrl+W - 关闭当前标签页
    QShortcut* closeTabShortcut = new QShortcut(QKeySequence("Ctrl+W"), this);
    connect(closeTabShortcut, &QShortcut::activated, this, [this]() {
        int currentIndex = m_workspacePage->currentTabIndex();
        if (currentIndex >= 0) {
            m_workspacePage->removeTab(currentIndex);
        }
    });
    
    // Ctrl+Tab - 切换到下一个标签页
    QShortcut* nextTabShortcut = new QShortcut(QKeySequence("Ctrl+Tab"), this);
    connect(nextTabShortcut, &QShortcut::activated, this, [this]() {
        int currentIndex = m_workspacePage->currentTabIndex();
        int tabCount = m_workspacePage->tabCount();
        if (tabCount > 1) {
            int nextIndex = (currentIndex + 1) % tabCount;
            m_workspacePage->setActiveTab(nextIndex);
        }
    });
}
```

| 快捷键 | 作用 | 作用范围 |
|--------|------|---------|
| `Ctrl + K` | 聚焦搜索框 | 全局 |
| `Ctrl + 1` | 切换到首页 | 全局 |
| `Ctrl + 2` | 切换到工作台 | 全局 |
| `Ctrl + W` | 关闭当前标签页 | 工作台视图 |
| `Ctrl + Tab` | 切换到下一个标签页 | 工作台视图 |
| `Ctrl + Shift + Tab` | 切换到上一个标签页 | 工作台视图 |

---

## 4. 数据与接口设计

### 4.1 应用配置数据结构

```cpp
// AppInfo.h
#pragma once
#include <QString>
#include <QJsonObject>
#include <QDateTime>

// 应用分类
enum class AppCategory {
    Dev,      // 开发工具
    Design,   // 设计工具
    Office,   // 办公软件
    Data,     // 数据工具
    Tool,     // 其他工具
    Other     // 未分类
};

// 图标颜色（渐变色类名）
enum class IconColor {
    C1,  // Blue   - linear-gradient(135deg, #89b4fa, #74c7ec)
    C2,  // Green  - linear-gradient(135deg, #a6e3a1, #94e2d5)
    C3,  // Purple - linear-gradient(135deg, #cba6f7, #b4befe)
    C4,  // Orange - linear-gradient(135deg, #fab387, #f9e2af)
    C5,  // Pink   - linear-gradient(135deg, #f38ba8, #eba0ac)
    C6   // Teal   - linear-gradient(135deg, #94e2d5, #89dceb)
};

// 已注册的应用信息
struct AppInfo {
    QString id;                    // 唯一标识 (UUID)
    QString name;                  // 应用名称
    QString description;           // 应用描述
    QString exePath;               // exe 文件完整路径
    QString iconPath;              // 自定义图标路径 (可选)
    IconColor iconColor;           // 图标渐变色
    QString abbreviation;          // 图标缩写文字 (1-3字符)
    AppCategory category;          // 应用分类
    QString section;               // 分组名称 (侧边栏分组)
    QStringList launchArgs;        // 启动参数 (可选)
    QString workingDir;            // 工作目录 (可选)
    QString windowTitle;           // 窗口标题匹配规则 (可选)
    int sortOrder;                 // 排序权重
    QDateTime createdAt;           // 创建时间
    QDateTime updatedAt;           // 更新时间
    
    // JSON 序列化
    QJsonObject toJson() const;
    static AppInfo fromJson(const QJsonObject& json);
};
```

**JSON 序列化实现**:

```cpp
// AppInfo.cpp
#include "AppInfo.h"
#include <QUuid>

QJsonObject AppInfo::toJson() const {
    QJsonObject obj;
    obj["id"] = id;
    obj["name"] = name;
    obj["description"] = description;
    obj["exePath"] = exePath;
    obj["iconPath"] = iconPath;
    obj["iconColor"] = static_cast<int>(iconColor);
    obj["abbreviation"] = abbreviation;
    obj["category"] = static_cast<int>(category);
    obj["section"] = section;
    obj["launchArgs"] = QJsonArray::fromStringList(launchArgs);
    obj["workingDir"] = workingDir;
    obj["windowTitle"] = windowTitle;
    obj["sortOrder"] = sortOrder;
    obj["createdAt"] = createdAt.toString(Qt::ISODate);
    obj["updatedAt"] = updatedAt.toString(Qt::ISODate);
    return obj;
}

AppInfo AppInfo::fromJson(const QJsonObject& json) {
    AppInfo info;
    info.id = json["id"].toString();
    info.name = json["name"].toString();
    info.description = json["description"].toString();
    info.exePath = json["exePath"].toString();
    info.iconPath = json["iconPath"].toString();
    info.iconColor = static_cast<IconColor>(json["iconColor"].toInt());
    info.abbreviation = json["abbreviation"].toString();
    info.category = static_cast<AppCategory>(json["category"].toInt());
    info.section = json["section"].toString();
    
    QJsonArray argsArray = json["launchArgs"].toArray();
    for (const auto& arg : argsArray) {
        info.launchArgs.append(arg.toString());
    }
    
    info.workingDir = json["workingDir"].toString();
    info.windowTitle = json["windowTitle"].toString();
    info.sortOrder = json["sortOrder"].toInt();
    info.createdAt = QDateTime::fromString(json["createdAt"].toString(), Qt::ISODate);
    info.updatedAt = QDateTime::fromString(json["updatedAt"].toString(), Qt::ISODate);
    
    return info;
}
```

### 4.2 用户配置

```cpp
// UserConfig.h
#pragma once
#include <QString>
#include <QRect>
#include <QStringList>

struct UserConfig {
    // 主题
    int theme;  // 0 = Dark, 1 = Light
    
    // 侧边栏
    bool sidebarCollapsed;
    int sidebarWidth;
    QString activeCategory;  // "all" 或具体分类
    
    // 视图
    QString lastActiveView;  // "home" 或 "workspace"
    
    // 窗口
    QRect windowBounds;
    bool windowMaximized;
    
    // 应用
    QStringList autoLaunchApps;  // 自动启动的应用 ID 列表
    QStringList recentAppIds;    // 最近使用的应用 ID (最多 10 个)
    QStringList favorites;       // 收藏的应用 ID
};
```

### 4.3 应用注册机制

#### AppRegistry 类

```cpp
// AppRegistry.h
class AppRegistry : public QObject {
    Q_OBJECT
public:
    static AppRegistry& instance();
    
    // 注册应用
    bool registerApp(const AppInfo& app);
    
    // 注销应用
    bool unregisterApp(const QString& appId);
    
    // 更新应用
    bool updateApp(const AppInfo& app);
    
    // 获取应用列表
    QList<AppInfo> getAllApps() const;
    
    // 按分类获取
    QList<AppInfo> getAppsByCategory(AppCategory category) const;
    
    // 搜索应用
    QList<AppInfo> searchApps(const QString& keyword) const;
    
    // 获取单个应用
    AppInfo getApp(const QString& appId) const;
    
    // 保存到文件
    bool saveToFile(const QString& filePath);
    
    // 从文件加载
    bool loadFromFile(const QString& filePath);
    
signals:
    void appRegistered(const AppInfo& app);
    void appUnregistered(const QString& appId);
    void appUpdated(const AppInfo& app);
    
private:
    QMap<QString, AppInfo> m_apps;
    QString m_filePath;
};
```

#### 添加应用对话框

```cpp
// AddAppDialog.h
class AddAppDialog : public QDialog {
    Q_OBJECT
public:
    explicit AddAppDialog(QWidget* parent = nullptr);
    
    AppInfo getAppInfo() const { return m_appInfo; }
    
private slots:
    void onBrowseExe();
    void onAccept();
    
private:
    AppInfo m_appInfo;
    
    QLineEdit* m_exePathEdit;
    QLineEdit* m_nameEdit;
    QLineEdit* m_descEdit;
    QComboBox* m_categoryCombo;
    QComboBox* m_colorCombo;
    QLineEdit* m_abbreviationEdit;
    QLineEdit* m_argsEdit;
    
    void setupUI();
    void extractAppInfo(const QString& exePath);
};

// AddAppDialog.cpp
void AddAppDialog::onBrowseExe() {
    QString filePath = QFileDialog::getOpenFileName(
        this,
        tr("选择应用程序"),
        QString(),
        tr("可执行文件 (*.exe);;所有文件 (*.*)")
    );
    
    if (!filePath.isEmpty()) {
        m_exePathEdit->setText(filePath);
        extractAppInfo(filePath);
    }
}

void AddAppDialog::extractAppInfo(const QString& exePath) {
    // 使用 Windows API 读取 exe 文件版本信息
    DWORD handle = 0;
    DWORD size = GetFileVersionInfoSizeW(exePath.toStdWString().c_str(), &handle);
    
    if (size > 0) {
        std::vector<BYTE> buffer(size);
        if (GetFileVersionInfoW(exePath.toStdWString().c_str(), handle, size, buffer.data())) {
            // 读取文件描述
            LPVOID value = nullptr;
            UINT valueSize = 0;
            if (VerQueryValueW(buffer.data(), L"\\StringFileInfo\\040904b0\\FileDescription",
                               &value, &valueSize)) {
                QString description = QString::fromWCharArray((LPCWSTR)value, valueSize);
                m_nameEdit->setText(description);
                m_descEdit->setText(description);
            }
            
            // 读取产品名称
            if (VerQueryValueW(buffer.data(), L"\\StringFileInfo\\040904b0\\ProductName",
                               &value, &valueSize)) {
                QString productName = QString::fromWCharArray((LPCWSTR)value, valueSize);
                if (m_nameEdit->text().isEmpty()) {
                    m_nameEdit->setText(productName);
                }
            }
        }
    }
    
    // 如果未能提取信息，使用文件名
    if (m_nameEdit->text().isEmpty()) {
        QFileInfo fileInfo(exePath);
        m_nameEdit->setText(fileInfo.baseName());
    }
    
    // 自动生成缩写
    QString name = m_nameEdit->text();
    QString abbreviation;
    if (name.length() >= 2) {
        abbreviation = name.left(2).toUpper();
    }
    m_abbreviationEdit->setText(abbreviation);
}
```

### 4.4 内部接口设计

#### ConfigManager 类

```cpp
// ConfigManager.h
class ConfigManager : public QObject {
    Q_OBJECT
public:
    static ConfigManager& instance();
    
    // 用户配置
    UserConfig getUserConfig() const;
    void setUserConfig(const UserConfig& config);
    
    // 单项配置
    int getTheme() const;
    void setTheme(int theme);
    
    bool isSidebarCollapsed() const;
    void setSidebarCollapsed(bool collapsed);
    
    QRect getWindowBounds() const;
    void setWindowBounds(const QRect& bounds);
    
    // 保存到文件
    bool save();
    
    // 从文件加载
    bool load();
    
signals:
    void themeChanged(int theme);
    void sidebarCollapsedChanged(bool collapsed);
    void configChanged();
    
private:
    UserConfig m_config;
    QSettings* m_settings;
    
    ConfigManager();
    ~ConfigManager();
};

// ConfigManager.cpp
ConfigManager::ConfigManager() {
    // 使用 QSettings 存储配置
    m_settings = new QSettings(
        QStandardPaths::writableLocation(QStandardPaths::AppDataLocation) + "/config.ini",
        QSettings::IniFormat
    );
}

bool ConfigManager::save() {
    m_settings->beginGroup("User");
    m_settings->setValue("Theme", m_config.theme);
    m_settings->setValue("SidebarCollapsed", m_config.sidebarCollapsed);
    m_settings->setValue("SidebarWidth", m_config.sidebarWidth);
    m_settings->setValue("ActiveCategory", m_config.activeCategory);
    m_settings->setValue("LastActiveView", m_config.lastActiveView);
    m_settings->setValue("WindowBounds", m_config.windowBounds);
    m_settings->setValue("WindowMaximized", m_config.windowMaximized);
    m_settings->setValue("AutoLaunchApps", m_config.autoLaunchApps);
    m_settings->setValue("RecentAppIds", m_config.recentAppIds);
    m_settings->setValue("Favorites", m_config.favorites);
    m_settings->endGroup();
    
    m_settings->sync();
    return true;
}

bool ConfigManager::load() {
    m_settings->beginGroup("User");
    m_config.theme = m_settings->value("Theme", 0).toInt();
    m_config.sidebarCollapsed = m_settings->value("SidebarCollapsed", false).toBool();
    m_config.sidebarWidth = m_settings->value("SidebarWidth", 240).toInt();
    m_config.activeCategory = m_settings->value("ActiveCategory", "all").toString();
    m_config.lastActiveView = m_settings->value("LastActiveView", "home").toString();
    m_config.windowBounds = m_settings->value("WindowBounds", QRect(100, 100, 1280, 800)).toRect();
    m_config.windowMaximized = m_settings->value("WindowMaximized", false).toBool();
    m_config.autoLaunchApps = m_settings->value("AutoLaunchApps").toStringList();
    m_config.recentAppIds = m_settings->value("RecentAppIds").toStringList();
    m_config.favorites = m_settings->value("Favorites").toStringList();
    m_settings->endGroup();
    
    return true;
}
```

### 4.5 持久化存储

#### 存储结构

```
AppData/
└── AppHub/
    ├── config.ini              # 用户配置 (QSettings INI 格式)
    ├── apps.json               # 已注册应用列表 (JSON 格式)
    ├── layouts.json            # 窗口布局保存
    └── cache/
        ├── icons/              # 提取的应用图标缓存
        └── thumbnails/         # 应用缩略图
```

#### config.ini 示例

```ini
[User]
Theme=0
SidebarCollapsed=false
SidebarWidth=240
ActiveCategory=all
LastActiveView=home
WindowBounds=@Rect(100 100 1280 800)
WindowMaximized=false
AutoLaunchApps=
RecentAppIds=vscode, chrome, terminal
Favorites=vscode
```

#### apps.json 示例

```json
[
  {
    "id": "app-vscode-001",
    "name": "VS Code",
    "description": "代码编辑器",
    "exePath": "C:\\Users\\user\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe",
    "iconPath": "",
    "iconColor": 0,
    "abbreviation": "VS",
    "category": 0,
    "section": "常用",
    "launchArgs": [],
    "workingDir": "",
    "windowTitle": "",
    "sortOrder": 0,
    "createdAt": "2026-06-03T10:00:00",
    "updatedAt": "2026-06-03T10:00:00"
  },
  {
    "id": "app-chrome-002",
    "name": "Chrome DevTools",
    "description": "浏览器调试工具",
    "exePath": "C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe",
    "iconPath": "",
    "iconColor": 3,
    "abbreviation": "Ch",
    "category": 0,
    "section": "常用",
    "launchArgs": ["--auto-open-devtools-for-tabs"],
    "workingDir": "",
    "windowTitle": "",
    "sortOrder": 1,
    "createdAt": "2026-06-03T10:00:00",
    "updatedAt": "2026-06-03T10:00:00"
  }
]
```

---

## 5. 非功能性需求

| 指标 | 要求 |
|------|------|
| **启动时间** | AppHub 主窗口在 1s 内显示，exe 嵌入在 5s 内完成 |
| **内存占用** | AppHub 自身 < 100MB，不含嵌入的 exe 进程 |
| **稳定性** | 子进程崩溃不影响主进程，自动清理并提示 |
| **安全性** | exe 路径验证，防止注入恶意路径 |
| **兼容性** | Windows 10/11 64bit；Qt 6.5+ |
| **可扩展性** | 架构预留插件系统接口（P2 阶段实现） |
| **构建体积** | 安装包 < 50MB（不含 Qt 运行时，使用动态链接） |

---

## 6. 里程碑与版本规划

```
v0.1 Alpha (4 周)
├── 基础框架搭建 (Qt 6 + CMake)
├── 主窗口 + 侧栏 + 首页布局
├── exe 启动 & Win32 嵌入 (单应用)
└── 基本配置读写 (QSettings)

v0.5 Beta (4 周)
├── 多标签页支持
├── 工作台视图 + 嵌入工具栏
├── 应用管理 (添加/删除/编辑)
├── 搜索和分类筛选
└── 暗色/亮色主题 (QSS)

v1.0 Release (3 周)
├── 稳定性优化 & 异常处理
├── 进程生命周期完善
├── 窗口独立弹出
├── 自动更新机制 (可选)
└── 安装包打包发布 (windeployqt + Inno Setup)

v1.5 (规划中)
├── 拖拽排序
├── 应用收藏 & 最近使用
├── 分屏布局
└── 性能优化
```

---

## 附录

### A. 相关文件索引

| 文件 | 说明 |
|------|------|
| `apphub.html` | 暗色交互原型 - 可操作的 UI 原型 |
| `apphub-light.html` | 亮色交互原型 - 可操作的 UI 原型 |
| `apphub-spec.html` | 暗色视觉规范 - 组件像素级标注 |
| `apphub-spec-light.html` | 亮色视觉规范 - 组件像素级标注 |
| `design-system.html` | 暗色设计系统 - 完整设计规范文档 |
| `design-system-light.html` | 亮色设计系统 - 完整设计规范文档 |
| `launcher.html` | 简洁启动器原型 |
| `client-platform.html` | 早期多标签页原型 |

### B. 技术参考

| 资源 | 说明 |
|------|------|
| [Qt 6 官方文档](https://doc.qt.io/qt-6/) | Qt 6 完整文档 |
| [QProcess](https://doc.qt.io/qt-6/qprocess.html) | Qt 进程管理类 |
| [QWindow::fromWinId()](https://doc.qt.io/qt-6/qwindow.html#fromWinId) | 从 HWND 创建 QWindow |
| [QWidget::createWindowContainer()](https://doc.qt.io/qt-6/qwidget.html#createWindowContainer) | 将 QWindow 嵌入 QWidget |
| [Qt Style Sheets](https://doc.qt.io/qt-6/stylesheet.html) | QSS 样式系统 |
| [Windows API](https://learn.microsoft.com/windows/win32/) | Win32 API 参考 |

### C. 术语表

| 术语 | 说明 |
|------|------|
| **嵌入 (Embed)** | 通过 Win32 SetParent API 将外部 exe 窗口放置到 AppHub 窗口内部 |
| **视口 (Viewport)** | AppHub 中用于容纳嵌入窗口的容器区域 |
| **工作台 (Workspace)** | 多标签页管理嵌入应用的视图模式 |
| **首页 (Home)** | 应用启动器网格视图，用于浏览和启动应用 |
| **子进程 (Child Process)** | 被 AppHub 启动和管理的外部 exe 程序 |
| **HWND** | Windows 窗口句柄，用于标识和操作窗口 |
| **QSS** | Qt Style Sheets，Qt 的样式表系统 |
| **信号槽 (Signals & Slots)** | Qt 的事件通信机制 |

### D. CMake 配置示例

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.22)
project(AppHub VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

# 查找 Qt 6
find_package(Qt6 REQUIRED COMPONENTS 
    Core 
    Gui 
    Widgets 
)

# 源文件
set(SOURCES
    src/main.cpp
    src/core/ProcessManager.cpp
    src/core/WindowManager.cpp
    src/core/ConfigManager.cpp
    src/core/AppRegistry.cpp
    src/ui/MainWindow.cpp
    src/ui/TitleBar.cpp
    src/ui/Sidebar.cpp
    src/ui/HomePage.cpp
    src/ui/WorkspacePage.cpp
    src/ui/TabBar.cpp
    src/ui/EmbedToolbar.cpp
    src/ui/AppCard.cpp
    src/ui/AppItem.cpp
    src/utils/Win32Helper.cpp
    src/utils/IconExtractor.cpp
    src/utils/ThemeManager.cpp
)

set(HEADERS
    src/core/ProcessManager.h
    src/core/WindowManager.h
    src/core/ConfigManager.h
    src/core/AppRegistry.h
    src/ui/MainWindow.h
    src/ui/TitleBar.h
    src/ui/Sidebar.h
    src/ui/HomePage.h
    src/ui/WorkspacePage.h
    src/ui/TabBar.h
    src/ui/EmbedToolbar.h
    src/ui/AppCard.h
    src/ui/AppItem.h
    src/utils/Win32Helper.h
    src/utils/IconExtractor.h
    src/utils/ThemeManager.h
)

# 资源文件
set(RESOURCES
    src/resources/resources.qrc
)

# 创建可执行文件
add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS} ${RESOURCES})

# 链接 Qt 库
target_link_libraries(${PROJECT_NAME} PRIVATE
    Qt6::Core
    Qt6::Gui
    Qt6::Widgets
)

# Windows 特定设置
if(WIN32)
    # 链接 Windows API 库
    target_link_libraries(${PROJECT_NAME} PRIVATE
        user32
        version
    )
    
    # 设置为 Windows GUI 应用程序
    set_target_properties(${PROJECT_NAME} PROPERTIES
        WIN32_EXECUTABLE TRUE
    )
endif()

# 安装规则
install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION bin
)

# 部署 Qt 依赖（使用 windeployqt）
if(WIN32 AND CMAKE_BUILD_TYPE STREQUAL "Release")
    find_program(WINDEPLOYQT windeployqt HINTS "${Qt6_DIR}/../../../bin")
    if(WINDEPLOYQT)
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
            COMMAND ${WINDEPLOYQT} --no-translations $<TARGET_FILE:${PROJECT_NAME}>
        )
    endif()
endif()
```

---

**文档结束**
