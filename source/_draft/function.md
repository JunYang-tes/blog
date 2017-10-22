# 简介
函数是任何一个JavaScript程序中的基本构建单位。用它们你可以构建抽象层、模拟类、隐藏信息和模块化。在TypeScript中，尽管本来就有类、名字空间和模块这些东西，函数依然伴演了描述`如何做某些事`的关键角色。TypeScript也在标准的JavaScript函数上添加了一些东西来使它们更易于使用。
# 函数
和JavaScript一样，TypeScript可创建命名的函数和匿名的函数。这允许你选择最合适你的应用的方式，是在创建一个API的函数列表还是一个传递给另一个函数的一次性函数(a one-off function to hand off to another function).
下面的例子简要的演示了JavaScript中这两这方式:
```javascript
// Named function
function add(x, y) {
    return x + y;
}

// Anonymous function
let myAdd = function(x, y) { return x + y; };
```
就像在JavaScript中一样，函数可以引用函数体外部的变量，这叫做`变量捕获`。
<!--While understanding how this works, and the trade-offs when using this technique, are outside of the scope of this article, having a firm understanding how this mechanic is an important piece of working with JavaScript and TypeScript.-->
```javascript
let z = 100;

function addToZ(x, y) {
    return x + y + z;
}
```
# 函数类型
## 添加类型
让我们给前面的例子加上函数类型:
```typescript
function add(x: number, y: number): number {
    return x + y;
}

let myAdd = function(x: number, y: number): number { return x + y; };
```
我们能为函数的每个参数添加类型、给函数自身添加类型以及给函数的返回值添加类型。TypeScript可以通过返回语句推断出返回值的类型，所以在大多数情况下，我们可以不用指定返回值的类型。
## 写函数类型
现在我们只定义了函数，让我们写出完整的函数类型:
```typescript
let myAdd: (x: number, y: number) => number =
    function(x: number, y: number): number { return x + y; };
```
一个函数类型有两个部分:参数的类型和返回值的类型。当要写出完整的函数的类型，这两个部分都是必要的。我们像写函数的参数列表那样写出参数的类型，给每个参数一个名字和一个类型。这个名字只为了增加代码的可读性，我们可写成
```typescript
let myAdd: (baseValue: number, increment: number) => number =
    function(x: number, y: number): number { return x + y; };
```
只要参数的类型和函数参数的类型是一致的，不管名字是否一致都是可以的。
第二部分是返回值类型。我们使用了一个胖箭头(=>)来表式其后是返回值的类型。如前面所说，这是函数类型中必不可少的一部分，所以如果一个函数不返回值你需要使用void来表示。
注意，函数类型仅由参数类型和返回值类型构成，而不包括捕获的变量的类型。结果是，捕获的变量作为了函数的隐性状态。


## 推断类型
你也许注意到了，如果你省略了等号一边的类型，TypeScript的编译器可以猜测出类型。
```typescript
// myAdd has the full function type
let myAdd = function(x: number, y: number): number { return  x + y; };

// The parameters 'x' and 'y' have the type number
let myAdd: (baseValue: number, increment: number) => number =
    function(x, y) { return x + y; };
```
这叫作`上下文类型`，类型推断的一种形似。这可以减少和类型相关的工作量。

## 可选和默认参数
TypeScript会假设每个参数都是必须的。在并不意味着不能传递`null`或`undefined`给它，而是在每个函数被调用的时候，编译器会检察用户是否为每个参数提拱了值。编译器也会假设这些参数就刚好是要传递到函数中的参数(注:数量假设)。简而言之，给出的参数的个数要和函数期望的参数的个数相匹配。
```typescript
function buildName(firstName: string, lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // error, too few parameters
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // ah, just right
```
在JavaScript中，每个参数都是可选的，用户可能会省略其中的一些参数，如果参数被省略了那么该参数就是`undefined`。在TypeScript中可以通过在参数名末尾添加`?`来使用这一功能。比如，如果第二个参数是可选的:
```typescript
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");                  // works correctly now
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // ah, just right
```
可选参数必须要在必选参数后面声明。
在TypeScript中，我们也可以参数设置一个预设值，如果调用者没有提拱该参数(或提拱了一个`undefined`)，那么该参数的值就是预设值。这被叫做`默认参数`.
```typescript
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // works correctly now, returns "Bob Smith"
let result2 = buildName("Bob", undefined);       // still works, also returns "Bob Smith"
let result3 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result4 = buildName("Bob", "Adams");         // ah, just right
```
默认参数被视作是可选参数，和可选参数一样，我们可以在调用函数的时候忽略它们。可选参数和默认参数的类型(注:指的是函数的类型)是共用的。
```typescript
function buildName(firstName: string, lastName?: string) {
    // ...
}
```
和
```typescript
function buildName(firstName: string, lastName = "Smith") {
    // ...
}
```
的类型都是`(firstName:string,lastName?:string)=>string`. `lastName`的默认在在类中是没有体现的，只留下该参数是可选的这一事实。
和原生可选参数不同，默认参数不需要声明在必须参数后面。<!--此时类型是什么样的?--> 。如果一个默认参数只必选参数之前，那我们需要明确的传递一个`undefined`给它。
```typescript
function buildName(firstName = "Will", lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // error, too few parameters
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // okay and returns "Bob Adams"
let result4 = buildName(undefined, "Adams");     // okay and returns "Will Adams"
```

## Rest 参数
必选，可选，默认参数有一个共同点:它们谈及的都是一个参数的情况。有时候你可能会想把多个参数"打包起来"，或者你压根就不知道函数实际上会接受到多少个参数。在JavaScript中，你可以使用`arguments`(注:JavaScript的魔术变量)来处理这种情况。
在TypeScript中，你可以把所有的参数收集到同一个变量中:
```typescript
function buildName(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
}

let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```
(注:ES6也有这种语法)
Rest参数被视为一组可选参数，你可以传递任意多的参数都Rest参数。编译器会创建一个参数数组，其名嘴在`...`后指定。
```typescript
function buildName(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
}

let buildNameFun: (fname: string, ...rest: string[]) => string = buildName;
```

# this
<!--a rite of passage,必由之路-->
学习如何使用`this`是学习JavaScript中的一条必经之路。TypeScript是JavaScript的超集，所以TypeScript的程序员也需要学习如何使用`this`以及指出什么时候this被错误的使用了。
幸运的是，TypeScript使用了一些技术来捕获`this`不正确的使用。如果你需要学习在JavaScript中如何使用`this`,请先阅读[《理解JavaScript函数和this》](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/)，该文介绍了`this`内部的东西，我们这里只介绍一些基础。
## this 和箭头函数
在JavaScript中，当函数被调用的时候其中的`this`是可用的。在是一个强大的、灵活的特性，但其代价是总是需要知道函数正在执行的上下文是什么。这是十分具有迷惑性的，特别是在返回一个函数或以一个函数为参数的时候。
例如:
```javascript
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        return function() {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```
注意，`createCardPicker`将返回一个函数。如果我们运行这个例子，我们会得到一个错误，而不是期望中的警告框。这是因为，createCardPicker返回的函数中的`this`是`window`而非`deck`对象。这是因为我们以`顶层非方法的语法`的方式在调用`cardPicker`(原注:在严格模式下，this会是undefined)。
我们可以通过确保function 绑定到正确的`this`的方式修正这个问题。这种情况下，无论这个函数多晚被调用，它总可以访问到deck对象。为了做到这点，我们可使用ES6的箭头函数，箭头函数会在函数被创建的时候捕获`this`。
```javascript
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        // NOTE: the line below is now an arrow function, allowing us to capture 'this' right here
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```
更进一步，如果你使用了`--noImplicitThis`选项，TypeScript会在你错误的使用this的时候给出一个警告。它会指出`this.suits[pickedSuit]`中的`this`是`any`类型的。

## this 参数
不幸的是，`this.suits[pickedSuit]`的类型还是是`any`。这是因为`this`是来自于函数表达式中的对象字面量。你可以通过明确的指定一个`this`参数来修正这个问题。`this`参数是位于参数表中第一个参数的假参数。
```typescript
function f(this: void) {
    // make sure `this` is unusable in this standalone function
}
```
现在添加两个接口来使类型更加清晰和更易复用:
```typescript
interface Card {
    suit: string;
    card: number;
}
interface Deck {
    suits: string[];
    cards: number[];
    createCardPicker(this: Deck): () => Card;
}
let deck: Deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    // NOTE: The function now explicitly specifies that its callee must be of type Deck
    createCardPicker: function(this: Deck) {
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```
现在TypeScript知道`createCardPicker`需要在`Deck`对象上被调用，这表明`this`的`Deck`类型的而不是`any`，所以`--noImplicitThis`不会导致问题。

## 回调中的`this`参数。
当你传递一个函数到一个库中，你也会在回调中遇到`this`的问题。因为这些库会将你的回调作为一个普通函数来调用，`this`就会是`undefined`

## Overloads
