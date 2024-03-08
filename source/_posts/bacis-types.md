---
title: 1.基本类型
date: 2017-10-10 17:32:03
tags: 
  - typescript
  - ts
categories:
- 翻译
- 编程语言
- TypeScript
---
# Boolean
最基本的数据类型为简单的true/false值，在TypeScript中被称为`boolean` 值。
```
let isDone:boolean = false;
```
# Number

和JavaScript 一样，所有数字都是浮点数，即`number`类型
除了十六进制和十进制字面量，TypeScript也支持ECMAScript 2015引入的二进制和八进制字面量

```typescript
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;
```

# String

在JavaScript中，编写程序的一项基本工作是对文本数据的处理。和其它的语言一样，我们使用`string`来表示这些文本数据类型。和JavaScript一样，TypeScript也使用双引号和单引号来表示字符串数据。
```typescript
let color: string = "blue";
color = 'red';
```

你也可以使用`模版字符串`,在模版字符串中可以嵌入表达式，同时，模版串内可以换行。模版串用反引号来表示，其内嵌的表达式形如`${ expression }`

```typescript
let fullName: string = `Bob Bobbington`;
let age: number = 37;
let sentence: string = `Hello, my name is ${ fullName }.

I'll be ${ age + 1 } years old next month.`;
```

这和按如下方式定义`sentence` 等价：
```typescript
let sentence: string = "Hello, my name is " + fullName + ".\n\n" +
    "I'll be " + (age + 1) + " years old next month.";
```

# 数组
和JavaScript一样，TypeScript允许你使用数组。数组的类型可有两种写法。
第一种是在元素类型的后面跟上一对方括号：
```typescript
let list:number[] = [1,2,3]
```
第二种是使用泛型数组参数:
```typescript
let list:Array<number> = [1,2,3]
```

# 元组(Tuple)

一个数组元素个数固定且类型已知，用元组来表示。例如，你可能需要表示一个`string`和一个`number`构成的二元组
```typescript
// Declare a tuple type
let x: [string, number];
// Initialize it
x = ["hello", 10]; // OK
// Initialize it incorrectly
x = [10, "hello"]; // Error
```
当我们访问一个索引已知的元组元素，那么这个元素的类型也就是已知的了。
```
console.log(x[0].substr(1)); // OK
console.log(x[1].substr(1)); // Error, 'number' does not have 'substr' typescript 知道x[1]是一个number类型的东西

```

当访问元组索引范围外的元素的时候，TypeScript会将其视为该元组已知类型的`并类型(Union types)`.
```typescript
x[3] = "world"; // OK, 'string' can be assigned to 'string | number'

console.log(x[5].toString()); // OK, 'string' and 'number' both have 'toString'

x[6] = true; // Error, 'boolean' isn't 'string | number'
```
并类型是一个高级话题，将在后文讨论。

# 枚举

除了JavaScript的数据类型，TypeScript还增加了一种有用的类型——枚举(enum). 和C#一样，枚举是给一组数字取一个友好的名字的方式。（注: typescript 2.4 中枚举也可以是字符串)
```typescript
enum Color {Red, Green, Blue}
let c: Color = Color.Green;
```

默认情况下，枚举值从0开始(后续加1)。你可以手动为枚举的某个成员设置一个值来改变其默认的值。
```typescript
enum Color {Red = 1, Green, Blue} 
let c: Color = Color.Green; // 2
```
或者，可以为枚举的每个成员设置值。
```typescript
enum Color {Red = 1, Green = 2, Blue = 4}
let c: Color = Color.Green;
```

一个方便的特性是你根据枚举值得到其对应的名字.
```typescript
enum Color {Red = 1, Green, Blue}
let colorName: string = Color[2];

console.log(colorName); // Green
```
# Any 类型
我们也许需要描述这样一种变量——我们在写程序的时候并不知道其类型，这些变量值可能来自一些动态的内容，例如——第三方库。这种情况，我们想对这些变量停用类型检察以便通过编译。为了做到这点，我们使用`any` 类型
```typescript
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

any 类型是和JavaScript库交互的有效方式，这可让你逐渐的启用或停用类型检察。你也许希望`Object`类现会扮演同样的角色，在其它某些语言中的却如此。但在typescript中，Object类型的变量只允许你将any类型的变量赋值给它，而不能调用上面的任意方法，那怕是的确存在的方法。

```typescript
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
```

# Void 类型
void有些像是和any相反，它表示——没有类型。你可能常在函数的返回值类型声明的地方见到该类型。
```typescript
function warnUser(): void {
    alert("This is my warning message");
}
```
用void来声明变量就没什么意义了，因为你只能给它赋`undefined`或`null`给它。

# Null 和 Undefined
默认情况下，null和undefined 是所有其它类型的子类型，所以null和undefined可以赋值给任何变量。
但当`--strictNullChecks`选项启用后，null和undefined就只能够赋值给`void`或其对应的类型（注：null，undefined 即是类型，也是值）。这有助于避免多数常见错误。
当你希望传递string或null或undefined的类型的值的时候则需要使用并类型 `string | null | undefined`。

>推荐使用`--strictNullChecks`,但该教程徦设该选项是关闭了的。

# Never 类型
never 类型代表了“`永远不会出现`”的类型，`never` 是那些总是抛出错误或永不返回的函数的“返回类型”()。
(Variables also acquire the type never when narrowed by any type guards that can never be true. ???)
never 类型也是所有类型的子类型，同时，没有类型是never类型的子类型，所以除了never自己，没有类型可以赋值给never。 any也不可以赋值给never。
函数返回类型为`never`的例子：
```typescript
// Function returning never must have unreachable end point
function error(message: string): never {
    throw new Error(message);
}

// Inferred return type is never
function fail() {
    return error("Something failed");
}

// Function returning never must have unreachable end point
function infiniteLoop(): never {
    while (true) {
    }
}
```

# 类型断言
有时候你会遇到这样的情况：你比TypeScript更清楚一个值是什么类型的。
类型断言是一种指定类型的方法，相当于告诉编译器：“相信我，我知道为在做什么”。类型断言和其它语言中的类型转换(type cast)是类似的，但不会执行特别的检察或数据重构操作。也没有运行时的开销，只是纯粹的为编译器使用而矣。TypeScript徦设你已经做过必要的检察了。
类型断言有两种形式，一种是`尖括号`语法:
```typescript
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;
```
另一种是`as`语法：
```typescript
    let someValue: any = "this is a string";

let strLength: number = (someValue as string).length;
```

这两种方式是等价的，使用哪种取决于你的偏好。但是，如果在TypeScript中使用JSX, 则只有`as`方式是可用的（注：JSX 用尖括号了表示组件，这会引入二意性）。

# 关于 `let`

你可能注意到了，我们使用`let`关键字代替了JavaScript中的`var`关键字。let 实际上是JavaScript 新标准的东西，TypeScript使其提前可用了。后面我们会进一步讨论let，let可以缓解JavaScript中的很多问题，所以，尽可能使用let来代替var吧！

# 进一步阅读
[never 和 void 的区别](https://stackoverflow.com/questions/37910669/what-is-the-difference-between-never-and-void-in-typescript)
