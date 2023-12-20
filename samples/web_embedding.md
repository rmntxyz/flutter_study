# @js.JSExport() ([링크](https://api.flutter.dev/flutter/dart-js_interop/JSExport-class.html))

해당 어노테이션이 있는 클래스는 js_util의 createDartExport를 통해서 JS 객체가 생성되어 다트로 반환. 특정 인스턴스 또는 클래스 전체에 사용 가능. 

## _flutter.loader

`web/js/index.html`에서 <body>태그 아래 쪽을 hostElement로 지정

```html
 <script>
      window.addEventListener("load", function (ev) {
        // Embed flutter into div#flutter_target
        let target = document.querySelector("#flutter_target");
        _flutter.loader.loadEntrypoint({
          onEntrypointLoaded: async function (engineInitializer) {
            let appRunner = await engineInitializer.initializeEngine({
              hostElement: target,
            });
            await appRunner.runApp();
          },
        });
      });
    </script>
    <script src="js/demo-js-interop.js" defer></script>
    <script src="js/demo-css-fx.js" defer></script>

```

## js 파일

`web/js/demo-js-interop.js`. index.html에 id 설정된 요소에 대한 함수 나열. 

## main.dart

StatefulWidget의 initState에서 js_util의 createDartExport 사용

```dart
  void initState() {
    super.initState();
    final export = js_util.createDartExport(this);
    js_util.setProperty(js_util.globalThis, '_appState', export);
    js_util.callMethod<void>(js_util.globalThis, '_stateSet', []);
  }
```

플러터 앱에서는 main.dart대로 실행되고, 웹에서는 `web/js/index.html`대로 실행되는 건가? 


  



