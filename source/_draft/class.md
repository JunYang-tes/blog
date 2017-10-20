# 简介
传统JavaScript使用函数后基于原型的继承来构建可复用的组件，对于习惯于面向对象的程序员来说这种复式可能有些笨拙[]。从ECMAScript 2015(也叫做ES6)开始,JavaScript程序员可用基于类的方式来构建面向对象的应用。在TypeScript中，开发者也可以使用这些技术。因为这些东西会被编译成跨平台的JavaScript，所以不需要等待新版得JavaScript被普遍支持。

# 类
来看一个简单的基于类的例子
```typescript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world");
```
这种语法和C#或Java比较相似。我们声明了一个`Greeter`类，头有三个成员:一个`greeting`属性，一个构造器，一个`greet`方法。
在类里面，我们通过`this.`的方式来访问类的成员。
最后一行，我们用`new`关键字来创建了一个`Greeter`的实例。这会调用我们前面定义的构造函数，创建一个Greeter的实例并在构造函数中初始化它。

# 继承

在TypeScript中，我们可以使用通用的面向对象的模式。在基于类的编程活动中，最基本的模式便是通过继承来扩展一个已经存在的类来创建新的类。
```typescript
class Animal {
    name: string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 5) {
        console.log("Slithering...");
        super.move(distanceInMeters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 45) {
        console.log("Galloping...");
        super.move(distanceInMeters);
    }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

这段代码里面包含了一些关于继承的特性。在这里，我们使用`extends`关键字来创建一个子类。这里你可以看见`Hores`和`Snake`继承了`Animal`类，并能访问基类中的属性。
包合构造函数的子类必须调用通过`super()`来基类中的构造函数。

这个例子一演示了如何在子类中覆盖父类中的方法。`Snake`和`Horse`中都创建了`move`方法而覆盖了基类中的方法。尽管`tom`变量被声明为了`Animal`，而实际上是`Horse`，但`tom.move`会调用到`Horse`中的方法。
```
Slithering...
Sammy the Python moved 5m.
Galloping...
Tommy the Palomino moved 34m.
```
# public,private 以及protected 修饰器

## public
public 是默认的修饰器。
在我们的例子中，我们可以自由的访问我们在类中定义的成员。如果你熟悉其它语言，你也许会注意到我们并没有通过`public`来说明这些成员的可访问性，比如，在C#中，需要通过public来明确的说明其可公开访问。在TypeScript中，没个成员默认是`public`的。
你也可以明确的指定`public`:
```typescript
class Animal {
    public name: string;
    public constructor(theName: string) { this.name = theName; }
    public move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

## private
理解private.
当一个成员被标示为`private`,将不可以在其包含它的类外面访问了。例如:
```typescript
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

new Animal("Cat").name; // Error: 'name' is private;
```
TypeScript的类型系统是结构化的类型系统。当我们比较两个不同的类型，不管它们从哪来，只要它们的成员是兼容的，我们就说这两种类型是兼容的。
然而，当比较的类型有`private`和`protected`的成员的时候，我们认为它们是不同的类型。那么怎样的两个类型会被认为是兼容的呢?如果这两个变量中有一个变量有私有成员，那么另一个变量的私有成员必须和这个变量的私有成员在同一个地方定义(注:继承自同一个类)，那么这两个变量才有可能是兼容的。对于`protected`也是这样的。
让我们用一个例子来说明这点:
```typescript
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
    constructor() { super("Rhino"); }
}

class Employee {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

let animal = new Animal("Goat");
let rhino = new Rhino();
let employee = new Employee("Bob");

animal = rhino;
animal = employee; // Error: 'Animal' and 'Employee' are not compatible
```
在这个例子中，我们有一个`Animal`类和其子类`Rhino`类，以及一个看起来很向`Animal`的`Employee`类(注:回想鸭类型)。我们创建这些类的实例并试着用它们相互赋值，看会发生什么。因为`Animal`和`
