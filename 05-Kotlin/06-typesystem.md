- null 이 될 수 있는 타입과 null 을 처리하는 구문의 문법
- 코틀리 원시 타입 소개와 자바 타입과의 관계
- 코틀린 컬렉션 소개와 자바 컬렉션과의 관계

## 6.1 널 가능성 (Nullability)

- NPE을 피할 수 있게 돕기 위한 코틀린 타입 시스템의 특성
- null 이 될 수 있는 값을 어떻게 표기하는가
- 코틀린이 그 값을 어떻게 처리하는가
- 코틀린과 자바 코드를 어떻게 함께 사용할 수 있는가

### 널이 될 수 있는 타입

> Type? = Type 또는  null

- 타입 지정시 해당 타입에 `null` 이 들어올 수 있는가?를 고민해봐야 한다
- 기본적으로는 널이 될 수 없는 타입이다.
- 널이 될 수 있는 타입은 메소드를 직접 호출할 수 없다.
- 널이 될 수 있는 타입을 널이 될 수 없는 타입에 대입할 수 없다. (파라미터 또한 대입 X)
```kotlin
    fun strLenSafe(s: String?): String? = s?.length   // s.length 처럼 직접 호출 불가
    
    val x: String? = null
    val y: String = x     // 불가
```

### 안전한 호출 연산자 ?.

- null 검사 + 메소드 호출
```kotlin
    s?.toUpperCase()
    
    if (s!= null) {
    		s.toUpperCase()
    } else {
    		null
    }
```

- null 일 경우 null이 반환된다
- 연쇄 호출 가능
```kotlin
    this?.company?.address?.country ?: "unknown"
```

### 엘비스 연산자 ?:

- null 대신 사용할 디폴트 값 지정
```kotlin
    val t: String = s ?: ""
    
    if (s == null) {
    		""
    } else {
    		s
    }
    
    fun strLenSafe(s: String?): Int = s?.length ?: 0
```

- 우항에 `return` 또는 `throw`  도 넣을 수 있다
```kotlin
    this.company?.address ?: throw IllegalArgumentException("No Address")
```

### 안전한 캐스트 as ?

- 타입을 캐스팅할 수 없다면 null 을 반환한다.
```kotlin
    class Person {
    		...
    		override fun equals(o: Any?): Boolean {
    				val otherPerson = o as? Person ?: return false
    				return otherPerson.firstName == this.firstName && 
    									otherPerson.lastName == this.lastName
    		}
    }
    
    if(o is Person) {
    		(Person)o.firstName
    } else null
```

### 널 아님 단언 !!

- 널이 될 수 없는 타입으로 강제로 바꿔 준다.
- 개발자가 잘못 생각했다면 예외가 발생해도 감수 하겠다.
```kotlin
    fun ignoreNulls(s: String?) {
    		val sNotNull: String = s!!      // NPE가 나는 지점
    	  println(sNotNull.length)
    }
```

- 널인지 검사했지만 컴파일러는 알 지 못함
- 반복적으로 널 검사한 변수를 사용해야 할때 유용
```kotlin
    if (s != null) {
    		s!!.toUpperCase()
    		s!!.length
    }

    // 연쇄 호출은 피해야 한다.
    // 어느 식에서 예외가 발생했는지 모른다
    person.company!!.address!!.country
```

### let 함수

- 널 검사 + 널 검사한 결과 인스턴스 사용
- null 이면 실행되지 않는다
```kotlin
    // 람다는 email 을 수신 객체로 받아 it 키워드로 접근한다.
    // email이 널이 아닌 경우에만 호출
    email?.let {
    		sendEmailTo(it)		
    }
    
    if(email != null) {
    		sendEmailTo(email!!)
    }
```

### 나중에 초기화할 프로퍼티

- 예: 안드로이드의 onCreate(), JUnit의 @Before
- 하지만 코틀린에서는 프로퍼티 초기화를 무조건 생성자에서 해야 한다
```kotlin
    class MyTest {
    		private var myService: MyService? = null
    		@Before fun setUp() {
    				myService = MyService()
    		}
    
    		@Test fun test() {
    				myService!!.performAction()
    				myService?.performAction()
    		}
    }
```

`lateinit`

- `var` 만 가능하다. `val` 은 final 이므로 생성시 초기화 필수
- 초기화하기 전에 접근시 `lateinit property myService has not been initialized` 예외 발생
```kotlin
    class MyTest {
    		private lateinit var myService: MyService
    		@Before fun setUp() {
    				myService = MyService()
    		}
    
    		@Test fun test() {
    				myService.performAction()
    		}
    }
```

### 널이 될 수 있는 타입 확장

- `?.` 로 안전한 호출을 하지 않아도 확장 함수가 알아서 널을 처리해준다
```kotlin
    val input: String? = ""
    
    // input이 null인지 " " 공백으로만 이루어진 문자인지 판별
    input.isNullOrBlank()
    
    // input이 null인지 "" 빈 문자열인지 판별
    input.isNullOrEmpty()

    fun String?.isNullOrBlank(): Boolean =
    		this == null || this.isBlank()         // 두번째 this에는 스마트 캐스트 적용
```

- 자바의 경우에는 `this` 로 `null`을 받지 못하고 NPE 발생
- 안전한 호출을 하지 않아도 해당 프로퍼티는 `nullable` 할 수 있다

### 타입 파라미터의 널 가능성

- 함수나 클래스 타입은 기본적으로 `nullable`
```kotlin
    fun <T> printHashCode(t: T) {    // t는 null이 될 수 있다
    		println(t?.hashCode())
    }
```

null 이 가능하지 않도록 하려면?
```kotlin
    fun <T: Any> printHashCode(t: T) {
    		println(t.hashCode())
    }
```

- 제네릭스에 대해선 9장에서 본다

### 널 가능성과 자바

- 자바의 어노테이션을 활용하자
```kotlin
    @Nullable String     // String?
    
    @NotNull String      // String
```

- 활용할 수 있는 어노테이션이 없다면 코틀린의 플랫폼 타입이 된다.
- 플랫폼 타입
```kotlin
    public class Person {
    		private final String name;
    		public Person(String name) {
    				this.name = name;
    		}
    		public String getName() {
    				return name;
    		}
    }
```

`getName()` 은 `null` 을 리턴할 수 있다!
```kotlin
    // 자바의 플랫폼 타입을 코틀린으로 가져와서 사용할 때
    val s: String? = person.name
    val s1: String = person.name
```
그러므로 Person 클래스를 코틀린에서 사용할 땐 널 검사를 철저히 해야한다.

자바 API 에는 대부분이 널관련 어노테이션을 쓰지 않으므로 주의해야한다. → 런타임 에러!

- 상속

## 6.2 코틀린의 원시 타입

- 자바처럼 원시 타입과 래퍼 타입을 구분하지 않는다
    - 자바에서는 원시 타입을 제네릭 등에 사용하기 위해 클래스로 한번 감싼 래퍼 클래스를 제공

### 원시 타입: Int, Boolean 등

- 자바에서의 원시 타입은 그 값을 직접 갖고 있지만, 참조 타입은 메모리상의 주소를 갖고 있다.
- 코틀린에서는 이를 구분하지 않는다
- 기본적으로 자바의 `int` 로 컴파일되고, 컬렉션/제네릭과 같이 사용될 때만 `Integer` 로 컴파일된다.
- 원시 타입은 결코 널이 될 수 없으므로 코틀린에서도 널이 될 수 없는 타입으로 취급
- 자바에 대응되는 코틀 원시 타입
    - 정수 타입 `Byte` , `Short` , `Int` , `Long`
    - 부동소수점 수 타입 `Float` , `Double`
    - 문자 타입 `Char`
    - 불리언 타입 `Boolean`

### 널이 될 수 있는 원시 타입: Int?, Boolean? 등
```kotlin
- 널이 될 수 있는 코틀린의 원시 타입 = 자바의 래퍼 타입 (자바의 원시 타입은 널이 될 수 없으므로)

    class Person(val name: String,
                                val age: Int?) {
            fun isOlderThan(otherPerson: Person): Boolean? {
                    if (this.age == null || otherPerson.age == null)
                            return null
                    else this.age > otherPerson.age
            }
    }
```
- 제네릭의 경우 자바의 래퍼 타입 사용
- JVM은 제네릭의 파라미터로 원시 타입을 허용하지 않는다
```kotlin
    // 자바의 List<Integer> 와 동일
    val listOfInts = listOf(1, 2, 3)
```
### 숫자 변환

- 코틀린
    - 한 타입의 숫자를 다른 타입의 숫자로 자동 변환하지 않는다
    - 대신 변환할 수 있는 함수를 제공 `toInt()`, `toByte()` , `toChar()` 등
```kotlin
    // 형 변환 에러! int를 자동으로 long으로 바꿔주지 않는다
    val i = 1
    val l: Long = i
    
    val l = i.toLong()
```
- 박스 타입의 `equals()` 는 값이 아니라 객체를 비교한다
```java
    // false!
    new Integer(42).equals(new Long(42))
```
- 코틀린은 혼란을 피하기 위해 개발자가 명시적으로 형변환을 해줘야한다.
```kotlin
    val x = 1
    val list = listOf(1L, 2L, 3L)
    
    // 컴파일 되지 않음, 묵시적 형 변환 불가
    println(x in list)
    
    // true
    println(x.toLong() in list)
```
- 코틀린의 원시 타입 리터럴
    1.  `L` 접미사: Long 타입 `123L`
    2. 표준 부동소수점 표기법: Double 타입 `0.12` , `1.2e10` 
    3. `f` 또는 `F` 접미사: Float 타입  `123.4f`
    4. `0x` 또는 `0X` 접두사: 16진법 리터럴 `0xCAFEBABE`
    5. `0b` 또는 `0B` 접두사: 2진 리터럴 `0b0000101` 

### Any, Any?:  최상위 타입

- 자바의 `Object` 와 동일
- 모든 널이 될 수  없는 타입의 조상
- 자바의 `Object` 는 참조 타입만 감싸지만, 코틀린의 `Any` 는 원시 타입도 받을 수 있다
```kotlin
    val answer: Any = 42
```
### Unit 타입: 코틀린의 void

- 아무것도 반환하지 않는 함수의 리턴 타입
- `Unit` 을 타입 인자로 제네릭에 사용할 수 있다 (`void` 와 차이점)
- 제네릭을 사용하지만 아무것도 반환하고 싶지 않을 때 유용
```kotlin
    interface Processor<T> {
            fun process: T
    }
    
    class NoResultProcessor : Processor<Unit> {
            override fun process() {    // 반환형이 Unit이므로 명시하지 않음
                    ...
            }
    }
```
- 자바의 경우 `void` 를 사용하면 `return null` 을 함수의 마지막에 무조건 써야함

### Nothing 타입: 이 함수는 절대 정상적으로 끝나지 않는다

- 결코 성공적으로 값을 돌려줄 일이 없는 함수
    - 테스트 라이브러리의 `fail()`
    - 무한 루프를 도는 함수
```kotlin
    fun fail(message: String): Nothing {
            throw IllegalException(message)
    }
```
- `Nothing` 은 아무 값도 포함하지 않는다
- 리턴 타입 or 리턴 타입에 쓰일 파라미터에만 쓸 수 있다
- 그 외 사용해봤자 아무 값도 저장되지 않는다
- 컴파일러는 결코 정상 종료되지 않음을 미리 알고 함수 호출을 분석할 때 `Nothing` 을 사용한다

## 컬렉션과 배열

### 널 가능성과 컬렉션

- 컬렉션을 정의할 때 `원소의 널 가능성`, `컬렉션의 널 가능성`에 대해 고민해야함
```kotlin
    val list: List<Int> = List()
    
    val list2: List<Int>? = null
    
    val list3: List<Int?> = List()
    
    val list4: List<Int?>? = null
```
- 원소가 널이 될 수 있는 타입일 경우, `filterNotNull` 함수가 유용하다
```kotlin
    fun addValidNumbers(numbers: List<Int>) {
            val sum = numbers.filterNotNull().sum()
    }
```
### 읽기 전용과 변경 가능한 컬렉션

- 컬렉션 안의 데이터에 `접근` / 컬렉션 안의 데이터를 `변경` 을 분리
- `val` 과 `var` 와 같은 맥락
- `kotlin.colletions.Collection` 은 이터레이션에 대한 함수만 있고, 추가/제거는 없음
    - size
    - iterator()
    - contains()
- `kotlin.collections.MutableCollection`
    - add()
    - remove()
    - clear()
- 읽기전용 컬렉션이라 하더라도 클래스에 따라 내부에 변경 가능 컬렉션을 갖고 있을 수 있다! 항상 `thread safe` 한 것이 아님을 명심

### 코틀린 컬렉션과 자바

[](https://www.notion.so/07c7cdd59243456492396350853d57c8#7785b932b21a499b8a82ee493a0bdd2a)

[컬렉션 생성 함수](https://www.notion.so/fe8e8df2510f4348bb957ee40efd8976)

- setOf, mapOf 를 사용하더라도 자바로 컴파일할 땐 읽기/쓰기 모두 가능한 util 클래스로 컴파일