# UI_Prototype 原始文件汇总笔记

## 说明

本笔记用于整理目录：

`D:\2026\PlatformTask2026\3yue-release18\Main\UI_Prototype`

中的源文件内容。

本次整理遵循两个原则：

1. 保留原始文件内容，不修改原文件
2. 仅在 Markdown 中做分类汇总，方便统一查看和学习

---

## 一、程序入口

### 1. `main.cpp`

```cpp
#include <QApplication>      // qianwen：Qt Widgets 程序入口所需的 QApplication
#include <QCoreApplication>  // qianwen：用于读取命令行参数列表
#include "MainWindow.h"      // qianwen：引入 B.exe 的主窗口类

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);                    // qianwen：创建 QApplication，初始化图形界面环境并接管命令行参数
    MainWindow w;                                    // qianwen：创建 B.exe 主窗口对象
    w.initEmbedMode(QCoreApplication::arguments());  // qianwen：在 show 前先决定是否进入嵌入模式，并完成相关初始化
    w.show();                                        // qianwen：显示主窗口；普通模式下是顶层窗口，嵌入模式下已被挂到 A 中
    return app.exec();                               // qianwen：进入 Qt 事件循环，处理窗口、socket 和系统消息
}
```

---

## 二、主窗口层

### 2. `MainWindow.h`

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>          // qianwen：提供 QMainWindow 主窗口基类
#include "EmbedController.h"    // qianwen：引入嵌入控制器，把嵌入逻辑和主窗口业务逻辑隔离开

/*
 * qianwen：B.exe 的主窗口类。
 * qianwen：这个类可以同时支持两种模式：
 * qianwen：1. 普通独立程序模式
 * qianwen：2. 嵌入到 A.exe 中的子窗口模式
 */
class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    // qianwen：构造函数，负责主窗口基础初始化。
    explicit MainWindow(QWidget *parent = nullptr);

    // qianwen：析构函数，当前示例没有额外释放逻辑。
    ~MainWindow() override;

    // qianwen：根据命令行参数决定是否进入嵌入模式，并在需要时连接 A 和挂接窗口。
    void initEmbedMode(const QStringList &args);

protected:
    // qianwen：重写 showEvent，在窗口真正 show 出来后通知 A READY。
    void showEvent(QShowEvent *event) override;

private:
    EmbedController m_embedController; // qianwen：专门管理嵌入模式逻辑的控制器对象
    bool m_embedInitialized;           // qianwen：是否已经完成嵌入初始化，决定 showEvent 中是否发送 READY
};

#endif // MAINWINDOW_H
```

### 3. `MainWindow.cpp`

```cpp
#include "MainWindow.h" // qianwen：引入主窗口类声明
#include <QShowEvent>   // qianwen：引入 QShowEvent 类型，供 showEvent 重写使用

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)       // qianwen：初始化主窗口基类
    , m_embedInitialized(false) // qianwen：默认还没有完成嵌入初始化
{
    resize(1000, 700);          // qianwen：设置普通独立运行时的默认窗口大小
}

MainWindow::~MainWindow()
{
    // qianwen：当前示例没有额外析构逻辑。
}

void MainWindow::initEmbedMode(const QStringList &args)
{
    if (!m_embedController.initFromArguments(args)) { // qianwen：先让控制器解析命令行参数；如果不是嵌入模式则直接返回
        return;                                       // qianwen：保持普通独立程序行为
    }

    if (!m_embedController.connectToHost()) {         // qianwen：如果是嵌入模式，则先连接 A 的本地 server
        return;                                       // qianwen：连接失败就不继续走嵌入流程
    }

    m_embedController.attachMainWindow(winId());      // qianwen：拿当前主窗口 winId，让控制器把自己挂到 A 的宿主窗口下面
    m_embedInitialized = true;                        // qianwen：标记嵌入初始化已经完成，showEvent 时可安全发送 READY
}

void MainWindow::showEvent(QShowEvent *event)
{
    QMainWindow::showEvent(event);                    // qianwen：先执行基类默认的显示逻辑

    if (m_embedInitialized) {                         // qianwen：只有嵌入初始化完成后，才向 A 发送 READY
        m_embedController.notifyReady(winId());       // qianwen：把当前主窗口 winId 一并带给 A
    }
}
```

---

## 三、嵌入控制层

### 4. `EmbedController.h`

```cpp
#ifndef EMBEDCONTROLLER_H
#define EMBEDCONTROLLER_H

#include <QObject>          // qianwen：提供 QObject 基类，方便用 Qt 对象树托管 socket
#include <QLocalSocket>     // qianwen：提供本地 socket，B 用它主动连接 A
#include <QStringList>      // qianwen：提供 QStringList，方便处理命令行参数

/*
 * qianwen：B 侧嵌入控制器。
 * qianwen：这个类把“嵌入模式”相关逻辑从主窗口里剥离出来，职责包括：
 * qianwen：1. 解析 embed 参数
 * qianwen：2. 连接 A 的本地 server
 * qianwen：3. 把 B 的主窗口改成 A 宿主窗口的子窗口
 * qianwen：4. 向 A 发送 READY
 */
class EmbedController : public QObject
{
    Q_OBJECT

public:
    // qianwen：构造函数，只做成员初始化。
    explicit EmbedController(QObject *parent = nullptr);

    // qianwen：从命令行参数中识别嵌入模式所需字段，并初始化内部状态。
    bool initFromArguments(const QStringList &args);

    // qianwen：返回当前是否处于嵌入模式。
    bool isEmbedMode() const;

    // qianwen：连接到 A 的本地 server。
    bool connectToHost();

    // qianwen：把 B 的主窗口附着到 A 的宿主窗口下面。
    void attachMainWindow(WId selfWinId);

    // qianwen：向 A 发送 READY 消息，通知当前窗口已经嵌入完成。
    void notifyReady(WId selfWinId);

private:
    // qianwen：从参数列表中提取某个前缀对应的值。
    QString valueOf(const QStringList &args, const QString &prefix) const;

private:
    bool m_embedMode;           // qianwen：当前是否已经进入嵌入模式
    QString m_serverName;       // qianwen：A 传给 B 的本地 server 名称
    QString m_sessionId;        // qianwen：A 为当前嵌入实例分配的会话唯一标识
    WId m_hostWinId;            // qianwen：A 宿主窗口句柄，B 会把自己挂到它下面
    bool m_readySent;           // qianwen：是否已经发送过 READY，防止重复通知
    QLocalSocket *m_socket;     // qianwen：B 到 A 的通信 socket
};

#endif // EMBEDCONTROLLER_H
```

### 5. `EmbedController.cpp`

```cpp
#include "EmbedController.h"    // qianwen：引入当前类声明

#include <QCoreApplication>     // qianwen：用于读取当前 B 进程 pid
#include <QDebug>               // qianwen：用于输出调试日志
#include <windows.h>            // qianwen：引入 Win32 API，用于 SetParent 和窗口样式修改

EmbedController::EmbedController(QObject *parent)
    : QObject(parent)                 // qianwen：初始化 QObject 基类
    , m_embedMode(false)              // qianwen：默认不是嵌入模式，只有解析成功后才会变成 true
    , m_hostWinId(0)                  // qianwen：默认宿主窗口句柄无效
    , m_readySent(false)              // qianwen：默认还没有发送 READY
    , m_socket(new QLocalSocket(this))// qianwen：创建本地 socket 对象，并挂到当前对象树下
{
    // qianwen：构造函数不主动 connectToServer，因为只有在识别出嵌入模式后才应该连接 A。
}

bool EmbedController::initFromArguments(const QStringList &args)
{
    m_serverName = valueOf(args, "--embed-server=");         // qianwen：提取 A 传来的 server 名称
    m_sessionId = valueOf(args, "--embed-session=");         // qianwen：提取当前会话唯一标识
    const QString hostValue = valueOf(args, "--embed-host=");// qianwen：提取宿主窗口句柄的字符串值

    bool ok = false;                                         // qianwen：用于接收句柄字符串转整数是否成功
    const qulonglong wid = hostValue.toULongLong(&ok);       // qianwen：把宿主窗口句柄字符串转成整数

    if (!m_serverName.isEmpty() && !m_sessionId.isEmpty() && ok) { // qianwen：只有关键字段齐全且句柄可解析时才进入嵌入模式
        m_hostWinId = static_cast<WId>(wid);                 // qianwen：保存宿主窗口句柄
        m_embedMode = true;                                  // qianwen：标记当前处于嵌入模式
    }

    return m_embedMode;                                      // qianwen：返回是否成功识别成嵌入模式
}

bool EmbedController::isEmbedMode() const
{
    return m_embedMode;                                      // qianwen：直接返回当前嵌入模式标识
}

bool EmbedController::connectToHost()
{
    if (!m_embedMode) {                                      // qianwen：如果不是嵌入模式，就不应该连接 A
        return false;
    }

    m_socket->connectToServer(m_serverName);                 // qianwen：按 A 提供的 serverName 主动连接本地 server
    if (!m_socket->waitForConnected(5000)) {                 // qianwen：同步等待最多 5 秒，确认连接是否建立成功
        qWarning() << "connect embed server failed:"         // qianwen：连接失败时输出诊断信息
                   << m_socket->errorString();               // qianwen：打印 QLocalSocket 提供的错误文本
        return false;                                        // qianwen：返回连接失败
    }

    return true;                                             // qianwen：连接成功
}

void EmbedController::attachMainWindow(WId selfWinId)
{
    if (!m_embedMode) {                                      // qianwen：不是嵌入模式就不做任何窗口挂接
        return;
    }

    if (!selfWinId) {                                        // qianwen：如果 B 自己的主窗口句柄无效，则无法继续
        return;
    }

    if (!m_hostWinId) {                                      // qianwen：如果宿主窗口句柄无效，也无法继续
        return;
    }

    HWND hSelf = reinterpret_cast<HWND>(selfWinId);          // qianwen：把 B 自己的 winId 转成 Win32 HWND
    HWND hHost = reinterpret_cast<HWND>(m_hostWinId);        // qianwen：把 A 的宿主 winId 转成 Win32 HWND

    LONG_PTR style = GetWindowLongPtr(hSelf, GWL_STYLE);     // qianwen：读取当前主窗口的普通样式
    style &= ~WS_CAPTION;                                    // qianwen：去掉标题栏
    style &= ~WS_THICKFRAME;                                 // qianwen：去掉可调大小边框
    style &= ~WS_POPUP;                                      // qianwen：去掉 popup 顶层弹出样式
    style |= WS_CHILD;                                       // qianwen：增加子窗口样式，表明它将挂到父窗口下面
    SetWindowLongPtr(hSelf, GWL_STYLE, style);               // qianwen：把修改后的普通样式写回主窗口

    LONG_PTR exStyle = GetWindowLongPtr(hSelf, GWL_EXSTYLE); // qianwen：读取扩展样式
    exStyle &= ~WS_EX_APPWINDOW;                             // qianwen：去掉 APPWINDOW 标记，避免继续按独立应用窗口显示
    SetWindowLongPtr(hSelf, GWL_EXSTYLE, exStyle);           // qianwen：把修改后的扩展样式写回主窗口

    SetParent(hSelf, hHost);                                 // qianwen：把 B 的主窗口真正挂到 A 的宿主窗口下面
    ShowWindow(hSelf, SW_SHOW);                              // qianwen：确保嵌入后的主窗口保持可见
}

void EmbedController::notifyReady(WId selfWinId)
{
    if (!m_embedMode) {                                      // qianwen：不是嵌入模式则不发 READY
        return;
    }

    if (m_readySent) {                                       // qianwen：如果已经发过 READY，就不再重复发
        return;
    }

    if (!m_socket) {                                         // qianwen：如果 socket 对象不存在，则无法发送消息
        return;
    }

    if (m_socket->state() != QLocalSocket::ConnectedState) { // qianwen：只有连接状态正常时才允许发送 READY
        return;
    }

    const QString msg = QString("READY:%1:%2:%3\n")          // qianwen：按协议拼接 READY 消息
        .arg(m_sessionId)                                    // qianwen：第 2 段放 sessionId，用于 A 校验归属
        .arg(QCoreApplication::applicationPid())             // qianwen：第 3 段放 B 的进程 pid，供 A 校验进程身份
        .arg(static_cast<qulonglong>(selfWinId));            // qianwen：第 4 段放 B 主窗口 winId，供 A 做尺寸同步

    m_socket->write(msg.toUtf8());                           // qianwen：把 READY 文本写入 socket 缓冲区
    m_socket->flush();                                       // qianwen：尽快把缓冲推送出去
    m_socket->waitForBytesWritten(3000);                     // qianwen：同步等待最多 3 秒，尽量确保消息已写出

    m_readySent = true;                                      // qianwen：标记 READY 已发送，避免重复通知
}

QString EmbedController::valueOf(const QStringList &args, const QString &prefix) const
{
    for (const QString &arg : args) {                        // qianwen：遍历全部命令行参数
        if (arg.startsWith(prefix)) {                        // qianwen：如果当前参数以前缀开头，则表示找到目标参数
            return arg.mid(prefix.size());                   // qianwen：裁掉前缀，只返回真正的值部分
        }
    }

    return QString();                                        // qianwen：未找到时返回空字符串
}
```

---

## 四、嵌入页面层

### 6. `EmbeddedExePage.h`

```cpp
#ifndef EMBEDDEDEXEPAGE_H
#define EMBEDDEDEXEPAGE_H

#include <QWidget>          // qianwen：提供 QWidget 基类，A 侧嵌入页面本身就是一个 QWidget 页面
#include <QProcess>         // qianwen：提供 QProcess，用来启动和管理 B.exe 子进程
#include <QLocalServer>     // qianwen：提供 QLocalServer，用来在 A 侧建立本地 socket 服务
#include <QLocalSocket>     // qianwen：提供 QLocalSocket，用来接收 B.exe 回连后的通信连接

/*
 * qianwen：单个被嵌入程序的配置结构。
 * qianwen：以后如果 A 是全家桶，这个结构可以描述不同 exe 的基础信息。
 */
struct EmbeddedExeConfig
{
    QString appId;              // qianwen：应用唯一标识，例如 "viewer"、"measure"、"editor"
    QString title;              // qianwen：页面标题或 tab 标题，界面层可直接使用
    QString program;            // qianwen：要启动的 exe 的完整路径
    QStringList arguments;      // qianwen：启动 exe 时附带的业务参数，不包含 embed 专用参数
};

/*
 * qianwen：A 侧单个嵌入页面类。
 * qianwen：一个页面实例对应一个被嵌入 exe 的完整会话。
 * qianwen：职责包括：
 * qianwen：1. 创建宿主区域
 * qianwen：2. 启动 B.exe
 * qianwen：3. 启动本地 server 等待 B 回连
 * qianwen：4. 接收 READY 消息
 * qianwen：5. 在 resize 时同步子窗口大小
 * qianwen：6. 在退出时清理进程和通信资源
 */
class EmbeddedExePage : public QWidget
{
    Q_OBJECT

public:
    // qianwen：构造函数，接收当前页面对应的被嵌入应用配置。
    explicit EmbeddedExePage(const EmbeddedExeConfig &config, QWidget *parent = nullptr);

    // qianwen：析构函数，内部会调用 stop 做资源清理。
    ~EmbeddedExePage() override;

    // qianwen：启动当前页面对应的被嵌入应用。
    bool start();

    // qianwen：停止当前嵌入会话，并清理进程、socket、server 等资源。
    void stop();

    // qianwen：返回当前页面是否已经收到 READY 并进入可正常显示状态。
    bool isReady() const;

    // qianwen：返回当前页面对应的应用标识。
    QString appId() const;

protected:
    // qianwen：重写 resizeEvent，在页面尺寸变化时同步调整 B.exe 子窗口大小。
    void resizeEvent(QResizeEvent *event) override;

private slots:
    // qianwen：当有新的客户端连到 A 的 QLocalServer 时触发。
    void onNewConnection();

    // qianwen：当当前连接上有数据可读时触发，用来接收 READY 消息。
    void onSocketReadyRead();

    // qianwen：当 B.exe 进程退出时触发，用来同步清理嵌入状态。
    void onProcessFinished(int exitCode, QProcess::ExitStatus exitStatus);

private:
    // qianwen：启动本地 server，供 B.exe 连接回来。
    bool startServer();

    // qianwen：停止本地 server，并释放占用的 serverName。
    void stopServer();

    // qianwen：生成当前会话使用的唯一 serverName，避免多实例冲突。
    QString buildServerName() const;

    // qianwen：根据宿主区域大小更新 B.exe 子窗口大小。
    void updateEmbeddedGeometry();

    // qianwen：清理当前 socket 连接对象和连接状态。
    void clearConnection();

private:
    EmbeddedExeConfig m_config; // qianwen：当前页面的应用配置，描述要启动哪个 exe
    QWidget *m_hostWidget;      // qianwen：真正承载 B.exe 子窗口的宿主控件，必须具备原生窗口句柄
    QProcess *m_process;        // qianwen：负责启动和管理 B.exe 的子进程对象
    QLocalServer *m_server;     // qianwen：A 侧本地 socket 服务端对象
    QLocalSocket *m_socket;     // qianwen：当前已经接受到的 B.exe 通信连接对象
    QString m_serverName;       // qianwen：当前 server 的名字，会通过命令行参数传给 B.exe
    QString m_sessionId;        // qianwen：当前嵌入会话唯一标识，便于多页签和多实例区分
    WId m_embeddedWinId;        // qianwen：B.exe 主窗口的 winId/句柄，收到 READY 后记录
    bool m_ready;               // qianwen：是否已经完成嵌入并可以正常显示
};

#endif // EMBEDDEDEXEPAGE_H
```

### 7. `EmbeddedExePage.cpp`

```cpp
#include "EmbeddedExePage.h"   // qianwen：引入当前类声明

#include <QVBoxLayout>         // qianwen：用于给页面创建简单垂直布局，把宿主控件铺满页面
#include <QCoreApplication>    // qianwen：用于获取当前 A 进程 pid
#include <QDateTime>           // qianwen：用于生成时间戳，构造唯一的 sessionId 和 serverName
#include <QResizeEvent>        // qianwen：提供 QResizeEvent 类型，供 resizeEvent 重写使用
#include <QDebug>              // qianwen：用于输出调试信息

#include <windows.h>           // qianwen：引入 Win32 API，后续需要用 MoveWindow 调整子窗口大小

EmbeddedExePage::EmbeddedExePage(const EmbeddedExeConfig &config, QWidget *parent)
    : QWidget(parent)                  // qianwen：初始化 QWidget 基类，让当前对象成为一个标准页面控件
    , m_config(config)                 // qianwen：保存传入的应用配置，后续 start 时直接使用
    , m_hostWidget(new QWidget(this))  // qianwen：创建宿主控件，B.exe 子窗口最终会挂到这个控件上
    , m_process(new QProcess(this))    // qianwen：创建子进程对象，用来启动 B.exe
    , m_server(new QLocalServer(this)) // qianwen：创建本地 server 对象，等 B.exe 回连
    , m_socket(nullptr)                // qianwen：当前还没有 socket 连接，因此初始化为空
    , m_embeddedWinId(0)               // qianwen：当前还没有收到 B 的主窗口句柄，因此初始化为 0
    , m_ready(false)                   // qianwen：当前还没有完成嵌入，因此初始化为 false
{
    m_hostWidget->setAttribute(Qt::WA_NativeWindow); // qianwen：强制宿主控件拥有原生窗口句柄，供 B 做 SetParent
    m_hostWidget->setStyleSheet("background: #202020;"); // qianwen：给宿主区域一个深色背景，方便视觉区分

    auto *layout = new QVBoxLayout(this);            // qianwen：创建垂直布局，把宿主控件铺到整个页面中
    layout->setContentsMargins(0, 0, 0, 0);         // qianwen：去掉边距，让宿主区域完全填满页面
    layout->addWidget(m_hostWidget);                // qianwen：把宿主控件放入布局

    connect(m_process,                              // qianwen：把子进程对象作为信号发送方
            &QProcess::finished,                    // qianwen：当 B.exe 结束时触发 finished 信号
            this,                                   // qianwen：当前页面对象作为槽接收方
            &EmbeddedExePage::onProcessFinished);   // qianwen：收到进程结束后进入 onProcessFinished 做清理

    connect(m_server,                               // qianwen：把本地 server 作为信号发送方
            &QLocalServer::newConnection,           // qianwen：当有新的客户端连接到 server 时触发
            this,                                   // qianwen：当前页面对象作为槽接收方
            &EmbeddedExePage::onNewConnection);     // qianwen：进入 onNewConnection 接收这条连接
}

EmbeddedExePage::~EmbeddedExePage()
{
    stop(); // qianwen：析构前统一调用 stop，确保子进程和通信资源都被回收
}

QString EmbeddedExePage::appId() const
{
    return m_config.appId; // qianwen：直接返回配置中保存的 appId
}

bool EmbeddedExePage::isReady() const
{
    return m_ready; // qianwen：返回当前是否已经收到 READY 并完成嵌入
}

bool EmbeddedExePage::start()
{
    stop(); // qianwen：先停止旧会话，避免重复启动导致状态叠加

    m_sessionId = QString("%1_%2_%3")                        // qianwen：构造当前会话唯一标识字符串
        .arg(m_config.appId)                                 // qianwen：把 appId 放进去，便于区分是哪类应用
        .arg(QCoreApplication::applicationPid())             // qianwen：加上 A 的进程 pid，降低冲突概率
        .arg(QDateTime::currentMSecsSinceEpoch());           // qianwen：再加上毫秒时间戳，保证每次启动都不同

    if (!startServer()) {                                    // qianwen：先启动本地 server，确保 B 稍后能连回来
        qWarning() << "startServer failed";                  // qianwen：如果 server 启动失败，输出日志
        return false;                                        // qianwen：启动失败则直接返回
    }

    QStringList args = m_config.arguments;                   // qianwen：先取业务参数作为基础参数列表
    args << QString("--embed-server=%1").arg(m_serverName);  // qianwen：追加 serverName，让 B 知道要连哪个 server
    args << QString("--embed-session=%1").arg(m_sessionId);  // qianwen：追加 sessionId，让 B 回消息时带回正确会话标识
    args << QString("--embed-host=%1")                       // qianwen：追加宿主窗口句柄参数，让 B 知道要挂到哪个窗口下面
            .arg(static_cast<qulonglong>(m_hostWidget->winId())); // qianwen：把宿主控件的 winId 转成整数传给 B

    m_process->start(m_config.program, args);                // qianwen：启动配置中指定的 B.exe，并传入完整参数列表
    if (!m_process->waitForStarted(5000)) {                  // qianwen：同步等待最多 5 秒，确认进程真的启动成功
        qWarning() << "start process failed:" << m_config.program; // qianwen：启动失败时打印 exe 路径
        stopServer();                                        // qianwen：既然子进程没起来，就顺手把 server 也关掉
        return false;                                        // qianwen：返回启动失败
    }

    return true;                                             // qianwen：走到这里说明 A 侧启动和监听都已成功
}

void EmbeddedExePage::stop()
{
    clearConnection();                  // qianwen：先清理 socket 连接，避免残留通信状态
    m_embeddedWinId = 0;                // qianwen：清空当前记录的子窗口句柄
    m_ready = false;                    // qianwen：把页面状态重置为未就绪

    if (m_process->state() != QProcess::NotRunning) { // qianwen：如果 B.exe 当前仍在运行，则需要关闭它
        m_process->terminate();                        // qianwen：先尝试优雅结束进程
        if (!m_process->waitForFinished(3000)) {      // qianwen：等待最多 3 秒，看它是否自行退出
            m_process->kill();                        // qianwen：如果还没退，就强制 kill
            m_process->waitForFinished(1000);         // qianwen：再等 1 秒，确保进程句柄状态收敛
        }
    }

    stopServer();                         // qianwen：最后关闭本地 server，并释放名字占用
}

void EmbeddedExePage::resizeEvent(QResizeEvent *event)
{
    QWidget::resizeEvent(event);          // qianwen：先走 QWidget 默认的 resize 处理流程
    updateEmbeddedGeometry();             // qianwen：随后按新尺寸同步更新 B.exe 子窗口大小
}

void EmbeddedExePage::onNewConnection()
{
    if (m_socket) {                                           // qianwen：如果当前已经有一条主连接，则新来的连接视为多余连接
        QLocalSocket *extra = m_server->nextPendingConnection(); // qianwen：从待接收队列里取出这条多余连接
        if (extra) {                                           // qianwen：如果确实取到了连接对象
            extra->disconnectFromServer();                     // qianwen：主动断开这条多余连接
            extra->deleteLater();                              // qianwen：把它延迟删除，交给 Qt 事件循环安全回收
        }
        return;                                                // qianwen：已有主连接时不再继续处理新连接
    }

    m_socket = m_server->nextPendingConnection();             // qianwen：取出第一条有效连接，作为当前会话的主通信连接
    if (!m_socket) {                                           // qianwen：防御性判断，如果没取到则直接返回
        return;
    }

    connect(m_socket,                                          // qianwen：把当前 socket 作为信号发送方
            &QLocalSocket::readyRead,                          // qianwen：当 socket 上有数据可读时触发
            this,                                              // qianwen：当前页面作为槽接收方
            &EmbeddedExePage::onSocketReadyRead);              // qianwen：进入 onSocketReadyRead 解析 READY 消息

    connect(m_socket,                                          // qianwen：继续监听这条 socket 的断开信号
            &QLocalSocket::disconnected,                       // qianwen：当 B 断开连接或退出时触发
            this,                                              // qianwen：由当前页面处理断连收尾
            [this]() {                                         // qianwen：这里用 lambda 简化断连后的状态清理
                if (m_socket) {                                // qianwen：只有当前确实还有 socket 时才做删除
                    m_socket->deleteLater();                   // qianwen：延迟删除 socket 对象
                    m_socket = nullptr;                        // qianwen：把成员指针清空，允许未来新连接重新接入
                }
            });
}

void EmbeddedExePage::onSocketReadyRead()
{
    if (!m_socket) {                                           // qianwen：如果 socket 已不存在，则没有可读数据可处理
        return;
    }

    const QString msg = QString::fromUtf8(m_socket->readAll()).trimmed(); // qianwen：一次性读出当前 socket 缓冲中的全部文本并去掉首尾空白
    const QStringList parts = msg.split(':');                  // qianwen：按冒号拆分协议字段，READY 协议采用冒号分隔

    if (parts.size() != 4 || parts[0] != "READY") {           // qianwen：协议必须是 4 段且首段必须是 READY
        qWarning() << "invalid message:" << msg;              // qianwen：协议不合法时打印原始文本
        return;                                               // qianwen：直接返回，不继续解析
    }

    const QString sessionId = parts[1];                       // qianwen：第 2 段是 sessionId，用于确认消息属于当前页面

    bool okPid = false;                                       // qianwen：接收 pid 文本转整数是否成功
    bool okWid = false;                                       // qianwen：接收 winId 文本转整数是否成功
    const qint64 pid = parts[2].toLongLong(&okPid);          // qianwen：把第 3 段转换成 B.exe 的进程 pid
    const qulonglong winIdValue = parts[3].toULongLong(&okWid); // qianwen：把第 4 段转换成 B 主窗口 winId

    if (!okPid || !okWid) {                                   // qianwen：如果 pid 或 winId 任意一个转换失败，说明协议数据格式不对
        qWarning() << "parse READY failed:" << msg;           // qianwen：打印失败消息
        return;                                               // qianwen：中止处理
    }

    if (sessionId != m_sessionId) {                           // qianwen：必须确认 B 回来的 sessionId 和当前页面启动出去的完全一致
        qWarning() << "session mismatch:" << sessionId << m_sessionId; // qianwen：不一致时输出日志，避免串页签
        return;                                               // qianwen：不是当前会话就直接忽略
    }

    if (pid != m_process->processId()) {                      // qianwen：进一步校验 B 回来的 pid 是否就是当前页面启动的那个进程
        qWarning() << "pid mismatch:" << pid << m_process->processId(); // qianwen：不一致说明连接来源不可信或状态错乱
        return;                                               // qianwen：直接忽略这条消息
    }

    m_embeddedWinId = static_cast<WId>(winIdValue);           // qianwen：记录 B.exe 主窗口句柄，后续 resize 时要用它 MoveWindow
    m_ready = true;                                           // qianwen：标记当前页面已经完成嵌入

    updateEmbeddedGeometry();                                 // qianwen：收到 READY 后立即根据当前宿主区域尺寸同步一次大小
}

void EmbeddedExePage::onProcessFinished(int, QProcess::ExitStatus)
{
    m_ready = false;                  // qianwen：进程结束后，页面不再处于 ready 状态
    m_embeddedWinId = 0;              // qianwen：把记录的子窗口句柄清零，防止后续误用无效窗口
    clearConnection();                // qianwen：清理 socket 连接对象和连接状态
    stopServer();                     // qianwen：关闭当前会话的本地 server
}

bool EmbeddedExePage::startServer()
{
    stopServer();                                     // qianwen：先关掉旧 server，避免重复 listen
    m_serverName = buildServerName();                 // qianwen：生成当前会话唯一 serverName
    QLocalServer::removeServer(m_serverName);         // qianwen：防御性清理系统中可能残留的同名 server

    if (!m_server->listen(m_serverName)) {            // qianwen：开始监听这个 serverName，等待 B connectToServer
        qWarning() << "listen failed:" << m_server->errorString(); // qianwen：监听失败时打印错误文本
        return false;                                 // qianwen：返回失败
    }

    return true;                                      // qianwen：监听成功
}

void EmbeddedExePage::stopServer()
{
    if (m_server->isListening()) {                    // qianwen：如果当前 server 正在监听，则先关闭监听
        m_server->close();                            // qianwen：关闭本地 server
    }

    if (!m_serverName.isEmpty()) {                   // qianwen：如果当前确实记录了 serverName，则还要把系统中的名字占用移除
        QLocalServer::removeServer(m_serverName);    // qianwen：移除这个名字对应的本地 server 节点
        m_serverName.clear();                        // qianwen：清空成员中的 serverName
    }
}

QString EmbeddedExePage::buildServerName() const
{
    return QString("embed_%1_%2_%3")                 // qianwen：构造统一格式的唯一 serverName
        .arg(m_config.appId)                         // qianwen：带上 appId，便于观察时区分应用类型
        .arg(QCoreApplication::applicationPid())     // qianwen：带上 A 当前进程 pid，降低不同 A 进程间冲突概率
        .arg(QDateTime::currentMSecsSinceEpoch());   // qianwen：带上毫秒时间戳，保证同一进程多次创建也不同名
}

void EmbeddedExePage::clearConnection()
{
    if (m_socket) {                                  // qianwen：只有当前确实存在 socket 时才需要清理
        m_socket->disconnect(this);                  // qianwen：断开当前页面对象与 socket 之间的所有信号槽连接
        m_socket->disconnectFromServer();            // qianwen：主动断开与 A/B 对端的连接关系
        m_socket->deleteLater();                     // qianwen：延迟删除 socket 对象，避免立即 delete 的时序问题
        m_socket = nullptr;                          // qianwen：把成员指针清空，表示当前不再持有连接
    }
}

void EmbeddedExePage::updateEmbeddedGeometry()
{
    if (!m_ready || !m_embeddedWinId) {              // qianwen：只有在 ready 且句柄有效时才允许调整子窗口尺寸
        return;
    }

    HWND hChild = reinterpret_cast<HWND>(m_embeddedWinId); // qianwen：把记录的 B 主窗口句柄转换成 Win32 HWND
    QRect rc = m_hostWidget->rect();                         // qianwen：获取宿主控件内部可用矩形区域
    MoveWindow(hChild,                                       // qianwen：指定要移动/缩放的子窗口句柄
               0,                                            // qianwen：左上角 x 坐标固定为宿主区域左边界
               0,                                            // qianwen：左上角 y 坐标固定为宿主区域上边界
               rc.width(),                                   // qianwen：宽度同步为宿主区域宽度
               rc.height(),                                  // qianwen：高度同步为宿主区域高度
               TRUE);                                        // qianwen：要求系统立即触发重绘
}
```

