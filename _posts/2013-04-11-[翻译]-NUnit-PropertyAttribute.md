---
layout: post
category : lessons
tags : [NUnit]
---
{% include JB/setup %}

#**PropertyAttribute (NUnit 2.4)**#

Property attribute 提供一种普遍的方法——使用 name/value 对来设置一些 test 或 fixture 的命名  properties 。

在下面的例子中， fixture 类 MathTests 提供一个值为723的Location，以及测试用例 AdditionTest 提供值为“Critical”的严重程度。

**示例**

  namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture, Property("Location",723)]
	  public class MathTests
	  {
	    [Test, Property("Severity", "Critical")]
		public void AdditionTest()
	    { /* ... */ }
	  }
	}

**使用注意**

NUnit 本身不会把 PropertyAttribute 用于任何目的， 但是其会显示在XML 输出文件 和 GUI的测试属性对话框。

可以写一些扩展去访问指定属性的值。也可以在测试中通过反射去访问属性的值。

##**自定义 Property Attributes**##

用户可以自定义派生自PropertyAttribute 且NUnit能识别的 attribute 。PropertyAttribute 提供一个受保护的 protected 构造函数，其接受一个 property 值，且设置此 property 的名称为被派生类的名称。NUnit本身就使用这项功能：它的一些专业的 attributes 实际上是特殊化的 **PropertyAttribute**

这是一个示例，其创建一个 Severity property。它就像其他 property 一样工作，除了有些更简单的语法和类型安全（is type-safe）。某个测试报告系统能够使用此 property 提供一个专门的报告。

	public enum SeverityLevel
	{
	    Critical,
	    Major,
	    Normal,
	    Minor
	}
	
	[AttributeUsage(AttributeTargets.Method, AllowMultiple=false)]
	public class SeverityAttribute : PropertyAttribute
	{
	    public SeverityAttribute( SeverityLevel level )
		    : base( level ); 
	}
	
	...
	
	[Test, Severity( SeverityLevel.Critical)]
	public void MyTest()
	{ /*...*/ }
	
从NUnit2.5开始，property attribute 能够包含多个 name/value 对。这种能力没有公开，但是可以通过被派生的 property 类来使用。 NUnit 的某些 attributes 使用这项功能。比如：请参阅 [RequiresThreadAttribute](http://nunit.org/index.php?p=requiresThread&r=2.6.2)



 
