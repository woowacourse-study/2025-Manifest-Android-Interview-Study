Bundle은 Activity, Fragment, Service와 같은 컴포넌트 간에 데이터를 전달하는 데 사용되는 키-값 쌍 데이터 구조이다. <br>
일반적으로 앱 내에서 작은 용량의 데이터를 효율적으로 전송하는 데 사용된다.  <br>
Bundle은 가볍고 안드로이드 운영 체제가 쉽게 관리하고 전송할 수 있는 형식으로 데이터를 직렬화하도록 설계되었다. <br>

<br>

## Bundle의 일반적인 사용 사례

### Activity간 데이터 전달

새 Activity를 시작할 때 Intent에 Bundle을 담아서 대상 Activity에 데이터를 전달할 수 있다.

```kotlin
val intent = Intent(this, ActivityB::class.java).apply {
    putExtra("user_name", "John Doe")
    putExtra("user_age", 25)
}

startActivity(intent)
```

데이터는 Intent.putExtra()를 통해 내부적으로 Bundle에 패키징된다.

<br>

### Fragment간 데이터 전달

Fragment 트랜잭션에서 Bundle은 `setArguments()` 및 `getArguments()`와 함께 전달되어 Fragment 간에 데이터를 보낸다.

```kotlin
// Sending data to Fragment
val fragment = MyFragment().apply {
    arguments = Bundle().apply {
        putString("user_name", "John Doe")
        putInt("user_age", 30)
    }
}

// Retrieving data in Fragment
val name = arguments?.getString("user_name")
val age = arguments?.getInt("user_age")
```

데이터는 Fragment의 arguments를 통해 내부적으로 Bundle에 패키징된다.

<br>

### 인스턴스 상태 저장 및 복원

Bundle은 `onSaveInstanceState()` 및 `onRestoreInstanceState()`와 같은 생명주기 메서드에서 구성 변경 중에 임시 UI 상태를 저장하고 복원하는 데 사용된다.

```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    outState.putString("user_input", editText.text.toString())
}

override fun onRestoreInstanceState(savedInstanceState: Bundle) {
    super.onRestoreInstanceState(savedInstanceState)
    val userInput = savedInstanceState.getString("user_input")
    editText.setText(userInput)
}
```

Bundle은 화면 회전과 같은 구성 변경으로부터 사용자가 입력했던 값이 보존되도록 한다.

<br>

### Service에 데이터 전달

Service를 시작할 때도 Intent에 Bundle을 담아 데이터를 전달할 수 있다. 

이때 `Intent.putExtras()` 또는 `putExtra()`를 사용한다.

```kotlin
// Service 시작 시 데이터 전달
val bundle = Bundle().apply {
    putString("command", "upload")
    putInt("file_count", 3)
}

val serviceIntent = Intent(this, MyService::class.java).apply {
    putExtras(bundle)
}

startService(serviceIntent)
```

```kotlin
class MyService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val command = intent?.getStringExtra("command")
        val fileCount = intent?.getIntExtra("file_count", 0)

        // 전달된 데이터 사용
        Log.d("MyService", "Command: $command, Files: $fileCount")

        return START_NOT_STICKY
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
```

<br>

## Bundle 작동 방식

Bundle은 데이터를 키-값 구조로 직렬화하여 작동한다. <br>
키는 문자열이며 값은 기본 유형, Serializable, Parcelable 객체 또는 다른 Bundle일 수 있다. <br>
이를 통해 데이터를 효율적으로 저장하고 전송할 수 있다. <br>

<br>

## Bundle에 저장 가능한 데이터 유형

```kotlin
val bundle = Bundle()

// 기본 자료형
bundle.putBoolean("is_logged_in", true)
bundle.putInt("user_age", 25)
bundle.putFloat("user_rating", 4.5f)

// 문자열 및 시퀀스
bundle.putString("user_name", "John Doe")
bundle.putCharSequence("description", "안녕하세요")

// 배열
bundle.putIntArray("scores", intArrayOf(100, 90, 95))
bundle.putStringArray("tags", arrayOf("android", "kotlin"))

// Serializable 객체
bundle.putSerializable("user_object", user as Serializable)

// Parcelable 객체
bundle.putParcelable("profile", userProfile)

// Parcelable 객체 리스트
bundle.putParcelableArrayList("item_list", arrayListOf(item1, item2))

// 중첩 Bundle
val innerBundle = Bundle()
innerBundle.putString("nested_key", "value")
bundle.putBundle("nested_bundle", innerBundle)
```

<br>

## 요약

Bundle은 컴포넌트 및 생명주기 이벤트 간에 데이터를 효율적으로 전달하고 보존하기 위한 안드로이드의 중요한 구성 요소이다. <br>
가볍고 유연한 구조로 인해 애플리케이션 상태 및 데이터 전송 관리에 필수적인 도구이다. <br>

