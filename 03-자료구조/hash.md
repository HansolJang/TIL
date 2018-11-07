# HashTable
- F(key) -> HashCode -> Index -> Value
- key: 숫자, 문자열, 파일 등 여러가지 형태
- key가 공백까지 완벽히 일치해야 동일한 해시코드가 나온다
- HashCode: 정수값인 해시코드를 나머지 연산하여 배열의 인덱스로 값에 바로 접근
- Hash Algorithm: 키를 해시코드로 변환하는 알고리즘, 얼마나 잘 분배하느냐에 따라 검색 시간이 O(1)에서 최악일 경우 O(n)이 된다.
- Index: HashCode % size

### Collision
- 키는 다르지만, 같은 해시코드 -> 같은 인덱스
- 다른 해시코드지만 -> 같은 인덱스

##### 충돌 해결 방식
1. Separate Chaining
	- 인덱스에 LinkedList를 두는 방법
	- 같은 인덱스에 값을 저장할 경우 insert 해준다
	- 같은 값에 편향될 경우 검색이 O(n)일 수 있다
	- Tree를 이용하면 더 효율적이다 (JDK 1.8의 경우 중복된 인덱스 값이 8개 미만이면 LinkedList, 이상이면 Tree를 사용)
2. Open Addressing
	- 버킷 중 빈 곳을 활용 (추가메모리X)
	- Linear Probing: index 뒤를 순서대로 보며 가장 먼저 찾은 빈 곳에 데이터를 저장하는 알고리즘

### Resizing
- 버킷이 어느 정도 찼을 경우 사이즈를 2배로 늘린다 (JDK 1.8은 0.75, 생성자에서 변경 가능하다)
- size가 변경됐으므로 index들도 모두 변해 기존 해시 테이블에서 새로운 해시 테이블로 옮기는 연산이 필요하다
- size는 2^n이므로 나머지 연산을 할 경우, 하위 a개의 비트만 인덱스로 활용하게 된다 -> 충돌이 더 많아진다!

### 보조 해시 함수
- size는 2의 배수더라도 나머지 연산은 **소수**로 해야 충돌이 가장 적다
- 보조 해시 함수는 '키'의 해시 값을 변형하여 충돌을 줄이도록 한다

~~~
static final int hash(Object key) { 
	int h; 
	return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16); 
}
~~~
java8의 보조 해시 함수     

- 나머지 연산자는 비트연산으로 대체하면 훨씬 빠르다  