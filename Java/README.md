# 🤖 Java 가이드 : 개념 및 개발

Java 언어의 기초 문법부터 객체지향, 멀티스레드, 컬렉션 등 여러 주제를 다룬 문서입니다.

---

## 📘 주요 개념

### String 문자열
- 문자열을 다루는 가장 기본적이고 중요한 참조형 타입 클래스
  ```java
  public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence, Constable, ConstantDesc {
    
    @Stable
    private final byte[] value;
  }
  ```
- 특징
  + Immutable class (final 키워드 사용)
    * 한 번 생성된 문자열 값은 변하지 않는다. 
    * 할당 연산자(=) 또는 플러스 연산자(+) 사용 시 참조 값이 바뀌지 않고 새로운 값을 만들 후 참조하는 값을 이동시킨다
    ```java
    String a = "test";
    a += "test"; // 기존 test 위치에 추가되는 것이 아니라 testtest 를 만들고 참조하는 값의 위치를 바꾼다
    ```
  + Method Area 생성 (String Pool) 
    ```java
    String a = "test";
    String b = "test";
    String c = new String("test");
    String d = new String("test");
    
    System.out.println(a == b); // true
    System.out.println(a == c); // false
    System.out.println(b == c); // false
    System.out.println(c == d); // false
    
    System.out.println(c.intern() == a); // true (intern() → Pool 참조)
    ```
    * 변수 a 에 문자열 할당 시 String Pool에 test 가 만들어고 a 는 해당 값을 참조한다. 동일한 문자열을 b = "test" 를 넣으면 새로운 값이 아닌 기존 값을 찾아서 참조하게 된다.
    * new 사용 시 Heap 생성 된 문자열의 참조 값은 할당 연산자로 생성된 문자열의 참조 값과 같지 않다
    * 동일한 값은 공유됨
    * intern() 키워드 Heap 객체 -> Pool 객체로 전환
- 형변환
  + 데이터 타입 -> String 으로 변환 시
  ```java
  String convertedValue = String.valueOf(Object); 
  ``` 
  + String 에서 다른 데이터 타입으로 변환 시
  ```java
  int index = 1;
  String str = "4885";

  int num = Integer.parseInt(str);
  double doubleNum = Double.parseDouble(str);
  float floatNum = Float.parseFloat(str);
  char c = str.charAt(index);
  ```
- 자주 쓰이는 함수
  + 대소문자 : a.toLowerCase(), a.toUpperCase()
  + 포함여부 : a.contains()
  + 값비교 : contentEquals()
  + 시작/종료 매치 : starsWith(), endsWith()
- 관련된 심화 주제
  + StringBuilder/StringBuffer, JVM 구조