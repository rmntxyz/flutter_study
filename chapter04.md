# StatefulWidgets

react의 setState와 완전 유사하지만  
차이점이라 느껴지는건 flutter의 `setState`의 목적 자체는 re-rendering에 있다는 점. (`build` 메소드 재실행)  
물론 react에서의 `setState` 또한 re-rendering을 발생시키지만  
그것에 더해 실제 데이터가 바뀌었을때 연관된 컴포넌트만 다시 그리려는 복잡한 계산이 같이 들어간다.  
flutter에서의 `setState`는 그냥 위젯 자체를 다시 그리는, 러프한 느낌의 트리거

많이 사용되지 않는다.

이벤트 처리 하나 보고 넘어가자  
`IconButton(onPressed: ...)`

# BuildContext

위젯 트리에 대한 모든 정보가 들어 있다.  
부모 요소에 접근할 수 있게 해준다.

이것 또한 react의 context기능과 매우 유사.
이 번 챕터에서 보여준 것은 테마의 참조 뿐이었지만  
react처럼 함수를 넘겨 준다거나 다양한 데이터를 context화 해서  
멀리 떨어진 계층의 위젯끼리 소통이 가능할 것이라 예상 된다.  
이것은 코드의 복잡성을 줄여주고 중복 코드를 최소화 시켜줄 것

한 가지 또 보고 넘어갈 만한 것
Non-Null assertion operator 인 `!`(punctuation)

```dart
TextStyle{
  ...
  color: Theme.of(context).textTheme.titleLarge!.color
}
```

# Widget Lifecycle

StatefulWidget은 살아있다 = 이벤트에 반응한다

## `initState` method

- 위젯 생성될 때 딱 한 번 호출
- `build` 호출 전에 실행 됨
- **외부 API를 호출하려면 여기** 에서

```dart
@override
void initState() {
  super.initState();
  print('initState!');
}
```

## `dispose` method

- 화면에서 사라질때 호출됨
  - 위젯 트리에서 제거될 때
- 위젯 내부에서 발생한 무언가를 정리할 때 사용
- 다른 언어에서 부르는 소멸자의 역할

```dart
@override
void dispose() {
  super.dispose();
  print('dispose!');
}
```
