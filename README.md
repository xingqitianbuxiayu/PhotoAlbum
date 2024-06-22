### 项目开发过程中知识点总结
1. 遵循的一些编程规范
```
* 成员变量的命名使用_first_variable，成员函数的命名使用void SlotOpenPro()来表明是自定义的类方法。
* 若类中需包含其他自定义的类对象指针作为成员变量，那么最好定义为自定义类对象的基类类型的指针，
  后续过程中用dynamic_cast进行基类到子类的转换，这样做可以防止互引用。
* git log或git reflog查看历史提交，找到某次提交的代号，随后使用git show 9ddc9dca00b --stat查看本次提交的详细文件列表
```

2. Qt知识点
```
* 对于Qt事件函数，当在自定义的类中重写了基类的事件函数，实现完自定义的处理逻辑之后，在最后最好再调用一下基类的事件处理函数，以防止基类事件函数不可用
  例如自定义了PicButton类，继承自QPushButton，在PicButton类中重写void event(QEvent *e), 在最后调用一下QPushButton::event(e)，来还原按钮的基本事件功能
* 继承自QObject的类对象最好使用deleteLater()，而不是直接使用delete()。
* 在UI界面编辑时，可以直接按住Ctrl拖动已在布局中的控件进行拷贝。
* this->setCentralWidget(QWidget*)会掌管其QWidget*的生命周期，无需自己手动再次释放。
* Stacked Widget控件包含两个QWidget（或者说两个页面）
* 将qss样式文件添加进qrc资源文件，并在程序中加载，可以为项目中的界面或控件设置样式
```

### 项目的构建文件photoAlbum.pro
1. 设置程序的图标，路径为项目下的相对路径
```
RC_ICONS = "icon/album.ico"
```

### 程序入口文件main.cpp
1. 定义应用程序QApplication，定义主窗口MainWindow类。  
2. 设置窗口标题、窗口最大化显示，在.pro文件中添加RC_ICONS = "icon/album.ico"为窗体设置显示图标  
3. 并且使用QFile为QApplication加载qss样式文件，qss样式文件中定义了应用程序中窗口各个部分的显示样式。

### 主窗口相关文件
#### MainWindow类
用于显示应用程序的主窗口，继承自QMainWindow
1. mainwindow.ui
```
* 在CentralWidget中添加一个QHBoxLayout, 并设置CentralWidget为栅格布局以让QHBoxLayout铺满整个界面，
* 随后在QHBoxLayout中一左一右添加一个名为proLayout和一个名为picLayout的QVBoxLayout，两个QVBoxLayout的scretch伸缩比例为1:4
* proLayout中包含一个QLabel和一个QTreeWidget以目录树的形式展示项目目录
* picLayout中包含两个QWidget（每个QWidget中包含一个QPushButton）和一个QLabel，用于图片展示和上一张下一张图片控制。
```
2. mainwindow.h：
```
* 添加了私有成员变量QWidget* _proTree，用于在MainWindow的左侧proLayout布局中显示一个树型View，来展示项目下的文件夹；
* 添加了私有成员变量QWidget* _picShow，用于在MainWindow的右侧picLayout布局中显示图片。
** 以上两个私有成员变量为QWidget*类型，在使用时需要通过动态转换为我们自定义的ProTree*和PicShow*类型，这样做是为了文件之间指针相互包含。
* 添加了私有槽函数void createPro(bool flag);用于绑定菜单项的添加项目动作。
* 添加了私有槽函数void openPro(bool flag);用于绑定菜单项的打开项目动作。
* 添加了信号void sigOpenPro(const QString& path);用于绑定ProTreeWidget类中的槽函数void slotOpenPro(const QString& path);
* 重写QWidget类的virtual void resizeEvent(QResizeEvent *event);函数，实现当mainwindow窗口改变时，调用_picShow对象的reloadPic方法（重新获取当前窗口大小并重绘），实现图片的重绘
```
3.  mainwindow.cpp
```
* 构造函数中：
** 调用setMinimumSize()设置窗口的最小宽度和高度
** new初始化私有数据成员QWidget* _proTree，并将其添加进ui->proLayout
** new初始化私有数据成员QWidget* _picShow，并将其添加进ui->picLayout
** 添加“文件(&F)”菜单项，在该菜单项中添加QAction“创建项目”动作和“打开项目”动作，并设置显示图标、文本和快捷键。
** 为“创建项目”动作的triggered信号绑定槽函数createPro，为“打开项目”动作的triggered信号绑定槽函数openPro
** 添加“文件设置(&S)”菜单项，在该菜单项中添加QAction“设置音乐”动作，并设置显示图标、文本和快捷键。

* 该函数中将会实现proTreeWidget与PicShow之间的通信

* 槽函数void createPro(bool flag)中实现创建项目相应逻辑：
** 初始化一个自定义的Wizard对象，并将Wizard类中的void SigProSettings(const QString name, const QString path);信号与ProTree中的void addProToTree(const QString name, const QString path);槽函数绑定
** 最后将这个Wizard以.exec()的方式显示出来，用于引导用户指定要创建的项目的路径和名称

* 槽函数void openPro(bool flag)中实现打开项目相应逻辑：

* 信号sigOpenPro
``` 

### 向导对话框相关文件
#### Wizard类
向导对话框与菜单“文件”的“创建项目”动作相关，继承自QWizard，添加了两个QWizardPage  
这两个QWizardPage会被提升为自定义的ProSetPage类和ConfirmPage类
1. wizard.ui
```
包含两个页面，第一个页面用于指定项目的名称和路径，第二个页面用于最终确认创建项目
```
2. wizard.h
```
* 重写QWizard类的void done(int result) override;函数，在引导对话框结束时，获取第一个页面的两个lineEdit中指定的项目名称和路径
* 定义了信号void SigProSettings(const QString name, const QString path);，用于将项目名称和路径发送给ProTree
```
3. wizard.cpp
```
* 重写void done(int result)函数
```

#### ProSetPage类
向导对话框的第一个页面，用于获取项目路径和名称，继承自QWizardPage
1. prosetpage.h
2. prosetpage.cpp
3. prosetpage.ui

#### ConfirmPage类
向导对话框的第二个页面，作提示用，继承自QWizardPage
1. confirmpage.h
2. confirmpage.cpp
3. confirmpage.ui

### 左侧proLayout布局相关文件
#### ProTree类
添加进proLayout布局中，包含一个QLabel和QTreeWidget，继承自QDialog  
QTreeWidget会提升为自定义的ProTreeWidget类
1. protree.h
2. protree.cpp
3. protree.ui

#### ProTreeWidget类
用于重写QTreeWidget相关函数，实现自定义树型View功能，继承自QTreeWidget
1. protreewidget.h
2. protreewidget.cpp

#### ProTreeItem类
用于重写QTreeWidgetItem相关函数，实现自定义树型Item功能，继承自QTreeWidgetItem
1. protreewidget.h
2. protreewidget.cpp

#### const.h头文件
用于定义一些枚举值，表示树型View中的项目的根文件夹、子文件夹、子文件

#### ProTreeThread类
用于将一个线程绑定一个进度条对话框，从而实现让子线程处理文件的拷贝以及文件或文件夹添加进ProTreeWidget，并以进度条形式显示
1. protreethread.h
2. protreethread.cpp

#### OpenTreeThread类
与ProTreeThread类相，不同的是无需实现文件拷贝，子线程只需将文件或文件夹添加进ProTreeWidget中

#### RemoveProDialog类
一个对话框，在ProTreeWidget类中调用，用于引导移除项目

### 右侧picLayout布局相关文件
#### PicShow类
图片的展示区域，以及上一张、下一张按钮

#### PicButton类
PicShow类中的两个按钮会升级为PicButton类，用于自定义按钮的展示效果（鼠标滑动进入、默认状态、鼠标悬停、鼠标点击时显示的效果）

### 轮播图播放界面相关文件
#### SlideShow类
用于展示轮播图，包括图片动画显示区域，上一张、下一张按钮、退出按钮、一个按钮控制暂停与播放

#### PicAnimationWidget类
SlideShow类中的一个按钮控制暂停与播放会被升级为PicAnimationWidget类，其余按钮升级为PicButton类

#### PreviewListWidget类
用于在轮播图界面中以列表视图来展示播放过的幻灯片图片

#### PicStatusButton类
用于设置一个按钮的六种显示状态的切换（播放和暂停时的默认、鼠标悬停、鼠标点击）

