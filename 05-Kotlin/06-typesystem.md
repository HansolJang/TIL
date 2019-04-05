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