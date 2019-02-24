코틀린 코드를 작성하고 실행해볼 수 있는 사이트

try.kotl.in

[https://try.kotlinlang.org/#/Kotlin in Action/Chapter 1/1.1/1.1_ATasteOfKotlin.kt](https://try.kotlinlang.org/#/Kotlin%20in%20Action/Chapter%201/1.1/1.1_ATasteOfKotlin.kt)

## 코틀린 목적

현재 자바가 사용되고 있는 모든 용도에 더 **간결**하고 **생산**적이며 **안전**한 대체 언어를 제공하자

## 특징1. 정적 타입 지정 언어

컴파일 시점에 모든 프로그램 구성 요소의 타입을 알 수 있다 (빠르다)

컴파일러가 메소드나 필드의 타입을 검증해준다 (타입에러를 상대적으로 빨리 알 수 있다. 안정성)

- 성능: 실행 시점에 어떤 메소드를 호출할지 알아내는 과정이 필요없으므로 메소드 호출이 더 빠름
- 신뢰성: 컴파일러가 정확성을 검증 (런타임 에러 확률이 줄어든다)
- 유지보수성: 코드에서 타입을 알 수 있으므로 처음 보는 코드를 다룰 때도 더 쉽다
- 도구 지원: 안전하게 리팩토링할 수 있고도구는 더 정확한 코드 완성 기능을 제공할 수 있다 (IDE, 자동완성 등)

그렇다면 자바처럼 코드로 모든 필드의 타입을 명시해야할까?

**타입추론**

컴파일러 문맥을 고려해 변수 타입을 결정

    val x = 1  // Int임을 자동으로 추론

동적 타입 지정 언어: 타입과 관계없이 모든 값을 변수에 넣을 수 있고, 이로 인해 컴파일 에러가 나지 않음 → 런타임 오류 발생! (JVM에서의 Groovy, JRuby / 파이썬)

## 특징2. Nullable Type(?)

타입 뒤에 ?을 붙이면 nullable

    val x: Int = null         // 에러!
    val str: String? = null 

## 특징3. Functional Programming 지원

함수형 프로그래밍의 특징

- first-class function: 함수를 일반 값처럼 다룰 수 있다. 함수를 변수에 저장하거나 파라미터로 전달하거나 함수에서 새로운 함수를 만들어 반환할 수 있다. → 람다 함수 지원
    - higher-order function: 함수를 파라미터로 받거나 함수를 리턴하는 함수

    // Collection에 멤버 함수를 추가
    fun <T, R> Collection<T>.fold(
        initial: R, 
        combine: (acc: R, nextElement: T) -> R
    ): R {
        var accumulator: R = initial
    		// 컬렉션을 돌면서 하나씩 combine 함수를 실행
        for (element: T in this) {
            accumulator = combine(accumulator, element)
        }
    		// 누적된 결과값을 리턴
        return accumulator
    }

    val items = listOf(1, 2, 3, 4, 5)
    
    items.fold(0, { 
        acc: Int, i: Int ->     // 파라미터
        val result = acc + i    // 더하기
        println("acc = $acc, i = $i, result = $result")
    	  result                  // return 을 명시하지 않는다
    })

    acc = 0, i = 1, result = 1
    acc = 1, i = 2, result = 3
    acc = 3, i = 3, result = 6
    acc = 6, i = 4, result = 10
    acc = 10, i = 5, result = 15

- immutability(불변성): 일단 만들어지고 나면 내부 상태가 절대로 바뀌지 않는 불변 객체를 사용해 프로그램을 작성한다. → data class 객체, val 변수
    - 직접 변경하면 그 객체, 변수를 참조하는 모든 곳에 알려야 함. 의도하지 않은 객체의 변경이 일어날 수 있음.
    - 복제나 비교 연산을 빠르게 할 수 있다
    - Java의 클래스는 toString(필드 순차 출력), equals(인스턴스간 비교), hashCode(해시 기반으로 키 사용 가능)을 override 해야하는데, 이를 자동으로 만들어 주는 것이 코틀린의 data class 이다.
- side effect 없음 (pure function): 입력이 같으면 출력이 항상 같음. 다른 객체의 상태를 변경하지 않음. 함수 외부와 상호작용하지 않음

코틀린은 함수형프로그래밍을 강제하지 않는다.

코틀린 표준 라이브러리는 객체와 컬렉션을 함수형 스타일로 다룰 수 있는 API를 제공한다.

함수형 프로그래밍의 장점

1. 더욱 간결하고 추상화된 프로그래밍이 가능하다. 코드 중복을 더욱 줄일 수 있다.

    // findPerson을 공통 함수로 묶고, 조건이 다른 함수를 따로 정의 
    fun findAlice() = findPerson{ it.name == "Alice" }
    fun findBob() = findPerson{ it.name == "Bob" }

2. 다중스레드에서도 안전하다. 불변 객체와 순수 함수를 사용하면 같은 데이터를 여러 스레드에서 바꿀 수 없다.

3. 테스트하기 쉽다. 순수 함수는 setup 코드 없이 바로 테스트할 수 있다.

## 코틀린 응용. 서버, 안드로이드 프로그래밍

자바를 사용하는 많은 새로운 기술이나 프레임워크와 상호작용이 가능하다.

- 웹 브라우저에 HTML 페이지를 돌려주는 웹 애플리케이션

    fun renderPersonList(persons: Collection<Person>) {
    		createHTML().table {
    				for(person in persons) {
    						tr {
    								td { +person.name }
    								td { +person.age }
    						}
    				}
    		}
    }

    object CountryTable : IdTable() {
    		val name = varchar("name", 250).uniqueIndex()
    		val iso = varchar("iso", 2).uniqueIndex()
    }
    
    object Country(id: EntityID) : Entitiy(id) {
    		var name: String by CountryTable.name
    		var iso: String by CountryTable.iso
    }
    
    val russia = Country.find {
    		CountryTable.iso.eq("ru")
    }.first()
    
    println(russia.name)

- 모바일 애플리케이션에 HTTP 통신으로 JSON API를 제공하는 Backend 애플리케이션

    NPE에 엄격하다. 컴파일도 되지 않음. → 예기치 않은 종료가 줄어든다.

    코틀린 런타임 시스템이 작기 때문에 apk 크기가 많이 늘어나지 않는다.

    인자로 받은 람다 함수를 inlining한다. (새로운 객체를 만들지 않는다. 따라서 객체 증가로 인해 GC가 늘어나서 프로그램이 자주 멈추는 일이 없다.)

    // Anko lib
    verticalLayout {
    		val name = editText()
    		button("Say Hello") {
    				onClick { toast("Hello, ${name.text}!") }
    		}
    }

- RPC 프로토콜을 통해 서로 통신하는 마이크로서비스

## 코틀린의 철학1. 실용성

- 실전 문제를 해결하기 위해 만들어진 실용적인 언어 (다른 언어가 채택한 검증된 해법 사용)
- 특정 프로그램 스타일이나 패러다임을 강요하지 않음
- 인텔리J와 함께 IDE을 개발하며 컴파일러를 개발했기 때문에 도구 활용성이 높다

## 코틀린의 철학2. 간결성

- 언어로 작성된 코드를 읽을 때 의도를 쉽게 파악할 수 있는 구문 구조
- 방해가 되는 준비 코드를 적게 프로그래밍할 수 있음

## 코틀린의 철학3. 안전성

- 일부 유형의 오류를 프로그램 설계가 원천적으로 방지해줄 수 있는가
- JVM: 메모리 안정성, 버퍼 오버플로 방지, 정적 타입 지정 언어
- NPE: 방지 데이터 타입 (?)
- ClassCastException: 어느정도 자동으로 검사해주고 캐스팅 없이 사용할 수 있음

    if(value instanof Stirng)
    	((String) value).toUpperCase()

    if(value is String)
    	value.toUpperCase()

## 코틀린의 철학4. 상호운용성

- 기존 자바 라이브러리를 그대로 사용할 수 있다 (자바 클래스 상속, 애노테이션 적용 등)
- 한 파일에 섞여 있어도 컴파일이 가능하다

## 코틀린 코드 컴파일

[](https://www.notion.so/a0c90a7239824282bfb69cdb87e38e90#4ae6c1456e6b4e08944dbea1512448ef)

    kotlinc <file or dir> -include-runtime -d <jar name>
    java -jar <jar name>

- .jar는 여러 .class 파일들의 압축 파일이다

**인터프리터 언어와 컴파일 언어**

[](https://www.notion.so/a0c90a7239824282bfb69cdb87e38e90#3b9faef2e1d943f9a4bfd5d35954b322)

- 컴파일 언어
    - 런타임 전에 기계어로 번역하고, 번역된 것으로 실행
    - 실행은 빠르나 프로그램이 크면 컴파일 시간이 오래걸림
    - Java의 경우 컴파일 시에 바이트코드(.class) → 바이트코드를 jvm 위에서 실행
- 인터프리터 언어
    - 소스코드를 바로 한줄 한줄씩 실행
    - 별도의 실행파일이 생기지 않고 바로 실행
    - 무거운 프로그램도 바로 실행 가능하다

## 자바 컴파일 과정

[](https://www.notion.so/a0c90a7239824282bfb69cdb87e38e90#da3d94ae04844e3a8bd72725888f6b98)

1. 자바 컴파일러가 CPU는 해석할 수 없는 반 기계어의 바이트코드를 .class 파일을 만듦
2. 프로그래머가 java 코드 작성
3. .class 파일들을 JRE에 포함된 JVM 내의 인터프리터에서 실행
4. 실행중인 프로세스는 힙, 스택, 등의 메모리를 할당 받음
    - 힙: New 명령어를 통해 생성한 인스턴스, 배열 등의 참조형 변수정보 저장. Method Area에 올라온 클래스들만 생성 가능. GC의 대상
    - 스택: 클래스 내의 메소드들에 사용되는 정보 저장. 매개변수, 지역변수, 리턴값. 실행 완료시 제거. 인스턴스, 배열 등의 new 객체는 실제로 힙에 있고, 스택은 힙의 메모리 주소를 갖고 있음
    - 메소드: 클래스와 메소드, 멤버(클래스, 인스턴스)변수와 상수(final) 정보 등이 저장되는 공간
    - PC register: 스레드 마다 생성. 다음 JVM 명령어의 주소값 저장
    - Native Method Stack: 자바 외 다른 언어일 경우 사용하는 스택 (C, C++)

## GC

1. Minor GC
    - New Generation과 Tenured Generation 에서 일어나는 GC
    - Eden: 최초로 new 한 객체가 할당받는 곳
    - Eden: 어느 정도 데이터가 쌓이면 참조 정도에 따라 Servivor1, 2 또는 회수됨
2. Major GC
    - Old 메모리가 허용치를 넘게 되면 참조되지 않은 객체를 한꺼번에 회수하는 GC 발생
    - stop-the-world: 이 때 GC가 끝날 때까지 이외의 모든 쓰레드는 작업을 멈춘다