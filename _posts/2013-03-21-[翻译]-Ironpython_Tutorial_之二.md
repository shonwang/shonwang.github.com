---
layout: post 
category: lessons
tags: [IronPython]
---
{% include JB/setup %}

[翻译] python Tutorial 之二
在上一篇中，简单介绍了IronPython的相关配置和控制台使用方面的内容，今天继续翻译接下来的内容。

**Exercise 3: 加载 .NET 库**

IronPython 仅能够直接引用一些最通用的 .NET 库。为了引用其它的 .NET 库, 需要显式引用. IronPython维护着一个引用的列表 (请查看在 Task 1 的 clr.References). 为了添加.net 引用，使用内置的“clr”模块方法:

`clr.AddReference` 用于直接添加.NET引用，或指定文件明或编译名称(完整或部分). 这个方法主要提供交互性的控索(interactive exploration). 我们推荐在代码模块中，使用如下方法函数, 因为它们会对加载的编译库提供更多的控制。

`clr.AddReferenceToFile` 添加对指定文件的引用，这个方法与加载的编译版本无关。最后, 它不确保正确的编译版本被加载。为了确保加载正确的编译版本, 请使用 clr.AddReferenceByName. 而且AddReferenceToFile要求编译（库）直接定位在sys.path的路径列表下.

`clr.AddReferenceToFileAndPath` 提供与AddReferenceToFile相似的功能. 不同之处在于它接受绝对路径。并且在加载之前，AddReferenceToFileAndPath 会添加文件路径到sys.path.

`clr.AddReferenceByName` 添加对指定完整assembly名字的引用。比如:`'System.Xml, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'`.

`clr.AddReferenceByPartialName` 添加对指定“部分”的assembly名称的引用. 这个方法不确保被加载的assembly版本正确。使用 clr.AddReferenceByName 添加对指定版本的引用.

**Task 1: 添加 System.Xml 引用**

在tutorial路径下启动 IronPython 控制台 (see Introduction for details).

为了引用 System.Xml, 首先要引用 Xml 组件到 IronPython.  使用下面代码来添加 System.Xml引用(您可在敲入"clr.References" 之前或之后，添加 clr.AddReference 代码看一下其中的变化):

	import clr
	clr.AddReference("System.Xml")
	from System.Xml import *
	dir()

注意clr.AddReference 函数即接受 System.Reflection.Assembly 对象，也接受“字符串”做为参数. 符串可以是一个完整的编译名称，部分名称或文件名. 为了对对编译引用施加更多的控制, 可适当引用上述函数.例如, 考虑如下的交替声明，对应上面的clr.AddReference("System.Xml"):

	clr.AddReferenceByName('System.Xml, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089')
	clr.AddReferenceByPartialName("System.Xml")

加载XML文件"load.xml" 到 XmlDocument. 这个 xml 文件包括IronPython 示例游戏"Puzzle" 的保存数据。为清楚起见，load.xml 文件被放置在Tutorial 路径下

	d = XmlDocument()
	d.Load("load.xml")

我们可以查询该文档。使用如下代码查询保存的“游戏”（数据）:

	n = d.SelectNodes("//Puzzle/SavedGames/Game/@caption")
	for e in n: print e.Value

The output in the console window will be:

	>>> n = d.SelectNodes("//Puzzle/SavedGames/Game/@caption")
	>>> for e in n: print e.Value
	...
	Seattle (default game)
	New York
	World
	North America

(选项) 引用 "xmlutil.py" 模块放在Tutorial 路径下，然后使用模块方法来遍历Xml 文件的内容:

	import xmlutil
	for e in xmlutil.Walk(d): print e.Name, e.Value
	#document None #comment ************************************************************************* * * Copyright (c) Microsoft Corporation.  * * This source code is subject to terms and conditions of the Microsoft Public License. A  * copy of the license can be found in the License.html file at the root of this distribution. If  * you cannot locate the Microsoft Public License, please send an email to  * ironpy@microsoft.com. By using this source code in any fashion, you are agreeing to be bound  * by the terms of the Microsoft Public License. * * You must not remove this notice, or any other, from this software. * * * ***************************************************************************  Puzzle None SavedGames None Game None caption Seattle (default game) type a y 714 x 327 level 11 dimension 3 Game None caption New York type r y 1538 x 1205 level 12 dimension 3 Game None caption World type h y 0 x 0 level 2 dimension 4 Game None caption North America type a x 2 y 5 level 4 dimension 3 TopLeftPreviewTile None x -3 y -3 dimension 3 level 5 Cache None allow true

遍历方法是一个生成器[generator] (Python 方法包含 "yield" 声明). 在遍历方法执行时, 它逐个返回(yields) XML 结点数据给调用者[caller]. 遍历代码如下:

	def Walk(xml):
	    yield xml
	    if hasattr(xml, "Attributes"):
		attrs = xml.Attributes
		if attrs:
		    for attr in attrs:
		        yield attr
	    for child in xml:
		for c in Walk(child):
		    yield c

按下Ctrl+Z 或 F6 回车键退出python 控制台。

**Task 2: Mapack - 加载.NET 库 - AddReferenceToFile**

这个任务要求 Mapack.dll 库来进行线性代数运算（linear algebra computations）.  这个库并不是IronPython发布的一部分. See prerequisites for download details.

启动tutorial 路径下的IronPython 控制台.(see Introduction for details).

使用 clr.AddReferenceToFile 函数加载矩阵[Matrix]库 "Mapack.dll":

	import clr
	clr.AddReferenceToFile("Mapack.dll")
	from Mapack import *
	dir()

控制台显示如下输出:

	>>> import clr
	>>> clr.AddReferenceToFile("Mapack.dll")
	>>> from Mapack import *
	>>> dir()
	['CholeskyDecomposition', 'EigenvalueDecomposition', 'LuDecomposition', 'Matrix', 'QrDecomposition','SingularValueDecomposition', '__builtins__', '__doc__', '__name__', 'clr']

创建 Matrix 类实例:

	m = Matrix()

哦(Oops), 错误的构造函数参数.  下一步您将会了解构造函数的有效方式.

	Traceback (most recent call last):
	File , line unknown, in Initialize##30
	TypeError: Matrix() takes at least 1 argument (0 given)

使用__doc__ 属性, 查看关于Matrix 构造函数的信息:

	print Matrix.__new__.__doc__
	>>> print Matrix.__new__.__doc__
	__new__(cls, int rows, int columns)
	__new__(cls, int rows, int columns, float value)
	__new__(cls, Array[Array[float]] value)

通过正确的使用构造函数，并手工正确的设置矩阵数据来创建Matrix 类实例。IronPython 支持.NET类自定义索引[custom indexing].

	m = Matrix(2, 2, 1.2)
	n = Matrix(2,1)
	n[0,0] = 4
	print m
	print n
	>>> m = Matrix(2, 2, 1.2)
	>>> n = Matrix(2,1)
	>>> n[0,0] = 4
	>>> print m
	1.2 0
	0 1.2 
	>>> print n
	4
	0 

(可选项) IronPython 也支持操作符重载.  矩阵重载操作符+, - (二进制和一元) 和 *.  您能看到操作符表示 (__add__, __mul__, __sub__, ...) :

	dir(m)

简单的矩阵复数（matrices）运算:

	m * n
	n.Transpose() * m
	m * 3
	n + -n

控制台显示如下输出:

	>>> m * n
	Mapack.Matrix
	>>> n.Transpose() * m
	Mapack.Matrix
	>>> m * 3
	Mapack.Matrix
	>>> n + -n
	Mapack.Matrix
	TypeError: Matrix() takes at least 1 argument (0 given)

按下Ctrl+Z 或 F6 回车退出 Python 控制台

**Exercise 4: 获取和使用Python 标准库**

在这个练习中，您将获取并指向 Python 标准库.

**Task 1: 配置IronPython，使用Python 标准库**

从 http://www.python.org/download/ 下载最新的Python （安装包）installer 并进行安装.  其余的练习将假设您已使用默认设置(比如：安装到c:"python25).

创建一个文件，将其命名为"site.py" 。然后将它放在IronPython "Lib" 路径下.  如果它已经存在, 您需要编辑它. "site.py" 文件会在每次运行 IronPython 时运行. 告之IronPython （Python）标准类库所在位置。您可以添加Python的"lib"路径到IronPython 的路径下. 这样做了之后，将下面代码敲入到"site.py" 文件。(用实际路径替换 “c:"python25"lib” 路径):

	import sys
	sys.path.append(r"c:"python25"lib")

在tutorial路径下启动python控制台.(see Introduction for details).

现在您可以在IronPython使用 Python 标准库, 比如获取当前的工作路径 (output uses assumed location of your IronPython installation):

	import os
	os.getcwd()
	'C:""ironpython""Tutorial'

**Tutorial（阶段）总结:**

在这部分tutorial 中您已进行了如下练习.

- IronPython 交互控制台

- 在IronPython中使用标准.net类库

- 加载.NET 库

- 获取并使用Python 标准类库

在本tutorial中, 您已熟悉了IronPython 交互控制台, 包括使用dir()方法动态控索（dynamic exploratory）。
并且使用“__doc__” 属性显示环境.  您也了解了在IronPython中加载和引用.NET 库 (使用引用声明), 创建.NET
类实例(包括泛型类), 调用方法，遍历.NET 集合, 以及重载 .NET 对象重载操作符. 最终, 您还了解了如何访问标准
Python 类库。

好了，今天的内容就先到这里了（其实到这里，还有一半内容）。

在下一篇中，将会介绍IronPython 高级篇（Advanced ）

[原文](http://www.cnblogs.com/daizhj/archive/2008/07/22/1247557.html)
