# 안드로이드 예외 추적 방법

안드로이드에서 예외를 추적하는 것은 앱의 안정성과 사용자 경험을 향상시키는 데 중요하다. 

아래는 주요 예외 추적 방법들이다.

<br>

## ✅ Logcat을 이용한 예외 로깅

Logcat은 Android Studio에서 제공되는 기본 로그 도구이다. 

예외 발생 시 시스템은 예외 유형, 메시지, 발생 위치 등을 포함한 **스택 트레이스**를 Logcat에 기록한다.

- `E/AndroidRuntime` 등 키워드로 필터링 가능
- 즉각적인 확인 및 빠른 피드백 가능

<br>

## ✅ try-catch를 이용한 예외 처리

예외가 발생할 가능성이 있는 부분을 try-catch 블록으로 감싸 예외를 제어한다.

```kotlin
try {
    val result = performRiskyOperation()
} catch (e: Exception) {
    Log.e("Error", "Exception occurred: ${e.message}", e)
}

```

- 예외 발생 시 앱이 크래시되지 않도록 방지
- Logcat에 예외 정보 기록 가능

<br>

## ✅ 전역 예외 핸들러 사용하기

앱 전반의 **처리되지 않은 예외**를 포착할 수 있다.

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        val defaultHandler = Thread.getDefaultUncaughtExceptionHandler()
        Thread.setDefaultUncaughtExceptionHandler { thread, exception ->
            Log.e("GlobalHandler", "Uncaught exception in thread ${thread.name}: ${exception.message}", exception)

            // 예외 저장 또는 전송 (예: Firebase Crashlytics)
            // FirebaseCrashlytics.getInstance().recordException(exception)

            // 기존 핸들러 호출 (선택 사항)
            defaultHandler?.uncaughtException(thread, exception)
        }
    }
}

```

- 앱 전반에서 예외를 포착할 수 있음
- QA 빌드에서만 활성화하면 디버깅에 유용

<br>

## ✅ Firebase Crashlytics 사용하기

Firebase Crashlytics는 프로덕션 환경에서 발생한 크래시와 예외를 자동으로 수집하고 분석한다.

```kotlin
try {
    val data = fetchData()
} catch (e: IOException) {
    FirebaseCrashlytics.getInstance().recordException(e)
}

```

- 스택 트레이스, 기기 상태, 사용자 정보 등과 함께 보고됨
- 앱 배포 후에도 안정성 모니터링 가능

<br>

## ✅ 브레이크포인트(Breakpoints)를 이용한 디버깅

Android Studio에서 특정 코드 라인에 브레이크포인트를 걸어 코드 실행을 일시 중지하고 상태를 점검할 수 있다.

- 디버그 모드에서 변수, 스택 트레이스 확인
- 예외 발생 전후 상태를 직접 탐색 가능

<br>

## ✅ 버그 리포트(Bug Report) 캡처하기

기기의 전체 상태 및 로그를 저장하여 문제 해결에 활용할 수 있다.

### 1. 개발자 옵션을 통한 리포트

- 설정 > 개발자 옵션 > 버그 신고

### 2. Android Emulator에서 캡처

- Emulator > Extended Controls > Bug Report

### 3. ADB를 이용한 버그 리포트

```bash
adb bugreport /path/to/save/bugreport.zip
# 또는
adb -s <device_serial> bugreport

```

- ZIP 파일에는 logcat, dumpsys, dumpstate 등이 포함됨

<br>

## 🧩 요약

| 도구/방법 | 용도 | 환경 |
| --- | --- | --- |
| Logcat | 기본 예외 로그 확인 | 전체 |
| try-catch | 런타임 예외 방지 | 전체 |
| 전역 예외 핸들러 | 미처리 예외 추적 | 전체 / QA |
| Firebase Crashlytics | 프로덕션 예외 추적 | 릴리스 |
| Breakpoints | 상세 디버깅 | 개발 |
| 버그 리포트 | 시스템 전체 상태 캡처 | QA / 개발 |

---

예외 추적 및 관리는 앱의 품질에 직결된다. 

각 도구와 기법을 목적에 맞게 적절히 조합해 사용하면 더 안정적인 앱을 만들 수 있다.
