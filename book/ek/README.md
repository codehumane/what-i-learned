
# 이펙티브 코틀린

# 2장 가독성

## 아이템 11. 가독성을 목표로 설계하라

> 개발자가 코드를 작성하는 데는 1분이 걸리지만, 이를 읽는 데는 10분이 걸린다.

- 프로그래밍은 쓰기보다 읽기가 중요.
- 따라서, 가독성 생각하며 코드 작성해야 함.

### 인식 부하 감소

- '인지 부하'를 줄이는 방향으로 코드 작성.
- '뇌가 프로그램의 작동 방식을 이해하는 과정'을 더 짧게 만들면 됨.
- 예를 들어, 자주 사용되는 패턴을 활용.
- 짧은 코드보다 익숙한 코드가 더 빠르게 읽힘.

```kt
if (person != null && person.isAdult) {
    view.showPerson(pers)
} else {
    view.showError()
}

person?.takeIf { it.isAdult }
    ?.let(view::showPerson)
    ?: view.showError()
```

- 첫 번째 구현은 익숙해서 빠르게 읽힘.
- 두 번째 구현은 사용 빈도 적은 패턴의 조합(elvis, takeIf, let)으로 복잡성 높음.
- 두 번째 구현은 수정에도 불리. `view#show` 다음 `view#hide`를 항상 해야 한다면?
- 실행 결과도 다름. 두 번째는 `showPerson`과 `showError` 모두 호출될 수도.

### 극단적이 되지 않기

- 그럼 `let`을 쓰지 말자는 극단적.
- 이는 좋은 코드를 위해 다양하게 활용되는 인기 있는 관용구.
- 아래와 같이, 스마트 캐스팅도 되고, 사람들이 쉽게 인식.

```kt
class Person(val name: String)
var person: Person? = null

fun printNam() {
    person?.let {
        print(it.name)
    }
}
```

- 또한, 연산을 인자 처리 후로 이동시키거나,
- 데코레이터로 객체 래핑할 때에도 유용.
- 이 때의 비용은 지불할 만.

```kt
students
    .filter { it.result >= 50 }
    .joinToString(separator = "\n") {
        "${it.name}, ${it.result}"
    }
    .let(::print)

var obj = FileInputStream("/file.gz")
    .let(::BufferedInputStream)
    .let(::ZipInpuStream)
    .let(::ObjectInputStream)
    .readObject() as SomeObj
```

### 컨벤션

```kt
val abc = "A" { "B" } and "C"
print(abc) // ABC

operator fun String.invoke(f: ()->String): String = this() + f()
infix fun String.and(s: String) = this + s
```

- 연산자는 의미에 맞게 사용해야(invoke는 이런 형태로 사용 x).
- 마지막 인자가 람다인 컨벤션이 여기에 적용된 건 복잡성 증대.
- and 네이밍이 내부 동작과 다름.
- 문자열 결합은 이미 내장된 기능이며, 굳이 새로 만들 필요 없음.

## 아이템 12. 연산자 오버로드를 할 때는 의미에 맞게 사용하라

- 연산자 오버로딩은 강력하고 매력적.
- 다만, 연산자의 의미에 맞는 오버로딩 해야함.

```kt
print(10 * !6) // 의미 x

operator fun Int.not() = factorial()
fun Int.factorial(): Int = (1..this).product()
fun Iterable<Int>.product(): Int = fold(1) { acc, i -> acc * i }
```

- 오버로딩한 함수 이름은 not.
- 팩토리얼에 사용하면 혼란과 오해 여지 있음.
- 코틀린의 모든 연산자는, 구체적 이름을 가진 함수에 대한 별칭일 뿐임.

### 분명하지 않은 경우

- 연산자 오버로딩 시, 의미가 명확치 않다면, infix 확장 함수 활용.
- 아래는 함수에 대한 * 연산자 오버로딩 예시.

```kt
// 함수를 세 번 반복하는 새로운 함수가 만들어진다고 생각
operator fun Int.times(operation: () -> Unit): ()->Unit = { repeat(this) { operation() } }
val tripleHello = 3 * { print("Hello") }
tripleHello()

// 함수가 세 번 호출된다고 생각
operator fun Int.times(operation: () -> Unit) = { repeat(this) { operation() } }
3 * { print("Hello") }

// 확장 함수로 의미 드러내기
infix fun Int.timesRepeated(operation: ()->Unit) = { repeat(this) { operation() } }
val tripleHello = 3 timesRepeated { print("Hello") }
tripleHello()

// 이미 stdlib에서 구현된 top-level function 활용
repeat(3) { print("Hello") }
```

### 규칙을 무시해도 되는 경우

- DSL에서는 연산자 오버로딩 규칙 무시해도 괜찮음.
- HTML DSL에서의 String.unaryPlus가 그 예.
- [kotlinx.html](https://kotlinlang.org/docs/typesafe-html-dsl.html)인가 봄.

```kt
body {
    div {
        +"Some text"
    }
}
```

## 아이템 13. Unit?을 리턴하지 말라

```kt
fun keyIsCorrect(key: String): Boolean = //...
fun verifyKey(key: String): Unit? = //...

if (!keyIsCorrect(key)) return
verifyKey(key) ?: return
```

- Unit?을 Boolean 대신 써볼 수 있겠으나,
- 이는 읽을 때는 그리 멋져 보이지 않음.
- 오해의 소지가 있고, 예측 어려운 오류를 만들 수도.

```kt
// 혼란 야기 + showData, showError 모두 호출 될 수도
getData()?.let{ view.showData(it) } ?: view.showError()

// vs
if (person != null && person.isAdult) {
    view.showPerson(person)
} else {
    view.showError()
}
```

- 책에는 없지만 Unit, Void, void, Nothing, null 구분 유의.
- primitive와 everything is object 비교도 생각해 볼 거리(generic, nullable, member function, ...).

## 아이템 14. 변수 타입이 명확하지 않은 경우 확실하게 지정하라

- 타입 추론 이야기 반복.
- 개발 시간 줄여주고, 가독성을 높일 수 있음.
- 그러나 유형 명확치 않은 경우 남용 자제.

```kt
val num = 10
val name = "March"
val ids = listOf(12, 112, 554, 997)
val data = getSomeData()
val data: UserData = getSomeData()
```

- 함수 정의를 보며 타임 확인을 한다면 가독성 떨어진다는 신호.
- IntelliJ 아닌 GitHub 환경도 고려해보면 좋음.
- 아이템 3(최대한 플랫폼 타임을 사용하지 말라)과,
- 아이템 4(inferred 타입으로 리턴하지 말라)도 함께 생각해 보기.

## 아이템 15. 리시버를 명시적으로 참조하라

- sender, receiver.
- this는 receiver를 가리키는 도구 중 하나.

### 여러 개의 리시버

스코프 내에 2개 이상의 리시버가 있으면 혼란.

```kt
class Node(val name: String) {
    
    fun makeChild(childName: String) =
        create("$name.$childName").apply {
            print("Created ${name}")
        }
    
    fun create(name: String): Node? = Node(name)

}

val node = Node("parent")
node.makeChild("child") // "Created parent" 출력
```

그래서 명시적으로 리시버 가리키기.

```kt
class Node(val name: String) {
    
    fun makeChild(childName: String) =
        create("$name.$childName").apply {
            print("Created ${this?.name}")
        }
    
    fun create(name: String): Node? = Node(name)
```

아래는 둘 이상의 리시버를 가리키는 예시.

```kt
class Node(val name: String) {
    
    fun makeChild(childName: String) =
        create("$name.$childName").apply {
            print("Created ${this?.name} in ${this@Node.name}")
        }
```

## 아이템 16. 프로퍼티는 동작이 아니라 상태를 나타내야 한다

일단, 프로퍼티의 개념 설명부터.

- 코틀린의 프로퍼티는 자바의 필드와 완전히 다른 개념.
- 둘 다 데이터를 저장한다는 점에서는 같음.
- 그러나 프로퍼티는 필드가 필요 없음.
- 오히려 개념적으로 접근자(함수)를 나타냄.
    - val일 땐 getter.
    - var일 땐 getter/setter.
    - 그래서 인터페이스에도 프로퍼티 정의 가능.
    - 함수이므로 확장 프로퍼티도 만들 수 있음.
    - 프로퍼티 위임도 가능.

```kt
// 자바의 필드
String name = null;

// 언뜻 비슷해 보일 수 있는 코틀린의 프로퍼티
var name: String? = null

// 자바 필드와는 다른 점을 드러내는 코틀린의 프로퍼티
var name: String? = null
    // backing field가 코틀린에서 필드가 쓰이는 유일한 곳 (val에서는 안 만들어짐)
    get() = field?.toUpperCase()
    set(value) {
        if (!value.isNullOrBlank()) {
            field = value
        }
    }

// 인터페이스에 프로퍼티 선언 가능
interface Person {
    val name: String
}

// 게터를 가진다는 프로퍼티이므로 오버라이드 가능
open class Supercomputer {
    open val theAnswer: Long = 42
}
class AppleComputer : Supercomputer() {
    override val theAnswer: Long = 1_800_275_2273
}

// 프로퍼티 위임도 가능
val db: Database by lazy { connectToDb() }

// 확장 프로퍼티
val Context.preferences: SharedPreferences
    get() = PreferenceManager.getDefaultSharedPreferences(this)
```

참고로, derived property에 대해 번역 이상한 것이 있어 원문 기록.

> For a read-write var property, we can make a property by defining a getter and a setter. Such properties are known as derived properties, and they are not uncommon. They are the main reason why all properties in Kotlin are encapsulated by default.

이러한 프로퍼티 특성이 있다고, 함수를 완전히 대체하듯 사용하면 위험.

- 원칙적으로, 프로퍼티는 상태를 나타내거나 설정하기 위한 목적으로 사용.
- 프로퍼티를 함수로 정의한다 했을 때 접두사로 get/set을 고려한다면 괜찮음.
- 연산 비용 크거나 복잡도가 O(1)보다 크다면 함수로.
- 비즈니스 로직 포함한다면 함수.
- 비결정적 특성(같은 동작 연속으로 했을 때 다른 값이 나올 수 있음)이면 함수.
- 변환이 일어난다면 함수.
- getter에서 상태 변경 일어난다면 함수.

```kt
val Tree<Int>.sum: Int
    get() = when (this) {
        is Leaf -> value
        is Node -> left.sum + right.sum
    }

fun Tree<Int>.sum(): Int = when (this) {
    is Leaf -> value
    is Node -> left.sum + right.sum
}
```

반대로, 상태 추출/설정 시엔 함수 대신 프로퍼티 사용.

