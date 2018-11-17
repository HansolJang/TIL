# Heap

##### 우선순위 큐: 우선순위가 가장 높은 자료가 가장 먼저 꺼내지는 자료구조
- 힙은 가장 큰 원소를 찾는 데 최적화된 형태의 이진 트리
- 삽입과 삭제(원소 꺼내기) 연산이 O(logN)
- 우선순위 큐를 구현하는 데 힙을 쓰자!
- *이진 트리*로 구현되는 힙이 많지만, 그렇지 않은 힙도 있다
- **목표: 최대 원소를 가능한 한 빠르게 찾자**
     
             
##### 배열을 이용한 힙 구현
- A[i]의 왼쪽 자식: A[2*i + 1]
- A[i]의 오른쪽 자식: A[2*i + 2]
- A[i]의 부모: A[(i-1) / 2]

### Binary Heap
- 부모 노드 원소 >= 자식 노드 원소 (왼쪽, 오른쪽 자식은 서로 상관X)
- 원소가 한쪽으로 치우치지 않기 위해 제약을 둔다
	1. 마지막 레벨을 제외한 모든 레벨에 노드가 꽉 차 있어야 한다.
	2. 마지막 레벨에 노드가 있을 때는 항상 가장 왼쪽부터 순서대로 채워져야 한다.
- 항상 균형있게 트리가 꽉 차있기 때문에 O(logN) 연산이 가능하다.

#### Insert
- 위의 2번 제약 조건을 활용한다.
	1. 원소를 무조건 마지막 노드에 위치 
	2. 부모와 대소 비교하며 자리를 바꾼다.
	3. 반복한다.
- O(lonN): 최대로 부모 노드와 자리를 바꾸는 횟수

![](http://cs.lmu.edu/~ray/images/heapinsert.png)

#### Remove
- 위의 1번 제약 조건을 활용한다.
	1. 트리의 제일 마지막 리프 노드를 루트 자리로 옮긴다.
	2. 자식 둘 중 큰 것과 자리를 바꾼다.
	3. 반복한다. 
- O(lonN): 최대로 자식 노드와 자리를 바꾸는 횟수

![](http://cs.lmu.edu/~ray/images/heapdelete.png)


### Heap Sort
- 힙을 이용한 정렬
- remove를 n번 하면 정렬이 된다! -> O(NlogN)
- 하나씩 remove하면서 그 원소를 제일 마지막에 위치하면 정렬된 배열을 결과로 가질 수 있다 -> 추가 메모리 공간이 필요 없다!


~~~
#include <cstdio>
#include <vector>
#include <algorithm>
using namespace std;
#define INVALID -1

vector<int> heap;

int getRoot(int i) {
	if(i == 0) return INVALID;
	return (i - 1) / 2;
}

int getLeftChild(int i) {
	return 2 * i + 1;
}

int getRightChild(int i) {
	return 2 * i + 2;
}

void heapInsert(int num) {
	heap.push_back(num);
	int idx = heap.size() - 1;
	// 부모와 자리 바꿀 수 있으면 바꾸기
	while(idx > 0 && heap[getRoot(idx)] < heap[idx]) {
		swap(heap[idx], heap[getRoot(idx)]);
		idx = getRoot(idx);
	}
}

int heapRemove() {
	int element = heap[0];
	heap[0] = heap[heap.size() - 1];
	heap.pop_back();
	// 루트부터 자식과 비교해 자리 찾기
	int here = 0;
	while(true) {
		int left = getLeftChild(here);
		int right = getRightChild(here);
		if(left >= heap.size()) break;

		int next = here;
		if(heap[next] < heap[left]) {
			next = left;
		} else if(right < heap.size() && heap[next] < heap[right]) {
			next = right;
		}
		if(next == here) break;
		swap(heap[here], heap[next]);
		here = next;
	}
	return element;
}
~~~