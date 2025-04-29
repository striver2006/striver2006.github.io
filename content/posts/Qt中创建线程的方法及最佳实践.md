+++
date = '2025-04-29T10:45:13+08:00'
draft = false
title = 'Qt中创建线程的方法及最佳实践'
tags = ["QT","线程"]
+++

### 

从您的代码看，您正在使用继承`QThread`并重写`run()`方法来创建线程。虽然这种方法可行，但Qt官方更推荐使用Worker对象模式。以下是Qt中创建线程的几种主要方法：

### 1. 继承QThread方式（您目前的方法）

```cpp
class UsbMountTask : public QThread {
protected:
    void run() override {
        while (m_running) {
            // 执行线程工作
        }
    }
};

// 使用方式
UsbMountTask *task = new UsbMountTask();
task->start();  // 启动线程
```

**缺点**：
- QThread对象本身属于旧线程，只有run()方法在新线程运行
- 容易在子类中误用GUI相关操作
- 不利于代码复用

### 2. Worker对象模式（推荐）

```cpp
// Worker类，包含实际工作逻辑
class UsbMounter : public QObject {
    Q_OBJECT
public slots:
    void process() {
        // 所有工作逻辑
        while (!m_stop) {
            // 执行USB挂载相关操作
            emit deviceMounted(device, mountPoint);
        }
        emit finished();
    }

signals:
    void deviceMounted(QString device, QString mountPoint);
    void finished();

private:
    std::atomic<bool> m_stop{false};
};

// 使用方式
QThread* thread = new QThread;
UsbMounter* worker = new UsbMounter();
worker->moveToThread(thread);

// 连接信号和槽
connect(thread, &QThread::started, worker, &UsbMounter::process);
connect(worker, &UsbMounter::finished, thread, &QThread::quit);
connect(worker, &UsbMounter::finished, worker, &UsbMounter::deleteLater);
connect(thread, &QThread::finished, thread, &QThread::deleteLater);

thread->start();
```

### 3. QThreadPool和QRunnable（适合简单任务）

```cpp
class UsbTask : public QRunnable {
public:
    void run() override {
        // 执行挂载逻辑
    }
};

// 使用方式
UsbTask *task = new UsbTask();
QThreadPool::globalInstance()->start(task);
```

### 4. 线程安全和通信

```cpp
// 线程间安全数据访问
QMutex mutex;
{
    QMutexLocker locker(&mutex);
    // 安全访问共享数据
}

// 使用信号槽机制在线程间通信
connect(worker, &UsbMounter::deviceMounted, 
        this, &MainWindow::onDeviceMounted, 
        Qt::QueuedConnection);  // 跨线程连接
```

### 5. 优雅停止线程

```cpp
// 在您的UsbMountTask中
void UsbMountTask::stop() {
    m_running = false;
    wait();  // 等待线程结束
}
```

建议将您的`UsbMounter`类改为QObject而非QThread，然后使用Worker对象模式，这样会使代码结构更清晰，职责划分更明确。