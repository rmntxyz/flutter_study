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



