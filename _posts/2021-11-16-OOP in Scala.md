---
title: "OOP in Scala"
categories: [OOP,Scala]
toc: true
classes: []
excerpt: "简单的记录了 Scala 中面向对象的特性"
---



尽管函数式风格不需要 OOP，但还是必须了解一些 Scala 的 OOP 特性。



#### 继承

很多概念上和主流 OOP 语言都一样，Scala 只能单继承，可以重写成员变量与方法。

```scala
class Animal(v: Int) {
    val creatureType = "wild"
    def eat(): Unit = println("eat")
}
class Dog(name: String, v: Int, 
          override val creatureType: String = "domestic") extends Animal(v) {
    override def eat(): Unit = println(s"$name eat")
}
val b: Animal = new Dog("a", 3)
b.eat()
```

关键字 `final` 和 Java 一样，可以作用在 Class 或者成员变量/函数上静止继承/覆写。关键字 `sealed` 可以只允许在该文件内继承。



#### Object and Class

Object 与 Class 的区别有什么区别？Object 可以看做一个单例，而 Class 不能拥有 Class-Level 的属性，也就是说 Class 没有 Static 成员变量/方法这一概念。可以使用这个概念来实现工厂模式。

当然 Object 和 Class 可以同时存在，叫做 Companion，可以互相访问彼此的私有成员。

```scala
object Person {
    val x = 5
    def apply(name: String) = new Person(name)
}
class Person(val name: String) {
    def gao() = println(name + "-" + Person.x)
}
val a = new Person("abc")
a.gao()
```

所以从这点上来看，Scala 比 Java 更加的 OO。



#### Case Class

给 class/object 添加 `case` 关键字可以使其具有以下功能：

1. class parameters are fields

2. sensible toString()

3. equals() and hashCode() implemented OOTB

4. CCs have handy copy method

5. CCs have companion objects

6. CCs are serializable

7. CCs have extractor patterns = CCs can be used in PATTERN MATCHING
   

```scala
case class CasePerson(name: String, age: Int)
// 1.
val jim = new CasePerson("Jim", 22)
println(jim.name) // Jim
// 2.
println(jim) // CasePerson(Jim,22)
// 3.
val jim2 = new CasePerson("Jim", 22)
println(jim == jim2) // true
// 4.
val bob = jim.copy("Bob")
println(bob) // CasePerson(Bob,22)
// 5.
val thePerson = CasePerson
val mary = CasePerson("Mary",22)
```



#### Abstract class and traits

trait 可以理解为接口，实现接口使用关键字 `with` 。

```scala
abstract class Animal {
    val creatureType: String
    def eat(): Unit
}

class Dog extends Animal {
    override val creatureType: String = "Canine"
    override def eat(): Unit = println("crunch crunch")
}

trait Carnivore {
    def eat(animal: Animal): Unit
}

class Crocodile extends Animal with Carnivore {
    override val creatureType: String = "Crocodile"
    override def eat(): Unit = println("nomnomnom")
    override def eat(animal: Animal): Unit = println(s"eating ${animal.creatureType}")
}

val dog = new Dog
val croc = new Crocodile
croc.eat(dog)
```

Traits vs Abstract Classes:

1. Traits do not hae constructor parameters
2. Multiple traits ma be inherited by the same class
3. traits = behavior, abstract class = "thing"

