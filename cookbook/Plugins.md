# Play and pause a video

## Add the `video_player` dependency

```sh
$ flutter pub add video_player
```

## Add permissions to your app

### Android

`/android/app/src/main/AndroidManifest.xml` 파일을 수정해야 한다.

`<application\>` 선언 바로 다음에 넣는다.

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application ...>

    </application>

    <uses-permission android:name="android.permission.INTERNET"/>
</manifest>
```

### iOS

`/ios/Runner/Info.plist` 파일을 수정한다.

```xml
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
</dict>
```

## Create and initialize a `VideoPlayerController`

컨트롤러 초기화를 먼저 해 주면 비디오 연결되고 컨트롤러가 재생 준비를 마친다.

웹상의 동영상에 연결해야 하므로 비동기 처리를 하기 위한 `StatefulWidget`을 사용한다.  
초기화는 `State`클래스 내에서 진행

```dart
class VideoPlayerScreen extends StatefulWidget {
  const VideoPlayerScreen({super.key});

  @override
  State<VideoPlayerScreen> createState() => _VideoPlayerScreenState();
}

class _VideoPlayerScreenState extends State<VideoPlayerScreen> {
  late VideoPlayerController _controller;
  late Future<void> _initializeVideoPlayerFuture;

  @override
  void initState() {
    super.initState();

    // Create and store the VideoPlayerController. The VideoPlayerController
    // offers several different constructors to play videos from assets, files,
    // or the internet.
    _controller = VideoPlayerController.networkUrl(
      Uri.parse(
        'https://flutter.github.io/assets-for-api-docs/assets/videos/butterfly.mp4',
      ),
    );

    _initializeVideoPlayerFuture = _controller.initialize();
  }

  @override
  void dispose() {
    // Ensure disposing of the VideoPlayerController to free up resources.
    _controller.dispose();

    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // Complete the code in the next step.
    return Container();
  }
}
```

`dispose` 메소드에서 `VideoPlayerController.dispose()`를 꼭 실행시켜서 위젯 메모리 정리를 바로바로 할 수 있게 해주는게 중요.

## Disply the video player

비디오의 비율이 올바른지 확인하기 위해서 `AspectRatio` 위젯으로 감싸줄 필요가 있다.

그리고 로딩 화면을 위한 스피너를 먼저 노출시켜 주기 위해  
위의 예제 코드에서 `Container` 대신 아래의 `FutureBuilder` 위젯을 넣는다.

```dart
FutureBuilder(
  future: _initializeVideoPlayerFuture,
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.done) {
      return AspectRatio(
        aspectRatio: _controller.value.aspectRatio,
        child: VideoPlayer(_controller),
      );
    } else {
      return const Center(
        child: CircularProgressIndicator(),
      );
    }
  },
)
```

## Play and pause the video

컨트롤러의 `play()`, `pause()` 메소드를 외부 액션을 통해 호출하도록 버튼을 만들어 준다.

현재 재생중인 상태를 확인하려면 `VideoPlayerController.value.isPlaying` 값에 접근하면 된다.

```dart
FloatingActionButton(
  onPressed: () {
    // Wrap the play or pause in a call to `setState`. This ensures the
    // correct icon is shown.
    setState(() {
      // If the video is playing, pause it.
      if (_controller.value.isPlaying) {
        _controller.pause();
      } else {
        // If the video is paused, play it.
        _controller.play();
      }
    });
  },
  // Display the correct icon depending on the state of the player.
  child: Icon(
    _controller.value.isPlaying ? Icons.pause : Icons.play_arrow,
  ),
)
```

# Add ads to your mobile Flutter app or game

광고를 편하게 넣고 싶다면 구글에서 제공하는 [Admob](https://admob.google.com/home/)을 사용하면 된다.

이것을 사용하기 쉽게 해 주는 패키지는 `google_mobile_ads`이다.

사용법은 직접 [링크](https://docs.flutter.dev/cookbook/plugins/google-mobile-ads#1-get-admob-app-ids)를 따라가서 보자

# Use camera

카메라를 사용할 수 있도록 해 주는 `camera` 플러그인

## Add the required dependencies

세 가지의 의존성 패키지를 설치한다.

```sh
$ flutter pub add camera path_provider path
```

### Android

minSdkVersion을 21보다 높게 설정해준다.

### iOS

`ios/Runner/Info.plist` 설정에서 카메라 및 마이크 접근을 허용해 주어야 한다.

```xml
<key>NSCameraUsageDescription</key>
<string>Explanation on why the camera access is needed.</string>
<key>NSMicrophoneUsageDescription</key>
<string>Explanation on why the microphone access is needed.</string>
```

## Get a list of the available cameras

사용 가능한 카메라의 리스트를 얻고 하나를 선택

```dart
// Ensure that plugin services are initialized so that `availableCameras()`
// can be called before `runApp()`
WidgetsFlutterBinding.ensureInitialized();

final cameras = await availableCameras();

final firstCamera = cameras.first;
```

## Create and initialize the `CameraController`

`CameraController`도 초기화가 필요하기 때문에
`StatefulWidget` 필요

```dart
// A screen that allows users to take a picture using a given camera.
class TakePictureScreen extends StatefulWidget {
  const TakePictureScreen({
    super.key,
    required this.camera,
  });

  final CameraDescription camera;

  @override
  TakePictureScreenState createState() => TakePictureScreenState();
}

class TakePictureScreenState extends State<TakePictureScreen> {
  late CameraController _controller;
  late Future<void> _initializeControllerFuture;

  @override
  void initState() {
    super.initState();
    // To display the current output from the Camera,
    // create a CameraController.
    _controller = CameraController(
      // Get a specific camera from the list of available cameras.
      widget.camera,
      // Define the resolution to use.
      ResolutionPreset.medium,
    );

    // Next, initialize the controller. This returns a Future.
    _initializeControllerFuture = _controller.initialize();
  }

  @override
  void dispose() {
    // Dispose of the controller when the widget is disposed.
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // Fill this out in the next steps.
    return Container();
  }
}
```

여기서도 보이지만 `dispose` 중요하다.

## Use a CameraPreview to display the camera’s feed

카메라 프리뷰를 표시해 주는 `CameraPreview` 위젯을 사용해 보자.

`CameraPreview`를 사용하기 전에 반드시 `CameraController`는 초기화 되어 있어야 한다.  
위 예제 코드에서 `_initializeControllerFuture()`가 완료될 때까지 기다려아 한다.

```dart
FutureBuilder<void>(
  future: _initializeControllerFuture,
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.done) {
      // If the Future is complete, display the preview.
      return CameraPreview(_controller);
    } else {
      // Otherwise, display a loading indicator.
      return const Center(child: CircularProgressIndicator());
    }
  },
)
```

## Take a picture with the `CameraController`

`CameraController`를 사용하면 크로스 플랫폼 간소화된 파일 추상화인 `XFile`을 반환하는 `takePicture()`` 메서드를 사용하여 사진을 찍을 수 있음

Android와 IOS 모두에서 새 이미지는 각각의 **캐시 디렉터리**에 저장되며, 해당 위치의 경로는 `XFile`에 존재

- 카메라 초기화 확인.
- `takePicture()`가 `Future<XFile>`을 반환하는지 확인

이를 꼭 확인해야 한다.

```dart
FloatingActionButton(
  onPressed: () async {
    try {
      await _initializeControllerFuture;

      final image = await _controller.takePicture();
    } catch (e) {
      print(e);
    }
  },
  child: const Icon(Icons.camera_alt),
)
```

## Display the picture with an Image widget

```dart
Image.file(File('path/to/my/picture.png'));
```

## Complete example

```dart
import 'dart:async';
import 'dart:io';

import 'package:camera/camera.dart';
import 'package:flutter/material.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  final cameras = await availableCameras();

  final firstCamera = cameras.first;

  runApp(
    MaterialApp(
      theme: ThemeData.dark(),
      home: TakePictureScreen(
        camera: firstCamera,
      ),
    ),
  );
}

class TakePictureScreen extends StatefulWidget {
  const TakePictureScreen({
    super.key,
    required this.camera,
  });

  final CameraDescription camera;

  @override
  TakePictureScreenState createState() => TakePictureScreenState();
}

class TakePictureScreenState extends State<TakePictureScreen> {
  late CameraController _controller;
  late Future<void> _initializeControllerFuture;

  @override
  void initState() {
    super.initState();
    _controller = CameraController(
      widget.camera,
      ResolutionPreset.medium,
    );

    _initializeControllerFuture = _controller.initialize();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Take a picture')),
      body: FutureBuilder<void>(
        future: _initializeControllerFuture,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.done) {
            return CameraPreview(_controller);
          } else {
            return const Center(child: CircularProgressIndicator());
          }
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () async {
          try {
            await _initializeControllerFuture;

            final image = await _controller.takePicture();

            if (!mounted) return;

            await Navigator.of(context).push(
              MaterialPageRoute(
                builder: (context) => DisplayPictureScreen(
                  imagePath: image.path,
                ),
              ),
            );
          } catch (e) {
            print(e);
          }
        },
        child: const Icon(Icons.camera_alt),
      ),
    );
  }
}

class DisplayPictureScreen extends StatelessWidget {
  final String imagePath;

  // Constant Constructor
  const DisplayPictureScreen({super.key, required this.imagePath});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Display the Picture')),
      body: Image.file(File(imagePath)),
    );
  }
}
```
