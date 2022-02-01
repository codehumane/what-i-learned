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
- 반면 가장 빠른 경로에는 Dijkstra 알고리즘을 사용.

## Working with Dijkstra's algorithm

![Dijkstra graph](./7-1-dijkstra-graph.jpeg)

Dijkstra 알고리즘은 4단계.

1. 가장 저렴한 노드 찾기
2. 이 노드 이웃들의 비용을 업데이트
3. 그래프의 모든 노드에 대해 이 작업을 반복
4. 최종 경로 계산

이렇게만 보면 다소 어려움. 하나씩 구체적으로 살펴봄.

### Step 1

- 가장 저렴한 노드 찾기.
- 시작점인 S에 있다고 가정.
- A로는 6분이 걸리고,
- B로는 2분이 걸림.
- F까지는 얼마나 걸릴지 아직 알 수 없으니 무한.
- 일단은 B가 가장 가까운 노드.
- 표로 나타내면 아래와 같음.

| node | time to node |
| ---- | ------------ |
| A    | 6            |
| B    | 2            |
| F    | ∞            |

### Step 2

- 이제 B에서 B의 이웃에게 가는 시간을 계산.
- B에서 A로 가는 시간은 3이고,
- B에서 F로 가는 시간은 이제 7임을 알 수 있음.
- 그리고 A로 가는 시간이 S -> A보다 S -> B -> A가 더 적음을 발견.
- 그래서 아래 표와 같이 갱신.
- 이런 식으로 좀 더 짧은 시간이 발견되면 계속 갱신.

| node | time to node |
| ---- | ------------ |
| A    | 6 -> 5       |
| B    | 2            |
| F    | ∞ -> 7       |

### Step 3

- 일단 Step 1을 반복.
- 이제 B 다음으로 가장 비용 적은 노드 찾기.
- 그럼 A.
- 이제 A의 이웃 노드들에 대한 비용 계산.
- 갱신이 일어나야 하는 곳은 A에서 F로의 경로.
- 표에서 F가 7이었는데 A를 거치면 6으로 충분.
- 따라서 아래와 같이 갱신.

| node | time to node |
| ---- | ------------ |
| A    | 6 -> 5       |
| B    | 2            |
| F    | ∞ -> 7 -> 6  |

- 이제 모든 노드에 대해 Dijkstra 알고리즘을 수행한 것.
- F 노드에 대해서는 안 해도 됨.

### Step 4

- 지금까지의 결과로 최종 경로를 계산할 수 있음.
- S -> B -> A -> F

## Terminology

몇 가지 예시를 더 들기에 앞서 용어 정리. 참고로, Dijkstra 알고리즘은 순환이 없는 그래프에서만 동작함. 순환이 없다는 것은 유향 그래프라는 얘기이기도 함.

- 그래프의 각 간선<sup>edge</sup>에 번호가 있었음.
- 이는 가중치<sup>weight</sup>라 부름.
- 그리고 이런 그래프를 가중 그래프라 부름.
- 그래프는 또한 순환<sup>cycle</sup>을 갖기도함.
- 이는 처음 시작한 곳으로 다시 도달할 수 있는 그래프를 말함.
- 마지막으로 유향<sup>directed</sup>과 무향<sup>undirected</sup> 그래프 이야기도.

## Trading for a piano

결국 아래 그림에서 book을 piano로 바꾸는 데 들이는 적은 비용을 찾는 것.

![dijkstra example 2](./7-2-dijkstra-example2.jpeg)

일단 시작 점의 간선 비용들부터 측정. 더불어 이번엔 parent도 함께 관리.

| parent | node | cost |
| ------ | ---- | ---- |
| B      | L    | 5    |
| B      | P    | 0    |
| -      | G    | ∞    |
| -      | D    | ∞    |
| -      | N    | ∞    |

**Step 1**

- 가장 저렴한 노드 찾아야 함.
- 지금은 P.

**Step 2**

- 이제 P에서 이웃 노드로의 비용 측정.
- G, D로의 비용 측정이 가능하며, 이를 표에 반영하면 아래와 같음.

| parent | node | cost |
| ------ | ---- | ---- |
| B      | L    | 5    |
| B      | P    | 0    |
| P      | G    | 30   |
| P      | D    | 35   |
| -      | N    | ∞    |

**Step 1 again**

- P다음으로 저렴한 노드 찾기.
- 5를 가진 L.

**Step 2 again**

- L의 이웃 값들을 모두 갱신.
- 그럼 G와 D인데,
- L에서 G와 D로 가는 값이,
- 기존 비용보다 저렴하므로,
- G, D의 parent와 cost를 모두 갱신.
- 결과는 아래와 같음.

| parent | node | cost |
| ------ | ---- | ---- |
| B      | L    | 5    |
| B      | P    | 0    |
| L      | G    | 20   |
| L      | D    | 25   |
| -      | N    | ∞    |

- 이제 L, P 다음으로 저렴한 G에 대해 Step 1, 2 반복.
- 그럼 N이 아래와 같이 갱신됨.

| parent | node | cost |
| ------ | ---- | ---- |
| B      | L    | 5    |
| B      | P    | 0    |
| L      | G    | 20   |
| L      | D    | 25   |
| G      | N    | 40   |

- 이제 D에 대해 Step 1, 2 반복.
- 이번에도 N이 갱신됨.

| parent | node | cost |
| ------ | ---- | ---- |
| B      | L    | 5    |
| B      | P    | 0    |
| L      | G    | 20   |
| L      | D    | 25   |
| D      | N    | 35   |

- 가장 저렴하게 피아노를 얻으면 35 달러가 됨을 알 수 있고,
- 표의 parent를 보고 경로도 완성할 수 있음.
- B → L → D → N

## Implementation

```py
node = find_lowest_cost_node(costs)

while node is not None:
    cost = costs[node]
    neighbors = graph[node]
    for n in neighbors.keys():
        new_cost = cost + neighbors[n]
        if costs[n] > new_cost:
            costs[n] = new_cost
            parents[n] = node
    processed.append(node)
    node = find_lowest_cost_node(costs)

def find_lowest_cost_node(costs):
    lowest_cost = float("inf")
    lowest_cost_node = None
    for node in costs:
        cost = costs[node]
        if cost < lowest_cost and node not in processed:
            lowest_cost = cost
            lowest_cost_node = node
    return lowest_cost_node
```

# Chapter 8. Greedy algorithms

## The classroom scheduling problem

아래의 수업 목록이 있을 때 가장 많은 수업을 듣기 위한 방법은?

| class | start | end   |
| ----- | ----- | ----- |
| art   | 9     | 10    |
| eng   | 9:30  | 10:30 |
| math  | 10    | 11    |
| cs    | 10:30 | 11:30 |
| music | 11    | 12    |

방법은 간단.

1. 가장 먼저 끝나는 수업을 선택
2. 그 수업이 끝난 뒤 시작하는 것을 고르는데, 이 때도 마찬가지로 가장 먼저 끝나는 대상을 고르면 됨

탐욕 알고리즘은 쉽다고 함. 각 단계에서 최적의 선택을 고르면 됨.

> In technical terms: at each step you pick the locally optimal solution, and in the end you're left with the globally optimal solution.
