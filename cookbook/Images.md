# Adding assets and images

플러터는 코드 뿐만 아니라 _assets_ 를 가지고 있을 수
있다.  
_assets_ 는 앱과 함께 번들링 되고 배포된다. 그리고 런타임에 접근가능하다.  
정적 데이타인 JSON, 설정 파일(yaml, toml, etc), 아이콘 혹은 이미지(JPEG, WebP, GIF, PNG, etc)

## Specifying assets

`pubspec.yaml` 파일에 앱에서 사용할 에셋들을 적는다.

```yaml
flutter:
  assets:
    - assets/my_icon.png
    - directory/
    - directory/subdirectory/
```

끝에 `/`를 붙이면 해당 디렉토리의 모든 셋들을 포함한다는 의미다.

> 디렉토리로 지정할 경우, 적혀있는에 디렉토리의 에셋들만 포함된다. 하위 디렉토리에 존재하는 에셋들을 포함시키려면 따로 명시해 주어야 한다.  
> 한 가지 예외가 있는데 그것은 **해상도 인식 이미지** 를 위한 하위 디렉토리는 자동으로 인식된다.

## Asset bundling

yaml에 명시한 `assets` 섹션은 번들링시 앱에 포함된다.  
이 에셋들은 경로가 식별자로 동작하며 파일에 선언한 에셋의 순서는 중요하지 않다.

빌드 하는 동안 플러터는 이 에셋들을 _asset bundle_ 이라고 부르는 특별한 저장소에 넣는다. 그리고 런타임에 앱에서 이 저장소로부터 에셋을 불러 사용한다.

## Loading assets

`AssetBundle` 오브젝트를 통해 에셋에 접근 할 수 있다.

### Loading text assets

플러터는 `rootBundle`을 통해 쉽게 에셋에 접근 가능하다.

하지만 `DefaultAssetBundle`을 사용하여 현재 `BuildContext`에 대한 `AssetBundle`을 가져오는 것 을 추천한다.  
이렇게 하면 부모 위젯이 런타임에 다른 `AssetBundle`로 대체할 수 있다. (언어셋 변경과 같이 동적으로 데이타 변경 시 사용 가능)

위젯 밖에서 에셋에 접근하려면 `AssetBundle`을 사용할 수 없다. 오직 `rootBundle`을 이용해 직접 에셋을 로드해야 한다.

```dart
import 'package:flutter/services.dart' show rootBundle;

Future<String> loadAsset() async {
  return await rootBundle.loadString('assets/config.json');
}
```

### Loading images

각 위젯을 `build()` 메소드에서 `AssetImage` 클래스를 사용하면 된다.

```dart
return const Image(image: AssetImage('assets/background.png'));
```

### Resolution-aware image assets

플러터는 현재 해상도(device pixel ratio)에 적합한 이미지를 불러올 수 있다.

`AssetImage`는 논리적으로 요청된 에셋을 현재 디바이스 픽셀 비율에 가장 근접하게 일치하는 에셋에 매핑한다.

이러한 매핑이 작동하려면 에셋이 특정 디렉토리 구저에 따라 정렬되어야 한다.

```
.../image.png
.../Mx/image.png
.../Nx/image.png
```

여기서 M, N은 pixel ratio를 뜻하며 현재 기기가 해당 값에 가장 근접한 값을 가진 디렉토리의 이미지를 가져와 사용한다.

```
.../my_icon.png       (mdpi baseline)
.../1.5x/my_icon.png  (hdpi)
.../2.0x/my_icon.png  (xhdpi)
.../3.0x/my_icon.png  (xxhdpi)
.../4.0x/my_icon.png  (xxxhdpi)
```

만약 pixel ratio가 2.7이라면 `.../3.0x/my_icon.png` 이미지 에셋이 선택된다.

만약 실제 파일이 없는 경우 해상도가 가장 낮은 에셋이 폴백으로 사용된다.

### Asset images in package dependencies

의존성 패키지에 있는 에셋을 가져오려고 하는 경우,

```dart
return const AssetImage('icons/heart.png', package: 'my_icons');
```

# Display Images from the internet

```dart
Image.network('https://picsum.photos/250?image=9'),
```

gif도 가능하다.

```dart
Image.network(
    'https://docs.flutter.dev/assets/images/dash/dash-fainting.gif');
```

`FadeInImage`를 사용하여 Fade-in 효과를 줄 수 있다.

```dart
FadeInImage.memoryNetwork(
  placeholder: kTransparentImage,
  image: 'https://picsum.photos/250?image=9',
),
```

에셋으로 placeholder를 사용하려면

```yaml
 flutter:
   assets:
+    - assets/loading.gif
```

```dart
FadeInImage.assetNetwork(
  placeholder: 'assets/loading.gif',
  image: 'https://picsum.photos/250?image=9',
),
```

기능이 많은 이미지 위젯을 사용하고 싶다면,
`extended_image` 패키지를 사용할 수도 있겠다.

```dart
ExtendedImage.network(
  imageUrl,
  width: 200,
  height: 200,
  fit: BoxFit.fill,
  cache: true,
  border: Border.all(color: Colors.red, width: 1.0),
  shape: BoxShape.circle,
  borderRadius: BorderRadius.all(Radius.circular(30.0)),
)
```

**컨트롤러 사용의 공식:**

1. `StatefulWidget`준비
2. `State`클래스에서 컨트롤러 타입 변수 `late` 키워드로 선언
3. `State`클래스에서 컨트롤러 초기화 관련 `Future` 타입 변수 `late` 키워드로 선언
4. `initState()` 메소드에서 각각 초기화 하면서 변수에 할당
5. `dispose` 메소드에 컨트롤러 dispose 추가

**비동기 통신 결과를 화면에 보여줄때 공식:**

1. `FutureBuilder` 사용
2. `future` param에 초기화 관련 `Future` 타입 선언 변수 할당
3. `builder` param에 위젯을 리턴하는 콜백 함수 구현하여 할당
4. `snapshot`에 담겨 넘어온 state를 살펴 로딩 전에 돌돌이 넣어 줌
