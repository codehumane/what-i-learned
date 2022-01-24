# Grokking Algorithms

# Chapter 5. Quicksort

- 알고리즘은 하나의 유형만을 해결.
- 한편 D&C(Divide and Conquer)는 문제 해결의 생각 도구.
- 어려운 문제를 만나면 "D&C를 쓸 수는 없을까?"를 떠올리기.
- 퀵소트는 대표적인 D&C.

## Divide & conquer

아래의 농장을 정사각형 모양으로 균등하게 나누고 싶음. 이 정사각형은 가능한 커야 함.

![땅을 가능한 큰 정사각형으로 균일하게 나누기](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617292231/files/OEBPS/Images/052fig02.jpg)

D&C는 크게 2가지 단계.

1. 기저 사례<sup>base case</sup>(가장 단순한 경우) 찾기.
2. 문제가 기저 사례가 될 때까지 문제를 나누고 줄이기.

이를 농장 문제에 적용.

- 일단 기저 사례부터 찾자.
- 세로가 가로의 2배가 되는 경우도 단순.
- 이제 재귀 사례를 생각해 내야 함.
- 1680x640 농장이라고 가정.
- 640x640으로 나눠보면, 400x640 공간이 남게 됨.
- 그럼 이 공간을 대상으로 다시 또 400x400으로 나눠보기. 반복.
- 1680x640 → 400x640 → 240x400 → 160x240 → 80x160 → 0x0
- 마지막에 만난 80x160이 기저 사례.

한 가지 예제 더 다룸. `[2, 4, 6]` 같은 정수형 배열이 주어졌을 때, 이 값들의 총합을 반환.

```py
def sum(arr):
    total = 0
    for x in arr:
        total += x
    return total

print sum([1, 2, 3, 4])
```

위와 같이 할 수도 있겠지만, 재귀 함수를 써보자.

1. Step 1: 기저 사례 찾기. 0 또는 1개 원소만 있는 경우.
2. Step 2: 문제를 기저 사례만큼 계속 나눠보기.

![재귀로 배열 합 구하기](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617292231/files/OEBPS/Images/058fig01_alt.jpg)
