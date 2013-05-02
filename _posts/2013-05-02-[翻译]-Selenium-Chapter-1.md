---
layout: post
category : lessons
tags : [NUnit]
---
{% include JB/setup %}

#**1. 定位元素**##

在本章，我们将覆盖：

- 使用浏览器工具检查元素和页面结构.
- 使用 findElement 方法定位一个元素.
- 使用 findElement 方法定位多个元素.
- 定位链接.
- 根据标签名定位元素.
- 使用 CSS 选择器定位元素.
- 使用 XPath 定位元素.
- 使用 text 定位元素.
- 使用高级的 CSS 选择器定位元素.
- 使用 jQuery 选择器.
- 定位表的行和单元格.
- 定位表中的子元素.

##**介绍**##

成功的自动化 GUI (Graphical User Interface 图形化用户界面) 测试依靠识别和定位被测应用程序 GUI 的元素，然后对这些元素执行操作和验证以完成测试流程。这些归结于测试工具有效识别各种 GUI 元素的能力。

Selenium WebDriver 提供了一种在网页上定位元素的高级技术。Selenium 功能丰富的 API 提供多种定位器策略，比如说：Name，ID, CSS 选择器，XPath 等等。我们也能够实现自定义的定位器策略来定位元素。

在本章，我们将会以ID, Name，和 Class 为开始，来探讨更多关于如何使用定位策略的知识。

在一些 Web 开发项目中，总是有一个很好的惯例来分配像 Name，IDs ，或者 Class 这样的 attributes 给 GUI 元素。这使得应用程序更加可测试且更加符合可访问性标准。可是，这种惯例并不总是可行，对于这种场景，我们必须使用高级的定位器策略，比如 CSS 选择器 和 XPath。

虽然 CSS 选择器 和 XPath 在 Selenium 用户中都很流行，但是相对 XPath 而言， 优先推荐使用 CSS 选择器，因为CSS 选择器更加简易，快速且性能卓越。

##**使用浏览器工具检查元素和页面结构**##

在我们开始探讨定位器之前，我们需要分析页面和元素来弄懂这些东西在应用程序中是如何构造的， 元素定义了哪些属性或者 attributes，应用程序是如何调用 JavaScirpt 或者 AJAX 的等等。

浏览器对最终用户隐藏了 HTML 代码和其他资源且渲染了应用程序的虚拟元素， 当我们想要使用 Selenium WebDriver 来实现自动与应用程序交互时，我们需要认真查看用来在浏览器中渲染页面和元素的源代码。我们需要确定像 attributes 值和元素结构这样的信息来定位元素，然后使用 Selenium WebDriver API 来执行用户操作；

……

浏览器开发者工具在测试环境中很有用。这些工具将会帮助你发现元素定位器的详细信息。这些工具解析页面代码然后以树状结构显示这些信息。这些工具也能够一些关于如何应用风格，页面资源，页面 DOM （Document Object Model），JavaScript 代码等等。

这些工具中的一些还提供了运行JavaScript 代码来调试和测试的能力。

在下面的秘籍中，我们将会探讨各式各样的Selenium WebDriver 支持的选择器类型，且这些工具将会帮助你发现和决定各种选择器策略或者由 Selenium WebDriver API 提供的方法。

##**使用 findElement 方法定位一个元素**##

在 Selenium WebDriver 中使用由 WebDriver 和 WebElement 类提供的 findElement() 和 finElements() 方法来定位元素。

finElement() 方法根据一个指定查找条件返回一个 WebElement 对象，或者如果没有找到匹配元素，则抛出一个异常。

findElements() 返回一个匹配查找条件的 WebElements 列表，如果没有找到，就返回一个空列表。

Find 方法使用一个定位器或者一个作为 By 类接口的查询对象来作为参数。Selenium WebDriver 提供 By 类来支持各种定位器策略。

……

使用 id， name， 或者 class attributes 定位元素是查找元素的优先方法。下面让我们尝试使用这些方法来定位元素。

**通过 ID attribute 查找元素**

*To be contiue...*



