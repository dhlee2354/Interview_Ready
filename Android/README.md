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


---


### Content Provider
- 정의
  + Content Provider는 애플리케이션이 자신의 데이터를 다른 애플리케이션과 안전하게 공유할 수 있도록 관리하는 컴포넌트 입니다.
  + 데이터는 파일 시스템, SQLite 데이터베이스, 웹 또는 애플리케이션이 접근할 수 있는 다른 영구 저장 위치에 저장될 수 있습니다.

- 데이터 공유
  + 애플리케이션 간에 구조화된 데이터를 공유하기 위한 표준 인터페이스를 제공합니다.
  + 안드로이드 시스템의 주소록, 미디어 스토어 등은 Content Provider를 통해 다른 앱에 데이터를 제공합니다.

- 데이터 캡슐화
  + 데이터의 실제 저장 방식을 숨기고, 추상화된 인터페이스를 통해 데이터에 접근 하도록 합니다.
  + 데이터 저장 방식이 변경되더라도 다른 앱의 코드에 영향을 주지 않고 데이터를 제공할 수 있습니다.

- 데이터 접근제어
  + 읽기/쓰기 권한을 세밀하게 제어하여 다른 애플리케이션이 데이터에 접근하는 것을 허용하거나 제한할 수 있습니다.

- 표준 CRUD 인터페이스
  + 데이터를 조작하기 위한 표준 메서드 집합을 제공합니다.
    * query() : 데이터 검색
    * insert() : 새 데이터 삽입
    * update() : 기존 데이터 수정
    * delete() : 데이터 삭제
    * getType() : 특정 URI에 해당하는 데이터의 MIME 타입을 반환

- 콘텐트 URI
  + Content Provider 내의 특정 데이터를 식별하기 위한 고유한 URI 체계를 사용합니다. 
  + content:// : 스킴, 콘텐츠 프로바이더임을 나타냅니다.
  + <authority>: 콘텐츠 프로바이더를 식별하는 고유한 이름
  + <path>: 프로아비더가 제공하는 데이터의 종류를 나타내는 경로
  + <optional_id>: 특정 레코드를 식별하기 위한 ID

- ContentResolver
  + 다른 애플리케이션은 ContentsResolver 객체를 사용하여 특정 콘텐츠 프로아비더와 상호작용합니다.
  + 클라이언트는 ContentResolver의 메서드를 호출하고 ContentResolver는 해당 URI를 기반으로 적절한 콘텐츠 프로바이더를 찾아 요청을 전달 합니다.

- 주의할 점
  + 콘텐츠 프로바이더의 메서드(특히 query())는 시간이 오래 걸릴 수 있으므로, UI 스레드에서 직접 호출하면 ANR이 발생할 수 있습니다. 비동기적으로 처리해야 합니다.
  + 데이터 공유 시 보안에 유의해야 합니다. 필요한 최소한의 권한만 부여하고, 민감한 정보는 신중하게 다루어야 합니다.
  + 자신의 앱 내에서만 데이터를 사용하는 경우에는 굳이 콘텐츠 프로바이더를 만들 필요가 없을 수도 있습니다.


---


### Service
- 정의
  + 서비스는 사용자와 직접적인 UI 상호작용 없이 백그라운드에서 오래 실행되는 작업을 수행하거나, 
    다른 애플리케이션에 기능을 제공하기 위한 애플리케이션에 기능을 제공하기 위한 애플리케이션 컴포넌트 입니다.

- 주요 특징 및 용도
  + 백그라운드 작업
    * 네트워크 트랜잭션 처리
    * 음악 재생
    * 백그라운드 데이터 동기화
    * 위치 정보 업데이트
  + UI 없음
    * 서비스는 자체적인 사용자 인터페이스를 가지지 않습니다.
  + 생명주기
    * 액티비티와는 다른 독립적인 생명주기를 가집니다.
  + 포그라운드 서비스
    * 사용자에게 현재 실행 중임을 인지시켜야 하는 중요한 작업을 수행하는 서비스입니다.
    * 반드시 상태 표시줄에 알림을 표시해야 합니다.
    * 시스템이 메모리 부족 상황에서도 포그라운드 서비스를 쉽게 종료시키지 않습니다.
    * 안드로이드 9 버전 이상에서는 포그라운드 서비스 실행시 FOREGROUND_SERVICE 권한이 필요합니다.
  + 서비스는 기본적으로 애플리케이션의 메인 스레드에서 실행됩니다. 따라서 서비스 내에서 CPU를 많이 사용하거나 시간이 오래 걸리는 작업을 직접 수행하면 ANR 오류가 발생할 수 있습니다.
  + 백그라운드 실행 제한 : 최신안드로이드 버전에서는 배터리 수명과 시스템 성능을 위해 백그라운드 서비스 실행에 제한이 강화되었습니다. 따라서 장기 실행 작업에서는 WorkManager와 같은 다른 솔루션을 고려하는 것이 좋습니다.


---


## 4대 컴포넌트 Activity
- 정의
  + 안드로이드에서 하나의 사용자 인터페이스 화면
  + 사용자와 직접 상호작용하는 컴포넌트이며 UI를 구성하고 앱 흐름을 제어하는 기본 단위
- 특징
  + | 항목                     | 내용                                       |
    | ---------------------- | ---------------------------------------- |
    | **UI 구성 단위**           | 앱의 화면 하나가 하나의 Activity                   |
    | **생명주기 존재**            | `onCreate()`부터 `onDestroy()`까지 상태 전환     |
    | **명시적/암시적 인텐트로 호출 가능** | 다른 Activity나 앱 호출 가능                     |
    | **Manifest 등록 필수**     | `AndroidManifest.xml`에 선언돼야 실행 가능        |
    | **백스택 관리**             | `startActivity()` → 이전 Activity는 백스택에 저장 |
    | **Context 역할 수행**      | 리소스 접근, 시스템 서비스 호출 가능                    |
- Intent
  + 명시적(Intent Explicit) 특정 컴포넌트 지정
    * ```kotlin
      val intent = Intent(this, DetailActivity::class.java)
      startActivity(intent)
      ```
  + 암시적(Intent Implicit) 작업(Action)에 맞는 컴포넌트 자동 매칭
    * ```kotlin
      val intent = Intent(Intent.ACTION_VIEW, Uri.parse("http://google.com"))
      startActivity(intent)
      ```
    * 링크를 띄울 수 있는 액션을 OS에서 받아 하단 Chooser 브라우저 목록 표시
- 면접 관련 질문
  + Activity 간 데이터 전달 방법?
    * Activity 간 데이터는 주로 Intent의 Extra를 통해 전달
    * 복잡한 데이터는 Serializable 또는 Parcelable 객체를 사용하며, 경우에 따라 Bundle을 통해 전달
  + Activity & Fragment 차이
    * | 항목      | Activity | Fragment              |
      | ------- | -------- | --------------------- |
      | 실행 단위   | 독립 실행 가능 | Activity 내부에서만 실행     |
      | 생명주기    | 독립적      | Activity 생명주기에 종속     |
      | 사용 목적   | 전체 화면 구성 | 부분 화면, 동적 UI 구성       |
      | 재사용성    | 낮음       | 높음                    |
      | Manager | 시스템이 관리  | `FragmentManager`가 관리 |
    * Fragment 사용 시 장점
      1. 메모리 효율성 높음 (View 만 교체 가능 → 가벼움)
      2. 화면 재사용 용이 (여러 Activity 에서 호출 가능)
    * Activity는 UI뿐 아니라 자체 Window와 Context를 포함하기에 시스템 차원에서 더 많은 초기화 비용이 발생.
    * 반면 Fragment는 Activity 위에서 동작하며, View와 Lifecycle만 별도로 관리하므로 생성 비용이 훨씬 가벼움
  + Activity 에서 메모리 누수 발생 상황?
    * "Activity는 View, Context 등을 포함하고 있어서, 생명주기와 무관한 참조가 남아 있으면 GC되지 않아 메모리 누수 발생할 수 있음
      1. 익명 클래스 Handler
      2. 전역 static 변수에 Context 저장
      3. 비동기 작업 (예: Retrofit, RxJava) 완료 전에 Activity가 종료됨
      4. ViewBinding을 onDestroy에서 해제하지 않음 (Fragment도 포함)
    * 해결 방법
      1. WeakReference 사용
      2. lifecycleScope, viewLifecycleOwner.lifecycleScope 활용
      3. onDestroy()에서 리소스 해제
  + Activity 화면 회전 시 상태 보존 방법?
    * 화면 회전 시 Activity는 재생성되므로, 
      1. **onSaveInstanceState()**를 사용해 데이터를 임시 저장, 
      2. **onCreate() 또는 onRestoreInstanceState()**에서 복원
    * ViewModel 사용하여, 구성 변경(Configuration Change) 시에도 데이터를 유지


---


## 4대 컴포넌트 BroadcastReceiver
- 정의
  + 시스템 또는 앱에서 발생하는 브로드캐스트 이벤트를 수신하고, 이에 대응하는 코드를 실행하는 컴포넌트
    * 안드로이드 시스템은 다양한 **이벤트(알람, 네트워크 변화, 부팅 완료 등)**를 브로드캐스트로 보냄
    * 앱도 커스텀 브로드캐스트를 만들어 보낼 수 있음
    
- 특징
  + | 항목    | 설명                                |
    | ----- | --------------------------------- |
    | 실행 방식 | 콜백 함수인 `onReceive()`가 호출되어 처리     |
    | 생명주기  | 화면 UI 없음. `onReceive()`만 실행되고 종료됨 |
    | 등록 방식 | 정적 등록 (Manifest), 동적 등록 (코드에서)    |
    | 목적    | 이벤트 감지 및 간단한 처리를 빠르게 수행           |

- 사용 방법
  + 정적 등록(Manifest)
    * ```xml
      <receiver android:name=".BootReceiver">
        <intent-filter>
           <action android:name="android.intent.action.BOOT_COMPLETED"/>
        </intent-filter>
      </receiver>
      ```
  + 동적 등록(코드 방식)
    * ```kotlin
      val receiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
          Log.d("Receiver", "Battery changed!")
        }
      }

      val filter = IntentFilter(Intent.ACTION_BATTERY_CHANGED)
      registerReceiver(receiver, filter)
      ```
      1. 앱이 실행 중일 때만 수신 가능
      2. 주로 Activity, Service 내에서 사용

- 주의사항
  + onReceive()는 UI 작업 금지 (짧고 빠른 처리만 수행해야 함)
  + 오래 걸리는 작업은 JobIntentService, WorkManager, ForegroundService 연계 필요
  + Android 8.0(Oreo) 이상에서는 Manifest 등록 제한이 많아 동적 등록 또는 백그라운드 제한 우회 필요

- 면접 관련 질문
  + Android 8.0(Oreo) 이후 백그라운드 앱의 정적 브로드캐스트 수신이 제한
    * 보안과 배터리 최적화
      1. Manifest 등록한 브로드캐스트 리시버는 일부 시스템 이벤트만 수신할 수 있게 되었고,
      2. 대부분 앱이 실행 중일 때 동적으로 등록해야 수신 가능
  + onReceive() 에서 긴 시간의 작업 실행 시?
    * 내부적으로는 앱 프로세스가 일시적으로 깨워진 상태에서 동작함
    * 긴 작업을 실행 시 **ANR(Application Not Responding)**이 발생하거나 시스템이 강제로 종료
  + BroadcastReceiver 대신 사용할 수 있는 대
    * Android 8.0 이상 JobScheduler, JobIntentService, WorkManager 같은 백그라운드 작업 매니저
    * LiveData, StateFlow, EventBus 방법 등으로 대체 가능
  + BroadcastReceiver vs Service
    * | 항목           | **BroadcastReceiver**                          | **Service**                                             |
      | ------------ | ---------------------------------------------- | ------------------------------------------------------- |
      | **목적**       | 이벤트 수신 후 짧은 작업 처리                              | 백그라운드에서 장시간 작업 수행                                       |
      | **트리거 방식**   | 외부 or 내부에서 **브로드캐스트**로 호출됨 (`sendBroadcast()`) | 명시적으로 **시작(`startService`)** 또는 바인딩(`bindService`)      |
      | **생명주기**     | `onReceive()` 단 하나, 실행 후 바로 종료                 | `onCreate()` → `onStartCommand()` → `onDestroy()` 등 존재  |
      | **UI와의 관계**  | UI 없음 / 사용자와 상호작용 X                            | UI 없음 / Activity와는 분리되어 있음                              |
      | **작업 시간 제약** | **매우 짧아야 함** (몇 초 이내) → 길면 ANR 발생              | 상대적으로 **장시간 작업 가능**                                     |
      | **실행 상태 유지** | 호출 후 자동 종료됨                                    | 명시적으로 종료되기 전까지 **계속 실행** 가능                             |
      | **멀티스레드 처리** | 자체 스레드 아님 (메인 스레드에서 동작)                        | 직접 스레드 생성 가능 (`HandlerThread`, `Coroutine`, `Thread` 등) |
      | **사용 사례**    | 네트워크 상태 변경 감지, 부팅 완료 감지, 앱 내 메시지 처리            | 파일 다운로드, 음악 재생, 알람 대기 등                                 |


---


## PendingIntent
- 정의
  + 다른 앱 또는 시스템이 지정된 작업(Intent) 대신 실행할 수 있도록 허가하는 객체
  + 알림(Notification), 알람(AlarmManager), 위젯(AppWidget), 브로드캐스트 예약 등에 쓰임
  > Intent 캡슐화하여 나중에 다른 컴포넌트(시스템 또는 외부 앱)가 실행할 수 있도록 위임하는 객체

- 필요한 이유?
  + 시스템이나 다른 앱이 내 앱의 Context 없이 특정 작업을 수행해야 할 때 사용
  + AlarmManager, NotificationManager 같은 시스템 서비스는 앱의 Intent를 직접 실행할 수 없음
  + 즉, 앱이 대신 실행해달라고 요청하는 형태로 감싸서 전달해야 함

- 주요 사용 케이스
  + | 상황                     | 사용 예                                |
    | ---------------------- | ----------------------------------- |
    | 알림 클릭 시 특정 Activity 열기 | `Notification`과 함께 사용               |
    | 특정 시간에 알림 보내기          | `AlarmManager`에서 예약 시 사용            |
    | 브로드캐스트 예약              | `PendingIntent.getBroadcast()` 사용   |
    | 서비스 시작                 | `PendingIntent.getService()` 사용     |
    | 앱 위젯에서 버튼 클릭 처리        | 위젯의 `RemoteViews`에 PendingIntent 연결 |
  + PendingIntent 메소드에는 getActivity(), getService() 도 있음

- 생성 방법
  + ```kotlin
    // Activity 실행
    val intent = Intent(context, MyActivity::class.java)
    val pendingIntent = PendingIntent.getActivity(context, 0, intent, PendingIntent.FLAG_IMMUTABLE)

    // Broadcast 실행
    val intent = Intent(context, MyReceiver::class.java)
    val pendingIntent = PendingIntent.getBroadcast(context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT)

    // Service 실행
    val intent = Intent(context, MyService::class.java)
    val pendingIntent = PendingIntent.getService(context, 0, intent, PendingIntent.FLAG_IMMUTABLE)
    ```

- 주요 Flag
  + | 플래그                   | 설명                                            |
    | --------------------- | --------------------------------------------- |
    | `FLAG_IMMUTABLE`      | **Intent 내용 변경 불가** (보안 강화, Android 12+에서 필수) |
    | `FLAG_MUTABLE`        | 시스템이 Intent 내부를 수정할 수 있도록 허용                  |
    | `FLAG_UPDATE_CURRENT` | 같은 PendingIntent가 있으면 업데이트                    |
    | `FLAG_CANCEL_CURRENT` | 기존 PendingIntent 취소 후 새로 생성                   |
    | `FLAG_NO_CREATE`      | 이미 존재하는 경우에만 반환, 없으면 null                     |
    | `FLAG_ONE_SHOT`      |  한 번만 사용되고 나면 자동으로 소멸되도록 설정                  |
  
  + (| 또는 or) 연산으로 조합 가능
    * PendingIntent.FLAG_IMMUTABLE or PendingIntent.FLAG_ONE_SHOT

- 실무 팁
  + Android 12(API 31) 이상에서는 FLAG_IMMUTABLE 또는 FLAG_MUTABLE 중 하나를 반드시 명시해야 함
  + PendingIntent는 시스템에 캐싱되므로 중복 생성 주의
  + Request Code가 같고, Intent 내용이 같으면 같은 PendingIntent로 간주됨
  + 보안을 위해 불필요하게 mutable한 Intent는 지양

- 면접 관련 질문
  + PendingIntent란 무엇이며, 왜 필요한가요?
  + PendingIntent.FLAG_IMMUTABLE은 언제 사용하나요?
  + PendingIntent.getActivity()와 getBroadcast()의 차이는?


---


### Bundle
- 정의
  + key-value 쌍으로 데이터를 저장하고 전달하는 데 사용되는 핵심 클래스이며, 다양한 타입의 데이터를 담을 수 있습니다.

- Activity, Fragment 간 데이터 전달
  + Intent 객체에 Bundle을 담아 다른 Activity로 데이터를 전달할 수 있습니다.
  + Fragment를 생성할 때 arguments로 Bundle을 전달하여 초기 데이터를 넘겨줄 수 있습니다.

- Activity, Fragment 상태 저장 및 복원
  + 화면 회전과 같이 Activity나 Fragment가 시스템에 의해 재생성될때, onSaveInstanceState 콜백 메서드를 통해 현재 상태를 Bundle에 저장할 수 있습니다.
  + 이후 onCreate나 onRestoreInstanceState 또는 onCreateView(), onViewCreated()에서 저장된 Bundle을 받아 상태를 복원할 수 있습니다.

- Service와 데이터 교환
  + Service에 데이터를 전달하거나 Service로부터 결과를 받을 때 Bundle을 사용할 수 있습니다.

- 다양한 Android 컴포넌트 간 데이터 전달
  + BroadcastReceiver가 브로드캐스트를 수신할 때 Intent에 포함된 Bundle을 통해 데이터를 받을 수 있습니다.
  + ContentProvider를 통해 데이터를 전달할 때도 Bundle이 사용될 수 있습니다.

- Bundle의 주요 특징
  + key-value 저장소 : 문자열 키를 사용하여 다양한 타입의 값을 저장합니다.
  + 다양한 데이터 타입 지원 : String, int, long,boolean, float, double, char, byte, short와 같은 기본 타입뿐만 아니라,
                         CharSequence, Parcelable, Serializable, 배열, ArrayList 등 다양한 객체 타입을 저장할 수 있습니다.
  + Parcelable 인터페이스 : Bundle 자체는 Parcelable 인터페이스를 구현합니다. 이는 Bundle 객체가 프로세스 간 통신을 위해 효율적으로 직렬화될 수 있음을 의미합니다.
                         안드로이드 시스템은 Parcelable을 사용하여 객체를 Intent에 담아 다른 컴포넌트로 전달하거나, 프로세스 경계를 넘어 데이터를 전송합니다.
  + 데이터 크기 제한 : Bundle을 통해 전달되는 데이터의 크기에는 제한이 있습니다. 너무 큰 데이터를 Bundle에 담아 전달하려고 하면 TranscationTooLargeException이 발생할 수 있습니다.
                   일반적으로 10KB에서 1MB 미만으로 유지하는 것이 좋습니다. 

- Bundle 사용 시 주의 사항
  + null 체크 : Bundle에서 값을 꺼낼 때는 해당 키가 존재하지 않거나 값이 null일 수 있으므로, 항상 null체크를 하거나 기본값을 제공하는 함수(예 : getXX(key, defaultValue))를 사용 하는 것이 안전 합니다.
  + 키 관리 : Bundle에 데이터를 넣고 뺄 때 사용하는 키 문자열은 정확히 일치해야 합니다. 오타를 방지하기 위해 상수로 정의하여 사용하는 것이 좋습니다.
  + 데이터 타입 일치 : 데이터를 넣을 때 사용한 putType() 메서드와 꺼낼 때 사용한 getType()메서드의 데이터 타입이 일치 해야 합니다.
  + Serializable vs Parcelable : 복잡한 객체를 Bundle에 담을 때 Serializable이나 Parcelable을 사용할 수 있습니다.
    * Parcelable : 안드로이드에서 성능이 더 좋도록 특별히 설계되었으므로, 안드로이드 컴포넌트 간 데이터 전달에는 Parcelable 사용이 권장됩니다.
    * Serializable : 구현이 더 간단하지만 리플렉션을 사용하기 때문에 성능이 상대적으로 느립니다.
