### Bundle

- Activity, Fragment, Service 같은 컴포넌트 간 데이털르 전달하는 데 사용되는 키-값 쌍 데이터 구조
- 안드로이드 운영체제가 쉽게 관리하고 전송할 수 있는 형식으로 데이터를 직렬화하도록 설계
- 사용 예시
    - Activity 간 데이터 전달 : Intent에 Bundle을 담아서 대상 Activity에 데이터 전달
    - Fragment 간 데이터 전달 : setArguments(), getArguments() 함수로 Fragment 간 데이터 전달
    - 인스턴스 상태 저장 및 복원 : onSaveInstnaceState(), onRestoreInstanceState() 와 같은 생명주기 메서드에서 구성 변경 중에 임시 UI 상태를 저장하고 복원하는 데 사용
    - Service에 데이터 전달 : Service 시작하거나, 바인딩된 Service에 데이터를 전달할 때 Bundle을 통해 데이터 운반
- UI 상태를 `임시` 저장하고 복원하는 데 적합한 경량 key-value 저장소 역할을 할 수 있다
    - 크기가 큰 데이터 이거나 비동기 처리가 필요한 상태 데이터의 경우 적합하지 않다.

Q. 구성 변경 중 onSaveInstanceState()는 UI 상태를 보존하기 위해 Bundle을 어떻게 활용하며, Bundle에 어떤 유형의 데이터를 담을 수 있나요?

- 상태 관리가 필요한 데이터의 경우 onSaveInstanceState() 생명주기에서 key-value 쌍으로 Bundle 객체에 저장한다.
- 저장된 데이터는 onCreate(), onRestoreInstanceState() 생명주기에 Bundle 객체를 통해 복원할 수 있다.
- Bundle 에는 primitive type, string, Parcelable, Serializable, 배열 및 ArrayList, Bundle, …과 같은 데이터 유형을 담을 수 있다.