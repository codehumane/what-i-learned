# 책소개

- [수학사전](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&linkClass=2918%20%20%20%20&barcode=9788955883404)
- 와쿠이 요시유키 지음
- 법칙, 공식, 원리를 쉽게 요약하여 나열함. 이런 저런 공부하다가 수학 개념이 나오면, 참고하기 좋음. 


# Complex Number

- 켤레 복소수: 복소수 z의 허수 부호 바꿈. `\bar z`로 표기.
- 복소수의 절대값: `\sqrt{a^2 + b^2}`의 값. |z|로 표기.
- 복소평면 (가우스 평면): 실수부를 가로축으로, 허수부를 세로축으로 표기. 실수처럼 수직선으로 표기 불가하므로.
- 극형식 (극좌표 표시)
  - 복소평면 상의 z에 대응하는 점 A가 있고,
  - 절대값이 |z| = OA = r 이며,
  - r과 실수축 양의 부분이 이루는 각을 θ라고 할 때,
  - `z = r(cosθ + i·sinθ)`로 표기.
- 이때의 θ는 편각이라고 부르며, `arg(z)`로 표기함.
- 극형식과 [삼각함수의 덧셈정리](https://ko.wikipedia.org/wiki/%EC%82%BC%EA%B0%81%ED%95%A8%EC%88%98%EC%9D%98_%EB%8D%A7%EC%85%88%EC%A0%95%EB%A6%AC)를 활용한 복소수의 곱과 나눗셈이 가능함.
  - 예컨대, 두 복소수 z1, z2가 있을 때 z1와 z2의 곱은 다음 문장과 동치.
  - "z1의 절댓값을 |z2|배 하고, 원점 중심으로 arg(z2)만큼 회전시킨 결과"
  - 또한, i의 극형식은 `cos(\frac{\pi}{2}) + i \cdot sin(\frac{\pi}{2})`이므로,
  - '복소수 z에 i를 곱한다'는 것은 90˚ 회전시킨다는 것과 같음.
- [드무아브르의 공식](https://ko.wikipedia.org/wiki/%EB%93%9C%EB%AC%B4%EC%95%84%EB%B8%8C%EB%A5%B4%EC%9D%98_%EA%B3%B5%EC%8B%9D)은 수학적 귀납법을 이용.
  - `cos2θ = cos^2θ - sin^2θ`와 `sin2θ = 2sinθcosθ`  성질을 이용하면 다음이 성립.

```
(cosθ + i \cdot sinθ)^2 \\
= cos^2θ - sin^2θ + 2i \cdot sinθ \cdot cosθ \\
= cos2θ + i \cdot sin2θ \\
```
