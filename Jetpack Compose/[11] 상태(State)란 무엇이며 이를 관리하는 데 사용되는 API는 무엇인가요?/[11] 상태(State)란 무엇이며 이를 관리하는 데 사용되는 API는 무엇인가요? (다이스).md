# [11] 상태(State)란 무엇이며 이를 관리하는 데 사용되는 API는 무엇인가요?

카테고리: Compose Runtime

## **Question**

```
상태(State)란 무엇이며 이를 관리하는 데 사용되는 API는 무엇인가요?
```

## **Answer**

```
상태(State)란 UI에서 동적으로 변하는 데이터를 의미하며, Compose에서는 상태가 변경되면 이를 감지하여 관련 UI를 자동으로 업데이트(Recomposition)합니다.

Compose에서 상태를 관리하기 위해 주로 사용하는 API는 다음과 같습니다
	•	mutableStateOf : 변할 수 있는 상태를 선언하고 관찰 가능하게 만듭니다.
	•	remember : Composable 함수 내에서 상태를 메모리에 유지하여 Recomposition 시 초기화되지 않도록 합니다.
	•	rememberSaveable : remember와 유사하지만, 프로세스 종료나 구성 변경(예: 화면 회전) 시에도 상태를 자동으로 복원합니다.

이 API들을 활용하면, UI 상태를 안전하게 관리하면서 Recomposition이 발생해도 필요한 데이터가 유지되도록 할 수 있습니다.
```
