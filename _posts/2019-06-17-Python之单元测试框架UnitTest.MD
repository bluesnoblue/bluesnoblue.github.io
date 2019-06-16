---
layout: post
title:  "Python之单元测试框架UnitTest"
date:   2019-06-17 01:13:00 +0800
categories: python, test
---

# 简介

**测试脚手架（test fixture）:**
测试脚手架表示为了开展一项或多项测试所需要进行的准备工作，以及所有相关的清理操作。举个例子，这可能包含创建临时或代理的数据库、目录，再或者启动一个服务器进程。

**测试用例（test case）:**
一个测试用例是一个独立的测试单元。它检查输入特定的数据时的响应。unittest提供一个基类： TestCase，用于新建测试用例。

**测试套件（test suite）:**

测试套件是一系列的测试用例，或测试套件，或两者皆有。它用于归档需要一起执行的测试。

**测试运行器（test runner）:**
测试运行器 是一个用于执行和输出测试结果的组件。这个运行器可能使用图形接口、文本接口，或返回一个特定的值表示运行测试的结果。



# 基本实例

unittest模块提供了一系列创建和运行测试的工具。这一段落演示了这些工具的一小部分，但也足以满足大部分用户的需求。
这是一段简短的代码，用来测试三种字符串方法

```python
import unittest


class TestStringMethods(unittest.TestCase):

    def test_upper(self):
        self.assertEqual('foo'.upper(), 'FOO')

    def test_isupper(self):
        self.assertTrue('FOO'.isupper())
        self.assertFalse('Foo'.isupper())

    def test_split(self):
        s = 'hello world'
        self.assertEqual(s.split(), ['hello', 'world'])
        # check that s.split fails when the separator is not a string
        with self.assertRaises(TypeError):
            s.split(2)


if __name__ == '__main__':
    unittest.main()
    
```

继承了unittest.TestCase就创建了一个测试样例，上述三个独立的测试是三个类的方法，这些方法命名都以test开头，这个命名约定告诉测试运行者类的哪些方法表示测试。

每个测试的关键是：调用 assertEqual() 来检查预期的输出； 调用 assertTrue() 或 assertFalse() 来验证一个条件；调用 assertRaises() 来验证抛出了一个特定的异常。之所以不使用Python内建的assert()抛出异常，是因为test runner需要根据这些封装后的方法抛出的异常做测试结果统计。

通过 setUp() 和 tearDown() 方法，可以设置测试开始前与完成后需要执行的指令。 在 组织你的测试代码 中，对此有更为详细的描述。

最后的代码块中，演示了运行测试的一个简单的方法。 unittest.main() 提供了一个测试脚本的命令行接口。当在命令行运行该测试脚本，上文的脚本生成如以下格式的输出:

```
...
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
```

# 组织你的测试代码

单元测试的构建单位是 test cases ：独立的、包含执行条件与正确性检查的方案。在 unittest 中，测试用例表示为 unittest.TestCase 的实例。通过编写 TestCase 的子类或使用 FunctionTestCase 编写你自己的测试用例。

一个 TestCase 实例的测试代码必须是完全自含的，因此它可以独立运行，或与其它任意组合任意数量的测试用例一起运行。

TestCase 的最简单的子类需要实现一个测试方法（例如一个命名以 test 开头的方法）以执行特定的测试代码：



```python
import unittest


class DefaultWidgetSizeTestCase(unittest.TestCase):
    def test_default_widget_size(self):
        widget = Widget('The widget')
        self.assertEqual(widget.size(), (50, 50))

```

可以看到，为了进行测试，我们使用了基类 TestCase 提供的其中一个 assert*() 方法。若测试不通过，将会引发一个带有说明信息的异常，并且 unittest 会将这个测试用例标记为测试不通过。任何其它类型的异常将会被当做错误处理。

可能同时存在多个前置操作相同的测试，我们可以把测试的前置操作从测试代码中拆解出来，并实现测试前置方法 setUp() 。在运行测试时，测试框架会自动地为每个单独测试调用前置方法。

```python
import unittest


class WidgetTestCase(unittest.TestCase):
    def setUp(self):
        self.widget = Widget('The widget')

    def test_default_widget_size(self):
        self.assertEqual(self.widget.size(), (50,50),
                         'incorrect default size')

    def test_widget_resize(self):
        self.widget.resize(100,150)
        self.assertEqual(self.widget.size(), (100,150),
                         'wrong size after resize')

```
```
注解:多个测试运行的顺序由内置字符串排序方法对测试名进行排序的结果决定。 
```

在测试运行时，若 setUp() 方法引发异常，测试框架会认为测试发生了错误，因此测试方法不会被运行。

相似的，我们提供了一个 tearDown() 方法在测试方法运行后进行清理工作。

```python
import unittest


class WidgetTestCase(unittest.TestCase):
    def setUp(self):
        self.widget = Widget('The widget')

    def tearDown(self):
        self.widget.dispose()

```
若 setUp() 成功运行，无论测试方法是否成功，都会运行 tearDown() 。

这样的一个测试代码运行的环境被称为 test fixture 。一个新的 TestCase 实例作为一个测试脚手架，用于运行各个独立的测试方法。在运行每个测试时，setUp() 、tearDown() 和 \_\_init\_\_() 会被调用一次。

建议你根据测试的实际需求将TestCase 组织在一起。unittest为此提供了一种机制：测试套件，又unittest的TestSuite类表示。在大多数情况下，调用unnittest.main(),将做为你收集所有模块中的测试用例并执行他们。

然而，如果你需要自定义你的测试套件的话，你可以参考以下方法组织你的测试：

```python
def get_suite():
    suite = unittest.TestSuite()
    suite.addTest(WidgetTestCase('test_default_widget_size'))
    suite.addTest(WidgetTestCase('test_widget_resize'))
    return suite


if __name__ == '__main__':
    runner = unittest.TextTestRunner()
    runner.run(suite())
    
```


您可以将测试用例和测试套件的定义与要测试的代码（如widget.py）放在相同的模块中，但将测试代码放在单独的模块中有几个优点： 

* 测试模块可以从命令行独立运行。 
* 测试代码可以更容易地与已发送的代码分离。 
* 在没有充分理由的情况下，更改测试代码以适合其测试的代码是不太可能的。 
* 测试代码的修改频率应该比它测试的代码低得多。 
* 测试代码可以更容易地重构。 
* 无论如何，用C编写的模块测试必须在单独的模块中进行，那么为什么不一致呢？ 
* 如果测试策略改变了，就不需要改变源代码。 

# 复用已有的测试代码

一些用户希望直接使用 unittest 运行已有的测试代码，而不需要把已有的每个测试函数转化为一个 TestCase 的子类。

因此， unittest 提供 FunctionTestCase 类。这个 TestCase 的子类可用于打包已有的测试函数，并支持设置前置与后置函数。

假定有一个测试函数：
```python
def testSomething():
    something = makeSomething()
    assert something.name is not None
    #  ...
```
可以创建等价的测试用例如下，其中前置和后置方法是可选的。
```python
testcase = unittest.FunctionTestCase(testSomething,
                                     setUp=makeSomethingDB,
                                     tearDown=deleteSomethingDB)
```
```
尽管可以使用FunctionTestCase将现有的测试库快速转换为基于UnitTest的系统，但不推荐使用这种方法。花时间建立适当的测试用例子类将使未来的测试重构变得极其容易。 
```
在某些情况下，现有测试可能是使用doctest模块编写的。如果是这样，doctest提供了一个doctestsuite类，该类可以从现有的基于doctest的测试中自动生成unittest.testsuite实例。

# 跳过测试与预计的失败

UnitTest支持跳过单个测试方法，甚至跳过整个测试类。此外，它还支持将测试标记为“预期的失败(expected failure)”，即破坏并将失败的测试，但不应计为测试结果(TestResult)的失败。 

跳过测试只是使用skip()装饰器或其条件变量之一，在setup()或测试方法中调用testcase.skip test（），或直接提升skiptest的问题。 

跳过测试的基本用法如下：
```python
class MyTestCase(unittest.TestCase):

    @unittest.skip("demonstrating skipping")
    def test_nothing(self):
        self.fail("shouldn't happen")

    @unittest.skipIf(mylib.__version__ < (1, 3),
                     "not supported in this library version")
    def test_format(self):
        # Tests that work for only a certain version of the library.
        pass

    @unittest.skipUnless(sys.platform.startswith("win"), "requires Windows")
    def test_windows_support(self):
        # windows specific testing code
        pass

    def test_maybe_skipped(self):
        if not external_resource_available():
            self.skipTest("external resource not available")
        # test code that depends on the external resource
        pass
```

在啰嗦模式下运行以上测试例子是，程序输入如下：

```
test_format (__main__.MyTestCase) ... skipped 'not supported in this library version'
test_nothing (__main__.MyTestCase) ... skipped 'demonstrating skipping'
test_maybe_skipped (__main__.MyTestCase) ... skipped 'external resource not available'
test_windows_support (__main__.MyTestCase) ... skipped 'requires Windows'

----------------------------------------------------------------------
Ran 4 tests in 0.005s

OK (skipped=4)
```

跳过测试类的写法跟跳过测试方法的写法相似:

```python
@unittest.skip("showing class skipping")
class MySkippedTestCase(unittest.TestCase):
    def test_not_run(self):
        pass
```

TestCase.setUp() 也可以跳过测试。可以用于所需资源不可用的情况下跳过接下来的测试。

使用 expectedFailure() 装饰器表明这个测试预计失败。:

```python
class ExpectedFailureTestCase(unittest.TestCase):
    @unittest.expectedFailure
    def test_fail(self):
        self.assertEqual(1, 0, "broken")
```

你也可以很简单得通过调用skip()方法来实现一个装饰器。此装饰器跳过测试，除非传递的对象具有特定属性：

```python
def skipUnlessHasattr(obj, attr):
    if hasattr(obj, attr):
        return lambda func: func
    return unittest.skip("{!r} doesn't have {!r}".format(obj, attr))
```

以下修饰符和异常实现测试跳过和预期的失败：

@unittest.skip(reason)
跳过被此装饰器装饰的测试。 reason 为测试被跳过的原因。

@unittest.skipIf(condition, reason)
当 condition 为真时，跳过被装饰的测试。

@unittest.skipUnless(condition, reason)
跳过被装饰的测试，除非 condition 为真。

@unittest.expectedFailure
把测试标记为预计失败。如果测试不通过，会被认为测试成功；如果测试通过了，则被认为是测试失败。

exception unittest.SkipTest(reason)
引发此异常以跳过一个测试。
通常来说，你可以使用 TestCase.skipTest() 或其中一个跳过测试的装饰器实现跳过测试的功能，而不是直接引发此异常。

被跳过的测试的 setUp() 和 tearDown() 不会被运行。被跳过的类的 setUpClass() 和 tearDownClass() 不会被运行。被跳过的模组的 setUpModule() 和 tearDownModule() 不会被运行。

# 测试用例

class unittest.TestCase(methodName='runTest')
TestCase类的实例作为unittest中的测试单元。这个类被用作一个基类，具体的测试由具体的子类实现。这个类实现了测试运行程序所需的接口，以允许它驱动测试，以及测试代码可以用来检查和报告各种失败的方法。 
每个TestCase实例都将运行一个基本方法：名为methodname的方法。在大多数使用testcase的情况下，既不会更改方法名，也不会重新实现默认的runtest()方法。在不提供方法名的情况下，可以成功实例化测试用例。这使得从交互式解释器对测试用例进行实验变得更容易。 
测试用例实例提供三组方法：一组用于运行测试，另一组用于测试实现检查条件和报告失败，以及一些查询方法，允许收集有关测试本身的信息。 

第一组（运行测试）中的方法是： 



setUp()
调用此方法来准备测试。在调用测试方法之前立即调用此函数；除了断言错误或skiptest之外，此方法引发的任何异常都将被视为错误，而不是测试失败。默认实现什么也不做。 



tearDown()
方法在调用测试方法并记录结果后立即调用。即使测试方法引发异常，也会调用此方法，因此子类中的实现可能需要特别注意检查内部状态。此方法引发的任何异常（断言错误或skiptest除外）将被视为额外的错误，而不是测试失败（从而增加报告的错误总数）。只有在setup()成功时才会调用此方法，而不管测试方法的结果如何。默认实现什么也不做。 



setUpClass()
在运行单个类中的测试之前调用的类方法。使用类作为唯一参数调用SetupClass，并且必须作为ClassMethod（）进行修饰：

```python
@classmethod
def setUpClass(cls):
    pass

```


tearDownClass()
在单个类中的测试运行后调用的类方法。TearDownclass是以类作为唯一参数调用的，必须作为ClassMethod（）进行修饰：

```python
@classmethod
def tearDownClass(cls):
    pass

```

run(sesult=None)
运行测试，将结果收集到作为结果传递的TestResult对象中。如果结果被省略或没有，将创建一个临时结果对象（通过调用defaultTestResult()方法）并使用。结果对象返回到run()的调用方。
通过简单地调用testcase实例也可以产生同样的效果。

skipTest(reason)
在测试方法或setup()期间调用此函数将跳过当前测试。有关更多信息，请参见跳过测试与预期的失败。

subTest(msg=None, \*\*params)
返回一个执行封闭代码块作为次测试的上下文管理器。msg和参数是可选的，测试失败的情况下，能使你明确地识别它们。
一个测试案例可以包含任何次测试声明的数目，并且可以任意地加以考虑。

debug()
运行测试而不收集结果。这允许将测试引发的异常传播到调用方，并可用于支持在调试器下运行测试。

testcase类提供了几个断言方法来检查和报告失败。下表列出了最常用的方法：

| methode                   | check that           |
| ------------------------- | -------------------- |
| assertEqual(a, b)         | a == b               |
| assertNotEqual(a, b)      | a != b               |
| assertTrue(x)             | bool(x) is Tue       |
| assertFalse(x)            | bool(x) is False     |
| assertIs(a, b)            | a is b               |
| assertIsNot(a, b)         | a is not b           |
| assertIsNone(x)           | a is None            |
| assertIsNotNone(x)        | x is not None        |
| assertIn(a, b)            | a in b               |
| assertNotIn(a, b)         | a not in b           |
| assertIsInstance(a, b)    | isinstance(a, b)     |
| assertNotIsInstance(a, b) | not isinstance(a, b) |

以上的断言方法都接受一个msg参数，如果指定了该参数，它将用作失败时的错误消息。

# 组织测试

class unittest.TestSuite(tests=())

此类表示单个测试用例和测试套件的聚合。类提供测试运行程序所需的接口，以允许它像其他测试用例一样运行。运行test suite实例与在该套件上迭代相同，分别运行每个测试。

如果给定了tests，它必须是单个测试用例或其他最初用于构建该套件的测试套件的可迭代对象，稍后将提供其他方法向集合中添加测试用例和套件。

testsuite对象的行为与testcase对象非常相似，只是它们实际上并不实现测试。相反，它们被用来将测试聚合到应该一起运行的测试组中。可以使用一些其他方法向TestSuite实例添加测试： 


addTest(test)
向套件中添加一个测试用例或测试套件。

addTests(tests)
将一个可迭代的TestCase或TestSuit实例添加到此测试套件。
这相当于迭代测试，为每个元素调用addtest（）。 

TestSuite与TestCase共享以下方法： 

run(result)
运行与此套件关联的测试，将结果收集到作为结果传递的测试结果对象中。请注意，与testcase.run（）不同，testsuite.run（）需要传入result对象。

debug()
运行与此套件关联的测试，而不收集结果。这允许将测试引发的异常传播到调用方，并可用于支持在调试器下运行测试。

countTestCases()
返回此测试对象表示的测试数，包括所有单个测试和子套件。

在TestSuite对象的典型用法中，run()方法由TestRunnerr调用，而不是由最终用户测试工具调用。

# 载入和执行测试

待更新....
