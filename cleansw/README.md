# 책소개

로버트 C. 마틴, [클린 소프트웨어 (애자일 원칙과 패턴, 그리고 실천 방법)](http://jpub.tistory.com/682)

# COMMAND

함수를 클래스보다 강조한다는 점에서 OOP 패러다임과 부딪힘. 하지만 유용함. 게다가 매우 간단함. 아래 그림은 적용 예시. 커맨드 자체에 대한 설명은 생략.

![command](command.png)

무엇이 좋은가?

- `Command`를 사용하는 곳에서 어떤 구현체들이 있는지 알 필요 없음. 단지 `Command`의 `do`를 실행함.
- `명령`의 개념이 캡슐화 된 것. 호출부와 실행부의 논리적 상호 연결이 분리됨.
- 물리적 분리 외에 시간적 분리도 가능함. 명령의 유효성을 검증하고 저장한 후, 필요한 시점에 실행 가능함.
- `Command`에 `undo`를 추가하기도 함. 명령을 취소하는 코드가 명령을 수행하는 코드와 함께 있을 수 있음.

# ACTIVE OBJECT

먼저 책의 구현부를 재작성함.

```java
class ActiveObjectEngineTest {

    @Test
    public void run() throws Exception {
        final ActiveObjectEngine engine = new ActiveObjectEngine();
        engine.add(new SleepCommand(1000, engine, new WakeupCommand()));
        engine.run();
    }
}

class ActiveObjectEngine {

    LinkedList<Command> commands = new LinkedList<>();

    void add(Command command) {
        commands.add(command);
    }

    void run() throws Exception {
        while (!commands.isEmpty()) {
            final Command command = commands.getFirst();
            commands.removeFirst();
            command.execute();
        }
    }
}

class SleepCommand implements Command {

    // 생략

    @Override
    public void execute() throws Exception {
        long currentTime = System.currentTimeMillis();
        if (!started) {
            started = true;
            startTime = currentTime;
            engine.add(this); // 자기 자신을 등록
            System.out.println("start");
        } else if ((currentTime - startTime) < sleepTime) {
            engine.add(this); // 자기 자신을 등록
        } else {
            engine.add(wakeupCommand);
            System.out.println("wakeup");
        }
    }
}

class WakeupCommand implements Command {

    @Override
    public void execute() throws Exception {
        // 실제로는 아무것도 하지 않음. SleepCommand와 다르게 자기 자신을 engine에 등록하지 않을 뿐.
    }
}
```

간단히 내용을 정리함.

- 커맨드 패턴의 응용 중 하나.
- 다중 제어 스레드 구현을 위한 오래된 기법.
- 스레드가 블록되지 않음.
- RTC<sup>run-to-complete</sup>라고 불림. 다음 작업이 시작되기 전에 현재 작업이 완료됨.

# TEMPLATE METHOD & STRATEGY

개념 자체에 대한 설명과 코드는 생략. 둘의 비교만 정리함. [Abstract Methods and Classes](http://docs.oracle.com/javase/tutorial/java/IandI/abstract.html)의 "Abstract Classes Compared to Interface"을 함께 보는 것도 괜찮아 보임.

1. 두 방식 모두 구체적 내용으로부터 일반적 알고리즘을 분리(일종의 특화)함.
2. 템플릿 메소드는 상속을, 스트래터지는 위임을 사용함.
3. 참고로 위임<sup>delegation</sup>, 구성<sup>composition</sup>, 상속<sup>inheritance</sup>의 개념을 서로 구분하여 사용하고 있음.
4. 스트래터지는 약간의 복잡성(클래스 수가 늘어나는 것을 이야기)을 유발함.
5. 더불어, 메모리 및 실행 시간 비용을 수반(유의미한 차이인지는 의문).
5. 한편, 템플릿 메소드는 부분적으로 [DIP](https://en.wikipedia.org/wiki/Dependency_inversion_principle)를 위반함.
6. 아래 코드처럼 `IntSorter`가 버블 정렬이 아닌 다른 정렬을 사용하고 싶을 때 내부를 수정할 수 밖에 없음.
7. (책에 언급되지는 않았지만) 특화의 결정 시점이 템플릿은 컴파일 타임, 스트래터지는 런타임.

```java
abstract class BubbleSorter {
  protected int sort() { // 생략 }
  protected abstract void swap(int index);
}

class IntSorter extends BubbleSorter {
  int sort(int[] ary) { return sort(); }
  protected void swap(int index) { // 생략 }
}
```

