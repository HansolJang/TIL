- 함수, 변수, 클래스, enum, 프로퍼티
- 제어 구조
- 스마트 캐스트
- 예외 던지기와 예외 잡기

# 2.1 기본 요소: 함수와 변수

코틀린이 어떻게 불변 데이터 사용을 장려하는지 알아본다.

## Hello, World!
```
    fun main(args: Array<String>) {
    	println("Hello, World!")
    }
```

- fun 으로 함수 정의
- 파라미터 이름: 파라미터 타입의 순서로 쓴다.
- 배열도 일반적인 클래스다. (자바의 []와 같이 불편하게 따로 처리할 필요가 없다)
- 코틀린 표준 라이브러리가 자바의 여러 표준 라이브러리를 감싸기 때문에 System.out.println → println으로 간결하게 사용한다.
- 줄 끝에 세미콜론(;)을 붙이지 않는다.

## 함수
```
    // a, b를 인자로 Int를 리턴하는 max 함수 정의
    fun max(a: Int, b: Int): Int {
    	// java의 삼항 연산자와 비슷하다 (a > b) ? a : b
    	return if (a > b) a else b 
    }
```

- 괄호와 반환 타입 사이를 콜론(:)으로 구분
- 코틀린 if: 문장이 아닌 **식**이다! (결과를 만들어 낸다)
- 문(statement)과 식(expression)의 구분
    - 식: 값을 만들어 냄. 다른 식의 하위 요소로 계산에 참여 (코틀린에서는 루프를 제외한 대부분의 제어 구조가 식이다, 간결한 표현이 가능하다)
    - 문: 블록을 묶기만 하며 값을 만들어 내지 못함

## 식이 본문인 함수

```
    fun max(a: Int, b: Int): Int = if(a > b) a else b
```

- 코틀린에선 위와 같이 식이 본문인 함수가 자주 쓰인다
- if, when, try 등 더 복잡한 제어 구조를 사용한다
- 반환 타입을 생략할 수 있다 → **컴파일러가 타입 추론**을 해준다
```
    fun max(a: Int, b: Int) = if(a > b) a else b
```

- `식이 블록인 함수는 타입을 추론할 수 없다.` 블록이 식이 아닌 문이기 때문이다 (return으로 명시해야한다)

## 변수

- 타입을 추론해준다 (초기화하는 것을 보고 추론)
- 변수를 선언한다는 것을 알기 위해 맨 앞에 키워드`val`, `var`을 명시한다
```
    // 타입 명시
    val answer: Int = 42
    // 타입 추론
    val answer = 42
    // 초기화하지 않아서 타입 추론할 수 없으므로 에러
    val answer
```

## mutable과 immutable 변수

1. **val (value)**

    변경 불가능한 참조를 저장 (재대입 불가능) (자바의 final) 

2. **var (varible)** 

    변경 가능한 참조를 저장

기본적으로 *모든 변수를 val* 로 선언하고, 나중에 꼭 필요할 때만 var을 사용할 것을 권고한다.

코드가 함수형에 가까워진다.
```
    // 조건에 따라 다르게 초기화할 수 있다
    val message: String
    
    if(canPerformOperation()) {
    	message = "Success"
    // ...
    } else {
    	message = "Failed"
    }

    // 참조 자체는 불변이지만, 내부는 수정가능하다
    val languages = arrayListOf("Java")
    languages.add("Kotlin")
```

## 문자열 템플릿
```
    fun main(args: Array<String>) {
    	val name = if (args.size > 0) args[0] else "Kotlin"
    	println("Hello, ${name}!")
    }
```

- 자바의 "Hello, " + name + "!" 보다 더 간결하다
- `${if(agrs.size > 0) args[0] else "${name}"}`과 같이 중괄호 안에 다양한 식을 넣을 수 있다.

# 2.2 클래스와 프로퍼티

- 자바보다 더 적은 양의 코드로 클래스와 관련된 대부분의 작업을 수행할 수 있다
```
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

## 프로퍼티

클래스 → 데이터를 캡슐화하자 → 데이터를 클래스안에 가두자

프로퍼티: 클래스 내의 필드와 접근자(getter/setter)를 묶어 부르는 이름

- 자바에서는 데이터를 캡슐화하기 위해 private으로 하고, getter/setter를 사용한다
- 코틀린은 기본 기능으로 제공한다
```
    class Person(
    	val name: String,         // 읽기 전용으로 필드(비공개)와 게터(공개) 생성
    	var isMarried: Boolean)   // 쓸 수 있는 필드(비공개)와 게터/세터(공개) 생성
```

- 자바는 클래스 외부에서 호출시 `person.getName()` 으로 호출하지만 코틀린은 `person.name` 으로 코드가 더 간결해진다

## 커스텀 접근자

```
    class Rectangle(val height: Int, val width: Int) {
        val isSquare: Boolean
            get() = height == width    // 호출시마다 새로 계산된 값을 리턴
    }
```

- 자체 값을 저장할 필드가 필요 없다.

## 코틀린 소스코드 구조: 디렉터리와 패키지

- `package`: 현재 작성하는 클래스와 함수 등 모두가 해당 패키지에 속한다.
- `import`: 다른 패키지를 사용하려면 임포트를 통해 선언을 불러와야 한다.

# 2.3 선택 표현과 처리: enum과 when

```
    enum class Color(
            val r: Int, val g: Int, val b: Int) {
        RED(255, 0, 0), ORANGE(255, 165, 0),
        YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255),
        INDIGO(75, 0, 130), VIOLET(238, 130, 238);
    
        fun rgb() = (r * 256 + g) * 256 + b
    }
    
    fun main(args: Array<String>) {
        println(Color.BLUE.rgb())
    }
```

- enum 클래스에도 프로퍼티와 메소드를 정의할 수 있다
- 상수 선언이 끝날 때 세미콜론`;`을 필수로 명시해야한다

```
    fun getWarmth(color: Color) = when(color) {
        Color.RED, Color.ORANGE, Color.YELLOW -> "warm"
        Color.GREEN -> "neutral"
        Color.BLUE, Color.INDIGO, Color.VIOLET -> "cold"
    }
```

- when 도 식이기 때문에 **식이 본문인 함수**로 정의가 가능하다.
- break를 명시하지 않아도 된다.
- 코틀린 when은 분기 조건에 상수나 숫자, 리터럴 뿐 아니라 **객체**를 허용한다.
- 인자가 없다면 조건이 불리언 결과를 계산하는 식이어야한다.

```
    // 인자가 없는 when
    fun mixOptimized(c1: Color, c2: Color) =
        when { 
            (c1 == RED && c2 == YELLOW) ||
            (c1 == YELLOW && c2 == RED) -> ORANGE
    
            (c1 == YELLOW && c2 == BLUE) ||
            (c1 == BLUE && c2 == YELLOW) -> GREEN
    
            (c1 == BLUE && c2 == VIOLET) ||
            (c1 == VIOLET && c2 == BLUE) -> INDIGO
    
            else -> throw Exception("Dirty color")
        }
```

## 스마트 캐스트: 타입 검사와 타입 캐스트를 조합

`(1 + 2) + 4`와 같은 간단한 산술식을 계산하는 함수를 만들어보자.

```
    interface Expr
    class Num(val value: Int) : Expr
    class Sum(val left: Expr, val right: Expr) : Expr
```

```
    printf(eval(Sum(Sum(Num(1), Num(2)), Num(4))))
```

- Expr이 수라면 그 수를 반환
- Expr이 합계라면 좌항, 우항을 더한 값을 반환
- **코틀린은 is 로 타입 겁사를 하고 나면, 자동으로 해당 타입으로 캐스팅해준다.**
```
    // if 로 구현
    // Kotlin if는 자체로 값을 반환하기 때문에 return을 쓰지 않는다
    fun eval(e: Expr): Int =
        if (e is Num) {
            e.value
        } else if (e is Sum) {
            eval(e.right) + eval(e.left)
        } else {
            throw IllegalArgumentException("Unknown expression")
        }

    // when 으로 구현
    // 구현체 내용이 길어지면 { } 블록을 사용한다
    fun eval(e: Expr): Int =
        when (e) {
            is Num -> {
    						... 다른 일 수행
                e.value   // 블록의 마지막 식이 블록의 결과로 반환
    				}
            is Sum ->
                eval(e.right) + eval(e.left)
            else ->
                throw IllegalArgumentException("Unknown expression")
        }
```


## 2.4 대상을 이터레이션: while과 for 루프

자바와 둘다 구조가 비슷하다.

### while
```
    while (조건) {
        /* ... */
    }
    
    do {
        /* ... */
    } while (조건)
```

### 수에 대한 이터레이션

- 여타 언어들에선 `for(초기값; 최종값; 증가값)`과 비슷한 형식의 반복문을 사용
- kotlin에서는 **범위**를 사용
```
    // 1, 10 모두 포함!
    val oneToTen = 1..10
    
    // 1부터 100까지 증가
    for (i in 1..100) { /*...*/ }
    
    // 100부터 -2씩 1까지 감소 
    for (i in 100 downTo 1 step 2) { /*...*/ }
    
    // 0부터 size-1 까지
    for (i in 0 until size) { /*...*/ }
```

### 맵에 대한 이터레이션
```
    // 문자도 범위로 사용 가능
    for (i in 'A'..'F') { /*...*/ }
    
    val map = TreeMap<Char, String>()
    // map도 key,value 쌍으로 가능
    for((letter, binary) in map) { /*...*/ }
    
    val list = arrayListOf("10", "11", "1001")
    // arraylist도 index, 원소 쌍으로 이터레이션 가능
    for((index, element) in list.withIndex()) { /*...*/ }
```

### in으로 원소 검사
```
    fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
    
    fun isNotDigit(c: Char) = c !in '0'..'9'
```

- 코틀린 표준 라이브러리가 `'a' ≤ c && c ≤ 'z'`로 변환
- `Comparable` interface를 구현한 클래스라면 모두 범위 연산이 가능하다

    // 이터레이션 불가!
    // Comparable을 구현했더라도 아래와 같이 모든 원소에 대해서 항상 이터레이션 하지는 못한다.
    for(string in "Java".."Kotlin") {}
    
    // 하지만 범위 확인은 가능!
    if("Kotlin" in "Java".."Scala") {}

- 책의 7.3.2절에서 내가 만든 데이터타입을 활용해 in을 사용하는 일반 규칙을 알아본다.

## 2.5 코틀린의 예외 처리

- 함수에서 예외를 throw 할 수 있다.
- 함수를 호출하는 쪽에서 예외를 잡아 처리할 수 있다.
- 예외를 catch하지 않으면 함수 호출 스택을 거슬러 올라가면서 처리할 수 있을 때까지 예외를 다시  throw한다.
- `throw 도 식이다.`

```
    // 식이 본문인 함수
    // if가 식이므로 데이터를 리턴
    // throw도 식이므로 예외를 리턴
    val percentage = 
            if (number in 0..100)
                    number
            else 
                    throw IllegalArgumentException()

    // 함수를 선언할 때 명시
    // 함수를 호출할 때도 try, catch 문을 강제
    int readNumber(BufferedReader reader) throws IOException { ... }

    // 예외를 명시하지 않음
    // 함수 호출에서도 try,catch는 선택
    fun readNumber(reader: BufferedReader): Int? {
            ...
            throw IOException()
    }
```

- try, catch 도 식이다.
- try는 무조건 중괄호로 {} 감싸야한다.

```
    fun readNumber(reader: bufferedReader) {
            val number = try {
                    Integer.parseInt(reader.readLine())
            } catch (e: NumberFormatException) {
                    // return 은 함수를 종료하므로, 예외가 발생할 경우 이후의 코드가 실행되지 않는다.
                    return
                    // null이 number에 대입되므로, 이후에 null이 출력된다.
                    null
            }
            println("$number")
    }
```