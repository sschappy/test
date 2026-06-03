# UI_Prototype 代码整理笔记

## 1. 目录说明

目录位置：

`D:\2026\PlatformTask2026\3yue-release18\Main\UI_Prototype`

当前目录下共有 7 个 Qt/C++ 文件，整体实现的是一套“宿主程序 A 嵌入外部 exe 页面”的原型代码。  
从代码职责看，可以整理为 4 个模块：

- 程序入口
- 主窗口层
- 嵌入控制层
- 嵌入页面层

---

## 2. 文件分类总览

| 分类 | 文件 | 作用 |
|---|---|---|
| 程序入口 | `main.cpp` | Qt 程序入口，创建主窗口并决定是否进入嵌入模式 |
| 主窗口层 | `MainWindow.h` / `MainWindow.cpp` | B.exe 的主窗口类，对接嵌入控制器 |
| 嵌入控制层 | `EmbedController.h` / `EmbedController.cpp` | B 侧嵌入控制器，负责解析参数、连接 A、把自己挂到宿主窗口下 |
| 嵌入页面层 | `EmbeddedExePage.h` / `EmbeddedExePage.cpp` | A 侧页面容器，负责启动 B、监听 socket、接收 READY、同步子窗口尺寸 |

---

## 3. 整体架构关系

这套原型本质上分成 A 侧和 B 侧两个部分。

### 3.1 A 侧职责

由 `EmbeddedExePage` 承担，主要负责：

- 创建一个宿主 `QWidget`
- 启动 B.exe
- 启动 `QLocalServer`
- 把宿主窗口句柄通过命令行传给 B
- 等待 B 回连并发送 `READY`
- 拿到 B 的 `winId`
- 后续在 resize 时同步子窗口大小

### 3.2 B 侧职责

由 `MainWindow + EmbedController` 承担，主要负责：

- 识别自己是否以“被嵌入模式”启动
- 根据命令行拿到 `serverName / sessionId / hostWinId`
- 主动连接 A 的本地 socket
- 把自己的主窗口改造成子窗口
- 挂到 A 的宿主窗口下
- 在窗口 show 之后通知 A：`READY:sessionId:pid:winId`

### 3.3 通信链路

完整流程如下：

1. A 创建 `EmbeddedExePage`
2. A 启动本地 `QLocalServer`
3. A 启动 B.exe，并追加 3 个嵌入参数
4. B 启动后解析参数
5. B 连接 A 的本地 server
6. B 调整自身窗口样式并 `SetParent`
7. B 在 `showEvent` 后发送 `READY`
8. A 收到 `READY`，记录 `winId`
9. A 后续通过 `MoveWindow` 维护 B 子窗口尺寸

这套设计的关键点是：

- A 负责“承载”
- B 负责“把自己变成子窗口”
- A/B 通过 `QLocalServer + QLocalSocket` 做握手同步

---

## 4. 分文件整理

## 4.1 程序入口

### 文件：`main.cpp`

### 核心作用

这是 B.exe 的程序入口文件。

### 它负责什么

- 创建 `QApplication`
- 创建 `MainWindow`
- 在 `show()` 前调用 `initEmbedMode()`
- 决定当前是否以嵌入模式运行

### 关键阅读点

#### 1. 为什么要在 `show()` 之前调用 `initEmbedMode()`

因为 B 要先判断自己是不是被 A 拉起的嵌入实例。  
如果是嵌入模式，就要先完成：

- 参数解析
- 连接宿主 A
- 设置窗口父子关系

这些动作都应该发生在窗口正式展示前。

#### 2. 这个文件本身不复杂，但它决定了启动顺序

可以把它理解成：

- 入口层只负责组装启动流程
- 具体嵌入逻辑不写在这里

### 笔记注释

- 这是一个典型的“入口薄、逻辑下沉”的写法。
- 如果后续 B 有普通模式和嵌入模式两种运行方式，这里就是模式切换入口。

---

## 4.2 主窗口层

### 文件：`MainWindow.h`

### 核心作用

定义 B.exe 的主窗口类。

### 它负责什么

- 对外提供 `initEmbedMode()`
- 在 `showEvent()` 中发送 READY
- 持有 `EmbedController`

### 关键成员

#### `EmbedController m_embedController`

作用：

- 承接 B 侧所有嵌入模式逻辑

意义：

- 避免把参数解析、socket 连接、Win32 嵌入细节全部写进 `MainWindow`

#### `bool m_embedInitialized`

作用：

- 标记嵌入初始化是否完成

意义：

- 只有当嵌入准备完毕，`showEvent()` 才会发送 `READY`

### 笔记注释

- `MainWindow` 在这里更像“界面壳层”。
- 真正的嵌入业务被转交给 `EmbedController`，这是 UI 层和业务逻辑分离的一个小型体现。

---

### 文件：`MainWindow.cpp`

### 核心作用

实现主窗口和嵌入控制器之间的配合。

### 关键逻辑整理

#### 1. 构造函数

主要做两件事：

- 初始化基类
- 设置普通独立运行时的默认窗口尺寸

这说明：

- 这个 B.exe 不是只能被嵌入
- 它也支持独立运行

#### 2. `initEmbedMode(const QStringList &args)`

这是主窗口切入嵌入模式的核心函数。

执行顺序：

1. 调 `m_embedController.initFromArguments(args)`
2. 如果不是嵌入模式，直接返回
3. 如果是嵌入模式，则连接 A 的本地 server
4. 连接成功后，调用 `attachMainWindow(winId())`
5. 标记 `m_embedInitialized = true`

可以看出这里的职责边界很清楚：

- `MainWindow` 只负责调度
- 真正怎么解析、怎么连接、怎么嵌入，都不在这里实现

#### 3. `showEvent(QShowEvent *event)`

这里的关键不是重写 `showEvent` 本身，而是：

- 只有窗口真正 show 出来后，才通知 A：READY

原因是：

- 这时 `winId` 才处于稳定可用状态
- A 收到 READY 后就会马上拿这个 `winId` 去做尺寸同步

### 笔记注释

- `showEvent` 在这里承担的是“嵌入完成通知时机控制”。
- 这是一个典型的生命周期节点：初始化完成不等于可显示完成。

---

## 4.3 嵌入控制层

### 文件：`EmbedController.h`

### 核心作用

这是 B 侧嵌入逻辑的核心控制类。

### 它负责什么

- 解析命令行中的嵌入参数
- 保存嵌入会话上下文
- 连接 A 的 `QLocalServer`
- 把 B 的主窗口改成子窗口
- 向 A 发送 READY

### 关键成员整理

#### `bool m_embedMode`

表示当前是否运行在嵌入模式。

#### `QString m_serverName`

表示 A 提供给 B 的本地 socket 服务名。

#### `QString m_sessionId`

表示当前嵌入会话唯一标识。  
它用于确保当前 READY 消息属于正确的嵌入实例。

#### `WId m_hostWinId`

表示 A 侧宿主控件窗口句柄。  
B 要把自己的主窗口挂到它下面。

#### `bool m_readySent`

用于避免重复发送 READY。

#### `QLocalSocket *m_socket`

用于 B 主动连接 A，并向 A 回发 READY。

### 笔记注释

- 这个类本质上是“B 侧嵌入会话控制器”。
- 它不是 UI 类，更接近流程控制 / 进程接入控制类。

---

### 文件：`EmbedController.cpp`

### 核心作用

实现 B 侧嵌入流程的完整细节。

### 关键函数整理

#### 1. `initFromArguments()`

作用：

- 从命令行里读取：
  - `--embed-server=`
  - `--embed-session=`
  - `--embed-host=`

只有这几个值都合法时，才会把 `m_embedMode` 置为 `true`。

这一步说明：

- 嵌入模式不是靠某个布尔开关硬编码
- 而是靠启动参数动态识别

#### 2. `connectToHost()`

作用：

- 用 `m_serverName` 连接 A 的本地 server

关键点：

- 这里用了 `waitForConnected(5000)` 做同步等待
- 如果连接失败，B 就不会继续嵌入流程

这说明：

- 当前原型更强调流程直接、时序简单
- 不是完全异步式写法

#### 3. `attachMainWindow(WId selfWinId)`

这是 B 变成 A 子窗口的关键函数。

主要步骤：

1. 拿到 `selfWinId` 和 `m_hostWinId`
2. 用 `GetWindowLongPtr` 读取当前窗口样式
3. 去掉：
   - `WS_CAPTION`
   - `WS_THICKFRAME`
   - `WS_POPUP`
4. 加上 `WS_CHILD`
5. 去掉扩展样式中的 `WS_EX_APPWINDOW`
6. `SetParent(hSelf, hHost)`
7. `ShowWindow(hSelf, SW_SHOW)`

这段逻辑的本质是：

- 先把“独立顶层窗口”改造成“子窗口”
- 再挂到 A 的宿主窗口下面

这是整个嵌入能力最核心的 Win32 层动作。

#### 4. `notifyReady(WId selfWinId)`

作用：

- 按约定协议向 A 发消息：

`READY:sessionId:pid:winId`

这条消息的意义：

- `sessionId`：标识是哪次嵌入会话
- `pid`：标识是哪个 B 进程
- `winId`：告诉 A 以后该操作哪个子窗口

#### 5. `valueOf()`

作用：

- 从命令行参数列表里取指定前缀的值

这属于小工具函数，职责很单一。

### 笔记注释

- `EmbedController` 是这个原型中“最像业务流程控制器”的类。
- 它把嵌入模式的识别、连接、挂接、通知串成了一条完整链路。
- 如果以后 B 要支持更多嵌入协议，这里就是第一扩展点。

---

## 4.4 嵌入页面层

### 文件：`EmbeddedExePage.h`

### 核心作用

这是 A 侧单个嵌入页签 / 页面容器的核心类。

### 结构说明

一个 `EmbeddedExePage` 实例，对应一个完整的 B.exe 嵌入会话。

### 主要职责

- 创建宿主区域
- 启动 B.exe
- 创建本地 server
- 接收 B 的回连
- 解析 READY
- 保存子窗口句柄
- 在 resize 时同步子窗口大小
- 在关闭时释放进程和连接

### 辅助数据结构：`EmbeddedExeConfig`

这个结构体描述一个待嵌入应用的基础信息：

- `appId`
- `title`
- `program`
- `arguments`

它的价值在于：

- 把“嵌入哪个 exe”的配置从页面逻辑中分离出来
- 为后续多个工具页签做准备

### 关键成员整理

#### `QWidget *m_hostWidget`

真正承载 B 子窗口的宿主控件。  
它必须具备原生窗口句柄。

#### `QProcess *m_process`

负责启动和管理 B.exe 进程。

#### `QLocalServer *m_server`

A 侧本地 server，等待 B 回连。

#### `QLocalSocket *m_socket`

表示当前已经接入的那条连接。

#### `QString m_serverName`

当前这次嵌入会话使用的 server 名。

#### `QString m_sessionId`

当前嵌入会话唯一标识。

#### `WId m_embeddedWinId`

B.exe 主窗口句柄。

#### `bool m_ready`

表示当前是否已接收到 READY，是否已经真正可显示。

### 笔记注释

- 这个类是 A 侧的“页面层 + 会话层”合体实现。
- 如果以后做大型全家桶，可考虑把“会话控制”再拆出去，避免页面类过重。

---

### 文件：`EmbeddedExePage.cpp`

### 核心作用

这是当前原型里逻辑最多的文件，也是 A 侧嵌入链路的核心实现。

### 关键逻辑整理

#### 1. 构造函数

构造时主要做了几件事：

1. 保存配置
2. 创建宿主控件 `m_hostWidget`
3. 创建 `QProcess`
4. 创建 `QLocalServer`
5. 给宿主控件加 `Qt::WA_NativeWindow`
6. 建布局，把宿主控件铺满页面
7. 连接：
   - `QProcess::finished`
   - `QLocalServer::newConnection`

最关键的一个点是：

#### `m_hostWidget->setAttribute(Qt::WA_NativeWindow);`

这句非常重要。  
因为如果没有原生窗口句柄，B 就没法通过 `SetParent` 挂进来。

#### 2. `start()`

这是 A 侧启动一轮嵌入会话的入口函数。

执行顺序：

1. `stop()` 清理旧会话
2. 生成新的 `sessionId`
3. `startServer()`
4. 组装启动参数
5. 调 `m_process->start(...)`
6. 等待子进程启动成功

这里的 3 个关键嵌入参数是：

- `--embed-server=...`
- `--embed-session=...`
- `--embed-host=...`

这 3 个参数共同把 A 的嵌入上下文传给 B。

#### 3. `stop()`

作用：

- 停掉当前嵌入会话并清理资源

执行顺序：

1. `clearConnection()`
2. 清理 `m_embeddedWinId`
3. 重置 `m_ready`
4. 如果进程还活着，先 `terminate()`
5. 超时后再 `kill()`
6. `stopServer()`

这说明这个类对生命周期管理是有意识的：

- 不是只负责启动
- 也负责完整收尾

#### 4. `onNewConnection()`

作用：

- 处理 B 回连到 A 的本地 server

关键逻辑：

- 如果已经有一条主连接，则后来的连接都视为多余连接并主动断掉
- 第一条有效连接才保存为 `m_socket`
- 然后绑定：
  - `readyRead`
  - `disconnected`

这意味着：

- 一个 `EmbeddedExePage` 设计上只接受一条主通信连接

#### 5. `onSocketReadyRead()`

这是 READY 协议解析核心。

处理流程：

1. 读取 socket 文本
2. 按 `:` 分割
3. 检查是否是 `READY` 协议
4. 解析 `sessionId / pid / winId`
5. 校验 `sessionId` 是否匹配当前页面
6. 校验 `pid` 是否匹配当前启动的 `QProcess`
7. 保存 `m_embeddedWinId`
8. 设置 `m_ready = true`
9. 立即调用 `updateEmbeddedGeometry()`

这里的两个校验非常重要：

#### `sessionId` 校验

防止串页签、串会话。

#### `pid` 校验

防止误接收其他进程发来的 READY。

这两个校验让当前原型在多实例场景下更稳一些。

#### 6. `onProcessFinished()`

作用：

- 当 B 进程结束时，同步清理当前嵌入状态

会做：

- `m_ready = false`
- `m_embeddedWinId = 0`
- `clearConnection()`
- `stopServer()`

这属于典型的进程生命周期收尾。

#### 7. `startServer()` / `stopServer()`

这两个函数负责本地 server 生命周期。

特点：

- 会先清理旧 server
- 会生成唯一 `serverName`
- 会调用 `QLocalServer::removeServer(...)` 防御残留

这说明作者考虑到了：

- 上一次异常退出
- 名称残留占用

#### 8. `buildServerName()`

用于构造唯一 server 名，组成元素包括：

- `appId`
- A 进程 pid
- 毫秒时间戳

这是一种简单直接的唯一性方案。

#### 9. `resizeEvent()` + `updateEmbeddedGeometry()`

这两段共同负责子窗口尺寸同步。

流程是：

- 页面尺寸变化时进入 `resizeEvent()`
- 然后调用 `updateEmbeddedGeometry()`
- `updateEmbeddedGeometry()` 内部用 `MoveWindow(...)`
- 将 B 的子窗口铺满 `m_hostWidget`

这里的前置条件是：

- `m_ready == true`
- `m_embeddedWinId` 有效

这套逻辑说明：

- A 不负责让 B 自己布局
- A 只负责把 B 的顶层承载区域尺寸同步过去

#### 10. `clearConnection()`

作用：

- 统一清理 `m_socket`

会做：

- 断开信号连接
- 断开 socket
- `deleteLater()`
- 指针置空

这属于典型的“统一清理出口”设计，便于避免清理逻辑散落。

### 笔记注释

- `EmbeddedExePage.cpp` 是当前原型最值得重点阅读的文件。
- 如果你想理解 A.exe 承载 B.exe 的主链路，这个文件是第一阅读入口。
- 如果后续要做 tab 化、多工具接入、多实例管理，这个类会是最先演进的核心点。

---

## 5. 建议的阅读顺序

如果目的是“看懂这套原型怎么工作”，建议按下面顺序看：

### 第一步：先看 `EmbeddedExePage.h`

原因：

- 先建立 A 侧整体职责认知
- 先知道有哪些成员和槽函数

### 第二步：再看 `EmbeddedExePage.cpp`

重点看：

- `start()`
- `onNewConnection()`
- `onSocketReadyRead()`
- `updateEmbeddedGeometry()`
- `stop()`

这是主链路。

### 第三步：看 `EmbedController.h`

原因：

- 建立 B 侧职责认知
- 知道 B 负责哪些动作

### 第四步：看 `EmbedController.cpp`

重点看：

- `initFromArguments()`
- `connectToHost()`
- `attachMainWindow()`
- `notifyReady()`

这是 B 侧嵌入链路。

### 第五步：最后看 `MainWindow.cpp` 和 `main.cpp`

原因：

- 这两个文件更像调度入口
- 主要是把 B 侧控制器串起来

---

## 6. 从架构视角看，这套原型体现了什么思路

### 6.1 UI 层和嵌入逻辑做了初步分离

体现位置：

- `MainWindow` 不直接实现嵌入细节
- 通过 `EmbedController` 承接 B 侧嵌入逻辑

### 6.2 会话有唯一标识

体现位置：

- `sessionId`

意义：

- 为多页签、多实例场景预留了基本能力

### 6.3 生命周期是成套考虑的

体现位置：

- `start()`
- `stop()`
- `onProcessFinished()`
- `clearConnection()`
- `startServer() / stopServer()`

意义：

- 不是只考虑“怎么启动”，也考虑“怎么收尾”

### 6.4 A/B 边界较清楚

边界划分：

- A 负责宿主、进程启动、监听、尺寸同步
- B 负责连接宿主、改窗口样式、发送 READY

这就是比较实际的模块边界。

### 6.5 扩展性已经有基础，但还没完全平台化

已有基础：

- `EmbeddedExeConfig`
- `appId`
- `title`
- `arguments`

还未完全平台化的点：

- 当前 `EmbeddedExePage` 仍然同时承担了页面和会话控制职责
- 还没有独立的注册表 / SessionManager / ToolManager

---

## 7. 可以补充到你学习笔记里的结论

### 结论 1

这套代码本质上实现的是：

> A 创建宿主环境，B 主动把自己改造成子窗口并回报 READY，随后 A 接管布局同步。

### 结论 2

真正的嵌入核心不在“启动 B.exe”，而在两件事：

- `SetParent` 把 B 挂到 A 下
- READY 握手让 A 知道该控制哪个窗口

### 结论 3

如果以后要做全家桶式 A.exe，这个原型后续大概率会往下面这个方向演进：

- `EmbeddedExePage`：页面容器
- `EmbedSession`：单次嵌入会话控制
- `ToolRegistry`：工具注册信息
- `SessionManager`：多实例管理
- `MainWindow`：tab 管理和页面切换

### 结论 4

从学习角度，这个目录最值得你反复看的不是 UI，而是下面 3 个点：

- 进程生命周期
- socket 握手协议
- Win32 窗口父子关系改造

---

## 8. 本目录文件对应的学习标签

为了后续复习方便，可以给这些文件打上标签：

| 文件 | 学习标签 |
|---|---|
| `main.cpp` | 程序入口、启动顺序、模式切换 |
| `MainWindow.h/.cpp` | 壳层窗口、事件时机、控制器协作 |
| `EmbedController.h/.cpp` | 嵌入协议、参数解析、socket 连接、Win32 窗口挂接 |
| `EmbeddedExePage.h/.cpp` | 会话管理、进程管理、server 监听、READY 协议、子窗口布局同步 |

---

## 9. 一句话总结

`UI_Prototype` 这组代码不是普通页面原型，而是一套“外部 exe 嵌入宿主 Qt 页面”的最小闭环原型。  
它最有价值的地方，不是界面，而是把下面这条链路打通了：

> 启动子进程 -> 建立本地握手 -> 改造窗口关系 -> 完成嵌入 -> 维持尺寸同步 -> 生命周期收尾

