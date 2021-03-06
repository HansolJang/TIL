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


## 5.3 lazy 컬렉션 연산

- `map` 과 `filter` 같은 함수들은 결과를 즉시 생성한다.
- 중첩적으로 부를 때마다 결과 리스트를 생성해 성능이 떨어진다.

    books.map(Book::title).filter { it.startswith("A") }

    books.asSequence() // 시퀀스로 변환
                .map(Book::title)
                .filter { it.startswith("A") }
                .toList()    // 결과를 다시 리스트로 변환 (시퀀스는 인덱스로 접근하지 못한다)

### Sequence

- 임시 컬렉션을 사용하지 않고 컬렉션 연산 수행 가능
- 코틀린 인터페이스
- `iterator` 메소드로 원소 별 접근
- 원소가 필요할 때 비로소 계산 (lazy)
- ***원소가 많을 땐 시퀀스로 지연 계산하도록 하자***
- 자바의 스트림과 같다 (안드로이드에서 자바 8미만을 사용할 경우 자바 8 스트림을 사용할 수 없다. 상황에 따라 코틀린 시퀀스, 자바 스트림 중 선택해 사용하자)
```kotlin
    books.asSequence()
                .map(Book::title)               // 중간연산1
                .filter { it.startswith("A") }  // 중간연산2
                .toList()                       // 최종연산
```
- 중간 연산은 최종 연산이 호출되기 전까지 lazy 된다.
- 최종연산인 `toList()` 가 없으면 기능이 수행되지 않는다.
- 원소 별로 순차적으로 수행 (첫번째 원소 map() → filter() , 두번째 원소...)
    - 컬렉션으로 함수 호출 했을 경우, map()에 대한 리스트 → filter에 대한 결과 리스트 생성

```kotlin
    listOf(1,2,3,4).asSequence()
            .map { it * it }.find { it > 3 }
```
[](https://www.notion.so/71a0601c7c3e4baa8f04c17c92183e3b#0e8fef9ee72e4f6c9a52045ee0c7d3b3)

- 최종연산과 중간연산 함수의 순서에 따라 수행 횟수가 달라질 수 있다.
    - 최종연산이 `toList` 일 경우 `filter` 를 먼저 호출하면 변환 횟수가 줄어든다.

### 시퀀스 만들기

- `generateSequence` : 이전 원소를 파라미터로 받아 다음 원소를 계산
```kotlin
    // 0부터 100까지 합 구하기
    generateSequence(0) { it + 1 }
            .takeWhile { it <= 100 }
            .sum() // 최종연산을 호출해야 비로소 수행
```
```kotlin
    // 상위 파일을 모두 조회하며 숨김 속성을 가졌는지 검사
    // File 클래스의 확장함수
    fun File.isInsideHiddenDirectory() =
            generateSequence(this) { it.parentFile }.any { it.isHidden }
```

- 객체의 조상으로 이뤄진 시퀀스가 필요할 때
- 조상과 자신과 같은 타입일 때 (`File`의 `parentFile` 도 `File` 타입이다)

## 5.4 자바 함수형 인터페이스 활용

- 코틀린 람다를 자바 API 에 활용하자
1. OnClickListener
```kotlin
    button.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View view) {
                    ...
            }
    })
```
```kotlin
    button.setOnClickListener { view ->
            ...
    }
```
- 추상메소드가 ***1개 (SAM)***일 경우 코틀린이 람다 식을 넣을 수 있도록 지원
    - SAM: Single Abstract Method

2. Runnable

- SAM을 구현한 람다를 넘길 경우 무명 인스턴스를 자동으로 만들어 준다
- 호출할 때마다 하나의 인스턴스를 재사용한다.
```kotlin
    // object로 무명 객체를 만들어 사용
    postponeComputation(1000, object : Runnalbe {
            override fun run() {
                    println(42)
            }
    })
```
```kotlin
    // 람다로 인스턴스 재사용
    postponeComputation(1000) {
            println(42)
    }
```
- Runnable에서 전달받은 변수를 사용한다면?
```kotlin
    fun handleComputation(id: String) {
            postponeComputation(1000) {
                    println(id)
            }
    }
```
- handleComputation 함수는 바로 끝나지만 Runnable에선 `id` 값을 유지해야 한다!
```kotlin
    class HandleComputation$1(val id: String): Runnable {
            override fun run() {
                    println(id)
            }
    }
```
- 위와 같이 `id` 값을 포획한 클래스를 새로 만들고, Runnable이 호출될 때 마다 이 클래스의 인스턴스를 새로 호출한다. 클래스 → 바이트코드 변환
- 자바 8부터 람다가 도입되었기 때문에, 이후에는 자브이 람다를 활용해 클래스 없이 바이트코드를 생성하게 바뀌었다.
- `inline` 키워드를 함수앞에 붙이면, 클래스와 인스턴스가 만들어지지 않고 바이트코드 자체로 넘어간다.

### SAM 생성자

- 메소드가 1개인 인터페이스를 구현하면서 인스턴스를 생성해주는 함수
- 컴파일러가 자동 생성한 함수 (함수형인터페이스의 이름과 같다)
- 람다 → 함수형 인터페이스의 인스턴스
- 보통 컴파일러가 자동으로 만들고 호출하지만 안될 경우 이 생성자를 사용해 인스턴스를 만들어야함
- 예: SAM의 인스턴스를 반환해야할 때, 인스턴스를 재사용 하고 싶을 때
```kotlin
    fun createAllDoneRunnable(): Runnable {     
            return Runnable {           // 이름으로 감싸야 인스턴스가 리턴된다.
                    println("all done")     
            }
    }
```
```kotlin
    val listener = OnClickListener {
            ....
    }
    
    button1.setOnClickListener(listener)
    button2.setOnClickListener(listener)
```
- 람다에는 `this` 가 없다.
- **람다에서의 `this` 는 그 바깥 클래스를 가리킨다.**
- `this` 가 필요한 경우 무명 객체로 구현해 사용할 것 (`object : OnClickListenr...`)

## 5.5 수신 객체 지정 람다: with과 apply

- 람다 본문안에서 다른 인스턴스의 메소드 호출

### with 함수

- 인스턴스의 이름을 반복하지 않고, 연산을 수행하자
```kotlin
    // with을 사용하지 않은 예제
    fun alphabet(): String {
            val result = StringBuilder()
            for (letter in 'A'..'Z') {
                    result.append(letter)
            }
            result.append("\nNow I know the alphabet!")
            return result.toString()
    }
```

- `result` 가 너무 많이 반복된다.
```kotlin
    // with 사용
    fun alphabet(): String {
            val stringBuilder = StringBuilder()
            return with (stringBuilder) {
                    for (letter in 'A'..'Z') {
                            this.append(letter)                // this 명시 가능
                    }
                    append("\nNow I know the alphabet!")   // this 생략 가능
                    this.toString()
            }
    }
```

- 1번째 파라미터: 인스턴스, 람다의 수신 객체로 사용
- 2번째 파라미터: 수행할 연산이 담긴 람다
- 확장함수도 수신 객체 지정 함수라고 볼 수 있다
```kotlin
    // 식이 본문인 함수로 리팩토링
    fun alphabet() = with (StringBuilder()) {
            for (letter in 'A'..'Z') {
                    append(letter)      
            }
            append("\nNow I know the alphabet!")
            toString()
    }
```

### apply 함수

- `with` 은 람다의 결과를 반환
- `apply` 는 수신 객체를 반환
- `apply()`  는 확장 함수이다.
```kotlin
    // apply 함수로 리팩토링
    fun alphabet() = StringBuilder().apply {
            for (letter in 'A'..'Z') {
                    append(letter)
            }
            append("\nNow I know the alphabet!")
    }.toString()
```

- 인스턴스를 생성하면서 그 인스턴스의 초기값을 설정할 때 유용하다.
```kotlin
    // android에서 textview를 생성하는 예제
    val tv = TextView().apply {
            text = "Sample text"
            textSize = 20.0
            setPadding(10, 0, 0, 0)
    }
```