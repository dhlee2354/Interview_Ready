# 🤖 Kotlin 가이드 : 개념 및 개발

Kotlin 언어의 문법, 함수형 프로그래밍, 코루틴 등 안드로이드 개발자에게 필요한 핵심 개념 등 모든 것을 다룬 문서입니다.

---

## 📘 주요 개념

### object 키워드
- 정의
    + 클래스를 정의함과 동시에 인스턴스를 생성하는 키워드
    + 코드 간결성 확보 및 Thread Safe
    + 최초 참조 시점에 단 한번만 인스턴스가 생성됨 (lazy + synchornized) & JVM class loader가 클래스 로딩을 Thread-safe하게 보장함
- 주요 사용 케이스
    + 싱글톤 (Singleton)
        * 하나만 존재하는 객체 정의
        ```kotlin
        object Logger {
            fun log(msg: String) {
                println("Log: $msg")
            }
        }
      
        Logger.log("message") // 호출
        ```
        * 생성자 호출 필요없고 Thread-safe 함
    + 동반객체(Companion Object)
        * 클래스 내부에서 정적 멤버처럼 사용
        ```kotlin
        class User(val name: String) {
            companion object {
                fun create(name: String): User = User(name)
                var country = "Korea" 
                @JvmStatic val BASE_ADDRESS = "서울특별시" // java static 처럼 
            }
        }
      
        User.create("Alice") // 클래스 이름으로 호출 가능
        User.coutnry // Seoul
        ```
        * @JvmField, @JvmStatic 붙이면 java 에서도 static 처럼 쓸 수 있음
    + 익명객체
        * 즉석에서 정의해 사용하는 임시 객체
        ```kotlin
        val buttonClickListener = object: View.OnClickListener {    
            override fun onClick(v: View?) {
                println("Clicked!")
            }
        }
        ```
        * 인터페이스나 추상 클래스를 즉석에서 구현
        * Android 리스너 구현에 자주 사용
    + Object Declaration
        * 상속 구조나 전략 객체로 사용
        * 동일한 인터페이스를 구현한 여러 객체 중 하나를 선택해 실행 전략을 바꾸는 패턴
        ```kotlin
        interface PaymentStrategy {
            fun pay(amount: Int)
        }

        object KakaoPay : PaymentStrategy {
            override fun pay(amount: Int) {
                println("카카오페이로 $amount 원 결제")
            }
        }

        object CreditCard : PaymentStrategy {
            override fun pay(amount: Int) {
                println("신용카드로 $amount 원 결제")
            }
        }
      
        class PaymentProcessor(var strategy: PaymentStrategy) {
            fun processPayment(amount: Int) {
                strategy.pay(amount)
            }
        }

        fun main() {
            val processor = PaymentProcessor(KakaoPay)
            processor.processPayment(10_000)  // 카카오페이로 결제

            processor.strategy = CreditCard
            processor.processPayment(20_000)  // 신용카드로 결제
        }
        ```
        * PaymentProcessor는 PaymentStrategy라는 상위 타입만 알고 있음
        * 실제 객체는 KakaoPay, CreditCard 등 다양하게 대체 가능 → 런타임 다형성
        * 조건에 따라 전략을 바꾸거나, 클라이언트 코드가 객체의 구체 타입을 몰라도 사용 가능


---


### lateinit & lazy
- 정의
  + 코틀린에서 non-null 타입 프로퍼티를 나중에 초기화할 수 있도록 지원하는 두 가지 메커니즘 입니다.

- lateinit
  + 선언 :
    ```kotlin
    lateinit var myProperty : String
    ```
  + 목적 : non-null 타입 프로퍼티를 선언 시점에 바로 초기화 할 수 없을 때 사용합니다.
  + 제약사항 :
    * var 프로퍼티에만 사용할 수 있습니다.
    * Nullable 타입에는 사용할 수 없습니다.
    * 기본 자료형을 사용하거나, 박싱된 타입을 사용해야 합니다.
    * 커스텀 getter/setter를 가질 수 없습니다.
  + 주의점 : lateinit 프로퍼티는 사용하기 전에 반드시 초기화되어야 합니다. 

- lazy
  + 선언 :
    ```kotlin
    val myLazyProperty : String by lazy { ... }
    ```
  + 목적 : 프로퍼티의 초기화 비용이 크거나, 실제로 사용될 때까지 초기화를 늦추고 싶을 때 사용합니다.
          lazy는 해당 프로퍼티에 처음 접근하는 순간 단 한번만 초기화 블록을 실행하고 그 결과를 저장합니다. 이후 저장된 값을 반환합니다.
  + 제약사항 : 
    * val 프로퍼티에만 사용할 수 있습니다. 
  + 특징 : 
    * 실제 사용 시점까지 초기화를 미룹니다.
    * 초기화 블록은 최초 접근 시에만 실행됩니다.
    * 기본적으로 LazyThreadSafetyMode.SYNCHRONIZED로 동작하여 여러 스레드에서 동시에 접근해도 안전하게 한 번만 초기화됩니다.

- 사용시점
  + lateinit 사용
    * 프로퍼티가 non-null 타입이어야 하고, 생성자에서 초기화할 수 없으며, 나중에 반드시 초기화될 것이라고 확실할 때
    * 초기화 시점을 개발자가 직접 제어해야 할 때

  + lazy 사용
    * 프로퍼티가 non-null 타입이어야 하고, 최초 접근 시점에 초기화되어야 하며, 그 값이 변경되지 않는 경우
    * 초기화 비용이 커서 실제 사용될 때까지 초기화를 늦추고 싶을 때
    * 싱글톤 패턴과 같이 특정 객체가 최초 사용 시점에 한번 만 생성되도록 하고 싶을 때


---

### Safe Call / Elvis 연산자
- 코틀린에서 Safe Call 연산자(?.)와 Elvis 연산자(?:)는 null-safety를 보장하기 위해서 자주 사용되는 두 가지 핵심 연산자 입니다.

- Safe Call 연산자 (?.)
  + 역할 : 객체가 null일 경우 메서드 호출이나 프로퍼티 접근을 건너뛰고 null을 반환합니다.
  + NullPointerException을 방지할 수 있습니다.

- 문법
  ```kotlin
  val result = someObject?.someMethod()
  ```

- 예제
  ```kotlin
    data class User(val name: String?)

    val user: User? = null
    val nameLength = user?.name?.length
    println(nameLength) // 출력: null
  ```
  
- Elvis 연산자 (?:)
  + 역할 : 좌측 값이 null일 경우 우측의 기본값을 사용합니다.
  
- 문법
  ```kotlin
  val value = nullableValue ?: defaultValue
  ```

- 예제
  ```kotlin
    val user: User? = null
    val nameLength = user?.name?.length ?: 0
    println(nameLength) // 출력: 0
  ```
  
- Safe Call 연산자 + Elvis 연산자 함께 쓰는 예
  ```kotlin
    val user: User? = null
    val nameLength = user?.name?.length ?: 0
    println(nameLength) // 출력: 0
  
    // Android 에서 자주 쓰는 패턴
    // null이면 return 처리
    val token = prefs?.getString("token", null) ?: return
  ```


---


## var vs val
- 개념
  + var 변경 가능한 변수 (mutable)
  + val 변경 불가능한 변수 (immutable)
- 공통점
  + | 항목                     | 내용                                    |
    | ---------------------- | ------------------------------------- |
    | **둘 다 변수 선언에 사용**      | Kotlin에서 변수를 선언할 때 `var` 또는 `val`을 사용 |
    | **타입 추론 가능**           | `var name = "John"`처럼 타입 명시 없이 선언 가능  |
    | **null 가능성, 범위 규칙 동일** | 둘 다 `?`로 nullable 여부 설정 가능, scope는 동일 |
- 차이점
  + | 항목             | `var`                              | `val`                                                              |
    | -------------- | ---------------------------------- | ------------------------------------------------------------------ |
    | **가변성**        | 값을 변경할 수 있음                        | 최초 할당 이후 값 변경 불가                                                   |
    | **재할당**        | 가능                                 | 불가능                                                                |
    | **사용 예시**      | 루프 인덱스, 상태 변화 변수                   | 상수, 변경되지 않는 참조값                                                    |
    | **컴파일러 처리**    | 매번 수정 가능성 고려                       | 불변성 보장 → 최적화 유리                                                    |
    | **컬렉션의 내용 변경** | `val`로 선언해도 **내부 요소는 변경 가능** (주의!) | `val list = mutableListOf(1, 2)`는 리스트 요소 변경 가능, 단 `list` 자체 재할당 불가 |
- 예시
  + ```kotlin
    // var: 값 변경 가능
    var name = "Alice"
    name = "Bob" // ✅ 가능

    // val: 값 변경 불가능
    val age = 30
    // age = 31 // ❌ 컴파일 오류

    // val로 선언된 불편 컬렉션의 내부는 변경 가능
    val list = listOf(1,2,3)
    // list 에는 객체를 변경할 수 있는 메소드가 제공되지 않음
    list = listOf(4,5,6) // ❌ 재할당 불가능
    
    // val로 선언된 가변 컬렉션의 내부는 변경 가능
    val numbers = mutableListOf(1, 2, 3)
    numbers.add(4) // ✅ 가능
    // numbers = mutableListOf(5, 6) // ❌ 재할당 불가능
    ```
  + val 의 의미는 재할당이 불가능하고 가변 컬렉션의 내부 값의 변경은 가능함
- 결론
  + 기본적으로 val 을 사용하고 변수를 만들고 필요한 경우에만 var 사용하는 것이 코틀린 가이드
  + val이 많을수록 코드 안정성, 스레드 안전성 증가
  + val 사용 시 **불변객체 지향(immutability)**로 인한 버그 예방 가능
  + 안드로이드 예
    ```kotlin
    class MyViewModel : ViewModel() {
       private val _uiState = MutableLiveData<UiState>()
       val uiState: LiveData<UiState> = _uiState  // 외부에는 읽기 전용으로 노출
    }
    ```
    * val을 쓰면 ViewModel의 구조적 명확성(읽기 전용 참조)이 보장됨
    * 실제 상태 변경은 내부 로직에서만 가능하게 설계 → 클린 아키텍처, MVVM 패턴에서 권장되는 방식
    * 의도하지 않은 재할당 방지
    * 상태 추적이 쉬움 → 참조가 고정돼 디버깅, 로깅, 추적이 수월


---


### 스코프 함수(let, run with, apply, also)
- 객체의 컨텍스트 내에서 코드 블록을 실행할 수 있게 해주는 특별한 함수들 입니다.
- 스코프 함수들은 코드를 더 간결하고 읽기 쉽게 만들어주며, 주로 객체에 대한 연속적인 연산을 수행하거나, 객체의 프로퍼티에 접근하여 초기화 또는 설정 작업을 할 때 유용하게 사용 됩니다.
- 코틀린 표준 라이브러리에는 let, run, with, apply, also 다섯가지의 스코프 함수가 있습니다.

- 컨텍스트 객체 참조 방식
  + this(수신 객체) : 람다 내부에서 컨텍스트 객체를 this로 참조합니다. 객체의 멤버에 직접 접근할 수 있습니다.(run, with, apply)
  + it(람다 인자) : 람다 내부에서 컨텍스트 객체를 단일 인자 it으로 참조합니다.(let, also)

- 반환 값
  + 컨텍스트 객체 자신(apply, also) : 람다 블록을 실행한 후 컨텍스트 객체를 반환합니다. 객체에 대한 추가적인 연쇄 호출이 필요할 때 유용합니다.
  + 람다 결과(let, run, with) : 람다 블록의 마지막 표현식 결과를 반환합니다. 특정 연산의 결과를 변수에 할당하거나, 다른 연산으로 전달해야 할 때 유용합니다.

- let
  + 컨텍스트 객체 : it
  + 반환 값 : 람다 결과
  + 주요 사용 경우
    * Null이 아닌 객체에 대해서만 코드를 실행하는 경우
    * 특정 객체를 사용하여 연산한 결과를 다른 변수에 할당할 때

- run
  + 컨텍스트 객체 : this
  + 반환 값 : 람다 결과
  + 주요 사용 경우
    * 객체의 프로퍼티를 설정하고 동시에 어떤 연산의 결과를 반환해야 할 때
    * 특정 객체의 컨텍스트 내에서 여러 연산을 수행하고 그 결과를 얻고 싶을 때

- with
  + 컨텍스트 객체 : this
  + 반환 값 : 람다 결과
  + 주요 사용 경우
    * 특정 객체의 여러 함수나 프로퍼티에 접근해야 할 때
    * run과 유사하지만, with는 확장 함수가 아니므로 객체를 첫 번째 인자로 전달해야 합니다.

- apply
  + 컨택스트 객체 : this
  + 반환 값 : 컨텍스트 객체 자신
  + 주요 사용 경우
    * 객체의 프로퍼티를 설정하고 그 객체를 반환받고 싶을 때
    * 빌더 패턴과 유사하게 객체를 설정할 때 유용합니다.

- also
  + 컨텍스트 객체 : it
  + 반환 값 : 컨텍스트 객체 자신
  + 주요 사용 경우
    * 객체에 대한 부가적인 작업을 수행하고 싶을 때
    * 원래 객체를 변경하지 않고 추가적인 동작을 수행할 때
    * apply와 유사하지만, 컨텍스트 객체를 it으로 참조합니다.

- 어떤 스코프 함수를 선택해야 할까요?
  + null 검사와 함께 특정 연산을 수행하고 결과를 반환 : let
  + 객체 설정 후 객체 자체를 반환 : apply
  + 객체 설정 후 다른 결과를 반환 : run
  + 객체에 대한 부가적인 작업 후 객체 자체를 반환 : also
  + 특정 객체의 멤버에 반복적으로 접근하여 연산 후 결과를 반환 : with