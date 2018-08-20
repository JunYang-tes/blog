---
title: Functor
date: 2018-08-17
tags: haskell, functor
categories:
- 翻译
- 编程语言
- Haskell
---

# Functors

本章介绍的三个概念均是把通用的模式抽象出来的例子。我们以下面这两个简单的例子开始。
```haskell
inc :: [Int] -> Int
inc [] = []
inc :: (n:ns) = n+1 : inc ns

sqr :: [Int] -> Int
sqr [] = []
sqr (n:ns) = n^2 : sqr ns
```
```javascript
const inc = (arr) => {
  if(arr.length) {
    let [head,...tail] = arr
    return [
      head +1,
      ...inc(tail)
    ]
  } else {
    return arr
  }
}

const sqr = (arr) =>{
  if(arr.length) {
    let [head,...tail] = arr
    return [
      head * head,
      ...sqr(tail)
    ]
  } else {
    return arr
  }
}

```

这两个函数拥有同样的结构：空列表被映射成它自己，对于非空列表先应用一个函数到第一个元素，再递归的对剩余的子列表调用自己。这两个函数唯一不同的只是作用在每个元素上的操作不同。对第一例子而言，这一操作是应用了(+1)这一函数。对第二个例子而言，应用了(^2). 对这一模式进行抽象可得到一个和库函数map相似的函数：
```haskell
map :: (a->b) -> [a] -> [b]
map f [] = []
map f (x:xs) = f x : map f xs
```
```javascript
const map = (f)=>(arr)=>{
  if(arr.length) {
    let [head,...tail] = arr
    return [
      fn(head),
      ...map(tail)
    ]
  }
}
```
使用这一函数，我们可以更简洁地实现上面的例子中的两个函数：
```haskell
inc = map (+1)
sqr = map (^2)
```
```javascript
const inc = map(a=>a+1)
const sqr = map(a=>a*a)
```
一般地，将一个函数应用到一种数据结构中的每个元素上的这种做法，并非只能应用到列表上，可进一步抽象出类型参数。支持这样操作的类型类即为functor. Haskell 中对其的定义如下:

```haskell
class Functor f where
  fmap:: (a->b) -> f a -> f b
```
其中，类型参数f是Functor的实例，其必须实现fmap函数。fmap 的第一个参数是一个a->b的映射，f a 是一种数据结构，其中的元素的类型是a。fmap的调用结果是f b。
# 例

list 类型可直接将fmap定义为map,而使list成为functor.
```haskell
instance Functor [] where
  -- fmap :: (a->b) -> [a] -> [b]
  fmap = map
```

在Javascript 中我们可以这样模拟。
```javascript
const functors = {
  [Array.prototyp]:map
}

const fmap = f => fa => {
  const map = functors[fa.__proto__]
  if(!map) {
    throw new Error("It's not a functor")
  } else {
    return map(f)(fa)
  }
}
```

其中的[]表示一个没有类型参数的list类型，[a] 也能写成[] a,表示一个元素是a类型的list。即a是[] a 的类型参数。上面的代码这没有显示的指定fmap的类型，因为haskell不能在定义实例的时候使用这样的类型信息。但类型信息可以起到很好的文档作用，所以我们把它写在文档里。

Maybe a 是Haskell中用来表示类型a可能会失败的情况，是这样定义的：
```haskell
data Maybe a = Nothing | Just a
```
Maybe 如下这般实现Functor:
```haskell
instance Functor Maybe where
  -- fmap :: (a->b) -> Maybe a -> Maybe b
  fmap _ Nothing = Nothing
  fmap g (Just x) = Just (g x)

fmap (+1) Nothing -- Nothing
fmap (+1) Just 1 -- Just 2
fmap not (Just False) -- Just True

```



```javascript
class Maybe {
  constructor(value) {
    this.value = value
  }
}
const Nothing = new Maybe()
function just(value) {
  return new Maybe(value)
}

const maybeMap = (f)=>maybe=>{
  if(maybe.value!=null) {
    // const v = f(maybe.value)
    return just(
      f(maybe.value)
    )
  } else {
    return Nothing
  }
}

functors[Maybe.prototype] = maybeMap

fmap(a=>a+1)(just(2)) // Just 3
fmap(a=>a+1)(Nothing) // Nothing

```

用户定义的类型也可成为Functor. 例如，假如我们定义了一个二叉树，在其叶子节点上有个类型参数a：
```haskell
data Tree a = Leaf a | Node (Tree a) (Tree a) deriving Show
```
```javascript
class Tree {}
class Node extends Tree {
  constructor(left,right) {
    this.left = left
    this.right = right
  }
}
class Leaf extends Tree {
  constructor(value) {
    this.value = value
  }
}
```

`deriving` 部分让其可以被显示出来。我们可以同过将函数f应用到叶子节点上来实现Functor。
```haskell
instance Functor Tree where
  -- fmap:: (a->b)-> Tree a -> Tree a
  fmap g (Leaf x) = Leaf (g x)
  fmap g (Node l r) = Node (fmap g l) (fmap g r)

fmap length (Leaf "abc") -- Leaf 3
fmap even (Node (Leaf 1) (Leaf 2))
-- Node (Leaf False) (Leaf True)
```
```javascript
const treeMap = f=>tree=>{
  if(tree instanceOf Leaf) {
    return new Leaf(
      f(tree.value)
    )
  } else {
    return new Node(
      tree.left && treeMap(f)(tree.left),
      tree.right && treeMap(f)(tree.right)
    )
  }
}
functors[Tree.prototype] = treeMap
```
Haskell 中的许多functor和这几个例字计较相似，对于f a这种数据结构，通常被称为容器类型，fmap将函数应用到其中的值上。当然，不是所有的functor都适用这样的模式。例如，IO 类型就不是通常意义下的容器类型，因为其值代表了输入输出动作，而我们无法访问其内部结构。但却可以很容易的实现Functor：
```haskell
instance Functor IO where
  -- fmap::(a->b)->IO a->IO b
  fmap g mx = do {
    x<- mx
    return (g x)
  }
```
在这里，fmap将函数f应用到参数mx的结果上，因此而提供了一种处理这些值的手段。
```haskell
fmap show (returnn True)
```
需注意到使用functor的两点主要益处：
1. fmap可用来处理任何实现了functor的数据结构中的数据。同时，我们对于那些本来就想同（抽象功能和签名）的函数使用了相同的函数名字，而不必想出各种名字给它们（fmap，多态）
2. 我们能定义能用于所有functor的通用函数，例如，上面只能用于list的inc函数可以如下般重新定义：
```haskell
inc :: Functor f=> f Int -> f Int
inc = fmap (+1)

inc (Just 1) -- Just 2
inc [1,2,3] -- [2,3,4]
inc (Node (Leaf 1) (Leaf 2)) -- Node (Leaf 2) (Leaf 3)
```

## Functor laws

要成为Functor，除了要实现fmap方法，还要满足下面的两个定律：
```
fmap id = id
fmap (g.h) = fmap g. fmap h
```
第一个等式说明，把id函数map到functor上的结果和把functor应用到id函数上的结果是一样的。但请注意，这两个含有id的函数是有不同的类型的，左边的id函数的类型是a->a,所以fmap id是f a -> f a 类型，因此，右边的id函数的类型是为f a -> f a。

第二个等式说明将fmap应用于函数的组合的结果和将fmap应用于函数再将返回的函数作组合的结果是一样的。为了让这两个组合的类型是兼容的，g 和 h的类型分别应为b->c 和a->b.

和fmap的多态联合起来，functor定律确保fmap的确执行了映射操作。例如，对于list而言，这确保了参数list经fmap后的新list不会多加，少掉元素，或是改变顺序。

例如，如果我们重定义内建的list functor的实现：
```haskell
instance Functor [] where
  fmap g [] = []
  fmap g (x:xs) = fmap g xs ++ [g x]
```
这样定义，类型是正确的，但却不满足funnctor定律：
```haskell
fmap id [1,2] -- [2,1]
id [1,2] -- [1,2]
```


```javascript
const map = f=arr=>{
  return arr.map(f).reverse()
}
map(
  compose(a=>a+1,a=>a+2)
)([1,2,3]) // [6,5,4]
compose(
  map(a=>a+1),
  map(a=>a+2)
)([1,2,3]) // [4,5,6]

```

## Applicatives

Functor 是对将一个函数应用到一种结构中的元素上的这一模式的抽象。现在，假如我们希望将这个点子进一不一般化，允许任意个数参数的函数被作用于元素上。例如，假如我们像这样定义一系列的fmap:
```haskell
fmap0:: a -> fa
fmap1:: (a ->b ) -> f a -> f b
fmap2:: (a -> b -> c) -> f a -> f b -> f c
fmap3:: (a -> b -> c -> d) -> f a -> f b -> f c -> f d
...
```
fmap1 便是上面的fmap。一种做法是定义一系列的Functor0,Functor1,Functor2 等等，然后，便可以这样:
```haskell
fmap2 (+) (Just 1) (Just 2)
--Just 3
```
但如果这样做，我们必须手动定义一系列的类型类，而且这些类型类都很相似。此外，我们并不知道要定义多少个这之类的类型类才够用。

如果注意到，就多参函数（如+）同过fmap作用于一个functor上，会得到一个位于functor中部分应用了的函数
```haskell
fmap (+) Just 2 // Maybe (2+)
```
```javascript
fmap(a=>b=>a+b)(just(2)) // Maybe f
```
那么，若我们定义一个函数，它可将一个Functor中的函数应用于另一个Functor中的值，那上面的问题就解决了：
```haskell
(<*>)::Functor f=> f (a->b) -> f a -> f b
-- 两参时：
Just (+) <*> Just (2) <*> Just (3) -- Just (5)

-- 三参时：
Just (\x y z->x+y+z) <*> Just (1) <*> Just (2) <*> Just 3 -- Just 6
-- 对List
[+] <*> [1] <*> [2]
[\x y z->x+y+z] <*> [1] <*> [2] <*> [3]

```
上面几个表达式的最左边都是在把一个函数放在Functor中，可以抽象一个函数出来：
```haskell
pure::Functor f => a-> f a
```

```haskell
pure :: a -> f a
(<*>):: f (a->b) -> f a -> f b

fmap0 = pure
fmap1 g x = pure g <*> x
fmap2 g x y = pure g <*> x <*> y
...
```
像`pure g <*> x1 <*> x2 <*> ... <*> xn`这用的表答式在haskell中被称为applicative 风格。它和普通的函数应用`f a1 a2 a3 ... an`是类似的。在这两种情况下，g和f都是柯里化的函数。

实现了pure和<*>函数的functor就被称为applicatives functor。haskell这该类型类定义如下：
```haskell
class Functor f=> Applicative f where
  pure :: a -> f a
  (<*>):: f (a->b) -> f a -> f b
```

## 例
Maybe 像这样实现Applicative functor:
```haskell
instance Applicative Maybe where
  pure = Just
  Nothing <*> _ = Nothing
  (Just g) <*> mx = fmap g mx

pure (+1) <*> Just 1 -- Just 2
pure (+) <*> Just 1 <*> Just 2 -- Just 3
pure (+) <*> Nothing <*> Just 2 -- Nothing

```
```javascript
const applicatives={
  /**
   * [prototype]:{
   *    pure:fn
   *    ap: fn 
   * }
   * */
}
//模拟中缀表达式
class AP {
  constructor(f) {
    this.fn = f
  }
  ap(functor) {
    const {ap,pure} = applicatives[functor.__proto__]
    if(!this.pure) {
      this.pure = pure(this.fn)
    }
    this.pure = ap(this.pure)(functor)
    return this;
  }
}
const pure = value=> new AP(value)

applicatives[Maybe.prototype]={
  pure:just
  ap:fn=>maybe=> maybe === Nothing ? Nothing : fmap(fn.value)(maybe)
}
 
pure(a=>a+1).ap(just(1)) // just 2
pure(a=>b=>a+b).ap(just(1)).ap(just(2)) // just 3
pure(a=>b=>a+b).ap(Nothing).ap(just(2)) // nothing

```

这样，applicative风格的Maybe支持一种异常编程风格，用这种风格，我们无需管理调用失败的情况，这会被applicative的机制自动处理掉。

现在来看下list的实现：
```haskell
instance Applicative [] where
  pure x = [x]
  gs <*> xs = [g x | g<-gs,x<-xs] 

pure (+1) <*> [1,2,3] -- [2,3,4]
pure (+) <*> [1] <*> [2] -- [3]
pure (*) <*> [1,2] <*> [3,4] -- ?
```

pure 用来把一个值放入一个列表中，形成一个单元素的列表。<*> 被定义为将列表中的每个函数作用到每个元素上。

如果实现一个函数用来将两个list中的元素两两相乘：
```haskell
prods: [Int] -> [Int] -> [Int]
prods xs ys = [x*y| x<-xs,y<-ys]
```
利用List是Applicative Functor的事实，我们可以这样实现该函数
```haskell
prods xs ys = pure * <xs> * <ys></ys>
```

```javascript
const arrPure=x=>[x]
const arrAp = fns=>arr=>{
  let ret=[]
  for(let f of fns) {
    for(let v of arr) {
      ret.push(f(v))
    }
  }
  return ret
}
applicatives[Array.prototype] = {
  pure:arrPure,
  ap:arrAp
}
pure(a=>a+1).ap([1,2,3]) -- [2,3,4]
pure(a=>b=>a+b).ap([1]).ap([2])
pure(a=>b=>a*b).ap([1,2]).ap([3,4])

const prods=(xs,ys)=>pure(a=>b=>a*b).ap(xs).ap(ys).pure

```
总之，applicative风格的list支持一种非决定性的编程形式。
<!--
the applicative style for list supports a form of non-deterministic programminng in which we can apply pure functionns to multivalued arguments without the need to manage the selection of values or the propagationn of failure, as this is toaken care of by the applicative machinery.
-->

最后一个例子来看IO如何实现Applicative functor
```haskell
instance Applicative IO where
  pure = return
  mg <*> mx = do {
    g<-mg
    x<-mx
    return (g x)
  }
```
在这个例子中，pure为IO类型的return 函数，<*>将一个不纯的函数应用到一个不纯的参数上。例如，一个从键盘读取n个字符的函数可以像这样定义：
```haskell
getChars :: Int -> IO String
getChars 0 = return []
getChars n = pure (:) <*> getChar <*> getChars(n-1)
```
在该例中，对于读取0个字符的情况，直接返回了一个空列表。对于其他情况，我们通过applicative 风格的调用把IO char和IO String连了起来。
更一般的讲，IO类型的applicative风格支持交互式编程方式，我们可以把纯函数作用到不纯的参数上，而无需管理这些IO动作序列，也无需亲自抽取IO动作的值。

## 副作用编程

对于Applicatives 最原始的动机是一般化将函数作用于多参之上这一点子。
上面惯例applicative的例子的共性是都涉及到了`副作用`。在各例中，applicative通过了`<*>`函数，使我们可以以applicative的风格来编写程序，将函数作用于其参数。关键的不同之处在于，参数不再是原始的值，而可以带上额外的信息，如是否失败了，或执行IO动作。因此，applicative functor可被看成是对`将纯函数应用在不纯的参数之上`这一主意的抽象。副作用的准却形式依赖于实际的Functor，如成功与否之于Maybe，I/O 动作之于IO。
除了提供了关于副作用编程的通一方法，使用applicative还有一个重要的好处：我们可以定义一个能够三用于所有Applicative Functor的通用函数。

```haskell
sequanceA :: Applicative f => [f a]=>f [a]
sequanceA [] = []
sequanceA (x:xs) = pure(:) <*> x <*> sequanceA xs
```
该方法将一个functor的列表转变成单个Functor，其中包含了一个值的列表，这是一个常用的applicative模式。
```haskell
getChars :: Int -> IO String
getChars n = sequenceA (replicate n getChar)
```
```javascript
function join(a,b){
  return [...a,...b]
}
function sequanceA(xs) {
  if(xs.length) {
    return []
  } else {
    let [x, ...rest] = xs
    return pure(a=>b=>=[
      a,
      ...b
    ])
    .ap(x)
    .ap(
      sequaneA(rest)
    )
    .pure
  }
}

```

## Applicative laws
```
pure id <*> x = x
pure (g x) = pure g <*> pure x
x <*> pure y = pure(\g -> g y) <*> x
x <*> (y <*> z) = (pure (.) <*> x <*> y) <*> z
```

## Monads

考虑下面的表达式求值的例子。

```haskell
data Expr = Val Int | Div Expr Exp

eval:: Expr -> Int
eval (Val n) = n
eval (Div x y) = eval x `div` eval y

```

```javascript
class Expr {}
class Val extends Expr {
  constructor(val) {
    this.val = val
  }
}
class Div extends Expr {
  constructor(exp1,exp2) {
    this.exp1 = exp1
    this.exp2 = exp2
  }
}

function eval(expr) {
  if(expr instanceOf Val) {
    return expr.val
  } else {
    return eval(expr.exp1) / eval(expr.exp2)
  }
}


```
eval函数没有处理除0的错误
```haskell
eval (Div (Val 1) (Val 0))
```
为了处理除0问题，我们用定义一个safediv,用Maybe来表示除法是否成功。
```haskell
sefediv:: Int -> Int -> Maybe Innt
safediv _ 0 = Nothing
safevid n m = Just (n `div` m)
```
```javascript
const safediv=n=>m=> m === 0 ? Nothing : just(n/m)
```
那么eval函数也要做出一些修改：
```haskell
eval:: Expr -> Maybe Int
eval (Val n) = Just n
eval (Div x y) = case eval x of
                   Nothing -> Nothing
                   Just n  —> case eval y of
                              Nothing -> Nothing
                              Just m  -> safediv n m
eval (Div (Val 1) (Val 0)) --Nothing
```
```javascript
function eval(expr) {
  if(expr instanceof Val) {
    return just (expr.val)
  } else {
    let exp1 = eval(expr.exp1)
    if(exp1 === Nothing) {
      return Nothing
    } else {
      let exp2 = eval(expr.exp2)
      if(exp2 === Nothing) {
        return Nothing
      } else {
        return safediv(exp1.value,exp2.value)
      }
    }
  }
}
```
这样定义的eval解决了除0的问题，但却有一点啰嗦。为了简化定义，我们可以利用Maybe是个applicative functor的特点,希望用applicative style 来解决这个问题：
```haskell
eval :: Expr -> Maybe Int
eval (Val n) = pure n
eval (Div x y) = pure safediv <*> eval x <*> eval y
```
```javascript
function eval(expr) {
  if(expr instanceOf Val) {
    return just(expr.val)
  } else {
    return pure(safediv).ap(
      eval(expr.exp1)
    ).ap(
      eval(expr.exp2)
    ).pure
  }
}
```
然而，这样的定义是类型错误的。因为safediv 的类型是`Int -> Int -> Maybe Int`, 那么`pure safediv <*> eval x <*> eval y` 的类型是 `Expr -> Maybe Int`, 那么就需要safediv的类型是`Int->Int->Int`才行。Haskell会阻止这段代码编译，而在JavaScript中则会得到不正确的计算结果（Maybe Maybe val,Maybe NaN等)。
所以前面applicative的这种模式并不适用于eval，因为applicative 要求我们将纯函数用于functor参数。
那如何才能以简洁一点的方式实现`Expr->Maybe Int`的eval函数呢？关键点再于我们需观察到最初版本中重复出现的模式：
```haskell
(>>=):: Maybe a -> (a->Maybe b) -> Maybe b
mx >>=f = case mx of
          Nothing -> Nothing
          Just x -> f x
eval :: Expr -> Maybe Int
eval (Val n) = Just n
eval (Div x y) = eval x >>= \n ->
                 eval y >>= \m ->
                 safediv n m
```
这里首先求了eval x 并通过 >>= 到到了n，再求eval y并得到了m，最终应用safediv作为整个表达式的值。这一过程也可以写成一行，写成多行有助于提高可读性。

从上例可得一个更一般的形式，使用>>=构建的表达式通常有下列结构：
```haskell
m1 >>= \x1 ->
m2 >>= \x2 ->
...
mn >>= \xn ->
f x1 x2 ... xn
```
即先对m1...mn 做>>=运算，再将f应用到得到的x1...xn上。
Haskell为此提供了一个专用的语法结构：
```haskell
do x1 <- m1
   x2 <- m2
   ...
   xn <- mn
   f x1 x2 ... mx

```
<!--This is the same notatin that is also used for interactive programming-->
这里的每一个 `xi <- mi` 需要对齐，如果不需使用xi，那么`xi<-m1`可缩写为`mi`。使用do结构来写eval如下：
```haskell
eval:: Expr -> Maybe Int
eval (Val n) = Just n
eval (Div x y) = do n <- eval x
                    m <- eval y
                    safediv n m
```
do 结构并非只能用于IO类型，而是可用于任何monad.在Haskell里，monad定义如下：
```haskell
calss Applicative m => Monad m where
  return :: a -> m a
  (>>=) :: m a -> (a -> m b) -> m b
  return = pure
```
即，monad是一个实现了return 和>>=的functor。return的默认定义时pure，说明return是Applicative Functor pure的别名。
Monad的类型类中包含return的定义是由历史原因导致的，是为了向后兼容历史代码。

## 例
```haskell
intance Monad Maybe where
  Nothing >>= _ = Nothing
  (Just x) >>= f = f x

instance Monad [] where
  xs >>= f = [y|x<-xs,y<- f x]

pairs :: [a] -> [b] ->[(a,b)]
pairs xs ys = do x<-xs
                 y<- ys
                return (x,y)
```


```javascript
const maybeBind = mx => f = mx === Nothinng ? Nothing ? f(mx.value)
function eval(expr) {
  if(expr instanceOf Val) {
    return just(expr.val)
  } else {
    return maybeBind(eval(expr.exp1))(
      n=>{
        maybeBind(eval(expr.exp2))(
          m=> safediv(n)(m)
        )
      }
    )
  }
}

const arrBind = arr => f => {
  let ret=[]
  for(let a of arr) {
    ret.push(
      ...f(a)
    )
  }
  return ret
}
const monads = {
  [Array.prototype]:{
    pure:x=>[x],
    bind:arrBind
  },
  [Maybe.prototype]:{
    pure:just,
    bind:maybeBind
  }
}
function bind(m1,f) {
  const b = monads[m1.__proto__]
  return b(m1,f)
}

function combine(m1,m2,f) {
  return bind(m1,v1=>{
    return bind(m2,v2=>f(v1,v2))
  })
}
function doblock(f,...monads) {
  if(monads.length===1) {
    return bind(monads[0],f)
  } else {
    let [first,...rest] = monads
    let tmp = first
    for(let m of rest){
      tmp = combine(tmp,m2,(...args)=>args)
    }
    return bind(tmp,args=>{
      if(f.length===1){
        let [first,...rest] = args
        let ret = f(first)
        for(let arg of rest){
          ret = f(ret)
        }
        return ret
      }else {
        returnn f(...args)
      }
    })
  }
}
function eval(expr) {
  if(expr instanceOf Val){
    return just(expr.val)
  } else {
    return doblock(
      (v1,v2)=>safediv(v1)(v2),
      eval(expr.exp1),
      eval(expr.exp2)
    )
  }
}

```

## state monad


