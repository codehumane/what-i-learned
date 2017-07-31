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