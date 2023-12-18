# 페이지 이동 애니메이션

## PageRouteBuilder

이동할 루트의 컨텐츠를 빌드하는 pageBuilder, 경로 전환 애니메이션을 빌드하는 transitionBuilder를 콜백으로 사용.

첫 번째 화면에서 PageRouteBuilder를 이용해 경로 전환을 할 수 있도록 설정. 지금까지 사용한 `Navigator.push(context, 경로)` 대신에 `Navigator.of(context).push(PageRouteBuilder를 반환하는 함수)` 사용.

```dart
class Page1 extends StatelessWidget {
  const Page1({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            Navigator.of(context).push(_createRoute());
          },
          child: const Text('Go!'),
        ),
      ),
    );
  }
}

Route _createRoute() {
  return PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => const Page2(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      return child;
    },
  );
}
```

## Tween

시작값에서 종료값으로 전환하는 클래스(Tween<T extends Object?> class, [링크](https://api.flutter.dev/flutter/animation/Tween-class.html))

## transitionBuilder의 animation 파라미터

Animation<double>[링크](https://docs.flutter.dev/ui/animations/tutorial#animationdouble). 0과 1 사이의 값을 생성.

## SlideTransition

플러터의 AnimationWidget에 속하는 클래스.

```dart
transitionsBuilder: (context, animation, secondaryAnimation, child) {
  const begin = Offset(0.0, 1.0);
  const end = Offset.zero;
  final tween = Tween(begin: begin, end: end);
  final offsetAnimation = animation.drive(tween);

  return SlideTransition(
    position: offsetAnimation,
    child: child,
  );
},
```

## CurveTween

Curve 클래스([링크](https://api.flutter.dev/flutter/animation/Curves-class.html)) 이용

```dart
var curve = Curves.ease;
var curveTween = CurveTween(curve: curve);
```

## chain()

두 가지 tween을 결합

```dart
const begin = Offset(0.0, 1.0);
const end = Offset.zero;
const curve = Curves.ease;

var tween = Tween(begin: begin, end: end).chain(CurveTween(curve: curve));

return SlideTransition(
  position: animation.drive(tween),
  child: child,
);
```

# 객체의 속성을 애니메이션화하기

## Animated Container

Container 위젯과 유사하지만 주어진 값을 이용해 자동으로 애니메이션. width, height, decoration에 변수를 입력하고, duration과 curve 지정 가능

```dart
double _width = 50;
double _height = 50;
Color _color = Colors.green;
.
.
.
@override
Widget build(BuildContext context) {
    .
    .
    .
        body: Center(
          child: AnimatedContainer(
            width: _width,
            height: _height,
            decoration: BoxDecoration(
              color: _color,
              borderRadius: _borderRadius,
            ),
            duration: const Duration(seconds: 1),
            curve: Curves.fastOutSlowIn,
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () {
            setState(() {
              final random = Random();

              _width = random.nextInt(300).toDouble();
              _height = random.nextInt(300).toDouble();

```

# 페이드 인 앤 아웃

## AnimatedOpacity
duration과 복수의 opacity를 지정하여 페이드 인/아웃 가능

```dart
body: Center(
        child: AnimatedOpacity(
          opacity: _visible ? 1.0 : 0.0,
          duration: const Duration(milliseconds: 500),
          child: Container(
            width: 200,
            height: 200,
            color: Colors.green,
          ),

```
