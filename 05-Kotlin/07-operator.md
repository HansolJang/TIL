- 연산자 오버로딩
- 관례: 여러 연산을 지원하기 위해 특별한 이름이 붙은 메소드
- 위임 프로퍼티

7장에서 계속 사용할 예제 클래스

- 2차원 상의 위치를 표현하는 `Point` 클래스

    data class Point(val x: Int, val y: Int)

## 7.1 산술 연산자 오버로딩

- 자바
    - 산술 연산에 `+`, `-`, `/`, `%` , `*` 등 사용 가능
    - String 에만 + 연산자 사용 가능

### 이항 산술 연산 오버로딩

- `fun` 앞에 `operator` 를 붙여야한다
- 확장함수로도 선언 가능 `operator fun Point.plus(other: Point)`
```kotlin
    data class Point(val x: Int, val y: Int) {
    	operator fun plus(other: Point): Point {
    		return Point(x + other.x, y + other.y)
    	}
    }
    
    val p1 = Point(10, 20)
    val p2 = Point(20, 30)
    println(p1 + p2)
    println(p1.plus(p2))
```
[오버로딩이 가능한 이항 산술 연산자](https://www.notion.so/d4d122fffe4242d9b2d6f82de909eea6)

- 오버로딩이 가능하다
    - 여러 파라미터를 받을 수 있다
- `<<` , `&` 와 같은 비트 연산자는 오버로딩할 수 없다.

### 복합 대입 연산자 오버로딩

`+=` , `-=` 와 같은 연산자
```kotlin
    operator fun <T> MutableCollection<T>.plusAssign(element: T) {
    		this.add(element)
    }
    
    val numbers = ArrayList<Int>()
    numbers += 42
```
### 단항 연산자 오버로딩

`-a` 와 같이 한 항에 대해 연산하는 연산자도 오버로딩할 수 있다
```kotlin
    operator fun Point.unaryMinus(): Point {
    	return Point(-x, -y)
    }
    
    val p = Point(10, 20)
    println(-p)
```

[오버로딩이 가능한 단항 산술 연산자](https://www.notion.so/971843ae975744428bfcb3fb811ae705)

## 7.2 비교 연산자 오버로딩

### 동등성 연산자: equals

코틀린 `==` = 자바 `equals()` → 값을 비교
```kotlin
    a == b
    // a가 널일 땐 b도 널이어야만 true
    a?.equals(b) ?: (b == null)

    // data class 의 경우 자동 생성되는 equals() 함수의 구현
    class Point(val x: Int, val y: Int) {
        override fun equals(obj: Any?): Boolean {
            if (obj === this) return true      // 자바의 == 연산과 같다. 같은 객체인지 확인
            if (obj !is Point) return false    // Point 타입인지 확인
            return obj.x == x && obj.y == y    // 스마트 캐스트, 인스턴스의 값비교
        }
    }
```
- `===` (객체와 타입까지 비교) 연산자는 오버로딩하지 못한다
- `equals()` 는 `Any` 클래스에 `operator fun` 으로 선언되어 있다. (확장함수보다 우선순위가 높으므로 확장함수로 선언할 수 없다)
- `!=` 연산자도 내부에서 `equals()` 를 호출한다

### 순서 연산자: compareTo

- 비교 연산자 (>,<,≤,≥) 는 `compareTo` 로 컴파일된다
```kotlin
    a >= b
    
    a.compareTo(b) >= 0
```
- `p1 < p2` = `p1.compareTo(p2) < 0`
```kotlin
    class Person(
            val firstName: String, 
            val lastName: String): Comparable<Person> {
        override fun compareTo(other: Person): Int {
            return compareValuesBy(this, other, 
                Person::lastName, Person::firstName)  // 성이 같으면 이름도 검사
        }
    }

    val p1 = Person("Alice", "Smith")
    val p2 = Person("Bob", "Johnson")
    // false!
    println(p1 < p2)
```

## 7.3 컬렉션과 범위에 대해 쓸 수 있는 관례

- 인덱스로 원소 조회 또는 쓰기 → `a[b]` 각괄호로 인덱스 접근 (인덱스 연산자)
- 값이 컬렉션에 있는지 확인 → `in` 연산자

### 인덱스로 원소에 접근: get과 set

`operator` 함수로 `get()` 을 선언하면 인덱스 연산자를 사용할 수 있다
```kotlin
    p[0]
    
    p.get(0)

    data class Point(val x: Int, val y: Int)
    
    operator fun Point.get(index: Int): Int {
        return when(index) {
            0 -> x
            1 -> y
            else ->
                throw IndexOutOfBoundsException("Invalid coordinate $index")
        }
    }
    
    fun main(args: Array<String>) {
        val p = Point(10, 20)
        println(p[1])
    }
```

`operator` 함수로 `set()` 을 선언하면 인덱스 연산자로 컬렉션에 쓸 수 있다
```kotlin
    data class MutablePoint(var x: Int, var y: Int)
    
    // 인덱스 파라미터와 바꿀 값
    operator fun MutablePoint.set(index: Int, value: Int) {
        when(index) {
            0 -> x = value
            1 -> y = value
            else ->
                throw IndexOutOfBoundsException("Invalid coordinate $index")
        }
    }
    
    fun main(args: Array<String>) {
        val p = MutablePoint(10, 20)
        p[1] = 42
        println(p)
    }
```

### in 관례

- 객체가 컬렉션에 들어가 있는지 검사 = `contains` 와 대응
```kotlin
    a in c
    
    c.contains(a)
```

### rangeTo  관례

- `..` 연산자는 `rangeTo` 에 대응된다
```kotlin
    start..end
    
    start.rangeTo(end)
    
    // in 연산자로 검사할 수 있는 범위를 반환
    operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>

    val now = LocalDate.now()
    val vacation = now..now.plusDays(10)
    println(now.plusWeeks(1) in vacation)
```
### for를 위한 iterator 관례

- 컬렉션에서 `iterator` 를 얻은 다음 `hasNext` 와 `next` 를 호출
- `iterator()` 도 확장 함수로 정의 가능
- Comparable에 rangeTo가 포함되어 있다
```kotlin
    operator fun CharSequence.iterator(): CharIterator
    
    for(c in "abc") {
        ...
    }

    // LocalDate의 커스터마이징된 이터레이터
    operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
        object : Iterator<LocalDate> {
            
            var current = start
    
            override fun hasNext() = 
                current <= endInclusive
            
            override fun next() = current.apply {
                current = plusDays(1)
            }
    }
    
    
    val today = LocalDate.now()
    val after1Month = today.plusMonths(1)
    for(day in today..after1Month) {
        ...
    }
```