---
title: 3.接口
date: 2017-10-26 22:30:02
tags: typescript, ts
categories:
- 翻译
- 编程语言
- TypeScript
---

# Introduction
TypeScript的核心概念之一就是类型检查，Typescript的类型检查是基于值的“形状”而言的，这种类型被称为“鸭类型”或“结构化类型”（注：如果一种生物走起路来像鸭子，叫起来像鸭子，就认为它是鸭子）。在TypeScript中，是给类型“命名”的一种角色，也是种约束你的代码的有效方式。

# 第一个接口

关于接口最简单的说明，如下例：
```typescript
function printLabel(labelledObj: { label: string }) {
    console.log(labelledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);

```
类型检查器检查对`printLabel`的调用，该函数需要一个参数，这个参数是一个有串类型的`label`属性的对象。调用时传递给该函数的参数实际上除了`label`属性，还有些其它属性。编译器只会确保有有相匹配的那些属性，但也有一些情况不是这样简单的处理。
我们可以改写这个例子，这次我们使用一个接口来描述`printLabel`的参数。

```typescript
interface LabelledValue {
    label: string;
}

function printLabel(labelledObj: LabelledValue) {
    console.log(labelledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

现在我们可用`LabelledValue`这个接口来描述`printLabel`的参数。我们并没有明确的让传递给`printLabel`的参数要实现`LabelledValue`这个接口，在其它语言中可能需要这样做。这里只在乎的是“形状”。如果我们传递过去的东西和`LabelledValue`是兼容的就可以的。

值得说明的是，类型检察器不在意属性出现的顺序，只在有意识必要的那些属性以及这些属性的类型是正确的。



#  可选属性
接口中并不是每个属性都是必须的，有的属性在某些情况下才会出现，甚至不会出现。在使用所谓的“option bags”（注:即把所有的选项放在一个对象里面）的模式的时候可选属性是很常用的。        
```typescript
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
    let newSquare = {color: "white", area: 100};
    if (config.color) {
        newSquare.color = config.color;
    }
    if (config.width) {
        newSquare.area = config.width * config.width;
    }
    return newSquare;
}

let mySquare = createSquare({color: "black"});
```
有可选属性的接口的语法和普通接口是类似的，只是每个可选属性的名字后面用一个`?`标记出来。

可选属性的优势是你可以描述那些可能出现的属性而且避免那些没有在接口中声明的属性(注:可防止拼写错误)。比如，我们把`color`错写成了`clor`，TypeScript将给出一个错误消息。
```typescript
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    let newSquare = {color: "white", area: 100};
    if (config.color) {
        // Error: Property 'clor' does not exist on type 'SquareConfig'
        newSquare.color = config.clor;
    }
    if (config.width) {
        newSquare.area = config.width * config.width;
    }
    return newSquare;
}

let mySquare = createSquare({color: "black"});
```
# 只读属性
一些属性只应该在一个对象创建的时候被修改，你可以通过在属性名前加一个`readonly`关键字来说明这点。
```typescript
interface Point {
    readonly x: number;
    readonly y: number;
}
```
你可以通过对象字面量的方式来创建`Point`对象，一旦创建对象后，就不再能修改`x`和`y`的值了。
```typescript
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // error!
```
TypeScript有一个`ReadonlyArray<T>`类型，基本和`Array<T>`一样，只不过那些修改类的方法被移除了，所以你可以确保数组在被创建之后就不再被修改了。
```typescript
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // error!
ro.push(5); // error!
ro.length = 100; // error!
a = ro; // error!
```
这段代码的最后一行表明，你不可以把一个ReadOnlyArray赋值给一个Array变量。
除非你使用类型断言:
```typescript
a=ro as number[]
```
**readonly** vs **const**
区别使用`readonly`还是`const`的最简单的方式是看是在属性上还是在变量上。前者使用`readonly`后者使用`const`


# 多余属性检查

在我们的第一个例子中，我们把`{ size: number; label: string; }`传递给接受`{ label: string; }`的函数。我们也了解了可选属性，以及其在"option bags"时候的用处。
然而，这两者简单的合起来用却会遇到和JavaScript中一样的麻烦。例如:
```typescript
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
```
注意这里在调用`createSquare`时候传递的参数中用的是`colour`而非`color`.在JavaScript中，这种错误会静悄悄的发生。你会觉得你的程序是对的:`width`的类型是兼容的，没有`color`属性，多出一个`colour`属性。
然而，TypeScript的立场是这也许会是程序中的一个bug。***当一个对象被赋值 给其它变量，或通过参数传递的时候，对象会被特殊对待，经过所谓的"`多余属性检查`"**。如果该对象含有目标类型所没有的属性，就会报错:
```typescript
// error: 'colour' not expected in type 'SquareConfig'
let mySquare = createSquare({ colour: "red", width: 100 });
```
要通过这种检查也是十分简单的，使用类型断言就可以了:
```typescript
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```
但是，一个更好的方法是添加字符串索引签名(string index signature),当然，是在如果你确定被传递的对象是可有一些额外的属性的情况下。如果`SquareConfig`可有其它的属性，你可以这样定义它:
```typescript
interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
```
简单讨论一下索引签名。我们说`SquareConfig`可有任何数量的属性。只有这些属性的名字不是`color`和`width`，其类型是any。
还有一种通过检察的方式——这种放式可能会让人惊讶——把对象赋值给另外一个变量:
```typescript
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```
因为`squareOptions`不会经历`多余属性检查`，那么也就不会有编译错误了。

记住，对于这段简单代码，你也许会认为不会通过检察。对于更复杂的对象字面量，它们有一些方法和状态变量，所以直接传递一个对象而非对象字面量的时候,TypeScript不会对其进行`多余属性检察`。而以对象字面量为`option bags`这样的参数的时候，`多余属性检察`的确可以必免很多bug.这也意味着，如果你在用`option bags`时遇到了`多余属性检察`报的错误，你也许就需要修改你的类型定义。例如，对于上面的例子，如果拥有`color`和`colour`属性的对象都可以作为`createSquare`的参数，那么你就需要修改`SquareConfig`的定义了。
# 函数类型

TypeScript中的`接口`这一概念可广泛的用来描述JavaScript中的东西。除了用来描述对象及其属性，接口也能用来描述函数的类型。

为了用接口来描述函数，我们给这些接口一个`调用签名`。这类似于函数的声明，参数表中的每个参数都要有类型和名字。
```typescript
interface SearchFunc {
    (source: string, subString: string): boolean;
}
```
一旦定义了这样的接口，我们就可以像使用普通接口一样的使用它。
下面的例子演示了如何用函数接口来定义一个变量并为其赋值。
```typescript
let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
    let result = source.search(subString);
    return result > -1;
}
```
对函数类型的类型检察不要求参数的名字相匹配:
```typescript
let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
    let result = src.search(sub);
    return result > -1;
}
```
如果将函数赋值给指明类型的变量，例如`SearchFunc`，而你没有指定参数的类型，TypeScript的`上下文类型`系统能够推断出参数的类型。
```typescript
let mySearch: SearchFunc;
mySearch = function(src, sub) {
    let result = src.search(sub);
    return result > -1;
}
```
这里函数的返回值暗示了其类型(`false`或`true`).如果这里的返回值不是布尔类型的，TypeScript将会给出一个类型不匹配的警告。

# 可索引的类型
除了可用接口来描述函数，我们有可用接口来描述`索引`，类似于`a[10]`或`ageMap["daniel"]。可索引的类型有一个`索引签名`，其用于描述我们如何来索引对象中的值，以及说明`索引`和返回值的类型。
例如:
```typescript
interface StringArray {
    [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```
这里，我们定义了一个有索引签名的`StringArray`接口。这个索引签名说明`StringArray`可用数字来作索引并返回一个`string`类型的值。

可用作索引的类型有`string`和`number`两种。可在同一个接口中使用这两种索引，但是数字索引的返回值类型必须是串索引的**子类型**。这是因为当我们用数字作为索引，JavaScript会把它转换为字符串。即用`100`作索引实际上和用`"100"`作索引是同一回事，所以我们需要这两种类型一致。

```typescript
class Animal {
    name: string;
}
class Dog extends Animal {
    breed: string;
}

// Error: indexing with a 'string' will sometimes get you an Animal!
interface NotOkay {
    [x: number]: Animal;
    [x: string]: Dog;
}
```
>注：徦设TypeScript中没有这个限制，那么就会有下面演示的问题

```typescript
let notOkey:NotOkay = {}
notOkey[10]=new Animal()
//TypeScript 认为notOkey["10"]为Dog，那么就会有潜在的问题
notOkey["10"]

```
尽管串索引是一个强有力的描述`字典模式`的方式，但它也强制约束了所有属性的类型。这是因为`obj.prop`和`obj["prop"]`是等价的。下面的例子中`name`和串索引的类型不一致，类型检察器将会给出一个错误。

```typescript
interface NumberDictionary {
    [index: string]: number;
    length: number;    // ok, length is a number
    name: string;      // error, the type of 'name' is not a subtype of the indexer
}
```

最后，你可以让索引是只读的:
```typescript
interface ReadonlyStringArray {
    readonly [index: number]: string;
}
let myArray: ReadonlyStringArray = ["Alice", "Bob"];
myArray[2] = "Mallory"; // error!
```

# 类类型

## 实现一个接口
在像C#和Java之类的语言中，接口的一个典型的用法是用来强制类要实现一些方法，TypeScript也可这样用。

```typescript
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```
你也可以在接口中指定成员方法，在类中实现这些方法。
```typescript
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```
接口用来描述类的公开部分。

>This prohibits you from using them to check that a class also has particular types for the private side of the class instance.(每个单词都认识，就是不知道它在说什么)

## 静态侧类型和实例侧类型的不同之处

<!--
When working with classes and interfaces, it helps to keep in mind that a class has two types: the type of the static side and the type of the instance side. You may notice that if you create an interface with a construct signature and try to create a class that implements this interface you get an error:
-->
当使用接口和类的时候，需注意的是一个类有两个类型：静态侧类型(type of static side)和实例侧类型（type of instance side）。如果你创建了一个拥有构造函数的签名的接口，并试图用一个类来实现这个接口，那你会得到一个错误：

```typescript
interface ClockConstructor {
    new (hour: number, minute: number);
}

class Clock implements ClockConstructor {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

<!--
This is because when a class implements an interface, only the instance side of the class is checked. Since the constructor sits in the static side, it is not included in this check.


Instead, you would need to work with the static side of the class directly. In this example, we define two interfaces, ClockConstructor for the constructor and ClockInterface for the instance methods. Then for convenience we define a constructor function createClock that creates instances of the type that is passed to it.
-->

这是因为当一个类实现一个接口的时候，只有类的实例侧会被检察。而构造函数属于静态侧，而不会被检察。
相反，你应该直接使用静态侧类型。在下面这个例子中我们定义了两个接口，用于构造函数的`ClockConstructor`和用于实例的`ClockInterface`.然后我们创建了`createClock`来创建传递给它的的类型的实例。

```typescript
interface ClockConstructor {
    new (hour: number, minute: number): ClockInterface;
}
interface ClockInterface {
    tick();
}

function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
    return new ctor(hour, minute);
}

class DigitalClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("beep beep");
    }
}
class AnalogClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("tick tock");
    }
}

let digital = createClock(DigitalClock, 12, 17);
let analog = createClock(AnalogClock, 7, 32);
```
因为`createClock`的第一个参数是`ClockConstructor`类型，而`AnalogClock`的构造函数的类型和这个接口是兼容的，所以`createClock(AnalogClock,7,32)`是可以的。



## 扩展接口
和类一样，接口也可以扩展其它的接口。这可以让你把一个接口的属性复制到另外一个接口中。这样就可以把接口拆分成可复用的组件。

```typescript
interface Shape {
    color: string;
}

interface Square extends Shape {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
```
一个接口可以扩展多个接口：
```typescript
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

# 混合类型
就像我们前面提到的那样，接口可以描述JavaScript中的丰富的类型。由于JavaScript的动态性和灵活性，你也许会遇到一个对象，它是好几种类型混合的结果。
例如，一个东西既是一个有属性的对象又是一个函数。
```typescript
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```
当你使用第三方JavaScript库的时候，你也许需要这一特性来完全描述对象的类型。

# 接口扩展类
当一个接口扩展只一个类，那么它继承了类的所有成员，但不包含这些成员的实现。表现的就像接口声明了这所有的接口，没有实现它们。接口甚至可以继承一到类的私有的或受保护的成员。这表明当你创建了一个继承了私有或受保护的成员，这个接口就只能被该类的子类实现。(注:原文说的是该接口只能被该类或其子类实现)
```typescript
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control implements SelectableControl {
    select() { }
}

class TextBox extends Control {

}

// Error: Property 'state' is missing in type 'Image'.
class Image implements SelectableControl {
    select() { }
}

class Location {

}
```
<!--
In the above example, SelectableControl contains all of the members of Control, including the private state property. Since state is a private member it is only possible for descendants of Control to implement SelectableControl. This is because only descendants of Control will have a state private member that originates in the same declaration, which is a requirement for private members to be compatible.

Within the Control class it is possible to access the state private member through an instance of SelectableControl. Effectively, a SelectableControl acts like a Control that is known to have a select method. The Button and TextBox classes are subtypes of SelectableControl (because they both inherit from Control and have a select method), but the Image and Location classes are not.
-->
在上面的例子中，`SelectableControl`包含了`Control`的所有成员，包私有的`state`成员。`state`是私有变量，只能在`Control`的子类中实现`SelectableControl`，这是因为`Control`的子类才有这些同处声明的私有成员，这是私有成员类型兼容的必要条件。




