- 람다식과 멤버 참조
- 함수형 스타일로 컬렉션 다루기
- 시퀀스: 지연 컬렉션 연산
- 자바 함수형 인터페이스를 코틀린에서 사용
- 수신 객체 지정 람다 사용

## 5.1 람다식과 멤버 참조

- 다른 함수에 넘길 수 있는 작은 코드 조각
- 함수를 선언하지 않고 코드 블록을 함수의 인자로 넘기는 방법
- 메소드가 1개인 무명 객체를 대신할 수 있다.
- 원래 코드와 비슷한 성능 (8장에서 설명)
```kotlin
    button.setOnClickListener(new OnClickListener() {
    		@Override
    		public void onClick(View view) {
    				...
    		}
    })

    button.setOnClickListener {
    ...
    }

    data class Person(val name: String, val age: Int)
    
    val people = listOf(Person("alice", 29), Person("bob", 29))
    people.maxBy { it.age }
    // people.maxBy(Person::age)
```

람다식 문법
```kotlin
    { x: Int, y: Int -> x + y }
```
- 중괄호로 여닫는다
- 파라미터와 구현체는 '- >'으로 구분한다
```kotlin
    // 정식 람다 문법
    people.maxBy({ p: Person -> p.age })
    
    // 문법 관습. 괄호를 밖으로 뺄 수 있다
    people.maxBy() { p: Person -> p.age }
    
    // 람다식이 유일한 파라미터일 경우 괄호 생략 가능
    people.maxBy { p: Person -> p.age }
    
    // 람다식의 인자가 하나일 경우 타입추론을 통해 it 키워드로 받을 수 있다
    people.maxBy { it.age }
```
- 람다를 변수에 사용할 땐 `it`  키워드 사용 불가 (타입 명시 필수)
- 람다식이 중첩적일 땐 `it` 키워드 보다는 명시적으로 타입을 적어주는 것이 헷갈리지 않음
```kotlin
    fun printProblemcounts(response: Collection<String>) {
    		var clientErrors = 0
    		response.forEach {
    				if(...) clientErrors++
    		}
    }
```

- 람다식 안에서 로컬 변수 접근이 가능하다 (자바는 안됨)
- 람다가 capture한 변수는 `clientErrors`
- 람다만 변수에 저장해서 나중에 실행해도 포획한 변수를 여전히 읽고 쓸 수 있다
- 함수의 생명주기와 람다의 생명주기가 다를텐데 어떻게 가능할까?
```kotlin
    class Ref<T>(var value: T) 
    
    >>> val counter = Ref(0)
    val inc = { counter.value++ }
```
    

- 변경 가능하게 하기 위해 `Ref<T>` 클래스로 한번 더 감싼 인스턴스를 만든다
- 프리미티브 `val` 은 변경할 수 없지만 레퍼런스 타입 `val` 의 멤버는 변경할 수 있다

넘기려는 코드가 이미 선언되어 있다면? 

- 함수를 값으로 바꾸어서 넘기자! → **멤버 참조**
- 이중 콜론으로 표현 (::)
```kotlin
    // age를 리턴하는 함수를 값으로 표현
    Person::age
```

## 5.2 컬렉션 함수형  API

- 컬렉션을 다루는 코틀린 표준 라이브러리(람다 문법) 몇가지

### filter와 map
```kotlin
    val list = listOf(1, 2, 3, 4)
    val evenList = list.filter { it % 2 == 0 }
    
    >>> (2, 4)
```
- 원치 않는 원소를 제거
```kotlin
    val list = listOf(1, 2, 3, 4)
    val squareList = list.map { it * it }
    
    >>> (1, 4, 9, 16)
```

- 원소를 변환
```kotlin
    val list = listOf(1, 2, 3, 4)
    val evenSquareList = list
                                                .filter { it % 2 == 0 }
                                                .map { it * it }
    
    >>> (4, 16)
```
- 연쇄호출 가능
```kotlin
    val list = listOf(1, 2, 3, 4)
    // 항상 maxBy를 계산해 비효율적
    list.filter { it == it.maxBy { it } }
    
    val maxElement = it.maxBy { it }
```    list.filter { it == maxElement }

### all, any, count, find
```kotlin
- `all` 과 `any` : 컬렉션의 **모든 원소**가 어떤 조건을 만족하는지
- `count` : 조건을 만족하는 원소 개수
- `find` : 조건을 만족하는 첫번째 원소 리턴
    - 만족하는 원소가 없다면 `null` 리턴
    - `null` 을 명시해주고 싶다면 같은 기능을 하는 `firstOrNull` 함수 사용

    val list = listOf(1, 2, 3)
    
    // 모든 원소가 3인 것은 아니다 = 3이 아닌 원소가 하나라도 있다
    !list.all { it == 3 }
    
    // 3이 아닌 원소가 하나라도 있다
    list.any { it != 3 }
```
### groupBy

- 컬렉션을 여러 그룹으로 나눠준다
```kotlin
    val list = listOf("a", "ab", "b")
    // 동일한 첫글자로 시작하는 그룹으로 묶는다
    list.groupBy(String::first)
    
    // 결과는 Map<String, List<String>>
    >>> {"a" = ["a", "ab"], "b" = ["b"]}
```
### flatMap

- 중첩된 컬렉션 안의 원소 처리
```kotlin
    class Book(val title: String, val authors: List<String>)

    // books 컬렉션의 모든 저자를 가져올 수 있다 (set: 중복 없이)
    // toSet()을 호출하지 않으면 List<String> 반환
    books.flatMap { it.authors }.toSet()
```
- `flatMap` : 파라미터를 모든 원소에 적용하고, 결과를 한데 모은다
- 리스트의 리스트의 원소를 모아야할 때 사용