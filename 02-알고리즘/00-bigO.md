# Big-O
- 데이터나 사용자의 증가에 따른 알고리즘 성능 예측
- 상수는 모두 O(1)

### O(1)
- 언제나 일정한 속도로 수행

~~~
fun max(a: Int, b: Int): Int {
	return if(a > b) a else b
}
~~~


### O(n)
- 1차원 방정식 그래프
- 데이터가 커질수록 데이터 개수만큼 성능이 나빠짐
 
~~~
fun sum(arr: Arr<Int>) {
	var result = 0
	for(num in arr) {
		result += num
	}
	return result
}
~~~

### O(n^2)
- 행렬
- 중첩된 반복문
- 두개의 연산모두 데이터 개수만큼 많이 돌 때

### O(nm)
- m을 n번 수행할때
- 중첩된 반복문이기 때문에 n^2라고 착각할 수 있음
- **n보다 m이 현저히 작을 수 있으므로 다른 기호로 표기해 주어야 한다!**

~~~
fun print(n: Array<Int>, m: Array<Int>) {
	for(i in 0..n) {
		for(j in 0..m) {
			print i + j
		}
	}
}
~~~

### O(2^n)
- cubic 3차원 육면체형 데이터
- 3중 반복문
- 수행시간에 수가 커질수록 급격히 늘어남

~~~
fun (n, r) {
	if(n <= 0) return 0
	else if(n == 1) return r[n] - 1
	return r[n] = fun (n-1, r) + f(n - 2,r)
}
~~~
