---
title: 箭头函数
categories:
- 编程
tags:
- 抄抄抄
- ES6
auto_spacing: true
---

#### 基本定义
```
  // ES5
  var selected = allJobs.filter(function (job) {
    return job.isSelected();
  });

  // ES6
  var selected = allJobs.filter(job => job.isSelected());
```

- 当函数只有  __一个参数__ 的时候，可以使用新标准中的箭头函数，它的语法非常简单：标识符=>表达式。你无需输入function和return，一些小括号、大括号以及 __分号__ 也可以省略。

<!-- more -->

- 如果要写一个接受多重参数（也可能没有参数，或者是不定参数、默认参数、参数解构）的函数，你需要用 __小括号__  包裹 __参数list__。

```
  // ES5
    var total = values.reduce(function (a, b) {
      return a + b;
    }, 0);
  // ES6
    var total = values.reduce((a, b) => a + b, 0);
```

如果函数逻辑稍微复杂一些，不能一行代码就完成的。怎么办呢？

- 除表达式外，箭头函数还可以包含一个 __块语句__

```
  // ES5
  $("#confetti-btn").click(function (event) {
    playTrumpet();
    fireConfettiCannon();
  });
```
这是它们在ES6中看起来的样子：
```
  // ES6
  $("#confetti-btn").click(event => {
    playTrumpet();
     fireConfettiCannon();
  });
```

#### 返回对象字面量
##### 注意！！这里有个坑

 使用了块语句( { } ) 的箭头函数不会自动返回值，你需要使用return语句将所需值返回。

 这样有什么问题呢？

```
   // 为与你玩耍的每一个小狗创建一个新的空对象
   var chewToys = puppies.map(puppy => {});   // 这样写会报Bug！
```
你以为 你为每一个小狗创建一个新的空对象，但是 其实根本不是，这样是 __会报错__ 的,为什么呢?

因为箭头函数还可以包含一个 __块语句__！！！

ES6中的规则是，__紧随__ 箭头的 { 被解析为块的开始，而不是对象的开始。因此，puppy => {} 这段代码就被解析为没有任何行为并返回undefined的箭头函数 !!

所以

返回对象字面量时

必须要用 小括号将返回的对象 括起来！！
--

```
  var chewToys = puppies.map(puppy => ({}))
```

#### this

箭头函数还有一个 特点

- 它 __没有自己的this__ 值，箭头函数内的this值 __继承自外围作用域__。

```
  {
    ...
    addAll: function addAll(pieces) {
      var self = this;

      _.each(pieces, function (piece) {
        self.add(piece);
      });
    },
    ...
  }
```
在上面这个对象的 addAll 方法中， 有一个 匿名函数，这个匿名函数是拿不到 外部函数的 this 的 ( 内部 上下文，拿不到 外部的 this, arguments )，而我们需要用 this.add 这个 方法，所以 我们，外部能拿到 this 的地方，将 this 赋值给另一个 内部函数可以访问到 变量, 然后我们在内部函数通过访问这个变量来访问 this.add 方法。

__但是用 箭头函数 就不用这么麻烦了。__

```
  // ES6
   {
     ...
     addAll: function addAll(pieces) {
       _.each(pieces,  piece => this.add(piece));
     },
     ...
   }
```

在这里 __addAll__ 方法先从它的 __调用者__处获取了this值，然后由于内部函数是一个 __箭头函数__，所以它继承了 __外围作用域__的this值 ( 即 addAll )。

#### 使用 call 或 apply 调用

- 由于 this 已经在词法层面完成了绑定，通过 call() 或 apply() 方法调用一个函数时，只是传入了参数而已，对 this 并 __没有什么影响__
-
```
  var adder = {
    base : 1,

    add : function(a) {
      var f = v => v + this.base;
      return f(a);
    },

    addThruCall: function(a) {
      var f = v => v + this.base;
      var b = {
        base : 2
      };

      return f.call(b, a);
    }
  };

  console.log(adder.add(1));         // 输出 2
  console.log(adder.addThruCall(1)); // 仍然输出 2（而不是3 ——译者注）
```

#### 不绑定 arguments
- 箭头函数还有一个特性， 不绑定 arguments，箭头函数不会在其内部暴露出  arguments 对象
- 若在函数中将 arguments 赋值给某一变量，则这个变量将会指向，这个箭头函数所在的作用域一个 名为 arguments 的值，如果没有的话就返货 undefined

```
  var arguments = 42;
  var arr = () => arguments;

  arr(); // 42

  function foo() {
    var f = () => arguments[0]; // foo's implicit arguments binding
    return f(2);
  }

  foo(1); // 1
```

上面箭头函数的 arguments 是指向，外部函数的 arguments, 而不是自身的。

- 虽然箭头函数没有自己的 arguments 对象，但是大多数情形下，rest参数可以给出一个解决方案

  ```
  function foo() {
    var f = (...args) => args[0];
    return f(2);
  }

  foo(1); // 2
```
#### Parsing order ( 这个不理解 @ @ )

The arrow in an arrow function is not an operator, but arrow functions have special parsing rules that interact differently with (operator precedence)[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence], compared to a regular function.

```
  let callback;

  callback = callback || function() {}; // ok
  callback = callback || () => {}; // SyntaxError: invalid arrow-function arguments
  callback = callback || (() => {});    // ok
```

####  箭头函数的运用的地方

在ES6中，你需要遵循以下规则：

- 通过object.method()语法调用的方法使用非箭头函数定义，这些函数需要从调用者的作用域中获取一个有意义的this值。
- 其它情况全都使用箭头函数。

{% blockquote %}

参考：

[深入浅出ES6（七）：箭头函数 Arrow Functions
](http://www.infoq.com/cn/articles/es6-in-depth-arrow-functions/)

[箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

{% endblockquote %}
