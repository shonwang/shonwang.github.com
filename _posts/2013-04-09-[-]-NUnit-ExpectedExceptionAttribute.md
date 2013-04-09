---
layout: post
category : lessons
tags : [NUnit]
---
{% include JB/setup %}

翻译自[http://nunit.org/index.php?p=actionAttributes&r=2.6.2](http://nunit.org/index.php?p=actionAttributes&r=2.6.2)

#**ExpectedExceptionAttribute (NUnit 2.0 plus Updates)**#

这指定执行测试时将会抛出一个异常，这个 attribute 有一些位置的，命名的参数，我们将会根据它们的使用目的来分别讨论它们。

##**指定期望的异常类型**##

最初的 attribute，是从NUnit 2.0 引进的，其接受一个参数，此参数提供期望异常的精确类型。比如：

  [ExpectedException( typeof( ArgumentException ) )]
	public void TestMethod()
	{
	...

从NUnit 2.2.4开始，可以以字符串的形式指定异常的类型，
避免了去参考异常定义的程序集。

	[ExpectedException( "System.ArgumentException" ) )]
	public void TestMethod()
	{
	...

上述两个例子功能相同，测试只有在 System.Argument exception 抛出的时候才算通过，成功。

##**指定期望的消息**##

NUnit 2.1 引入了一个拥有第二个参数的构造函数，
其指定异常 message property 的精确文本信息，在 NUnit 2.2.4之后，the same extension was made to the constructor taking a string argument. With NUnit 2.4, these arguments are marked obsolete（过时）, and a named parameter is provided instead...

	// Obsolete form:
	[ExpectedException( typeof( ArgumentException ), "expected message" )]
	[ExpectedException( "System.ArgumentException", "expected message" )]
	
	// Prefered form:
	[ExpectedException( typeof( ArgumentException ), ExpectedMessage="expected message" )]
	[ExpectedException( "System.ArgumentException", ExpectedMessage="expected message" )]

在NUnit 2.4， 可以指定其他的异常测试消息， 除了简单的精确匹配。可以使用 MatchType 命名参数，它是一个枚举，定义如下：

	public enum MessageMatch
	{
	    /// Expect an exact match
	    Exact,	
	    /// Expect a message containing the parameter string
	    Contains,
	    /// Match the regular expression provided as a parameter
	    Regex,
	    /// Expect a message starting with the parameter string
	    StartsWith
	}

下面的例子只有当抛出一个 ArgumentException 异常且收到一条包含 “unspecified” 的消息才算通过。

	[ExpectedException( typeof( ArgumentException), ExpectedMessage="unspecified", MatchType=MessageMatch.Contains )]
	public void TestMethod()
	{
	...

如果指定 MatchType， 那就像之前一样使用精确匹配。

##**指定一条自定义的错误信息**##

在NUnit 2.4 如果对 ExpectedException attribute 不满意，可以指定显示一条自定义的消息。这需要通过 UserMessage 命名参数来完成。

	[ExpectedException( typeof( ArgumentException ), UserMessage="Custom message" )]
	public void TestMethod()
	{
	...

##**在代码中处理异常**##

如果一个异常的处理需求（processing required for an exception）太复杂而不能在 attribute 的声明中表现出来，通常做法是在测试代码中添加 try/catch 区块来处理它。作为另一种选择， NUnit2.4 允许指定一个方法并调用它来处理异常。当多个异常需要同一种方式处理的时候，这是特别有用的。

一个普通的异常处理程序可以通过实现定义如下的 **IExpectExceptionInterface** 来指定：

	public interface IExpectException
	{
	    void HandleException( System.Exception ex );
	}

此异常处理程序只有当方法标记了 **ExpectedException** attribute 时才会被调用。如果在代码中执行所有的检查——包括异常的类型，那么 attribute 可能会不需要指定一些为了指明期望异常的参数。

一个异常处理程序可能会用 **Handler** 命名参数来指定一个特别的方法。

	[ExpectedException( Handler="HandlerMethod" )]
	public void TestMethod()
	{
	...
	}
	
	public void HandlerMethod( System.Exception ex )
	{
	...
	}

这种技术可能不需要通过实现或结合 **IExpectException** 来使用。对于后者，这种被指定的异常处理程序适用于任何指定它的方法， 同时普通的异常处理程序适用于其他一些指定了 **ExpectedException** 的方法。

然而指定了它，这种异常处理方法就应该检查该异常，然后 **Assert** （断言）相关属性。在测试中，任何作为结果的失败的信息都和其他断言执行后的结果信息保持一致。

**示例**

	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture]
	  public class SuccessTests
	  {
	    [Test]
	    [ExpectedException(typeof(InvalidOperationException))]
	    public void ExpectAnExceptionByType()
	    { /* ... */ }
	
	    [Test]
	    [ExpectedException("System.InvalidOperationException")]
	    public void ExpectAnExceptionByName()
	    { /* ... */ }
	  }
	}



