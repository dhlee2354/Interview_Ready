# 🤖 Android 가이드 : 개념 및 개발

Android 개발에 필요한 핵심 개념, 구조, 실무 적용 예시들을 정리한 문서입니다.

---

## 📘 주요 개념

### Activity 생명주기
- 정의
  + Activity 는 사용자와 화면 단위로 상호작용하는 컴포넌트
  + 사용자가 앱 실행/나가기/회전 등 에 따라 Activity는 여러 상태를 거치며 전환
  + 액티비티 생명주기에 대한 이해를 통해 리소스 누수나 사용자 경험 저하 방지 필요
- 상세설명
  + | 메서드             | 호출 시점                          | 주로 하는 일                       |
    | --------------- | ------------------------------ | ----------------------------- |
    | **onCreate()**  | Activity 최초 생성 시               | 뷰 바인딩, 초기 변수 설정, 리소스 할당       |
    | **onStart()**   | Activity가 화면에 보이기 시작할 때        | UI 준비 (아직 포커스는 없음)            |
    | **onResume()**  | Activity가 포커스를 얻고 사용자와 상호작용 가능 | 애니메이션 시작, 센서 리스너 등록           |
    | **onPause()**   | 다른 Activity가 앞에 나올 때 (부분 가림)   | 리소스 해제 (예: 센서, 카메라 등)         |
    | **onStop()**    | 완전히 화면에서 보이지 않을 때              | DB 저장, 리소스 정리, Broadcast 해제 등 |
    | **onRestart()** | 정지되었다가 다시 보여질 때                | 필요한 UI 갱신                     |
    | **onDestroy()** | Activity 종료되거나 시스템에 의해 제거될 때   | 모든 리소스 해제, GC 대상화             |
- 생명주기 상태 예시
  +  앱 시작 시 `onCreate()` → `onStart()` → `onResume()`
  +  다른 앱 열어서 가릴 때 `onPause()` → `onStop()`
  +  다시 돌아왔을 때 `onRestart()` → `onStart()` → `onResume()`
  +  앱 완전 종료 또는 시스템에 의한 제거 `onPause()` → `onStop()` → `onDestroy()`
- 핵심 포인트
  + `onCreate()` - UI 설정
  + `onPause()` - 리소스(센서, GPS, 애니메이션) 해제
  + `onStop()` - 저장이 필요한 데이터 처리
  + 화면 회전 시 
    * `onPause()` → `onStop()` → `onSaveInstanceState` → `onDestroy()`
    * →`onCreate()` → `onStart()` → `onRestoreInstanceState()` → `onResume()`
    * 왜 액티비티는 재생성 할까?
      1. 레이아웃을 다시 계산해야하는데 기존을 파괴하고 새로 만드는 것이 깔끔한 방법
      2. 이전 상태 유지시 레이아웃도 꼬이고 메모리 누수 위험이 커짐
- Cold/Warm/Hot Start 
  + | 구분             | 정의                       | Activity 생명주기 흐름                           | 특징                               |
    | -------------- | ------------------------ | ------------------------------------------ | -------------------------------- |
    | **Cold Start** | 앱이 완전히 종료된 상태에서 처음 실행됨   | `onCreate()` → `onStart()` → `onResume()`  | 리소스 로딩, Application 객체 생성, 가장 느림 |
    | **Warm Start** | 앱이 백그라운드에 있다가 다시 돌아옴     | `onRestart()` → `onStart()` → `onResume()` | Activity는 살아있고, 앱 프로세스도 유지됨      |
    | **Hot Start**  | 화면 꺼짐, 잠깐 홈으로 나갔다가 다시 복귀 | `onResume()` 단독                            | UI도 메모리에 있고, 거의 즉시 복구됨           |
  + Cold Start 최적화
    * Application 에서 무거운 초기화 지양
    * SplashActivity 최소화
    * 비동기 초기화 (WorkManager, Coroutine, Executors)
  + Warm Start 대응 : 재개 시 필요한 데이터만 빠르게 로딩
  + Hot Start 대응 : Resume 시 불필요한 연산 배제


---


### Fragment 생명주기
- 정의
  + Fragment는 자체적인 생명주기를 가지는 UI 구성요소로, Activity 내에서 모듈식 UI를 구축하는 데 사용 됩니다.
  + Fragment의 생명주기를 이해하는 것은 리소스를 올바르게 관리하고 사용자 경험을 향상시키는 데 매우 중요합니다.

- onAttach()
  + Fragment가 Activity에 연결될 때 호출됩니다.
  + 이 시점에서 Fragment는 Activity의 Context를 얻을 수 있습니다.
  + 주로 Activity와 통신을 위한 인터페이스를 초기화하거나, Activity로부터 필요한 데이터를 전달받는 작업을 수행합니다.

- onCreate()
  + Fragment가 생설될 때 호출됩니다.
  + Fragment의 핵심 로직을 초기화하는 작업을 수행합니다.

- onCreateView()
  + Fragment의 UI 레이아웃을 생성하고 반환할 때 호출됩니다.
  + XML 레이아웃 파일을 inflate하여 Fragment의 뷰 계층 구조를 만듭니다.
  + 반환된 뷰가 화면에 표시 됩니다.

- onViewCreated()
  + onCreateView()에서 반환된 뷰가 완전히 생성된 후 호출 됩니다.
  + 이 시점부터 Fragment의 뷰에 안전하게 접근하여 초기화 작업을 수행 할 수 있습니다.

- onStart()
  + Fragment가 사용자에게 보이기 시작할 때 호출됩니다.

- onResume()
  + Fragment가 사용자와 상호작용할 수 있는 상태가 될때 호출됩니다.

- onStop()
  + Fragment가 더 이상 사용자에게 보이지 않게 될 때 호출 됩니다.
  + 화면에 보이지 않는 동안 필요하지 않은 리소스를 해제합니다.

- onSaveInstanceState()
  + Fragment의 상태가 예기치 않게 소멸 될 가능성이 있을 때, 현재 상태를 저장하기 위해 호출 됩니다.
  + Bundle 객체에 저장할 데이터를 넣어두면, 나중에 onCreate(), onCreateView(), onViewCreated()에서 해당 Bundle을 통해 상태를 복원할 수 있습니다.

- onDestroyView()
  + Fragment와 관련된 뷰가 제거될 때 호출됩니다.

- onDestroy()
  + Fragment가 완전히 소멸될 때 호출됩니다.
  + Fragment의 모든 리소스를 정리하고, 백그라운드 스레드를 중지하는 등의 최종 정리 작업을 수행합니다.
  + 이후 해당 Fragment의 인스턴스는 더 이상 사용되지 않습니다.

- onDetach()
  + Fragment가 Activity와의 연결이 끊어질 때 호출됩니다.

- Cold Start
  + 앱이 완전히 종료된 상태에서 처음 실행할 때 입니다.
  + `onAttach()` → `onCreate()`→ `onCreateView()`→ `onViewCreated()`→ `onStart()`→ `onResume()`

- Warm Start
  + 앱의 프로세스는 살아있지만, Activity가 메모리에서 제거되었거나,백그라운에 있다가 다시 시작될 때 입니다.
  + `onStart()`→ `onResume()`

- Hot Start
  + 앱의 프로세스와 Activity가 모두 메모리에 살아있고, 단순히 화면의 전면으로 다시 가져오는 경우 입니다.
  + `onResume()`