- 함수 타입
- 고차 함수와 코드를 구조화할 때 고차 함수를 사용하는 방법
- 인라인 함수
- 비로컬 `return` 과 레이블
- 무명 함수

## 8.1 high order function

- 다른 함수를 **파라미터**로 받는 함수
- 함수를 **반환**하는 함수

### 함수 타입

- 고차 함수를 선언하려면 인자로 함수 타입을 어떻게 선언하는지 부터 알아야한다
```kotlin
    // (파라미터 타입, ...) -> 리턴 타입
    val sum: (Int, Int) -> Int = {x, y -> x + y}
    val action: () -> Unit = { println(42) }
```
- 함수 타입에서 파라미터의 데이터 타입을 알기 때문에 뒤따르는 **람다에서 스마트 캐스트 가능**
```kotlin
    // 리턴이 nullable
    var canReturnNull: (Int, Int) -> Int? = { x, y => null }
    
    // 함수 타입이 nullable
    var funOrNull: ((Int, Int) -> Int)? = null
```

### 인자로 받은 함수 호출
```kotlin
fun twoAndThree(operation: (Int, Int) -> Int) {
    	val result = operation(2, 3)
    	println("The result is $result")
    }
```
- 파라미터(함수)의 이름을 `operation` 으로 하고 함수 구현에서 그 이름 그대로 호출
```kotlin
    // 조건에 맞는 문자만 문자열로 만들어 리턴
    fun String.filter(predicate: (Char) -> Boolean) {
    	val sb = StringBuilder()
      for (index in 0 until length) {
    		val element = get(index)
    		if (predicate(element)) sb.append(element)
    	}
    	return sb.toString()
    }
    
    // abcc
    >>> println("abc1c".filter { it in 'a'..'z'} )
```

### 자바에서 코틀린 함수 타입 사용

- 일반 인터페이스로 컴파일
- 함수 타입 변수 = `FunctionN` 인터페이스를 구현한 인스턴스
- `invoke()` : 함수 실행하는 함수
```kotlin
    fun processTheAnswer(f: (Int) -> Int) {
    	println(f(42))
    }

    // 자바 8 이상에서 람다 사용
    processTheAnswer(number -> number + 1)

    // 자바 8 이전
    processTheAnswer(
    	new Function1<Integer, Integer>() {
    		@Override
    		public Integer invoke(Integer number) {
    			System.out.println(number);
    			return number + 1;
    		}
    })
```
### 함수 타입 파라미터의 Nullabilty 와 Default

- 함수 타입 파라미터 또한 디폴트 값 지정 가능
```kotlin
    fun processTheAnswer(f: (Int) -> Int = { it + 1 }) {
    	println(f(42))
    }
   ```

- 함수 타입 파라미터가 널이 될 수 있음
```kotlin
    fun processTheAnswer((f: (Int) -> Int)? = null) {
    	println(f?.invoke(42))
    }
```
### 함수에서 함수를 반환

- 사용자가 선택한 배송 수단에 따라 **배송비를 계산하는 방법**이 달라진다면?
```kotlin
    enum class Delivery { STANDARD, EXPEDITED }
    
    class Order(val itemCount: Int)
    
    fun getShippingCostCalculator(
            delivery: Delivery): (Order) -> Double {
        if (delivery == Delivery.EXPEDITED) {
            return { order -> 6 + 2.1 * order.itemCount }
        }
    
        return { order -> 1.2 * order.itemCount }
    }
```

### 람다를 활용한 중복 제거

- 웹사이트 방문 기록 분석
    - 방문 사이트 경로, 머문 시간, 운영체제
```kotlin
    data class SiteVisit(
        val path: String,
        val duration: Double,
        val os: OS
    )
    
    enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }
    
    val log = listOf(
        SiteVisit("/", 34.0, OS.WINDOWS),
        SiteVisit("/", 22.0, OS.MAC),
        SiteVisit("/login", 12.0, OS.WINDOWS),
        SiteVisit("/signup", 8.0, OS.IOS),
        SiteVisit("/", 16.3, OS.ANDROID)
    )
    
    // 윈도우 사용자의 평균 방문 시간
    val averageWindowsDuration = log
        .filter { it.os == OS.WINDOWS }
        .map(SiteVisit::duration)
        .average()
```

- 윈도우 뿐만 아니라 특정 OS 에 대해서 평균 방문 시간을 알고싶다면 ? → 파라미터로 빼자
```kotlin
    // 로그 데이터에 대한 확장 함수로 정의
    fun List<SiteVisit>.averageDurationFor(os: OS) =
            filter { it.os == os }
    				.map(SiteVisit::duration)
    				.average()
```

- 모바일(ios, android) 의 평균 머문 시간을 알고 싶다면?
```kotlin
    val averageMobileDuration = log
        .filter { it.os in setOf(OS.IOS, OS.ANDROID) }
        .map(SiteVisit::duration)
        .average()
        ```

→ 재사용하기 힘들다

→ 조건 자체를 함수 타입으로 넘기도록 수정하자
```kotlin
    fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
            filter(predicate)
    				.map(SiteVisit::duration)
    				.average()
    
    
    fun main(args: Array<String>) {
        println(log.averageDurationFor {
            it.os in setOf(OS.ANDROID, OS.IOS) })
        println(log.averageDurationFor {
            it.os == OS.IOS && it.path == "/signup" })
    }
    ```