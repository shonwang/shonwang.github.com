---
layout: post
category : lessons
tags : [NUnit]
---
{% include JB/setup %}

翻译自[http://nunit.org/index.php?p=actionAttributes&r=2.6.2](http://nunit.org/index.php?p=actionAttributes&r=2.6.2)

#**CategoryAttribute (NUnit 2.2)**#

Category attribute 提供了一个可供选择的suites来处理测试组（groups of tests）。单独的测试用例或者 fixtures 可能被会被识别为一个特定的分类。GUI和控制台测试运行者允许指定一个分类列表来引入或排除运行。当使用了分类以后，只有在分类中选中了的测试才会运行。这些在分类中没有选中的测试是根本不会被提到的。

这项功能可以通过在控制台中使用/include 和 /exclude 参数以及GUI中的“Categories”的标签来理解。GUI提供一个虚拟的指示，其指示了何时选择了何种类别。

**Test Fixture 语法**

  namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture]
	  [Category("LongRunning")]
	  public class LongRunningTests
	  {
	    // ...
	  }
	}

**Test 语法**

	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture]
	  public class SuccessTests
	  {
	    [Test]
	    [Category("Long")]
	    public void VeryLongTest()
	    { /* ... */ }
	}

##**自定义 Category Attributes**##

从 NUnit 2.4 开始，定义派生自**CategoryAttribute** 的自定义 Category Attributes 以及让NUnit识别它们都变成了可能。默认情况下，CategoryAttribute 的构造函数的分类名称（category name）是你类（class）的名字。

这里是一个例子，创建了一个 “Critical ” 分类的测试，它其他分类一样工作，只是用了更简单的语法。测试报告系统可能会该 attribute 来提供特定的报告。

	[AttributeUsage(AttributeTargets.Method, AllowMultiple=false)]
	public class CriticalAttribute : CategoryAttribute { }
	
	...
	
	[Test, Critical]
	public void MyTest()
	{ /*...*/ }
