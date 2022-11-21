# Counting Sort

- [LeetCode Counting Sort](https://leetcode.com/explore/learn/card/sorting/695/non-comparison-based-sorts/4437/) 정리.
- non-comparison based sort에서 가장 간단한 빌딩 블럭 중 하나.

## 간단 버전

- `A = [1,5,0,3,6,4,2]`라는 배열이 있다고 해보자.
- 이 배열의 특수한 속성은, 최대값이 6이고, 최소값이 0이며, 각 값은 배열에서 정확히 한 번만 존재한다는 것.
- 따라서, 아주 간단한 알고리즘으로 한 번의 순회로 정렬할 수 있음.
- O(N) 시간과 O(N)의 추가 공간만 필요.

```
1. Initialize an array output of size 7
2. For every element A[i]
    - output[A[i]] = A[i]
```

- 이 알고리즘 동작을 보장하는 속성은 다음의 3가지.
- 먼저, 배열의 각 원소가 0과 N-1 사이(0 <= A[i] <= N-1).
- 다음으로, 어떤 원소도 중복되지 않음.
- 마지막으로, 배열 크기는 N(모든 원소가 정확히 1번씩 나타나는 것).
- 이 알고리즘이 counting sort의 가장 단순화된 버전임.

## 확장 버전

- counting sort는 미리 정의된 범위의 "키들"을 사용함.
- 위 예시에서는 키들이 인덱스에 1:1 대응했음.
- 여기서 좀 더 확장이 가능함.
- 먼저, 고유하지 않은 키들도 다룰 수 있음.
- 다음으로, 연속적이지 않은 키들도 다룰 수 있음.
- 마지막으로, 숫자가 키가 아니더라도, 미리 정의된 집합의 값들이라면(알파벳처럼) 적용 가능.

## 카운팅 아이디어

- 배열에서 가능한 최소값이 0이고 최대값으로 K가 가능할 때,
- 1번과 2번 단계를 발기 위해 필요한 것은,
- 0과 K 사이의 값의 빈도를 추적하는 것.
- 입력 배열이 `A = [5,4,5,5,1,1,3]`이라고 해보자.
- 앞서 했던 것처럼, max(A) + 1 크기의 새로운 배열 `counts`를 생성.
- 참고로, 숫자는 0부터 시작하므로 max(A) + 1 크기로 만드는 것.
- 이 배열의 각 인덱스 i에는 A 원소에서 i값이 출현하는 빈도를 저장.
- 지금 예시에서는 그럼 `[0,2,0,1,1,3,0]`이 됨.
- 그리고 이 배열을 활용하면, 원래 배열에서의 각 원소 시작 인덱스를 알 수 있음.
- `counts` 배열에서 왼쪽부터 값을 축적해 나가면 됨(인덱스 i는 앞선 인덱스들의 총합).
- `startingIndices = [0,0,2,2,3,4,7]`
- 결국, 원래 입력 원소들의 위치를 나타내는 배열을 만든 것.

## 구현

전체 구현은 [여기에 작성함](https://github.com/codehumane/algorithm/commit/bbb16db13f1f254b115ccd4c9c4caf7c77bd5512).

```java
public void sort(int[] list) {
    if (list.length == 0) return;

    var counts = counts(list);
    accumulate(counts);
    var sorted = sort(list, counts);
    log.debug("before: {}, after: {}", list, sorted);
    System.arraycopy(sorted, 0, list, 0, list.length);
}

private int[] counts(int[] list) {
    var max = Arrays.stream(list).max().orElseThrow();
    var counts = new int[max + 1];

    for (int num : list) {
        counts[num]++;
    }

    return counts;
}

private void accumulate(int[] counts) {
    var startIdx = 0;

    for (int i = 0; i < counts.length; i++) {
        var count = counts[i];
        counts[i] = startIdx;
        startIdx += count;
    }
}

private int[] sort(int[] list, int[] counts) {
    var sorted = new int[list.length];

    for (int num : list) {
        sorted[counts[num]++] = num;
    }

    return sorted;
}
```

## 복잡도

- O(N + K)
- N은 입력 배열 크기, K는 배열 원소의 최대값.
- 공간복잡도 역시 O(N + K)
