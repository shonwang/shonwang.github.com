---
layout: post 
category: lessons
tags: [IronPython]
---
{% include JB/setup %}

[翻译] python Tutorial 之一

**声明**：本文做为IronPython-2.0 B3的Tutorial 中文译文，内容全部来自其英文原文，其中本人认为存在疑问的或翻译不当之处会用原文中的内容加以标记，且本文内容完全用于研究和学习IronPython 之用，限于本人英文翻译功底有限，如有错误，欢迎大家批评指正。

原文链接如下：[http://www.cnblogs.com/daizhj/articles/1245093.html](http://www.cnblogs.com/daizhj/articles/1245093.html)

**Introduction**

IronPython 作为 Python 语言在.net平台上的实现(www.python.org)，它是一种动态语言且支持许多编程范例[paradigms] ，诸如面向对象，也允许您使用.net代码进行编译。本tutorial的目标就是使您快速熟悉IronPython 控制台（console），并向您展示如何有效的使用扩展（extensive）.net库。这个 tutorial同时还会展示如何从一些特殊领域入手，这些领域诸如COM的内部操作，用C#扩展 IronPython ，以及嵌入式的IronPython 等。当然本tutorial决不是针对Python语言自身进行介绍，如果您想查阅这方面的资料，我们推荐您去www.python.org  或 看一下由Mark Lutz 与 David Ascher 合著的“Learning Python”这本书。 

本tutorial所需要的运行环境如下：

•Microsoft .NET Framework Version 2.0 Redistributable Package (x86) •Required to run IronPython. 

译者注：本人机器上是.net3.5，也可运行部分的示例，所以大家不必非要下载2.0

•.NET Framework 2.0 Software Development Kit (SDK) (x86) 

•Required for the COM interoperability, extending and embedding tutorials. 

•Microsoft WinFX Runtime Components (Avalon) 

•Required for "Advanced IronPython" and "Embedding IronPython" tutorials. 

•Mapack (example assembly found on the internet) 

•Required for the "Basic IronPython" tutorial, exercise "Loading .NET Libraries". 

•Extract Mapack.dll from the zip file directly into the Tutorial directory. 

译者注:如果您本机上安装的是vs2008 RTM（笔者的开发环境）,那么你就可以跳过上面的几个安装

直接进入到下面的正文了，而如果您安装的是vs2005，那么请您检查相应的条件是否已不足。
  
本tutorial假设IronPython的发布包已被解压缩至C:\IronPython. 请注意您个人的安装可能会存在一定的差异。

本tutorial假设您将会使用IronPython 控制台功能。将tutorial指导您从tutorial目录启动控制台程序时，您应该定位到tutorial目录下(>cd c:\ironpython\tutorial) ，并运行控制台程序(>..\ipy.exe)。

译者注：ipy.exe可以下面链接中直接下载，我这样做主要是觉得园子里有些朋友可以只是想了解并试用一下，并不想下载那个IronPython的源码包或相应的sdk等资源。

好了，让我们开始正文吧。

**Tutorial 1: 基本IronPython应用**

本章节主要强调的是IronPython的interpreter ，以及与.NET 类库的互动场景（environment ）

本章节阅读大概需要30分钟 

The objective of this tutorial is to launch the IronPython interpreter, explore the environment of the interactive console and use IronPython to interact with .NET libraries.
The exercises in this tutorial are:

•The IronPython interactive console

•Using the standard .NET libraries from IronPython

•Loading additional .NET libraries


**练习 1: IronPython 交互控件台**

In this exercise, you will start the IronPython interactive interpreter and perform simple tasks to become acquainted with the IronPython environment. If you are familiar with using the Python 
interactive console, the import statement and exploring the Python interactive environment using dir() function and __doc__ attribute, you can skip this exercise.

**任务 1: IronPython console**

1.从本Tutorial的路径下启动IronPython console（c:"ironpython"ipy.exe）。它将如在DOS窗口下出现如下提示

	IronPython 1.0 on .NET 2.0.50727.42
	 Copyright (c) Microsoft Corporation. All rights reserved.
	>>> _

2.运行下面的内容（在每一条内容之后，按下回车键都会打印（prints）一条结果）。 

	2+2
	print "Hello World!"
	for i in range(3): print i
	x = 10
	print x

 运行完上面操作后，控制台窗口将会显示如下内容：

	>>> 2+2
	4
	>>> print "Hello World!"
	Hello World!
	>>> for i in range(3): print i
	...
	0
	1
	2
	>>> x = 10
	>>> print x
	10
	>>> 
	
3.IronPython 控制台支持多行声明（multi-line statements），这经常要进行函数定义:
...

Unlike C# or Java, where blocks of code are grouped by curly brackets "{...}", blocks of code in Python are grouped based on their level of indentation.  Every new block of code must be indented one more level than the previous block of code.  Blocks of code are used for function and class definitions as well as 'if' statements and loops.

例如：定义“add”方法（注：你需要在return 之前敲入空格符）:

	return a + b

要完成函数定义，按下Enter键。

	add(3, 2)
	add("Iron", "Python")

完成这一步后，控制台内容如下：

	>>> def add(a, b):
	...     return a + b
	...
	>>> add(3, 2)
	5
	>>> add("Iron", "Python")
	'IronPython'
	>>> 

4.要退出控制台程序，按下Ctrl+Z 并回车即可(alternatively, press F6 followed by Enter).

	^Z

**Task 2: 内置模块和交互式探查(interactive exploration)**

1.从tutorial路径启动IronPython 控制台 (see Introduction for details).

2.使用内置dir() 函数, 列出IronPython 环境的内容:

	dir()

控制台显示如下:

	>>> dir()
	['__builtins__', '__doc__', '__name__']

3.IronPython 有几个内置模块，经常使用的是"sys". 引入"sys" 模块使用 "import" 声明:

	import sys

4.Python 引入声明与 C# 的 "using" 或Visual Basic的"Imports"声明相似。The important difference is that the C# and VB statements bring the names from the imported namespace into the global namespace to be accessed directly. Python’s import doesn’t do that. 

要访问引入模块的名称和属性, 使用module名称的前缀:

	sys.version

5.使用dir()方法去显示环境:

	dir()

改变环境 (global namespace), 现在它包括"sys" 模块:

	>>> dir()
	['__builtins__', '__doc__', '__name__', 'sys']

6.使用dir() 方法来显示"sys" 模块的内容:

	dir(sys)

7.打印出 "sys" 模块属性的一些值：

	sys.path
	sys.executable

**Task 3: 外部 Python 模块**

本任务使用 "first.py" （文件）模块位于Tutorial 路径.

1.声明 "first.py" 模块（您可以运行在Tutorial 路径下的ipy.exe，“first”将位于sys.path路径下）:

	import first

2.用dir()函数显示"first" 模块:


	dir(first)
	>>> dir(first)
	['__builtins__', '__file__', '__name__', 'add', 'factorial', 'hi']

3.为打印 "add" 和 "factorial" 函数文档内容，使用__doc__属性：

	first.add.__doc__
	first.factorial.__doc__

__doc__属性可用于显示 .NET 方法和它们的参数类型。

	>>> first.add.__doc__
	'add(a, b) - returns a + b'
	>>> first.factorial.__doc__
	'factorial(n) - returns factorial of n'

4.调用 "first" 模块方法并打印“hi”属性内容

	first.add(1,2)
	first.factorial(5)
	first.hi

输出内容为：

	>>> first.add(1,2)
	3
	>>> first.factorial(5)
	120
	>>> first.hi
	'Hello from IronPython!'

5.在控制台下Ctrl+Z 或 F6 按回车退出。


**Exercise 2: 使用标准.NET 库**

IronPython 的能力在于它可以无缝访问.NET 类库。这个练习将会示范如何使用（标准）.NET 类库。

**Task 1: 基本.NET 类库使用**

1.在tutorial路径下启动IronPython 控制台 (see Introduction for details).

2.使用 "import" 声明, 引入 .NET 系统名空间:

	import System

3.显示System.Environment 类并访问其下面的属性：:


	dir(System.Environment)
	System.Environment.OSVersion
	System.Environment.CommandLine

如下是其命令行的输入：

	>>> dir(System.Environment)
	>>> System.Environment.OSVersion
	<System.OperatingSystem object at 0x000000000000002B [Microsoft Windows NT 6.0.6000.0]
	>>> System.Environment.CommandLine
	'C:""IronPython""ipy.exe'

4.使用“from ... import ..." 引入相应类库声明，并使用dir()显示全局名空间内容。

	from System.Math import *
	dir()

现在您可以调用Math 方法，而不必指定名空间和类前缀：   

	Sin(PI/2)

显示输出为:

	>>> from System.Math import *
	>>> dir()
	['Abs', 'Acos', 'Asin', 'Atan', 'Atan2', 'BigMul', 'Ceiling', 'Cos', 'Cosh', 'DivRem', 'E', 'Equals', 'Exp', 'Floor', 'GetHashCode', 'GetType', 'IEEERemainder', 'Log', 'Log10', 'Max', 'Min', 'PI', 'Pow', 'Round', 'Sign', 'Sin', 'Sinh', 'Sqrt', 'System', 'Tan', 'Tanh', 'ToString', 'Truncate', '__builtins__', '__doc__', '__name__']
	>>> Sin(PI/2)
	1.0

**Task 2: 使用.NET 类**

1.声明“System.Collections”名空间到“全局名空间”内：

	from System.Collections import *

2.创建 Hashtable 类实例并使用dir()显示实例的内容：

	h = Hashtable()
	dir(h)

3.在hash表中插入一些元素:

	h["a"] = "IronPython"
	h["b"] = "Tutorial"

IronPython 支持C# 风格语法来访问hash表元素. 相同的语法可用于支持索引的对象(Arrays, Array lists etc):

	h["a"]

上面的操作步骤将会显示如下输出:

	>>> h["a"] = "IronPython"
	>>> h["b"] = "Tutorial"
	>>> h["a"]
	'IronPython'

4.使用声明"for ... in ..." 列举hash表的内容. hash 表元素作为"DictionaryEntry" 类实例. 打印"Key"和"Value"属性:

	for e in h: print e.Key, ":", e.Value

控制台显示如下输入:

	>>> for e in h: print e.Key, ":", e.Value
	...
	a : IronPython
	b : Tutorial

5.你可以向python内置列表传递参数来初始化集合类. 您可以通过[1,2,3]来创建指定元素来创建Python 列表[1,2,3]. You create tuples by specifying elements in the parentheses: (1,2,3).

	l = ArrayList([1,2,3])
	for i in l: print i
	s = Stack((1,2,3))
	while s.Count: s.Pop()

输出如下:

	>>> l = ArrayList([1,2,3])
	>>> for i in l: print i
	 ...
	1
	2
	3
	>>> s = Stack((1,2,3))
	>>> while s.Count: s.Pop()
	...
	3
	2
	1

**Task 3: 泛型**

1.从System.Collections.Generic名空间中，来声明泛型集合类:

	from System.Collections.Generic import *

2.初始化泛型类实例, 其泛型类型参数必须被指定.  IronPython 使用如下语法来指定类型参数: 

generic_type[type_argument, ...] （泛型类型[类型参数]）.  创建泛型string列表实例:

	l = List[str]()

3.添加字符串数据到列表中（ 因为创建了字符串列表，可以进行添加）:

	l.Add("Hello")
	l.Add("Hi")

4.尝试添加其它类型的数据:

	l.Add(3)
	l.Add(2.5)
	l.Add([1,2,3])

显然，添加“非字符串”型数据会导致如下“类型错误”输出:

	>>> l.Add(3)
	Traceback (most recent call last):
	File , line unknown, in Initialize##40 
	TypeError: expected str, got int 

5.遍历泛型集合:

	for i in l: print i
	The output will be:
	>>> for i in l: print i
	...
	Hello
	Hi

	TypeError: expected str, got int TypeError: expected str, got float TypeError: expected str, got list

6.按下Ctrl+Z 或 F6 回车，退出IronPython 控制台。

好了，今天就先翻译到这里了，看了看原文，发现刚翻译了1/4, 看来要再用2-3个篇幅来完成全篇翻译。我想等全部翻译结束后，将翻译的各篇文章拼揍起来。

控制台程序下载链接：[http://www.cnblogs.com/files/daizhj/ipy.zip](http://www.cnblogs.com/files/daizhj/ipy.zip)

原文: [http://www.cnblogs.com/daizhj/archive/2008/07/21/1247345.html](http://www.cnblogs.com/daizhj/archive/2008/07/21/1247345.html) 

