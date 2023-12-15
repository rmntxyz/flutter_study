# UI

`lib`-`screens` 디렉토리 안에 `home_screen.dart` 파일 하나를 만들고  
그 안에서 자동 완성을 사용해서 `StatefulWidget`을 만들자

그럼 아래와 같이 자동 완성이 된다.

```dart
import 'package:flutter/material.dart';

class MyWidget extends StatefulWidget {
  const MyWidget({super.key});

  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  @override
  Widget build(BuildContext context) {
    return const Placeholder();
  }
}
```

이것이 의미하는 바는

## Flexible

하나의 박스가 얼마나 공간을 차지할지 정할 수 있음. CSS의 flex와 거의 동일

## ThemeData

- `backgroundColor`: _deprecated_
  - use `colorScheme` instead

## Use of Context

```dart
style: TextStyle(
  color: Theme.of(context).cardColor,
  fontSize: 89,
  fontWeight: FontWeight.w600,
),
```

## Center

수평, 수직 중앙

```dart
child: Center(
  child: IconButton(
    iconSize: 120,
    color: Theme.of(context).cardColor,
    onPressed: onStartPressed,
    icon: const Icon(Icons.play_circle_outline),
  ),
),
```

## Expanded

부모의 배열 방향으로 크기를 확장한다.

## MainAxisAlignment.center

`Column` 부모 안에서 사용하면 수직 중앙으로 정렬된다.

# Timer

## `late` modifire

나중에 초기화 할 거라고 하고 일단 선언 먼저 해야 할 때 사용

start / pause 할 수 있는 기능을 가진 위젯은 아래와 같음

여기에서 이벤트 핸들러와 이벤트를 위젯에 연결시키는 방법, 그리고 기본적인 Timer 사용법을 살펴볼 수 있다.
`setState()`를 호출하는 위치도 확인

```dart
class _HomeScreenState extends State<HomeScreen> {
  int totalSeconds = 1500;
  bool isRunning = false;
  late Timer timer;

  void onTick(Timer timer) {
    setState(() {
      totalSeconds = totalSeconds - 1;
    });
  }

  void onStartPressed() {
    timer = Timer.periodic(
      const Duration(seconds: 1),
      onTick,
    );
    setState(() {
      isRunning = true;
    });
  }

  void onPausePressed() {
    timer.cancel();
    setState(() {
      isRunning = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Theme.of(context).backgroundColor,
      body: Column(
        children: [
          Flexible(
            flex: 1,
            child: Container(
              alignment: Alignment.bottomCenter,
              child: Text(
                '$totalSeconds',
                style: TextStyle(
                  color: Theme.of(context).cardColor,
                  fontSize: 89,
                  fontWeight: FontWeight.w600,
                ),
              ),
            ),
          ),
          Flexible(
            flex: 3,
            child: Center(
              child: IconButton(
                iconSize: 120,
                color: Theme.of(context).cardColor,
                onPressed: isRunning ? onPausePressed : onStartPressed,
                icon: Icon(isRunning
                    ? Icons.pause_circle_outline
                    : Icons.play_circle_outline),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

## Data Format

초 단위 넣으면 분 단위로 포멧팅 해주는 `Duration` 클레스를 사용한다.  
그 리턴값(string)을 잘라서 사용한다.

```dart
String format(int seconds) {
  var duration = Duration(seconds: seconds);
  return duration.toString().split(".").first.substring(2, 7);
}
```
