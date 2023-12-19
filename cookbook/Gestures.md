# Pointers

- `PointerDownEvent`
- `PointerMoveEvent`
- `PointerUpEvent`
- `PointerCancelEvent`

그리고 이 이벤트들을 핸들링 하기 위해 `Listener` 위젯을 사용한다.

사용방법은 [링크](https://api.flutter.dev/flutter/widgets/Listener-class.html)의 예제를 살펴보자

그런데 보통 gesture event를 많이 쓴다.

# Gestures

**Tab**

- `onTabDown`
- `onTabUp`
- `onTap`
- `onTapCancel`

**Double tap**

- `onDoubleTap`

**Long press**

- `onLongPress`

**Vertical drag**

- `onVerticalDragStart`
- `onVerticalDragUpdate`
- `onVerticalDragEnd`

**Horizontal drag**

- `onHorizontalDragStart`
- `onHorizontalDragUpdate`
- `onHorizontalDragEnd`

**Pan**

- `onPanStart`  
  `onHorizontalDragStart`나 `onVerticalDragStart`이 설정되어 있으면 콜백이 충돌한다.
- `onPanUpdate`  
  `onHorizontalDragUpdate`나 `onVerticalDragUpdate`이 설정되어 있으면 콜백이 충돌한다.
- `onPanEnd`  
  `onHorizontalDragEnd`나 `onVerticalDragEnd`이 설정되어 있으면 콜백이 충돌한다.

## Gesture disambiguation

제스쳐가 등록된 위젯이 중첩되어 쌓여있거나 혹은 하나의 `GestureDetector`가 다수의 제스쳐를 다뤄야 할 경우 어떤 제스쳐를 인식할지 결정해야 한다.

제스쳐 인식기(recognizer)가 여러개 있을 경우, 각 인식기를 제스쳐 아레나에 참여시켜 사용자가 의도하는 제스쳐를 명확하게 구분하도록 한다.
그 규칙은 아래와 같다.

- 인식기는 자신을 제거하여 아레나에서 떠난다. 만약 하나의 인식기만 아레나에 남는다면 해당 인식기가 승리
- 인식기는 스스로를 승리자로 선언한다. 나머지 모든 인식기들은 패배로 간주

예를들어 수직과 수평 드래그를 구분해야 하는 상황을 가정해 보자.

1. 유저가 point down 이벤트를 발생시켰다.
2. 두 제스쳐 이벤트의 인식기는 아레나에 진입
3. 유저의 행동을 살핀다.
4. 유저가 탭한 상태로 임계값 이상 수평 방향 이동
5. 수평 드래그 인식기가 자신을 승리로 선언한다.
6. 수평 드래그 제스쳐 이벤트가 발생

이건 `onTab`, `onDoubleTab` 사이에서도 마찬가지.  
둘 다 리스너를 등록하면 탭 이벤트가 더블 탭으로 인식하는 클릭 사이의 텀 만큼 나중에 콜백을 실행한다.  
 혹시라도 임계시간 안에 또 한 번의 탭 이벤트가 발생하면 더블클릭의 승리로 처리하기 위한 딜레이 시간이 작용하기 때문이다.  
 만약 더블 탭 이벤트가 없다면 이 딜레이 시간은 사라지고 바로 탭 이벤트가 호출된다.

# Handle taps

`GestureDetector` 위젯을 사용하면 된다.  
[예제](https://docs.flutter.dev/cookbook/gestures/handling-taps) 보면 사용방법을 알 수 있는데,

그런데 그냥 버튼의 `onPressed` 구현하면 안되나...

# Implement swipe to dismiss

스와이프로 리스트 항목 중 하나를 제거해버리는 제스쳐를 구현하는 방법에 대해 다룬다.

`Dismissible` 위젯이 또 있는데, (없는게 없네)  
이 위젯의 `onDismissed`를 구현하면 아이템이 사라졌을때의 동작을 기술할 수 있다.

`ListView` - `ListTile` 구조의 일반적인 리스트 출력 구조를
`ListView` - `Dissmissible(ListTile)` 로 감싸서 만들면 된다.

백그라운드의 색을 넣어서 (빨강) 이 리스트들이 dissmissible 하다는 것을 유저가 인식할 수 있도록 해 줘야 한다.

# Add Material touch ripples

탭했을때 리플 애니메이션이 발생하는 버튼을 만들 수 있다.  
`InkWell`이라는 위젯을 사용하면 되는데, 굳이?

```dart
class MyButton extends StatelessWidget {
  const MyButton({super.key});

  @override
  Widget build(BuildContext context) {
    // The InkWell wraps the custom flat button widget.
    return InkWell(
      // When the user taps the button, show a snackbar.
      onTap: () {
        ScaffoldMessenger.of(context).showSnackBar(const SnackBar(
          content: Text('Tap'),
        ));
      },
      child: const Padding(
        padding: EdgeInsets.all(12),
        child: Text('Flat Button'),
      ),
    );
  }
}
```
