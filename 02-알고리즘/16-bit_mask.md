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

bool 배열을 수의 범위 만큼 가져야만 한다. 메모리를 줄일 순 없을까?   
-> 짝수를 제외하면 반으로 줄일 수 있다!   
-> 비트마스크를 사용하면 최적화 가능

##### 에라토스테네스의 체를 이용한 빠른 소인수분해
- 각 수의 가장 작은 소수를 기록 (예: 28 = 2\*2\*7 -> 2)

~~~
// 가장 작은 소수를 저장
// 결과가 minFactor[i] = i 이면, 소수
// 0, 1 제외하고 모두 소수로 초기화
int minFactor[MAX_N+1];

void erathosthenes2() {
	...초기화
	int sqrtn = sqrt(n);
	for(int i = 2; i < sqrtn; i++) {
		// 아직 보지 않았다면
		if(minFactor[i] == i) {
			// 모든 배수의 가장 작은 소수를 자신으로 모두 업데이트
			for(int j = i*i; j <= n; j += i) {
				minFactor[j] = i;
			}
		}
	}
}

vector<int> factor(int n) {
	vector<int> ret;
	// n이 1이 될 때까지 계속 소인수로 나눈다
	while(n > 1) {
		 ret.push_back(minFactor[n]);
		 n /= minFactor[n];
	}
	return ret;
}

~~~

## 유클리드 알고리즘
- 두 수의 최대공약수를 구하는 방법
- 가장 오래된 알고리즘
- `a`, `b`(p > q)의 최대공약수(`G`)는 `a % b`, `b`의 최대공약수와 같다

### 증명
`gcd(a,b) = k `: a, b의 최대공약수는 k이다.  
`a = bq + r`: r은 a를 b로 나눈 나머지다

~~~
// m, n은 서로소다
a = mk
b = nk

// 위 식을 a = bq + r에 대입하면,
mk = nkq + r
r = (m-nq)k
~~~
r도 k를 갖고 있고, b도 k를 갖고있다.    
k가 공약수는 맞지만 최대공약수인지는 모른다.   
n, m-nq는 서로소인지를 증명해야 한다. -> `서로소가 아니라고 가정하자`

~~~
n = xG
m-nq = yG
(G != 1)

// m-nq에 대입하면,
m - xGq = yG
m = (xq + y)G 
~~~
n에도 G, m에도 G가 있어서 서로소라는 전제에 모순이 된다.    
따라서 n과 m-nq는 서로소이다.
따라서 `gcd(a, b) = gcd(b, r)`이다