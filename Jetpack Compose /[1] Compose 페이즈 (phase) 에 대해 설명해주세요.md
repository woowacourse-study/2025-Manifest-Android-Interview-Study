Compose의 렌더링 파이프라인은 다음과 같이 3단계로 이루어져 있다.
  - Composition
  - Layout
  - Drawing

# Composition

- `@Composable` 메소드를 통해 UI 트리를 생성한다.
- 슬롯 테이블을 이용하여 UI 구조를 초기화하고 컴포저블 간의 관계를 정의한다.
- 상태가 변경될 경우 리컴포지션을 진행한다.

# Layout

- Composition 이후에 실행된다.
- 각 UI 요소의 크기나 위치를 결정한다.
- 자식 컴포저블을 측정하고, 측정값을 기반으로 크기를 결정한 뒤 위치를 결정한다.

# Drawing

- UI 요소들이 화면에 렌더링된다.
- 이 과정에서 Skia 엔진을 사용한다. (하드웨어 가속)
- `Canvas` API를 이용해 드로잉 로직을 커스텀할 수 있다.

[안드로이드 공식 문서 - Jetpack Compose 단계](https://developer.android.com/develop/ui/compose/phases?hl=ko)
