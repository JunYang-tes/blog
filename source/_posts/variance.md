---
title: typescript 4.7 显式Variance注解
date: 2022-4
tags: 
  - typescript
  - ts
categories:
- 编程语言
- TypeScript
---


# typescript 4.7 显式Variance注解

**什么是Variance**

variance 是用来描述泛型类型之间的关系的。

```tsx
interface Animal {
	animalStuff: any
}
interface Dog extends Animal {
	dogStuff:any
}

type A<T> = ...
```

什么是子类型？同常我们讲一个类型是另一个类型的子类型。我们将的是可赋值性。既需要父类型的地方可以安全的使用子类型代替。比如这里，Dog 是Aniaml的子类型，在需要Animal的地方，可以使用Dog。从集合的角度来看，子类型是父类型的子集。

对于泛型类型来说，一个类型是不是另一个类型的子类型，可能就不能简单的是或者否来回答。分以下三种情况，每种情况有一个术语来称呼：

若A<Dog> 也 是 A<Animal> 的子类型，这叫**协变covariant，**Dog ⊆ Animal, A<Dog> ⊆ A<Animal>。

若A<Animal> 是A<Dog> 的子类型，这叫**逆变contravariant，**Dog ⊆ Animal, A<Dog> ⊇ A<Animal>。

若A<Animal> 不是A<Dog> 的子类型，A<Dog> 也不是A<Animal>的子类型，这就叫**不变invariant，**

**协变**

```tsx
interface Animal {
	animalStuff: any
}
interface Dog extends Animal {
	dogStuff:any
}
type A<T> = {
	value: T
}
type B<T> = ()=>T

declare let a:A<Animal>
declare let b:A<Dog>

```

A<T>是协变的。因为A<Dog> 可以赋值给A<Animal>

```tsx
/*
b 可以赋值个 a, 因为 b 为 { value: Dog } a 为 { value: Animal },
我们可以把Dog 当做Animal 使用，A是协变的。
*/
a=b
/*
反过来不行，因为不能把Aniaml 当做 Dog来使用。
*/
b=a;// type error
```

同样B<T>也是协变的：

```tsx
declare let c: B<Animal>
declare let d: B<Dog>
/*
c 是一个返回Animal的函数，因此在使用c的时候我们只会使用Animal的属性，
d 是一个返回Dog的函数，如果把d 赋值给c，则会把d返回的Dog 当Animal来使用
从类型层面来说，这自然是安全的。
*/
c = d;
/*
反过来，如果把Animal 当做Dog来使用，则类型不安全，因为有些属性是Dog 有而Animal没有的。
所以，c不可赋值给d, 类型B是协变的。
*/
d = c;// error

```

typescript 4.7 使用`out` 来显示表示某个泛型是协变的。

```tsx
interface Animal {
	animalStuff: any
}
interface Dog extends Animal {
	dogStuff:any
}
type A<out T> = {
	value: T
}
```

**逆变**

```tsx
interface Animal {
	animalStuff: any
}
interface Dog extends Animal {
	dogStuff:string
}
interface Cat extends Animal {
    catStuff:unknown
}

type A<T> = (v:T) =>void
declare let wantAnimal: A<Animal>
declare let wantDog: A<Dog>
declare let aCat:Cat
```

现在的A<T>的变性是协变、逆变还是不变呢？A<T>是逆变的，因为A<Dog> 不是A<Aniaml>的子类型。那么为什么A<Dog> 不是A<Animal>的子类型呢？下列赋值是不全的：

```tsx
wantAnimal = wantDog
```

因为wantDog 接受一个更小范围的值，而wantAnimal接受一个更大范围的值。例如wantAniml(aCat)是可以的，因为Cat 是Animal的子类型。而wantDog(cat)是不行的，因为Cat 是Cat ，Dog 是Dog,它们都是派生自Animal的。假如wantDog可以赋值给wantAnimal，那么wantAnimal(aCat)再运行时就可能出错（如果wantDog访问了dog独有的属性）。

```tsx
function dog(d:Dog) {
	console.log(d.dogStuff.toUpperCase());
}
wantAnimal = dog as any;
wantAnimal(aCat) // Oops,call toUpperCase on undefined
```

逆变用`in` 关键字来描述。

```tsx
interface Animal {
	animalStuff: any
}
interface Dog extends Animal {
	dogStuff:any
}

type A<in T> = (v:T) =>void
```

**不变**

不变就是把前面两种情况组合起来

```tsx
interface Animal {
	animalStuff: any
}
interface Dog extends Animal {
	dogStuff:any
}

type A<T> = {
	get:()=> T,
	set:(v:T) =>void
}
declare let a:A<Animal>
declare let b:A<Dog>
/*
如个A只有get,那么A是协变的，如果A只有set，那么A是逆变的，可惜A都有，A就是不变的了
*/
a=b;// error, set不兼容
b=a;// error, get不兼容
```

同时使用`in`和`out`来标识不变

```tsx
interface Animal {
	animalStuff: any
}
interface Dog extends Animal {
	dogStuff:any
}

type A<in out T> = {
	get:()=> T,
	set:(v:T) =>void
}
type B<out T,in U> = {
	get:()=> T,
	set:(v:U) =>void
}
declare let a:A<Animal>
declare let b:A<Dog>
/*
如个A只有get,那么A是协变的，如果A只有set，那么A是逆变的，可惜A都有，A就是不变的了
*/
a=b;// error, set不兼容
b=a;// error, get不兼容
```

**为什么要引入这两个关键字**

`in` 表示这个类型参数是用做”输入“的，`out`表示这个类型参数是用做输出的。对于较为复杂的类型，如果显示的标出类型参数的variance, 就不需要使用者思考这个类型参数的variance了。另外，typescript 自己会去计算每个类型参数的variance,对于复杂的类型，这个计算开销是比较大的。
