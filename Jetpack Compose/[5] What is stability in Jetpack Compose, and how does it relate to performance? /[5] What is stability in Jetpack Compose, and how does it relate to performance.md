Compose Compiler는 불필요한 Recomposition을 줄여 효과적인 UI 갱신을 위해 Composable 함수의 파라미터를 **Stable**과 **Unstable** 두 가지로 나눈다.

# Stable vs Unstable

## Stable

- 원시 타입 : `String` 포함, 예상치 못한 변화가 없으므로 안정적
- 함수 : 예측 가능한 동작으로 인해 안정적 (불안정한 값을 캡쳐하는 경우 해당 함수 또한 불안정하다고 간주)
- 클래스 : 불변 속성을 가지고 있는 데이터 클래스는 안정적, `@Stable` 혹은 `@Immutable` 어노테이션이 추가된 클래스 또한 안정적인 것으로 간주
    
    ```kotlin
    data class User(
    	val id: Int, // 불변 원시 타입
    	val name: String,
    )
    ```
    

## Unstable

- 인터페이스 혹은 추상 클래스 : `List`, `Map`, `Any`와 같은 타입은 컴파일 타임에 구현을 보장할 수 없기 때문에 불안정하다고 간주됨
- 가변 프로퍼티를 포함한 클래스 : 최소 하나의 가변 프로퍼티 혹은 불안정한 타입을 포함한 데이터 클래스
    
    ```kotlin
    data class MutableUser(
    	val id: Int,
    	var name: String, // 가변
    )
    ```
    

# Smart Recomposition

위와 같은 타입 추론은 결국 Compose Compiler가 Compose Runtime을 위해 하는 일

안정성이 결정되면 Compose Runtime이 **Smart Recomposition**을 실행

→ Compiler가 제공한 정보를 활용하여 불필요한 Recomposition을 건너뜀

- 안정성에 따른 결정 : 매개변수가 안정적이며 그 값이 변경되지 않은 경우 (`equals()`가 `true`), 해당 UI 컴포넌트의 Recomposition을 건너뜀. 매개변수가 불안정하거나, 안정적이지만 값이 변경된 경우 (`equals()`가 `false`) Compose Runtime은 Recomposition을 시작하여 해당 UI를 무효화하고 다시 그림.
- 동등성 검사 : `equals()` 를 통한 비교는 해당 타입이 안정적이라고 간주될 때만 수행하며, Composable 함수의 입력값이 변경될 때마다 해당 타입의 `equals()`를 사용하여 이전 값과 비교함

# Composable 함수 추론

Compose Compiler는 Composable 함수를 여러 항목으로 분류하여 실행을 최적화한다. 

- Restartable : Compose Compiler에 의해 추론된 Composable 함수의 타입으로, Recomposition 과정의 기반을 제공하는 함수. 입력값 혹은 상태가 변경되면 Compose Runtime이 UI 갱신을 위해 해당 함수를 실행한다. 대부분의 Composable 함수는 기본적으로 Restartable로 취급된다.
- Skippable : Smart Recomposition에 의해 설정된 특정 조건 하에 Recomposition을 완전히 건너뛸 수 있다.
