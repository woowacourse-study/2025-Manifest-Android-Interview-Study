# [25] 빌드 변형(build variants)과 플레이버(flavors)란 무엇인가요?

안드로이드 프로젝트에서 하나의 코드베이스로 여러 버전의 앱을 관리해야 할 때가 많다.

예를 들어, 광고가 있는 무료 버전과 광고가 없는 유료 버전, 혹은 개발용과 배포용 앱 등.

이럴 때 가장 유용하게 쓸 수 있는 기능이 바로 **빌드 변형(Build Variants)**과 제**품 플레이버(Product Flavors)**이다.

<br>

## 빌드 변형(Build Variants)이란?

**빌드 변형(Build Variant)**은 **빌드 타입(Build Type)**과 **제품 플레이버(Product Flavor)**를 조합한 결과이다. Android Gradle 플러그인은 이 조합을 기반으로 APK 또는 AAB 번들을 다양하게 생성할 수 있도록 도와준다.

예: `freeDebug`, `paidRelease` 등

<br>

## 빌드 타입(Build Types)

**빌드 타입**은 앱이 **어떻게 빌드되는가**를 정의한다.

기본적으로 Android 프로젝트에는 다음 두 가지 타입이 존재한다.

### Debug

- **개발 중 사용**되는 빌드 타입
- 로그 출력, 디버깅 도구, 테스트 도구가 활성화됨
- 일반적으로 `applicationIdSuffix = ".debug"`와 같은 설정을 통해 실제 출시 앱과 구분

### Release

- **배포용 최적화** 빌드 타입
- ProGuard 등 난독화 적용
- 리소스 최적화 및 릴리스 키로 서명 필요

> ✅ 커스텀 빌드 타입도 추가할 수 있다. (예: staging, internalTest 등)

<br>

## 제품 플레이버(Product Flavors)

**제품 플레이버**는 **앱의 변형(Variation)** 을 정의한다.

같은 코드베이스를 기반으로 다양한 앱 버전을 만들 수 있게 해준다.

### 예시: 무료 버전 vs 유료 버전

- `free` 플레이버는 광고 포함
- `paid` 플레이버는 프리미엄 기능 활성화

또는 지역 기반 앱 분리도 가능하다:

- `us`, `eu` 등의 지역별 플레이버

### 예제: `build.gradle.kts`에서의 설정

```kotlin
android {
    ...
    flavorDimensions += "version" // flavor 구분 차원

    productFlavors {
        create("free") {
            dimension = "version"
            applicationIdSuffix = ".free"
            versionNameSuffix = "-free"
        }
        create("paid") {
            dimension = "version"
            applicationIdSuffix = ".paid"
            versionNameSuffix = "-paid"
        }
    }
}
```

<br>

## 빌드 변형 조합하기

위 예시에서 `buildTypes`가 `debug`, `release`,

그리고 `productFlavors`가 `free`, `paid`라면 다음 **4가지 빌드 변형**이 생성된다:

- `freeDebug`
- `freeRelease`
- `paidDebug`
- `paidRelease`

각 조합은 서로 다른 리소스, 코드, 설정을 사용할 수 있어 **완전히 다른 앱처럼 동작**할 수 있다.

<br>

## 실전 사용 예시

### 예: 광고 여부에 따라 동작 분기

```kotlin
if (BuildConfig.FLAVOR == "free") {
    showAds()
} else {
    hideAds()
}
```

### 예: flavor별 리소스 분리

```
src/
 ├─ free/
 │   └─ res/values/strings.xml   // 광고 포함 텍스트
 ├─ paid/
 │   └─ res/values/strings.xml   // 광고 제거 텍스트
```

<br>

## 빌드 타입 & 플레이버 사용의 이점

### 1. **중복 코드 없이 다양한 버전 관리**

프로젝트를 복제하지 않고도 여러 앱 버전을 운영할 수 있다.

### 2. **커스텀 기능 제공**

유료 사용자에게만 특정 기능 제공, 디버그에서만 로깅 출력 등

### 3. **빌드 자동화**

변형에 따라 자동으로 서명, 난독화, 앱 이름/아이콘 변경 등 자동 처리 가능

<br>

## 자주 묻는 질문

### Q. **빌드 타입과 제품 플레이버의 차이는 무엇인가요?**

| 구분 | 빌드 타입 (Build Type) | 제품 플레이버 (Product Flavor) |
| --- | --- | --- |
| 목적 | **빌드 방식** 결정 (개발 vs 배포) | **앱 기능/내용** 결정 (무료 vs 유료, 지역별 등) |
| 기본 제공 | `debug`, `release` | 없음 (직접 정의 필요) |
| 주요 용도 | 디버깅 설정, 난독화, 로그 등 | 앱 ID, 리소스, 기능, 문자열 등 변경 |
| 조합 결과 | 빌드 변형(Build Variant) 생성 |  |

빌드 변형은 이 둘의 **교차 조합**으로 생성된다.

예: `free` 플레이버 + `debug` 타입 → `freeDebug` 변형

<br>

## 마무리

안드로이드의 **빌드 변형 시스템**은 하나의 프로젝트 안에서 다양한 앱을 손쉽게 빌드하고 배포할 수 있도록 돕는 강력한 도구이다.

> 필요하다면 staging, beta, internal, mockApi 등 다양한 빌드 타입 또는 플레이버를 조합하여 팀 내 요구사항에 맞는 구조를 만들 수도 있다.
> 
> 
> 단, 너무 많은 변형을 만들 경우 **복잡도 관리**가 어려워질 수 있으니 신중하게 설계하는 것이 좋다.
>
