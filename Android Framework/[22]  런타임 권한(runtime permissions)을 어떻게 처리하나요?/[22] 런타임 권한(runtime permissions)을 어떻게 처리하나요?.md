## 22. 런타임 권한(runtime permissions)을 어떻게 처리하나요?

- Android 6.0 (API 23)부터는 **위험 권한(Dangerous Permissions)** 을 앱 실행 중에 명시적으로 요청해야 한다.
- 이는 사용자가 기능을 사용할 **직전에 권한을 요청하도록 하여 개인정보 보호를 강화**하기 위한 정책이다.
- 사용자는 특정 권한 요청을 **거부하거나 "다시 묻지 않기" 선택 가능**, 이에 따라 앱은 유연하게 대응해야 한다.

<br>

### 위험 권한 (Dangerous Permissions) 종류

- `CAMERA`, `RECORD_AUDIO`, `READ_CONTACTS`, `ACCESS_FINE_LOCATION`, `READ_EXTERNAL_STORAGE`, 등.
- [docs](https://developer.android.com/reference/android/Manifest.permission)

<br>

### 권한 선언

권한을 요청하기 전에 AndroidManifest.xml 파일에 필요한 권한을 선언해야 한다.

```xml
<uses-permission android:name="android.permission.CAMERA" />
```

> 📌 위험 권한은 **AndroidManifest.xml**에 선언 후, **런타임에 별도로 요청**해야 사용 가능.

<br>

### 권한 상태 확인 및 요청 흐름

- 런타임 시에는 사용자가 해당 권한이 필요한 기능과 상호 작용할 때만 권한을 요청해야 한다.
- 사용자에게 요청하기 전에 `ContextCompat.checkSelfPermission()`을 사용하여 권한이 이미 부여되었는지 확인하는 것이 중요하다.


```kotlin
when {
    ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) == PackageManager.PERMISSION_GRANTED -> {
        // ✅ 이미 권한 있음
        startCamera()
    }

    ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.CAMERA) -> {
        // ❗ 이전에 거부한 적이 있어 rationale 필요
        showPermissionRationale()
    }

    else -> {
        // 🚀 권한 요청
        requestPermissionLauncher.launch(Manifest.permission.CAMERA)
    }
}
```

<br>

### 권한 요청하기

- 권한 요청에 권장되는 방식은 권한 처리를 단순화하는 `ActivityResultLauncher` API를 사용하는 것이다.
- 해당 API를 사용하면 시스템은 사용자에게 권한 요청을 허용하거나 거부하도록 안내한다.

```kotlin
private val requestPermissionLauncher = registerForActivityResult(
    ActivityResultContracts.RequestPermission()
) { isGranted: Boolean ->
    if (isGranted) {
        // ✅ 권한 부여됨
        startCamera()
    } else {
        // ❌ 권한 거부됨
        showPermissionDeniedDialog()
    }
}
```

> ✅ **ActivityResult API**를 통해 콜백 기반으로 권한 결과를 쉽게 처리 가능.

<br>

### 권한 요청 근거(Rationale) 제공하기

- 경우에 따라 시스템은 shouldShowRequestPermissonRationale()을 사용하여 권한을 요청하기 전에 해당 기능을 사용하기 위해 권한이 필요한 근거(Rationale)를 표시할 것을 권장한다.
- 이는 사용자 경험을 개선하고 권한 획득 가능성을 높인다.

```kotlin
fun showPermissionRationale() {
    AlertDialog.Builder(this)
        .setTitle("권한 필요")
        .setMessage("이 기능이 제대로 동작하기 위해 카메라 접근 권한이 필요합니다.")
        .setPositiveButton("확인") { _, _ ->
            requestPermissionLauncher.launch(Manifest.permission.CAMERA)
        }
        .setNegativeButton("취소", null)
        .show()
}
```

> ⚠️ 사용자 경험을 고려해, **거부 이력이 있는 경우 rationale 제공** 권장.

<br>

### 권한 거부 처리하기

사용자가 권한을 여러 번 거부하면 안드로이드는 이를 영구 거부로 처리하여 앱이 다시 요청할 수 없게 된다.
앱은 사용자에게 기능 제한에 대해 알리고 필요한 경우 시스템 설정으로 안내해야 한다.

```kotlin
if (
    ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED &&
    !ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.CAMERA)
) {
    // ❗ 영구 거부 상태
    showSettingsDialog()
}
```

```kotlin
fun showSettingsDialog() {
    AlertDialog.Builder(this)
        .setTitle("권한 필요")
        .setMessage("앱 설정에서 카메라 권한을 수동으로 허용해주세요.")
        .setPositiveButton("설정 열기") { _, _ ->
            val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).apply {
                data = Uri.parse("package:$packageName")
            }
            startActivity(intent)
        }
        .setNegativeButton("취소", null)
        .show()
}
```

<br>

### 위치 권한 처리

- 위치 권한은 Foreground 및 Background 접근으로 분류된다.
- Foreground 위치 접근에는 ACCESS_FINE_LOCATION 또는 ACCESS_COARSE_LOCATION이 필요하다.
- Background 접근에는 추가적으로 ACCESS_BACKGROUND_LOCATION 권한이 필요하다.

```xml
<uses‑permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses‑permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
```

안드로이드 10 (API 29)부터 Background 위치를 요청하는 앱은 먼저 Foreground 접근 권한을 요청한 다음 별도로 백그라운드 권한을 요청해야 한다.
- **절차적 흐름**
  1. 먼저 ACCESS_FINE_LOCATION (or COARSE) 요청
  2. 이후 별도로 ACCESS_BACKGROUND_LOCATION 요청
 
> ⚠️ background 권한은 **정당한 이유가 없는 경우 승인되지 않을 수 있음** (Google Play 정책 강화)

<br>

### 일회성 권한

- 안드로이드 11 (API 30)은 위치, 카메라, 마이크에 대해 [일회성 권한](https://developer.android.com/training/permissions/requesting#one‑time)을 도입했다.
- 사용자는 임시적으로 접근 권한을 부여할 수 있으며, 앱을 종료하면 해당 권한은 사라진다.

<br>

## Best Practice


### 카메라 권한

- 기능 실행 시점에 요청 (앱 시작 시 아님)
- "다시 묻지 않음" 선택 시 설정 유도 UI 제공
- 예시: QR 스캔, 프로필 사진 등록 등 **명확한 트리거가 있을 때 요청**

<br>

### 위치 권한

- **Foreground 위치 권한만으로도 대부분의 기능 구현 가능**
- Background 권한은 다음 조건에 해당할 경우에만 요청:
  - 지속적인 위치 추적
  - 위치 기반 알림/마케팅
- 위치 권한 요청 전, **권한이 필요한 기능을 먼저 보여준 후 요청**

<br>

### 마이크 권한

- 권한이 필요한 경우, **"녹음 중"이라는 시각적 UI 표시** 필수
- 녹음 목적(예: 음성 검색, 통화 녹음 등)을 명확히 사용자에게 전달
- Android 12 이상에서는 마이크 사용 중 상태바에 아이콘이 자동 표시됨

<br>

## 정리

안드로이드의 런타임 권한 시스템은 사용자가 민감한 정보를 앱에 노출하기 전에, 반드시 명시적으로 동의할 수 있도록 설계되어 있다.
이는 사용자의 개인정보를 불필요하게 수집하거나 오용하지 못하도록 하는 데 중요한 역할을 한다.

앱 개발자는 민감한 권한을 요청할 때 몇 가지 시나리오를 고려해야 한다. 
- 첫째, 권한은 앱 실행 시가 아니라, 사용자가 해당 기능을 실제로 필요로 하는 시점에 요청하는 것이 가장 좋다.
- 둘째, 사용자가 이전에 권한을 거부했을 경우, 왜 이 권한이 필요한지 명확히 설명해주는 안내 UI를 제공해야 한다.
- 셋째, 권한이 거부되었을 때 앱이 어떤 식으로 제한되거나 대체 기능을 제공하는지도 고려해야 한다.
- 마지막으로 ‘다시 묻지 않음’ 상태에서는 설정 화면으로 자연스럽게 유도하는 흐름이 필요하다.

이런 설계를 통해 앱은 사용자 경험을 해치지 않으면서도 개인 정보를 보호하는 방향으로 작동할 수 있다.

<br>

## 참고 문서

- [Android 권한 가이드 (공식)](https://developer.android.com/training/permissions/requesting)
- [Android Manifest Intervice](https://leanpub.com/manifest-android-interview-kr)


