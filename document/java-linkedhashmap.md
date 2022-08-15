# Java LinkedHashMap

[LinkedHahsMap javadoc](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html) 정리.

## 요약

```java
public class LinkedHashMap<K,V>
extends HashMap<K,V>
implements Map<K,V>
```

- `Map` 인터페이스에 대한,
- hash table과 linked list 구현.

## doubly-linked list

- 모든 엔트리를 doubly-linked list로 관리함.
- 이것이 `HashMap`과의 차이.
- 내부에 선언된 `Entry` 함께 참고.

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

## insertion-order

- 이터레이션 순서가 예측 가능함.
- 여기에는 insertion-order 사용.
- 맵에 키가 삽입된 순서대로 정렬된다는 것.
- 키가 이미 맵에 존재할 때의 재삽입은 순서에 영향 X.

## HashMap, TreeMap과의 비교

- `HashMap`와 달리 순서를 보장해 주며,
- `TreeMap`보다는 저렴한 비용이 듦.
- 아래와 같이 원본과 동일한 순서로 맵을 복제할 수 있음.
- 이 때 원본 맵의 구현이 무엇인지는 상관 없음.
- 복제 시점의 순서를 보관할 필요가 있을 때 유용.

```java
void foo(Map m) {
    Map copy = new LinkedHashMap(m);
    ...
}
```

- [StackOverflow 비교표 답변](https://stackoverflow.com/a/17708526)도 함께 참고.

| Property                         | HashMap                                                      | TreeMap                                                      | LinkedHashMap                                                |
| :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Iteration Order                  | no guaranteed order, will remain constant over time          | sorted according to the natural ordering                     | insertion-order                                              |
| Get / put / remove / containsKey | O(1)                                                         | O(log(n))                                                    | O(1)                                                         |
| Interfaces                       | Map                                                          | NavigableMap, Map, SortedMap                                 | Map                                                          |
| Null values/keys                 | allowed                                                      | only values                                                  | allowed                                                      |
| Fail-fast behavior               | Fail-fast behavior of an iterator cannot be guaranteed, impossible to make any hard guarantees in the presence of unsynchronized concurrent modification | Fail-fast behavior of an iterator cannot be guaranteed, impossible to make any hard guarantees in the presence of unsynchronized concurrent modification | Fail-fast behavior of an iterator cannot be guaranteed, impossible to make any hard guarantees in the presence of unsynchronized concurrent modification |
| Implementation                   | buckets                                                      | Red-Black Tree                                               | double-linked buckets                                        |
| Is synchronized                  | implementation is not synchronized                           | implementation is not synchronized                           | implementation is not synchronized                           |

## access-order

- linked hash map을 생성하는 [특수한 생성자](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html#LinkedHashMap-int-float-boolean-)도 제공됨.
- 여기서는 원소들의 순서를 access-order로 지정할 수 있음.
- 가장 오래 전에 접근한 것부터 최근에 접근한 순서대로.
- 이는 LRU 캐시를 만드는 데 적합함.
- 참고로, 원소 접근이 일어나는 메서드는 아래의 것들.
- `put`, `putIfAbsent`, `get`, `getOrDefault`, `compute`, `computeIfAbsent`, `computeIfPresent`, `merge`
- `replace`는 대체할 값이 존재하는 경우에만 접근이 일어남.
- `putAll`은 주어진 map의 키-값 순서대로 하나씩 엔트리 접근을 만들어 냄.
- 그 외의 메서드들은 엔트리 접근을 만들지 않음.
- 특히 collection-views에 대한 연산들은 이터레이션 순서에 아무 영향을 주지 않음.

## removeEldestEntry

- protected로 제공하는 [removeEldestEntry(Map.Entry)](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html#removeEldestEntry-java.util.Map.Entry-)가 있음.
- 이 메서드는 `put`, `putAll`에서 새로운 엔트리가 맵에 삽입되고 난 뒤 호출됨.
- 이는 맵이 캐시 역할을 할 때 유용.
- 오래된 엔트리를 제거함으로써 메모리 사용을 줄여주기 때문.
- 아래와 같은 식으로 이 메서드를 오버라이드해서 오래된 데이터 제거 정책 생성.

```java
protected boolean removeEldestEntry(Map.Entry eldest) {
    return size() > MAX_ENTRIES;
}
```

- 이 안에서 맵을 수정하면 무조건 안 될 것이라 생각했는데,
- 문서에는 예외적으로 그럴 수 있다고 하며 다만 이 경우엔 `false`를 반환하라고 함.
- 신기함.

> This method typically does not modify the map in any way, instead allowing the map to modify itself as directed by its return value. It is permitted for this method to modify the map directly, but if it does so, it must return false (indicating that the map should not attempt any further modification). The effects of returning true after modifying the map from within this method are unspecified.

## HashMap과의 성능 비교

- `HashMap`처럼 `add`, `contains`, `remove` 같은 기본 연산자들에 대해 상수 시간 성능을 보임.
- 물론, 해시 함수가 원소들을 버켓들에 고르게 분포시킨다는 가정 하에.
- 성능은 `HashMap`에 비해서는 좀 아래에 있음.
- linked list를 유지해야 한다는 추가적 비용 때문.
- 다만, collection-views에 대한 이터레이션 시간은 capacity와 관계 없이 map의 size에 비례함.
- `HashMap`의 이터레이션 시간은 capacity에 비례하므로 이보다는 좀 더 비쌈.

## 성능 파라미터

- 성능에 관한 2가지 파라미터가 있음.
- initial capacity와 load factor가 그것.
- [이들은 HashMap에도 정확히 정의되어 있음](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html).

> The capacity is the number of buckets in the hash table, and the initial capacity is simply the capacity at the time the hash table is created. The load factor is a measure of how full the hash table is allowed to get before its capacity is automatically increased. When the number of entries in the hash table exceeds the product of the load factor and the current capacity, the hash table is rehashed (that is, internal data structures are rebuilt) so that the hash table has approximately twice the number of buckets.

- 그리고 load factor로 .75를 잡는 것이 일반적으로 좋은 트레이드 오프 지점이라고 함.

> As a general rule, the default load factor (.75) offers a good tradeoff between time and space costs. Higher values decrease the space overhead but increase the lookup cost (reflected in most of the operations of the HashMap class, including get and put). The expected number of entries in the map and its load factor should be taken into account when setting its initial capacity, so as to minimize the number of rehash operations. If the initial capacity is greater than the maximum number of entries divided by the load factor, no rehash operations will ever occur.

- 다만, initial capacity로 과도하게 높은 값을 고르는 것이,
- HashMap에서의 불이익 보다는 적음.
- 이터레이션 시간이 capacity에 영향을 받지 않기 때문.

## structural modification

- 1개 이상의 매핑을 추가하거나 삭제할 때 구조적 수정 일어남.
- 혹은 access-orderd linked hash map에서의 이터레이션 순서에 영향을 주는 작업 시.
- 위에서 언급된 것처럼 replace 같은 연산자는 구조적 수정 안 일어날 수도 있음.

나머지 내용은 다른 자료 구조에서도 자주 나오는 일반적인 내용들이라 기록 생략.
