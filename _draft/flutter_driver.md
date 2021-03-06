## find top-level constant 
const find=const CommonFinders._()
A convenient accessor to frequently used finders.

Examples:

~~~dart
driver.tap(find.text('Save'));
driver.scroll(find.byValueKey(42));
~~~

### Implementation

~~~dart
const CommonFinders find = CommonFinders._()
~~~

## tap method

~~~dart
Future<void> tap (
    SerializableFinder finder, { 
    Duration timeout
}) 
~~~

Taps at the center of the widget located by `finder`.

### Implementation

~~~dart
Future<void> tap(SerializableFinder finder, { Duration timeout }) async {
  await _sendCommand(Tap(finder, timeout: timeout));
}
~~~

## SerializableFinder类
Flutter Driver finders的基类, 描述驱动程序应如何搜索元素的对象。

### 构造函数
SerializableFinder()
A const constructor to allow subclasses to be const.
const


### 属性
**finderType** →** String**
Identifies the type of finder to be used by the driver extension.
标识 driver extension要使用的finder类型。 
read-only

**hashCode** → **int**
这个对象的hash值
read-only, inherited

**runtimeType** → **Type**
对象运行时类型的表示
read-only, inherited

### 方法
**serialize**() → **Map**<String, String>
将common fields序列化为 JSON
@mustCallSuper

**noSuchMethod**(Invocation invocation) → **dynamic**
当访问不存在的方法或属性时调用
inherited

**toString**() → **String**
返回此对象的字符串表示形式
inherited

### 运算符

operator ==(dynamic other) → bool
相等运算符
inherited

### 静态方法
deserialize(Map<String, String> json) → SerializableFinder
Deserializes a finder from JSON generated by serialize. 

## CommonFinders类
频繁地使用的finders提供方便的访问器。

### 属性

**hashCode** →  **int**
这个对象的hash值
read-only, inherited

**runtimeType** →  **Type**
对象运行时类型的表示

### 方法

**ancestor**({**SerializableFinder** of, **SerializableFinder** matching, **bool** matchRoot: false }) →  **SerializableFinder**
查找作为of参数的原型且与matching参数匹配的widget 。 

**descendant**({SerializableFinder of, SerializableFinder matching, bool matchRoot: false }) → **SerializableFinder**
查找作为of参数的子代且与matching参数匹配的widget 。 

**bySemanticsLabel**(**Pattern** label) → **SerializableFinder**
查找具有给定语义label的widgets。

**byTooltip**(**String** message) → **SerializableFinder**
查找带有给定message的tooltip的widgets。

**byType**(String type) → **SerializableFinder**
查找其类名与给定字符串type匹配的widgets。 

**byValueKey**(dynamic key) → **SerializableFinder**
按key查找小部件。只能使用String 和int值。

**pageBack**() → SerializableFinder
 在Material或Cupertino页面的框架上查找后退按钮。

**text**(String text) → **SerializableFinder**
查找包含等于text的字符串的文本和可编辑文本widgets

**noSuchMethod**(Invocation invocation) → **dynamic**
 当访问不存在的方法或属性时调用。 
 inherited

**toString**() → **String**
返回此对象的字符串表示形式

### 运算符

**operator ==**(dynamic other) → bool
相等运算符
inherited