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