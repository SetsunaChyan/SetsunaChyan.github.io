---
title: "FP in Scala"
categories: [FP,Scala]
toc: true
classes: []
excerpt: "通过 Scala 初识函数式的世界"
---



JVM 最初是为 JAVA 设计的，并不原生支持函数式编程。一种使其支持函数式的方式就是把所有的函数都视作一个对象。 

Scala 中一个函数可以写成

```scala
val Add: (Int, Int) => Int = new Function2[Int, Int, Int] {
    override def apply(a: Int, b: Int): Int = a + b
}
```

Scala 支持最多 22 个参数，也就是一直到 Function22。

当然可以通过语法糖写成如下等价形式

```scala
val Add: (Int, Int) => Int = (a, b) => a + b
// 等价的
val Add = (a:Int , b:Int) => a + b
```



#### 高阶函数

高阶函数指把函数作为参数或者返回值的一类函数。

如果固定了一个函数的某些参数，你将得到一个接受余下参数的新函数，这就是函数的柯里化。

形式的来说，$(A,B)\to C \iff A \to B \to C$ 。

```scala
val Add: Int => Int => Int = a => b => a + b
// (a: Int) => (b: Int) => a + b
val Add3 = Add(3)
println(Add3(5))
println(Add(3)(5)) // curried function
```

当然可以进一步使用语法糖，用下划线代替参数

```scala
val niceAdd: Int => Int => Int = a => a + _
val niceAdd3 = niceAdd(3)
println(niceAdd3(5))
```

一个更复杂的例子，函数 `nTimes` 可以执行 $n$ 次函数 $f$ 

```scala
val nTimes: (Int => Int, Int) => (Int => Int) = (f, n) => {
    if (n <= 0) (x: Int) => x
    else (x: Int) => nTimes(f, n - 1)(f(x))
}
val Add1 = Add(1)
val Add10 = nTimes(Add1, 10)
println(Add10(5))
```



##### compose

$\text{compose}(f,g)=f(g(x))$

```scala
def compose[T1, T2, T3](f1: T1 => T2, f2: T2 => T3): T1 => T3 =
    (x: T1) => f2(f1(x))
```

##### pipe

$\text{pipe}(f,g)=g(f(x))$

```scala
def pipe[T1, T2, T3](f1: T2 => T3, f2: T1 => T2): T1 => T3 =
    (x: T1) => f1(f2(x))
```

##### toCurry

把函数柯里化。

```scala
def toCurry(f: (Int, Int) => Int): Int => Int => Int =
	(x: Int) => (y: Int) => f(x, y)
def Add(x: Int, y: Int): Int = x + y
println(toCurry(Add)(3)(5)) // 8
```

##### fromCurry

把柯里化的函数复原。

```scala
def fromCurry(f: Int => Int => Int): (Int, Int) => Int =
	(x: Int, y: Int) => f(x)(y)
def AddCurry(x: Int): Int => Int = (y: Int) => x + y
println(fromCurry(AddCurry)(3,5)) // 8
```

##### map

```scala
def map[A, B](l: List[A], f: A => B): List[B] =
    if(l.isEmpty) Nil
    else f(l.head) :: map(l.tail, f)
```

##### filter

```scala
def filter[A](l: List[A], f: A => Boolean): List[A] =
    if (l.isEmpty) Nil
    else if (f(l.head)) l.head :: filter(l.tail, f)
    else filter(l.tail, f)
```

##### flatMap

```scala
def flatMap[A, B](l: List[A], f: A => List[B]): List[B] =
    if (l.isEmpty) Nil
    else f(l.head) ::: flatMap(l.tail, f)
```



如果容器实现了 `map`,`flatMap`,`filter` 三个方法，就可以用 forComprehensions 的写法

```scala
val numbers = List(1, 2, 3, 4)
val chars = List('a', 'b', 'c', 'd')
val colors = List("black", "white")
val forComprehensions = for {
    n <- numbers if n % 2 == 0
    c <- chars if c >= 'c'
    co <- colors
} yield "" + c + n + "-" + co
println(forComprehensions)
```



##### reduce

很重要的一个高阶函数，通过 reduce 可以实现一系列操作，包括上述的 `map` 。

这里的 `max` 不支持空列表，这里的 `composeAll` 只能处理相同类型之间的函数，无法真正做到 compose，这是因为默认的 List 内部元素类型必须一致，所以仍然有改进的空间。

```scala
def reduce[T, A](f: (T, A) => A): A => List[T] => A =
    (init: A) => (l: List[T]) => {
        if (l.isEmpty) init
        else f(l.head, reduce(f)(init)(l.tail))
    }
def sum = reduce((x: Int, o: Int) => x + o)(0)
def product = reduce((x: Int, o: Int) => x * o)(1)
def count[T] = reduce((x: T, o: Int) => o + 1)(0)
def forAll[T](f: T => Boolean) =
    reduce((x: T, o: Boolean) => f(x) && o)(true)
def max[T <: Comparable[T]] =
    reduce((x: Int, o: Int) => if (x >= o) x else o)
def filter[T](f: T => Boolean) =
    reduce((x: T, o: List[T]) => if (f(x)) x :: o else o)(Nil)
def map[T, A](f: T => A) =
    reduce((x: T, o: List[A]) => f(x) :: o)(Nil)
def composeAll[T] = reduce((x: T => T, o: T) => x(o))

val testList = List.tabulate(10)((x: Int) => x + 1)
val Add5 = (x: Int) => x + 5
val Times6 = (x: Int) => x * 6
val funList = List(Add5, Times6)
println(sum(testList)) // 55
println(product(testList)) // 3628800
println(count(testList)) // 10
println(forAll((x: Int) => x >= 2)(testList)) // false
println(max(testList.head)(testList)) // 10
println(filter((x: Int) => x >= 5)(testList)) // List(5, 6, 7, 8, 9, 10)
println(map((x: Int) => x * x)(testList)) // List(1, 4, 9, 16, 25, 36, 49, 64, 81, 100)
println(composeAll(3)(funList)) // 23
```

