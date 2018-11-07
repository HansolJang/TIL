# HashTable
- F(key) -> HashCode -> Index -> LinkedList 검색 -> Value
- key: 숫자, 문자열, 파일 등 여러가지 형태
- key가 공백까지 완벽히 일치해야 동일한 해시코드가 나온다
- HashCode: 정수값인 해시코드를 나머지 연산하여 배열의 인덱스로 값에 바로 접근
- Hash Algorithm: 키를 해시코드로 변환하는 알고리즘, 얼마나 잘 분배하느냐에 따라 검색 시간이 O(1)에서 최악일 경우 O(n)이 된다.
- Index: HashCode % size

### Collision
- 키는 다르지만, 같은 해시코드 -> 같은 인덱스
- 다른 해시코드지만 -> 같은 인덱스
- 충돌이 일어날 경우 해당 인덱스의 LinkedList에 insert한다.
