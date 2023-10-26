---
title: typescript 4.7 显式Variance注解
date: 2022-4
tags: typescript, ts
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

Dog 是Aniaml的子类型，在需要Animal的地方，可以使用Dog。

若A<Dog> 也 是 A<Animal> 的子类型，这叫协变**covariant**。

若A<Animal> 是A<Dog> 的子类型，这叫**逆变contravariant**

若A<Animal> 不是A<Dog> 的子类型，A<Dog> 也不是A<Animal>的子类型，这就叫**不变invariant**

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

declare let a:A<Animal>
declare let b:A<Dog>
/*
b 可以赋值个 a, 因为 b 为 { value: Dog } a 为 { value: Animal },
我们可以把Dog 当做Animal 使用，A是协变的。
*/
a=b;
/*
反过来不行，因为不能把Aniaml 当做 Dog来使用。
*/
b=a;// type error

type B<T> = () => T
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
	dogStuff:any
}

type A<T> = (v:T) =>void
declare let a: A<Animal>
declare let b: A<Dog>
/*
a 是 (v:Animal) => void ，在使用a 时候，它希望我们传给他一个animal 或animal的子类型
b 是 (v:Dog) => void, 如果能将b赋值给a，那么在使用a时就不能传animal的子类型给a了，
a(dog) // OK
a=b; // 假设OK
a(dog) // 类型OK，但实际不安全。
因此不能让b可以赋值给a。A是逆变的。
*/
a = b;// error
b = a; // OK
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
