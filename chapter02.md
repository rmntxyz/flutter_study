# Installation

## SDK

```sh
$ brew install --cask flutter
```

## Simulator

### [Android](https://docs.flutter.dev/get-started/install/macos#android-setup)

안드로이드 스튜디오 설치 및 에뮬레이터 설치

### [iOS](https://docs.flutter.dev/get-started/install/macos)

이것도 공식 페이지 가서 확인하자.

### diagnosis local env

```sh
$ flutter doctor
```

알아서 추천 명령어나 해야할 일 제시해 줌
모두 pass 되는지 확인
![flutter_doctor](/images/flutter_doctor.png)

## dart pad

파일을 못만든다.  
하지만 모든 코드를 하나의 파일에 넣어서 테스트 해 볼 수 있다.

# Running

```sh
$ flutter create {proj-name}
$ cd {proj-name}
$ flutter run
```

하지만 이 run 명령어는 직접 실행하지 않을거고 아래의 확장 프로그램을 설치해서 실행을 대신하자.

## VSCode extension

- Dart
- Flutter

## project scaffolding

여러 디렉토리가 있는데, 크로스 플랫폼 빌드를 위한 것들이 대부분.  
중요한 파일은 `main.dart`

# development

## [Widget](https://docs.flutter.dev/ui/widgets)

플러터의 대부분은 위젯으로 통한다.  
레고 블럭처럼 개별적인 위젯을 쌓아가면서 만드는 것  
Next.js의 Component와 같다.

### StatelessWidget

이 클래스를 상속받으면 `build(BuildContext context)` 메소드를 구현해 주어야 함  
이 메소드 내에서 구현된 것을 그리게 됨  
`@override` 어노테이션을 사용해 줘야 함

이 번 예제에서는 App의 `build()`에서는 cupertino or material UI 둘 중 하나를 선택해 준다. 일단 `MaterialApp()`을 리턴해 준다.

### MaterialApp

이 클레스에서 봐야 할 것은 `home`  
여기서부터 위젯 스케폴드의 시작점이 된다.
마치 HTML의 body 느낌

### Scaffold

```dart
MaterialApp(
  home: Scaffold(
    appBar: AppBar(
      //...
    )
  )
)
```

### Text, Center

기본적인 위젯들

### trailing comma

dart에서는 중요한 컨벤션.
코드를 보기 편하게 포메팅 해준다.

## Classes

1. Widget을 사용할 때마다 클래스의 인스턴스가 생성된다.
2. 생성자(constructor)
   - 클래스 선언하면서 args로 넘겨주는 값들은 클래스의 생성자가 받아서 쓴다.
   - named parameter는 생성자에 넘기는 args를 이름을 지정하여 넘길 수 있게 함
   - 이건 위에서 봤던 `MaterialApp` 클래스의 `home`과 같은 것
   - 물음표는 optional param을 의미 (TS와 동일)
