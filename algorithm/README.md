# 책소개

- [알고리즘 : 컴퓨터 과학의 기본, 숫자 알고리즘에서 양자 알고리즘까지](http://www.yes24.com/24/goods/24937708?scode=032&OzSrank=8)
- 산죠이 다스굽타,크리스토스 파파디미트리우,우메쉬 바지라니 공저
- 알고리즘 단순 풀이보다는 알고리즘을 작동시키는 수학적 지식을 전달하고 있음.

# 프롤로그

## 알고리즘 용어의 탄생 배경

- 알콰리즈미<sup>Al Khwarizmi</sup>라는 사람이 십진법을 활용한 여러가지 수학적 방법들을 책에 기록함.
- 이 방법들은 간결했고, 모호하지 않고, 오류가 없었음. 즉, [알고리즘](https://en.wikipedia.org/wiki/Algorithm)임.
- 그리고 십진법이 널리 전파되는데 가장 큰 기여를 함.
- 알고리즘<sup>Algorithm</sup>이란 용어는 이를 기리기 위해 만들어 짐.

## 피보나치에 대하여

- 피보나치<sup>Fibonacci</sup>는 자리법 표기의 유용함(알콰리즈마의 업적)을 널리 알리고자 노력함.
- 피보나치 수열로 더 유명하며, 이 수열은 아래와 같이 정의함.
- F<sub>0</sub> = 1, F<sub>1</sub> = 1 (seed values)
- F<sub>n</sub> = F<sub>n-1</sub> + F<sub>n-2</sub>

### 지수 알고리즘

- F<sub>100</sub>이나 F<sub>200</sub>의 값은 얼마인가?
- 이를 해결하는 한가지 알고리즘은 F<sub>n</sub>의 재귀적 정의를 그대로 수행하는 것.
- 의사코드는 아래와 같음.

```
function fib1(n)
if n = 0: return 0
if n = 1: return 1
return fib1(n-1) + fib2(n-2)
```

- 그리고 이 알고리즘은 수행 시간은 다음과 같음. (책에서는 조금 다른 기준을 사용하는데, 개인적으로 이해하기 쉬운 함수 실행수를 기준으로 바꾸어 기록함)
- T(n) = 1 (n = 0, n = 1)
- T(n) = T(n-1) + T(n-2) + 1 (n > 1)
- 이를 F<sub>n</sub>의 점화식<sup>recurrence relation</sup>과 비교해보면, T(n) ≥ F<sub>n</sub>가 됨. (여기서 F<sub>n</sub>의 점화식이란, n이 2이상인 경우의 함수 결과값을 가리킴)
- F<sub>n</sub>은 거듭제곱<sup>exponentiation</sup>(2^n)만큼 증가함.
- 즉, 수행시간이 지수적으로 증가함.
- 지수적 시간의 저주에 대해서도 소개하고 있음.

*참고로, 알고리즘의 수행시간을 표기하는 방법에 대해서는, [Khan Academy의 점근적 표기법](https://ko.khanacademy.org/computing/computer-science/algorithms/asymptotic-notation/a/asymptotic-notation)을 참고.

### 다항 시간 알고리즘

- `fib1`이 느린 이유는 재귀 호출 트리를 보면 잘 이해할 수 있음. ([여기](https://www.ics.uci.edu/~eppstein/161/960109.html)를 참고)
- 아래 의사코드처럼, 중간 결과들을 저장하면 수행시간이 다항시간으로 줄어듬.

```
function fib2(n)
if n = 0: return 0
create an array f[0 ... n]
f[0] = 0, f[1] = 1
for i = 2 ... n:
  f[i] = f[i - 1]
return f[n]
```

## O 표기법

- 책에서는 Big-O 표기법을 아래와 같이 설명함.

> f(n)과 g(n)을 양의 정수로부터 양의 실수로의 함수라고 하자. 어떤 상수 c에 대해 f(n) ≤ c*g(n)을 만족하는 상수 c가 있으면, f=O(g)(f는 g보다 빨리 증가하지 않는다는 의미)라고 한다.

- Big-θ와 Big-Ω에 대해서도 설명함.

> f = Ω(g)는 g = O(f)를 의미한다.
> f = θ(g)는 f = O(g)이고 f = Ω(g)를 의미한다.

- 마지막으로 Big-O의 일반적 규칙을 4가지를 소개함.
- 상수 배는 생략한다.
- a>b이면 n<sup>b</sup>를 생략하고 n<sup>a</sup>만 남긴다.
- 지수식과 다항식이 있으면 지수식만 남긴다.
- 다항식과 로그식이 있으면 다항식만 남긴다.

# 숫자 알고리즘

## 기본 산술 연산

### 덧셈

아래는 십진수의 기본 속성을 설명함.

> 세 개의 한 자릿수를 더한 합계는 최대 두 자릿수를 넘을 수 없다.

이는 이진수에도 적용하여, 아래 그림과 같은 알고리즘을 생각할 수 있다.

![이진수 덧셈 알고리즘 예제](binary-addition-example.png)

*그림 출처: http://chortle.ccsu.edu/assemblytutorial/Chapter-08/ass08_3.html

수행 시간은 어떠한가?

- 책에서는 c<sub>0</sub> + c<sub>1</sub>n 이라고 표기함.
- 여기서 c<sub>0</sub>과 c<sub>1</sub>은 상수를 가리키며, 따라서 수행시간은 선형적임.
- 참고로, 상수를 단지 c<sub>0</sub>와 c<sub>1</sub>으로 표기한 이유는, 여기서 중요한 값이 아니라는 의도로 보여짐.

수행시간이 더 빠른 알고리즘은 없는가?

- 각 자릿수의 수를 모두 더하려면 최소한 두 수를 읽고, 결과를 써야 하며, n 번의 연산을 수행해야 함.
- 따라서 위 알고리즘이 최적이라고 볼 수 있음.

### 곱셈과 나눗셈

그림으로 알고리즘을 살펴보면 아래와 같다.

![이진수 곱셈 알고리즘 예제](binary-multiply-example.png)

마찬가지로 수행시간은 얼마나 될까?

- 위 그림에서 Y의 자릿수가 n인 경우, 중간값이 n개 생김.
- 중간값들은 자릿수만큼 왼쪽으로 이동했으므로, 최대 2n 자리의 숫자가 됨.
- 이 중간값들을 더하는 데 걸리는 총 시간은 아래와 같음.
- O(n) + O(n) + … + O(n) : 총 n-1번 수행 = O(n^2)
- 다항 시간이긴 하지만, 덧셈보다는 느림.

알콰리즈미가 알아낸 또다른 곰셈법도 있음.

![al-khwarizmi-multiplication](al-khwarizmi-multiplication.png)

*그림 출처: https://sites.google.com/site/markdolanprogramming/cis-3223/algorithms-chapter-1

- x와 y가 있을때, x는 계속 반으로 나누고 나머지를 버리고 y는 계속 2를 곱함.
- x가 1이 될 때까지 계속 반복함.
- x의 수들 중 짝수인 열을 제외하고 y의 숫자들을 모두 더함.
- 의사코드는 아래와 같음. (개인적으로 좀 더 잘 이해하기 위해, 책과 다르게 x, y를 서로 바꿈)

```
function multiply(x, y)
입력: n비트의 두 정수 x와 y, y ≥ 0
출력: 두 수의 곱

if x = 0: return 0
z = multiply(y, ⎣x/2⎦)
if x is even:
  return 2z
else:
  return y + 2z
```

수행시간은?

- 일단 n번의 재귀호출이 발생함.
- 재귀호출마다 2로 나누기(비트  우측 시프트), 홀수 여부 검사(마지막 비트 검사), 2 곱하기(비트 좌측 시프트), 한 번의 덧셈이 발생하므로, 총 O(n)번의 비트 연산이 수행됨.
- 전체 수행 시간은 O(n^2)이며 앞서 소개한 곱셈 알고리즘과 동일함.
- 이보다 더 나은 알고리즘도 있으나, 2장에서 소개될 예정이라고 함. 아마도 [카라추바 알고리즘](https://ko.wikipedia.org/wiki/%EC%B9%B4%EB%9D%BC%EC%B6%94%EB%B0%94_%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98)일듯.

다음은 나눗셈 알고리즘이다. 의사 코드는 아래와 같다.

```
function divide(x, y)
입력: n 비트의 두 정수 x와 y, y ≥ 1
출력: x를 y로 나눌 때 몫과 나머지

if x = 0: return (q, r) = (0, 0)
(q, r) = divide(⎣x/2⎦, y)
q = 2*q, r = 2*r
if x is odd: r = r + 1
if r ≥ y: r = r - y, q = q + 1
return (q, r)
```

- `x = yq + r`이고 `r < y`를 성립하는 몫 `q`와 나머지 `r`을 찾는 과정임.
- 수행시간은 n^2에 비례함.

그런데 개인적으로 아래와 같은 방식이 떠올랐고, 이게 제일 간단하지 않나 싶다.

```
function divide2(x, y)
입력: n 비트의 두 정수 x와 y, y ≥ 1
출력: x를 y로 나눌 때 몫과 나머지

q = 0
while x ≥ y do
  x = x - y;
  q++;
end
return (q, x)
```

이를 간단히 구현해 본 코드는 [여기](https://github.com/codehumane/learn-algorithm-in-java/blob/master/src/main/java/math/Division.java), 테스트 코드는 [여기](https://github.com/codehumane/learn-algorithm-in-java/blob/master/src/test/java/math/DivisionTest.java)를 참고. 이 외에도 여러가지 방법들이 있음을 [여기](https://en.wikipedia.org/wiki/Division_algorithm#Newton.E2.80.93Raphson_division)서 확인 가능함.

## 모듈러 연산

모듈러<sup>modulo</sup> 연산에 대한 설명은 아래와 같다.

- `x modulo N`은 `x`를 `N`으로 나눌 때의 나머지를 가리킴.
- 즉, `x = qN + r` 이고, `0 ≤ r < N` 이면, `x modulo N`은 `r`임.
- 범위를 {0, 1, …, N-1}로 정의해두고 이를 벗어나면 다시 처음부터 순환하는 것과 같음.
- 한계 크기에 제한받지 않고 수를 다루기 위한 방식임.
- 24시간이 지나면 시간이 다시 0이 되는 것을 생각하면 됨.

모듈러를 이용한 새로운 동치 개념도 생각해볼 수 있다.

- `x ≡ y (mod N)` ⇔ `x mod N = y mod N` ⇔  `(x - y)가 N으로 나누어 떨어진다`

이는 아래 그림을 통해 좀 더 쉽게 이해할 수 있다.

![모듈러 동치 집합](modulo-congruence.png)

*출처: https://en.khanacademy.org/computing/computer-science/cryptography/modarithmetic/a/congruence-modulo

동치 개념을 이용해서, 각 정수를 N개의 동치 집합으로 나눠볼 수도 있다.

- 이 동치 집합은 `{i + kN : k ∈ ℤ}`이고, i는 0에서 N-1 사이가 됨.
- 참고로, ∈ 표기는 어떤 원소<sup>element</sup>가 집합에 속한다는 것을 나타냄.
- 또한, ℤ는 정수의 집합이다. 이 외에도 수학에서 자주 사용되는 집합들은 ℕ, ℝ 등의 특정 기호를 부여함.
- 더 많은 집합 기호는 [여기](https://ko.wikipedia.org/wiki/%EC%A7%91%ED%95%A9)를, 동치 관계에 대한 좀더 자세한 설명은 [여기](https://www.khanacademy.org/computing/computer-science/cryptography/modarithmetic/a/equivalence-relations)를 참고.

동치 집합을 이용하여 덧셈과 곱셈을 아래와 같이 정의할 수도 있다.

> 대체 규칙: x ≡ x' (mod N)이고 y ≡ y' (mod N)이면, x + y ≡ x' + y' (mod N)이고 xy ≡ x'y' (mod N)이다.

- 이를 이용하면 아래의 질문에 좀 더 쉽게 대답할 수 있음.
- "한 편당 3시간 분량인 25편의 TVg 프로그램을 모두 보고 나면 몇 시가 되는가?"
- `(25*3) mod 24`는 대체 규칙(`25 ≡ 1 mod 24`)으로 `1*3 = 3 mod 24`과 같고, 결과는 `3`이 됨.
- 이 외에도 "2^345를 31로 나눈 값은 얼마인가?"를 아래 과정과 같이 쉽게 계산할 수 있음.
- 2^345 ≡ (2^5)^69 ≡ 32^69 ≡ 1^69 ≡ 1 (mod 31)

참고로, 모듈러 연산도 정수의 덧셈과 곱셈에서의 결합, 교환, 분배법칙이 성립한다.

- 결합법칙: `x + (y + z) ≡ (x + y) + z (mod N)`
- 교환법칙: `xy ≡ yx (mod N)`
- 분배법칙: `x(y + z) ≡ xy + yz (mod N)`


### 모듈러 지수 연산

`x^y mod N`을 구해야 하는데, `x^y`의 결과가 계산기나 컴퓨터가 다룰 수 없는 크기의 수라면? 한 가지 방법은 대체 규칙을 이용해,  `x mod N`을 지수 만큼 반복하는 것이다.  `5^3 mod 3`을 예로 들면 다음과 같다.

```
5^5 mod 3
= (5 * 5 * 5 * 5 * 5) mod 3
= ((5 mod 3) * 5 * 5 * 5 * 5) mod 3
= ((10 mod 3) * 5 * 5 * 5) mod 3
= ((5 mod 3) * 5 * 5) mod 3
= ((10 mod 3) * 5) mod 3
= 5 mod 3
= 2
```

이를 구현한 코드는 [여기](https://github.com/codehumane/learn-algorithm-in-java/blob/master/src/main/java/math/ModularExponentiation.java), 테스트 코드는 [여기](https://github.com/codehumane/learn-algorithm-in-java/blob/master/src/test/java/math/ModularExponentiationTest.java)에 기록했다. 코드를 보면 알겠지만 알고리즘의 수행시간은 `y`의 크기에 비례한다. 정확히는 `y + 1`만큼 함수가 호출된다. 즉, O(y) 이다. 좀 더 빠른 알고리즘은 제곱을 이용하는 것이다. 아래와 같이 말이다.

```
5^5 mod 3
= (5 * 5 * 5 * 5 * 5) mod 3
= ((5 mod 3) * (5 mod 3) * 5 * 5 * 5) mod 3
= ((4 mod 3) * (4 mod 3) * 5) mod 3
= 5 mod 3
= 2
```

이를 구현한 코드는 [여기](https://github.com/codehumane/learn-algorithm-in-java/blob/master/src/main/java/math/ModularExponentiation.java)에 기록함. 코드를 보면 알겠지만 함수의 호출 횟수는 최대  `ln(y) + y/2 - 1`로 줄어들었다. Big O 표기법으로는 O(ln(y))이다. [Khan Academy의 Fast modular exponentiation](https://www.khanacademy.org/computing/computer-science/cryptography/modarithmetic/a/fast-modular-exponentiation)에 따르면, 책에는 언급되지 않는 조금 더 빠른 방법을 제시하고 있다.

- 지수를 바이너리(2진수)로 변환한다.
- 가장 오른쪽 숫자부터 순서를 매기고 이를 k라고 부르자. 이 때 시작 값은 0이다.
- 숫자가 1이면 2^k로 변환하고, 그렇지 않으면 0으로 바꾸자.
- 이렇게 하면 지수가 2의 배수로 이뤄진 덧셈이 된다.
- 이제 오로지 제곱 알고리즘만을 이용하여 모듈러 연산을 구할 수 있다.
- `5^117 mod 19`를 예로 들어 설명하면 다음과 같다.

```
5^117 mod 19

// 변환
= 5^(1 + 4 + 16 + 32 + 64) mod 19
= (5^1 * 5^4 * 5^16 * 5^32 * 5^64) mod 19

// 첫번째 계산
= ((5 mod 19) * 5^4 * 5^16 * 5^32 * 5^64) mod 19
= (5 * 5^4 * 5^16 * 5^32 * 5^64) mod 19

// 두번째 계산
= (5 * ((5^2 mod 19) * (5^2 mod 19))) * 5^16 * 5^32 * 5^64) mod 19
= (5 * (((5^1 mod 19) * (5^1 mod 19)) * (5^2 mod 19)) ) * 5^16 * 5^32 * 5^64) mod 19
= (5 * ((5 * 5) * (5^2 mod 19)) ) * 5^16 * 5^32 * 5^64) mod 19
= (5 * ((25 mod 19) * (5^2 mod 19)) ) * 5^16 * 5^32 * 5^64) mod 19
= (5 * (6 * 6) * 5^16 * 5^32 * 5^64) mod 19
= (5 * (36 mod 19) * 5^16 * 5^32 * 5^64) mod 19
= (5 * 17 * 5^16 * 5^32 * 5^64) mod 19

// 다섯번째까지 반복하면..
= (5 * 17 * 16 * 9 * 5) mod 19
= 61200 mod 19
= 1
```

이를 구현한 코드는 [여기](https://github.com/codehumane/learn-algorithm-in-java/blob/master/src/main/java/math/ModularExponentiation.java), 테스트 코드는 [여기](https://github.com/codehumane/learn-algorithm-in-java/blob/master/src/test/java/math/ModularExponentiationTest.java)에 기록했다. 시간 복잡도를 분석해보면 최대 `⎡ln(y)⎤ + ⎡ln(⎡ln(y)⎤)⎤ + ⎡ln(⎡ln(y) - 1⎤)⎤+⎡ln(⎡ln(y) - 2⎤)⎤+ … +  1`가 될 것 같다(?).

### 유클리드의 최대 공약수 알고리즘

최대 공약수<sup>gcd, greatest common divisor</sup>를 찾기 위해 사용되는 유클리드의 법칙은 아래와 같다.

> 유클리드 법칙 1: x와 y가 양의 정수이고 x ≥ y를 만족하면, 최대 공약수 gcd(x, y) = gcd(x mod y, y) 이다.

이 법칙의 증명 과정은 [칸아카데미 - 유클리드 호제법](https://ko.khanacademy.org/computing/computer-science/cryptography/modarithmetic/a/the-euclidean-algorithm)이 좀 더 잘 설명하고 있다. 혹은 간단히 `x ≡ y (mod N) 이면, (x - y)가 N으로 나누어 떨어진다`는 개념을 반복하면, 위 법칙을 쉽게 이해할 수 있다. 그 외에 책에는 언급되지 않는 두 가지 법칙이 더 필요한데, 이들은 다음과 같다.

> 유클리드 법칙 2: gcd(x, 0) = x
>
> 유클리드 법칙 3: gcd(y, 0) = y

유클리드 법칙 1로 값을 줄여나가다 보면, 법칙 2 또는 3을 만나게 되며, 이 때의 값이 바로 최대공약수가 된다. 간단히 코드로 표현하면 다음과 같다. 전체 코드는 [여기](https://github.com/codehumane/learn-algorithm-in-java/blob/master/src/main/java/math/GCD.java)를 참고.

```java
// x >= y
int gcd(int x, int y) {
  if (y == 0) return x;
  return gcd(x, x%y);
}
```

이제 이 알고리즘의 수행시간을 생각해보자. 한 번의 함수 호출 시, x는 x/2 이하로 줄어든다. 이는 책에서 나온 아래 그림을 통해 쉽게 유추할 수 있다.

![x mod y의 최대 크기](./euclid-time-complexity.jpg)

그리고 그 다음 재귀호출 시에는 x와 y가 서로 바뀌므로, 이번에는 y가 y/2 이하로 줄어든다. 이를 반복해가면, 결국 함수의 호출이 최대 ⎡ln(x) + ln(y)⎤만큼 일어남을 알 수 있다.

### 유클리드 알고리즘의 확장

누군가 `gcd(a, b) = d`라고 할 때, 이를 어떻게 증명할 수 있을까? 책에서는 아래의 정리를 통해 가능하다고 한다.

> a와 b가 d로 나누어 떨어지고, d = ax + by를 만족하는 정수 x와 y가 존재하면 d는 gcd(a, b)이다.

정리의 증명은 다음과 같다.

1. 첫 번째 조건에 따라 d는 a와 b의 공약수이다.
2. 이로 인해, d ≤ gcd(a, b)가 성립한다.
3. 또한, gcd(a, b)는 a와 b의 공약수이므로, ax + by의 약수이다.
4. 이로 인해, gcd(a, b) ≤ d가 성립한다.
5. 결국, d = gcd(a, b)가 된다.


모듈러 집합을 떠올리며 증명을 이해할 수 있긴 하지만, 책의 뒷부분에서 소개되는 식(수학적 귀납법)이 조금 더 와닿는다.

1. `d = gcd(a, b) = ax + by`
2. `gcd(a, b) = gcd(b, a mod b)` 이므로,
3. 유클리드 법칙을 이용하면, `gcd(a, b) = gcd(b, a mod b) = bx' + (a mod b)y'`
4. 또한, `(a mod b)`를 `(a - ⎣a/b⎦b)`로 바꿔 쓸 수 있으므로,
5. `bx' + (a mod b)y' = bx' + (a - ⎣a/b⎦b)y' = ay' + b(x' - ⎣a/b⎦y')`
6. `x = y', y = x' - ⎣a/b⎦y'`라고 표현할 수 있으므로, `d = ax + by`가 성립한다.

전체식은 아래와 같다.

> d = gcd(a, b) = gcd(b, a mod b) = bx' + (a mod b)y' = bx' + (a - ⎣a/b⎦b)y' = ay' + b(x' - ⎣a/b⎦y') = ax + by

이제 이 정리를 이용해서 알고리즘을 작성해 볼 수 있다. 아래는 두 개의 양의 정수 a, b(a ≥ b ≥ 0)가 주어졌을 때, d = gcd(a, b) 이면서 ax + by = d를 만족하는 정수 x, y, d를 구하는 코드이다.

```java
static Result get(int a, int b) {
    if (b == 0) return Result.of(1, 0, a);
    final Result r = get(b, a % b);
    return Result.of(r.y, r.x - a / b * r.y, r.d);
}
```

전체 코드는 [여기](https://github.com/codehumane/learn-algorithm-in-java/blob/master/src/main/java/math/EuclidExtended.java)에 기록함.

### 모듈러 역수

책은 모듈러 나눗셈을 소제목으로 사용하고 있지만, 실제로는 모듈러 역수<sup>modular inverse</sup>를 다루고 있다. 또한 [이 문서](https://ko.khanacademy.org/computing/computer-science/cryptography/modarithmetic/a/modular-inverses)에 설명하는 바와 같이 모듈러 연산에는 나눗셈이 없기도 하다. 어쨌든, 0이 아닌 모든 실수 a에 대해 곱셈의 결과가 1이 되는 수를 가리켜 a의 역수라고 부른다. `1/a`를 생각하면 된다. 표기는 `a^-1`로 한다. 모듈러 연산에서도 아래와 같이 역수를 정의할 수 있다.

> (a * a^-1) ≡ 1 (mod N) 혹은 (a * a^-1) mod N = 1

굳이 책을 보지 않아도 쉽게 떠올려 볼 수 있는 알고리즘은 다음과 같다. a^-1을 b라고 두고, b를 0부터 N-1까지 늘려가며 a * b를 N으로 나눠보는 것이다. 나머지가 1이면, 이 때의 b가 바로 역수가 된다. 코드는 [여기](https://github.com/codehumane/learn-algorithm-in-java/blob/master/src/main/java/math/ModularInverse.java)에 기록했다. 한가지 주의할 것은, a와 N이 서로소<sup>coprime</sup>인 경우에만 역수가 존재한다는 점이다. 책에서는 명시적으로 서로소를 언급하는 대신, 아래와 같이 최대 공약수를 이용하여 증명하고 있다.

> x modulo N의 역은 최대 하나 밖에 없고, … (중략) … 이 역은 항상 존재하지는 않습니다.

1. `ax mod N`은 `ax + kN` 형태로 나타낼 수 있음 (`k`가 음수라고 생각하면 이해하기 쉽다)
2. 따라서, `ax mod N`은 `gcd(a, N)`으로 나누어 떨어짐 (당연한 이야기)
3. `gcd(a, N) > 1`이면, 어떤 `x`에 대해서도 `ax ≡ 1 mod N`을 만족할 수 없음
4. 따라서, `gcd(a, N) > 1`이면, `a`는 `mod N` 에 대한 역수를 가질 수 없음

여기서 3번째 문장이 이해되지 않는다. 하지만, 이렇게 이야기하면 이해하기 쉽다. `ax mod N = 1`을 만족해야 하는데, 만약 a와 N이 2 이상의 공약수를 가진다면, a가 N으로 나누어떨어진다는 이야기다. 즉, 나머지가 1이 될 수 없다. 따라서, a가 mod N에 대한 역수를 가질 수 없다고 할 수 있다.

이제는 책에서 소개되는 유클리드 규칙을 이용한 알고리즘을 살펴보자.

1. `gcd(a, N) = 1`이면, `ax + Ny = 1`을 만족하는 `x`와 `y`를 찾을 수 있다.
2. `ax ≡ 1 (mod N)`이므로 `x`는 `a`의 역수가 된다.

전체 코드는 [여기](https://github.com/codehumane/learn-algorithm-in-java/blob/master/src/main/java/math/ModularInverse.java)를 참고하고, 간단히 기록하면 아래와 같다.

```java
static int get(int a, int N) {
    final Result result = EuclidExtended.get(a, N);
    if (result.getD() > 1) return -1;
    return result.getX();
}
```

