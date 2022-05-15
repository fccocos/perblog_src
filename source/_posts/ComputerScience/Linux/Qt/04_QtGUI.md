## QMainWindow

QNainWindow是一个为用户提供主窗口程序的类，包含一个菜单栏（menu bar)、多个工具栏（toolbars)、多个钟接部件（dock widgets)、一个状态栏（status Bar)及一个中心部件（central widget),是许多应用程序的基础，如文本编辑器，图片编辑器等。

### window窗口构成

[![L7RG59.png](https://s1.ax1x.com/2022/04/26/L7RG59.png)](https://imgtu.com/i/L7RG59)

## 菜单栏

```c++
#include "mainwindow.h"
#include <QMenuBar>
#include <QMenu>
#include <QAction>
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    //取出菜单栏
    QMenuBar *menuBar1 = this->menuBar();
    //向菜单栏中添加 菜单
    QMenu* filemenu=menuBar1->addMenu("文件");
    QMenu* editmenu=menuBar1->addMenu("编辑");
    //向菜单中添加菜单项
    QAction *openAction = filemenu->addAction("打开");
    //添加分割线
    filemenu->addSeparator();
    QAction *saveAction = filemenu->addAction("保存");
}

MainWindow::~MainWindow()
{
}


```

## 工具栏

```c++
//获取工具栏
QToolBar* toolBar = this->addToolBar("");
//向工具栏中添加菜单项
toolBar->addAction(openAction);
toolBar->addAction(saveAction);
```

## 状态栏

```c++
//获取状态栏
    QStatusBar* statusBar = this->statusBar();
    statusBar->addWidget(new QLabel("状态"));
```

## 铆接部件

```c++
//添加铆接部件(浮动窗口)
 QDockWidget *dockWidget = new QDockWidget("这是一个铆接部件");
//将浮动窗口添加到mainwindow中
 this->addDockWidget(Qt::TopDockWidgetArea,dockWidget);
```

## 核心部件（中心部件）

```c++
//添加一个文本编辑器
QTextEdit *textEdit = new QTextEdit();
//将文本编辑器放入到中心部件中
this->setCentralWidget(textEdit);
```

## 资源文件

Qt资源系统是一个跨平台的资源机制，用于将程序运行时所需要的资源以二进制的形式存储于可执行文件内部。如果你的程序需要加载特定的资源（图标、文本翻译等）,那么，将其放置在资源文件中，就再也不需要担心这些文件的丢失。也就是说，如果你将资源以资源文件形式存储，它是会编译到可执行文件内部。

使用QtCreator可以很方便地创建资源文件。我们可以在工程上点右键，选择“添加新文件…”，可以在Qt分类下找到“Qt资源文件”：

加载图片并为菜单项添加图片

```c++
//定义一个图片对象
QPixmap pic;
//加载图片
pic.load("://avatar");
//为菜单先添加一个图片
openAction->setIcon(pic);
```

## 对话框

### 基本概念

对话框是GUI程序中不可或缺的组成部分。很多不能或者不适合放入主窗口的功能组件都必须放在对话框中设置。对话框通常会是一个顶层窗口，出现在程序最上层，用于实现短期任务或者简洁的用户交互。

Qt中使用QDialog类实现对话框。就像主窗口一样，我们通常会设计一个类继承QDialog。QDialog(及其子类，以及所有Qt:Dialog类型的类）的对于其parent指针都有额外的解释：**如果parent为NULL,则该对话框会作为一个顶层窗口，否则则作为其父组件的子对话框（此时，其默认出现的位置是parent的中心）。**顶层窗口与非顶层窗口的区别在于，顶层窗口在任务栏会有自己的位置，而非顶层窗口则会共享其父组件的位置。对话框分为模态对话框和非模态对话框。

- 模态对话框，就是会阻塞同一应用程序中其它窗口的输入。
- 模态对话框很常见，比如“打开文件”功能。你可以尝试一下记事本的打开文件，当打开文件对话框出现时，我们是不能对除此对话框之外的窗口部分进行操作的。
- 与此相反的是非模态对话框，例如查找对话框，我们可以在显示着查找对话框的同时，继续对记事本的内容进行编辑。

### 标准对话框

所谓标准对话框，是Qt内置的一系列对话框，用于简化开发。事实上，有很多对话框都是通用的，比如打开文件、设置颜色、打印设置等。这些对话框在所有程序中几乎相同，因此没有必要在每一个程序中都自己实现这么一个对话框。

- Qt的内置对话框大致分为以下几类：
- QColorDialog:                  选择颜色；
- QFileDialog:                      选择文件或者目录：
- QFontDialog:                    选择字体；
- QInputDialog:                  允许用户输入一个值，并将其值返回；

- QMessageBox:                 模态对话框，用于显示信息、询问问题等；
- QPageSetupDialog:         为打印机提供纸张相关的选项；
- QPrintDialog:                    打印机配置；
- QPrintPreviewDialog:      打印预览；
- QProgressDialog:             显示操作过程。

```c++
#include "mainwindow.h"
#include <QMenuBar>
#include <QMenu>
#include <QAction>
#include <QList>
#include <QString>
#include <QFileDialog>
#include <QDebug>
#include <QColorDialog>
#include <QWidget>
#include <QColor>
#include <QFontDialog>
#include <QFont>
#include <QInputDialog>
#include <QMessageBox>
#include <QPushButton>
#include <QCheckBox>
#include <QtPrintSupport/QPageSetupDialog>
#include <QtPrintSupport/QPrintPreviewDialog>
#include <QtPrintSupport/QPrinter>
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    QList<QAction*> actions;
    QMenuBar* menuBar = this->menuBar();

    QMenu* fileItem = menuBar->addMenu("文件");
    actions.push_back(fileItem->addAction("打开文件夹"));
    actions.push_back(fileItem->addAction("选择颜色"));
    actions.push_back(fileItem->addAction("选择字体"));
    actions.push_back(fileItem->addAction("输入"));
    actions.push_back(fileItem->addAction("模态对话框"));
    actions.push_back(fileItem->addAction("打印纸张相关信息"));
    actions.push_back(fileItem->addAction("打印机设置"));
    actions.push_back(fileItem->addAction("打印预览"));
    actions.push_back(fileItem->addAction("显示操作过程"));
    fileItem->addActions(actions);
    connect(actions[0],&QAction::triggered,this,[]{
        QString str = QFileDialog::getOpenFileName();
        qDebug()<<str;
    });
    connect(actions[1],&QAction::triggered,this,[]{
        QColor col(80,68,32);
        QColor color = QColorDialog::getColor(col,nullptr,"取色器");
        qDebug()<<color;
    });
    connect(actions[2],&QAction::triggered,this,[]{
        bool flag = true;
        QFont font =  QFontDialog::getFont(&flag);
        qDebug()<<font;
    });
    connect(actions[3],&QAction::triggered,this,[]{
        QWidget* widget = new QWidget();
        qDebug()<< QInputDialog::getText(widget,"输入","input");
        qDebug()<<QInputDialog::getInt(widget,"输入","int");
    });
    connect(actions[4],&QAction::triggered,this,[]{
        QMessageBox msg;
        msg.setText("当前文件已修改，是否保存？");
        msg.addButton(QMessageBox::Ok);
        msg.addButton(QMessageBox::No);
        msg.addAction(new QAction(&msg));
        msg.setCheckBox(new QCheckBox(&msg));
        msg.exec();

    });

    connect(actions[5],&QAction::triggered,this,[]{
        qDebug()<<"QPageSetupDialog";
    });

    connect(actions[6],&QAction::triggered,this,[]{
        qDebug()<<"QPrintPreviewDialog";
    });

}

MainWindow::~MainWindow()
{
}


```

### 模态对话框

- Qt有两种级别的模态对话框：

  - 应用程序级别的模态

    当该种模态的对话框出现时，用户必须首先对对话框进行交互，直到关闭对话框，然后才能访问程序中其他的窗口。

  - 窗口级别的模态

    该模态仅仅阻塞与对话框关联的窗口，但是依然允许用户与程序中其它窗口交互。窗口级别的模态尤其适用于多窗口模式。

一般默认是应用程序级别的模态。

在下面的示例中，我们调用了exec()将对话框显示出来，因此这就是一个模态对话框。当对话框出现时，我们不能与主窗口进行任何交互，直到我们关闭了该对话框。

```c++
    btn = new QPushButton("打开一个模态对话框", this);
    btn->move(200,100);
    connect(btn,&QPushButton::clicked,this,[]{
        QDialog* dialog = new QDialog();
        dialog->setWindowTitle(tr("Hello,dialog"));
        dialog->exec();
    });
```

### 非模态对话框

```c++
    btn1 = new QPushButton("打开一个非模态对话框", this);
    btn1->move(200,50);
    connect(btn1,&QPushButton::clicked,this,[]{
        QDialog* dialog = new QDialog();
        dialog->setAttribute(Qt::WA_DeleteOnClose);//关闭窗口自动释放
        dialog->setWindowTitle(tr("Hello,dialog"));
        dialog->show();
    });

```

### 消息对话框

