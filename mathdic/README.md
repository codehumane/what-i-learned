# 책소개

- [수학사전](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&linkClass=2918%20%20%20%20&barcode=9788955883404)
- 와쿠이 요시유키 지음
- 법칙, 공식, 원리를 쉽게 요약하여 나열함. 이런 저런 공부하다가 수학 개념이 나오면, 참고하기 좋음. 


# 복소수

- 켤레 복소수, 복소수의 절대값, 복소평면(가우스 평면)의 개념.
- 복소수의 극형식(극좌표 표시): `z = a + bi = r(cosθ + i·sinθ)`
- 편각: `θ = arg(z)`
- 극형식과 [삼각함수의 덧셈정리](https://ko.wikipedia.org/wiki/%EC%82%BC%EA%B0%81%ED%95%A8%EC%88%98%EC%9D%98_%EB%8D%A7%EC%85%88%EC%A0%95%EB%A6%AC)를 활용한 복소수의 곱과 나눗셈이 가능함.
  - 예컨대, 두 복소수 z1, z2가 있을 때, z1와 z2의 곱은 "z1의 절댓값을 |z2|배 하고, 원점 중심으로 arg(z2)만큼 회전시킨 결과"임.
  - 또한, i의 극형식은 `cos(\frac{\pi}{2}) + i \cdot sin(\frac{\pi}{2})`이므로, '복소수 z에 i를 곱한다'는 것은 90˚ 회전시킨다는 것과 같음.
- [드무아브르의 공식](https://ko.wikipedia.org/wiki/%EB%93%9C%EB%AC%B4%EC%95%84%EB%B8%8C%EB%A5%B4%EC%9D%98_%EA%B3%B5%EC%8B%9D)은 수학적 귀납법을 이용.
  - `cos2θ = cos^2θ - sin^2θ`와 `sin2θ = 2sinθcosθ`  성질을 이용하면 다음이 성립.

```
(cosθ + i \cdot sinθ)^2 \\
= cos^2θ - sin^2θ + 2i \cdot sinθ \cdot cosθ \\
= cos2θ + i \cdot sin2θ \\
```

