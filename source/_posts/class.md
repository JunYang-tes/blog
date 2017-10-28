---
title: 4.类
date: 2017-10-27 
tags: typescript, ts
categories:
- 翻译
- 编程语言
- TypeScript
---

# 简介
传统JavaScript使用函数后基于原型的继承来构建可复用的组件，对于习惯于面向对象的程序员来说这种复式可能有些笨拙。从ECMAScript 2015(也叫做ES6)开始,JavaScript程序员可用基于类的方式来构建面向对象的应用。在TypeScript中，开发者也可以使用这些技术。因为这些东西会被编译成跨平台的JavaScript，所以不需要等待新版的JavaScript被普遍支持。

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
在我们的例子中，我们可以自由的访问我们在类中定义的成员。如果你熟悉其它语言，你也许会注意到我们并没有通过`public`来说明这些成员的可访问性，比如，在C#中，需要通过public来明确的说明其可公开访问。在TypeScript中，每个成员默认是`public`的。
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
然而，当比较的类型有`private`和`protected`的成员的时候，我们却有不同的比较方式。那么怎样的两个类型会被认为是兼容的呢?如果这两个变量中有一个变量有私有成员，那么另一个变量的私有成员必须和这个变量的私有成员在同一个地方定义(注:继承自同一个类)，那么这两个变量才有可能是兼容的。对于`protected`也是这样的。
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
在这个例子中，我们有一个`Animal`类和其子类`Rhino`类，以及一个看起来很向`Animal`的`Employee`类(注:回想鸭类型)。我们创建这些类的实例并试着用它们相互赋值，看会发生什么。因为`Animal`和`Rhino`的private成员有相同的"出处"，所以它们是兼容的。然而`Employee`却不是这么回事了。当我们式着将`Employee`类型的变量赋值给`Animal`类型的变量的时候，我们会得到一个类型不兼容的错误提示。尽管`Employee`也有一个叫`name`的私有变量，但该变量却不是在Animal中定义的那个。

## 理解protected
`protected`和`private`是类似的，不过呢，protected修饰的成员在起子类中也是可以访问的。
```typescript
class Person {
    protected name: string;
    constructor(name: string) { this.name = name; }
}

class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name);
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
console.log(howard.getElevatorPitch());
console.log(howard.name); // error
```
我们不能在`Person`类外面访问`name`属性，但我们可以在`Employee`类的方法中访问它，因为Employee继承于`Person`.
构造器也可用protected修饰，这表明这个类不能从外部实例化，但是可被继承。例如:
```typescript
class Person {
    protected name: string;
    protected constructor(theName: string) { this.name = theName; }
}

// Employee can extend Person
class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name);
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
let john = new Person("John"); // Error: The 'Person' constructor is protected
```

## Readonly 修饰符
你也可以使用`readonly`关键字来表示一个属性是只读的。只读属性只能在其声明的地方或构造器中被初始化。
```typescrpt
class Octopus {
    readonly name: string;
    readonly numberOfLegs: number = 8;
    constructor (theName: string) {
        this.name = theName;
    }
}
let dad = new Octopus("Man with the 8 strong legs");
dad.name = "Man with the 3-piece suit"; // error! name is readonly.
```
## 参数属性
在上一个例子中，在Octopus类中，我们声明了只读属性`name` 并在Octopus类的构造函数中初始化了这个属性。
这其实是一种常见的模式，`参数属性`可以简化这个过程，让你在同一个地方创建并初始化一个成员。
```typescript
class Octopus {
    readonly numberOfLegs: number = 8;
    constructor(readonly name: string) {
    }
}
```
注意我们使用`readonly name:string`来声明了一个参数，这会在类上创建并初始化一个`name`成员。这样我们就把成员的声明和赋值放在了同一个地方。
`属性参数`用一个前缀来声明，这个前缀可以是访问修饰符或者`readonly` 或这同时有这两者。使用`private`来声明参数属性将得到一个私有的属性，同样public,protected 声明的参数属性将得到公开或受保护的属性。

# 访问器
TypeScript支持`getter`和`setter`来拦截对对象成员的访问。这让你可以更好的控制对象成员是如何被访问的。
让我们把一个简单的类转化成使用`get`和`set`的类。让我们从一个没有`getter`和`setter`的类开始
```typescript
class Employee {
fullName:strin
}
let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    console.log(employee.fullName);
}
```
尽管让`fullName`可被直接访问会带来便利，但当人们可以突发奇想的修改类成员也会带来一些麻烦。
在下面的版本中，我们在修改fullName之前做一些检察以确保修改者有正确的修改密码。我们的做法是将对`fullName`的直接访问替换为用一个`set`函数来做。也相对应的添加一个`get`函数来使`fullName`可被获取
```typescript
let passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            console.log("Error: Unauthorized update of employee!");
        }
    }
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    console.log(employee.fullName);
}
```
>关于访问器的说明
首先，使用访问器你需要把编译的输出目标设置为ES5或之后版。降级为ES3是不被支持的。
其次，只有`get`没有`set`的访问器会被推断为`readonly`.在产生`.d.ts`文件时这很有用，因为使用该属性的人可以知道这是一个只读的属性。

# 静态属性
<!--Up to this point-->
到目前为止,我们只讨论了实例的成员(注:后半句多余，没译)。我们也可以创建类的静态成员——既那些在类上可访问的成员。在下面的例子中，我们在`origin`上使用`static`关键字，使其作为所有`网格`的值。所有实例通过`类名.`的方式来访问类成员。和`this.`类似，我们用`Grid.`来访问静态成员。
```typescript
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
Abstract Classes
```
# 抽象类

抽象类常做基类，他们不能直接被实例化。可接口不同的是，抽象类可包括其成员的一些实现。使用`abstract`关键字来定义抽象类和抽象方法。
```typescript
abstract class Animal {
    abstract makeSound(): void;
    move(): void {
        console.log("roaming the earth...");
    }
}
```
抽象类中的抽象方法必须在在类中被实现(注:除非子类也是抽象类)。抽象方法的语法和接口方法的语法是类似的，都是只有方法的签名而没有方法体。不同的是，抽象方法必须用`abstract`来修饰还可以加访问修饰符。
```typescript
abstract class Department {

    constructor(public name: string) {
    }

    printName(): void {
        console.log("Department name: " + this.name);
    }

    abstract printMeeting(): void; // must be implemented in derived classes
}

class AccountingDepartment extends Department {

    constructor() {
        super("Accounting and Auditing"); // constructors in derived classes must call super()
    }

    printMeeting(): void {
        console.log("The Accounting Department meets each Monday at 10am.");
    }

    generateReports(): void {
        console.log("Generating accounting reports...");
    }
}

let department: Department; // ok to create a reference to an abstract type
department = new Department(); // error: cannot create an instance of an abstract class
department = new AccountingDepartment(); // ok to create and assign a non-abstract subclass
department.printName();
department.printMeeting();
department.generateReports(); // error: method doesn't exist on declared abstract type
```
# 高级技术
## 构造函数
当你在TypeScript中声明了一个类，你实际上同时声明了很多东西。首先是类:
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

let greeter: Greeter;
greeter = new Greeter("world");
console.log(greeter.greet());
```
当我们指定`let greeter:Greeter`，我们使用`Greeter`作为`Greeter`类的实例的类型，面向对象语言的程序员对此很熟悉。
我们也创建了一个叫做`构造函数`的东西。当我们new一个类的时候，这个函数就会被调用。为了看看实际上是什么样子，让我们看看上面的代码编译出来的`JavaScript`
```typescript
let Greeter = (function () {
    function Greeter(message) {
        this.greeting = message;
    }
    Greeter.prototype.greet = function () {
        return "Hello, " + this.greeting;
    };
    return Greeter;
})();

let greeter;
greeter = new Greeter("world");
console.log(greeter.greet());
```
构造函数被分配给了`Greeter`变量，都我们new Greeter的时候，就会调用这个构造函数并得到一个实例。构造函数上也包含了类的静态成员。另一个思考类的方式是其可分为`实例侧`和`静态侧`
修改一下前面的例子来演示其中的不同
```typescript
class Greeter {
    static standardGreeting = "Hello, there";
    greeting: string;
    greet() {
        if (this.greeting) {
            return "Hello, " + this.greeting;
        }
        else {
            return Greeter.standardGreeting;
        }
    }
}

let greeter1: Greeter;
greeter1 = new Greeter();
console.log(greeter1.greet());

let greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";

let greeter2: Greeter = new greeterMaker();
console.log(greeter2.greet());
```
在这个例子中，`greeter1`和前面的类似，我们使用`Greeter`类来创建它，并使用使用创建后的对象。
接下来，我们直接使用类。我们创建了一个`greeterMaker`变量，这个变量引用了类本身，或者说它是类的构造函数。这的`typeof Greeter`意思是:给我Greeter类自身的类型而不是它的一个实例。或者，更准确的说:给我那个叫Greeter的符号的类型。这个类型将包含Greeter的所有静态成员以及创建Greeter实例的构造函数。现在我们可以在greeterMaker上使用new关键字来创建Greeter的实例。

## 类用作接口
正如前一节所说，一个类声明创建了两个东西:一个类型和一个构造函数。因为类创建了类型，所以你可以把类用在某些能用接口的地方。
```typescript
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3};
```

