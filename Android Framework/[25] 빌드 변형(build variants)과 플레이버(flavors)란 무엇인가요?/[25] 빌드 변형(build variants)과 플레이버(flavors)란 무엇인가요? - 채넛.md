### 빌드 변형

- 빌드 타입과 플레이버 조합으로 만들어지는 결과물
    - freeDebug, freeRelease, paidDebug, paidRelease, …
- 빌드 타입
    - 애플리케이션이 어떻게 빌드되는지를 나타낸다.
    - 기본적인 빌드 타입
        - debug : 개발 중에 사용되는 빌드
            - 종종 디버그 도구, 로그 및 테스트용 디버그 툴을 활용하여 개발자가 개발 향상성을 높이도록 한다.
        - release : 배포에 최적화된 구성
            - 리소스 최적화 및 최소화, 난독화가 적용되고 스토어 게시를 위해 별도의 릴리스 키로 서명되어야 한다.
    - 커스텀 빌드 타입을 추가할 수 있다.

### 플레이버

- 앱의 서로 다른 변형을 정의하기 위한 설정 단위
    - 무료 버전과 유료버전 / 국가별 버전 등을 만들고 싶을 때 사용
- 각 flavor는 독립적인 소스코드, 자원, 매니페스트 등을 가질 수 있어 기능 차별화, API 엔드포인트 변경, 광고 설정 유무 등에 활용할 수 있다.

```kotlin
android {
	...
	productFlavors {
			free {
				applicationId "com.example.app.free"
				versionName "1.0-free"
			}
			paid {
				applicationId "com.example.app.paid"
				versionName "1.0-paid"
	}
}
```