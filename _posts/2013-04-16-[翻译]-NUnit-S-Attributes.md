---
layout: post
category : lessons
tags : [NUnit]
---
{% include JB/setup %}

#**SequentialAttribute (NUnit 2.5)**#

**SequentialAttribute** 用于在某个测试中，来告诉NUnit
通过选择由测试参数提供的个别数据项，且没有生成其他额外的情况下，来生成测试用例。

**注意：**如果参数数据由多个 attribute 提供，NUnit使用这些数据项的顺序是不能保证的。可是它可以在给定的运行时，操作系统下保持不变，如期望的那样。

**示例：**

下面的测试将会被执行3次，如下：

  MyTest(1, "A")
	MyTest(2, "B")
	MyTest(3, null)

	[Test, Sequential]
	public void MyTest(
	    [Values(1,2,3)] int x,
	    [Values("A","B")] string s)
	{
	    ...
	}

#**SetCultureAttribute (NUnit 2.4.2)**#

SetCulture attribute用来为某个测试持续的时间段来设置当前文化背景。它能够在 test 或者 fixture 层级使用。该文化背景保持直到相应的 test 或者 fixture 完成，然后重置到它的初始值。如果你想要使用当前文化背景设置来决定是否运行某一个测试，使用 Culture attribute 来代替。

只能指定一个文化背景。在多个文化背景下运行测试已经在未来的增强计划中了。现在你能够封装你的测试代码到一个能够被每一个单独测试方法调用的私有方法中，从而得到相同的结果。

**示例**

	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture]
	  [SetCulture("fr-FR")]
	  public class FrenchCultureTests
	  {
	    // ...
	  }
	}

#**SetUpAttribute (NUnit 2.0 / 2.5)**#

此 attribute 用在 [TestFixture](http://nunit.org/index.php?p=testFixture&r=2.6.2) 中， 它提供一组普通的在每个测试方法调用前都必须执行的功能。它也可以用在 [SetUpFixture](http://nunit.org/index.php?p=setupFixture&r=2.6.2) 它提供和上述相同的功能，只不过它是在命名空间或者程序集的层级。

**NUnit 2.5 之前**， 一个类只能有一个 SetUp 方法且还需要一个接口方法。

**从NUnit2.5开始**，SetUp 方法既可以是静态方法，也可以使接口方法，且你能够在一个 fixture 定义多个该方法。一般来说，多个 Setup 方法只能在不同的继承层次结构中定义， 如下解释。

如果 SetUp 方法失败或者抛出一个异常，测试不会执行，将报告失败或者错误。

**示例**

	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture]
	  public class SuccessTests
	  {
	    [SetUp] public void Init()
	    { /* ... */ }
	
	    [TearDown] public void Cleanup()
	    { /* ... */ }
	
	    [Test] public void Add()
	    { /* ... */ }
	  }
	}

**继承**

SetUp attribute 可以继承自任意基类，所以， 如果一个基类定义了一个 SetUp 方法，此方法将会在派生类中的每一个测试方法执行之前被调用。

NUnit 2.5 之前，你只能有一个 SetUp 方法。如果你想要在基类中有一些 SetUp 的功能，且在派生类中添加更多，你需要自己调用此基类的方法。

在 NUnit 2.5 中， you can achieve the same result by defining a SetUp method in the base class and another in the derived class. NUnit 将会在这些派生类之前调用基类的 SetUp 方法。

**注意：**
虽然可以在同一个类中定义多个SetUp方法，但是你应该很少这么使用。不像在继承层次结构分开的类中定义方法，它们执行的顺序是不能保证的。

#**SetUpFixtureAttribute (NUnit 2.4)**#

此 attribute 用来标记某个类，该类包括一次性的作用于给定的命名空间中所有 test fixture 的 setup 或者 teardown 方法。该类至少包括一个标记了 SetUpAttribute 的方法和一个标记了 TearDownAttribute 的方法。

在使用 setup fixture 的类中有一些限制：

- 它必须是公开的，否则NUnit看不见它。

- 它必须有一个默认的构造函数，否则 NUnit 将不能构造它。

在某个 SetUpFixture 中的 SetUp 方法在它命名空间中任意fixtures 之前都会执行一次。TearDown 方法在所有 fixtures 完成执行后都会执行一次。在下面的例子中， RunBeforeAnyTests() 方法在NUnit Tests 命名空间中任意的 tests 或者 setup 方法之前被调用，RunAfterAnyTests() 方法在命名空间中所有测试，也就是这些测试中的个体或  fixture teardowns 完成执行后被调用。

在给定的命名空间中只能创建一个SetUpFixture。A SetUpFixture outside of any namespace provides SetUp and TearDown for the entire assembly. 

**示例**

	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [SetUpFixture]
	  public class MySetUpClass
	  {
	    [SetUp]
		RunBeforeAnyTests()
		{
		  // ...
		}
	
	    [TearDown]
		RunAfterAnyTests()
		{
		  // ...
		}
	  }
	}


#**SuiteAttribute (NUnit 2.0/2.4.4)**#

Suite Attribute 用于定义从命令行使用 **/fixture** 选项运行的测试子集，它是从 NUnit 2.0 引进的，用来代替从TestSuite 类继承的这个老办法。

最初, NUnit 的开发人员相信，由于命名空间中 suites 的动态创建，suites 机制的需求将会减少。它提供了向后兼容。

这并没有被证明是真的。今天仍然有很多人使用Suites， 所以我们正在努力恢复它们的可用性。

Suite 机制依赖于标记了 SuiteAttribute 的静态 property。在经典实现中，所有2.0以后的版本都支持它，该 property 返回一个 TestSuite， 其包含了要执行的 tests。

**老方法**

	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	  using NUnit.Core;
	
	  public class AllTests
	  {
	    [Suite]
	    public static TestSuite Suite
	    {
	      get
	      {
	        TestSuite suite = new TestSuite("All Tests");
	        suite.Add(new OneTestCase());
	        suite.Add(new Assemblies.AssemblyTests());
	        suite.Add(new AssertionTest());
	        return suite;
	      }
	    }
	  }
	}

这个方法有一个严重的问题： 它需要引用 nunit.core 程序集，用户在测试的时候通常不引用这个程序集。这意味着在不再次编译的情况下，tests 不能移植到NUnit的各个版本上。在某些情况下，引入一个以上版本的 core assembly 会导致程序出错。

从 NUnit 2.4.4 开始， 有了一个可用的新办法。标记了 SuiteAttribute 的 property 可能会简单的返回一个包含了 test fixtrue 对象或者类型的集合。如果返回了一个类型，NUnit会创建一个该类型的对象用作 fixture。这样做可以把参数化的构造函数，或者可变 properties的对象用作 fixture 

通过使用 SuiteAttribute 创建的Test suites 可以包含 TestFixtureSetUp 和 TestFixtureTearDown 方法，为引用了该suites 的测试执行一次性的设置和 teardown。

**新方法 - Fixture 对象**

	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  private class AllTests
	  {
	    [Suite]
	    public static IEnumerable Suite
	    {
	      get
	      {
	        ArrayList suite = new ArrayList();
	        suite.Add(new OneTestCase());
	        suite.Add(new AssemblyTests());
	        suite.Add(new NoNamespaceTestFixture());
	        return suite;
	      }
	    }
	  }
	}

**新方法 - Fixture 类型**

	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  private class AllTests
	  {
	    [Suite]
	    public static IEnumerable Suite
	    {
	      get
	      {
	        ArrayList suite = new ArrayList();
	        suite.Add(typeof(OneTestCase));
	        suite.Add(typeof(AssemblyTests));
	        suite.Add(typeof(NoNamespaceTestFixture));
	        return suite;
	      }
	    }
	  }
	}

**限制**

NUnit 支持用户定义的 Suites，有两个限制：

1. 在某个 Suite 中使用新方法的时候，不能直接引入个别的测试。任何想要如此做的人，需要使用老方法，且创建一个派生自NUnit.Core.TestCase. 这是不推荐的，因为它需要引用 core assembly。

2. Suites 目前是不能显示在 GUI 中的，或者当任意一种运行工具遇到它们的时候，是不会自动运行。Suite 机制的历史目的是用来提供一种方法，该方法能够在每次运行的最高等级聚集测试。因此，在控制台或者GUI命令行上，它们只有在使用 /fixture 选项的时候才能支持。

解决这些限制的方法已经作为未来NUnit版本计划的一部分在研究中了。




