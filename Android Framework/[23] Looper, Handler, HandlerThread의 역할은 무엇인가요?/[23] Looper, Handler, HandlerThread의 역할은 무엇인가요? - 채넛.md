 ## Looper, Handler, HandlerThread의 역할은 무엇인가요?
 
 Looper, Handler, HandlerThread : 안드로이드만의 메시지 처리 프레임워크의 핵심

AppCompatActivity → FragmentActivity → ComponentActivity → androidx.core.app.ComponentActivity → Activity 

Activity 의 멤버변수로 ActivityThread 타입의 변수를 갖고있다.

- 내부에 자바언어의 진입점을 의미하는 main() 함수가 존재하고 있고 이 부분이 안드로이드 앱의 시작점이자 메인 스레드가 된다.

```kotlin
// ActivityThread 내부 main 함수
 public static void main(String[] args) {
        // Trace 객체 : 성능 분석과 디버깅을 위해 코드 실행 구간을 추적하는 역할
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
				
	      // 
		      ... 
	      //
	      
	      // 메인 루퍼 준비 (안드로이드에서 제공)
        Looper.prepareMainLooper();
				
				// 안드로이드 프레임워크가 앱 프로세스 시작할 때 프로세스 인스턴스를 구분하기 위해 넘겨주는 값 찾기 
        long startSeq = 0;
        if (args != null) {
            for (int i = args.length - 1; i >= 0; --i) {
                if (args[i] != null && args[i].startsWith(PROC_START_SEQ_IDENT)) {
                    startSeq = Long.parseLong(
                            args[i].substring(PROC_START_SEQ_IDENT.length()));
                }
            }
        }
        
        // ActivityThread 기본생성자에서 ResourcesManager로부터 리소스 관리를 위한 인스턴스를 얻는다.
        ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

				// 로그 기능, Android OS 내부에서 로깅이 필요할 때를 위한 기능 (일반 앱 개발자가 직접 사용할 일 X)
        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // 루퍼 실행 전 Trace 객체(앱 프로세스 초기화 같은 특정 구간 성능 ) 리소스 정리
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }

```

- 앱 프로세스의 UI 메인 스레드를 초기화하고, 무한 메시지 루프를 돌면서 앱의 모든 UI/이벤트를 처리

```kotlin
//Looper
    private static boolean loopOnce(final Looper me,
            final long ident, final int thresholdOverride) {
        Message msg = me.mQueue.next(); // might block
        if (msg == null) { // 메시지 큐에 메시지가 없다면 false 반환
            return false;
        }
				
				// ... 로깅, 옵저빙, 성능 측정 로직 ... 
       
        try {
            msg.target.dispatchMessage(msg); // 핸들러에게 메시지 전송
            if (observer != null) { 
                observer.messageDispatched(token, msg); // 옵저버에게 메시지 처리 완료 알림
            }
            dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
        } catch (Exception exception) {
            if (observer != null) { 
                observer.dispatchingThrewException(token, msg, exception);
            }
            throw exception;
        } finally {
            ThreadLocalWorkSource.restore(origWorkSource);
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }
        
        // ... 로깅 및 성능에 문제 있을 때의 로직들 ...

        msg.recycleUnchecked(); // 메시지 객체 재활용할 수 있도록 호출 (내부에서 상태 초기화 및 재활용)

        return true;
    }

```

```kotlin
// Looper
public static void loop() {
	       // .. 기타 로직 ..
	       
        for (;;) { // 무한 루프
            if (!loopOnce(me, ident, thresholdOverride)) {
                return;
            }
        }
    }
```

### Looper, Handler, HandlerThread

→ 스레드를 관리하고 비동기 통신을 처리하기 위해 작동하는 컴포넌트

→ 백그라운드에서 작업하며 UI 업데이트를 위해 메인스레드와 상호작용하기 위해 필수적인 컴포넌트

## Looper

- MessageQueue 기반의 event loop를 관리하는 데 사용되는 클래스
    - MessageQueue에 있는 message, runnable을 계속해서 하나씩 꺼내주는 작업을 한다.
- Looper는 스레드에 종속되며, MessageQueue는 각 Looper에 종속된다.
- 루퍼는 자신을 생성한 Thread Local Storage에 저장된다.
    - 메인 스레드 일 경우, 안드로이드 프레임워크에서 직접 생성해준다.
        - prepareMainLoop()가 deprecated(API 30이후), 안드로이드 프레임워크에서 직접 생성해준다.
        - 그렇기에 getMainLooper()를 통해 가져다 사용하기만 하면된다.
    - 일반 스레드 일 경우, 개발자가 직접 생성한다.
        - prepare() 함수를 사용하여 생성할 수 있다.
        - loop() 를 호출하여 Looper가 작동하게 할 수 있다.
- 메시지를 처리하는 모든 스레드에는 Looper가 필요하다
- 사용예시
    
    ```kotlin
    class LooperThread : Thread() {
        var mHandler: Handler? = null
    
        override fun run() {
            Looper.prepare()
    
            mHandler = object : Handler(Looper.myLooper()!!) {
                override fun handleMessage(msg: Message) {
                    // process incoming messages here
                }
            }
    
            Looper.loop()
        }
    }
    ```
  
    - 스레드와 메시지 큐를 관리하기 위한 기반 ( 루퍼와 스레드 )
    - 루퍼는 작업을 지속 처리할 수 있도록 보장, 핸들러는 작업 통신을 위한 인터페이스 제공
    
    - prepare를 호출하면 스레드에 Looper를 연결
    - loop를 호출하면 루프를 시작
    

### Handler

- 스레드의 메시지 큐 내에서 메시지나 작업을 보내고 처리하는 데 사용
- 다른 스레드로 작업 또는 메시지 전달 ( 백그라운드에서 UI 업데이트, … )
- 메시지 루프와 대부분의 상호작용은 Handler 클래스를 통해 이루어 진다.
- 반드시 Looper를 명시적으로 지정해야 한다.
    - Looper를 암시적으로 선택했을 때의 문제
        - 핸들러가 새 작업을 기다리지 않고 종료하는 경우의 작업 손실 가능성
        - 루퍼가 활성화되지 않은 스레드에서 핸들러가 생성되는 경우의 충돌
        - 핸들러가 연결된 스레드가 개발자가 예상한 것과 다를 때의 충돌
- send, post 방식 존재
    - send : Message 전송할 때
    - post : Runnable 형태의 Message 전송할 때

### HandlerThread

- 내장된 Looper를 갖는 thread
- 백그라운드 스레드 생성하는 과정을 단순화
- Looper를 가진 워커 스레드 생성해서 해당 스레드에서 작업을 순차 처리

### Message

- 실제 수행해야할 작업에 대한 명세가 들어있는 자료구조
- MessageQueue는 Message를 담는 자료구조
- Message 객체는 사용한 객체를 초기화하여 재사용하는 방식인 오브젝트 풀 방식을 사용한다
- MessageQueue는 중간 삽입의 가능성도 존재하므로 LinkedQueue로 구현되어 있다.

### 참고

https://medium.com/write-android/%EB%A9%94%EC%9D%B8-%EC%8A%A4%EB%A0%88%EB%93%9C%EC%99%80-handler-part-1-handler%EC%99%80-looper-%EA%B7%B8%EB%A6%AC%EA%B3%A0-messagequeue%EC%9D%98-%EB%8F%99%EC%9E%91-%EB%B0%A9%EC%8B%9D-f0bee443d71e

https://medium.com/android-dev-br/threads-handler-looper-e-message-queue-parte-2-3711ee9d1b4