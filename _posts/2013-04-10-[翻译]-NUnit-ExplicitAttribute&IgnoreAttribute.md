---
layout: post
category : lessons
tags : [NUnit]
---
{% include JB/setup %}

#**ExplicitAttribute (NUnit 2.2)**#

Explicit attribute 会忽略测试或者是测试 fixture，除非明确地选择了运行。对于 test or fixture，如果在GUI中选择了它，或者在命令行模式下通过指定它的名字来作为要运行的fixture，再或者它是通过组过滤器引用来的，那么它将会运行。

一个可选的字符串参数可用于提供标记测试为明确的原因。

如果test 或者 fixture 标记了Explicit attribute，那么在测试过程中，它将会被跳过，除非，特别地选中它，通过上述之一的方法。测试不会影响运行结果： 不会把它看作是忽略的，甚至不会计入到测试的总数。在GUI中，测试的树节点将会保持为灰色，状态栏的颜色也不会受到影响。

**注意**：*在NUnit 2.4之前的版本，这些测试会显示为忽略。*

**Test Fixture Syntax**

  namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture, Explicit]
	  public class ExplicitTests
	  {
	    // ...
	  }
	}

**Test Syntax**

	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture]
	  public class SuccessTests
	  {
	    [Test, Explicit]
	    public void ExplicitTest()
	    { /* ... */ }
	}
	
#**IgnoreAttribute (NUnit 2.0)**#

Ignore Attribute 是一个在一段时间内不运行某个 test 或者 test fixture 的 attribute。人们用 Ignore Attribute 来标记 Test 或者 TestFixture。然后运行的程序看到这个 attribute 后就会不运行这些个测试。如果某个测试没有运行，进度条将会变黄，且将会在报告中提示说这个测试没有运行。

这个功能通常用于临时性地不运行某个 test 或者 fixture。这和注释掉测试代码或者重命名方法比起来，是一个更好的机制。因为这样测试代码可以和其他代码一起编译，且在运行的时候还能指出哪个测试没有运行过。这就确保了不会遗漏测试。

**Test Fixture Syntax**

	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture]
	  [Ignore("Ignore a fixture")]
	  public class SuccessTests
	  {
	    // ...
	  }
	}

**Test Syntax**

	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture]
	  public class SuccessTests
	  {
	    [Test]
	    [Ignore("Ignore a test")]
	    public void IgnoredTest()
	    { /* ... */ }
	}

