# 정렬과 순서 통계량

- 입력: 숫자 n개로 이루어진 수열
- 출력: 입력 수열을 오름차순으로 재배치한 수열

### 정렬 알고리즘
- 레코드: 정렬할 대상 (키 + 부속 데이터)
- 정렬 알고리즘: 레코드에 상관 없이 정렬할 순서만을 결정


### 힙 정렬

#### 힙
- 완전 이진 트리 (가장 낮은 레벨은 왼쪽부터 차례대로 채우고, 나머진 가득 차있다)
- 힙이 높이: 하향 경로 중 가장 긴 것의 간선 개수
- 트리를 배열에 넣을 경우,
	- PARENT(i) = i / 2의 내림값 (i >> 1)
	- LEFT(i) = 2i (i << 1)
	- RIGHT(i) = 2i + 1 (i << 1 + 1)
- **최대 힙일 경우, 임의의 노드 값은 부모보다 클 수 없다.**
- 기본 연산: O(lg n)
	- MAX-HEAPIFY: 최대 힙 특성을 유지하는 프로시저. O(lg n)
	- BUILD-MAX-HEAP: 정렬되지 않은 입력 배열로부터 최대 힙을 만든다. O(n)
	- HEAPSORT: 배열 내부를 정렬한다. O(n lg n)
	- MAX-HEAP-INSERT, HEAP-EXTRACT-MAX, HEAP-INCREASE-KEY, HEAP-MAXIMUM: 우선순위 큐 구현에 필요한 프로시저. 모두 O(lg n)

1. MAX-HEAPIFY
- 임의의 노드 값은 부모보다 클 수 없다.
- 부모가 자식보다 작을 수 있다.
- **값을 내려보냄**으로써 최대 힙 특성을 지킬 수 있도록 한다.

MAX-HEAPIFY(A, i) = A[i] 값이 자식보다 작으면 한 레벨 내려보내는 프로시저
```
l = LEFT(i)
r = RIGHT(i)
왼쪽 자식 보다 작으면
if l <= A.heap-size and A[l] > A[i]
	largest = l
else largest = i
오른쪽 자식 보다 작으면
if r <= A.heap.size and A[r] > A[largest]
	largest = r
if largest != i
	배열 swap
	exchange A[i] with A[largest]
	방금 바꾼 값도 자식보다 작진 않은지 검사하기 위해 재귀호출해 i의 서브트리를 모두 검사한다
	MAX-HEAPIFY(A, largest)
```

- 수행시간: T(n) <= T(2n / 3) + θ(1)
	- T(2n / 3): 재귀호출된 서브트리에서 MAX-HEAPIFY을 수행하는 시간 = 서브 트리 최대 개수
	- θ(1): 자식 노드와 값 대소 비교하는 시간
- 마스터 정리에 의해, O(lg n) = O(h = 트리의 높이)

2. BUILD-MAX-HEAP(A)
- 배열을 최대 힙으로 바꿔주는 프로시저
- MAX-HEAPIFY를 바닥부터 거꾸로 순차 수행하면 최대 힙
- 마지막 레벨의 노드(리프)들은 원소가 하나 이므로 호출하지 않아도 된다. (A[length/2 + 1]..A[length]까지 skip)

BUILD-MAX-HEAP(A) = A 배열을 최대 힙으로 만드는 프로시저
```
A.heap-size = A.length
for i = A.length / 2 downto 1
	MAX-HEAPIFY(A, i)
```

- 수행시간: MAX-HEAPIFY * n 이므로 O(n lg n)
- 더 정확히 구하면,
	- MAX-HEAPIFY는 높이가 h인 노드에서 호출될 때 O(h)
	- 높이가 h인 노드수: 최대 n / 2<sup>h+1</sup>
	- 수행시간은 높이가 h인 노드수 * 높이h 의 높이별 합
	- O(n * (h / 2<sup>h</sup>)의 합) = O(n)