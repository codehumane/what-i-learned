
# LeetCode Radix Sort

https://leetcode.com/explore/learn/card/sorting/695/non-comparison-based-sorts/4438/

## Counting sort와의 비교

[Counting sort](https://leetcode.com/explore/learn/card/sorting/695/non-comparison-based-sorts/4437/)와의 비교로 Radix sort 소개.

- Counting sort는 사이즈 제약이 없는 알파벳 문자열을 다루기 어려움.
- 게다가, 최대값이 매우 큰 경우, 메모리 오버헤드로 성능 저하.
- Radix sort는 Counting sort의 확장으로, 이런 문제들을 해결함.
- 문자열이나 정수의 집합에 대해 잘 동작함(최대값이 큰 경우에도).
- 여기서는 Radis sort의 변이들 중 LSD(Least Significant Digit)를 다룸.

## LSD Radix Sort

### 간단 소개

- LSD에서는 정수의 가장 작은 자릿수부터 시작해서,
- 각 자릿수에 대해 counting sort를 수행.
- 문자열이라면 가장 우측 문자부터 수행.
- counting sort는 stable 정렬이므로,
- 동률이면 원소들의 상대적 순서 유지됨.

### 예제

`A = [256, 336, 736, 443, 831, 907]`를 예시로 설명.

1. 마지막 자릿수를 기준 counting sort: [831, 443, 256, 336, 736, 907]
2. 두 번째 자릿수에 대해 동일한 수행: [907, 831, 336, 736, 443, 256]
3. 첫 번째 자릿수에 대해 동일한 수행: [256, 336, 443, 736, 831, 907]

### 패딩

- 자릿수가 다른 수를 비교할 땐, 자릿수가 적은 수에는 0을 채우면 됨.
- 길이가 다른 문자열이라면, 앞에 최소 값으로 간주되는 문자를 채우면 됨.

### full LSD radis sort alrogirhtm for integers

1. 가장 큰 수의 자릿수를 구하기. 이를 W라고 하자.
2. 각 정수에 대해, 1부터 W까지의 자릿수를, 작은 자릿수부터 순회.
3. 각 자릿수 그룹에 대해 counting sort.

### 구현

[여기](https://www.geeksforgeeks.org/radix-sort/) 코드 참고해서 [직접 작성](https://github.com/codehumane/algorithm/commit/727e7e7a744e2d6257646f6e725aa3c576de1176).

```java
@Slf4j
public class RadixSort {

    public void sort(int[] input) {
        if (input.length < 2) return;

        var digit = 1;
        var max = max(input);

        while (max >= digit) {
            countingSort(input, digit);
            digit *= 10;
        }
    }

    private int max(int[] input) {
        var max = input[0];
        for (int i = 1; i < input.length; i++) {
            max = Math.max(max, input[i]);
        }

        return max;
    }

    // e.g. 256 336 736 443 831 907 (1의 자리)
    private void countingSort(int[] input, int digit) {
        var temp = new int[input.length];
        var count = new int[10];

        // 0 1 0 1 0 0 3 1 0 0
        for (int num : input) {
            count[digitNumber(num, digit)]++;
        }

        // 0 1 1 2 2 2 5 6 6 6
        for (int i = 1; i < count.length; i++) {
            count[i] += count[i - 1];
        }

        // 0 1 1 2 2 2 5 6 6 6
        // 0 1 1 2 2 2 5 5 6 6
        // 0 0 1 2 2 2 5 5 6 6
        // 0 0 1 1 2 2 5 5 6 6
        // 0 0 1 1 2 2 4 5 6 6
        // 0 0 1 1 2 2 3 5 6 6
        // 0 0 1 1 2 2 2 5 6 6
        for (int i = input.length - 1; i >= 0; i--) {
            var num = input[i];
            var digitNum = digitNumber(num, digit);
            temp[count[digitNum] - 1] = num;
            count[digitNum]--;
        }

        // 단순 복사
        log.debug("digit: {}, before: {}, after: {}", digit, input, temp);
        System.arraycopy(temp, 0, input, 0, input.length);
    }

    private int digitNumber(int num, int digit) {
        return (num / digit) % 10;
    }

}
```

### 실행 시간

일단, 실행 시간 계산을 위해서는 몇 가지 파라미터가 필요.

- W: 배열 원소의 최대값 자릿수
- N: 배열 크기
- K: 알파벳 종류 혹은 정수의 경우는 10

그래서 복잡도는 아래와 같음.

- 시간 복잡도: O(W(N + K))
- 공간 복잡도: O(N + K)
    - 이는 counting sort와 동일

### 장단점

- 장점은 W와 K가 너무 크지 않은 정수나 문자열에 대해 매우 빠르다는 것.
- 그리고 시간 소요가 선형적<sup>linear</sup>.
- 또한 stable sort(동률에 대해서는 처음 주어진 순서가 유지되는 것). 
- 단점은 메모리 오버헤드가 조금 있다는 것.
- 만약 N 그리고/또는 K가 크다면 다른 정렬에 비해 성능 감소.
- 특성상 모든 자릿수를 살펴봐야 한다는 것도 단점이 될 수도.
