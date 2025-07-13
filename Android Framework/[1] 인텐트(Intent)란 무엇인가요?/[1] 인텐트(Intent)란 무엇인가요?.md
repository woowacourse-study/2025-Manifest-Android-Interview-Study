# 인텐트(Intent)란 무엇인가요 ?

`PPT 만들기 전 자료조사 파일 입니다`

### 인텐트란 ?

다른 앱 컴포넌트(Activity, Service, Broadcast receiver, Content Provider)에 요청을 보내는 메시징 객체 

### 사용 예시

- Activity
    
    송신측
    
    ```kotlin
    val intent = Intent(this, ToIntentActivity().javaClass)
                    .putExtra("info", "정보 전송")
    startActivity(intent)
    ```
    
    수신측
    
    ```kotlin
    val info = intent.getStringExtra("info")
    ```
    
    기본(Primitive)형 타입 메서드와 참조형 타입 메서드
    
    ```kotlin
    public int getIntExtra(String name, int defaultValue) {
        return mExtras == null ? defaultValue :
            mExtras.getInt(name, defaultValue);
    }
    
    public @Nullable String getStringExtra(String name) {
        return mExtras == null ? null : mExtras.getString(name);
    }
    ```
    
    기본형(Primitive) 타입은 null 값을 가질 수 없기에 동일한 이름의 번들 객체가 존재하지 않다면 기본값을 설정할 수 있도록 메서드를 지원한다.
    
    참조형 타입의 경우 null 값이 들어올 수도 있기에 기본값을 제공하는 대신 null 값을 반환하도록 설계되어 있다.
    
- Service
    - Service 간단 설명 : 사용자 인터페이스 없이 백그라운드에서 작업을 수행하는 구성요소
    - IntentService
        
        ```kotlin
        IntentService is an extension of the Service component class that handles asynchronous requests (expressed as Intents) on demand. Clients send requests through android.content.Context.startService(Intent) calls; the service is started as needed, handles each Intent in turn using a worker thread, and stops itself when it runs out of work.
        This "work queue processor" pattern is commonly used to offload tasks from an application's main thread. The IntentService class exists to simplify this pattern and take care of the mechanics. To use it, extend IntentService and implement onHandleIntent(Intent). IntentService will receive the Intents, launch a worker thread, and stop the service as appropriate.
        All requests are handled on a single worker thread -- they may take as long as necessary (and will not block the application's main loop), but only one request will be processed at a time.
        Developer Guides
        For a detailed discussion about how to create services, read the Services developer guide.
        Deprecated
        IntentService is subject to all the background execution limits imposed with Android 8.0 (API level 26). Consider using androidx.work.WorkManager instead.
        See Also: androidx.core.app.JobIntentService
        ```
        
        - 요약 : IntentService 사용한 요청은 하나의 워커 스레드에서 처리한다.  앱의 메인 루프를 막지는 않지만 오래 걸리거나 금방 끝나는 모든 요청을 한 번에 하나의 요청만 처리한다. Android 8.0(오레오,API26) 부터 모든 백그라운드 실행 제한의 영향을 받으므로, Jetpack 라이브러리인 WorkManger 사용을 권장한다.
            - 하단에 JobIntentService에 대한 내용도 있다.
                - Job Api는 Service를 대체하기 위해 나왔다.
                - JobScheduler :  API 21 이상에서만 사용 가능, 일부 기기에서 버그 및 제한 존재.
                - → JobDispatcher : 구글 플레이 서비스 의존성, 일관성 부족.
                - → JobIntentService : 즉시 실행이 보장되지 않거나, 제조사 커스텀 OS에서 동작 불안정.
                - → WorkManager
        - WorkManager
            - Android Jetpack 라이브러리
            - 백그라운드 작업 관리 시스템으로, 앱이 종료되거나 기기가 재부팅되어도 반드시 실행되어야 하는 작업을 안전하게 예약하고 실행할 수 있다.
            - 내부적으로 API 레벨에 따라 JobScheduler, AlarmManager, BroadcastReceiver 등을 자동 선택
            - 백그라운드에서 주기적으로 서버와 앱의 데이터를 동기화하거나, 백그라운드에서 즉시 실행할 작업 예약에 사용된다.
            - Doze mode 같은 절전 기능을 지키고 있으므로 신경쓰지 않아도 된다.

### 인텐트 유형

- 명시적 인텐트
    - 컴포넌트 이름을 명시해서 사용하는 방식
    - 자신의 앱 내에서 컴포넌트를 시작하기 위해 주로 사용하는 방식이다.
    - 사용 예시
        - 앱 내에서 새로운 액티비티를 시작
        - 백그라운드에서 파일을 다운로드 하는 서비스를 시작
    - 코드 예시
    
    ```kotlin
    val intent = Intent(this, ToIntentActivity().javaClass)
        .putExtra("info", "정보 전송")
    startActivity(intent)
    ```
    
- 암시적 인텐트
    - 특정 컴포넌트 이름을 지정하지 않고 수행할 동작을 선언하는 방식
    - action, category, data 기반으로 어떤 컴포넌트가 Intent를 처리할 수 있는 지 결정한다.
    - Action
        - ACTION_VIEW(사용자에게 보여줄 때 사용) : 웹페이지, 연락처, 사진, …
        - ACTION_SEND(다른 앱으로 공유할 때 사용) : 이미지, 텍스트 전송, …
        - ACTION_DIAL(다이얼 앱을 열어 전화번호 입력한 상태로 표시)
        - ACTION_CALL(지정된 번호로 바로 전화 걸 때 사용하며 권한이 필요하다)
        - ACTION_EDIT(수정할 때 사용) : 연락처 수정, …
        - ACTION_SENDTO(특정 수신자에게 데이터 전송할 때 사용) : 이메일, SMS, …
    - 사용 예시
        - 사용자가 지도에 위치를 보여주기를 원한다면, 다른 앱에 지정된 위치를 지도에 표시하도록 요청
        - 브라우저에서 웹 페이지를 열기
    - 코드 예시
    
    ```kotlin
    // 전화 다이얼러 열고 번호 입력
    val dialIntent = Intent(Intent.ACTION_DIAL, "tel:010-1234-5678".toUri())
    startActivity(dialIntent)
    
    // 지도 앱에서 위치 보기
    val mapIntent = Intent(Intent.ACTION_VIEW, "geo:37.565350,127.01445".toUri())
    startActivity(mapIntent)
    
    // 이메일 보내기
    val emailIntent = Intent().apply {
    		action = Intent.ACTION_SENDTO
        "mailto:someone@email.com".toUri().also { data = it }
        putExtra(Intent.EXTRA_SUBJECT, "제목")
        putExtra(Intent.EXTRA_TEXT, "내용")
    }
    startActivity(emailIntent)
    
    // uri 해당하는 페이지로 이동
    val intent = Intent(Intent.ACTION_VIEW)
    intent.data =
        "https://github.com/woowacourse-study/2025-Manifest-Android-Interview-Study".toUri()
    startActivity(intent)
    ```
    
- 카테고리 LAUNCHER와 HOME의 차이
    
    안드로이드 인텐트 카테고리: LAUNCHER vs HOME 차이점
    안드로이드에서 `category.LAUNCHER`와 `category.HOME`는 각각의 목적과 사용처가 분명히 다릅니다.
    
    1. `android.intent.category.LAUNCHER`
    •	목적:
    앱의 메인 액티비티(시작 화면)를 **런처(앱 서랍, 홈 화면)**에 아이콘으로 등록합니다.
    •	사용 예시:
    일반적인 앱의 메인 액티비티에 사용하여, 사용자가 앱 목록에서 아이콘을 눌러 앱을 실행할 수 있게 합니다.
    •	동작:
    이 카테고리가 붙은 액티비티는 앱 서랍이나 홈 화면에 아이콘이 나타나고, 사용자가 아이콘을 눌렀을 때 실행됩니다.
    •	특징:
    •	대부분의 일반 앱에서 사용
    •	여러 액티비티에 각각 지정하면 앱 아이콘이 여러 개 나타날 수 있음.
    2. `android.intent.category.HOME`
    •	목적:
    **홈 런처(홈 화면 앱)**를 만들 때 사용합니다.
    •	사용 예시:
    새로운 홈 런처 앱(예: Nova Launcher, One UI Home 등)을 개발할 때, 홈 버튼을 눌렀을 때 실행될 액티비티에 지정합니다.
    •	동작:
    사용자가 홈 버튼을 누르면, 이 카테고리가 붙은 액티비티를 실행할 수 있는 앱 목록이 뜨고, 기본 홈 앱으로 설정할 수 있습니다.
    •	특징:
    •	일반 앱에서는 거의 사용하지 않음
    •	홈 화면 역할을 하는 앱(런처 앱)에서만 사용
    •	기기 부팅 시 가장 먼저 실행되는 홈 화면을 대체할 수 있음.

❓ 명시적 인텐트와 암시적 인텐트의 차이점은 무엇이며, 각각 어떤 시나리오에서 사용해야 하나요?

- 실행할 컴포넌트(액티비티, 서비스,…)를 정확히 명시하느냐와 아니냐의 차이가 있습니다.
- 명시적 인텐트의 경우 앱 내부에서 현재 액티비티에서 다른 액티비티로 이동할 때 사용할 수 있습니다. 예를 들어, 채팅이라는 서비스를 이용할 때 방을 클릭해서 채팅방 내부로 이동하는 경우의 화면전환이 있습니다.
    
    암시적 인텐트의 경우 작업의 의도만 다른 앱에 전달하여 컴포넌트가 선택되도록 할 때 사용할 수 있습니다. 예를 들어, 사용자가 링크를 클릭할 경우 웹 브라우저로 이동하는 경우가 있습니다. 다른 경우로는 번호에 암시적 인텐트를 적용해 사용자 기기의 다이얼에 번호를 입력해주는 경우가 있습니다. 
    
    암시적 인텐트 사용에서 주의해야 할 점으로는 개인정보 민감 작업은 제한될 수 있습니다.
    

❓안드로이드 시스템은 암시적 인텐트를 처리할 앱을 어떻게 결정하며, 적합한 애플리케이션을 찾지 못하면 어떻게 되나요?

- 시스템이 인텐트 필터를 통해 적합한 앱을 찾고, 없으면 예외가 발생합니다.