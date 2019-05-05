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

3. 힙 정렬 알고리즘
- 배열을 최대 힙으로 만든다.
- 가장 큰 원소가 루트에 있다.
- 루트 노드 A[1]를 힙의 가장 마지막 원소 A[n]와 바꾼다.
- A[1..n-1] 에 대해 위를 반복한다.
- 새로운 루트가 가장 크지 않을 수 있으므로 MAX-HEAPIFY(A, 1)를 호출한다.
```
BUILD-MAX-HEAP(A)
for i = A.length downto 2
	exchange A[1] with A[i]
	A.heap-size = A.heap-size - 1
	MAX-HEAPIFY(A, 1)
```

- 수행시간
	- BUILD-MAX-HEAP = O(n)
	- MAX-HEAPIFY(A, 1)를 (n - 1)번 실행
	- MAX-HEAPIFY는 O(lg n)
	- 다 합치면 O(n) + (n - 1) * O(lg n) = O(n lg n)

```kotlin
val heap = ArrayList<Int>()

    fun parent(i: Int) = i / 2

    fun left(i: Int) = i * 2

    fun right(i: Int) = i * 2 + 1

    fun maxHeapify(i: Int) {
        val l = left(i)
        val r = right(i)
        var largest = -1
        if (l < heap.size && heap[l] > heap[i])
            largest = l
        else if (r < heap.size && heap[r] > heap[i])
            largest = r
        if (largest > -1) {
            swap(i, largest)
            maxHeapify(largest)
        }
    }

    fun swap(i: Int, j: Int) {
        if (i < heap.size && j < heap.size) {
            heap[i] += heap[j]
            heap[j] = heap[i] - heap[j]
            heap[i] -= heap[j]
        }
    }

    fun buildMaxHeap() {
        for(i in heap.size / 2 downTo 1)
            maxHeapify(i)
    }
```


#### 우선순위 큐
- 키 값을 가진 원소 S들의 집합
- 우선순위가 가장 큰 것을 바로 꺼낼 수 있는 자료 구조
- 최대 우선순위 큐와 최소 우선순위 큐가 있다
- 제공하는 연산
	- INSERT(S, x): S에 원소 x를 넣는다
	- MAXIMUM(S): S에서 키 값이 최대인 원소를 리턴한다
	- EXTRACT-MAX(S): S에서 키 값이 최대인 원소를 제거한다
	- INCREASE-KEY(S, x, k): 원소 x의 키 값을 k로 증가시킨다. 이때 k는 x의 현재 키 값보다 작아지지 않는다고 가정한다.
- 구현 방법에는 여러가지가 있고, 단순히 list를 사용해도 되지만, 더 효율적인 힙을 이용해 주로 구현한다.
	- list로 구현하면, 대부분의 연산이 O(n)
	- heap으로 구현하면, 이진 트리의 특성때문에 대부분의 연산이 O(lg n)
- 힙은 크기가 n인 집합에 대해 어떤 우선순위 큐 연산이라도 O(lg n)시간에 할 수 있다.
- 최대 우선순위 큐는 특정 OS에서 다음 수행할 프로세스를 고를 때 사용되고 있다.
- 최소 우선순위 큐는 INSERT, MINIMUM, EXTRACT-MIN, DECREASE-KEY 연산을 제공한다.
	- 최소 우선순위 큐는 사건 반응형 시뮬레이터에서 사용된다.
	- 사건 발생 순서대로 순차로 선택할 때 사용한다.
- 실제로 응용할 때는 핸들을 정확하게 유지해야한다. (핸들: 원소 각각에 대응되어 원소를 찾게 해주는 키 같은 데이터, 예를 들어 배열의 인덱스)

1. HEAP-MAXIMUM(A)
```
HEAP-MAXIMUM(A)
return A[1]
```
- A에서 최대값을 리턴한다.
- θ(1)

2. HEAP-EXTRACT-MAX(A)
- A에서 최댓값을 제거하고 리턴한다.
- 마지막 노드를 첫번째 노드에 넣는다.
- 남은 배열을 최대 힙으로 만든다.
- MAX-HEAPIFY(A, 1) 만큼의 시간만 걸리므로 수행시간은 O(lg n)이다.
```
HEAP-EXTRACT-MAX(A)
if (A.heap-size < 1)
	error "heap underflow"
max = A[1]
A[1] = A[A.heap-size - 1]
A.heap-size = A.heap-size - 1
MAX-HEAPIFY(A, 1)
```

3. INCREASE-KEY(A, i, key)
- 원소 x의 키 값을 k로 증가시킨다.
- k >= x라고 가정한다. (부모와 비교해 올라가기만 하면 된다.)
	- 자식과 원소 전체의 순서를 알 수 없기 때문에
- i: 키를 증가시킬 원소가 있는 배열의 인덱스
- 새로운 키가 들어갈 적절한 장소를 골라 끼워넣는다.
- 최대 트리의 높이만큼 반복문이 시행되므로 수행시간은 O(lg n)이다.
```
INCREASE-KEY(A, i, key)
if (key < A[i])
	error "새로운 키가 현재보다 작다"
A[i] = key
while i > 1 and A[PARENT(i) < A[i]]
	A[i] <-> A[PARENT(i)]
	i = PARENT(i)
```

4. MAX-HEAP-INSERT(A, key)
- A에 key 값을 삽입한다.
- 우선 트리에 키값이 `-∞`인 노드를 리프에 추가하여 확장하고, INCREASE-KEY(A, i, key)를 호출해 제자리를 찾는다.
- 마지막 노드에서 INCREASE-KEY의 반복문이 일어나는 횟수는 트리의 높이 = `log N`이므로 수행시간도 O(lg n)이다.
```
MAX-HEAP-INSERT(A, key)
A.heap-size = A.heap-size + 1
A[heap-size] = -∞
HEAP-INCREASE-KEY(A, A.heap-size - 1, key)
```

### 퀵 소트
- 최악의 경우 θ(n<sup>2</sup>)
- 평균 수행 시간 θ(n lg n)
- θ(n lg n)에 숨겨진 상수 인자도 매우 작다.
- 내부 정렬 (불필요한 메모리 공간을 사용하지 않는다)
- 일반적으로 실제 문제에 가장 유용한 정렬

#### 퀵 정렬의 분할 정복법
1. 분할
	- `A[p..r]`을 `q`를 중심으로 `A[p..q - 1]`, `A[q + 1.. r]`로 나눈다
	- `A[p..q - 1]`에는 `A[q]`보다 작은 원소들만 몰고, `A[q + 1.. r]`에는 A`[q]`보다 큰 원소들만 몬다
	- 모두 다 옮기고 나면 정렬된 `q`의 위치가 나온다 (`A[q]`가 정렬되었다)
2. 정복
	- 퀵정렬을 재귀호출해서 `A[p..q - 1]`, `A[q + 1.. r]`을 정렬한다
3. 결합
	- 재귀호출하며 원소하나씩 정렬되기 때문에 합칠 필요가 없다

```
QUICK-SORT(A, p, r)
if (p < r) {
	q = PARTITION(A, p, r)
	QUICK-SORT(A, p, q - 1)
	QUICK-SORT(A, q + 1, r)
}
```

#### PARTITION
- 배열 내부를 재배치 한다
- q의 정렬된 위치를 반환한다
- j가 원소 개수 만큼 반복하므로 θ(n)
```
PARTITION(A, p, r)
// 가장 마지막 원소를 피봇으로 설정. 이 원소의 정렬된 위치를 찾자
x = A[r]
// 현재 i보다 클 가능성이 있는 수
i = p - 1
for j = p to r - 1
	if A[j] <= x
			i = i + 1			// 큰 원소를 만날 때까지 한칸씩 이동
			A[i] <-> A[j] swap	// 현재 원소와 큰 원소 바꾸기
A[i + 1] <-> A[r] swap			// i가 마지막 작은 원소의 위치 이므로 i, i + 1 사이에 A[r] 삽입
return i + 1
```

#### 퀵 정렬의 성능
- 분할 후 양쪽 배열의 크기가 비슷한가
- 기준 원소가 중앙값에 가까운가 (분할이 거의 반으로 가능한가)

##### 분할을 가장 나쁘게 하는 경우
- 최악의 경우: n - 1개와 0개의 배열로 나누는 경우
```
T(n) = T(n - 1) + T(0) + θ(n)
	= T(n - 1) + θ(n)
```
- 위 재귀 관계식을 모두 더하면 θ(n<sup>2</sup>) => 삽입정렬 수준?
- 모든 원소의 값이 같은 경우 삽입정렬을 하면 O(n) 시간에 정렬 가능

##### 분할을 가장 잘 하는 경우
- 분할이 1/2에 가까운 경우
```
T(n) = 2T(n / 2) + θ(n)
```
- 마스터 정리에 따라 θ(n lg n)

##### 균등 분할
- 최적에 더 가까운 수행 속도를 보인다
- 분할의 균형 정도 -> 수행 시간에 얼마나 영향을 미치는가
![](http://staff.ustc.edu.cn/~csli/graduate/algorithms/book6/158_b.gif)
- 사실 얼마나 나누던 θ(n lg n)에 가깝다

##### 평균적인 경우에 대한 직관적 고찰
- 어떤 배열이 평균적으로 입력될 것인가
- 평균적인 경우 좋은 분할 (=1/2에 가까운 분할)과 나쁜 분할(=n-1,0)이 섞여 있다
![](http://thumbnail.egloos.net/600x0/http://pds26.egloos.com/pds/201510/19/89/a0322389_5624ed6146bd9.png)
- 나쁜 분할과 좋은 분할이 번갈아 가면서 일어나므로 좋은 분할에 흡수

##### 랜덤화된 퀵 정렬
- 평균에 가깝게 하기위해 임의로 배열을 랜덤화하자
- 무작위 추출
	- 무조건 마지막 원소를 r로 정하는 것이 아니라 랜덤하게 고른 뒤에 r가 swap
- 모든 입력에 대해 성능의 기댓값이 높아짐 -> Ω(n lg n)으로 줄일 수 있다

더 좋게 만드는 방법은?
- 스택의 깊이를 낮게: 꼬리재귀 사용
- 세개의 무작위 추출된 값중 중앙값을 선택

```
RANDOMIZED-PARTITION
i = RANDOM(p, r)
A[r] <-> A[i]
return PARTITION(A, p, r)
```

```
RANDOMIZED-QUICKSORT(A, p, r)
if (p < r)
	q = RANDOMIZED-PARTITION(A, p, r)
	RANDOMIZED-QUICKSORT(A, p, q - 1)
	RANDOMIZED-QUICKSORT(A, q + 1, r)
```

### 선형 시간 정렬
- 힙 소트, 퀵 소트와 같은 비교 정렬의 최적은 Ω(n lg n)

#### 계수 정렬 (counting sort)
- n개의 입력 원소 각각이 0부터 k 사이에 있는 정수라고 가정 (입력이 적은 범위라고 가정)
- 위와 같이 가정하기 때문에 O(n)이 가능
- k = O(n)일 때 계수 정렬은 θ(n)
- 각 원소 x보다 작은 원소의 개수를 센다
- A[1..n]: 입력
- B[1..n]: 정렬된 출력 저장
- C[0..k]: 임시 작업 공간 (k는 입력A에서 가장 큰 수)

![](http://2.bp.blogspot.com/-TFuBFNY09jw/VZwHFKcMh4I/AAAAAAAAAuA/jMf40Fp20V8/s1600/counting_sort.png)

![](https://www.myassignmenthelp.net/images/counting-sort.png)

1. (a) - 5행 실행
	- 각 원소의 개수가 C 배열에 저장
2. (b) - 8행 실행
	- 원소의 개수를 누적한 값을 C 배열에 저장
3. (c) - 10~12행 1번 실행
	- C배열의 값이 현재 피벗의 정렬된 인덱스가됨
	- 같은 값이 여러개 있을 수 있으므로 C 배열의 값 - 1

- 2 ~ 3행: θ(k)
- 4 ~ 5행: θ(n)
- 7 ~ 8행: θ(k)
- 10 ~ 12행: θ(k + n)
- 다 더하면 총 시간은 θ (k + n)
- 계수 정렬의 안정성: 출력 배열에서의 값이 입력값의 순서와 동일(값이 같을 경우)

#### 기수 정렬 (radix sort)
- 가장 낮은 자리수부터 정렬
- 각 자리수별로 정렬하는 것을 반복
- 정렬이 안정적이어야만 함 (이전에 정렬된 순서를 그대로 유지해주어야함)
- 키가 필드 여러개로 구성되어 있을 때 기수 정렬을 사용 (예: 날짜 정렬. 일, 월, 년 순으로 기수 정렬 사용)
- 최대 자리수인 `d` 만큼 정렬이 반복
- `d`자리 숫자가 `n`개 주어지고 각 자리에는 값이 `k`까지 들어갈 수 있을 때, θ(d(n + k))

![](http://www.ritambhara.in/wp-content/uploads/2018/01/Screen-Shot-2018-01-13-at-3.26.15-PM.png)

```
RADIX-SORT(A, d)
for i = 1 to d
	안정된 정렬을 사용해 배열 A를 i 자리를 기준으로 정렬

```

#### 버킷 정렬 (bucket sort)
- 입력이 균등 분포(uniform distribution)이라고 가정
- 평균 `O(n)`
- 입력이 균일하고 독립적으로 만들어졌다고 가정
- 버킷에 담긴 원소들이 한쪽에 쏠리지 않고 균등에 가깝게 분포 -> 버킷마다의 정렬 시간을 O(1)로 줄이자
![](http://www2.hawaii.edu/~nodari/teaching/f15/Notes/Topic-10/pseudocode-bucket-sort.jpg)
