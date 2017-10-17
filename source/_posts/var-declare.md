---
title: 2.变量声明
date: 2017-10-13 22:30:02
tags: typescript, ts
categories:
- 翻译
- 编程语言
- TypeScript
---

# 变量声明
`let` 和`const`是JavaScript中的两种声明方式。就像前文说过的那样，在某些方面`let`和`var`是相似的，但是使用`let`可以避免许多JavaScript程序员常常遇见的陷阱。`const` 是对`let`的增强，它可以防止对变量的从新赋值。

TypeScript是JavaScript的超集，所以`let`和`const`在TypeScript中自然可用。本文将会仔细介绍这两种声明方式，。。。

# var 声明方式

在JavaScript中一直(注：ES6之前)使用var关键字来声明变量。
```
var a=10;
```
就如你所想，这里声明了一个叫`a`的变量，并且赋值10给它。
我们也可以在函数中声明一个变量：
```javascript
function f() {
    var message = "Hello, world!";
    return message;
}
```

我们也可以在内嵌的其它函数中访问这些变量：
```javascript
function f() {
    var a = 10;
    return function g() {
        var b = a + 1;
        return b;
    }
}

var g = f();
g(); // returns '11'
```
这段代码中，函数g捕获了变量a，当g被调用的时候，尽管这时候f的调用已经完成了，a都可以被访问，就像f中的代码访问a一样。
```javascript
function f() {
    var a = 1;

    a = 2;
    var b = g();
    a = 3;

    return b;

    function g() {
        return a;
    }
}

f(); // returns '2'
```

# 作用域规则
var 声明方式拥有其它语言没有的奇怪的作用域（注:），举例来说：
```javascript
function f(shouldInitialize: boolean) {
    if (shouldInitialize) {
        var x = 10;
    }

    return x;
}

f(true);  // returns '10'
f(false); // returns 'undefined'
```
有些读者对于这个例子可能要多思考两次（即懵一下），变量`x`在if块中声明，但我们却能在if块的外面访问它（注：在常见的语言中会有一个编译时错误：x 未声明）！导致这一现象的原因是，var 方式声明的变量在其声明所在的函数、模块或全局作用域总是可见的。有人把这称为var作用域或函数作用域。函数参数也是函数作用域。

```javascript
function sumMatrix(matrix: number[][]) {
    var sum = 0;
    for (var i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (var i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```
有了上面的经验，有的读者可能容易看出问题所在。最内的for循环的`i`覆盖了外层i的值，因为i的作用域在这整个函数！类似的bug不容易在code review时（注：代码审查，开发环节）被发现，而成为问题之源。

# 变量捕获的怪异行为
快速指出下列代码的输出：
```javascript
for (var i = 0; i < 10; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```

(致不熟悉setTimeout的人：setTimeout会在指定的多少毫秒后执行指定的函数）

答案是什么呢？
```
10
10
10
10
10
10
10
10
10
10
```
许多JavaScript开发人员熟知JavaScript的这一特性，但如果你有些诧异，你也不孤单，因为很多人以为的输出是：
```
0
1
2
3
4
5
6
7
8
9
```
还记得前面提过的变量捕获吗？每一个传递给setTimeout的函数捕获了同作用域下的同一个变量i。
让我们稍稍想一下这意味着什么。setTimeout 会在若干毫秒后执行传给它的函数（注：哪怕是setTime(fn,0),请参考《异步JavaScript》）,这时候for循环已经完成，i 是 10。所以，之后运行的函数（setTimeout的回调）输出都是10。

一个常用的解决方案是在每次循环中用`立即执行函数表达式(IIFE,Immediately Invoked Function Expression)`来捕获`i`
```javascript
for (var i = 0; i < 10; i++) {
    // capture the current state of 'i'
    // by invoking a function with its current value
    (function(i) {//在了的i覆盖了上层作用域的i
        setTimeout(function() { console.log(i); }, 100 * i);
    })(i);
}
```
这种看起来奇奇怪怪的模式实际上是非常通用的。参数i实际上覆盖了for循环中的i，只是名字相同而矣（注：若理解起来别扭，把IIFE内部的i改成j)。

>如下
```javascript
for (var i = 0; i < 10; i++) {
    // capture the current state of 'i'
    // by invoking a function with its current value
    (function(j) {
        setTimeout(function() { console.log(j); }, 100 * j);
    })(i);
}
```

# `let` 方式声明
现在，你已经知道`var`的这些问题了，这也正是`let`被引入的原因，除了关键字不同，两种声明的写法是一样的：
```typescript
let hello = "Hello!";
```
关键的区别不在于语法，而在于语义——我们即将介绍。

# 块作用域
当一个变量用`let`声明后，它使用所谓的`词法作用域`（或称`块作用域`）.不同于`var` 会将作用域“泄露”到其所在函数，块作用域只在其块内可见，例如for循环体。

```typescript
function f(input: boolean) {
    let a = 100;

    if (input) {
        // Still okay to reference 'a'
        let b = a + 1;
        return b;
    }

    // Error: 'b' doesn't exist here
    return b;
}
```
在段代码中，我们声明了两个变量——a、b，a的作用域在f的函数体内，而b的作用域在if的语句块内。

在catch语句中声明的变量也有自己的块作用域：
```typescript
try {
    throw "oh no!";
}
catch (e) {
    console.log("Oh well.");
}

// Error: 'e' doesn't exist here
console.log(e);
```
块作用域变量的另一个性质是在其声明前是不可以被读写的。（注：有句废话没翻译）

```typescript
a++; // illegal to use 'a' before it's declared;
let a;
```
值得注意的是，对于块级变量，你仍可以在声明前捕获它。
```typescript
function aFun(){
    function foo(){
        console.log(a);
    }
    // illegal call 'foo' before 'a' is declared
    // runtimes should throw an error here
    /*
     * 注：是否有runtime错误要看tsc编译参数，如果编译成ES5及更早版本则没有runtime错误
     * 请对比（徦设文件名为test.ts）：
     * tsc -t ES5 test.ts 
     * tsc -t ES2016 test.ts
     */
    foo();
    let a=1;
    return foo;
}
```
# 重声明以及覆盖

上文提及的var方式声明的变量可以在同一作用域被声明多次，而不会报错：
```typescrpt
function f(x) {
    var x;
    var x;

    if (true) {
        var x;
    }
}
```
在这个例子中，所有对`x`的声明都指向了同一个`x`，这样做的后果常常是成为了bug之源。因此,`let`不允许这样的声明。
```typescript
let x = 10;
let x = 20; // error: can't re-declare 'x' in the same scope
```
TypeScript 会告诉我们，同一个块作用域不能有两个同名的变量。
```typescript
function f(x) {
    let x = 100; // error: interferes with parameter declaration
}

function g() {
    let x = 100;
    var x = 100; // error: can't have both declarations of 'x'
}
```
当然如果是父——子作用域是可以的：
```typescript
function f(condition, x) {
    if (condition) {
        let x = 100;
        return x;
    }

    return x;
}

f(false, 0); // returns '0'
f(true, 0);  // returns '100'

```
这种在子作用域声明一个和父作用域同名的变量的形为就是——变量覆盖（shadowing)。这是一把双刃剑，在“不小心”覆盖了另一个变量的时候就可能会引入一些bug，同时变量覆盖也能访止一些bug，如前面的`sumMatrix`
```typescript
function sumMatrix(matrix: number[][]) {
    let sum = 0;
    for (let i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (let i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```
这段代码是能够完成矩阵求和的，因为内层for循环的i覆盖了外层循环的i，那么外层的i就不会被意外的修改了。

在意于写清晰明了的代码的时候应该必免使用变量覆盖这一特性。尽管在有些场合，这一特性会带来一些优势，你最好仔细考量。

# 块作用域变量捕获

当我们首次了解var变量的捕获的情况的时候，我们粗略的调查了变量
被捕获后的形为。详细说来，每当一个作用域（中的代码）在执行的时候，有一个包含变量的“环境”被创建出来。这个环境和其捕获的变量在其所在的作用域执行完成后依然可以存在！
```typescript
function theCityThatAlwaysSleeps() {
    let getCity;

    if (true) {
        let city = "Seattle";
        getCity = function() {
            return city;
        }
    }

    return getCity();
}
```
因为我们捕获了`city`变量，所以在if块执行之后我们仍可以访问它。
回想前面`setTimeout`那个例子，我们需要使用IIFE来捕获变量。
实际上，我们创建了一个新的变量环境来捕获变绿。IIFE的方式写起来有些老火，幸运的是在TypeScript中不必那样。
`let`方式在循环语句中声明的变量很是不同，除了创建一个新的环境，这类声明（指let和const）也会在每次循环的时候创建一个新的作用域，这正是我们使用IIFE所作的事情。修改一下`setTimeout`那个例子，使用`let`来声明变量：
```typescript
for (let i = 0; i < 10 ; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```
输出则入我们所想的那样：
```
0
1
2
3
4
5
6
7
8
9
```

# const 声明

const是声明变量的另一种形式：
```typescrpt
const numLivesForCat = 9;
```

它和let很像，但是，就像其名字所暗示的那样，其值是不能够被修改的。换句话说，`const`和`let`具有一样的作用域规则，但是你不能为之重赋值。

可别和`不可变（immutable）`相混淆：
```typescript
const numLivesForCat = 9;
const kitty = {
    name: "Aurora",
    numLives: numLivesForCat,
}

// Error
kitty = {
    name: "Danielle",
    numLives: numLivesForCat
};

// all "okay"
kitty.name = "Rory";
kitty.name = "Kitty";
kitty.name = "Cat";
kitty.numLives--;
```
除非你采取了特殊措施，`const`变量的内部状态（注：对像的属性，数组的元素等）是可变的（注：const 变量指的是引用不变）。所幸，TypeScript可指定属性为`readonly`来避免属性被修改。[接口]()章会详述这个问题。


# `let` vs. `const`

设计两种具有相似作用域语义的变量声明方式，这使我们自然会问“使用哪个?”。答案和很多问题一样：看情况。

根据[最少特权原则](https://en.wikipedia.org/wiki/Principle_of_least_privilege),所有无计划修改的变量应声明成`const`。原因是，如果一个变量不需要被修改，那么其头代码也不应有能修改这些变量的能力，也需要思考它们是否真得需要重赋值这些变量。使用`const`也使的在推算数据流的时候（注：即在脑中执行代码）代码的行为更可被预测。

[placeholder]

# 解构


TypeScript拥有的es2015的另一个特性是`解构`。[这里]是关于解构的完整说明，本节将大略的描述解构。


## 数组解构

最简单的数组解构是数组的解构赋值。
```typescript
let input = [1, 2];
let [first, second] = input;
console.log(first); // outputs 1
console.log(second); // outputs 2
```
这里创建了两个变量，first和second，和使用索引的方式效果一个，但是更加方便。
```typescript
first = input[0];
second = input[1];
```

解构赋值也可使用在已经声明的变量上，
```typescript
// swap variables
[first, second] = [second, first];
```

以及用在函数的参数中：
```typescript
function f([first, second]: [number, number]) {
    console.log(first);
    console.log(second);
}
f([1, 2]);
```

你可以使用...语法来创建一个list变量来保存剩余元素：

```typescript
let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // outputs 1
console.log(rest); // outputs [ 2, 3, 4 ]
```

当然，你也可以忽略你不关心的尾部的元素。
```typescript
let [first] = [1, 2, 3, 4];
console.log(first); // outputs 1
```
当然，其它元素也是可忽略的：
```typescript
let [, second, , fourth] = [1, 2, 3, 4];
```

## 对象解构

你也可以解构对象：
```typescript
let o = {
    a: "foo",
    b: 12,
    c: "bar"
};
let { a, b } = o;
```

这里从a对象创建了a、b变量，其值分别是`o.a`,`o.b`并忽略了`a.c`。

和数组解构一样你也可以不要声明而赋值：
```typescript
({ a, b } = { a: "baz", b: 101 });
```
注意，我们必须用园括号把代码包起来，js会把{作为块来解析
你也可以使用`...`语法来创建一个包含其它未被解构属性的变量

```typescript
let { a, ...passthrough } = o;
let total = passthrough.b + passthrough.c.length;
```

### 属性重命名


你也可以给属性一个不同的名字
```typescript
let { a: newName1, b: newName2 } = o;
```
这种语法开始让人有点迷糊，你可以把`a:newName1`解释为`newName1是a`,这和下面的代码是等价的：

```typescript
let newName1 = o.a;
let newName2 = o.b;
```
这里的`：`不是表示类型，如果你想指定变量的类型，那么需要在整个解构后。
### 默认值

默认值可以在某个属性是undefined的时候指定一个默认的值。
```typescript
function keepWholeObject(wholeObject: { a: string, b?: number }) {
    let { a, b = 1001 } = wholeObject;
}
```
这里,在函数`keepWholeObject`中有变量:`a`,`b`,以及`wholeObject`。

### 函数声明
解构也可用在函数的声明中，如下例：
```typescript
type C = { a: string, b?: number }
function f({ a, b }: C): void {
    // ...
}
```
[placeholder]

## 延展

>不知道spread对应的术语，乱译为`延展`

spread运算和解构运算作用是相对的，它可以把一个数组中的元素放到其他数组中。或者把一个对象的属性放到其它对象中。
比如：
```typescript
let first = [1, 2];
let second = [3, 4];
let bothPlus = [0, ...first, ...second, 5];
```
`bothPlus`数组为[0,1,2,3,4,5],`Spread`创建了`first`和`second`的**浅拷贝**
你也可以`spread`一个对象
```typescript
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { ...defaults, food: "rich" };
```
`search`为`{ food: "rich", price: "$$", ambiance: "noisy" }`,对象的延展比数组的延展要复杂一些。和数组延展一样，处理的过程是“从左到右的”,只是处理的结果仍然是一个数组而矣。这意味着，后出现的属性将覆盖先出现的属性。所以，如果我们修改上面的例子：
```typescript
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { food: "rich", ...defaults };
```
那么得到的search的food属性的值将会是`spicy`。

对象延展还有一些限制，结果只会包含对象[自身的、可枚举的属性](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)(注：自身的表示非原型链上的属性)
所以，当你延展一个对象时，将丢失其上的方法。
```typescript
class C {
  p = 12;
  m() {
  }
}
let c = new C();
let clone = { ...c };
clone.p; // ok
clone.m(); // error!
```
此外，Typescript编译器不允许泛型函数的类型参数。这一特型在将来的版本中可能会受支持。