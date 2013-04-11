---
layout: post
category : lessons
tags : [NUnit]
---
{% include JB/setup %}

#**RandomAttribute (NUnit 2.5)**#

**RandomAttribute** 用于指定一组随机值，提供给参数化测试方法的个别参数。 Since NUnit combines the data provided for each parameter into a set of test cases, data must be provided for all parameters if it is provided for any of them.

默认情况下， NUnit 是根据参数提供的所有数据点可能的组合来创建测试用例的——组合方法。这种默认情况可以通过为测试方法本身指定特定的 attribute 来修改。

RandomAttribute 支持下面的构造函数：

	public Random( int count );
	public Random( double min, double max, int count );
	public Random( int min, int max, int count );

**示例**

下面的测试将会执行15次，3个 x 的值， 与 5 个 -1.0 到 +1.0 双精度型随机数的每一次组合。

	[Test]
	public void MyTest(
	    [Values(1,2,3)] int x,
	    [Random(-1.0, 1.0, 5)] double d)
	{
	    ...
	}

#**RangeAttribute (NUnit 2.5)**#

RangeAttribute supports the following constructors:

	public RangeAttribute( int from, int to );
	public RangeAttribute( int from, int to, int step );
	public RangeAttribute( long from, long to, long step );
	public RangeAttribute( float from, float to, float step );
	public RangeAttribute( double from, double to, double step );


**示例**

The following test will be executed nine times, as follows:

	MyTest(1, 0.2)
	MyTest(1, 0.4)
	MyTest(1, 0.6)
	MyTest(2, 0.2)
	MyTest(2, 0.4)
	MyTest(2, 0.6)
	MyTest(3, 0.2)
	MyTest(3, 0.4)
	MyTest(3, 0.6)

	[Test]
	public void MyTest(
	    [Values(1,2,3) int x,
	    [Range(0.2,0.6,0.2] double d)
	{
	    ...
	}

#**RepeatAttribute (NUnit 2.5)**#

**RepeatAttribute** 用于指定需要执行多次的测试方法， 如果任意一次重复中出现了失败的结果，剩下的将不会执行，且失败的结果将被提交到报告中。

**注意：**

1. 不能在 TestFixture 和其他任意的 test suite 类型上使用RepeatAttribute。只有当个的 test 能够重复。

2. 因为某些参数化测试方法表现为 suit，当碰到这种方法时，RepeatAttribute 将会被忽略。

#**RequiredAddinAttribute (NUnit 2.5)**#

RequiredAddin 用于指定某个程序集，该程序集需要特定的插件才能保证功能正常。如果插件没有安装，那么整个程序集都标记为不可运行。

**注意：**
在 Alpha-3 release 版本中，此 attribute 能够用于类或者方法，这是有功能限制的，两个原因：

1. 如果方法或者类不被当作 test， 会导致插件丢失，NUnit将不会执行它。

2. 如果方法或者类被一些不同的插件处理，插件可能不会识别此 attribute。

**示例**

	[assembly: RequiredAddin("MyTestFixtureAddin")]
	[assembly: RequiredAddin("MyTestAddin")]
	[assembly: RequiredAddin("MyDecoratorAddin")]
	
	...
	
	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [MyTestFixture]
	  public class MyTests
	  {
	    [MyTest]
		public void SomeTest()
		{
		  ...
		}
	  }
	  
	  [TestFixture, MyDecorator]
	  public class MoreTests
	  {
	    [Test, MyDecorator]
		public void AnotherTest()
		{
		  ...
		}
	  }
	}

#**RequiresMTAAttribute (NUnit 2.5)**#

RequiresMTAAttribute 用于某个测试方法，类或程序集，用来指出这些测试需要在 multi-threaded apartment 下运行。如果父测试没有在MTA 下运行，那么将会创建一个新的线程。

**注意：**

在测试方法中，你可能也会使用 **MTAThreadAttribute** 。虽然此运行时只有在可执行程序集的进入点才能识别此 attribute，但是，许多用户期望它能够工作于测试中，所以我们把他看作一个 synonym。

**示例**

	// An MTA thread will be created and used to run
	// all the tests in the assembly
	[assembly:RequiresMTA]
	
	...
	
	// TestFixture requiring a separate thread
	[TestFixture, RequiresMTA]
	public class FixtureRequiringMTA
	{
	  // An MTA thread will be created and all
	  // tests in the fixture will run on it
	  // unless the containing assembly is
	  // already running on an MTA Thread
	}
	
	[TestFixture]
	public class AnotherFixture
	{
	  [Test, RequiresMTA]
	  public void TestRequiringMTA()
	  {
	    // A separate thread will be created for this test
		// unless the containing fixture is already running 
		// in the MTA.
	  }
	}

#**RequiresSTAAttribute (NUnit 2.5)**#

The RequiresSTAAttribute is used on a test method, class or assembly to specify that the tests should be run in the Single-threaded apartment. It causes creation of a new thread if the parent test is not already running in the STA.

Note: On test methods, you may also use the STAThreadAttribute. Although the runtime only recognizes this attribute on the entrypoint of an executable assembly, many users have expected it to work on tests, so we treat it as a synonym.
Examples


	// An STA thread will be created and used to run
	// all the tests in the assembly
	[assembly:RequiresSTA]
	
	...
	
	// TestFixture requiring a separate thread
	[TestFixture, RequiresSTA]
	public class FixtureRequiringSTA
	{
	  // An STA thread will be created and all
	  // tests in the fixture will run on it
	  // unless the containing assembly is
	  // already running on an STA Thread
	}
	
	[TestFixture]
	public class AnotherFixture
	{
	  [Test, RequiresSTA]
	  public void TestRequiringSTA()
	  {
	    // A separate thread will be created for this test
		// unless the containing fixture is already running 
		// in the STA.
	  }
	}

#**RequiresThreadAttribute (NUnit 2.5)**#

The RequiresThreadAttribute is used to indicate that a test method, class or assembly should be run on a separate thread. 可选地， 能够在构造函数中指定期望的线程 apartment 。

**注意：**
此 attribute，使用或不使用 ApartmentState 参数都会导致创建新的线程。只想在当前 ApartmentState 不被占用的时候，创建一个线程，请使用 **RequiresSTAAttribute** 或者 **RequiresMTAAttribute**。

**示例：**

	// A thread will be created and used to run
	// all the tests in the assembly
	[assembly:RequiresThread]
	
	...
	
	// TestFixture requiring a separate thread
	[TestFixture, RequiresThread]
	public class FixtureOnThread
	{
	  // A separate thread will be created and all
	  // tests in the fixture will run on it.
	}
	
	[TestFixture]
	public class AnotherFixture
	{
	  [Test, RequiresThread]
	  public void TestRequiringThread()
	  {
	    // A separate thread will be created for this test
	  }
	  
	  [Test, RequiresThread(ApartmentState.STA)]
	  public void TestRequiringSTAThread()
	  {
	    // A separate STA thread will be created for tnis test.
	  }
	}
	
