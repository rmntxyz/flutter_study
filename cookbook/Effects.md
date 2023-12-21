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
tileMode: begin 지점 이전과 end 지점 이후의 그라데이션 방식 지정([링크](https://api.flutter.dev/flutter/painting/LinearGradient/tileMode).html)

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

4. 화면 전체 스켈레톤 뷰에 그라데이션 넣기: 각 그라데이션 위젯을 감싸는 다른 그라데이션 위젯 필요. 하위 위젯이 필요로 하는 그라데이션과 사이즈를 상위 위젯에 전달하는 식.

- `findRenderObject`: 위젯이 렌더링하는 객체를 찾는 메소드 (`Size get size => (context.findRenderObject() as RenderBox).size`). 이 방법으로 하위 그라데이션 위젯의 폭과 높이를 파악
- `localToGlobal`: 해당 위젯이 전체 화면에서 차지하는 위치 파악. (`widget.localToGlobal(Offset.zero)`)
- `Rect.fromLTWH(double left, double top, double width, double height)`: 주어진 left와 top으로부터 주어진 폭과 높이로 사각형 생성

5. 그라데이션 애니메이션화하기: LienearGradient의 `transform` 프로퍼티 사용

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

# Parallax

이미지 리스트(SingleChildScrollView의 자식)를 위로 스크롤하면 이미지가 아래쪽으로, 아래로 스크롤하면 위쪽으로 이동함으로써 보이는 효과. 리스트 아이템의 스크롤 포지션이 변하면서 내부의 이미지(Positioned.fill로 설정)도 포지션 변경 필요. 화면 상 리스트 아이템의 위치는 레이아웃 단계에서는 알 수 없으므로 내부 이미지의 위치도 이후에 일어나는 그리기 단계에서 파악해야 함. 이 때에 사용하는 위젯이 Flow로서 위젯이 그려지기 직전에 하부 위젯의 위치 조정을 가능하게 함

## Photo filter carousel과 Parallax effct 예제 속의 Flow 사용 비교

- 모두 스크롤 중 자식 위젯의 위치 조정을 목적으로 Flow 사용.
- 모두 Scrollable 위젯을 상위 위젯으로 사용(Filter carousel은 PageView, Parallax는 SingleChildScrollView)
- 하지만 Filter carousel은 Flow 없이도 PageController만으로 정상 작동(예제에서도 Flow는 마지막으로 종합된 코드에서만 사용. 왜 사용했을까? -> Flow를 제외한 코드에서는 처음 로드 시 \_controller.position.hasContentDimensions == false여서 필터가 보이지 않으며, 스크롤이 발생해야 필터가 표시되고 이후 정상 작동. Flow 없이 이 문제를 해결할 수 없을까... -> dimensions가 없을 때에는 selectedIndex = 0, scrolledAmount = 0으로 지정해주니 해결되었다. 예제에서는 왜 Flow를 썼을까...)

## Parallax effect 구현하기

1. 내부 이미지 위젯을 기존의 Positioned.fill에서 `Flow`로 변경

before

```dart
  Widget _buildParallaxBackground(BuildContext context) {
    return Positioned.fill(child: Image.network(imageUrl, fit: BoxFit.cover));
  }
```

after

```dart
Widget _buildParallaxBackground(BuildContext context) {
    return Flow(
      delegate: ParallaxFlowDelegate(
        scrollable: Scrollable.of(context),
        listItemContext: context,
        backgroundImageKey: _backgroundImageKey,
      ),
      children: [
        Image.network(
          imageUrl,
          key: _backgroundImageKey,
          fit: BoxFit.cover,
        ),
      ],
    );
  }
```

2. `FlowDelegate`(paintChildren, shouldRepaint 함수 정의 필요)을 이용해 자식 위젯의 크기와 위치 조정

- `getConstraintsForChild` 메소드로 이미지 폭 결정

```dart
class ParallaxFlowDelegate extends FlowDelegate {
  ParallaxFlowDelegate();

  @override
  BoxConstraints getConstraintsForChild(int i, BoxConstraints constraints) {
    return BoxConstraints.tightFor(
      width: constraints.maxWidth,
    );
  }
```

- 파라미터 값(scrollable, listItemContext, backgroundImageKey)이 변경되면 이미지가 다시 그려지도록 `shouldRepaint` 작성

```dart
@override
bool shouldRepaint(ParallaxFlowDelegate oldDelegate) {
  return scrollable != oldDelegate.scrollable ||
      listItemContext != oldDelegate.listItemContext ||
      backgroundImageKey != oldDelegate.backgroundImageKey;
}
```

- 화면/뷰포트(예제에서는 Scrollable.of(리스트 아이템의 context)로 상위 위젯인 Column의 state 이용)에서 개별 항목이 자리한 위치 계산: `localToGlobal` 이용

```dart
@override
void paintChildren(FlowPaintingContext context) {
  // Calculate the position of this list item within the viewport.
  final scrollableBox = scrollable.context.findRenderObject() as RenderBox;
  final listItemBox = listItemContext.findRenderObject() as RenderBox;
  final listItemOffset = listItemBox.localToGlobal(
      listItemBox.size.centerLeft(Offset.zero),
      ancestor: scrollableBox);
}
```

3. `Alignment.inscribe`를 이용해 이미지가 그려질 자리 마련

- Alignment.inscribe(Size size, Rect rect) 형태로서 주어진 rect 안에서 주어진 size의 사각형을 그림

4. `paintChild`메소드로 transform 값 지정과 함께 이미지 그리기

```dart
 context.paintChild(
    0,
    transform:
        Transform.translate(offset: Offset(0.0, childRect.top)).transform,
  );
```

5. 스크롤 포지션이 변경될 때마다 이미지가 다시 그려지도록 FlowDelegate의 `repaint: scrollable.position`를 superclass 지정

```dart
class ParallaxFlowDelegate extends FlowDelegate {
  ParallaxFlowDelegate({
    required this.scrollable,
    required this.listItemContext,
    required this.backgroundImageKey,
  }) : super(repaint: scrollable.position);
}
```

# FAB

눌렀을 때 세부 메뉴가 펼쳐지듯 나오는 아이콘. 메뉴가 열린 후에는 닫기 아이콘으로 대체.

## IgnorePointer

ignoring 프로퍼티를 통해 boolean를 받으며, ignoring:true일 경우 child로 지정된 위젯이 사용자 제스처 무시. 예제의 경우 메뉴가 열린 후 열기 버튼이 동작하지 않도록 열기 버튼을 IgnorePointer의 child로 사용. 예제에서는 메뉴가 열린 후 오픈 버튼(닫기 버튼, 메뉴 아이콘들과 함께 Stack의 children으로 사용되었으며 메뉴가 열린 후엔 opacity: 0이 되도록 설정)을 disable하기 위해 사용.

## Positioned & Stack

Stack 클래스의 자식의 위치를 결정하는 위젯. Stack과 Positioned 사이엔 Stateless/Stateful 위젯만 있어야 정상 작동(RenderObjectWidget 등은 사용 불가). top(non-null, 자동으로 top:0인 듯), bottom, left, right 값에 따라 stack 안에서 위치가 결정

```dart

  @override
  Widget build(BuildContext context) {
    return SizedBox.expand(
      child: Stack(
        alignment: Alignment.bottomRight,
        clipBehavior: Clip.none,
        children: [
          ...
          Positioned(
          right: ...
        ],
      ),
    );
  }


```
