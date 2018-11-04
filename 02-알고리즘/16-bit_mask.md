# Step16. 비트마스크
- 집합의 구현
- 응용 예제

### 장점
1. 더 빠른 수행 시간: 대부분 O(1)에 수행된다.
2. 더 간결한 코드
3. 더 작은 메모리 용량

|이름|            연산          |  코드  |
|:--|:----------------------- |:------|
|AND|두 정수 a,b를 비트별로 and 연산| a & b |
|OR |두 정수 a,b를 비트별로 or  연산| a \| b |
|XOR|두 정수 a,b를 비트별로 xor 연산(toggle)| a ^ b |
|NOT|   정수 a의 비트별 not 연산   |   ~a  |
|Shift|정수 a를 왼쪽으로 b비트 시프트| a << b |
|Shift|정수 a를 오른쪽으로 b비트 시프트| a >> b |


### 에라토스테네스의 체
- c++, java 모두 컴파일러에 따라 다르지만 보통 bool 타입의 자료형이 1byte이다
- bool로 정말 1비트를 사용할 순 없을까?
- char(1byte)형을 쪼개어 1비트씩 사용하자
- >> 3: 8로 나눈 몫
- & 7: 8로 나눈 나머지

~~~
// 소수를 구하려는 범위의 최댓값
int n;
// 소수 여부 저장
unsigned char sieve[(MAX_N+7) / 8]; // 몫을 +1
// k가 소수인지 확인
inline bool isPrime(int k) {
	return sieve[k >>3] & (1 << (k & 7));
}
// k가 소수가 아니라고 표시
inline void setComposite(int k) {
	sieve[k >> 3] &= ~(1 << (k & 7));
}
void erathosthenes() {
	memset(sieve, 255, sizeof(sieve));
	setComposite(0);
	setComposite(1);
	int sqrtn = sqrt(n);
	for(int i = 2; i < sqrtn; i++) {
		// 지워지지 않았다면
		if(isPrime[i]) {
			// 배수 모두 지우기
			for(int j = i*i; j <= n; j += i) {
				setComposite(j);
			}
		}
	}
}
~~~
