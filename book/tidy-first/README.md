# Tidy First?

책 다 읽고 보니, Tidy First가 아니라, Tidy First?가 제목임을 알게 됨. 간단히 정리.

# Tidyings

## 1. Guard Clauses

- 보호 절에 관한 일반적 이야기.
- 그런데 잠깐 괄호 열고 아래 규칙을 잠시 언급.
- 하나의 루틴에 하나의 반환이라는 규칙.
- 이는 포트란에서 왔다고 함.
- 포트란에서는 하나의 루틴이 여러 진입점과 종료점을 가질 수 있었고, 이로 인해 디버깅이 매우 어려웠다고 함.
- 반면, 보호 절이 담긴 코드는 전제조건이 명시적이므로 분석이 쉬움.

## 4. New Interface, Old Implementation

- pass-through 인터페이스라고 표현.
- 기존 인터페이스가 어렵거나 복잡하거나 혼란스러운 문제가 있다면,
- 단지 기존의 인터페이스를 호출하는 새로운 인터페이스를 만들고,
- 호출부는 새로운 인터페이스를 호출.
- 기존 인터페이스를 새로운 인터페이스 안으로 인라인 시킬 수도 있음.
- 아래의 경우도 pass-through 방식을 사용하는 느낌을 받음.
- 거꾸로 코딩하기 - 루틴의 마지막 줄부터 시작하고, 필요한 중간 결과들이 마치 모두 있는 것 처럼 작성.
- 테스트 먼저 작성 - 통과해야 할 테스트를 먼저 작성.
- 헬퍼 설계 - 뭔가를 해주는 무언가가 있다면, 나머지는 쉬움.

## 5. Reading Order

- 번역이 또 이상.
- 독자가 읽기 원하는 순서로 코드를 재배치하라.
- Reorder the code in the file in the order in which a reader would prefer to encounter it.

## 10. Explicit Parameters

- 명시적이지 않은 파라미터를 넘기면 문제라고 하는데,
- 정확히 어떤 것을 명시적이지 않다고 하는지가 명시적이지 않음.
- 한 사례로 파라미터로 맵을 넘기는 걸 언급.
- 맵을 넘기는 건 분명 위험.
- 이렇게 런타임에서야 위험이 드러나는 파라미터를 명시적이지 않다고 하는 걸까?
- 일단은, 아래와 같은 코드는, 어떤 데이터가 필수 값인지 이해하기 어렵게 만들고, 파라미터 값을 몰래 수정할 수도 있는 문제가 있음.

```
params = { a: 1, b: 2 }
foo(params)

function foo(params)
    ...params.a... ...params.b...
```

- 그리고 2개의 루틴으로 나누라고 함.
- 하나는 파라미터를 모으고, 하나는 명시적으로 파라미터를 전달하는 부분.

```
function foo(params)
    foo_body(params.a, params.b)

function foo_body(a,b)
    ...a... ...b...
```

- 아래 문장도 번역이 이상.
- Another case for explicit parameters is when you find the use of environment variables deep in the bowels of the code.
- 명시적 파라미터가 필요한 또 하나의 경우는, 코드의 깊숙한 곳에서 환경 변수를 사용하는 것.

## 12. Extract Helper

- 추출 조건 중 형식지가 아니었던 부분이 나와 기록.
- 목적이 분명하고, **나머지 코드와는 상호작용이 적은** 코드.
- 도우미 추출의 또 하나의 특수 사례는 시간적 결합을 표현하는 경우.

```
// 만약 시간적 결합이 있다면
foo.a()
foo.b()
// 아래와 같이 바꾸기
ab()
    a()
    b()
```
