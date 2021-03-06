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

## 7.4 구조 분해 선언과 component 함수
```kotlin
    val p = Point(10, 20)
    val (x, y) = p
```
- `Point` 를 `x` , `y` 로 분해해 호출하는 것
- `componentN` 함수에 대응된다
```kotlin
    val (a, b) = p
    
    val a = p.component1()
    val b = p.component2()

    class Point(val x: Int, val y: Int) {
        operator fun component1() = x
        operator fun component2() = y
    }
```
- 원소가 너무 많아지면 가독성이 떨어진다
- 코틀린은 맨 앞 **다섯 원소**에 대한 component 함수를 지원한다
```kotlin
    val x = listOf(1,2,3,4,5)
    val (a, b, c, d, e) = x
    
    // IndexOutOfBoundsException
    val y = listOf(1,2)
    val (f, g, h) = y
```
- 변수 선언이 들어갈 수 있는 어느 위치에 사용 가능 → 루프 안에서 유용
- 코틀린 표준 라이브러리에서 `map` 에 대한 `component1` , `component2` 를 제공하기 때문에 구조 분해 선언이 가능
```kotlin
    val map = mapOf("Oracle" to "Java", "JetBrains" to "Kotlin")
    for((key, value) in map) {
        println("$key -> $value")
    }
```

## 7.5 프로퍼티 접근자 로직 재활용: 위임 프로퍼티

- 필드를 DB 테이블, 브라우저 세션, 맵 등에 저장하는 등 복잡한 로직 구현 가능
- 인스턴스가 직접 작업을 수행하지 않고 다른 **도우미 인스턴스**(위임 객체)에 작업을 위임
```kotlin
    class Foo {
        var p: Type by Delegate()
    }
```

- `p` 접근 로직을 다른 인스턴스에 위임
- `Delegate()` 위임할 인스턴스
```kotlin
    // by 연산자를 통해 컴파일러가 만들어 내는 코드
    class Foo {
    
        private val delegate = Delegate()
    
        var p: Type
        set(value: Type) = delegate.setValue(..., value)
        get() = delegate.getValue(...)
    }
    
    // Delegate 클래스 구현
    class Delegate {
        operator fun getValue(...) { ... }
        operator fun setValue(..., value: Type) { ... }
    }
```

### 위임 프로퍼티 사용: `by lazy()` 를 사용한 초기화 지연

- 인스턴스의 일부분을 초기화하지 않고 있다가 실제로 그 부분의 값이 필요할 때 초기화
- 항상 초기화할 필요가 없는 필드에 유용
- Person이 작성한 이메일을 DB에서 최초 한번만 조회하는 상황
```kotlin
    // by lazy() 없이 backing property 로 초기화 지연 구현
    class Person(val name: String) {
    
        // emails 의 위임 프로퍼티 역할 (nullable)
        private var _emails: List<Email>? = null
    
        // not null
        val emails: List<Email>
        get() {
            // 최초 접근 시에 DB에서 이메일 조회
            if (_emails == null) {
                _emails = loadEmails(this)
            }
            return _emails!!
        }
    }
```
- `_emails` : 값 저장 (mutable = `var`), null 가능
- `emails` : 읽기 연산 제공( = `val`), not null
- 이런 구현을 해야할 상황이 많지만 약간 귀찮다
```kotlin
    // by lazy() 로 초기화 지연 구현
    class Person(val name: String) {
        val emails: List<Email> by lazy { loadEmails(this) }
    }
```
- `lazy()` 는 `getValue()`  의 객체를 반환한다. 람다는 초기화할 때 호출.
- Thread safe

### 위임 프로퍼티 컴파일 규칙
```kotlin
    class C {
        var prop: Type by MyDelegate()
    }
    
    val c = C()

    // 컴파일러가 만들어주는 코드
    class C {
        private val <delegate> = MyDelegate()
        var prop: Type
        get() = <delegate>.getValue(this, <property>)
        set(value: Type) = <delegate>.setValue(this, <property>, value)
    }
```

### 위임 프로퍼티 구현

- 객체 필드의 값이 바뀌면 UI도 업데이트
- 자바의 `PropertyChangeSupport` (리스너 목록 관리), `PropertyChangeEvent` (이벤트가 들어오면 리스너에 통지)
- 단계별로 리팩토링 해보자
```kotlin
    // 리스너 등록/해지
    open class PropertyChangeAware {
        protected val changeSupport = PropertyChangeSupport(this)
        fun addPropertyChangeListener(listener: PropertyChangeListener) {
            changeSupport.addPropertyChangeListener(listener)
        }
        fun removePropertyChangeListener(listener: PropertyChangeListener) {
            changeSupport.removePropertyChangeListener(listener)
        }
    }
```
- 사람의 나이와 급여가 바뀔 때 리스너에게 이벤트 통지
```kotlin
    class Person(
        val name: String, age: Int, salary: Int): PropertyChangeAware() {
        
        var age: Int = age
        set(newValue) {
            val oldValue = field
            field = newValue
            // 리스너에게 이벤트 통지
            changeSupport.firePropertyChange("age", oldValue, newValue)
        }
    
        var salary = salary
        set(newValue) {
            val oldValue = field
            field = newValue
            // 리스너에게 이벤트 통지
            changeSupport.firePropertyChange("salary", oldValue, newValue)
        }
    }

    // 화면에서 사용할 때
    val p = Person("Dmitry", 34, 2000)
    p.addPropertyChangeListener(
        PropertyChangeListener { event ->
            // ... do something
            // UI 갱신
        }
    )
    p.age = 35
    p.salary = 2100
```
### 리팩토링 step1.

- 세터코드의 중복이 있다
- 프로퍼티의 값을 저장하고 필요에 따라 통지해주는 클래스를 분리
```kotlin
    // 프로퍼티 저장 + 이벤트 통지
    class ObservableProperty(
        val propName: String, var propValue: Int,
        val changeSupport: PropertyChangeSupport
    ) {
        fun getValue(): Int = propValue
        fun setValue(newValue: Int) {
            val oldValue = propValue
            propValue = newValue
            changeSupport.firePropertyChange(propName, oldValue, newValue)
        }
    }
    
    class Person(
        val name: String, age: Int, salary: Int
    ) : PropertyChangeAware() {
    
        val _age = ObservableProperty("age", age, changeSupport)
        var age: Int
            get() = _age.getValue()
            set(value) { _age.setValue(value) }
    
        val _salary = ObservableProperty("salary", salary, changeSupport)
        var salary: Int
            get() = _salary.getValue()
            set(value) { _salary.setValue(value) }
    }
```
### 리팩토링 step2.

- 필드 각각 `ObservableProperty` 만들고, 게터, 세터를 구현해야함
- 필드 각각 `backing property` 필요 → 위임 프로퍼티를 사용하자
```kotlin
    class ObservableProperty(
        var propValue: Int, val changeSupport: PropertyChangeSupport
    ) {
        operator fun getValue(p: Person, prop: KProperty<*>): Int = propValue
    
        operator fun setValue(p: Person, prop: KProperty<*>, newValue: Int) {
            val oldValue = propValue
            propValue = newValue
            changeSupport.firePropertyChange(prop.name, oldValue, newValue)
        }
    }
    
    class Person(
        val name: String, age: Int, salary: Int
    ) : PropertyChangeAware() {
    
        var age: Int by ObservableProperty(age, changeSupport)
        var salary: Int by ObservableProperty(salary, changeSupport)
    }
```
- `operator fun` 으로 getValue, setValue 함수 재정의
- `KProperty` : 코틀린이 프로퍼티를 표현하는 방법. 리플렉션 사용해 [`Kproperty.name`](http://kproperty.name) 으로 해당 프로퍼티 접근 (10장)
- `KProperty` 사용했으므로 주생성자의 `propName` 삭제

### 코틀린 표준라이브러리 사용

- `ObservableProperty` 말고 코틀린이 제공해주는 클래스가 있다.
- `PropertyChangeSupport` (이벤트 통지/리스너관리) 와 연결되어있지 않아서 `PropertyChangeSupport`  를 사용하는 방법을 알려주는 람다를 함께 넘겨줘야 한다.
```kotlin
    class Person(
        val name: String, age: Int, salary: Int
    ) : PropertyChangeAware() {
    
        private val observer = {
            prop: KProperty<*>, oldValue: Int, newValue: Int ->
            changeSupport.firePropertyChange(prop.name, oldValue, newValue)
        }
    
        var age: Int by Delegates.observable(age, observer)
        var salary: Int by Delegates.observable(salary, observer)
    }
```