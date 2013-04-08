---
layout: post
category : lessons
tags : [NUnit]
---
{% include JB/setup %}

翻译自[http://nunit.org/index.php?p=actionAttributes&r=2.6.2](http://nunit.org/index.php?p=actionAttributes&r=2.6.2)

#**Action Attributes (NUnit 2.6)**#

**Action Attributes** 是NUnit的一项功能，它设计用来更好的实现可组合性的测试逻辑（composability of test logic）。通常我们在做单元测试的时候会有这样的逻辑：我们会想要在测试周期中运行某些事件（比如： SetUp, TearDown, FixtureSetUp, FixtureTearDown, etc.）NUnit 已经可以通过用适当的NUnit提供的属性（ attributes）装饰 fixture类和方法来执行上述事件的代码。**Action Attributes** 允许用户在测试运行之前或之后创建一些自定义属性（ attributes）去封装一些指定的动作（Action）以供使用；

## **可组合性的问题 （The Problem of Composability）**##
假设我们在许多 fixtures 上有一些测试，运行它们每一个测试时，都需要创建和销毁一个相同的内存测试数据库。我们可以创建一个 fixtures 的基类，然后让依赖于测试的每一个 fixture 从这个基类派生。作为另一种选择，我们能够在每一个 fixture 共享的普通命名空间层级创建一个 **SetUpFixture**。

直到我们需要一些其他的可复用的功能之前，这样做很不错。 比如说配置或重置一个 ServiceLocator 。我们可以让 一个 fixture 基类或者是 SetUpFixture 来完成这个功能，但是现在我们把两个不同的职责混在一个基类里面了。至于 SetUpFixture，这仅仅适用于假设所有类要求的功能都放置在一个普通的命名空间中。在某些情况下，我们可能**不**想要测试数据库， 但是我们想配置 ServiceLocator；有些时候却又恰恰相反，另一些些时候甚至两者都想要，所以我们必须对这个基类做一些配置。

如果我们现在发现第三种我们需要复用的功能，比如用任意一种方法配置线程的 CurrentPrincipal ，解决方案的复杂程度将迅速提升。我们已经违反了单一责任原则（Single Responsibility Principle）并且承受着它所带来的痛苦。

我真正所想要是能够将不同的可复用测试逻辑分开 然后又能够根据我们的测试需求组合在一起的能力。

## **解决问题 （Resolving the Problem）**##

**Action Attributes** 能够帮助我们摆脱困境，思考一下这个例子：
  [TestFixture, ResetServiceLocator]
	public class MyTests
	{
	 [Test, CreateTestDatabase]
	 public void Test1() { /* ... */ }
	
	 [Test, CreateTestDatabase, AsAdministratorPrincipal]
	 public void Test2() { /* ... */ }
	
	 [Test, CreateTestDatabase, AsNamedPrincipal("charlie.poole")]
	 public void Test3() { /* ... */ }
	
	 [Test, AsGuestPrincipal]
	 public void Test4() { /* ... */ }
	}

这里我们使用 “用户已定义” 属性（user-defined attributes）确定了 5 个不同的动作，我们会把这些动作用不同的方式组合在一起来适应不同的测试：

- ResetServiceLocator
- CreateTestDatabase
- AsAdministratorPrincipal
- AsNamedPrincipal
- AsGuestPrincipal 

我们能够在其他的测试 fixtures 中复用这些动作，通过简单地用适当的属性装饰一下它们。不需要继承自某个基类。我们甚至能够开发配属一个普通测试动作类。

##**实现一个（Implementing an） Action Attribute**##

Action Attribute 是程序员定义的。它们实现了 像下面这样定义的 **ITestAction** 接口：

	public interface ITestAction
	{
	    void BeforeTest(TestDetails details);
	
	    void AfterTest(TestDetails details);
	
	    ActionTargets Targets { get; }
	}

为了方便，你可能会从 NUnit 的 **TestActionAttribute** 派生你自己的 action attribute，TestActionAttribute是一个虚拟实现了每一个接口成员的抽象类。作为另一种选择，你可能会从**System.Attribute** 派生，然后直接实现这个接口。

**Action Targets**

**Targets** 属性（property）的返回值决定合适调用 **BeforeTest** 和 **AfterTest** 方法。ActionTargets定义如下：

	[Flags]
	public enum ActionTargets
	{
	    Default = 0,
	
	    Test = 1,
	
	    Suite = 2
	}

当一个 attribute 返回一个要么应用于类，要么应用于参数化方法的 **ActionTargets.Suite**时，NUnit将会在执行test suite 之前优先执行 attribute 的 **BeforeTest** 方法，并且在 test suite 完成执行后执行 **AfterTest** 方法。这类似于 **TestFixtureSetUp** 和 **TestFixtureTearDown** 属性的使用；

**Test Details**

**BeforeTest** 和 **AfterTest** 方法提供一些特定的测试即将运行（before）或者是刚刚运行（after）的信息，**TestDetails** 提供下面一些私有属性（properties），这些属性可能被 before 或者 after 方法使用来决定该采取什么动作：

- **Fixture** - 一个用来表现用户 fixture 是否可用，或者是否为空的对象；

- **Method** - 实现测试是否可用，是否为空的方法信息；

- **FullName** - 一个提供测试完整名称的字符串；

- **Type** - 一个表现测试类型的字符串，比如："Test Case"

- **IsSuite** - 如果测试是 suite 为 true 否者 false；

##**示例**##

下所有的示例都采用下面的 Action attribute 示例：

	[AttributeUsage(AttributeTargets.Method | AttributeTargets.Class |
	                AttributeTargets.Interface | AttributeTargets.Assembly,
	                AllowMultiple = true)]
	public class ConsoleActionAttribute : Attribute, ITestAction
	{
	    private string _Message;
	
	    public ConsoleActionAttribute(string message) { _Message = message; }
	
	    public void BeforeTest(TestDetails details)
	    {
	        WriteToConsole("Before", details);
	    }
	
	    public void AfterTest(TestDetails details)
	    {
	        WriteToConsole("After", details);
	    }
	
	    public ActionTargets Targets
	    {
	        get { return ActionTargets.Test | ActionTargets.Suite; }
	    }
	
	    private void WriteToConsole(string eventMessage, TestDetails details)
	    {
	        Console.WriteLine("{0} {1}: {2}, from {3}.{4}.",
	            eventMessage,
	            details.IsSuite ? "Suite" : "Case",
	            _Message,
	            fixture != null ? fixture.GetType().Name : "{no fixture}",
	            method != null ? method.Name : "{no method}");
	    }
	}

注意上面的 Action Attribute 返回的是 ActionTargets.Test 和 ActionTargets.Suite 的并集。这是允许的，但是在正常情况下可能不会这么用。在这完成本例后，在接下来的很多例子中我们都可以复用它。这个 attribute 包含一个单个的构造函数参数，一个将会输出到控制台的消息。所有的 Before 和 After 方法都通过 WriteToConsole 方法输出信息到控制台。

##**附加了Actions的方法**##

**例1 （应用于一个简单的测试方法）：**

	[TestFixture]
	public class ActionAttributeSampleTests
	{
	    [Test][ConsoleAction("Hello")]
	    public void SimpleTest()
	    {
	        Console.WriteLine("Test ran.");
	    }
	}

控制台输出：

  *Before Case: Hello, from ActionAttributeSampleTests.SimpleTest.
  Test ran.
  After Case: Hello, from ActionAttributeSampleTests.SimpleTest.*

**例2 （对一个测试方法应用两次 action）**

	[TestFixture]
	public class ActionAttributeSampleTests
	{
	    [Test] [ConsoleAction("Hello")]
	    [ConsoleAction("Greetings")]
	    public void SimpleTest()
	    {
	        Console.WriteLine("Test run.");
	    }
	}

控制台输出：

  *Before Case: Greetings, from ActionAttributeSampleTests.SimpleTest.
  Before Case: Hello, from ActionAttributeSampleTests.SimpleTest.
  Test run.
  After Case: Hello, from ActionAttributeSampleTests.SimpleTest.
  After Case: Greetings, from ActionAttributeSampleTests.SimpleTest.*

备注：

你可以应用相同的 attribute 多次。注意虽然 attribute 应用顺序是不确定的，但是对于一个单一发行的.Net版本来说，它普遍是很稳定的。

**例3 （应用于一个带有测试用例的测试方法）**

	[TestFixture]
	public class ActionAttributeSampleTests
	{
	    [Test] [ConsoleAction("Hello")]
	    [TestCase("02")]
	    [TestCase("01")]
	    public void SimpleTest(string number)
	    {
	        Console.WriteLine("Test run {0}.", number);
	    }
	}

控制台输出：

  *Before Suite: Hello, from ActionAttributeSampleTests.SimpleTest.
  Before Case: Hello, from ActionAttributeSampleTests.SimpleTest.
  Test run 01.
  After Case: Hello, from ActionAttributeSampleTests.SimpleTest.
  Before Case: Hello, from ActionAttributeSampleTests.SimpleTest.
  Test run 02.
  After Case: Hello, from ActionAttributeSampleTests.SimpleTest.
  After Suite: Hello, from ActionAttributeSampleTests.SimpleTest.*

备注：

当一个或多个 [TestCase] attribute 应用于一个方法时，
NUnit 把这个方法看作是一个 test suite， 你将会注意到BeforeTest 在suite之前运行了一次，且AfterTest 在其后运行了一次。额外的，BeforeTest 和 AfterTest 将会对每一条单独的测试用例再各运行一次。注意测试用例执行的顺序是不确定地~

##**附加了Actions的类型**##

**例1：**

	[TestFixture] [ConsoleAction("Hello")]
	public class ActionAttributeSampleTests
	{
	    [Test]
	    public void SimpleTestOne()
	    {
	        Console.WriteLine("Test One.");
	    }
	    
	    [Test]
	    public void SimpleTestTwo()
	    {
	        Console.WriteLine("Test Two.");
	    }
	}

控制台输出：

  *Before Suite: Hello, from ActionAttributeSampleTests.{no method}.
  Before Case: Hello, from ActionAttributeSampleTests.SimpleTestOne.
  Test ran.
  After Case: Hello, from ActionAttributeSampleTests.SimpleTestOne.
  Before Case: Hello, from ActionAttributeSampleTests.SimpleTestTwo.
  Test ran.
  After Case: Hello, from ActionAttributeSampleTests.SimpleTestTwo.
  After Suite: Hello, from ActionAttributeSampleTests.{no method}.*

备注：

在这种情况下，这个类是一个 test suite。BeforeTest 和 AfterTest 会对这个类运行一次，然后再对每个测试都运行一次。

**例2：附加到接口**

	[ConsoleAction("Hello")]
	public interface IHaveAnAction
	{
	}
	
	[TestFixture]
	public class ActionAttributeSampleTests : IHaveAnAction
	{
	    [Test] 
	    public void SimpleTest()
	    {
	        Console.WriteLine("Test run.");
	    }
	}

控制台输出：

  *Before Suite: Hello, from ActionAttributeSampleTests.{no method}.
  Before Case: Hello, from ActionAttributeSampleTests.SimpleTest.
  Test run.
  After Case: Hello, from ActionAttributeSampleTests.SimpleTest.
  After Suite: Hello, from ActionAttributeSampleTests.{no method}.*

备注：

Action attributes 能够应用于接口。如果一个标记了 [TestFixture] 实现了一个接口，且有一个 action 应用于这个接口，这个类就从接口继承了该 action attribute。这相当于你应用 action attribute 到这个类本身。

**例3：（action attribute 应用到接口且 attribute 使用接口提供的数据来测试）**

	[AttributeUsage(AttributeTargets.Interface)]
	public class InterfaceAwareActionAttribute : TestActionAttribute
	{
	    private readonly string _Message;
	
	    public InterfaceAwareActionAttribute(string message) { _Message = message; }
	
	    public override void BeforeTest(TestDetails details)
	    {
	        IHaveAnAction obj = details.Fixture as IHaveAnAction;
	        if(obj != null)
	            obj.Message = _Message;
	    }
	
	    public override ActionTargets Target
	    {
	        get { return ActionTargets.Test; }
	    }
	}
	
	[InterfaceAwareAction("Hello")]
	public interface IHaveAnAction { string Message { get; set; } }
	
	[TestFixture]
	public class ActionAttributeSampleTests : IHaveAnAction
	{
	    [Test] 
	    public void SimpleTest()
	    {
	        Console.WriteLine("{0}, World!", Message);
	    }
	
	    public string Message { get; set; }
	}
	
控制台输出：

  *Hello, World!*

备注：

这里我们看到了一个新的 action attribute， [InterfaceAwareAction]。 这个 attribute 使用 TestDetails 的 Fixture 属性（property）传递到 BeforeTest 方法，然后 casts it （传送？投递？传递？） 到一个接口——IHaveAnAction。如果 fixtrue 实现了 IHaveAnAction 接口，attribute 就会把 Message property 设置为传递到此 attribute 构造函数中的字符串。自从 attribute 应用于接口，任意实现这个接口的类就可以获得它自己的 Message property， 并设置为由 action attribute 提供的字符串。 当 action attribute 提供一些数据或者服务来测试的时候，这是非常有用的。

注意： 这个 attribute 继承自 **TestActionAttribute**。它使用默认（什么也不做）的 **AfterTest**实现，然后重写了 **BeforeTest** 和 **Target**。

##**附加了Actions的程序集**##

**例1：**

	[assembly: ConsoleAction("Hello")]
	
	[TestFixture]
	public class ActionAttributeSampleTests
	{
	    [Test] 
	    public void SimpleTest()
	    {
	        Console.WriteLine("Test run.");
	    }
	}

控制台输出：

  *Before Suite: Hello, from {no fixture}.{no method}.
  Before Case: Hello, from ActionAttributeSampleTests.SimpleTest.
  Test run.
  After Case: Hello, from ActionAttributeSampleTests.SimpleTest.
  After Suite: Hello, from {no fixture}.{no method}.*

备注：

在这个例子中是 [ConsoleAction] attribute 应用于整个程序集。NUnit 把程序集看作 test suite（实际上是 a suite of suites）。自从[ConsoleAction] attribute 实现了 ITestSuiteAction 和 ITestCaseAction， NUnit 将会在程序集所有测试运行之前运行一次 BeforeTest， 在程序集所有测试运行以后运行 AfterTest 。此外， BeforeTest 和 AfterTest 将会在程序集中的每条用例中再次运行。action attributes 经常用于程序集，这是不可能的。可是 当生成 action attributes 来确保状态在每次测试之前和之后都得到清理，以此来预防个别的测试影响到其他测试的结果时，这是很有用的。比如说：如果你有一些静态或者缓存数据或服务，一个 action attribute 能够为每一个测试清理它们。
