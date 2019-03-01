# Learn Kotlin by Example

https://play.kotlinlang.org/byExample/overview 문서를 한 번 쭉 읽고 인상 깊었던 것을 요약해서 기록

## Introduction

- pair: "Ferrari" to "Katrina"
- infix functions: infix fun Int.times(str: String) = str.repeat(this)
- null safety: ?
- class declaration: class Contact(val id: Int, var email: String)
- [Any](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-any/index.html): The root of the Kotlin class hierarchy

## Control Flow

- When (statement/expression)

```kotlin
fun cases(obj: Any) {
  when (obj) {
    1 -> println("One")
    "Hello" -> println("Greeting")
    is Long -> println("Long")
    !is String -> println("Not a string")
    else -> println("Unknwon")
  }
}
```

> Note that all branch conditions are checked sequentially until one of them is satisfied. So, only the first suitable branch will be executed.

- iterators: 클래스 안에서 iterator operator를 구현해서 iterator 처럼 쓸 수 있음

```kotlin
class Zoo(val animals: List<Animal>) {
  operator fun iterator(): Iterator<Animal> {
    return animals.iterator()
  }
}

fun main() {
  val zoo = Zoo(listOf(Animal("zebra"), Animal("lion")))
  for (animal in zoo) {
    println("It's a ${animal.name}")
  }
}
```

- ranges: for (i in 0..3), for (i in 2..8 step 2), for (i in 3 downTo 0)
- a == b -> a == null ? b == null : a.equals(b) 편함
- 삼항 연산자<sup>ternary operator</sup>가 없는 대신 아래와 같이 함

```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

## Special Classes

- Data Classes
- Sealed Classes: 상속을 같은 파일 내에서만 허용
- object expressions/declaration
- companion object

```kotlin
class BigBen {
  companion object Bonger {
    fun getBongs(nTimes: Int) {
      for (i in 1 .. nTimes) {
        print("BONG ")
      }
    }
  }
}

fun main() {
  BigBen.getBongs(12)
}
```

## Functional

- higher-order function

```kotlin

// Taking Functions as Parameters

fun calculate(x: Int, y: Int, operation: (Int, Int) -> Int): Int {
  return operation(x, y)
}

fun sum(x: Int, y: Int) = x + y

fun main() {
  val sumResult = calculate(4, 5, ::sum)
  val mulResult = calculate(4, 5) { a, b -> a * b }
  println("sumResult $sumResult, mulResult $mulResult")
}

// Returning Functions

fun operation(): (Int) -> Int {
  return ::square
}

fun square(X: Int) = x * x

fun main() {
  val func = operation()
  println(func(2))
}
```

- extension functions and properties

```kotlin
data class Order(val items: Collection<Item>)
fun Order.maxPricedItemName() = this.items.maxBy { it.price }?.name ?: "NO_PRODUCTS"
val Order.commaDelimitedItemNames: String
    get() = items.map { it.name }.joinToString()
```

## Collections

- listOf() vs mutableListOf(): 더 짧은 게 immutable(readonly)이라는 게 인상 깊음
- map = collection of key/value pairs: 그래서 선언도 이렇게 함. mapOf(1 to 100, 2 to 200)
- associateBy: associative 생성
- partition, zip 등이 언어에서 제공되니 편함: A zip B, A.zip(B) { a, b -> "$a$b" }
- getOrElse: list, map에 모두 사용 가능
- stream도 별개로 존재

## Scope Functions

- let/run: scoping and null-checks (executes a code block and return its result)

```kotlin
str?.let {
  println("this is not null - $it")
}
```

- with: 조금 다르면서 유사. non-extension function that can access members: you can omit the instance name

```kotlin
with(configuration) {
  println("$host:$port")
}

// 이 문장과 동치
println("${configuration.host}:${configuration:port}")
```

- apply/also: code block on an abject. handy for initializing objects (활용도가 아직은 의문)

```kotlin
data class Person(var name: String, var age: Int, var about: String) {
  constructor() : this("", 0, "")
}

fun main() {
  val jake = Person()
  val stringDescription = jake.apply {
    name = "Jake"
    age = 30
    about = "Andorid developer"
  }.toString()
  println(stringDescription)
}
```

## Delgation

- by

```kotlin
interface SoundBehavior {
  fun makeSound()
}

class ScreamBehavior(val n:String): SoundBehavior {
  override fun makeSound() = println("${n.toUpperCase()} !!!")
}

class RockAndRollBehavior(val n:String): SoundBehavior {
  override fun makeSound() = println("I'm The King of Rock 'N' Roll: $n")
}

class TomAraya(n:String): SoundBehavior by ScreamBehavior(n)

class ElvisPresley(n:String): SoundBehavior by RockAndRollBehavior(n)

fun main() {
  val tom = TomAraya("Trash Metal")
  tom.makeSound()

  val elvis = ElvisPresley = ElvisPresley("Dancin' to the Jailhouse Rock.")
  elvis.makeSound()
}
```

- delegate properties: 좀 더 특화된 delegate을 보임. 이 또한 행위이기에. 편해 보임. 반면, 좀 장황하기도 하고.

```kotlin
import kotlin.reflect.KProperty

class Example {
  var p: String by Delegate()

  override fun toString() = "Example Class"
}

class Delegate() {
  operator fun getValue(thisRef: Any?, prop: KProperty<*>): String {
    return "
  }
}

fun main() {
  val e = Example()
  println(e.p)
  e.p = "NEW"
}
```

- lazy, observable, map delegate 등도 존재

## Productivity Boosters

- Named Arguments
- String Templates
- Destructuring Declarations

```kotlin
val (x, y, z) = arrayOf(5, 10, 15)

for ((name, age) in map) {
  println("$name is $age years old")
}

val (min, max) = findMinMax(listOf(100, 90, 50))
```

```kotlin
data class User(val usrename: String, val email: String)
fun getUser() = User("Mary", "mary@some.com")
fun main() {
  val user = getUser()
  val (username, email) = user
  println(username == user.component1()) // automatically defined component1, component2
  val (_, emailAddress) = getUser() // 변수가 의미 없을 때
}
```