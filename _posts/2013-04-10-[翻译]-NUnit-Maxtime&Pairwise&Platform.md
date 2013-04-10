---
layout: post
category : lessons
tags : [NUnit]
---
{% include JB/setup %}

#**MaxtimeAttribute (NUnit 2.5)**#

MaxtimeAttribute 用于为测试用在测试方法上指定一个以毫秒为单位的最大测试时间。如果测试用例完成测试时花费超过了指定时间，那么在报告上将会显示为失败。

**示例**

  [Test, Maxtime(2000)]
	public void TimedTest()
	{
	    ...
	}

**注意：**

1. 任意断言（assertion ）失败的优先级高于时间流逝检查。

2. 这个 attribute 在时间超出以后，是不能取消测试的。它只是等待测试完成，然后比较已经流逝的时间和指定的最大时间。如果你想要取消长时间运行的测试，请参阅[TimeoutAttribute](http://nunit.org/index.php?p=timeout&r=2.6.2)

#**PairwiseAttribute (NUnit 2.5)**#

PairwiseAttribute 用于在测试中指出NUnit应该以下面这种方式来生成测试用例：使用所有可能的成双成对的值。当超过两个功能（或者参数）时，这是一种解决测试用例组合激增的著名方法。

**注意:** 在当前 Alpha release 版本, 此 attribute 是可用的， 除了忽略的和使用默认组合方法的组合的数据项。

这也没个例子。。。。：（

#**PlatformAttribute (NUnit 2.2.2)**#

PlatformAttribute 用于为某个 test 或者 fixture 指定一个运行平台，使用一个区分大小写的字符串值指定平台，还可以通过分别使用Include 或 Exclude properties 从运行中引入或排除平台。引入平台时有两种情况，可以为 PlatformAttribute 的构造函数指定一个参数来引入，在某些情况下，可以是使用多个逗号隔开的值来指定。

如果标记了Platform attribute 的 test 或者 fixture 不符合指定的平台需求，那么它将会跳过。该测试不会影响运行结果：没有把它当作忽略对待，甚至没有把它计入到测试总数中去。在GUI中，测试的树节点保持灰色，状态栏的颜色也不会受到影响。

***注意：** 在NUnit 2.4之前的版本，这些测试是显示为忽略的。*

**Test Fixture Syntax**

	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture]
	  [Platform("NET-2.0")]
	  public class DotNetTwoTests
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
	    [Platform(Exclude="Win98,WinME")]
	    public void SomeTest()
	    { /* ... */ }
	}

##**平台说明符（Platform Specifiers）**##

下面的值是能够被平台说明符识别的。它们以大写，小写或者大小写混合的形式表现：

    Win
    Win32
    Win32S
    Win32Windows
    Win32NT
    WinCE
    Win95
    Win98
    WinMe
    NT3
    NT4
    NT5
    NT6
    Win2K
    WinXP
    Win2003Server
    Vista
    Win2008Server
    Win2008ServerR2
    Windows7
    Win2012Server
    Windows8
    Unix
    Linux
    MacOsX
    XBox
    Net
    Net-1.0
    Net-1.1
    Net-2.0
    Net-3.0 (1)
    Net-3.5 (2)
    Net-4.0
    Net-4.5 (3)
    NetCF
    SSCLI
    Rotor
    Mono
    Mono-1.0
    Mono-2.0
    Mono-3.0 (4)
    Mono-3.5 (5)
    Mono-4.0

**注意：**

    Includes Net-2.0
    Includes Net-2.0 and Net-3.0
    Includes Net-4.0
    Includes Mono-2.0
    Includes Mono-2.0 and Mono-3.0 

