# Hero 애니메이션

동일한 이미지를 지닌 두 화면 간 이동 시 애니메이션 구현. 양쪽 스크린의 이미지를 Hero 위젯으로 감싼 후 같은 태그 할당

# 루트 지정 & 이동하기

## MaterialApp에서 화면 별 루트 지정 가능

```dart
MaterialApp(
  title: '',
  initialRoute: '/',
  routes: {
    '/': (context) => const FirstScreen(),
    '/second': (context) => const SecondScreen(),
    ExtractArgumentsScreen.routeName: (context) =>
        const ExtractArgumentsScreen(),

  },
)
.
.
.
class ExtractArgumentsScreen extends StatelessWidget {
  const ExtractArgumentsScreen({super.key});

  static const routeName = '/extractArguments';

  @override
  Widget build(

```

## Navigator.pushNamed()를 통해 지정 루트로 (args와 함께) 이동

```dart
onPressed: () {
  Navigator.pushNamed(context, '/second', arguments);
}
```

## 지정된 루트 없이 특정 화면으로 이동하고 싶을 땐 Navigator.push() & MaterialPageRoute 이용

```dart
onPressed: () {
   Navigator.push(
    context,
    MaterialPageRoute(builder: (context) => const SecondRoute()),
  );
```

# 이전 화면으로 되돌아가기

Drawer 닫을 때와 마찬가지로 Navigator.pop() 사용

```dart
onPressed: () {
            Navigator.pop(context);
          },

```

# 항목 선택 시 상세 화면으로 데이터 전달

1. 전달할 데이터 클래스 정의

```dart
class Todo {
  final String title;
  final String description;

  const Todo(this.title, this.description);
}
```

2. 데이터 배열 생성

```dart
final todos = List.generate(
  20,
  (i) => Todo(
    'Todo $i',
    'A description of what needs to be done for Todo $i',
  ),
);
```

3. 데이터 배열을 ListView로 표시한 후 개별 항목에서 이동 루트(상세화면) 지정

```dart
ListView.builder(
  itemCount: todos.length,
  itemBuilder: (context, index) {
    return ListTile(
      title: Text(todos[index].title),
      onTap: () {
        Navigator.push(
          context,
          MaterialPageRoute(
            builder: (context) => DetailScreen(todo: todos[index]),
          ),
        );
      },
    );
  },
),
```

# 전달 받은 인자 추출 & 표시

1. argument class 정의

```dart
class ScreenArguments {
  final String title;
  final String message;

  ScreenArguments(this.title, this.message);
}
```

2. 위젯 빌드 메소드에서 ModalRoute.of()를 사용해 args 추출 & 표시

```dart
Widget build(BuildContext context) {
    final args = ModalRoute.of(context)!.settings.arguments as ScreenArguments;

    return Scaffold(
      appBar: AppBar(
        title: Text(args.title),
      ),
      body: Center(
        child: Text(args.message),
      ),
    );
  }
```

3. MaterialApp에서 onGenerateRoute을 사용하는 방법도 있음. 그런데 MaterialApp이 복잡해지지 않을까? 모든 화면을 한 눈에 관리하기 위함인가?

# 화면에서 데이터 반환하기

1. 첫 번째 화면에서 두 번째 화면으로 Navigator.push() 후 결과값을 표시하도록 설정

2. 두 번째 화면에서 유저 액션이 끝나면 `Navigator.pop(context, data)` 형태로 첫 번쨰 화면으로 전달

```dart
class FirstScreen extends StatefulWidget {
  const FirstScreen({super.key});

  @override
  State<FirstScreen> createState() => _FirstScreenState();
}

class _FirstScreenState extends State<FirstScreen> {
  @override
  Widget build (BuildContext context) {
    return ElevatedButton(
      onPressed: () {
         _navigateAndDisplaySelection(context);
      },
      child: const Text('Pick an option, any option!')
    )
  }

  Future<void> _navigateAndDisplaySelection(BuildContext context) async {

    final result = await Navigator.push(
      context,
      MaterialPageRoute(builder: (context) => const SecondScreen()),
    );

    if (!mounted) return;

    ScaffoldMessenger.of(context)
      ..removeCurrentSnackBar()
      ..showSnackBar(SnackBar(content: Text('$result')));
  }
}

```

```dart
class SecondScreen extends StatelessWidget {
  const SecondScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Pick an option'),
      ),
      body: Center(
        .
        .
        .
            Padding(
              padding: const EdgeInsets.all(8),
              child: ElevatedButton(
                onPressed: () {
                  Navigator.pop(context, 'Yep!');
                },
                child: const Text('Yep!'),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8),
              child: ElevatedButton(
                onPressed: () {
                  Navigator.pop(context, 'Nope.');
                },
                child: const Text('Nope.'),
              ),

```

# 안드로이드/iOS 앱 링크 만들기

## 안드로이드

https://docs.flutter.dev/cookbook/navigation/set-up-app-links

## iOS

https://docs.flutter.dev/cookbook/navigation/set-up-universal-links
