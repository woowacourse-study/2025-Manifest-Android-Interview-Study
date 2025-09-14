### PPT 정리 전 자료조사

### Recomposition 이란 ?

- 이미 렌더링된 UI 레이아웃을 업데이트하기 위해 세 가지 주요 과정를 통해 상태 변경이 발생할 때마다 UI를 다시 그리는 매커니즘 사용하는데, 이를 Recomposition 이라고 한다.
- 여기에서 세 가지 주요 과정란 ?
    - Compostion -> Layout -> Drawing
        - Composition : 컴포저블 함수를 실행하고 UI 정보를 분석
        - Layout : UI 정보를 측정하고, 레이아웃 트리의 각 노드에 UI 요소를 배치하는 방법을 결정
        - Drawing : UI 요소를 Canvas에 그리기
    - 대부분의 Composable 함수는 항상 같은 순서로 이뤄진다.
    - 여기에서 세 단계를 거치지 않는 경우는 언제일까 ?
        1. Composition만 발생하고 Layout/Drawing은 발생하지 않는 경우
            - 상태 변화로 인해 구조만 변했지만 UI의 배치나 그리기는 변하지 않는 경우
            - ```kotlin
			  var count by remember { mutableStateOf(0) }
			  
			  Text("Count: $count")
			  ```
            - count 값이 변경되면 Text의 문자열은 바뀌지만, 글꼴 크기,스타일이 동일하면 Layout은 그대로 재사용하고, Draw만 다시하거나 캐싱된 값으로 최적화하기도 한다.
        2. . Layout, Drawing은 발생하고 Composition은 생략되는 경우
            - Composable 트리 자체에는 변화가 없지만, 부모의 크기 제약 조건이 달라져서 Layout만 다시 하는 경우
              , 부모 Box의 크기 변경, ...

    3. Drawing만 발생하고 Composition/Layout은 생략되는 경우
        - 상태변화로 인해 실제로는 픽셀만 달라지는 경우
            - Canvas 내부에서 Color 값만 바뀌는 경우

        4. 세 단계 모두 스킵되는 경우
            - remember로 메모이제이션 된 값이 변하지 않았고, 관련된 상태 변화가 발생하지 않았다면, 아무 단계도 일어나지 않는다.

- 화면전환은 Composition 트리가 다 날아가고 새롭게 만들어지므로 Composition -> Layout -> Drawing 과정을 거치게 된다.

- 전체 UI 트리와 Element 를 재구성하는 것은 비용이 많이 든다. -> Compose Runtime은 최적화 시스템으로 람다나 변경사항이 없는 항목들을 skip해서 최적화한다. -> 예시로 리사이클러뷰의
  아이템 하나를 업데이트한다고 하면 전체를 업데이트하는 것이 아닌 단일 항목만 업데이트하는 것과 같다.

- 최적화하기 위해 Jetpack Compose는 변경되었는 지 추론할 수 있어야 한다.
    - Unstable 타입 : 변경 가능하며 변경 시, Composition에 알리지 않는 데이터를 보유한다. - Compose는 변경되었음을 검증할 수 없다.
    - Stable 타입 : 변경 가능하지만, 변경 시 Composition에 알린다. - 상태에 대한 변경이 있을 때마다 항상 알려지기에 안정적이다. - skip 가능하도록 표시 가능
    - Immutable 타입 : 변경 불가능한 데이터, 데이터가 절대 변경되지 않는다 - Compose는 안정적인 데이터로 처리할 수 있다. - skip 가능하도록 표시 가능

- holding class들은 내부 속성으로 var를 사용하지 말기
    - 변경 가능하긴 하지만, composition을 알릴 수 없어, 컴포저블을 unstable 하게 만든다.
  ```kotlin
  // DO 
  data class InherentlyStableClass(val text: String) 
  // DO NOT
  data class InherentlyUnstableClass(var text: String)
  ```
- private 프로퍼티이더라도 var 이면 unstable 로 판단한다.
- Compose는 클래스, 인터페이스, 객체에 대한 안정성만을 추론할 수 있다. 외부의 모듈 등, 외부 클래스는 unstable로 판단한다.
    - 동일한 속성을 사용한다고 하더라도 flatten하게 모든 속성을 우리만의 클래스로 옮겨야 stable 하게된다.
- Collections 에서 불변성을 기대하지 말기
    - 대안 : 코틀린의 불변 컬렉션 사용하기
    - 다른 해결책 : 컬렉션을 감싸고 있는 Wrapper 클래스를 생성하고 @Immutable 로 표시하기
    - 그러나 둘 다 이상적인 해결책은 아니다.
- Flow는 unstable 하다.
    - 관찰 가능하다고 보여지더라도, Flow는 새 값을 방출할 때 composition에 알리지 않아 본질적으로 unstable 하다. - 필요시에만 사용해야 한다.
- inline된 컴포저블은 restartable 하거나 skippable 할 수 없다.
    - `Column, Row, Box` 같은 일반적인 Composable들은 모두 inline이다.

### Hoist state

- 무상태 컴포저블을 만들기 위한 행위
- 모든 필요한 상태는 컴포저블의 호출자로부터 전달되어야 한다.
- 모든 이벤트는 상태의 소스로 위로 흐르도록 해야 한다.

너무 높은 범위에서 읽지 말기

### Recomposition이 발생하는 조건

1. 매개변수에 변경이 발생했을 때
    - 컴포저블 함수는 입력 매개변수가 변경될 때 recomposition을 트리거한다.
    - Compose 런타임은 equals() 함수를 사용해서 새 매개변수와 이전 매개변수 값을 비교한다.
        - 만약 false 일 경우, 런타임은 변경 사항을 인지하고 recomposition을 트리거하여 동기화가 필요한 부분에 한해서만 UI를 업데이트한다.
2. 상태 변경이 관찰되었을 때

- Jetpack Compose는 remember 함수와 State API를 함께 사용하여 상태 변경을 모니터링 한다.
- 상태 객체를 메모리에 보존하고 recomposition이 발생했을 때 메모리에 저장된 값을 복원하여 UI에 최신 상태를 일관되게 반영하도록 보장한다.

### 참고

[Layout Inspector 시작하기](https://developer.android.com/studio/debug/layout-inspector)

[Layout Inspector 설명](https://developer.android.com/develop/ui/compose/tooling/debug)

[Compose phases](https://developer.android.com/develop/ui/compose/phases)

[Recomposition 최적화](https://proandroiddev.com/optimize-app-performance-by-mastering-stability-in-jetpack-compose-69f40a8c785d)

[Recomposition 최적화 가이드라인](https://proandroiddev.com/6-jetpack-compose-guidelines-to-optimize-your-app-performance-be18533721f9)
