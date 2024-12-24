+++
date = '2024-12-24T09:54:13+08:00'
draft = false
title = 'QT5程序崩溃问题的解决'
tags = ["QT5", "崩溃", "WinDBG"]
+++

### 1\. 问题现象
在现场的Windows7笔记本部署自动化测试工具后，多次出现闪退的问题。<br>

### 2\. 调试过程
使用微软的WinDBG程序对程序两次崩溃时抓取到minidump文件继续分析，如下图所示，两次崩溃分别发生在QT5Cored这个库中的
QPersistentModelIndex::isValid()和QModelIndex::row()函数，原因都是非法指针访问。<br>
第一次崩溃的信息：
![崩溃详情-1](../images/QT5程序崩溃问题的解决/崩溃详情-1.png "崩溃详情-1")
第二崩溃的信息：
![崩溃详情-2](../images/QT5程序崩溃问题的解决/崩溃详情-2.png "崩溃详情-2")
<br>以
经过分析分析两次崩溃的完整堆栈，我们可以发现两次崩溃都是QT5的事件循环系统调用QT5Cored这个库中的
QItemSelectionModel::isSelected函数时发生的崩溃。<br>
第一次崩溃的堆栈：
![崩溃堆栈-1](../images/QT5程序崩溃问题的解决/崩溃堆栈-1.png "崩溃堆栈-1")
第二次崩溃的堆栈：
![崩溃堆栈-2](../images/QT5程序崩溃问题的解决/崩溃堆栈-1.png "崩溃堆栈-2")
而这两个崩溃堆栈的源头都是Qt 应用程序的事件循环函数QApplication::exec()，这个是QT程序在启动时就运行的，一直伴随整
个程序的生命周期。两个崩溃堆栈都不涉及自动化测试程序的逻辑代码，因此并不是程序本身的代码出现非法指针访问。
崩溃堆栈的源头：
![崩溃堆栈入口](../images/QT5程序崩溃问题的解决/崩溃堆栈入口.png "崩溃堆栈入口")
通过两个崩溃堆栈的分析，崩溃发生在QT的事件处理循环逻辑中，整个过程不涉及外界传递给这个逻辑的任何变量，都是在处理QT界面上的操作事件。
因为都是崩溃在QItemSelectionModel::isSelected，它是用来判断一个单元格是否被选中的，跟程序逻辑唯一存在关联的就是程序存在禁止用
户随意选中单元格的逻辑，但这个逻辑只是修改节点状态，不会导致节点为空的。<br>
跟这个可能有关的代码在于：
```C++
// 在TableView的Item代理类中的Paint函数中有以下逻辑：
//如果单元格被选中，就取消选中状态
if (opt.state & QStyle::State_Selected) {
    opt.state &= ~QStyle::State_Selected; // 取消选中状态
}
```
```C++
//凡是不可操作的单元格，不允许选中！！
void TestTableView::selectionChanged(const QItemSelection &selected, const QItemSelection &deselected) {
    // Call the base class implementation to handle selection changes
    QTableView::selectionChanged(selected, deselected);
    qDebug() << "Selection changed";

    // Iterate over the selected indexes and remove the forbidden cell from the selection
    for (const QModelIndex &index : selected.indexes()) {
        QStandardItem *item = dynamic_cast<QStandardItemModel*>(model())->itemFromIndex(index);
        auto* testItem = dynamic_cast<TestFormItem*>(item);
        if (!testItem->isOperable()) {
            //取消选中状态
            selectionModel()->select(index, QItemSelectionModel::Deselect);
        }
    }
}
```
### 3\. 解决方案
经过阅读QT5最终崩溃的代码，有可能是通过这这种方式对选中状态进行修改导致QItemSelectionModel::isSelected函数判断时出现数据不同步的问题，从而
导致指针非法访问。<br>
后来通过调整实现方式，采用以下函数实现放开或者禁止用户选中、编辑TableView单元格的则能解决崩溃的问题，因此有理由认为这个问题是QT5的Bug。
```C++
//禁止对单元格进行编辑
this->setEditTriggers(QAbstractItemView::NoEditTriggers); // Disable editing
//修改表格的选中模式
if (needSelect) {
    this->setSelectionMode(QAbstractItemView::SingleSelection); // Allow single selection
} else {
    this->setSelectionMode(QAbstractItemView::NoSelection); // Disable selection
}
```
