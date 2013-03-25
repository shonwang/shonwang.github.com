---
layout: post 
category: lessons
tags: [IronPython]
---
{% include JB/setup %} 

[翻译] python Tutorial 之三
**Tutorial 2: IronPython 高级篇**

IronPython的漂亮之处大部分在于它的动态风格开发-- 通过添加功能元素，改变（modifying）运行着的应用（程序）。在Windows 应用中, 这通常要求 delegates 和 event 处理 (i.e., 向现有的form窗口添加按钮，然后对按钮添加按下的处理功能代码).

本tutorial 专注于在IronPython 中创建 delegates, 事件处理, 以及使用Windows窗体创建 Windows应用和WPF(以前的Avalon).

阅读完本tutorial估计耗时: 25 分钟

本tutorial 的目标就是了解如何在IronPython中创建 delegates 和事件处理代码, 以及通过Windows 窗体和WPF创建Windows 应用。
`Events and Delegates`
`Windows Forms`
`Windows Presentation Foundation (Avalon)`

**Exercise 1: 事件和Delegates**

在本练习中, 您将会创建一个简单的事件处理，以及了解如何探索（explore）事件处理用法. 这个事件处理即是：FileSystemWatcher - 一个触发（ raises ）文件系统变化“事件”的组件.

**Task 1: File System Watcher**

在tutorial路径下启动IronPython 控制台. (see Introduction for details).

引用System.IO 到全局名空间:

`from System.IO import *`

创建FileSystemWatcher 类实例并设置路径属性到当前路径下:

	w = FileSystemWatcher()
	dir(w)
	w.Path = "."

创建事件处理方法. 因为不了解事件处理的参数, 我们获取一定数量的参数 (the *arg notation):

`def handle(*args): print args`

注册'Changed', 'Created' 和 'Deleted' 事件处理（event handler ）:

	w.Changed += handle
	w.Created += handle
	w.Deleted += handle

将watcher 置为raise 事件有效状态:

`w.EnableRaisingEvents = True`

打开Tutorial路径并创建文件. 简单的方法就是右击鼠标选择 "新建"文本文档". 文件watcher 将会触发"Created"事件.

您可以打开文档，随意敲入一些内容，然后保存该文档，看一下这个操作触发了 "Changed" 事件. 之后

再删除该文件，看一下"Deleted" 事件被触发.
 
在这些步骤之后，window 命令行将会显示类似下面的输出:

	(System.IO.FileSystemWatcher, <System.IO.FileSystemEventArgs object at 0x03CE0BB8>)
	(System.IO.FileSystemWatcher, <System.IO.FileSystemEventArgs object at 0x039B3B4D>)
	(System.IO.FileSystemWatcher, <System.IO.FileSystemEventArgs object at 0x039B3B4D>)
	(System.IO.FileSystemWatcher, <System.IO.FileSystemEventArgs object at 0x014EF022>)

下一个工作，我们将会创建一个改进的事件处理(器). 现在, 从文件watcher中移除当前的事件处理:

	w.Changed -= handle
	w.Created -= handle
	w.Deleted -= handle

(可选项) 您可以再尝试第七步，看一下相应事件, 尽管它们依旧触发, 但它不再被Python "handler"函数处理了.

**Task 2: 改进事件处理**

在上一个任务里，步骤7中我们了解参数类型被传递到了所有三个事件中:

`FileSystemWatcher` - 触发事件的对象实例

`FileSystemEventArgs` - 关于事件触发的信息

使用dir() 去显示事件参数来找到事件所包含的信息:

	from System.IO import *
	w = FileSystemWatcher()
	w.Path = "."
	dir(FileSystemEventArgs)

控制台会显示如下输出：

	>>> dir(FileSystemEventArgs)
	['ChangeType', 'Empty', 'Equals', 'Finalize', 'FullPath', 'GetHashCode', 'GetType', 'MemberwiseClone','Name', 'ReferenceEquals', 'ToString', '__class__', '__delattr__', '__doc__', 		'__getattribute__','__hash__', '__init__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__str__', 'get_ChangeType', 'get_FullPath', 'get_Name']['ChangeType', 		'get_ChangeType','get_Name', 'FullPath', 'get_FullPath', 'Empty', 'Name']

通过事件参数属性的内容, 我们能够创建一个更好的事件处理，它将打印出事件参数对象的ChangeType和完成路径属性:

`def handle(w, a): print a.ChangeType, a.FullPath`

注册新事件处理到 'Changed', 'Created' 和 'Deleted' 事件:

	w.Changed += handle
	w.Created += handle
	w.Deleted += handle

确保事件触发为有效状态:

`w.EnableRaisingEvents = True`

再次打开Tutorial 路径，然后创建一个新的文件 ("Created" 事件),  在记事本中编辑并保存该文件("Changed" 事件)然后删除该文件 ("Deleted" 事件).

在该步骤之后, window命令行将会显示如下类似输出:

	Created ."New Text Document.txt
	Changed ."New Text Document.txt
	Changed ."New Text Document.txt
	Deleted ."New Text Document.txt

从文件watcher中移除事件处理:

	w.Changed -= handle
	w.Created -= handle
	w.Deleted -= handle

按下Ctrl+Z 或 F6 回车键，退出IronPython 控制台.

**Task 3: 在Python 中定义事件**

最后，再看一下如何在Python代码中获取事件处理语法（event handler ）:pyevent - 该模块提供Python事件支持make_event - 该方法返回事件tuple，包括可调用对象（callable object ）。该对象允许对事件连接和未连接.

创建事件:

	import pyevent
	hook,caller = pyevent.make_event()

它返回两个对象。第一个对象允许用户挂钓方法到事件上。第二个对象允许事件持有者(owner) 引发事件.（This allows for the separation of these abilities just like .NET ）现在把它放到类中看一下如何使用。

定义一个包括事件的类：

	class MyClass(object):
	    OnNewInstance,_NewInstance= pyevent.make_event()
	    def __new__(cls):
		res = object.__new__(object)
		MyClass._NewInstance(res)

	def NewInst(x): print 'new inst: ', x
	MyClass.OnNewInstance += NewInst
	a = MyClass()
	new inst

就像CLR 事件，您也可以移除事件处理（句柄）:

`MyClass.OnNewInstance -= NewInst`

**Exercise 2: Windows 窗体**

为了开发交互式 Windows 应用, IronPython 须进行初始化. 默认情况下, Python 控制台运行在单一线程下. 当线程等待控制台窗口输入文本时, 被控制台动态创建的Windows 应用不能处理 Windows消息. 因此, 当前应用不会重画自身或处理输入到UI上.

我们提供一个“.py” 文件去初始化IronPython 到Windows Forms 开发如下.

在这个练习中, 您将会通过IronPython控制台动态创建一个简单的Windows Forms 应用.

**Task 1: 简单Windows Forms 应用**

在tutorial 路径下启动IronPython 控制台. (see Introduction for details).通过加载winforms 模块/脚本，实始化Windows Forms:

`import winforms`

Python 模块自动初始化 (执行) 引用，Windows Forms 初始化代码作为引入声明部分被执行。

引入`System.Windows.Forms` 和 `System.Drawing` 名空间到全局名空间:

	from System.Windows.Forms import *
	from System.Drawing import *

创建 Form 类实例并显示它:

	f = Form()
	f.Show()

你可以按下alt-tab或查看运行程序，因为它可能不会跳到您桌面的最上面。

现在设置窗体Text 属性:

	f.Text = "My First Interactive Application"

在运行的应用程序中, 注意form 的单击事件. 为 Click 事件创建一个事件处理句柄。单击 form 接受事件. 然后移除事件处理

	def click(*args): print args
	f.Click += click

单击form 接受事件..

输出类似如下:

	>>> (System.Windows.Forms.Form, Text: My First Interactive Application, <System.Windows.Forms.MouseEventArgs object at 0x02324551>)

因为要进一步开发，现在移除单击处理句柄.

	f.Click -= click

使用dir() 函数显示MouseEventArgs

	dir(MouseEventArgs)

了解MouseEventArgs内容, 创建一个改进单击事件处理句柄:

	def click(f, a):
	    l = Label(Text = "Hello")
	    l.Location = a.Location
	    f.Controls.Add(l)

注册事件处理句柄:

	f.Click += click

现在用鼠标点击窗体，将会添加 "Hello" 标签. 我们也能够访问经鼠标单击添加进来的控件并改变它们：

	for i in f.Controls: i.Font = Font("Verdana", 15)
	for i in f.Controls: i.Text = "Hi"

在单击一会之后，form 会变得相当“拥挤”, 现在我们清空它:

	f.Controls.Clear()
	f.Close()

按下Ctrl+Z 或 F6 回车退出IronPython 控制台

The standalone version of this script is located in wfdemo.py in the Tutorial directory.
Exercise 3: Windows Presentation Foundation (Avalon)

这个练习要求安装WPF. Please see prerequisites for more information.

如同Windows Forms, Windows Presentation Foundation 也要求初始化用于支持互动开发. 初始化代码可参见"Tutorial"avalon.py" 文件.

在此练习中, 您可以创建简单的Windows Presentation Foundation 应用.

**Task 1: 简单WPF (Avalon) 应用**

在tutorial路径下启动IronPython 控制台 (see Introduction for details).实始化 Windows Presentation Foundation:

	from avalon import *

创建WPF(Avalon) 窗体, 显示该窗体, 设置一些属性:

	w = Window()
	w.Show()

您需要按下alt-tab 或查看运行程序，因为它可能不会跳到桌面最上层.

	w.Title = "My Avalon Application"
	w.SizeToContent = SizeToContent.WidthAndHeight

通过设置窗体属性到 "size to content", the window shrinks.

现在添加如下内容:

	w.Content = TextBlock()
	w.Content.Text = "Hello IronPython!"
	w.Content.FontSize = 50

移除window 内容:

	w.Content = None

**Task 2: Avalon 计算器**

If you are continuing from the Task 1, proceed to step 2.  Otherwise, please follow thesteps 1-3 from the Task 1.

Windows Presentation Foundation 使用 XAML 格式描述绘制布局和基本UI行为。加载"calc.xaml"并显示结果内容:

	from avalon import *
	w = Window()
	w.Show()
	w.Content = LoadXaml("calc.xaml")

如果您按从任务1的流程到这个任务, 然后 w.SizeToContent 可被人为设计, but in either case, you may need to manually drag the window's border down and to the right to see the calculator UI. 然后运行calculator的对象模型 (函数被定义在avalon.py 文件中)

	for n in Walk(w): print n

使用Python's 列表语法，我们能够获得 calculator 的所有按钮:

	[ n for n in Walk(w) if isinstance(n, Button) ]

	System.Windows.Controls.Button

控制台打印出所有的按钮列表.  为保存按钮列表对象到一个变量中, 使用解析的"_" 变量, 它一直会持有最近的 non-None 值结果：

	buttons = _

我们能够对所有按钮进行修改，比如修改颜色和字体:

	for b in buttons: b.FontSize *= 2
	for b in buttons: b.Foreground = SolidColorBrush(Colors.Blue)

运行calculator , 我们对每个按钮提供事件处理句柄. 它可通过 calculator.py 文件引入:

	import calculator

计算器模块包括 Calculator 类。 它会跟踪计算器表达式.  通过运行计算器 (that is, registerevent handlers for the UI), 敲入:
	
	calculator.enliven(w)


    单击计算器按钮并对表达式进行求值。

探索 calculator.py 脚本文件 (位于Tutorial 路径下).  兴趣点是Calculator的.__init__ 方法:

	def __init__(self, controls):                                           (1)
	    self.expression = ""                                                (2)
		                                                                (3)
	    for i in controls:                                                  (4)
		if isinstance(i, Button):                                       (5)
		    if hasattr(self, "on_" + i.Name):                           (6)
		        i.Click += getattr(self, "on_" + i.Name)                (7)
		elif isinstance(i, TextBox):                                    (8)
		    if i.Name == "Result":                                      (9)
		        self.result = i                                         (10)

	    self.result.Text = self.expression                                  (11)

 方法的"controls" 参数是按钮和文本框列表, 与在步骤3-4中所创建的按钮列表相似. 初始化代码遍历该列表(line 4), 识别按钮(line 5), 并使用按钮的名称("One", "Multiply", " Equals", ...) 去查看calculator属性(方法) ，即通过相应名称 (如："on_One", "on_Multiply", 和"on_Equals", 一一对应). 如果属性和方法有效，我们可以挂钓此事件, 使用名称去获取该属性(line 7). 第二个兴趣点是on_Equals 方法, 特别是突出显示行.  Python 有内置函数 "eval", 它运行字符串表达式并返回运行结果.  Calculator 使用 "eval" 运行“计算表达式”(calculator expressions). "str" 函数会将值转换成字符串显示。

	def on_Equals(self, b, e):
	    try:
		result = str(eval(self.expression))
		self.result.Text = result
		self.expression = result
	    except:
		self.result.Text = "<<ERROR>>"
		self.expression = ""

按下Ctrl+Z 或 F6 回车退出IronPython 控制台

**Tutorial 总结**

IronPython 提供非常简单的方式开发应用程序，在动态和exploratory方式下. Windows Forms 与 Windows Presentation Foundation (Avalon) 都可以很容易的用最小步骤进行开发. 优势在于其为可视并且可在运行系统上动态修改而不需要重编译.

在本tutorial 中您执行了如下练习:

- Events and Delegates

- Windows Forms

- Windows Presentation Foundation (Avalon)

在本练习中, 您已熟悉了在IronPython中使用 delegates 和 handling events  - 开发交互式WinForms 或 Avalon应用的重要部分。接着您动态创建了交互式Windows Forms应用并创建了两个Windows Presentation Foundation应用. 

好了，今天的内容就先到这里了（其实到这里，还有一半内容）。

在下一篇中，将会介绍IronPython 中的 COM 协同工作能力。

原文 [http://www.cnblogs.com/daizhj/archive/2008/07/23/1247570.html](http://www.cnblogs.com/daizhj/archive/2008/07/23/1247570.html)
