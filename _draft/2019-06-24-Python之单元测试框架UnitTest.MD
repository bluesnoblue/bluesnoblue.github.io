---
layout: post
title:  "Python之单元测试框架UnitTest"
date:   2019-06-24 01:13:00 +0800
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

**class unittest.TestCase(methodName='runTest')**
TestCase类的实例作为unittest中的测试单元。这个类被用作一个基类，具体的测试由具体的子类实现。这个类实现了测试运行程序所需的接口，以允许它驱动测试，以及测试代码可以用来检查和报告各种失败的方法。 
每个TestCase实例都将运行一个基本方法：名为methodname的方法。在大多数使用testcase的情况下，既不会更改方法名，也不会重新实现默认的runtest()方法。在不提供方法名的情况下，可以成功实例化测试用例。这使得从交互式解释器对测试用例进行实验变得更容易。 
测试用例实例提供三组方法：一组用于运行测试，另一组用于测试实现检查条件和报告失败，以及一些查询方法，允许收集有关测试本身的信息。 

第一组（运行测试）中的方法是： 

**setUp()**
调用此方法来准备测试。在调用测试方法之前立即调用此函数；除了断言错误或skiptest之外，此方法引发的任何异常都将被视为错误，而不是测试失败。默认实现什么也不做。 

**tearDown()**
方法在调用测试方法并记录结果后立即调用。即使测试方法引发异常，也会调用此方法，因此子类中的实现可能需要特别注意检查内部状态。此方法引发的任何异常（断言错误或skiptest除外）将被视为额外的错误，而不是测试失败（从而增加报告的错误总数）。只有在setup()成功时才会调用此方法，而不管测试方法的结果如何。默认实现什么也不做。 

**setUpClass()**
在运行单个类中的测试之前调用的类方法。使用类作为唯一参数调用SetupClass，并且必须作为ClassMethod（）进行修饰：

```python
@classmethod
def setUpClass(cls):
    pass

```

**tearDownClass()**
在单个类中的测试运行后调用的类方法。TearDownclass是以类作为唯一参数调用的，必须作为ClassMethod（）进行修饰：

```python
@classmethod
def tearDownClass(cls):
    pass

```

**run(sesult=None)**
运行测试，将结果收集到作为结果传递的TestResult对象中。如果结果被省略或没有，将创建一个临时结果对象（通过调用defaultTestResult()方法）并使用。结果对象返回到run()的调用方。
通过简单地调用testcase实例也可以产生同样的效果。

**skipTest(reason)**
在测试方法或setup()期间调用此函数将跳过当前测试。有关更多信息，请参见跳过测试与预期的失败。

**subTest(msg=None, \*\*params)**
返回一个执行封闭代码块作为次测试的上下文管理器。msg和参数是可选的，测试失败的情况下，能使你明确地识别它们。
一个测试案例可以包含任何次测试声明的数目，并且可以任意地加以考虑。

**debug()**
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

**class unittest.TestSuite(tests=())**

此类表示单个测试用例和测试套件的聚合。类提供测试运行程序所需的接口，以允许它像其他测试用例一样运行。运行test suite实例与在该套件上迭代相同，分别运行每个测试。

如果给定了tests，它必须是单个测试用例或其他最初用于构建该套件的测试套件的可迭代对象，稍后将提供其他方法向集合中添加测试用例和套件。

testsuite对象的行为与testcase对象非常相似，只是它们实际上并不实现测试。相反，它们被用来将测试聚合到应该一起运行的测试组中。可以使用一些其他方法向TestSuite实例添加测试： 

**addTest(test)**
向套件中添加一个测试用例或测试套件。

**addTests(tests)**
将一个可迭代的TestCase或TestSuit实例添加到此测试套件。
这相当于迭代测试，为每个元素调用addtest（）。 

**TestSuite与TestCase共享以下方法： **

**run(result)**
运行与此套件关联的测试，将结果收集到作为结果传递的测试结果对象中。请注意，与testcase.run（）不同，testsuite.run（）需要传入result对象。

**debug()**
运行与此套件关联的测试，而不收集结果。这允许将测试引发的异常传播到调用方，并可用于支持在调试器下运行测试。

**countTestCases()**
返回此测试对象表示的测试数，包括所有单个测试和子套件。

在TestSuite对象的典型用法中，run()方法由TestRunnerr调用，而不是由最终用户测试工具调用。

# 载入和执行测试

**class unittest.TestLoader**

TestLoader类用于从类和模块创建测试套件。 通常，不需要创建此类的实例; unittest模块提供了一个可以作为unittest.defaultTestLoader共享的实例。 但是，使用子类或实例可以自定义一些可配置的属性。

**TestLoader对象有以下属性：**

**Errors**
加载测试时遇到的非致命错误列表。 任何时候都不会被加载器重置。 致命错误由相关的方法发出信号，该方法向调用者提出异常。 非致命错误也由合成测试指示，该测试将在运行时引发原始错误。

**TestLoader对象有以下方法：**

**loadTestsFromTestCase(testCaseClass)**
返回TestCase派生的testCaseClass中包含的所有测试用例的套件。
为getTestCaseNames()每个方法创建一个测试用例实例。 默认情况下，这些是以test开头的方法名称。 如果getTestCaseNames()没有返回任何方法，但是实现了runTest（）方法，则会为该方法创建一个测试用例。

**loadTestsFromModule(module, pattern=None)**
返回给定模块中包含的所有测试用例的套件。 此方法在模块中搜索从TestCase派生的类，并为为该类定义的每个测试方法创建该类的实例。

```
注解
虽然使用TestCase派生类的层次结构可以方便地共享fixture和helper函数，但是在不打算直接实例化的基类上定义测试方法并不能很好地使用此方法。 但是，当fixture不同并在子类中定义时，这样做会很有用。
```

如果模块提供了load_tests函数，则会调用它来加载测试。 这允许模块自定义测试加载。 这是load_tests协议。 pattern参数作为load_tests的第三个参数传递。

不推荐使用未记录和非正式的use_load_tests默认参数，但仍然可以接受它以实现向后兼容性。 该方法现在还接受仅关键字参数模式，该模式作为第三个参数传递给load_tests。

**loadTestsFromName(name, module=None)**
根据传递的字符串说明符，返回所有测试用用例的套件

说明符名称是“点名称”，可以解析模块，测试用例类，测试用例类中的测试方法，TestSuite实例或返回TestCase或TestSuite实例的可调用对象。这些检查根据以下列出的顺序被采用。也就是说，可能的测试用例类的方法将被选为“测试用例类中的测试方法”，而不是“可调用对象”。

例如，如果您的模块SampleTests包含一个TestCase派生类SampleTestCase，其中包含三个测试方法( test_one()，test_two()和test_three() )，则说明符'SampleTests.SampleTestCase'将导致此方法返回套件 这将运行所有三种测试方法。 使用说明符'SampleTests.SampleTestCase.test_two'将导致它返回一个只运行test_two()测试方法的测试套件。 说明符可以引用尚未导入的模块和包; 它们将作为副作用导入。

该方法可选地解析相对于给定模块的名称。

如果在遍历name时发生ImportError或AttributeError，则将返回在运行时引发该错误的综合测试。 这些错误包含在self.errors累积的错误中。

**loadTestsFromNames(names, module=None)**

与loadTestsFromName()类似，但采用一系列名称而不是单个名称。 返回值是一个测试套件，它支持为每个名称定义的所有测试。

**getTestCaseNames(testCaseClass)**

返回在testCaseClass中找到的已排序的方法名称序列; 这应该是TestCase的子类。

**discover(start_dir, pattern='test*.py', top_level_dir=None)**

通过从指定的起始目录递归到子目录中查找所有测试模块，并返回包含它们的TestSuite对象。 仅加载与模式匹配的测试文件。 （使用shell样式模式匹配。）仅加载可导入的模块名称（即有效的Python标识符）。

所有测试模块必须可从项目的顶层导入。 如果起始目录不是顶级目录，则必须单独指定顶级目录。

如果导入模块失败（例如由于语法错误），则会将其记录为单个错误，并且将继续发现。 如果导入失败是由于引发了SkipTest，则会将其记录为跳过而不是错误。

如果找到包（包含名为__init__.py的文件的目录），将检查包的load_tests函数。 如果存在，那么它将被称为package.load_tests（loader，tests，pattern）。 测试发现需要注意确保在调用期间仅检查一次包测试，即使load_tests函数本身调用loader.discover也是如此。

故意不将该模式存储为loader属性，以便包可以继续自己发现。 存储top_level_dir，因此load_tests不需要将此参数传递给loader.discover()。

start_dir可以是虚线模块名称以及目录。

导入时引发SkipTest的模块记录为跳过，而不是错误。 发现适用于命名空间包。 在导入之前对路径进行排序，以便即使基础文件系统的排序不依赖于文件名，执行顺序也是相同的。

现在检查已找到的包的load_tests，无论它们的路径是否与模式匹配，因为包名称不可能与默认模式匹配。

可以通过在实例上进行子类化或赋值来配置TestLoader的以下属性：

**testMethodPrefix**

给出方法名称前缀的字符串，将被解释为测试方法。 默认值为“test”。
这会影响getTestCaseNames()和所有loadTestsFrom \*()方法。

**sortTestMethodsUsing**

在getTestCaseNames()和所有loadTestsFrom\*()方法中对方法名称进行排序时用于比较方法名称的函数。

**suiteClass**

可调用对象，从测试列表构造测试套件。 不需要对结果对象的方法。 默认值是TestSuite类。

这会影响所有loadTestsFrom\*()方法。

**testNamePatterns**

测试方法必须匹配的Unix shell样式通配符测试名称模式列表，以包含在测试套件中（请参阅-v选项）。

如果此属性不是None（缺省值），则要包含在测试套件中的所有测试方法都必须与此列表中的某个模式匹配。 请注意，匹配始终使用fnmatch.fnmatchcase（）执行，因此与传递给-v选项的模式不同，必须使用*通配符转换简单子字符串模式。

这会影响所有loadTestsFrom\*()方法。

**class unittest.TestResult**

此类用于编译有关哪些测试成功以及哪些测试失败的信息。

TestResult对象存储一组测试的结果。 TestCase和TestSuite类确保正确记录结果; 测试作者无需担心记录测试结果。

**在检查运行一组测试的结果时，TestResult实例具有以下属性：**

**errors**

包含2元组TestCase实例和包含格式化回溯的字符串的列表。 每个元组代表一个引发意外异常的测试。

**failures**

包含2元组TestCase实例和包含格式化回溯的字符串的列表。 每个元组代表一个测试，其中使用TestCase.assert\* ()方法显式地发出故障信号。

**skipped**

包含2元组TestCase实例和字符串的列表，其中包含跳过测试的原因。

**expectedFailures**

包含2元组TestCase实例和包含格式化回溯的字符串的列表。 每个元组代表测试用例的预期失败。

**unexpectedSuccesses**

包含TestCase实例的列表，这些实例已标记为预期失败，但已成功。

**shouldStop**

当测试的执行应该由stop()停止时，设置为True。

**testsRun**

到目前为止测试的总数。       

**buffer**

如果设置为true，则sys.stdout和sys.stderr将在调用的startTest()和stopTest()之间进行缓冲。 如果测试失败或错误，收集的输出将仅回显到真实的sys.stdout和sys.stderr。 任何输出也附加到故障/错误消息。        

**failfast**

如果设置为true，则会在第一次失败或错误时调用stop()暂停测试运行。

**tb_locals**

如果设置为true，则局部变量将显示在回溯中。

**wasSuccessful()**

如果到目前为止所有测试都已经过，则返回True，否则返回False。

如果使用expectedFailure()装饰器标记的测试中有任何意外的结果，则返回False。

**stop()**

可以通过将shouldStop属性设置为True来调用此方法来表示应该中止正在运行的测试集。 TestRunner对象应该尊重此标志并返回而不运行任何其他测试。

例如，当用户通过键盘发出中断信号时，TextTestRunner类使用此功能来停止测试框架。 提供TestRunner实现的交互式工具可以以类似的方式使用它。

TestResult类的以下方法用于维护内部数据结构，并且可以在子类中进行扩展以支持其他报告要求。 这对于构建在运行测试时支持交互式报告的工具特别有用。

**startTest(test)**

在即将运行测试用例测试时调用。

**stopTest(test)**

无论结果如何，在执行测试用例测试后调用。

**startTestRun()**

在执行任何测试之前调用一次。

**stopTestRun()**

执行所有测试后调用一次。

**addError(test, err)**

在测试用例测试引发意外异常时调用。 err是sys.exc_info（）返回的形式的元组:(类型，值，跟踪）。

默认实现将元组（test，formatted_err）附加到实例的errors属性，其中formatted_err是从err派生的格式化回溯。

**addFailure(test, err)**

当测试用例测试发出故障信号时调用。 err是sys.exc_info（）返回的形式的元组:(类型，值，跟踪）。

默认实现将元组（test，formatted_err）附加到实例的failure属性，其中formatted_err是从err派生的格式化回溯。

**addSuccess(test)**

在测试用例测试成功时调用。

默认实现什么都不做。

**addSkip(test, reason)**

在跳过测试用例测试时调用。 原因是测试给跳过的原因。

默认实现将元组（test，reason）附加到实例的skipped属性。

**addExpectedFailure(test, err)**

在测试用例测试失败时调用，但是用expectedFailure（）装饰器标记。

默认实现将元组（test，formatted_err）附加到实例的expectedFailures属性，其中formatted_err是从err派生的格式化回溯。

**addUnexpectedSuccess(test)**

当测试用例测试用expectedFailure（）装饰器标记时调用，但成功。

默认实现将测试附加到实例的unexpectedSuccesses属性。

**addSubTest(test, subtest, outcome)**

在子测试结束时调用。 test是与测试方法对应的测试用例。 subtest是一个描述子测试的自定义TestCase实例。

如果结果为None，则子测试成功。 否则，它失败并出现异常，其中results是sys.exc_info（）返回的形式的元组:(类型，值，跟踪）。

结果成功时，默认实现不执行任何操作，并将子测试失败记录为正常故障。

**class unittest.TextTestResult(stream, descriptions, verbosity)**

TextTestRunner使用的TestResult的具体实现。

此类之前名为_TextTestResult。 旧名称仍作为别名存在，但已弃用。

**unittest.defaultTestLoader**

要共享的TestLoader类的实例。 如果不需要自定义TestLoader，则可以使用此实例而不是重复创建新实例。

**class unittest.TextTestRunner(stream=None, descriptions=True, verbosity=1, failfast=False, buffer=False, resultclass=None, warnings=None,\*, tb_locals=False)**

将结果输出到流的基本测试运行器实现。 如果stream为None，则使用缺省值sys.stderr作为输出流。 这个类有一些可配置的参数，但基本上非常简单。 运行测试套件的图形应用程序应提供备用实现。 当将功能添加到unittest时，此类实现应接受/ * / * kwargs作为构造运行程序更改的接口。

默认情况下，此运行器显示DeprecationWarning，PendingDeprecationWarning，ResourceWarning和ImportWarning，即使它们在默认情况下被忽略也是如此。 不推荐使用的单元测试方法引起的弃用警告也是特殊的，当警告过滤器为“默认”或“始终”时，它们每个模块只出现一次，以避免过多的警告消息。 可以使用Python的-Wd或-Wa选项覆盖此行为（请参阅警告控件）并将警告留给无。

**_makeResult()**

此方法返回run（）使用的TestResult实例。 它不是直接调用的，但可以在子类中重写以提供自定义的TestResult。

_makeResult（）将在TextTestRunner构造函数中传递的类或可调用对象实例化为resultclass参数。 如果未提供resultclass，则默认为TextTestResult。 结果类使用以下参数进行实例化：

```
stream, descriptions, verbosity
```

**run(test)**

此方法是TextTestRunner的主要公共接口。 此方法采用TestSuite或TestCase实例。 通过调用\_makeResult()创建TestResult并运行测试并将结果打印到stdout。

**unittest.main(module='__main__', defaultTest=None, argv=None, testRunner=None, testLoader=unittest.defaultTestLoader, exit=True, verbosity=1, failfast=None, catchbreak=None, buffer=None, warnings=None)**

一个命令行程序，它从模块加载一组测试并运行它们; 这主要是为了使测试模块可以方便地执行。 此函数最简单的用法是在测试脚本的末尾包含以下行：
```python
    if __name__ == '__main__':
        unittest.main()
```

您可以通过传递详细信息参数来运行具有更详细信息的测试：

```python
    if __name__ == '__main__':
        unittest.main(verbosity=2)
```
如果没有通过argv指定测试名称，则defaultTest参数可以是单个测试的名称，也可以是要运行的可迭代测试名称。 如果未指定或None且未通过argv提供测试名称，则运行模块中的所有测试。

argv参数可以是传递给程序的选项列表，第一个元素是程序名称。 如果未指定或None，则使用sys.argv的值。

testRunner参数可以是测试运行器类，也可以是已创建的实例。 默认情况下，main调用sys.exit（），并带有退出代码，指示运行测试成功或失败。

testLoader参数必须是TestLoader实例，默认为defaultTestLoader。

通过传入参数exit = False从交互式解释器中使用main支持。 这将在不调用sys.exit（）的情况下在标准输出上显示结果：

```
>>> from unittest import main
>>> main(module='test_module', exit=False)
```

failfast，catchbreak和buffer参数与同名命令行选项具有相同的效果。

warnings参数指定运行测试时应使用的警告过滤器。 如果未指定，如果将-W选项传递给python，则它将保持为None，否则将设置为“default”。

调用main实际上返回TestProgram类的实例。 这会将测试结果存储为result属性。

## load_tests Protocol

模块或包可以通过实现名为load_tests的函数来自定义在正常测试运行或测试发现期间如何从中加载测试。

如果测试模块定义了load_tests，则TestLoader.loadTestsFromModule（）将使用以下参数调用它：

```python
load_tests(loader, standard_tests, pattern)
```

其中pattern从loadTestsFromModule直接传递。 它默认为None。

它应该返回一个TestSuite。

loader是TestLoader执行加载的实例。 standard_tests是默认从模块加载的测试。 测试模块通常只想在标准测试集中添加或删除测试。 在将包作为测试发现的一部分加载时使用第三个参数。

从一组特定的TestCase类加载测试的典型load_tests函数可能如下所示：
```python
test_cases = (TestCase1, TestCase2, TestCase3)

def load_tests(loader, tests, pattern):
    suite = TestSuite()
    for test_class in test_cases:
        tests = loader.loadTestsFromTestCase(test_class)
        suite.addTests(tests)
    return suite
```

如果通过命令行或通过调用TestLoader.discover（）在包含包的目录中启动发现，则将检查包__init__.py是否为load_tests。 如果该函数不存在，则发现将递归到包中，就像它只是另一个目录一样。 否则，发现包的测试将留给load_tests，使用以下参数调用它：

```python
load_tests(loader, standard_tests, pattern)
```

这应该返回一个代表包中所有测试的TestSuite。 （standard_tests只包含从__init__.py收集的测试。）

因为模式被传递到load_tests，所以包可以继续（并可能修改）测试发现。 测试包的'do nothing'load_tests函数如下所示：

```python
def load_tests(loader, standard_tests, pattern):
    # top level directory cached on loader instance
    this_dir = os.path.dirname(__file__)
    package_tests = loader.discover(start_dir=this_dir, pattern=pattern)
    standard_tests.addTests(package_tests)
    return standard_tests
```

# 类和模块套件

类和模块级夹具在TestSuite中实现。当测试套件遇到来自新类的测试时，将调用前一个类（如果有的话）中的tearDownClass（），然后调用新类中的setUpClass（）。

类似地，如果测试来自与先前测试不同的模块，则运行前一模块的tearDownModule，然后运行新模块中的setUpModule。

在所有测试运行完毕后，运行最终的tearDownClass和tearDownModule。

请注意，共享套件与[潜在]功能（如测试并行化）不能很好地兼容，并且会破坏测试隔离。应小心使用它们。

unittest测试加载器创建的测试的默认顺序是将来自相同模块和类的所有测试组合在一起。这将导致每个类和模块只调用一次setUpClass / setUpModule（etc）。如果随机化顺序，以便来自不同模块和类的测试彼此相邻，则可以在单个测试运行中多次调用这些共享夹具函数。

共享套件不适用于具有非标准订购的套件。对于不想支持共享夹具的框架，仍然存在BaseTestSuite。

如果在其中一个共享夹具功能期间引发任何异常，则将测试报告为错误。由于没有相应的测试实例，因此创建\_ErrorHolder对象（具有与TestCase相同的接口）来表示错误。如果您只是使用标准的unittest测试运行器，那么这个细节并不重要，但如果您是框架作者，它可能是相关的。

## setUpClass 和 tearDownClass

这些必须作为类方法实现：

```python
import unittest

class Test(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        cls._connection = createExpensiveConnectionObject()

    @classmethod
    def tearDownClass(cls):
        cls._connection.destroy()
```

如果你想在调用的基类上使用setUpClass和tearDownClass，那么你必须自己调用它们。 TestCase中的实现是空的。

如果在setUpClass期间引发异常，则不会运行类中的测试，并且不会运行tearDownClass。 跳过的类不会运行setUpClass或tearDownClass。 如果异常是SkipTest异常，则该类将被报告为已跳过而不是错误。

## setUpModule and tearDownModule 

这些应该作为功能实现：

```python
def setUpModule():
    createConnection()

def tearDownModule():
    closeConnection()
```

如果在setUpModule中引发异常，则模块中的所有测试都不会运行，并且不会运行tearDownModule。 如果异常是SkipTest异常，则会将模块报告为已跳过而不是错误。

# 信号处理

unittest的-c /\-\-catch命令行选项以及unittest.main（）的catchbreak参数在测试运行期间提供了对控制-C的更友好的处理。 启用catch中断行为后，control-C将允许当前正在运行的测试完成，然后测试运行将结束并报告所有结果。 第二个control-c将以通常的方式引发KeyboardInterrupt。

control-c处理信号处理程序试图与安装自己的signal.SIGINT处理程序的代码或测试保持兼容。 如果调用unittest处理程序但不是已安装的signal.SIGINT处理程序，即它已被被测系统替换并委托给它，则它会调用默认处理程序。 这通常是代码替换已安装的处理程序并委托给它的预期行为。 对于需要禁用unittest control-c处理的单个测试，可以使用removeHandler（）装饰器。

框架作者有一些实用程序函数可以在测试框架中启用control-c处理功能。


**unittest.installHandler()**

安装control-c处理程序。 当收到signal.SIGINT时（通常是响应用户按下control-c），所有注册的结果都会调用stop（）。

**unittest.registerResult(result)**

注册TestResult对象以进行control-c处理。 注册结果会存储对它的弱引用，因此不会阻止结果被垃圾回收。

如果未启用control-c处理，则注册TestResult对象没有副作用，因此测试框架可以无条件地注册它们创建的所有结果，而与是否启用处理无关。

**unittest.removeResult(result)**

删除注册结果。 删除结果后，将不再在该结果对象上调用stop（）以响应control-c。

**unittest.removeHandler(function=None)**

如果不带参数调用此函数，则删除control-c处理程序（如果已安装）。 此函数还可以用作测试装饰器，以便在执行测试时临时删除处理程序：
```python
@unittest.removeHandler
def test_signal_handling(self):
    ...
```