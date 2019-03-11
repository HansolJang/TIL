# 분할정복 접근법
- 재귀적 구조 활용
1. 분할: 전체 문제를 작은 부분 문제들로 분할
2. 정복: 부분 문제들을 재귀적으로 해결. 부분 문제의 크기가 충분히 작으면 직접적인 방법으로 푼다. (기저사례)
3. 결합: 찾은 해들을 결합하여 원래 문제의 해를 도출


#### 예시: 병합정렬
1. 분할: 정렬할 n개 원소의 배열을 n/2개씩 부분 수열 두 개로 분할
2. 정복: 병합 정렬을 이용해 두 부분 배열을 재귀적으로 정렬
3. 결합: 정렬된 두 개의 부분 배열을 병합해 정렬된 배열 하나로 만든다

- 정렬할 수열의 크기가 `1`이면 그것은 이미 정렬된 것이므로 더는 할 일이 없어 재귀 호출이 `바닥`에 이르게 된다.

##### Merge(A, p, q, r)
- 위의 `결합`부분에 해당하는 프로시저

![](https://i.stack.imgur.com/Owtfo.png)

- 1 ~ 2: A[p..q], A[p+1..r]의 크기를 구한다.
- 3 ~ 7: A[p..q]를 L에 A[p+1..r]를 R에 복사한다.
- 8 ~ 9: 정렬이 끝났다는 것을 알려줄 경계 카드를 삽입한다.
- 10 ~ 17
    - L, R의 가장 왼쪽 원소를 비교하며 더 작은 것을 A[k] 자리에 넣는다

- 원소의 크기만큼의 연산으로 두 배열을 결합할 수 있다 -> O(n) 


![](http://www.personal.kent.edu/~rmuhamma/Algorithms/MyAlgorithms/Sorting/Gifs/mergeSort.gif)
![](https://i.stack.imgur.com/wk49i.png)


#### 분할정복 알고리즘의 분석
- 점화식을 풀어서 알고리즘에 대한 성능 한계를 구하자
- 가장 작은 문제는 정렬할 원소가 1개일 때로, 상수 시간 소요 O(1)
- 입력의 크기가 n인 문제를 b개로 나눠서 총 a개의 문제로 만들어냈다고 할때, 
```
총 수행시간 T(n) = aT(n / b) + D(n) + C(n)
```

##### 병합 정렬의 분석
1. 분할: 부분 배열의 중간 위치를 계산. 나누기 연산 O(1)
2. 정복: 두 개의 부분 문제를 재귀적으로 푸는데, 각 부분 문제는 크기가 n/2이므로 `2 * T(n / 2)`
3. 결합: 위에 설명했듯이 O(n)

```
수행시간 T(n) = O(1)     
or 2T(n / 2) + O(n)     
```

![](https://0jun0815.github.io/assets/images/algorithm/2018-11-06-merge-sort/tree2.png)

- 즉 O(n lg n) (밑은 2)가 된다.

```kotlin
fun mergeSort(list: List<Int>): List<Int> {
    if (list.size <= 1) return list
    val middle = list.size / 2
    var left = list.subList(0, middle);
    var right = list.subList(middle, list.size);
    return merge(mergeSort(left), mergeSort(right))
}


fun merge(left: List<Int>, right: List<Int>): List<Int>  {
    var indexLeft = 0
    var indexRight = 0
    var newList : MutableList<Int> = mutableListOf()

    while (indexLeft < left.count() 
           && indexRight < right.count())
        if (left[indexLeft] <= right[indexRight]) 
            newList.add(left[indexLeft++])
        else 
            newList.add(right[indexRight++])

    for(i in indexLeft until left.size) newList.add(left[i])
    for(i in indexRight until right.size) newList.add(right[i])

    return newList;
}
```

- 재귀 대상: 부분 문제가 충분히 커서 재귀적으로 풀 수 있는 상태
- base case: 부분 문제가 충분히 작아져 재귀 호출을 할 수 없는 상태

### 점화식
- 분할정복 알고리즘의 수행시간을 자연스럽게 표현할 수 있다
- 자신을 더 작은 입력으로 표현하는 방정식/부등식

점화식을 푸는 3가지 방법(= 점근적 θ나 O를 얻는 방법)
1. 치환법
    - 한계를 추측하고, 그 추측이 옳다는 것을 증명하기 위해 수학적 귀납법을 사용
2. 재귀 트리 방법
    - 점화식을 각 노드가 필요한 비용 트리로 변환하고, 그 합으로 점근적 한계를 구하는 방법 (위의 병합정렬에서 사용한 방법)
3. 마스터 방법
    - 하나의 문제가 몇개(b)의 부분문제로 나뉘고, 시간이 얼마나 더 걸리고(a), 결합하는 시간(f(n))은 얼마인지를 점화식으로 나타내는 방법
    - T(n) = aT(n / b) + f(n)

- 점화식을 서술하고 해를 구할 때 보통 한계 조건, 올림, 내림을 생략한다
- 문제를 모두 풀고난 후에 고려할만 한 조건인지 결정한다


#### 예시2: 최대 부분 배열 문제
- 주어진 배열에서 연속된 원소의 합이 최대가 되는 배열 중 하나를 구하는 문제

![](https://www.geeksforgeeks.org/wp-content/uploads/kadane-Algorithm.png)

- 수열을 중간값(mid)을 기준으로 둘로 나누면, 최대 부분 배열 중 하나 A[i..j]는
    - low <= i <= j <= mid: A[low..mid]에 완전히 포함되는 경우
    - mid + 1 <= i <= j <= high: A[mid+1..high]에 완전히 포함되는 경우
    - low <= i <= mid <= j <= high: 중간값에 걸쳐있을 경우
        - A[i..mid]의 최대와 A[mid + 1..j]의 최대를 구해 합치면 된다 

![](http://sonny.io/wp-content/uploads/2016/07/max_crossing1.png)

A[mid]를 포함해 걸쳐있는 최대 부분 배열을 찾는 프로시저
FIND-MAX-CROSSING-SUBARRAY

![](http://pds27.egloos.com/pds/201501/29/89/a0322389_54c9d85e785d9.png)

- 1~7행: A[mid]를 무조건 포함하는 A[i..mid]이 최대가 되는 i를 찾는다
- 8~14행: A[mid+1]을 무조건 포함하는 A[mid+1..j]가 최대가 되는 j를 찾는다
- 두개를 합하면 겹치는 최대 부분 수열을 Θ(n)에 구할 수 있다
- 이 프로시저를 토대로 최대 부분 수열을 찾는 수도코드를 작성할 수 있다

FIND-MAX-SUBARRAY

![](http://www2.hawaii.edu/~nodari/teaching/s15/Notes/Topic-07/find-maximum-subarray.jpg)

- 1~2행: 베이스 케이스. 원소가 하나일 땐 최대 부분수열이 그 자체이다.
- 4행: A[low..mid]에서의 최대 부분 수열을 리턴하도록 재귀호출한다.
- 5행: A[mid+1..high]에서의 최대 부분 수열을 리턴하도록 재귀호출한다.
- 6행: A[mid]를 무조건 포함하는 최대 부분 수열을 리턴하도록 재귀호출한다.
- 7~11행: 4,5,6행의 결과 중 가장 큰 부분 수열을 리턴한다.

이 분할 정복 알고리즘의 수행시간은
T(n) = Θ(1)             if n = 1    
        2T(n/2) + Θ(n)  if n > 1

```kotlin
fun main(args: Array<String>) {
    val array = arrayListOf<Int>(-2,-3,4,-1,-2,1,5,-3)
    val ms = array.findMaximumSubArray(0, array.lastIndex)
    println("sum: ${ms.first}, start: ${ms.second.first}, end: ${ms.second.second}")
}

// 최대 부분 수열의 합, 시작, 끝 인덱스 리턴
fun ArrayList<Int>.findMaximumSubArray(low: Int, high: Int): Pair<Int, Pair<Int, Int>> {

    // 로컬 함수
    fun findCrossingMaximumSubArray(low: Int, mid: Int, high: Int): Pair<Int, Pair<Int, Int>> {
        var lowSum = 0
        var sum = 0
        var maxLow = 0
        var maxHigh = 0
        for(i in mid downTo low) {
            sum += this[i]
            if(lowSum < sum) {
                lowSum = sum
                maxLow = i
            }                 
        }
        var highSum = 0
        sum = 0
        for(j in mid + 1..high) {
            sum += this[j]
            if(highSum < sum) {
                highSum = sum
                maxHigh = j
            }
        }
        return lowSum + highSum to (maxLow to maxHigh) 
    }

    if (low == high) return this[low] to (low to high)
    val mid = (low + high) / 2
    return listOf(this.findMaximumSubArray(low, mid),
            this.findMaximumSubArray(mid + 1, high),
            findCrossingMaximumSubArray(low, mid, high))
            .maxBy { it.first }!!
}
```
