---
nav: blog
layout: post
title: "流畅的python - Dynamic attributes and properties"
author: "wangchao"
tags:
  - python
  - 'Future'
  - '并行'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[你不知道的javascript(上)](http://www.ituring.com.cn/book/1488)


- [第一章作用域](#chcapter01)

<span id="chcapter01"></span>

### 第一章作用域

- 作用域是一套规则，用于确定在何处以及如何查找变量（标识符）。如果查找的目的是对变量进行赋值，那么就会使用`LHS查询`；如果目的是获取变量的值，就会使用`RHS查询`。
- 赋值操作符会导致`LHS查询`。＝操作符或调用函数时传入参数的操作都会导致关联作用域的赋值操作。
- JavaScript引擎首先会在代码执行前对其进行编译，在这个过程中，像`var a = 2`这样的声明会被分解成两个独立的步骤：
   - 首先，`var a`在其作用域中声明新变量。这会在最开始的阶段，也就是代码执行前进行。
   - 接下来，`a = 2`会查询（LHS查询）变量a并对其进行赋值。
- `LHS和RHS查询`都会在当前执行作用域中开始，如果有需要（也就是说它们没有找到所需的标识符），就会向上级作用域继续查找目标标识符，这样每次上升一级作用域（一层楼），最后抵达全局作用域（顶层），无论找到或没找到都将停止。
- 不成功的`RHS引用`会导致抛出`ReferenceError异常`。不成功的`LHS引用`会导致自动隐式地创建一个`全局变量`（非严格模式下），该变量使用`LHS引用`的目标作为标识符，或者抛出`ReferenceError异常`（严格模式下）。     
     
** 小测验答案 **

```javascript
function foo(a) {
    var b = a;
    return a + b;
}

var c = foo( 2 );
```
1. 找出所有的LHS查询（这里有3处！）
   * `c = ..;、a = 2（隐式变量分配）、b = ..`
2. 找出所有的RHS查询（这里有4处！）
   * `foo(2..、= a;、a ..、.. b`

