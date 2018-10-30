# Step14. 정수론
- 소수
- 소인수분해
- 에라토스테네스의 체
- 유클리드 알고리즘

## 소수
- 양의 약수가 1과 자기 자신 두개 뿐인 자연수
- 합성수: 약수가 3개 이상인 수
- **1은 소수도 합성수도 아니다**

#### 소수 판별
-  2 ~ 루트n 까지 모두 나눠본다.
 
~~~
bool isPrime(int n) {
	// 예외 (1, 2, 짝수 제외)
	if(n <= 1) return false;
	if(n == 2) return true;
	if(n % 2 == 0) return false;
	// 제곱근까지 검사 (홀수만 확인)
	int sqrtn = sqrt(n);
	for(int div = 3; div <= sqrtn; div += 2) {
		if(div % n == 0) return false;
	}
	return true;
}
~~~

32비트 정수 최댓값의 제곱근은 2^16 = 65536 이므로 그렇게 느리지 않다.

#### 소인수분해
- 합성수를 소수의 곱으로 표현

~~~
vector<int> factorSimple(int n) {
	vector<int> ret;
	int sqrtn = sqrt(n);
	// [2, n제곱근] 범위 시도
	for(int div = 2; div <= sqrtn; div++) 
		while(n % div == 0) {
			n /= div;
			ret.push_back(div);
		}
	if(n > 1) ret.push_back(n);
	return ret;
}
~~~
