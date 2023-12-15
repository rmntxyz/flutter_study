# Add the http package.

```sh
$ flutter pub add http
```

```dart
import 'package:http/http.dart' as http;
```

## Android

AndroidManifest.xml 파일의 `<application />` 전에 넣는다.

```xml
<!-- Required to fetch data from the internet. -->
<uses-permission android:name="android.permission.INTERNET" />
```

## iOS

- for debug: `macos/Runner/DebugProfile.entitlements`
- for release: `macos/Runner/Release.entitlements`

```xml
<!-- Required to fetch data from the internet. -->
<key>com.apple.security.network.client</key>
<true/>
```

# Make a request

```dart
Future<http.Response> fetchAlbum() {
  return http.get(Uri.parse('https://DOMAIN/PATH'));
}
```

`Future`는 JS의 `Promise`와 같다.

```dart
Future<int> asyncValue = Future(foo);
asyncValue.then((int value) {
  return bar(value);
}).catchError((e) {
  return 499;
});
```

비동기 작업의 이해가 필요.

# Convert the response into a custom Dart object.

```dart
// 함수를 async로 선언
Future<Album> fetchAlbum() async {
  // final => 한 번 할당하면 재할당 안 됨
  final response = await http
      .get(Uri.parse('https://jsonplaceholder.typicode.com/albums/1'));

  if (response.statusCode == 200) {
    // JSON parsing
    return Album.fromJson(jsonDecode(response.body) as Map<String, dynamic>);
  } else {
    throw Exception('Failed to load album');
  }
}
```

여기서 Album 클래스에서 파싱하는 메소드 `Album.fromJson` 을 보면,

```dart
class Album {
  final int userId;
  final int id;
  final String title;

  const Album({
    required this.userId,
    required this.id,
    required this.title,
  });

  factory Album.fromJson(Map<String, dynamic> json) {
    return switch (json) {
      {
        'userId': int userId,
        'id': int id,
        'title': String title,
      } =>
        Album(
          userId: userId,
          id: id,
          title: title,
        ),
      _ => throw const FormatException('Failed to load album.'),
    };
  }
}
```

`factory` 키워드를 사용하게 된다.  
`switch` 문이 나온 이유는 팩토리 메소드 내에서 다양한 클래스의 인스턴스를 만들어 낼 수 있기 때문인데 여기서는 오로지 하나의 경우만이 존재하므로 `case` 키워드는 보이지 않는다.

`switch`문 안쪽의 문구가 특이한데, 이걸 보고 이해하자.

```dart
void main() {
  var dayOfWeek = 'Monday';
  var dayNumber = switch (dayOfWeek) {
    'Monday' => 1,
    'Tuesday' => 2,
    'Wednesday' => 3,
    'Thursday' => 4,
    'Friday' => 5,
    'Saturday' => 6,
    'Sunday' => 7,
    _ => 10, //Default value
  };
  print(dayNumber);
}
```

Arrow Function이 사용된다는 점과 `_`을 `switch-case`문의 `default`로 사용한다는 점이 신기하다.

## serialize

````dart
class User {
  final String name;
  final String email;

  User(this.name, this.email);

  User.fromJson(Map<String, dynamic> json)
      : name = json['name'] as String,
        email = json['email'] as String;

  Map<String, dynamic> toJson() => {
        'name': name,
        'email': email,
      };
}```

## Parse JSON Array to List
```dart
// A function that converts a response body into a List<Photo>.
List<Photo> parsePhotos(String responseBody) {
  final parsed =
      (jsonDecode(responseBody) as List).cast<Map<String, dynamic>>();

  return parsed.map<Photo>((json) => Photo.fromJson(json)).toList();
}

Future<List<Photo>> fetchPhotos(http.Client client) async {
  final response = await client
      .get(Uri.parse('https://jsonplaceholder.typicode.com/photos'));

  // Synchronously run parsePhotos in the main isolate.
  return parsePhotos(response.body);
}```

## [isolate](https://docs.flutter.dev/cookbook/networking/background-parsing)
JSON parsing에 시간이 오래 걸리면 파싱을 백그라운드에서 진행할 수 있도록 코드 실행을 분리시켜 주는 `compute`를 사용한다.

# Fetch the data

```dart
class _MyAppState extends State<MyApp> {
  late Future<Album> futureAlbum;

  @override
  void initState() {
    super.initState();
    futureAlbum = fetchAlbum();
  }
  // ···
}
````

`StatefulWidget`의 `State`위젯에서 `initState` 실행할때 같이 fetch 할 수 있게 해준다.

# Display the data

`FutureBuilder`를 사용한다.

두 가지 파라메터를 전달해야 하는데

1. `Future`. 우리가 응답 받을 객체
2. builder 함수. 플러터가 받은 값을 통해 그릴 것을 기술할 함수

```dart
FutureBuilder<Album>(
  future: futureAlbum,
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      // Null 일 수 있으므로 assertion
      return Text(snapshot.data!.title);
    } else if (snapshot.hasError) {
      return Text('${snapshot.error}');
    }

    // By default, show a loading spinner.
    return const CircularProgressIndicator();
  },
)
```

## why in `initState()`

- 비교적 많이 호출되는 `build`에 구현하는 것은 비효율적이다.
- 상태 변수에 `Future`를 직접 할당 후 사용하는데 이를 할당하게 되면 캐싱되므로 재요청 없이 빠르게 다시 그릴 수 있다.
