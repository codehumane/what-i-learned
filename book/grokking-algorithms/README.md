# Grokking Algorithms

# Chapter 5. Quicksort

- 알고리즘은 하나의 유형만을 해결.
- 한편 D&C(Divide and Conquer)는 문제 해결의 생각 도구.
- 어려운 문제를 만나면 "D&C를 쓸 수는 없을까?"를 떠올리기.
- 퀵정렬는 대표적인 D&C.

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

## Quicksort

일단 기저 사례는 2가지.

- 빈 배열
- 원소가 1개인 배열

코드로 나타내면 아래와 같음.

```py
def quicksort(array):
    if len(array) < 2:
        return array
```

2개인 경우를 고민.

- 만약 첫 번째 원소가,
- 두 번째 원소보다 작다면 그대로 두고,
- 크다면 서로 교체.

3개면 어떻게 될까?

- D&C를 사용하고 있으므로,
- 기저 사례가 될 때까지 쪼개기.

이제 퀵정렬를 소개.

- 배열에서 한 원소를 고름.
- 이를 피벗<sup>pivot</sup>이라 부름.
- 이제 이 원소보다 작은 것은 왼쪽으로,
- 큰 것은 오른쪽으로 보냄.
- 이는 파티셔닝이라 불림.
- `[33,15,10]`에서 `33`을 피벗으로 고르면,
- `[15,10]`과 `[]`로 파티셔닝 됨.
- 만약 파티셔닝 된 서브 배열들이 정렬되어 있다면,
- 이 둘을 합치는 것으로 전체 정렬이 됨.
- 원소가 2개인 `[15,10]` 배열과,
- 빈 배열인 `[]`에 대해서는,
- 위에서 얘기한 것처럼 정렬하면 됨.

[GeeksforGeeks에 나온 수도코드](https://www.geeksforgeeks.org/quick-sort/).

```
/* low  --> Starting index,  high  --> Ending index */
quickSort(arr[], low, high)
{
    if (low < high)
    {
        /* pi is partitioning index, arr[pi] is now
           at right place */
        pi = partition(arr, low, high);

        quickSort(arr, low, pi - 1);  // Before pi
        quickSort(arr, pi + 1, high); // After pi
    }
}
```

[코틀린으로 작성했었던 퀵정렬](https://github.com/codehumane/algorithm/blob/master/src/main/kotlin/basic/sort/quicksort.kt)도 참고.

## Big O notation revisited

- 퀵정렬는 피벗으로 무엇을 선택했느냐에 따라 성능이 달라짐.
- 최악의 경우는 O(n^2).
- 선택정렬과 같은 것.
- 평균적으로는 O(n·logn) 소요.
- 한편 병합정렬은 항상 O(n·logn).
- 그렇다면 병합정렬을 써야 하는 건 아닌지 의아할 것.

### Merge sort vs. quicksort

- O(n)의 실제 의미는 c x n.
- 여기서 c는 some fixed amount of time.
- 일반적으로는 c는 의미 없음.
- 그러나 때로는 큰 차이를 만들기도 함.
- 퀵정렬과 병합정렬에서도 이 차이가 의미 있는 수준.
- 둘 다 O(n·logn)이지만 퀵정렬이 빠름.
- 왜 그런지는 [여기](https://www.geeksforgeeks.org/quicksort-better-mergesort/)에 좀 더 설명 되어 있음.
- 부가적 공간이 필요 없고, 최악 케이스는 발생 가능성 적고, 참조 국소성 때문.
- 한편, 병합정렬이 큰 데이터 구조체에서는 더 낫다고 함. 
- 게다가 퀵정렬은 최악의 케이스(n^2)보단 평균 케이스(n·logn)가 주로 일어남.
- 병합정렬 코드는 [여기](https://github.com/codehumane/algorithm/blob/master/src/main/java/sort/MergeSort.java), 퀵정렬은 [여기](https://github.com/codehumane/algorithm/blob/master/src/main/kotlin/basic/sort/quicksort.kt) 참고.

### Average case vs. worst case

- 피벗으로 첫 번째 것을 고르면 느림.
- 가운데 것을 고르면 빠름.
- 이미 정렬된 `[1,2,3,4,5,6,7,8]` 배열이 있다고 가정.
- 첫 번째를 피벗으로 고르면 8번의 콜 스택 발생.
- 가운데 값을 피벗으로 고르면 4번의 콜 스택.
- 그래서 O(n·logn).

# Chapter 7. Dijkstra's algorithm

- 그래프에서 A에서 B로 가는 방법 이야기.
- 경로가 가장 빠른 것과 가장 짧은 것은 서로 다름.
- 만약 각 선마다 소요 시간을 추가한다면,
- 가장 짧은 시간이 걸리는 경로를 찾을 수 있음.
- 더 많은 선을 거치더라도 더 빠를 수 있음.
- 가장 적은 선을 거치는 경로는 BFS로 찾을 수 있음.
- 반면 가장 빠른 경로에는 Dikstra 알고리즘을 사용.
