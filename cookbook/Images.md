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

하지만
//////////////////////////////작성중/////////////////////////////////////

위젯 밖에서 에셋에 접근하려면 `AssetBundle`을 사용할 수 없다. 오직 `rootBundle`을 이용해 직접 에셋을 로드해야 한다.

```dart
import 'package:flutter/services.dart' show rootBundle;

Future<String> loadAsset() async {
  return await rootBundle.loadString('assets/config.json');
}
```

### Loading images

```dart
return const Image(image: AssetImage('assets/background.png'));
```
