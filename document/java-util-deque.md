# Deque & ArrayDeque

Deque와 ArrayDeque 내용을 오라클 문서 기준으로 간단히 기록.

## Deque

먼저 Deque 설명. [여기](https://docs.oracle.com/javase/7/docs/api/java/util/Deque.html)의 설명을 요약 기록함.

- `ArrayDeque`는 `Deque`의 구현체.
- `Deque`는 `Queue`의 확장인터페이스.
- java doc의 `Deque` 설명 첫 번째 문장은 아래와 같음.

> A linear collection that supports element insertion and removal at both ends.

- 여기서 기억할 만한 것은 both ends.
- 마침 "de"는 "double ended"를 가리키는 말.
- 그러니 Deque는 "double ended queue"를 가리키는 것.
- 따라서, 양쪽 끝으로 insert, remove, examin 연산을 제공.
- 그리고 이 연산들은 두 가지 형태로 제공됨.
- 예외를 던지거나 boolean 등의 특수 값을 반환하거나.
- 정리하면 아래와 같음.

| 구분 | First Element (Head) | First Element (Head) | Last Eelement (Tail) | Last Eelement (Tail)
| - | - | - | - | -
| 형태 | exception | special value | exception | special value
| insert | addFirst(e) | offerFirst(e) | addLast(e) | offerLast(e)
| remove | removeFirst(e) | pollFirst() | removeLast() | pollLast()
| examine | getFirst() | peekFirst() | getLast() | peekLast()

- special value 형태는 보통 capacity-restricted를 위한 것이긴 하나,
- Deque의 일반적인 구현체들은 자신이 가진 엘리먼트 갯수의 제한이 없음.
- ArrayDeque가 떠오름. 구현체 열어보면 바로 확인 됨.
- 이 녀석은 List를 사용하고, 용량이 부족할 때 2배로 길이 증가됨.
- Deque는 Queue 인터페이스의 확장이라 했다.
- Queue의 FIFO에 대응하는 Deque 연산들은 아래와 같음.

| Queue Method | Equivalent Deque Method
| - | -
| add(e) | addLast(e)
| offer(e) | offerLast(e)
| remove() | removeFirst()
| poll() | pollFirst()
| element() | getFirst()
| peek() | peekFirst()

- Deque는 LIFO 스택에도 사용.
- 참고로, 레거시 Stack 클래스보다 Deque 사용을 권장.

| Stack Method | Equivalent Deque Method
| - | -
| push(e) | addFirst(e)
| pop() | removeFirst()
| peek() | peekFirst()

## ArrayDeque

다음으로 `ArrayDeque` 설명.

> Resizable-array implementation of the Deque interface.

- 용량 제한 없음. 필요하면 늘어남.
- 일단, `ArrayDeque` 생성자는 아래와 같음.
- 그리고 `addLast`, `addFirst`를 함께 보면 내부 동작 원리가 보임.

```java
public ArrayDeque() {
    elements = new Object[16];
}

... (중략)

public void addFirst(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[head = (head - 1) & (elements.length - 1)] = e;
    if (head == tail)
        doubleCapacity();
}

public void addLast(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[tail] = e;
    if ( (tail = (tail + 1) & (elements.length - 1)) == head)
        doubleCapacity();
}
```

- [여기](https://www.baeldung.com/java-array-deque)에 ArrayDeque 구현체의 그림 설명이 잘 되어 있음.
- 하나만 가져와 보면 아래와 같음.

![array deque head & tail](https://www.baeldung.com/wp-content/uploads/2017/11/ArrayDeque.jpg)

- 그 외에도 thread-safe 하지 않으며,
- null 엘리먼트 사용 불가.
- 스택으로 사용될 땐 `Stack` 보다 빠르고,
- 큐로 사용될 땐 `LinkedList` 보다 빠름.
- 당연히(?) 대부분의 연산이 amortized constant time.
