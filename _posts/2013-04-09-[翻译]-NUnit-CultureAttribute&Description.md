---
layout: post
category : lessons
tags : [NUnit]
---
{% include JB/setup %}

翻译自[http://nunit.org/index.php?p=actionAttributes&r=2.6.2](http://nunit.org/index.php?p=actionAttributes&r=2.6.2)

#**CultureAttribute (NUnit 2.4.2)**#

Culture attribute 用来列举一些文化背景，其决定哪个测试或者 fixture 应该运行。这不会影响文化设置，仅用于决定是否运行测试。如果你想要在运行测试时改变文化， 使用 SetCulture attribute代替。

如果规定的文化需求与相应的测试不符合，那么它将会被跳过。在GUI中，该测试的树节点将会保持为灰色，且状态栏的颜色是不会受到影响的。

Culture attribute 的用途之一就是在不同的文化背景下提供可供选择的测试。你有可能会指定一些特别的文化背景，比如：“en-GB” 或者像“de”的中立文化背景。

## **Test Fixture 语法** ##
  
	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture]
	  [Culture("fr-FR")]
	  public class FrenchCultureTests
	  {
	    // ...
	  }
	}

## **Test  语法** ##

	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture]
	  public class SuccessTests
	  {
	    [Test]
	    [Culture(Exclude="en,de")]
	    public void SomeTest()
	    { /* ... */ }
	}

#**Description (NUnit 2.4)**#

Description attribute用来为Test，Fixture， 程序集添加一些描述性的文本。 文本以XML输出文件的形式呈现，显示于 Test Properties 的对话框。

**示例**

	[assembly: Description("Assembly description here")]
	
	namespace NUnit.Tests
	{
	  using System;
	  using NUnit.Framework;
	
	  [TestFixture, Description("Fixture description here")]
	  public class SomeTests
	  {
	    [Test, Description("Test description here")] 
	    public void OneTest()
	    { /* ... */ }
	  }
	}

注意：Test，TestFixture attribute 继续支持一个可选的 Description property，Description attribute 应该用于新的应用程序。如果两者都使用，那么Description attribute优先。







