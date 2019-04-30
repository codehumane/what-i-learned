
# Introduction to Reactive Programming

https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Intro

## Why

짧은 내용 안에 핵심을 잘 담아냈다고 생각함.

> declarative code (in a manner that is similar to functional programming) in order to build asynchronous sequences of events.

- 선언적 코드
- 비동기 프로세싱 파이파라인을 구축
- 이벤트 기반 모델
- 데이터가 이용 가능할 때, 컨슈머에게 푸시되는

이런 특징들이 중요한 이유는, 리소스들을 효율적으로 사용하고, 많은 클라이언트들을 받아주기 위한 가용량을 늘리기 위함. 저수준의 동시성 또는 병렬성 코드를 작성하지 않고도 말이다.

> It also facilitates composition, which in turn makes asynchronous code more readable and maintanable.

- 또한, 구성을 이용해 가독성과 유지보수성을 높임.

## Interactions

- source `Publisher` produces data
- does nothing until a `Subscriber`has registered(subscribed)

![reactive stream sequences/interactions](https://tech.io/servlet/fileservlet?id=26381655039806)

- 여기에 더해 *operators*라는 개념도 있음.
- 데이터의 각 단계에 어떤 처리를 적용할지를 나타냄.
- 오퍼레이터의 적용은 새로운 중간단계의 `Publisher를 반환.
- 또한 체이닝 됨.
- 마지막 형태는 결국 `Subscriber`.
- 자바의 Stream이랑 많이 유사.
