# 코딩 인터뷰 완전 분석

# big-O

- big-O, big-Θ, big-Ω 비교
- Θ는 O(상한)와 Ω(하한) 둘 다 의미
- big-O에 대한 기본적인 개념들 소개
- 상환시간에 대한 얘기도. ArrayList의 삽입 시간이, 배열이 꽉 찼을 때는 O(N)이 되는데, 이를 평소의 삽입 시간에 분배하려는 시도. 결과는 O(1).
- 로그의 밑은 상수항으로 취급되므로 무시해도 되지만, 지수에서는 큰 차이가 있음.
- 재귀적 수행 시간도 언급. 아래 코드는 O(N^2)가 아니라 O(2^N).

```java
int f(int n) {
  if (n <= 1) {
    return 1;
  }
  return f(n - 1) + f(n - 1);
}
```

# 기술적 문제

## 알고 있어야 할 것들

이진 트리의 균형을 맞추는 특정 방법 등의 복잡한 알고리즘 보다는 아래의 기본적인 것들을 알고 있어야 한다고 이야기.

자료구조

- Linked List
- Tree
- Tries
- Graph
- Stack & Queue
- Vector / ArrayList
- Hash Table

알고리즘

- BFS
- DFS
- Binary Search
- Merge Sort
- Quick Sort

개념

- Bit Manipulation
- Memory (Stack vs Heap)
- Recursion
- Dynamic Programming
- big-O 시간 & 공간

## 실제 문제 살펴보기

> 뻔한 내용이지만, 이렇게 정리하는 것도 마지막일 거라고 생각하고 일단 기록함.

경청하기

- 정확히 이해했는지 확인부터.
- 확실하지 않은 부분이 있다면 질문을 통해 짚고 넘어가야.
- 문제와 관련된 독특한 정보는 머릿 속에 기억(이유가 있기 때문에 주어지는 정보인 경우가 많음).
    - "정렬된 배열 두 개가 주어졌을 때"
    - "서버에서 반복적으로 수행되는 알고리즘을 설계하려고 하는데"

예제를 직접 그려보기

- 바로 문제를 푸는 대신, 예제를 직접 그려 보기를 권장.
- 실제 값을 대입해서 사용하고(단서를 찾고자),
- 충분히 큰 예제를 써야 하고(패턴을 찾고자),
- 특별한 예제는 지양(잘못된 가정을 피하고자. 예컨대 트리 중에서도 균형 잡힌 완벽 트리는 지양)

무식한 방법으로 일단 해보기

- brute force
- 일단 만들고,
- 시간과 공간복잡도를 이야기 하고,
- 최적화를 위한 토론의 시작점으로 삼기.

최적화

- 간과한 정보가 있는지 찾는다. 배열이 이미 정렬되어 있다던가 하는.
- 새로운 예제를 만들어 보기도.
- 공간과 시간의 실익을 따지고 균형 맞추기.
- 미리 계산해 두라. 해시 테이블에 넣어 두거나, 정렬을 해 둔다던가.

검토하기

- 바로 코딩에 들어가지 말고, 잠시 생각하며 이해를 확실히 한다.
- 정확히 무엇을 작성해야 하는지 모른다면, 시간도 오래 걸리고 심각한 실수도 하기 마련.

코드 작성하기

- 모듈화된 코드를 사용. 책에서 말하는 내용은 composed method의 의미.
- 필요하다면 다른 클래스나 구조체 사용.
- 좋은 변수명.

테스트

- 개념적 테스트부터 시작. (코드를 한 줄씩 읽어 내려가며 어떤 일을 수행하는지 분석하는)
- 작은 규모부터 테스트.
- Null, 단일 원소, 극단적 입력 등 특별한 경우를 테스트.

## 최적화 및 문제풀이 기술 #5: 자료구조 브레인스토밍

다른 내용들은 익숙한데, 이 부분은 그렇지 못해서 따로 기록까지 진행. 일단, 여기서 다루는 문제는 아래와 같음.

> 임의의 숫자를 만들어 낸 뒤 확장 가능한 배열에 차례로 저장한다. 새로운 입력이 들어올 때마다 중간값(median)을 구하려면 어떻게 해야 하는가?

이를 풀기위한 접근법으로 여러 자료구조를 먼저 떠올림.

- 연결리스트
- 배열
- 이진트리
- 힙

연결리스트는 배제. 수열을 정렬하거나 특정 수에 바로 접근하기에는 약한 구조. 배열은 가능하긴 함. 배열을 매번 정렬된 상태로 유지하면 됨. 하지만 비용이 작지는 않음. 이진트리는 배열보다는 나음. 원소 삽입마다 정렬된 상태를 유지하기 쉽기 때문. 그리고 원소 갯수가 홀수이면, 트리의 루트 원소가 중앙값이 됨. 하지만, 짝수일 때의 중앙값은 가운데 두 값의 평균을 취해야 한다. 이진트리가 괜찮긴 하지만 짝수인 경우를 생각하면 좀 더 고민이 필요해 보임. 마지막으로 힙. 힙을 그냥 쓰면 안 되고, 두 개를 사용하는 것을 고려. 한 쪽은 작은 수들을, 다른 한 쪽은 큰 수들을 두고 관리. 이진트리의 한계가 극복됨.

참고로, 중앙값에 대한 정의는 [여기](https://ko.wikipedia.org/wiki/%EC%A4%91%EC%95%99%EA%B0%92)를 참고. 그리고 이진트리와 힙의 차이는 [여기](https://cs.stackexchange.com/questions/27860/whats-the-difference-between-a-binary-search-tree-and-a-binary-heap) 설명만으로도 충분해 보임.

# 면접 문제

## 배열과 문자열

### 해시테이블

- 해시테이블을 구현하는 방법으로 balanced binary search tree 자료구조도 함께 이야기.
- 이 때는 탐색이 O(logN)이 됨.
- 대신, 크기가 큰 배열을 미리 할당해 놓지 않아도 되므로 잠재적으로 작은 공간을 사용.
- 또한, 키의 집합을 특정 순서로 차례대로 접근하므로 상황에 따라 유용하기도.
- 원래 알고 있는 방식은 배열과 연결리스트 사용.
- 따라서 탐색은 O(1).
- 물론 최악의 경우는 매번 해시 충돌이 일어나 탐색이 O(N)이 될 수도.

### ArrayList와 가변 크기 배열

- ArrayList에 데이터를 덧붙일 때는 O(1)이 걸림.
- 상환 입력 시간(amortized insertion time)을 고려해도 마찬가지.
- 크기가 N인 배열이 있다고 가정하고, 크기가 1일 때부터 얼마나 많은 원소가 복사됐는지를 살펴보면 됨.
- 1 + 2 + ... + n/4 + n/2
- [제논의 역설](https://ko.wikipedia.org/wiki/%EC%A0%9C%EB%85%BC%EC%9D%98_%EC%97%AD%EC%84%A4)이 떠오름. 무한급수.


### StringBuilder

- 지금은 유효하지 않은 지식. 그냥 생각해 보는 재미.
- 아래와 같은 코드가 있다고 할 때 수행시간은 얼마나 될까?

```java
String joinWords(String[] words) {
    String sentence = "";
    for (String w : words) {
        sentence = sentence + w;
    }
    return sentence;
}
```

- 모든 문자열의 길이가 x라고 하고, n개의 문자열이 주어졌다고 가정.
- 그럼 수행시간은 O(x + 2x + ... + nx) = O(xn^2)
- 매번 두 개의 문자열을 읽어들인 뒤, 새로운 문자열로 복사하기 때문.
- 한편, StringBuilder는 가변 크기 배열을 이용하여 필요할 때만 문자열을 복사.

## 연결리스트

- 연결리스트가 단방향인지 양방향인지 고려.
- 그리고 runner 기법도 소개. 부가 포인터라고도 불림.
- 연결리스트 순회 시, 두 개의 포인터를 사용하는 것.
- 예컨대, slow pointer와 fast pointer를 두고,
- fast pointer가 2칸씩 이동할 때 slow pointer는 1칸씩 이동시킴.
- fast pointer가 끝에 도달하면, slow pointer는 절반에 도달.
- 한 번의 순회로 절반이 어딘지 알아낼 수 있는 것.


## 스택과 큐

### 스택 구현하기

- 말 그대로 쌓는다는 의미(stack)
- LIFO이며, pop, push, peek, isEmpty 연산 제공.
- 재귀 알고리즘에 유용. 재귀 함수를 빠져나와 퇴각 검색(backtrack)을 할 때.
- 책에서는 배열과 달리 i번째 항목에 상수 시간에 접근할 수 없다고 함.
- 하지만, java.util.Stack을 열어 보면 그렇지 않음. add와 get 구현은 아래와 같음.

```java
public
class Stack<E> extends Vector<E> {
  
  public E push(E item) {
    addElement(item);
    return item;
  }

  public synchronized void addElement(E obj) {
    modCount++;
    add(obj, elementData, elementCount);
  }

  private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
      elementData = grow();
    elementData[s] = e;
    elementCount = s + 1;
  }

}

// get은 vector에 있음
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
 
  public synchronized E get(int index) {
    if (index >= elementCount)
      throw new ArrayIndexOutOfBoundsException(index);

    return elementData(index);
  }
}
```

- 그래서, queue로 쓸 때는 LinkedList 보다 빠르고, stack으로 쓸 때는 Stack 보다 빠른 ArrayDeque를 살펴봄.
- 하지만 get을 제공하지는 않음. 내부적으로는 array를 사용하는데 get을 제공하지 않는 이유는 뭔지 궁금.

### 큐 구현하기

- FIFO. add, remove, peek, isEmpty 연산 제공.
- stack과 마찬가지로 연결리스트를 이용해 구현 가능.
- BFS나 캐시 구현에 종종 사용. 노드를 접근한 순서를 기억하고 있다가 뭔가를 해야 할 때 유용.


## 트리와 그래프

### 트리의 종류

- 트리를 이해하는 데 재귀적 설명을 이용하는 게 인상적.
- 트리는 하나의 루트를 갖는다.
- 루트 노드는 0개 이상의 자식 노드를 갖는다.
- 자식 노드 또한 0개 이상의 자식 노드를 갖는다. 이는 반복적으로 정의됨.
- 간단히 작성해 보면 아래와 같음.

```kotlin
data class Tree<T>(val root: Node<T>) {

  data class Node<T>(
    val data: T,
    val parent: Node<T>?,
    val children: List<Node<T>>
  )

}
```

#### 트리 vs. 이진 트리

- 모든 트리가 이진 트리는 아님.
- 이진 트리는 각 노드가 최대 2개의 자식을 갖는 트리.

#### 이진 트리 vs. 이진 탐색 트리

- 트리 문제가 주어지면 이진 탐색 트리일 것이라고 가정하는 경향에 유의.
- 이진 탐색 트리는 모든 노드가 다음의 순서 규칙을 따르는 이진 트리를 일컬음.
- '모든 왼쪽 자식들 <= n < 모든 오른쪽 자식들'
- 참고로, 같은 값을 처리하는 방식에 따라 이진 탐색 트리는 약간씩 정의가 달라질 수 있음.
- 어떤 곳은 중복된 값을 가지면 안 되고, 어떤 곳은 오른쪽이나 양쪽 어느 곳이든 존재할 수 있기도 함. 미리 명확히 할 필요 O.
- 아래는 위키피디아의 정의.

> A binary search tree is a rooted binary tree, whose internal nodes each store a key (and optionally, an associated value) and each have two distinguished sub-trees, commonly denoted left and right. The tree additionally satisfies the binary search property, which states that the key in each node must be greater than or equal to any key stored in the left sub-tree, and less than or equal to any key stored in the right sub-tree.

![binary search tree](https://upload.wikimedia.org/wikipedia/commons/thumb/d/da/Binary_search_tree.svg/200px-Binary_search_tree.svg.png)

#### 균형 vs. 비균형

- 균형을 잡는다는 것이 왼쪽과 오른쪽 트리 크기가 완전히 같다는 것을 의미하지는 않음.
- '불균형'과 '너무 불균형은 아닌것'과 '균형'은 모두 다르며 각각 의미가 있음.
- 레드-블랙 트리의 경우, O(logN) 시간에 insert와 find를 할 수 있을 정도로 균형이 잡혀 있지만 완벽한 균형은 아님.
- 하지만 정작 균형에 대한 정의는 빠져 있음.

#### 완전 이진 트리

- complete binary tree
- 모든 높이에서 노드가 꽉 차 있는 이진 트리.
- 다만, 마지막 레벨은 꽉 차 있지 않아도 되나, 왼쪽부터 채워져 있어야 함.
- 생각하는 프로그래밍에 나온 [힙에 대한 정리](https://github.com/codehumane/what-i-learned/blob/master/book/pp/README.md#%ED%9E%99heaps)도 참고하면 좋을 듯. 다만, 위에서 아래로 오름차순인지 내림차순인지의 특성이 더해질 뿐.

#### 전 이진 트리

- full binary tree
- 모든 노드가 자식이 없거나 정확히 2개 있는 경우.

#### 포화 이진 트리

- full binary tree 이면서, complete binary tree인 경우
- 모든 말단 노드가 같은 높이에 있어야 함.
- 마지막 단계의 노드 갯수는 최대가 되는 것.
- 노드의 갯수가 2^(k-1). k는 트리의 높이.

### 이진 트리 순회

이에 대한 설명은 [위키피디아 페이지](https://ko.wikipedia.org/wiki/%ED%8A%B8%EB%A6%AC_%EC%88%9C%ED%9A%8C)의 내용이 개인적으로 더 좋음.

![tree traversal example](https://upload.wikimedia.org/wikipedia/commons/thumb/6/67/Sorted_binary_tree.svg/250px-Sorted_binary_tree.svg.png)

- 중위 순회: A > B > C > D > E > F > G > H > I (left, root, right)
- 전위 순회: F > B > A > D > C > E > G > I > H (root, left, right)
- 후위 순회: A > C > D > E > B > H > I > G > F (left, right, root)

### 이진 힙

- 여기서는 최소 힙만 다룸.
- 트리의 마지막 단계를 제외하면 포화 이진 트리.
- 마지막 단계는 왼쪽부터 채워지기만 하면 됨.
- 그리고 최소 힙이므로 각 노드의 원소가 자식들의 원소보다 작음.
- 핵심 연산은 삽입과 최소 원소 뽑아내기.

![heap](https://upload.wikimedia.org/wikipedia/commons/thumb/3/38/Max-Heap.svg/240px-Max-Heap.svg.png)

#### 삽입

- 힙의 특성을 고려하여, 마지막 단계의 가장 오른쪽에 데이터를 삽입.
- 그리고 자리를 찾을 때까지 부모와 자리를 바꾸어 가며 이동.
- 당연히 O(logN) 소요.

#### 최소 원소 뽑아내기

- 최소 원소 찾기 자체는 쉬움. 루트.
- 뽑아낼 때는 아래의 절차를 따름.
- 루트 원소 제거.
- 마지막 단계의 가장 오른쪽 원소를 루트로 이동.
- 이 원소를 자식 노드와 크기 비교해 가며 교환.
- 최소 힙일 때는 왼쪽 자식 노드와 오른쪽 자식 노드 중 작은 것과 교환.
- 최대 힙은 반대. 조금만 생각해 보면 당연.
- 마찬가지로 O(logN) 소요.

### 트라이 (접두사 트리)

일단 위키피디아에서 가져온 아래의 그림으로 기본적인 설명은 대체.

![trie(prefix tree or digital tree)](https://upload.wikimedia.org/wikipedia/commons/thumb/b/be/Trie_example.svg/250px-Trie_example.svg.png)

- 문자열을 검색할 때는 해시테이블을 활용할 수 있지만,
- 주어진 문자열이 다른 문자들의 접두사인지 여부를 알려면,
- trie 구조가 빠름.
- 길이가 K인 접두사가 주어지면, 이 접두사가 유효한지 여부 판단은 O(K)에 수행.
- 이는 해시테이블 검색 비용과 동일한 것. 해시테이블도 O(1)이 아니라 문자열 K를 읽어들이는 O(K) 수행이 필요.

### 그래프

- 트리 -> 그래프 (트리는 충분조건)
- 책에서는 간단하게, cycle이 없는 하나의 connected graph를 트리로 정의.
- 또, 노드와 노드를 간선(edge)로 모아 둔 것이라고 하기도.
- 그래프에는 방향성이 있기도, 없기도 함.
- 사이클이 존재하기도, 존재하지 않기도. 사이클이 없으면 비순환 그래프(acyclic graph).
- 여러 개의 고립된 부분 그래프(isolated subgraphs)로 구성될 수도.
- 그래프 표현에는 인접 리스트 또는 인접 행렬을 사용.

#### 인접 리스트

- adjacency list
- 간단히 작성해 보면 아래와 같음.
- 트리에서는 굳이 Tree 클래스를 두지 않아도 됐었지만,
- 그래프는 부분 그래프로 이뤄지기도 하므로 Graph라는 클래스를 따로 둠.

```
class Graph {
    public Node[] nodes;
}

class Node {
    public String name;
    public Node[] children;
}
```

#### 인접 행렬

- N x N boolean matrix
- matrix[i][j]가 true이면 i에서 j로의 간선이 있다는 의미.
- 0과 1을 이용한 integer matrix를 사용할 수도.
- 무방향 그래프를 인접 행렬로 표현한다면 이 행렬은 대칭행렬(symmetric matrix)이 됨.
- 그래프 알고리즘은 인접 행렬과 인접 리스트에서 모두 사용 가능하지만, 인접 행렬은 효율성이 떨어질 수도.
- 어떤 노드에 인접한 노드를 찾기 위해 모든 노드를 순회해야 할 수도 있기 때문.

#### 그래프 탐색

- 당연히 DFS와 BFS 이야기.
- 모든 노드를 탐색하고자 할 때는 DFS가 좀 더 선호된다고 함. 간단해서.
- 한편, 노드 사이의 최단 경로나 임의의 경로를 찾고 싶을 때는 BFS가 일반적으로 더 나음.
- DFS를 이용하면, 최단 경로가 아닐 수도 있고, 관련 없는 너무 많은 관계를 살피게 될지도.
- 아래 코드는 <알고리즘> 책에 나왔던 의사 코드.
- 직접 구현했던 코드는 [여기](https://github.com/codehumane/algorithm/blob/3f5fa5d702/src/main/java/data/BreathFirstSearch.java) 참고.

```
for all u ∈ V:
  dist(u) = ∞

dist(s) = Θ
Q = [s] (queue containing just s)
while Q is not empty:
  u = eject(Q)
  for all edges (u, v) ∈ E:
    if dist(v) = ∞:
      inject(Q, v)
      dist(v) = dist(u) + 1
```

- 양방향 탐색(bidirectional search)도 언급. 최단 경로를 찾을 때 사용되곤 함.
- 모든 노드가 k개의 이웃 노드와 연결되어 있고, s에서 t로의 최단 거리가 d인 그래프를 가정.
- BFS는 O(k^d), 양방향 탐색은 O(k^(d/2))가 됨.

## 비트 조작

### 손으로 비트 조작 해보기

문제를 단순화 하기 위해 모든 숫자는 4비트라고 가정. 그리고 ^ 표기는 XOR, ~ 표기는 NOT.

문제 | 답
--- | --
0011 * 0101 | 1111
0110 + 0110 | 1100
0011 * 0011 | 1001
0100 * 0011 | 1100
1101 >> 2 | 0011
1101 ^ (~1101) | 1111
1101 ^ 0101 | 1000
1011 & (~0 << 2) | 1000

### 비트 조작을 할 때 알아야 할 사실들과 트릭들

아래 표현식을 알아두면 좋다고 함. 당연한 등식들. 여기서 0s는 모든 비트가 0인 값이고, 1s는 모든 비트가 1인 값.

| 등식 |
| -- |
| x ^ 0s = x |
| x & 0s = 0 |
| x \| 0s = x |
| x ^ 1s = ~x |
| x & 1s = x |
| x \| 1s = 1s |
| x ^ x = 0 |
| x & x = x |
| x \| x = x |

### 2의 보수와 음수

- 컴퓨터는 음수를 저장할 때 2의 보수 형태로 저장.
- 그리고 맨 앞자리는 부호 비트(sign bit)로 사용.
- 다시 말해, -K를 N비트의 2진수로 표현하면 `concat(1, 2^(N-1)-K)`가 됨.
- 또 다른 방법은 양수로 표현된 2진수를 뒤집은 뒤 1을 더해 주는 것.

양수 | 비트 | 음수 | 비트
---- | ---- | ---- | ----
7 | 0 111 | -1 | 1 111
6 | 0 110 | -2 | 1 110
5 | 0 101 | -3 | 1 101
4 | 0 100 | -4 | 1 100
3 | 0 011 | -5 | 1 011
2 | 0 010 | -6 | 1 010
1 | 0 001 | -7 | 1 001
0 | 0 000 | -0 | 1 000

### 산술 우측 시프트 vs. 논리 우측 시프트

- 산술 우측 시프트(arithmetic right shift)는 맨 앞의 부호비트는 그대로 두고 비트를 오른쪽으로 이동.
- 논리 우측 시프트(logical right shift)는 맨 앞의 부호비트를 포함하여 오른쪽으로 이동. 비게 된 부분에는 0을 채움.
- 산술 우측 시프트는 값을 '대략' 2로 나눈 효과가 있음. `>>` 연산과 같음.

![logical left shift one bit](https://upload.wikimedia.org/wikipedia/commons/thumb/5/5c/Rotate_left_logically.svg/300px-Rotate_left_logically.svg.png)

### 기본적인 비트 조작: 비트값 확인 및 채워넣기

```kotlin
// 비트값 확인
fun getBit(num: int, i: int) = ((num & (1 << i)) != 0)

// 비트값 채우기
fun setBit(num: int, i: int) = num | (1 << i)

// 비트값 삭제
fun clearBit(num: int, i: int) = num & ~(1 << i)

// 비트값 바꾸기
fun updateBit(num: int, i: int, bitIs1: boolean) = (num & ~(1 << i)) | ((if bitIs1 1 else 0) << i)
```

마지막 비트값 바꾸기는 자세히 보면, 비트값 삭제(0과 1인 경우 모두)와 비트값 채우기(1인 경우만)의 조합임을 알 수 있음.

## 수학 및 논리 퍼즐

### 소수

모든 자연수는 소수의 곱으로 나타낼 수 있음.

#### 가분성(divisibility)

- 어떤 수 x로 y를 나눌 수 있으려면,
- x를 소수의 곱으로 분할한 결과가,
- y를 소수의 곱으로 분할한 결과의 부분집합이어야 함.
- x = 2^(j0) * 3^(j1) * 5^(j2) * ... 이고,
- y = 2^(k0) * 3^(k1) * 5^(k2) * ... 일 때,
- 모든 i에 대해 ji <= ki를 만족해야 한다.
- 이로부터 x와 y의 최대 공약수, 최소 공배수는 다음과 같이 표현 가능.
- gcd(x, y) = 2^min(j0,k0) * 3^min(j1,k1) * 5^min(j2,k2) * ...
- lcm(x, y) = 2^max(j0,k0) * 3^max(j1,k1) * 5^max(j2,k2) * ...

#### 소수판별

- 어떤 수 n이 소수인지 판단하는 기본적인 방법은 2에서 n-1까지 루프를 돌면서 나누어지는 지를 확인하는 것.
- 살짝 개선해 본다면, 루프를 n의 제곱근까지만 돌면 됨.
- 다음으로는 [에라토스테네스의 체](https://ko.wikipedia.org/wiki/%EC%97%90%EB%9D%BC%ED%86%A0%EC%8A%A4%ED%85%8C%EB%84%A4%EC%8A%A4%EC%9D%98_%EC%B2%B4)를 활용.
- 에라토스테네스를 활용하되 배열에 홀수만 저장할 수도.

### 확률

#### A∩B의 확률

- [베이즈 정리](https://namu.wiki/w/%EB%B2%A0%EC%9D%B4%EC%A6%88%20%EC%A0%95%EB%A6%AC)를 이야기 함.
- 사건 A와 B가 있고, B가 일어난 것을 전제로 할 때 A가 일어날 조건부 확률을 구하고자 함.
- 그런데, 지금 알고 있는 건 A가 일어난 것을 전제로 한 B의 조건부 확률, A 확률, B 확률임.
- 그러면 아래와 같이 구하면 됨.

```
P(A|B) = P(A∩B) / P(B) = P(A) * P(B|A) / P(B)
```

- 책들에서 종종 역확률 이야기로 같이 언급되는 내용.

#### A∪B의 확률

```
P(A∪B) = P(A) + P(B) - P(A∩B)
```

#### 독립성

- P(B|A) = P(B)
- P(A∩B) = P(A) * P(B)

#### 상호 배타성

- P(A∩B) = 0
- P(A∪B) = P(A) + P(B)

## 재귀와 동적 프로그래밍

- 부분문제(subproblem)에 대한 해법을 통해 완성.
- 상향식, 하향식, 반반 접근법이 있음. 병합정렬이 반반 접근법.
- 재귀적 알고리즘은 공간 효율성이 안 좋을 수 있음. O(n)의 메모리를 사용해야 할 수도.
- 따라서 재귀적(recursive) 대신 순환적(iterative) 방법은 없을지 먼저 고민.
- 동적 프로그래밍과 메모이제이션도 언급.

### 피보나치

- 피보나치 문제를 재귀, 하향식, 상향식 동적 프로그래밍으로 각각 접근.

#### 재귀

- 일단, 피보나치 재귀 방식은 아래와 같음.

```kotlin
fun fibonacci(n: Int): Int {
    if (n == 1) return 0
    if (n == 2) return 1
    return fibonacci(n - 1) + fibonacci(n - 2)
}
```

- 이 방식의 시간 복잡도는 O(2^n).
- 수행 트리를 그려보면 쉽게 알 수 있음.

![피보나치 재귀 시간복잡도](https://qph.fs.quoracdn.net/main-qimg-07b90165ddb687ddf6700e90fea22c24.webp)

#### 하향식 동적 프로그래밍 (메모이제이션)

- 위 그림을 보면 같은 입력 값의 연산이 꽤 반복됨을 알 수 있음.
- 이 연산들을 캐시하는 방식은 아래와 같음.
- 시간 복잡도는 O(n).

```kotlin
fun fibonacci(n: Int): Int {
    val cache = IntArray(n + 1)
    return fibonacciMemoization(n, cache)
}

fun fibonacciMemoization(n: Int, cache: IntArray): Int {
    if (n == 1) return 0
    if (n == 2) return 1

    if (cache[n] == 0)
        cache[n] = fibonacci(n - 1) + fibonacci(n - 2)

    return cache[n]
}
```

#### 상향식 동적 프로그래밍

- 코드를 보면 알 수 있듯, 상향식도 가능함.
- 구현은 아래와 같고, 시간 복잡도도 O(n).

```kotlin
fun fibonacciTopDown(n: Int): Int {
    if (n < 2) return n - 1

    val cache = IntArray(n + 1) { -1 }
    cache[1] = 0
    cache[2] = 1

    (3..n).forEach {
        if (cache[it] == -1)
            cache[it] = cache[it - 1] + cache[it - 2]
    }

    return cache[n]
}
```

- 좀 더 살펴보면, `cache` 배열을 아래와 같이 제거할 수도 있음.
- `forEach` 문도 사라지고 좀 더 간결해 보임.

```kotlin
fun fibonacciTopDownWithoutArray(n: Int): Int {
    if (n < 2) return n - 1

    var cache1 = 0
    var cache2 = 1

    repeat(n - 2) {
        val current = cache1 + cache2
        cache1 = cache2
        cache2 = current
    }

    return cache2
}
```

## 시스템 설계 및 규모 확장성

### 시스템 설계: 단계별 접근법

TinyURL 같은 시스템 설계를 예시로 사용.

#### 1단계: 문제의 범위를 한정

- 시스템을 알아야 설계가 가능.
- 개개인이 원하는 형태로 축약 URL을 만들어야 하는가?
- 아니면 자동으로 축약 URL이 만들어지는가?
- 클릭에 관한 통계 정보를 기록할 필요가 있는가?
- 한번 설정된 URL은 영구 보존인가? 삭제는 되는가?
- 주요 특징이나 사용 사례를 나열해 볼 것.

#### 2단계: 합리적인 가정을 만들라

- 하루 백만개의 URL을 생성해야 한다는 가정으로 데이터 양의 계산 등이 가능.
- 메모리에 제약이 없다는 등의 비합리적 가정은 X.
- URL이 생기면 즉시 사용 가능해야 할 것. 한편, 10분 정도 오래된 데이터는 허용 가능할 것.

#### 3단계: 중요한 부분을 먼저 그리라

- 시스템이 어떻게 생겼는지 그려라.
- 처음부터 마지막까지 어떻게 흘러가는지 보라.
- 현재 단계에서 확장성 고민은 X. 최대한 단순하고 명백하게 접근.

#### 4단계: 핵심 문제점을 찾으라

- 기본적 설계를 마친 후 어느 지점이 병목일지 찾기.
- 시스템이 풀어야 할 주된 문제점은 무엇인가.
- 어떤 URL은 드물게 사용되는 반면, 어떤 URL은 사용량이 갑자기 치솟을 수 있음.
- 인기 있는 URL이 끊임없이 데이터베이스를 치고 들어오는 것을 원하지는 않을 것.

#### 5단계: 핵심 문제점을 해결할 수 있도록 다시 설계하라

- 4단계 극복을 위한 해결책 고민.
- 시스템 전체를 뒤엎을 수도, 일부만 수정할 수도.

## 정렬과 탐색

- Person 객체로 이뤄진 크기가 큰 배열이 있음.
- 이 배열을 나이순으로 정렬해야 한다면?
- 우선, 배열의 크기가 큼을 알 수 있음.
- 또한, 나이순 정렬이므로 값의 범위가 좁다는 것도.
- 따라서, 버킷 정렬<sup>bucket sort</sup> 혹은 <sup>radix sort</sup>가 적합.
- 책에서는 [버블](https://en.wikipedia.org/wiki/Bubble_sort), [선택](https://en.wikipedia.org/wiki/Selection_sort), [병합](https://en.wikipedia.org/wiki/Merge_sort), [퀵](https://en.wikipedia.org/wiki/Quicksort), [기수](https://en.wikipedia.org/wiki/Radix_sort)를 소개.
- 선택은 최소값 찾아서 앞으로 보내기.
- 버블은 원소를 최대한 맨 뒤로 보내기.
- 퀵은 pivot.
- 탐색에 대한 내용은 기록 생략.
- 기수는 낮은 자리수부터 비교해 정렬(자릿수 고정).

| 종류 | Big-θ | Big-Ω | Big-O | Memory |
| --- | ----- | ----- | ----- | ------ |
| 선택 | n^2 | n^2 | n^2 | 1 |
| 버블 | n | n^2 | n^2 | 1 |
| 병합 | n log n | n log n | n log n | n |
| 퀵 | n log n | n log n | n^2 | log n |
| 기수 | - | kn | kn | - |

## 테스팅

> 면접에 테스팅 관련 질문이 주어졌을 때의 대응 얘기이긴 하지만, 개발할 때 있어서의 중요한 접근법이기도 하여 기록.

테스트 관련 질문을 받으면, 단순히 테스트 케이스를 적절히 만들어내야 하는 것처럼 보임. 물론 중요한 요소. 하지만 다음의 것들도 매우 중요.

- 큰 그림에 대한 이해
  - 소프트웨어가 지향하는 바는?
  - TC 간의 우선순위는?
  - 쇼핑몰 같은 경우 상품 이미지가 뜨는 것이 매우 중요하긴 하나, 배송 목록에 제품이 잘 추가되는지, 대금이 이중 청구 되지는 않았는지도 매우 중요한 요소.
- 소프트웨어가 보다 더 큰 생태계의 일부로 어떻게 귀속되는지를 이해
- 조직화
  - 떠오르는 대로 TC를 말하는 것은 X.
  - 카메라에 대한 TC를 만든다면, 사진 촬영, 이미지 관리, 설정 등의 시스템 범주화 후 TC를 만들어 가기.

### 객체 테스트하기

"펜을 테스트하라" 또는 "클립을 테스트하려면 어떻게 해야 하는가"와 같은 질문을 받았다고 가정.

1. 사용자는 누구인가? 사용 목적은 무엇인가?
  - 이런 질문을 던지면 의외의 대답이 나오기도.
  - "선생님들이 종이를 묶어두기 위해 사용한다"와 같은.
2. 어떤 유스케이스가 있나?
  - 구체적으로 어떤 사용사례들이 있는지 확인.
  - 여기서는 단순히 뭔가를 묶어두는 용도만 있다고 가정.
3. 한계 조건은?
  - 한 번에 몇 장의 종이를 묶어두려 하는가?
  - 한 번에 30장을 묶어둔다고 하면 약간 구부려져도 되는가?
  - 아주 추운 곳에서도 정상적으로 동작해야 하나?
4. 스트레스 조건과 장애 조건은?
  - 문제가 없는 제품은 없음.
  - 장애가 발생하는 조건을 알아두는 것도 필요.
  - 예컨대, 세탁기에 한 번에 45벌 이상의 옷을 넣으면, 물이 공급되지 않을 수 있음.
5. 테스트는 어떻게 수행할 것인가?
  - 의자를 5년 동안 아무 문제 없이 사용하고 싶다고 해보자.
  - 집에 의자를 가져가서 5년 동안 앉아볼 것인가?
  - 대신, '5년 동안 문제 없이'를 정의해야 함.
  - 100번을 앉아도 문제 없으면 정상으로 간주해도 되나?

### 소프트웨어 테스트

'객체 테스트하기'와 유사. 다만, 성능 테스트에 대한 세부사항이 많음. 그 외에 인상깊었던 부분만 기록.

- 모든 걸 자동화하면 좋겠지만, 포르노 컨텐츠를 걸러내는 것과 같이 사람의 인지가 필요한 부분도 있음.
- 블랙박스 테스팅이 필요한지, 화이트박스 테스팅이 필요한지를 초반에 논의하고 접근.

### 문제 해결에 관한 문제

크롬 브라우저가 실행되자마자 죽는다는 리포트를 받았다면 어떻게 접근할 것인가?

1. 시나리오를 이해
  - 상황을 가능한 한 정확히 이해할 수 있기 위해 질문을 던짐.
  - 사용자는 얼마나 오랫동안 이 문제를 겪었나?
  - 브라우저 버전은? 운영체제 버전은?
  - 이 문제가 항상 똑같이 발생하는가? 발생 빈도는? 언제 발생하나?
  - 오류 보고서가 표시되는가?
2. 문제를 쪼개라.
  - 문제를 테스트 가능한 단위로 분할.
  - 윈도우 시작 메뉴로 간다.
  - 크롬 아이콘을 클릭.
  - 브라우저가 시작됨.
  - 브라우저가 설정을 읽어 들임.
  - 브라우저가 홈페이지에 HTTP 요청을 날림.
  - 브라우저가 HTTP 응답을 받음.
  - 브라우저가 웹 페이지를 파싱.
  - 브라우저가 웹 페이지를 화면에 표시.

## 스레드와 락

일반적인 이해는 필요한 내용.

### 자바의 스레드

- 자바의 모든 스레드는 `java.lang.Thread` 클래스 객체에 의해 생성되고 제어됨.
- `main()` 실행을 위해 하나의 사용자 스레드(user thread)가 자동으로 만들어 지는데, 이 스레드를 주 스레드(main thread)라고 부름.
- 스레드 생성하는 방법으로는 `Runnable` 인터페이스 구현과 `Thread` 클래스 상속이 있음.
- 다중 상속 지원 안 되는 점을 고려할 때 `Runnable`이 선호됨.

### 동기화와 락

- 스레드 간 메모리 동기화가 이득이 되기도 하지만 공유된 자원을 동시에 수정하려고 할 때 문제가 되기도.
- 이를 제어하기 위해 동기화(synchronization) 방법을 제공.
- 동기화된 메서드.
  - 메서드 시그니처에 `synchronized` 키워드 선언.
  - 인스턴스 메서드에 선언: 같은 객체의 메서드 동시 접근을 차단.
  - 스태틱 메서드에 선언: 객체 구분 없이 동시 접근 차단.
- 동기화 된 블럭
  - `synchronized(this) { // 블라 블라 }`
  - 메서드 단위가 아닌 특정 블럭 단위로 적용.
- 락(lock)
  - 좀 더 세밀한 제어가 가능.
  - [여기](https://winterbe.com/posts/2015/04/30/java8-concurrency-tutorial-synchronized-locks-examples/) 보면 `Semaphore`, `StampedLock`, `ReadWriteLock`, `ReentrantLock` 등 여러 도구들이 존재.

### 교착상태와 교착상태 방지

교착상태 발생의 4가지 조건을 이야기.

1. 상호 배제(mutual exclusion)
2. 들고 기다리기(hold and wait)
3. 선취(preemption) 불가능
4. 대기 상태의 사이클(circular wait)

이 중 1개의 조건만 제거해도 교착상태 방지. 주로 4번을 막는 데 초점이 맞춰져 있고, 나머지는 회피가 어려움.

# 해법

## 배열과 문자열 해법

### 중복이 없는가

> 문자열이 주어졌을 때, 이 문자열에 같은 문자가 중복되어 등장하는지 확인.

- 면접관에게 문자열이 ASCII인지 유니코드인지 물어봐야 한다고 함.
- 하지만 잘못된 질문이라 생각함. 자바에서는 `Character` 클래스가 한 글자를 UTF-16으로 처리하기 때문.
- 이모지 같이 16바이트가 넘는 글자를 허용할 것인지 질문하는 게 타당.
- 그런데 내용을 조금만 더 읽다 보니 ASCII 문자열인지를 확인하는 것이 이해가 됨. 아래 문장 때문.

> 또한 문자열의 길이가 문자 집합의 크기보다 클 경우 바로 false를 반환해도 된다. 결국 256가지 문자를 한 번씩만 사용해서 길이가 280인 문자열을 만들 수는 없으니까 말이다.

- ASCII 문자열만 다룬다고 하면 구현이 여러가지로 간편해지기 때문에 확인하는 것.
- 그렇게 구현된 코드는 아래와 같음.

```java
boolean isUniqueChars(String str) {
  if (str.length() > 128) return false;
  boolean[] char_set = new boolean[128];
  for (int i = 0; i < str.length(); i++>) {
    int val = str.charAt(i);
    if (char_set[val]) {
      return false;
    }
    char_set[val] = true;
  }
  return true;
}
```

- 시간 복잡도는 O(n). 공간 복잡도는 O(1).
- 하지만 문자의 갯수가 최대 128개를 넘지 않을테니 O(1)이라고 봐도.
- 비트 벡터를 사용하면 필요한 공간을 1/8로 줄일 수 있음.
- 공간이 1/8인 까닭은 아마도 boolean array의 각 엘리먼트의 크기가 8비트인데 반해,
- 비트 벡터는 1비트만 쓰면 되기 때문인 것으로 이해됨.
- 하지만 실제 책의 예시를 보면 `BitSet`이 아닌 int를 비트 벡터로 사용하고 있음.
- 기존 128바이트에서 4바이트로 줄어들었으니 1/32의 공간만 사용된 것.
- 예시 코드는 아래 참고. 여기서는 문자열이 소문자 a부터 z까지로 구성된다고 가정.

```java
boolean isUniqueChars(String str) {
  int checker = 0;
  for (int i = 0; i < str.length(); i++) {
    int val = str.charAt(i) - 'a';
    if ((checker & (1 << val)) > 0) {
      return false;
    }
    checker |= (1 << val);
  }
  return true;
}
```

- 자료구조를 추가로 사용할 수 없다면 아래와 같이 할 수도 있음.
- 문자열 내의 각 문자를 다른 모든 문자와 비교. O(n^2) 수행시간 소요. 공간 복잡도는 O(1).
- 입력 문자열을 수정해도 된다면, O(n log n) 시간에 문자열을 정렬한 뒤 문자열을 처음부터 흝어 나가면서 인접한 문자가 동일한지 검사해 볼 수도 있음. 다만, 많은 정렬 알고리즘이 공간을 추가로 쓴다는 사실에 유의.

### URLify

> 문자열에 들어 있는 모든 공백을 '%20'으로 바꾸는 메서드를 작성하라. (생략)

- 문자열 조작에 널리 쓰이는 방법 중 하나는 문자열을 뒤에서부터 거꾸로 편집해 나가는 것.
- 이렇게 하면 덮어쓸 걱정을 하지 않고 문자들을 바꿔나갈 수 있음.
- 개인적으로는 배열을 하나 더 만들어 치환하면 될 것으로 생각했음. 배열 하나를 더 사용해야 한다는 것이 상대적 단점.
- 한편 char 배열이 아닌 String을 사용하는 방법도 얘기. 다만 매번 복사를 해야 한다는 단점. 하지만 한 번만 루프를 도는 것으로 결과 반환이 가능.
- 책에 나온 코드는 다소 이상한 점이 있어 [geeksforgeeks 코드](https://www.geeksforgeeks.org/urlify-given-string-replace-spaces/) 참고.

### 회문 순열(palindrome permutation)

비트 벡터에서 한 개만 1인지 여부를 판단하는 게 재밌어서 기록함. 회문이란 앞으로 읽으나 뒤로 읽으나 같은 단어나 구절을 의미하며, 회문 '순열'인지 판단하는 것이므로 문자열 재배열을 통해 회문이 가능한지를 판단해야 함. 접근법의 다양성이나 설명의 상세함 측면에서 [여기 LeetCode 글](https://leetcode.com/articles/palindrome-permutation/)이 더 좋음.

- Brute Force: O(128 * N) / O(1)
- Using HashMap: O(N) / O(N). `HashMap<Character, Integer>`을 이용해 두 번의 루프로 해결.
- Using Array: O(N) / O(128). 각 문자들이 ASCII 문자라고 가정하고, `int[128]`을 이용해 2번의 접근법을 취함. 2번에 비해 적은 공간 사용. 코드로 보면 아래와 같음.

```java
public class Solution {
    public boolean canPermutePalindrome(String s) {
        int[] map = new int[128];
        for (int i = 0; i < s.length(); i++) {
            map[s.charAt(i)]++;
        }
        int count = 0;
        for (int key = 0; key < map.length && count <= 1; key++) {
            count += map[key] % 2;
        }
        return count <= 1;
    }
}
```

- Single Pass: O(N) / O(128). 3번의 방식을 취하되 루프를 한 번 도는 것으로 해결. 그리고 루프를 돌며 `count` 정수형 변수를 이용해 홀수 존재여부를 판단.

```java
public class Solution {
    public boolean canPermutePalindrome(String s) {
        int[] map = new int[128];
        int count = 0;
        for (int i = 0; i < s.length(); i++) {
            map[s.charAt(i)]++;
            if (map[s.charAt(i)] % 2 == 0)
                count--;
            else
                count++;
        }
        return count <= 1;
    }
}
```

- 책에는 나오지 않은 Using Set. O(n) / O(1). 4번의 접근법과 유사하나 `count` 대신 `HashSet`을 사용. 코드 가독성이 좋아짐.

```java
public class Solution {
    public boolean canPermutePalindrome(String s) {
        Set < Character > set = new HashSet < > ();
        for (int i = 0; i < s.length(); i++) {
            if (!set.add(s.charAt(i)))
                set.remove(s.charAt(i));
        }
        return set.size() <= 1;
    }
}
```

- Bit Vector. LeetCode 글에는 나오지 않지만, 책에서는 언급된, 비트 벡터에 1이 딱 한 개만 있는지 판별하는 것이 흥미로운 방법. `int`를 비트 벡터로 쓰고 있는데, 이 숫자에서 -1을 한 것과 이 숫자를 & 연산하면 0이 나와야 함.

```java
public class Solution {
  public boolean isPermutationOfPalindrome(String phrase) {
    int bitVector = createBitVector(phrase);
    return bitVector == 0 || checkExactlyOneBitSet(bitVector);
  }

  /* Create a bit vector for the string. For each letter with value i, toggle the ith bit. */
  private int createBitVector(String phrase) {
    int bitVector = 0;
    for (char c : phrase.toCharArray()) {
      int x = getCharNumber(c);
      bitVector = toggle(bitVector, x);
    }
    return bitVector;
  }

  private int toggle(int bitVector, int index) {
    if (index < 0)	return bitVector;
    int mask = 1 << index;
    if((bitVector & mask) == 0)
      bitVector |= mask;
    else
      bitVector &= ~mask;
    return bitVector;
  }

  /* Check that exactly one bit is set by subtracting one from the integer and ANDing it with the original integer */
  private boolean checkExactlyOneBitSet(int bitVector) {
    return (bitVector & (bitVector-1)) == 0;
  }
}
```

### 문자열 압축

문제 설명은 [여기 LeetCode](https://leetcode.com/problems/string-compression/) 참고. 첫 번째 풀이 코드는 아래와 같음.

```java
String compressBad(String str) {
  String compressedString = "";
  int countConsecutive = 0;
  
  for (int i = 0; i < str.length(); i++) {
    countConsecutive++;

    if (i + 1 >= str.length() || str.charAt(i) != str.charAt(i + 1)) {
      compressedString += "" + str.charAt(i) + countConsecutive;
      countConsecutive = 0;
    }
  }

  return compressedString.length() < str.length() ? compressedString : str;
}
```

- 문자열의 길이가 p이고, 연속된 문자의 종류가 k라고 하면, O(p + k^2)의 수행시간 소요.
- k^2인 까닭은 문자열 concatenation 연산 때문.
- 이를 `StringBuiler`나 배열을 이용한 방식으로 바꿔볼 수 있음.
- 한편, 이런 문자열 처리를 하지 않고, 압축하는 게 더 좋을지를 미리 판단해 볼 수도 있음.

```java
String compress(String str) {
  if (countCompression(str) >= str.length()) return str;
  // 생략
}

int countCompression(String str) {
  // 구현 생략 (문자열 조작하는 코드와 유사)
}
```

### 0행렬

> M x N 행렬의 한 원소가 0일 경우, 해당 원소가 속한 행과 열의 모든 원소를 0으로 설정하는 알고리즘을 작성하라.

- 행렬을 순회해 가면서 값이 0인 셀을 발견하면, 바로 그 열과 행을 모두 0으로 만든다 -> X
- 안 되는 까닭은, 0인 셀 발견 시, 0으로 바뀐 건지 아니면 원래 0인지 알 수 없기 때문.
- 그래서 0의 위치를 기록하기 위한 행렬 하나를 더 둘 수도 있음. 하지만 공간복잡도가 O(MN)이 됨.
- 하지만 0의 위치가 정확히 필요한 것은 아님.
- 어떤 행의 값들이, 어떤 열의 값들이 0인지만 알면 0으로 바꾸는 작업이 가능.
- 따라서 `new boolean[matrix.length]`와 `new boolean[matrix[0].legnth]` 배열만으로 충분.
- 이 배열을 비트 벡터로 바꿔볼 수는 있겠으나 공간 복잡도는 여전히 O(N).
- 좀 더 나아가서, 주어진 행렬의 첫 번째 열과 행 공간을 0의 행과 열 표기 공간으로 사용할 수도.

```java
void setZeros(int[][] matrix) {
  boolean rowHasZero = false;
  boolean colHasZero = false;
  
  // 첫 번재 행에 0이 있는지 확인
  for (int j = 0; j < matrix[0].length; j++) {
    if (matrix[0][j] == 0) {
      rowHasZero = true;
      break;
    }
  }

  // 첫 번재 열에 0이 있는지 확인
  for (int i = 0; i < matrix.length; i++) {
    if (matrix[i][0] == 0) {
      colHasZero = true;
      break;
    }
  }

  // 나머지 배열에 0이 있는지 확인
  for (int i = 1; i < matrix.legnth; i++) {
    for (int j = 1; j < matrix[0].length; j++) {
      if (matrix[i][j] == 0) {
        matrix[i][0] = 0;
        matrix[0][j] = 0;
      }
    }
  }

  // 첫 번째 열을 이용해서 행을 0으로 바꾼다.
  for (int i = 1; i < matrix.length; i++>) {
    if (matrix[i][0] == 0) {
      nullifyRow(matrix, i);
    }
  }

  // 첫 번째 행을 이용해서 열을 0으로 바꾼다.
  for (int j = 1; j < matrix[0].length; j++>) {
    if (matrix[0][j] == 0) {
      nullifyColumn(matrix, j);
    }
  }

  // 첫 번째 행을 0으로 바꾼다.
  if (rowHasZero) {
    nullifyRow(matrix, 0);
  }

  // 첫 번째 열을 0으로 바꾼다.
  if (colHasZero) {
    nullifyColumn(matrix, 0);
  }
}
```

### 문자열 회전

> 한 단어가 주어진 문자열 안에 포함되어 있는지를 판별하는 isSubstring 메서드가 있다고 가정. 이를 한 번만 호출해서, s1과 s2 문자열이 서로 회전 관계에 있는지를 판별. 예컨대, waterbottle은 erbottlewat를 회전시킨 결과.

- s1 = xy = waterbottle
- x = wat
- y = erbottle
- s2 = yx = erbottlewat
- xyxy = waterbottlewaterbottle
- 회전 지점이 어디인지 상관 없음.
- s1s1과 s2를 이용한 isSubstring으로 회전 문자열 여부 판단 가능.
