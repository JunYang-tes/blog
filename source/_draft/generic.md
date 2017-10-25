# 简介
软件工程活动中的一个主要方面是构建组件，这些组件不仅仅是有定义好的API，也要是可复印的。那些对于今天的数据是可用的对于将来的数据也是可用的组件将在构建大型软件系统时给你最大的灵活性。
在C#或Java这样的语言中，一个主要的功能就是用`泛型`来创建可复用的组件。`泛型`表示这些组件可用在许多不同的数据类型上，尔不是单一的数据类型。这就允许用户在这些组件里面使用自己的数据类型。

# Hello World
让我们写一个泛型版的`Hello World`: ID 函数。ID函数指的是一种你给它什么它就返回给你什么的函数，就像`echo`命令一样。
```typescript
function identity(arg: number): number {
    return arg;
}
```
或者，我们可以使用`any`类型:
```typescript
function identity(arg: any): any {
    return arg;
}
```
这里实际上该用泛型，但是用了`any`,这会导致函数返回`any` 类型而丢失掉了类型信息。例如，传入了一个`number`类型，对于返回的类型却只知道它是`any`(注:从而失去了类型检查和进一步的类型推导)。

所以，我们需要一种捕获类型的方法，然后我们可用这种方法了标示返回类型。在这里，我们使用`类型变量`——一种特殊的变量为类型而生而不是为值而生。
```typescript
function identity<T>(arg:T){
    return arg
}
```

现在我们给Id函数加入了类型变量`T`。这个`T`让我们可以捕获用户提供的类型(例如`number`)，所以我们就可以使用这个类型。在这里，我们把`T`用做返回值类型。
<!--
 On inspection, we can now see the same type is used for the argument and the return type. This allows us to traffic that type information in one side of the function and out the other.
 -->
 
 我们说这个版本的id函数是个泛型函数，应为它可以用在很多类型上。和用`any`不同，它和`number`类型的哪个id函数一样是精准的(如，不会丢失类型信息)。
 
 一旦我们写好了泛型函数，我们有两种调用它的方式。第一种是传递所有的参数给它，包括类型参数:
 
```typescript
let output = identity<string>("myOutput")
```
这里，我们明确的指定了T为`string`，其作为调用函数的一个类型参数，需用用尖括号括起来而不是圆括号。
第二种方式更为通用，即使用`类型推断`，即我们让编译器根据我们传入的类型自行设置`T`的值。
```typescript
let output = identity("myString")
```
注意，我们没有用尖括号来明确指明类型参数，编译器根据`"myString"`来决定`T`的值。尽管类型参数的自动推断可以使代码更短，可读性更强，但是有些复杂的情况下编译器不能推断出泛型的类型，这时就需要你明确指定泛型的类型。
# 使用泛型类型变量
当你使用泛型时你会注意到当你创建像`identity`这样的泛型函数的时候，编译器会确保你在函数体内正确的使用了泛型。即，你实际应该把看做是所有类型。
如果我们想在函数体内把`arg`的`length`输出出来，会发生什么呢? 我们可能会这样写:
```typescript
function loggingIdentity<T>(arg:T):T{
    console.log(arg.length);
    return arg;
}
```
如果我们这样做，编译器会报告一个错误，即我们使用了`arg`的`length`成员，但是我们没说过`arg`有这样一个成员。上面说过，你应该把T看成是所有类型，所有可能传进来的`arg`是一个数字，那么它是没有`length`属性的。

这里我们实际上是希望这个函数接受`T`数组而不是T。如果`arg`是`T`的数组，那么就可以访问`.length`属性了。
```typescript
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```
现在你可以把这个函数解读为:一个拥有类型参数`T`以及函数参数`arg`—— 为数组T类型 ——并返回`T`数组类型的泛型函数。如果我们传入一个数字数组，我们会得到一个返回的数字数组，那么`T`为`number`。<!-- This allows us to use our generic type variable `T` as part of the types we're working with,rather than the whole type,giving us greater flexibility -->
这让我们可以把泛型类型`T`作为所有我们用到的类型的一个部分，而不是所有类型。这样给我们更多的灵活性。
对上面的例子我们也可使用下面的形式:
```typescript
function loggingIdentity<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```
你也许很熟悉这种写法，在下节中，我们将会讨论如何创建类似于`Array<T>`这样的泛型。
# 泛形类型
在前面几个小节中我们创建了一个泛型函数,它可又接受一些类型，在这节中我们将探索函数自身的类型以及如何创建泛型接口。
泛型函数的类型和非泛型函数的类型是类似的，需把泛型参数写在最前面。
```typescript
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <T>(arg: T) => T = identity;
```
在类型中，我们也可以使用不同的泛型参数名，只要泛型参数的数量和用的位置是对的就可以。
```typescript
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <U>(arg: U) => U = identity;
```
我们也可用对象字面量的方式来表示泛型函数得类型:
```typescript
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: {<T>(arg: T): T} = identity;
```
我们可以把这个例子写成一个接口。
```typescript
interface GenericIdentityFn {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn = identity;
```
或者，我们可以把泛型参数做为整个接口得泛型参数。这样，该接口的所有成员都可以使用该泛型参数。
```typescript
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```
注意，我们的例子变的有点不同了。
<!--
. Instead of describing a generic function, we now have a non-generic function signature that is a part of a generic type. When we use GenericIdentityFn, we now will also need to specify the corresponding type argument (here: number), effectively locking in what the underlying call signature will use. Understanding when to put the type parameter directly on the call signature and when to put it on the interface itself will be helpful in describing what aspects of a type are generic.

In addition to generic interfaces, we can also create generic classes. Note that it is not possible to create generic enums and namespaces.
-->

# 泛型类
泛型类和浮现接口有类似的结构。泛型类在类名后面有一个泛型参数表。
```typescript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

<!--This is a pretty literal use of the GenericNumber class, but you may have noticed that nothing is restricting it to only use the number type. We could have instead used string or even more complex objects.-->

```typescript
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };

alert(stringNumeric.add(stringNumeric.zeroValue, "test"));
```
就像使用接口一样，在类上使用泛型参数可让类中的的属性或方法可以使用这个泛型类型。

在[类]()这章描述过，类的类型有两种情况，；








oooooooi\




一月又一月乵   ，。8888888888888888888888888