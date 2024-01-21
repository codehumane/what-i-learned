
# 코틀린 코루틴

별 기대 없이 읽었는데, 8장까지 읽어도 계속 재밌어서, 기록도 함께 진행.

# 9장. 취소

- 단순히 스레드를 죽이면 연결을 닫고 자원을 해제할 수 없음.
- 상태가 여전히 액티브인지 자주 확인하는 것도 좋지 않음.
- 코루틴이 제시하는 취소<sup>cancellation</sup>은 간단하고 편리하고 안전.

## 기본적인 취소

- `Job` 인터페이스는 `cancel` 메서드를 가짐.
- 이는 아래의 효과를 가짐.
  - 첫 번째 중단점에서 잡을 끝냄.
  - 자식 잡이 있다면 함께 취소. 부모는 영향 X.
  - 취소된 잡은 새로운 코루틴의 부모로 사용될 수 없음.
  - Cancelling 상태였다가 Cancelled 상태로 바뀜.
- `CancelationException`의 서브 타입을 함께 인자로 넘겨줄 수 있음.
- `cancel` 후 `join`을 호출하는 것이 일반적이며, 취소가 완료될 때까지 중단됨.
- 이것 없으면 경쟁 상태가 발생할 수도.
- `Job#cancelAndJoin`이라는 편의성 확장 함수도 있음.
- `Job()` 팩토리 함수로 생성된 잡도 같은 방법으로 취소할 수 있으며,
- 잡에 딸린 여러 코루틴을 한 번에 취소할 때 자주 사용됨.

## 취소는 어떻게 작동하는가?

- 잡이 취소되면 'Cancelling' 상태로 바뀌고,
- 상태 바뀐 뒤의 첫 번째 중단점에서 `CancellationException` 예외를 던짐.
- `finally` 블럭에서 이 예외를 잡아 자원을 정리하는 것이 좋음.

## 취소 중 코루틴을 한 번 더 호출하기

- 코루틴 취소 후처리 과정에 제한이 있을까?
- 자원을 정리해야 한다면 계속해서 실행될 수 있음.
- 그러나 정리 과정 중 중단을 허용치는 않음.
- 'Cancelling' 상태에서 중단이나 다른 코루틴 시작 불가.
- 시작하려고 하면 무시됨.
- 그런데 만약 DB 롤백 등 중단 함수를 호출해야 한다면?
- 이 때는 `withContext(Noncancellable)`로 포장.

```kt
suspend fun main(): Unit = coroutineScope {
    val job = Job()
    lanch(job) {
        try {
            delay(200)
            println("Coroutine finished")
        } finally {
            println("Finally")
            withContext(NonCancellable) {
                delay(1000L)
                println("Cleanup done")
            }
        }
    }
    delay(100)
    job.cancelAndJoin()
    println("Done")
}
// Finally
// Cleanup done
// Done
```

## invokeOnCompletion

- 자원 해제의 또 하나의 방법.
- `Job#invokeOnCompletion` 호출하기.
- Completed나 Cancelled 같은 마지막 상태 도달 시 호출될 핸들러를 지정하는 것.

```kt
suspend fun main(): Unit = coroutineScope {
    val job = lanch {
        delay(1000)
    }
    job.invokeOnCompletion { exception: Throwable? ->
        println("Finished")
    }
    delay(400)
    job.cancelAndJoin()
}
// Finished
```

## 중단될 수 없는 걸 중단하기

- 중단점이 없어서 취소가 불가한 경우의 대응을 이야기.
- 아래 코드는 `delay`` 대신 `Thread.sleep`을 사용해서 중단점이 없음.

```kt
supend fun main(): Unit = coroutineScope {
    val job = Job()
    lanch(job) {
        repeat(1_000) { i ->
            Thread.sleep(200) // 이 대신 파일 읽기나 무거운 연산이 있어도 마찬가지
            println("Prnting $i")
        }
    }
    delay(1000)
    job.cancelAndJoin()
    println("Cancelled successfully")
    delay(1000)
}
// 중단점이 없기에,
// Printing이 1초까지만 찍히지 않고,
// 3분 넘게 계속 일어남.
```

- 이런 상황에 대한 대처법 중 하나로, 주기적인 `yield()` 호출이 있음.
- 중단점을 강제로 중간 중간 넣어주는 것.
- 다른 방법은, 잡의 상태를 추적하는 것.
- `CoroutineScope#isActive`를 이용.
- 마지막 방법은, 잡이 액티브 상태가 아닐 경우 `CancellationException`을 던지는 `ensureActive()` 함수를 사용하는 것.
- `yield`에 비해서는 `ensureActive()`의 역할이 더 경량이므로 좀 더 선호됨.
- `yield`는 스코프가 필요치 않고, 중단하고 재개하는 일을 하므로, 스레드 풀을 가진 디스패처 사용 시 스레드가 바뀔 수 있는 문제 있음.
