# Android Service 정리

---

# Android Service 정리

---

## Service란?

Service는 사용자 인터페이스 없이 백그라운드에서 작업을 수행할 수 있도록 하는 Android 컴포넌트입니다.

Activity와 달리 UI가 없으며, 앱이 포그라운드에 없을 때도 실행될 수 있습니다.

**주요 용도**

- 파일 다운로드
- 음악 재생
- 네트워크 요청 처리
- 데이터 동기화

---

## Started Service

`startService()` 호출 시 시작되며, `stopSelf()` 또는 `stopService()`로 중지될 때까지 실행됩니다.

```kotlin
class MyService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // 장기 실행 작업 수행
        return START_STICKY
    }

    override fun onBind(intent: Intent?): IBinder? = null
}

```

> ⚠️ Android 8.0(API 26) 이상에서는 백그라운드에서 위치 정보를 직접 접근할 수 없습니다.
> 
> 
> 대신 `ForegroundService` 또는 `WorkManager` 사용 권장
> 

---

## Bound Service

다른 컴포넌트가 `bindService()`를 통해 연결할 수 있는 서비스입니다.

모든 클라이언트가 언바인드되면 자동으로 중지됩니다.

**주요 용도**

- 앱 내부 모듈 간 직접 통신
- 상태 확인 및 즉시 처리 필요 시

```kotlin
class BoundService : Service() {
    private val binder = LocalBinder()

    inner class LocalBinder : Binder() {
        fun getService(): BoundService = this@BoundService
    }

    override fun onBind(intent: Intent?): IBinder = binder
}

```

---

## Foreground Service

지속적인 알림(Notification)을 통해 사용자가 작업을 인지할 수 있도록 하는 서비스입니다.

장시간 실행되며, 시스템 자원이 부족할 때에도 상대적으로 종료 우선순위가 낮습니다.

**주요 용도**

- 음악 재생
- 위치 추적
- 내비게이션 안내

```kotlin
class ForegroundService : Service() {

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val notification = createNotification()
        startForeground(1, notification)
        return START_STICKY
    }

    private fun createNotification(): Notification {
        val channelId = "ForegroundServiceChannel"
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                channelId,
                "Foreground Service Channel",
                NotificationManager.IMPORTANCE_DEFAULT
            )
            getSystemService(NotificationManager::class.java)?.createNotificationChannel(channel)
        }

        return NotificationCompat.Builder(this, channelId)
            .setContentTitle("Foreground Service")
            .setContentText("Running...")
            .setSmallIcon(R.drawable.ic_notification)
            .build()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}

```

---

## 서비스 유형 비교

| 유형 | 백그라운드 실행 | 자동 중지 조건 | 알림 필요 여부 |
| --- | --- | --- | --- |
| Started Service | ✅ | ❌ 명시적 종료 필요 | ❌ |
| Bound Service | ✅ | ✅ 모든 클라이언트가 언바인드되면 종료 | ❌ |
| Foreground Service | ✅ | ❌ 명시적 종료 필요 | ✅ (필수) |

---

## Service vs Foreground Service

| 항목 | 일반 Service | Foreground Service |
| --- | --- | --- |
| 사용자 인지 | ❌ 표시 없음 | ✅ 알림을 통해 항상 표시 |
| 시스템 우선순위 | 낮음 (종료 가능성 높음) | 높음 (우선 유지됨) |
| 알림 필요 여부 | ❌ 없음 | ✅ 필수 (API 26 이상 필수 요건) |
| 주 사용 목적 | 짧은 백그라운드 작업 | 장기 작업 + 사용자 인지 필요 시 |
| 예시 | 데이터 저장, 동기화 | 음악 재생, 위치 추적, 타이머 등 |

---

## 생명주기

### Started Service

- `onCreate()` : 최초 생성 시 호출
- `onStartCommand()` : 작업 실행
- `onDestroy()` : 명시적으로 종료될 때 호출

### Bound Service

- `onCreate()` : 최초 생성 시 호출
- `onBind()` : 컴포넌트가 바인딩할 때 호출
- `onUnbind()` : 마지막 클라이언트 언바인드 시 호출
- `onDestroy()` : 서비스 종료 시 호출

---

## 예시로 이해하기

### Started Service – 위치 추적

```kotlin
class LocationTrackingService : Service() {

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        startForeground(1, createNotification())
        startLocationUpdates()
        return START_STICKY
    }

    private fun startLocationUpdates() {
        // 위치 추적 시작
    }

    private fun createNotification(): Notification {
        val channelId = "LocationChannel"
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                channelId,
                "Location Tracking",
                NotificationManager.IMPORTANCE_DEFAULT
            )
            getSystemService(NotificationManager::class.java)?.createNotificationChannel(channel)
        }

        return NotificationCompat.Builder(this, channelId)
            .setContentTitle("위치 추적 중")
            .setContentText("사용자의 위치를 추적 중입니다")
            .setSmallIcon(R.drawable.ic_location)
            .build()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}

```

### Bound Service – 음악 제어

```kotlin
class MusicService : Service() {

    private val binder = MusicBinder()
    private var playing = false

    inner class MusicBinder : Binder() {
        fun play() = playMusic()
        fun pause() = pauseMusic()
        fun isPlaying(): Boolean = playing
    }

    override fun onBind(intent: Intent?): IBinder = binder

    private fun playMusic() {
        playing = true
        // 음악 재생 로직
    }

    private fun pauseMusic() {
        playing = false
        // 음악 정지 로직
    }
}

```

```kotlin
class MusicActivity : AppCompatActivity() {

    private var musicService: MusicService? = null
    private var isBound = false

    private val connection = object : ServiceConnection {
        override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
            musicService = (service as MusicService.MusicBinder).let {
                it as MusicService
            }
            isBound = true
        }

        override fun onServiceDisconnected(name: ComponentName?) {
            musicService = null
            isBound = false
        }
    }

    override fun onStart() {
        super.onStart()
        Intent(this, MusicService::class.java).also {
            bindService(it, connection, Context.BIND_AUTO_CREATE)
        }
    }

    override fun onStop() {
        super.onStop()
        if (isBound) {
            unbindService(connection)
            isBound = false
        }
    }

    private fun onPlayButtonClick() {
        musicService?.play()
    }
}

```

---

## 요약

- **Service**: UI 없이 백그라운드 작업을 수행하는 Android 컴포넌트
- **Started Service**: 독립적 작업, 명시적 종료 필요
- **Bound Service**: 클라이언트와 직접 통신, 클라이언트 해제 시 종료
- **Foreground Service**: 사용자 인식 필요, 알림 필수, 시스템 우선순위 높음

각 서비스의 생명주기와 목적을 이해하고 적절히 활용하면, 효율적이고 안정적인 앱을 만들 수 있습니다.