

# Note of Qt

[TOC]





### 初始窗口构建

```C++
#include <QApplication>
#include <QLabel>
int main(int argc, char *argv[])
{
	QApplication app(argc, argv);
	QLabel label("Hello, world");
	label.show();
  //调用 app.exec()，开启事件循环
	return app.exec();
  
}
```

<img src="/Users/kangyixiao/Library/Application Support/typora-user-images/截屏2020-09-16 下午5.24.40.png" alt="截屏2020-09-16 下午5.24.40" style="zoom:50%;" />

## 信号槽Slot

```C++ 
#include <QApplication> 
#include <QPushButton>
int main(int argc, char *argv[]) {
	QApplication app(argc, argv);
	QPushButton button("Quit");
	QObject::connect(&button, &QPushButton::clicked, &QApplication::quit);
	button.show();
	return app.exec(); 
}
```

### connect 函数的5个重载

```C++
//connect 函数的最常用形式，第一个是发出信号的对象， 第二个是发送对象发出的信号，第三个是接收信号的对象，第四个是接收对象在接收到信号 之后所需要调用的函数。也就是说，当 sender 发出了 signal 信号之后，会自动调用 receiver的 slot 函数。
connect(sender, signal, receiver, slot);

MetaObject::Connection connect(const QObject *, const char *, const QObject *, const char *,Q t::ConnectionType);

QMetaObject::Connection connect(const QObject *, const QMetaMethod &, const QObject *, const QMetaMethod &,Qt::ConnectionType);

// 这个函数其实是将 this 指针作为 receiver。
QMetaObject::Connection connect(const QObject *, const char *, const char *,
Qt::ConnectionType) const;
// PointerToMemberFunction是指向成员函数的指针
QMetaObject::Connection connect(const QObject *, PointerToMemberFunction,const QObject *, PointerToMemberFunction, Q t::ConnectionType)；
  
QMetaObject::Connection connect(const QObject *, PointerToMemberFunction, F unctor );
```

信号槽要求信号和槽的参数一致，所谓一致，是参数类型一致。如果不一致，允许的情况是， 槽函数的参数可以比信号的少，即便如此，槽函数存在的那些参数的顺序也必须和信号的前 面几个一致起来。

### 自定义信号槽

```C++
#include <QObject>
////////// newspaper.h
//只有继承了 QObject 类的类，才具有信号槽的能力。所以，为了使用信号槽，必须继承 QObject。
class Newspaper : public QObject {
  //凡是 QObject 类(不管是直接子类还是间接子类)，都应该在第一行代码写上 Q_OBJECT,添加这个宏为后续操作增加了很多方便，不然会出错。
    Q_OBJECT
public:
    Newspaper(const QString & name) :
    m_name(name) {
    }
    
    void send() {
      //emit 的含义是发出，也就是发出 newPaper()信号。
        emit newPaper(m_name); 
    }
signals :
  //信号就是一个个的函数名，返回值是 void,参数是该类需要让外界知道的数据，不需要在 cpp 函数中添加任何实现
    void newPaper(const QString &name);
private :
    QString m_name;
};

////////// reader.h #include <QObject>
#include <QDebug>
class Reader : public QObject {
  //添加了Q_OBJECT宏。
    Q_OBJECT 
public:
    Reader() {}
  //槽函数必须是public,cpp中需要实现
    void receiveNewspaper(const QString & name)
    {
        qDebug() << "Receives Newspaper: " << name;
    } 
};

////////// main.cpp #include <QCoreApplication>
#include "newspaper.h"
#include "reader.h"
int main(int argc, char *argv[])
{
    QCoreApplication app(argc, argv);
    Newspaper newspaper("Newspaper A");
    Reader reader;
    QObject::connect(&newspaper, &Newspaper::newPaper,
                     &reader, & Reader ::receiveNewspaper);
  //一定要emit 之后才会产生反应，上一步仅仅是关联起来。
    newspaper.send ();
    return app.exec();
}

```



### MainWindow

```C++
#include "mainwindow.h"
#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    MainWindow w;
    w.show();

    return a.exec();
}
```

  **窗口分区**

<img src="Note of Qt.assets/截屏2020-09-16 下午5.49.36.png" alt="截屏2020-09-16 下午5.49.36" style="zoom:50%;" />

1. Window Title，也就是标题栏，通常用于显示标题和控制按钮，比如最 大化、最小化和关闭等。

2. 最外层称为 Tool Bar Area，用于显示工具条区域。之所以是矩形表示，是因为，Qt 的主窗口支持多个工具条。你可以将工具条拖放到不同的位置。
3. 在工具条区域内部 是 Dock Widget Area，这是停靠窗口的显示区域。所谓停靠窗口，就像 Photoshop 的工具 箱一样，可以停靠在主窗口的四周，也可以浮动显示
4. 主窗口最中间称为 CentralWidget，就是我们程序的工作区。通常我们会将程序最主要的工作区域放置在这里，类似 Word 的 稿纸或者 Photoshop 的画布等等。
5. 窗口最底部是 Status Bar，称为状态栏。当我们鼠标滑过某些组件时，可以在状态栏显示某些信息，比如浏览器中，鼠标滑过带有链接的文字，你会在底部看到链接的实际 URL。

### 添加动作



```C++
#include <QAction>
#include <QMenuBar>
#include <QMessageBox>
#include <QStatusBar>
#include <QToolBar>
#include "mainwindow.h"
MainWindow::MainWindow(QWidget *parent) : QMainWindow( parent )
{
    // setWindowTitle()，设置主窗口的标题,tr()是一个用于 Qt 国际化的函数。
    setWindowTitle(tr("Main Window"));
    // QIcon的参数，以 : 开始，意味着从资源文件 中查找资源。
    // :/images/doc-open就是找到了这里的 document-open.png 这个文件。内置格式为png
    //第二个参数中，文本值前面有一个 &，意味着这将成为一个快捷键。
    openAction = new QAction(QIcon(":/images/doc-open"), tr("&Open..."), this);
    // setShortcut()函数，用于说明这个 QAction 的快捷键,使用 QKeySequence 类来添加快捷键，
    //会根据平台的不同来定义相应的快捷键。
    openAction-> setShortcuts(QKeySequence:: Open) ;
    //setStatusTip()则实现了当用户鼠标滑过这个 action 时，会在主窗口下方的状态栏显示相应的提示。
    openAction->setStatusTip(tr("Open an existing file"));
    connect(openAction, &QAction::triggered, this, &MainWindow::open);
    //menuBar()、toolBar()和 statusBar()三个是 QMainWindow 的函数，用于创建并 返回菜单栏、工具栏和状态栏
    //我们向菜单栏添加了一个 File 菜单，并且把这个 QAction 对象添加到这个菜单
    QMenu *file = menuBar()->addMenu(tr("&File"));//&File用处是什么
    file->addAction(openAction);
    //同时新增加了一个 File 工具栏，把QAction 对象添加到了这个工具栏。tr("&File")设置toolbar的名字
    QToolBar *toolBar = addToolBar(tr("&File"));
    toolBar ->addAction (openAction);
    statusBar() ;
}

MainWindow::~ MainWindow () {
}

void MainWindow::open() {
    //打开具有给定标题和文本的信息消息框。新打开的窗口显示Open
    //information(QWidget *parent, const QString &title, const QString &text, int button0, int button1 = 0, int button2 = 0)
    QMessageBox::information(this, tr("Information"), tr("Open"));
}
```

快捷键

按F1 显示快捷键在不同系统中的按法

<img src="Note of Qt.assets/截屏2020-09-16 下午6.14.34.png" alt="截屏2020-09-16 下午6.14.34" style="zoom:50%;" />

## 资源文件

点击工程文件，添加现有文件，如下选择

<img src="Note of Qt.assets/截屏2020-09-16 下午7.03.51.png" alt="截屏2020-09-16 下午7.03.51" style="zoom:33%;" />

设置资源库的位置

<img src="Note of Qt.assets/截屏2020-09-16 下午7.07.21.png" alt="截屏2020-09-16 下午7.07.21" style="zoom:50%;" /

“添加”，我们首先需要添加前缀，比如我们将前缀取名为 /images,再添加文件，现在可以用:/images/1.png 找到，取别名,现在可以用/images/1找到

![截屏2020-09-16 下午7.11.35](Note of Qt.assets/截屏2020-09-16 下午7.11.35.png)

## 布局管理器

**当signal 是重载函数时，显示指定函数。**

那么我们就显式指定一 个函数。方法就是，我们创建一个函数指针，这个函数指针参数指定为 int:

```C++
void (QSpinBox:: *spinBoxSignal)(int) = &QSpinBox::valueChanged;
```

然后我们将这个函数指针作为 signal，与 QSlider 的函数连接:

```C++
void (QSpinBox:: *spinBoxSignal)(int) = &QSpinBox::valueChanged;
QObject::connect(spinBox, &QSpinBox::valueChanged, slider, &QSlider::setValue );
```

工具栏可以有多个

```C++
QToolBar *toolBar = addToolBar(tr("&File")); 
toolBar ->addAction (open Action);
QToolBar *toolBar2 = addToolBar(tr("Tool Bar 2")); 
toolBar2->addAction(openAction);
```

<img src="Note of Qt.assets/截屏2020-09-16 下午7.32.40.png" alt="截屏2020-09-16 下午7.32.40" style="zoom:50%;" />

## 对话框Dialog

```c++
void MainWindow::open() {
    QDialog dialog;
    //打开具有给定标题和文本的信息消息框。新打开的窗口显示Open
    //information(QWidget *parent, const QString &title, const QString &text, int button0, int button1 = 0, int button2 = 0)
    //QMessageBox::information(this, tr("Information"), tr("Open"));
    dialog.setWindowTitle(tr("Hello, dialog!"));
    dialog. exec( );
}
```



<img src="Note of Qt.assets/截屏2020-09-16 下午8.15.12.png" alt="截屏2020-09-16 下午8.15.12" style="zoom:50%;" />

设置dialog的parent指针之后（dialog(this)），dialog 窗口会在mainwindow 里面，不设置就是自由的.

<img src="Note of Qt.assets/截屏2020-09-16 下午8.18.15.png" alt="截屏2020-09-16 下午8.18.15" style="zoom:50%;" />

**Dialog模式**

1. show()默认显示的是非模态对话框,即此对话框出现后你还可以对其他窗口进行操作,可以用setModal函数进行设置窗口为模态,即无法操作其他窗口,即被阻塞. 而exec()出现的只能是模态对话框.

2. show()函数不会阻塞当前线程，对话 框会显示出来，然后函数立即返回，代码继续执行。注意，dialog 是建立在栈上的，show()函数返回，MainWindow::open()函数结束，dialog 超出作用域被析构，因此对话框消失了。 知道了原因就好改了，我们将 dialog 改成堆上建立。
3. setAttribute()函数设置对话框关闭时，自动销毁对话框。另外，QObject 还有一个 deleteLater()函数，该函数会在当前事件循环结束时销毁该对话框(具体到这里，需要使用
   exec()开始一个新的事件循环)。

```C++
void MainWindow::open() {
    QDialog *dialog = new QDialog; dialog->setWindowTitle(tr("Hello, dialog!"));
    //防止内存泄露，setAttribute()函数设置对话框关闭时，自动销毁对话框。
    dialog->setAttribute(Qt ::WA_DeleteOnClose);
    dialog->show ();
}
```

### 对话框数据传递

**exec()返回值**

```c++
void MainWindow::open() {
    QDialog dialog(this); 
    dialog.setWindowTitle(tr("Hello, dialog!"));
    dialog.exec( );
    qDebug() << dialog.result();
}
```

注意，exec()开始了一个事件循环，代码被阻塞到这里。由于 exec()函数没有返回，因此下面的 result()函数也就不会被执行。直到对话框关闭，exec()函数返回，此时，我们就可以取得对话框的数据。QDialog::exec()是有返回值的，其返回值是 QDialog::Accepted (1)或者 QDialog::Rejected(0)。一般我们会使用类似下面的代码

### 标准对话框 QMessageBox

```c++
void MainWindow::open() {

    //QMessageBox *messagebox = new QMessageBox;
    //messagebox ->setWindowTitle("hello,messageBox!");
    //messagebox-> about(this, "hi", "how are you?");
    // messagebox->setAttribute(Qt ::WA_DeleteOnClose);
    // messagebox->show();
    //不用去实例化QMessageBox,直接使用
    if (QMessageBox::Yes == QMessageBox::question(this, tr("Question "),tr("Are you OK?"),
                                                  QMessageBox::Yes | QMessageBox::No, QMessageBox::Yes)) {
        QMessageBox::information(this, tr("Hmmm..."), tr("I'm glad to hear that!")); } 
    else {
        QMessageBox::information(this, tr("Hmmm..."), tr("I'm sorry!")); }
}

```

<img src="Note of Qt.assets/截屏2020-09-16 下午9.49.39.png" alt="截屏2020-09-16 下午9.49.39" style="zoom:50%;" height=300 /><img src="Note of Qt.assets/截屏2020-09-16 下午9.49.51.png" alt="截屏2020-09-16 下午9.49.51" style="zoom:50%;" height=300 />

