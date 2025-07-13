# Serializable과 Parcelable의 차이점은 무엇인가요 ?

`PPT 만들기 전 자료조사 파일 입니다`

### 직렬화를 사용하는 이유

- 각각의 프로세스는 별도의 메모리를 갖기 때문에 프로세스 간 데이터 공유를 위해 별도의 기법이 필요하다.
- IPC를 통해 우리는 다른 프로세스에 byte-stream을 전달할 수 있고, byte-stream 형태로 만들기 위해 직렬화가 필요하다.

cf. 별도의 기법 : IPC(Inter-Process Communication)

Serializable

- java.io.Serializable 인터페이스를 구현하여 직렬화, 역직렬화를 지원한다.
- Android 의존성을 갖지 않는 표준 Java 인터페이스
- 별도로 구현해야 하는 메서드가 없는 마커 인터페이스로, JVM 내부에서 직렬화, 역직렬화 해준다.
    - JVM 내부에서 처리하기에 자바에서 제공하는 타입, Serializable 타입 외 커스텀 타입은 처리해주지 않는다.
- 리플렉션 기반으로 구현되어있고 임시 객체들이 생성되며 GC 호출로 인해 성능에 영향을 줄 수 있다.
- writeObject, readObject, readObjectNoData 메서드를 구현하여 커스텀해서 사용할 수 있다.
- DeepDive Point : 공식문서에서는 Effective Java의 직렬화 관련 목차를 확인하는 것을 제안한다.
- 사용 예시
    
    ```kotlin
    class Person(val firstName: String, val lastName: String, val age: Int) : Serializable
    ```
    

Parcelable

- Android 의존성을 갖고 있으며 직렬화, 역직렬화를 지원한다.
- 리플렉션을 사용하지 않는다.
- 안드로이드 컴포넌트 내에서 프로세스 간 통신(IPC)이라면 사용하는 것을 권장한다.

- 사용 예시
    
    ```kotlin
    class Person(
        val firstName: String, val lastName: String, val age: Int
    ) : Parcelable {
        constructor(parcel: Parcel) : this(
            parcel.readString().toString(),
            parcel.readString().toString(),
            parcel.readInt()
        ) {
        }
    
    		// 직렬화
        override fun writeToParcel(parcel: Parcel, flags: Int) {
            parcel.writeString(firstName)
            parcel.writeString(lastName)
            parcel.writeInt(age)
        }
    
    		// 특별한 객체 타입이 포함되어 있는 지(특별한 객체가 있을 경우 그에 맞는 플래그를 반환)
        override fun describeContents(): Int {
            return 0
        }
    
    		// 복원하거나, 배열을 생성하는 역할(역직렬화)
        companion object CREATOR : Parcelable.Creator<Person> {
            override fun createFromParcel(parcel: Parcel): Person {
                return Person(parcel)
            }
    
            override fun newArray(size: Int): Array<Person?> {
                return arrayOfNulls(size)
            }
        }
    
    }
    ```
    

Parcelable 구현체 제공

```kotlin
// build.gradle
plugins {
    id("kotlin-parcelize")
}
```

- build.gradle에 해당 플러그인을 추가한다.

```kotlin
import kotlinx.parcelize.Parcelize

@Parcelize
class User(val firstName: String, val lastName: String, val age: Int): Parcelable
```

- @Parcelize 어노테이션을 기본 생성자에 사용하면 직접 구현 메서드를 작성하지 않아도 된다.
- 기본 생성자 외의 필드에는 적용 되지 않는다.
    - 기본 생성자에서 직렬화를 제외하고 싶으면 @IgnoredOnParcel 어노테이션을 붙인다.
- 커스텀 직렬화 로직이 필요하다면 companion class 내부에 Parceler<T> 를 구현할 수 있다.

### Parcel

- 안드로이드에서 컴포넌트간의 고성능 IPC 전송을 가능하게 하는 컨테이너 클래스
- 마샬링(직렬화), 언마샬링(역직렬화)하여 안드로이드의 IPC 통신에서 데이터를 전송할 수 있게 한다.
- Parcelable은 객체를 직렬화하여 Parcel을 통해 전달할 수 있도록 한다.
- Parcelable을 구현한 객체는 Parcel에 쓰고 복원할 수 있어 컴포넌트 간 복잡한 데이터 전달하는 데 적합하다.

❓안드로이드에서 Serializable과 Parcelable의 차이점은 무엇이며, 일반적으로 컴포넌트 간 데이터 전달에 Parcelable이 선호되는 이유는 무엇인가요?

- Serializable은 Java 표준 직렬화 인터페이스고, Parcelable은 안드로이드에 의존적이다.
- Serializable은 리플렉션 기술을 사용하여 직렬화, 역직렬화 과정에서 임시 객체들을 생성하고, 이로 인해 GC 부담이 커져 성능이 저하될 수 있는 반면 Parcelable은 직렬화, 역직렬화 과정을 개발자가 작성하기에 불필요한 객체 생성이 발생하지 않고 리플렉션을 사용하지 않아 성능 문제를 개선할 수 있다.
- Parcelable이 안드로이드 환경에 맞춰 개발되었다. 구현이 번거롭다는 단점이 있었지만 플러그인을 추가해 @Parcelize 어노테이션을 사용한다면 보일러 플레이트 코드도 줄일 수 있게 되었다.
- 추가적으로 리플렉션을 사용하지 않아 속도가 빠르다는 장점이 컴포넌트 간 데이터 전달에서 선호하는 이유가 될 수 있다.
- Serializable은 Java 표준이므로 Android 외의 환경과 호환성이 필요한 경우 선호될 것 같다.

### 참고

https://developer.android.com/reference/java/io/Serializable

https://developer.android.com/kotlin/parcelize#kts

https://medium.com/jaesung-dev/android-%EC%A7%81%EB%A0%AC%ED%99%94%EC%99%80-%EC%97%AD%EC%A7%81%EB%A0%AC%ED%99%94-18fd04f1c0ed

---

## 직렬화 라이브러리

### kotlinx.serialization

- https://kotlinlang.org/docs/serialization.html
- @Serializable 어노테이션을 붙여야 한다.
- backing field는 직렬화 대상이다.  ( 아래의 코드에서는 name, stars만 직렬화 )
    
    ```kotlin
    @Serializable
    class Project(
        // name is a property with backing field -- serialized
        var name: String
    ) {
        var stars: Int = 0 // property with a backing field -- serialized
    
        val path: String // getter only, no backing field -- not serialized
            get() = "kotlin/$name"
    
        var id by ::name // delegated property -- not serialized
    }
    
    fun main() {
        val data = Project("kotlinx.serialization").apply { stars = 9000 }
        println(Json.encodeToString(data))
    }
    ```
    
- 주생성자가 직렬화 대상이다.
    
    ```kotlin
    @Serializable
    class Project(path: String) {
        val owner: String = path.substringBefore('/')
        val name: String = path.substringAfter('/')
    }
    
    // 주생성자를 private으로 변경하고 부생성자에서 직렬화를 원하는 형태로 변경하며 주생성자를 재호출하도록 변경
    @Serializable
    class Project private constructor(val owner: String, val name: String) {
        constructor(path: String) : this(
            owner = path.substringBefore('/'),
            name = path.substringAfter('/')
        )
    
        val path: String
            get() = "$owner/$name"
    }
    ```
    
- init 블럭으로 검증을 진행할 수 있다.
- 파라미터 기본값을 설정할 수 있다.
- 리플렉션을 사용하지 않기에 어노테이션 추가가 필요하다.
- @Transient 로 직렬화 대상에서 제외시킬 수 있다.
- 컴파일 safe 하다.

### Gson

- 자바 라이브러리
- default value 사용할 수 없다
- 값이 0 또는 null(not null type이여도 null로 변환해버림) 로 직렬화 역직렬화 한다. → 코틀린의 null 타입 안정성을 해치게 된다.

### Moshi

- 자바 라이브러리
- Gson에 비해 가볍다

`kotlinx.serialization`은 코틀린 언어 기반이고 default value를 직렬화, 역직렬화에 활용할 수 있습니다.

그리고 코틀린 언어의 null type 안전성도 보장해주기에 코틀린 언어를 효율적으로 사용할 수 있습니다.

`Gson`, `Moshi`는 자바 기반 라이브러리이고 `Gson`의 경우 default value를 인식하지 못하며 `Moshi`의 경우 `KotlinJsonAdapterFactory`를 추가해야 default value를 인식할 수 있습니다.

값이 없는 경우 반환 받을 타입이 not-null type일지라도 참조형 타입의 경우 null, 기본형 타입의 경우 기본값으로 반환하게 되어 코틀린 언어의 null type 안전성을 해치게 되는 경우가 발생하게 됩니다.

또한, `kotlinx.serialization`과는 달리 리플렉션을 활용하여 직렬화, 역직렬화를 구현했다는 차이점이있습니다.

`Gson`과 `Moshi` 둘을 비교한다면 `Moshi`가 `Gson`에 비해 코틀린 친화적이고 성능과 버전 업데이트 여부에서도 이점이 있다고 할 수 있을 것 같습니다.