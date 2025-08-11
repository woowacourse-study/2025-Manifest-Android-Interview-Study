# 예외(exceptions)를 어떻게 추적하나요?

---

### 안드로이드에서의 예외처리

- Logcat을 이용한 예외 로깅
    - 예외 유형, 메시지 및 예외가 발생한 코드 줄을 포함한 자세한 stack trace 를 기록
    - Logcat 종류
        - V(Verbose) : 상세한 정보 확인을 위해 사용
        - D(Debug) : 디버깅 목적
        - I(Info) : 일반적인 정보 메시지, 프로그램의 정상적인 동작 확인 목적
        - W(Warn) : 경고 메시지, 잠재적인 문제나 주의가 필요한 상황을 나타냄
        - E(Error) : 오류 메시지, 프로그램이 비정상적으로 동작하거나 오류가 발생했음을 알림
        - F(wtf, Fatal) : 심각한 오류 메시지, 프로그램이 더 이상 정상적으로 실행될 수 없는 상황을 알림
    - 예시 ( 로그를 필터링해서 검색할 수도 있다. )
        
        ```kotlin
        val tv = findViewById<TextView>(R.id.tv_intent)
        tv.setOnClickListener {
            throw NullPointerException("예외 발생")
        }
        ```
        
- try-catch를 이용한 예외 처리
    - try-catch 블록을 사용해서 예외를 제어하고 앱 크래시를 방지할 수 있다.
    
    ```kotlin
    override fun adapt(call: Call<T>): Any =
            suspend {
                try {
                    val response = call.execute()
                    val code = response.code()
                    val body = response.body()
                    val message = response.message()
    
                    when {
                        response.isSuccessful && body != null -> ApiResult.Success(body)
                        code in 400..499 -> ApiResult.ClientError(code, message)
                        code in 500..599 -> ApiResult.ServerError(code, message)
                        else -> ApiResult.ApiError(code, message)
                    }
                } catch (e: IOException) {
                    ApiResult.NetworkError
                } catch (e: Exception) {
                    ApiResult.UnknownError(e)
                }
            }
    ```
    
- 전역 예외 핸들러 사용
    - Thread.setDefaultUncaughtExceptionHandler 사용하여 전역 예외 핸들러 설정
    - 중앙 집중식 오류 보고 또는 로깅에 유용
    - 애플리케이션 전체의 런타임 문제를 디버깅하고 모니터링하는 데 매우 효과적
    
    ```kotlin
    class MyApplication : Application() {
    
        override fun onCreate() {
            super.onCreate()
    
            // 전역 예외 처리기 등록
            Thread.setDefaultUncaughtExceptionHandler { thread, throwable ->
                // 예외 로그 출력
                Log.e("UncaughtException", "Thread: ${thread.name}, Exception: ${throwable.message}")
                Log.e("UncaughtException", Log.getStackTraceString(throwable))
    
                // 예외 정보를 파일로 저장하거나 서버 전송 가능
                // 또는 사용자에게 예외 발생 정보를 제공할 수도 있다.
    
                // 앱 종료 처리 (안 하면 앱이 불안정 상태로 남을 수 있음)
                android.os.Process.killProcess(android.os.Process.myPid())
                exitProcess(10)
            }
        }
    }
    ```
    
- Firebase Crashlytics
    - Firebase 콘솔에서 프로젝트 설정 + google-service.json 파일 프로젝트에 추가 + build.gradle 설정이 필요하다.
    - 프로덕션 환경에서 예외를 추적하는 도구
    - 처리되지 않은 예외를 자동 기록, stack trace, 기기 상태 및 사용자 정보와 함께 자세한 보고서 제공
    - 내부적으로 UncaughtExceptionHandler를 등록한다.
    - Thread.setDefaultUncaughtExceptionHandler와 Crashlytics는 반드시 한 번만 등록 가능하므로, 중복으로 사용할 경우 나중에 등록한 핸들러가 앞서 등록한 핸들러를 대체한다.
        - 여러 개를 사용하고 싶다면 인스턴스화 해서 핸들러 내부에서 생성한 인스턴스의 메서드를 호출하면 된다.
    - 전역 사용 예시
    
    ```kotlin
    class MyApplication : Application() {
        override fun onCreate() {
            super.onCreate()
            val defaultHandler = Thread.getDefaultUncaughtExceptionHandler()
            Thread.setDefaultUncaughtExceptionHandler { thread, exception ->
                Log.e("GlobalHandler", "Uncaught exception in thread ${thread.name}: ${exception.message}", exception)
                // 예외 세부 정보 저장 또는 서드 파티 솔루션으로 전송 (Crashlytics 등)
                // FirebaseCrashlytics.getInstance().recordException(exception)
                // 기존 핸들러 호출 (선택 사항, 시스템 기본 크래시 동작 유지)
                defaultHandler?.uncaughtException(thread, exception)
            }
        }
    }
    ```
    
    - ContentProivder에 의해 Thread.getDefaultUncaughtExceptionHandler() 는 Crashlytics 가 등록된다. ( 기본 initOrder는 100으로 설정되어 있다. )
    - 코드 내에서 사용 예시
    
    ```kotlin
    try {
        val data = fetchData()
    } catch (e: IOException) {
        FirebaseCrashlytics.getInstance().recordException(e)
    }
    ```
    
- 브레이크포인트를 이용한 디버깅
    - 디버그 모드 활성화하고 브레이크포인트에 도달했을 때 변수, 메서드 호출 및 예외 스택 트레이스를 상세하게 탐색 가능하다.
- 버그 리포트 캡처
    - 안드로이드에서 버그 리포트를 캡처하면 기기 로그, stack trace 및 시스템 정보를 수집하여 문제를 진단하고 수정하는데 도움이 된다. ADB는 서드파티 솔루션 없이 버그 리포트를 캡처하고 생성하는 방법을 제공한다.
        1. 설정 > 개발자 옵션 > 버그 신고 / 버그 신고 유형을 선택하고 생성된 보고서를 공유
        2. Android Emulator에서 버그 신고 캡처 : 확장 컨트롤을 열고 버그 신고를 선택한 다음 관련 세부 정보와 함께 보고서를 저장
        3. ABD(Android Debug Bridge)를 사용하여 버그 신고 캡처 : 터미널에서 adb bugreport /path/to/save/bugreport 실행하거나, adb -s <divice_serial_number> bugreport 로 특정 기기를 지정
    - 생성된 zip 파일에는 디버깅에 필수적인 dumpsys, dumpstate, logcat과 같은 로그가 포함되어 있다.
        

### Q. Logcat을 사용하여 개발 환경에서 예외를 디버깅하는 것과 Firebase Crashlytics와 같은 도구를 사용하여 프로덕션 환경에서 예외를 처리하는 것의 차이점은 무엇인가요? 또한, Logcat과 같은 로컬 환경에서 추적된 예외랑 프로덕션에서 추적된 예외를 각각 어떻게 해결하시나요?

Logcat 은 개발자가 연결한 기기에서 실시간으로 로그를 확인할 수 있고 Firebase Crashlytics 는 실제 사용자 기기에서 발생한 앱 크래시 및 예외를 수집할 수 있으며 집계된 데이터를 확인 및 분석할 수 있다.

Logcat 은 개발 중 발생하는 로그, 예외, 디버깅 목적인 반면 Crashlytics는 처리되지 않은 crash 를 목적으로 사용된다.

Logcat 은 IDE 에서 기본 제공하는 반면, Crashlytics는 Firebase 프로젝트 연동 및 설정이 필요하다.

Logcat 은 로깅만 지원하고, Crashlytics는 carsh 발생 시 이메일, Slack 등 외부 알림 및 분석 도구와 연동할 수 있다.

Logcat은 실시간 디버깅 및 로그를 확인하거나 필터링해서 문제 로그를 탐색할 수 있으며 강제로 예외를 발생시키는 테스트를 진행해볼 수 있다. 

Firebase Crashlytics는 실제 사용자 환경에서 발생한 crash를 자동 수집하여 기록하며 빈도와 영향도가 큰 이슈부터 순차적으로 해결할 수 있도록 할 수 있다. 다만, 사용자 환경이 다양해 동일한 재현이 어려울 수 있어 수집된 로그, 데이터, 사용자 피드백을 기반으로 원인을 유추해야 한다.

Logcat은 개발자가 직접 연결한 기기에서 실시간으로 상세 정보를 보며 디버깅하는 도구로 개발 및 테스트 단계에 최적화되어 있고 Firebase Crashlytics는 프로덕션 환경에서 발생하는 사용자들의 예외와 크래시를 자동으로 수집하고 분석해 신속한 문제 해결을 돕는 도구다.

Logcat, Firebase Crashlytics를 적절히 활용해 예외를 처리하는 것이 문제 해결에 효율적이다.

---

### 참고

https://developer.android.com/studio/debug/bug-report?hl=ko