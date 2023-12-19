# UI 요소 드래그하기

## LongPressDraggable

길게 누르는 동작을 감지하고 사용자 손가락 근처에 새로운 위젯 표시. 생성된 위젯은 사용자 손가락을 따라 이동

```dart
LongPressDraggable<Item>(
  data: item,
  dragAnchorStrategy: pointerDragAnchorStrategy,
  feedback: DraggingListItem(
    dragKey: _draggableKey,
    photoProvider: item.imageProvider,
  ),
  child: MenuListItem(
    name: item.name,
    price: item.formattedTotalItemPrice,
    photoProvider: item.imageProvider,
  ),
);
```

pointerDragAnchorStrategy: 아이템이 사용자의 손가락을 따라오게 하는 프로퍼티 값

## DragTarget

드래그한 아이템을 드랍하는 곳. 드랍된 아이템을 처리하는 함수는 `onAccept`에 기입

```dart
DragTarget<Item>(
  builder: (context, candidateItems, rejectedItems) {
    return CustomerCart(
      hasItems: customer.items.isNotEmpty,
      highlighted: candidateItems.isNotEmpty,
      customer: customer,
    );
  },
  onAccept: (item) {
    _itemDroppedOnCustomerCart(
      item: item,
      customer: customer,
    );
  },
)
```

# 채팅창 그라데이션

## CustomPaint & CustomPainter

CustomPaint는 이미지를 그리기 위한 캔버스를 제공, CustomPainter는 paint(객체를 그려야 할 때마다 실행)와 shouldRepaint(새로운 인스턴스가 주어졌을 때 실행) 메소드를 이용하는 인터페이스

```dart
@immutable
class BubbleBackground extends StatelessWidget {
  const BubbleBackground({
    super.key,
    required this.colors,
    this.child,
  });

  final List<Color> colors;
  final Widget? child;

  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      painter: BubblePainter(
        colors: colors,
      ),
      child: child,
    );
  }
}

class BubblePainter extends CustomPainter {
  BubblePainter({
    required List<Color> colors,
  }) : _colors = colors;

  final List<Color> _colors;

  @override
  void paint(Canvas canvas, Size size) {
    ...
    final paint = Paint()
    ...
     canvas.drawRect(Offset.zero & size, paint);
  }

  @override
  bool shouldRepaint(BubblePainter oldDelegate) {
    ...
    return ...;
  }
}
```

# 그라데이션 스켈레톤 뷰

## ShadarMask

화면에 그려진 요소에 한해 색상을 입히는 위젯

1. 이미지 로드 전에 검정색 컨테이너가 보이도록 이미지를 검정색 컨테이너 위젯으로 감싸기

```dart
class CircleListItem extends StatelessWidget {
  const CircleListItem({super.key});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 8),
      child: Container(
        width: 54,
        height: 54,
        decoration: const BoxDecoration(
          color: Colors.black,
          shape: BoxShape.circle,
        ),
        child: ClipOval(
          child: Image.network(
            'https://docs.flutter.dev/cookbook'
            '/img-files/effects/split-check/Avatar1.jpg',
            fit: BoxFit.cover,
          ),
        ),
      ),
    );
  }
}
```

2. 그라데이션 색상 지정

`LinearGradient` 이용

```dart
const _shimmerGradient = LinearGradient(
  colors: [
    Color(0xFFEBEBF4),
    Color(0xFFF4F4F4),
    Color(0xFFEBEBF4),
  ],
  stops: [
    0.1,
    0.3,
    0.4,
  ],
  begin: Alignment(-1.0, -0.3),
  end: Alignment(1.0, 0.3),
  tileMode: TileMode.clamp,
);
```

Alignment: 사각형 내의 지점을 나타내는 클래스
tilMode: begin 지점 이전과 end 지점 이후의 그라데이션 방식 지정[링크](https://api.flutter.dev/flutter/painting/LinearGradient/tileMode.html)

3. 그라데이션 위젯 생성

LinearGradient의 `createShader`메소드 이용

```dart
 Widget build(BuildContext context) {
    if (!widget.isLoading) {
      return widget.child;
    }

    return ShaderMask(
      blendMode: BlendMode.srcATop,
      shaderCallback: (bounds) {
        return _shimmerGradient.createShader(bounds);
      },
      child: widget.child,
    );
  }
```

3. 그라데이션 위젯으로 검정색 컨테이너 감싸기

4. 화면 전체 스켈레톤 뷰에 그라데이션 넣기 (나중에 직접 해보기)

5. 그라데이션 애니메이션화하기

LienearGradient의 `transform` 프로퍼티 사용

# 다운로드 버튼

StatelessWidget으로 버튼을 만들고 DownloadStatus에 따라 버튼 모양/텍스트 상이하게 표시. 예제에서는 다운로드를 시뮬레이션했는데, 실제 다운로드 사용 시 상태를 어떻게 파악할까?

```dart
enum DownloadStatus {
  notDownloaded,
  fetchingDownload,
  downloading,
  downloaded,
}

@immutable
class DownloadButton extends StatelessWidget {
  const DownloadButton({
    super.key,
    required this.status,
    this.transitionDuration = const Duration(
      milliseconds: 500,
    ),
  });

  final DownloadStatus status;
  final Duration transitionDuration;

  bool get _isDownloading => status == DownloadStatus.      downloading;

  bool get _isFetching => status == DownloadStatus.fetchingDownload;

  bool get _isDownloaded => status == DownloadStatus.downloaded;

  @override
  Widget build(BuildContext context) {
    return ButtonShapeWidget(
      transitionDuration: transitionDuration,
      isDownloaded: _isDownloaded,
      isDownloading: _isDownloading,
      isFetching: _isFetching,
    );
  }
}

@immutable
class ButtonShapeWidget extends StatelessWidget {
  const ButtonShapeWidget({
    super.key,
    required this.isDownloading,
    required this.isDownloaded,
    required this.isFetching,
    required this.transitionDuration,
  });

  final bool isDownloading;
  final bool isDownloaded;
  final bool isFetching;
  final Duration transitionDuration;

  @override
  Widget build(BuildContext context)
```

immutable: 생성 후 수정될 수 없는 객체
enum: 여러 상수로 이루어진 고정 집합
(예:

```dart
enum Vehicle implements Comparable<Vehicle> {
  car(tires: 4, passengers: 5, carbonPerKilometer: 400),
  bus(tires: 6, passengers: 50, carbonPerKilometer: 800),
  bicycle(tires: 2, passengers: 1, carbonPerKilometer: 0);
```

)

# 중첩된 네비게이션

모든 route를 top-level에 설정해두면 엄청나게 많은 진입점이 존재하게 될 것. 모든 것을 관리하기도 쉽지 않을 것이고.

그렇다면 하나의 route에 서로 다른 다수의 페이지를 중첩시켜 다룰 수 있게 하면 관리가 용이해질 것이다.

top-level `Navigator`에 중첩시킨 또 다른 `Navigator`를 선언하여 보다 깔끔하게 구조화 가능하다.

```dart
const routeHome = '/';
const routeSettings = '/settings';
const routePrefixDeviceSetup = '/setup/';
const routeDeviceSetupStart = '/setup/$routeDeviceSetupStartPage';
const routeDeviceSetupStartPage = 'find_devices';
const routeDeviceSetupSelectDevicePage = 'select_device';
const routeDeviceSetupConnectingPage = 'connecting';
const routeDeviceSetupFinishedPage = 'finished';
```

위 라우팅 구조를 분석해보면,

- `/` 홈 화면
- `/settings`: 설정 화면
- `/setup/`: 셋업 화면의 프리픽스, 이 후에 나올 페이지들의 상위 레벨 라우팅 경로로 존재
- `/setup/$routeDeviceSetupStartPage`: `find_devices` 페이지를 해당 라우터의 첫 페이지로 쓰기 위함
- 나머지는 각 서브 라우팅 페이지들의 이름들이다.

`MaterialApp` 같은 경우에는 top-level `Navigator`의 기능을 한다.  
때문에 `Navigator` 클래스가 가지고 있는 `onGenerateRoute` 콜백 파라메터를 인스턴스 생성시 넣어줄 수 있다.

`onGenerateRoute`는 push/pop 발생시 실행된다.  
여기서 파라메터로 넘어온 값을 보고 해당되는 페이지를 라우팅 해 준다.

```dart
  onGenerateRoute: (settings) {
    late Widget page;
    if (settings.name == routeHome) {
      page = const HomeScreen();
    } else if (settings.name == routeSettings) {
      page = const SettingsScreen();
    } else if (settings.name!.startsWith(routePrefixDeviceSetup)) {
      final subRoute =
          settings.name!.substring(routePrefixDeviceSetup.length);
      page = SetupFlow(
        setupPageRoute: subRoute,
      );
    } else {
      throw Exception('Unknown route: ${settings.name}');
    }

    return MaterialPageRoute<dynamic>(
      builder: (context) {
        return page;
      },
      settings: settings,
    );
  },
```

중첩된 페이지는 SetupFlow라는 이름의 `StatefulWidget`을 만들어서 관리한다.  
간단하게 서브 네임 받아서 해당 페이지 위젯을 보여주고 내부 `Navigator`에 push해 주는 역할.
위젯 내부에는 `Navigator`가 별도로 존재하는데 별도의 `GlobalKey`값을 넣어서 생성하기 때문에

```dart
_navigatorKey.currentState!.pushNamed(routeDeviceSetupSelectDevicePage);
```

클래스 내에서 요런 식으로 접근이 가능하도록 되어 있다.

이 클래스 안에서 각각의 페이지는 자신이 가지는 콜백에 다음 페이지에 해당하는 서브 네임을 `pushNamed(SUB_NAME)`하는 함수를 등록해서 연속적으로 진행되도록 했다.

```dart
  Route _onGenerateRoute(RouteSettings settings) {
    late Widget page;
    switch (settings.name) {
      case routeDeviceSetupStartPage:
        page = WaitingPage(
          message: 'Searching for nearby bulb...',
          onWaitComplete: _onDiscoveryComplete,
        );
      case routeDeviceSetupSelectDevicePage:
        page = SelectDevicePage(
          onDeviceSelected: _onDeviceSelected,
        );
      case routeDeviceSetupConnectingPage:
        page = WaitingPage(
          message: 'Connecting...',
          onWaitComplete: _onConnectionEstablished,
        );
      case routeDeviceSetupFinishedPage:
        page = FinishedPage(
          onFinishPressed: _exitSetup,
        );
  }
```

이것이 nested Navigator의 동작 로직

근데 내 생각에는 [`go_router`](https://pub.dev/packages/go_router) 써야 될 듯? 위 방식은 너무 raw 함.

**참고: `??` = "if null"**

# 포토 필터 캐로셀

핵심은 `FilterSelector`의 `State`위젯의

1. `PageController`
2. `build()`메소드
3. 위젯을 리턴하는 함수인 `_buildCarousel`에 있는 `Flow` 인스턴스 생성법
4. 그 arg인 `delegate` 사용법

## `PageController`

페이지 컨트롤러를 생성, 페이지네이션을 이루는 각 아이템을 몇 개 노출할 것인지 뷰포트 분할 값을 지정해 준다.

```dart
 @override
  void initState() {
    super.initState();
    _page = 0;
    _controller = PageController(
      initialPage: _page,
      viewportFraction: _viewportFractionPerItem,
    );
    _controller.addListener(_onPageChanged);
  }
```

## `build()` 메소드

`Scrollable` 위젯의 `axisDirection`, `physics`, `viewportBuilder`를 보자

```dart
  @override
  Widget build(BuildContext context) {
    return Scrollable(
      controller: _controller,
      axisDirection: AxisDirection.right,
      physics: const PageScrollPhysics(),
      viewportBuilder: (context, viewportOffset) {
        return LayoutBuilder(
          builder: (context, constraints) {
            final itemSize = constraints.maxWidth * _viewportFractionPerItem;
            viewportOffset
              ..applyViewportDimension(constraints.maxWidth)
              ..applyContentDimensions(0.0, itemSize * (filterCount - 1));

            return Stack(
              alignment: Alignment.bottomCenter,
              children: [
                _buildShadowGradient(itemSize),
                _buildCarousel(
                  viewportOffset: viewportOffset,
                  itemSize: itemSize,
                ),
                _buildSelectionRing(itemSize),
              ],
            );
          },
        );
      },
    );
  }
```

## `Flow` 와 `delegate`

Flow 선언 (FilterSelector 내부에 선언된 위젯이다.)

```dart
  Widget _buildCarousel({
    required ViewportOffset viewportOffset,
    required double itemSize,
  }) {
    return Container(
      height: itemSize,
      margin: widget.padding,
      child: Flow(
        // 동작을 위임할 수 있다.
        delegate: CarouselFlowDelegate(
          viewportOffset: viewportOffset,
          filtersPerScreen: _filtersPerScreen,
        ),
        children: [
          for (int i = 0; i < filterCount; i++)
            FilterItem(
              onFilterSelected: () => _onFilterTapped(i),
              color: itemColor(i),
            ),
        ],
      ),
    );
  }
```

`Flow`의 동작을 위임하여 구현하는 클래스

```dart
class CarouselFlowDelegate extends FlowDelegate {
  CarouselFlowDelegate({
    required this.viewportOffset,
    required this.filtersPerScreen,
  }) : super(repaint: viewportOffset);

  final ViewportOffset viewportOffset;
  final int filtersPerScreen;

  @override
  void paintChildren(FlowPaintingContext context) {
    final count = context.childCount;

    // All available painting width
    final size = context.size.width;

    // The distance that a single item "page" takes up from the perspective
    // of the scroll paging system. We also use this size for the width and
    // height of a single item.
    final itemExtent = size / filtersPerScreen;

    // The current scroll position expressed as an item fraction, e.g., 0.0,
    // or 1.0, or 1.3, or 2.9, etc. A value of 1.3 indicates that item at
    // index 1 is active, and the user has scrolled 30% towards the item at
    // index 2.
    final active = viewportOffset.pixels / itemExtent;

    // Index of the first item we need to paint at this moment.
    // At most, we paint 3 items to the left of the active item.
    final min = math.max(0, active.floor() - 5).toInt();

    // Index of the last item we need to paint at this moment.
    // At most, we paint 3 items to the right of the active item.
    final max = math.min(count - 1, active.ceil() + 5).toInt();

    // Generate transforms for the visible items and sort by distance.
    for (var index = min; index <= max; index++) {
      final itemXFromCenter = itemExtent * index - viewportOffset.pixels;
      final percentFromCenter = 1.0 - (itemXFromCenter / (size / 2)).abs();
      final itemScale = 0.5 + (percentFromCenter * 0.5);
      final opacity = 0.25 + (percentFromCenter * 0.75);

      final itemTransform = Matrix4.identity()
        ..translate((size - itemExtent) / 2)
        ..translate(itemXFromCenter)
        ..translate(itemExtent / 2, itemExtent / 2)
        ..multiply(Matrix4.diagonal3Values(itemScale, itemScale, 1.0))
        ..translate(-itemExtent / 2, -itemExtent / 2);

      context.paintChild(
        index,
        transform: itemTransform,
        opacity: opacity,
      );
    }
  }

  @override
  bool shouldRepaint(covariant CarouselFlowDelegate oldDelegate) {
    return oldDelegate.viewportOffset != viewportOffset;
  }
}
```
