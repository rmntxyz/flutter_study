# 6.1

## key in widget

여러 위젯 간의 구분을 위한 고유 식별자

## appBar

앱 상단의 바. 배경색(background color), 타이틀(타이틀 텍스트 색상은 foreground color로 변경할 수 있다), 그림자(elevation) 등 설정 가능. appBar 아래 body 배경색은 scaffold의 background color로 지정

# 6.2

## http 패키지

데이터 페칭을 위한 url request에 필요.

### 설치 방법

1. pub.dev(node.js의 npm처럼, 다트 & 플러터용 패키지 제공)에서 http 검색
2. pubspec.yaml의 dependencies 섹션에 `http: {버전 넘버}` 적은 후 상단 파일 탭 우측에서 다운로드 화살표(Get Packages) 클릭하면 해당 버전의 패키지 설치 시작

=== or ===

1. 콘솔에 `flutter pub add http` 또는 `dart pub add http`

## get 함수 사용

get함수를 http 패키지에서 임포트하여 바로 사용할 수 있다.

```dart
import 'package:http/http.dart';

void getTodaysToons() async {
final url = Uri.parse(...)
final response = await get(url);
}
```

하지만 구체성을 위해 `get(url)` 대신 `http.get(url)`로 사용

```dart
import 'package:http/http.dart' as http;

Future<List<WebtoonModel>> getTodaysToons() async {
final url = Uri.parse(...)
final response = await http.get(url);

return ...
}
```

# 6.3 - 6.4

## jsonDecode(response.body)

string 타입으로 받은 데이터를 json으로 변환

## fromJson

json 데이터 속의 객체를 인스턴스화(TS로 치자면 타입 지정?)하기 위해 플러터에서 자주 사용되는 named constructor

```dart
class WebtoonModel{
    final String title, thumb, id;

    WebtoonModel.fromJson(Map<String, dynamic> json)
    : title = json['title'],
      thumb = json['thumb'],
      id = json['id'];
}
```

# 6.5 - 6.6

## 페치한 데이터 사용하기

### 첫 번째 방법

StatefulWidget과 setState 사용
![statefulWidget](/images/statefulWidget.png)

### 두 번째 방법(더 바람직)

FutureBuilder 사용

1. 위젯의 body에서 사용
2. future: 사용할 데이터 지정. await 적을 필요 없음
3. builder: context & snapshot 두 가지 인자 중 snapshot을 통해 알 수 있는 정보 - hasData, data, hasError, error, connectionStatus 등

![futureBuilder](/images/futureBuilder.png)

(사진 속 warning은 webtoon를 final로 지정하면 사라짐)

# 6.7

## ListView

항목이 많을 때 Row나 Column은 부적절.

### ListView(children: [...])

항목을 한 번에 로드. 화면에 나타날 항목만 로드하도록 수정해야 함

### ListView.builder

1. scroll direction 등 다양한 설정 가능
2. 항목 개수 지정 가능
3. itemBuilder: 사용자가 보고 있는 아이템에 한해 빌드. 두 가지 인자(context & index)를 사용하며 index를 통해 빌드된 아이템이 무엇인지 확인 가능

### ListView.separated

ListView.builder에 필수 인자 하나 더 추가: separatorBuilder - 리스트 아이템 사이의 공간을 빌드

## 이미지 표시 에러

1. ios simulator의 경우

```dart
Image.network(
    webtoon.thumb,
     headers: const {"Referer": "https://comic.naver.com"},
),

```

2. 웹의 경우

```dart
Image.network(
    webtoon.thumb,
     headers: const {"User-Agent": "..."},
),

```

# 6.8

## extract method

ListView 코드가 길어졌을 때 노란 전구에서 `extract method`를 선택 후 method 이름(여기서는 makeList)만 지정해주면 해당 엘레먼트를 반환하는 함수가 자동으로 따로 생성. 함수의 타입(ListView)과 변수 타입(AsyncSnapshot)도 자동 지정

## unbounded height 에러

코드 저장 & 시뮬레이터 리프레시 후 레이아웃에 필요한 height을 알 수 없을 때 발생. 해당 엘레먼트를 Expanded widget에 넣어주면 가용 공간을 모두 사용하도록 지정됨

```dart
Expanded(child: makeList(snapshot))
```

# 6.9

## GestureDetector

onTap, onLongPress, onVerticalDrag, onHorizontalDrag 등 대부분의 동작 감지

## Navigator

특정 경로로 푸쉬 가능

### Navigator의 route

MaterialPageRoute: 이동 목적지인 widget을 감싸서 화면처럼 보이게 해주는 class. 빌더에 목적지 위젯 입력

### fullscreenDialogue

- true: 새 화면이 아래에서 위로. 좌측 상단에 닫기 아이콘 생성
- false: 새 화면이 오른쪽에서 왼쪽으로. 닫기 아이콘 대신 왼쪽 화살표 아이콘(appBar의 foregroundColor대로)

## DetailScreen

홈화면을 떠나게 되므로 다시 scaffold 작성 필요. appBar에 앱 이름 대신 웹툰 제목을 받아서 표시

```dart
GestureDetector(
    onTap: () {
        Navigator.push(
            context,
            MaterialPageRoute(
                builder: (context) => DetailScreen(
                    title: title,
                    thumb: thumb,
                    id: id,
                ),
                fullscreenDialog: true
            ),
        );
    },
    child: Column(
        ...
    )
)
```

# 6.10 - 6.11

## Hero

홈 화면에서 상세 화면 이동 시 웹툰 커버 이미지가 중앙으로 이동하며 자연스럽게 화면이 전환되는 애니메이션 효과. 홈 화면과 상세 화면 모두에서 커버 이미지를 Hero로 감싸고 id를 tag로 지정. fullscreenDialogue는 삭제해도 무방.

# 6.12 - 6.14

## 상세 화면

id를 이용해 개별 웹툰 정보를 받아 빌드해야 하므로 initState 필요 -> statefulWidget 사용

# 6.15

## singleChildScrollView

overflow 문제가 있을 때 해당 엘레먼트를 singleChildScrollView로 감싸기

# 6.16

1. 웹뷰를 위해 url launcher 설치:
   `flutter pub add url_launcher`
2. configuration 파일에 코드 복사(ios, 안드로이드 모두)
3. 웹으로 이동하는 함수 삽입

```dart
onButtonTap() async {
    await launchUrlString(
        "https://comic.naver.com/webtoon/detail?titleId=$webtoonId&no=${episode.id}");
  }
@override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onButtonTap,
      child: Container(
        margin: const EdgeInsets.only(bottom: 10),
```

# 6.17

## like button

### shared preferences

휴대폰에 데이터를 담을 수 있는 저장소.

1. `flutter pub add shared_preferences`로 설치

2. `SharedPreferences.getInstance()`를 사용해 연결

3. 데이터 적기

```dart
 await prefs.setStringList('likedToons', []);
```

그 외에 `setInt, setBool, setDouble, setString` 가능

4. 데이터 읽기
   `getInt, getBool, getDouble, getString, getStringList`

```dart
   final likedToons = prefs.getStringList('likedToons');
```

5. 데이터 삭제/추가

```dart
 if (isLiked) {
        likedToons.remove(widget.id);
      } else {
        likedToons.add(widget.id);
      }
```

종합하면,

```dart
late SharedPreferences prefs;
  bool isLiked = false;

  Future initPrefs() async {
    prefs = await SharedPreferences.getInstance();
    final likedToons = prefs.getStringList('likedToons');
    if (likedToons != null) {
      if (likedToons.contains(widget.id) == true) {
        setState(() {
          isLiked = true;
        });
      }
    } else {
      await prefs.setStringList('likedToons', []);
    }
  }

  @override
  void initState() {
    super.initState();
    webtoon = ApiService.getToonById(widget.id);
    episodes = ApiService.getLatestEpisodesById(widget.id);
    initPrefs();
  }

  onHeartTap() async {
    final likedToons = prefs.getStringList('likedToons');
    if (likedToons != null) {
      if (isLiked) {
        likedToons.remove(widget.id);
      } else {
        likedToons.add(widget.id);
      }
      await prefs.setStringList('likedToons', likedToons);
      setState(() {
        isLiked = !isLiked;
      });
    }
  }
```
