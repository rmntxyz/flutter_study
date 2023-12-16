# Tabs

DefaultTabController를 쓰는 방법이 제일 간편

```dart
   return MaterialApp(
      home: DefaultTabController(
        length: 3,
        child: Scaffold(
          appBar: AppBar(
            bottom: const TabBar(
              tabs: [
                Tab(icon: Icon(Icons.directions_car)),
                Tab(icon: Icon(Icons.directions_transit)),
                Tab(icon: Icon(Icons.directions_bike)),
              ],
            ),
            title: const Text('Tabs Demo'),
          ),
          body: const TabBarView(
            children: [
              Icon(Icons.directions_car),
              Icon(Icons.directions_transit),
              Icon(Icons.directions_bike),
            ],
          ),
        ),
      ),
    );
```

# Drawer

Tab에 너무 많을 때 drawer가 유용한 대안

1. Scaffold 안에 추가

```dart
Scaffold(
  drawer: Drawer(
    child: // Populate the Drawer in the next step.
  ),
);
```

2. ListView(항목이 많을 경우 Column보다 유용)에 DrawerHeader와 ListTile 지정

```dart
Drawer(
  child: ListView(
    children: [
      const DrawerHeader(
        decoration: BoxDecoration(
          color: Colors.blue,
        ),
        child: Text('Drawer Header'),
      ),
      ListTile(
        title: const Text('Item 1'),
        onTap: () {
          // ...
        },
      ),
      ListTile(
        title: const Text('Item 2'),


```

3. drawer는 열리면 navigation stack에 추가되므로 사용자의 메뉴 선택 후 drawer가 닫히도록 navigation stack에서 drawer 제거

```dart
ListTile(
  title: const Text('Item 1'),
  onTap: () {
    // Update the state of the app
    // ...
    // Then close the drawer
    Navigator.pop(context);
  },
),
```

# snackbar

toast와 유사.

1. ScaffoldMessenger를 이용해 이벤트(예제에서는 "show snackbar" 버튼 클릭) 발생 시 snackbar 표시

```dart
 ElevatedButton(
        onPressed: () {
          final snackBar = SnackBar(
            content: const Text('Yay! A SnackBar!'),
          );
          ScaffoldMessenger.of(context).showSnackBar(snackBar);
        },
        child: const Text('Show SnackBar'),
      ),

```

2. snack bar에 기능 추가

```dart
final snackBar = SnackBar(
  content: const Text('Yay! A SnackBar!'),
  action: SnackBarAction(
    label: 'Undo',
    onPressed: () {
      // Some code to undo the change.
    },
  ),
);
```

# 외부 폰트 사용하기

1. `flutter pub add google_fonts`로 구글 폰트를 설치하거나 개별 폰트 파일을 플러터 앱 root에 fonts 폴더를 만들어 저장
2. `pubspec.yaml`에서 폰트 선언

```dart
flutter:
  fonts:
    - family: Raleway
      fonts:
        - asset: fonts/Raleway-Regular.ttf
        - asset: fonts/Raleway-Italic.ttf
          style: italic
    - family: RobotoMono
      fonts:
        - asset: fonts/RobotoMono-Regular.ttf
        - asset: fonts/RobotoMono-Bold.ttf
          weight: 700
```

3. 개별 위젯에서 사용 시

```dart
 import 'package:google_fonts/google_fonts.dart'
 ...
 child: Text(
  'Roboto Mono sample',
  style: TextStyle(fontFamily: 'RobotoMono'),
)
```

4. 디폴트로 사용 시

```dart
return MaterialApp(
  title: 'Custom Fonts',
  // Set Raleway as the default app font.
  theme: ThemeData(fontFamily: 'Raleway'),
  home: const MyHomePage(),
);
```

# OrientationBuilder

휴대폰 방향에 따라 GridView의 column 수를 다르게 설정 가능

```dart
body: OrientationBuilder(
  builder: (context, orientation) {
    return GridView.count(
      // Create a grid with 2 columns in portrait mode,
      // or 3 columns in landscape mode.
      crossAxisCount: orientation == Orientation.portrait ? 2 : 3,
    );
  },
),
```

main()에서 `SystemChrome.setPreferredOrientations()`을 통해 화면 방향(portraitUp, landscapeLeft, portraitDown, landscapeRight) 고정 가능

```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized();
  SystemChrome.setPreferredOrientations([
    DeviceOrientation.portraitUp,
  ]);
  runApp(const MyApp());
}
```

# Themes

## 앱 전역을 위한 테마

ThemeData 사용

```dart
MaterialApp(
  title: appName,
  theme: ThemeData(
    useMaterial3: true,

    // Define the default brightness and colors.
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.purple,
      // ···
      brightness: Brightness.dark,
    ),

    // Define the default `TextTheme`. Use this to specify the default
    // text styling for headlines, titles, bodies of text, and more.
    textTheme: TextTheme(
      displayLarge: const TextStyle(
        fontSize: 72,
        fontWeight: FontWeight.bold,
      ),
      // ···
      titleLarge: GoogleFonts.oswald(
        fontSize: 30,
        fontStyle: FontStyle.italic,
      ),
      bodyMedium: GoogleFonts.merriweather(),
      displaySmall: GoogleFonts.pacifico(),
    ),
  ),
  home: const MyHomePage(
    title: appName,
  ),
);
```

## 상위 컴포넌트의 테마 적용

Theme.of(context)

```dart
Widget(
  color: Theme.of(context).colorScheme.primary,
  child: Text(
```

## 상위 컴포넌트의 테마 확장

Theme.of(context).copyWith

```dart
Widget(
  data: Theme.of(context).copyWith(
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.pink,
    ),
  ),
  child: Text(
```

## 새로운 테마 설정

ThemeData 사용

```dart
Widget(
  data: ThemeData(
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.pink,
    ),
  ),
  child: Text(
```
