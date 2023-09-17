
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
