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