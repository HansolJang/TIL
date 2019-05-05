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