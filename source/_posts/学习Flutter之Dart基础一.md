---
title: 学习Flutter之Dart基础（一）
date: 2019-07-22
tags: [iOS,Flutter ,Dart]
category: [iOS,Flutter ,Dart]
---
## Dart

#### 函数（方法）

`return type`  ` functionName（paramName: value）{` 

`//函数体`

code ...

`}`

如果方法体只有一句代码，可以使用缩写形式（胖箭头形式）：`=>表达式`。

- 方法参数有必须参数和可选参数，必现参数放在前面，可选参数放后面，

- 可选参数：使用中括号括起来：`[paramName: value]`。

- 参数默认值：在参数列表中，使用`=`对参数进行默认值设置`[type paramName = value]`。默认参数是编译常量，在传入的参数为空时，使用默认值。

#### const 和final

`const` 和`final`修饰的变量，是不可修改的。

`const`修饰的为编译常量。

`final`修饰的只能赋值一次，且在第一次使用的时候赋值。

如果一个类的构造函数使用`const`修饰，则函数返回的对象是不可修改的。



### string

`String`： 字符串字面量 使用 `''`或者`""`括起来。

`string1 == string2 `比较的是两个字符串的字符编码。如果字符编码一直，则认为两个字符串相等。

`${表达式}`是占位符（插值）表达式，`{}`可以省略。如果表达式的结果是一个对象，则默认调用对象的`toString()`方法来获取字符串。

使用三个单引号 \`\`\` 或者双引号`"""`来定义多行字符串。



### bool

`bool`布尔类型，只有两个值，`true`和`false`，跟其他语言区别是，其他语言只要对象非空，则会转为`true`，在

`Dart`中，都会转为`false`。比如OC中的`NSObject != nil，判定为true`，在dart中，因为`object不等于 true` ，所以判断会为`false`对于一个空对象，也将判定为`false`。	



### List

有序集合（array）使用`List`声明，使用中括号来表示`[object0,object1,...]`。

`List`默认是可变的，不可变`List`使用`const`修饰。`const`修饰的`List`，在比如在进行`add`操作时，编译器抛出异常：


``` dart

Uncaught exception:
Unsupported operation: add
```



### maps

字典集合使用`map`声明，使用一对花括号`{}`来表示。内容为键值对。

`map`默认是可变的，不可变`map`使用`const`修饰。



### Runes

通常使用 `\uXXXX` 的方式来表示 Unicode code point， 这里的 XXXX 是4个 16 进制的数。 例如，心形符号 (♥) 是 `\u2665`。 对于非 4 个数值的情况， 把编码值放到大括号中即可。 例如，笑脸 emoji (😆) 是 `\u{1f600}`。

在`dart`中，Unicode默认是16位的UTF-16编码，当需要表示32位的Unicode，使用runes表示字符串的UTF-32code编码序列。
```