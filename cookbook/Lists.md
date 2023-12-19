# Grid list

가장 단순한 `GridView.count()`

```dart
GridView.count(
  // Create a grid with 2 columns. If you change the scrollDirection to
  // horizontal, this produces 2 rows.
  crossAxisCount: 2,
  // Generate 100 widgets that display their index in the List.
  children: List.generate(100, (index) {
    return Center(
      child: Text(
        'Item $index',
        style: Theme.of(context).textTheme.headlineSmall,
      ),
    );
  }),
),
```

# Horizontal list

`ListView`가 수평 리스트를 지원한다.

```dart
ListView(
  // This next line does the trick.
  scrollDirection: Axis.horizontal,
  children: <Widget>[
    Container(
      width: 160,
      color: Colors.red,
    ),
    Container(
      width: 160,
      color: Colors.blue,
    ),
    Container(
      width: 160,
      color: Colors.green,
    ),
    Container(
      width: 160,
      color: Colors.yellow,
    ),
    Container(
      width: 160,
      color: Colors.orange,
    ),
  ],
),
```

# 다른 타입의 아이템을을 하나의 리스트에 출력하기

상속 개념이 중요하다.

```dart
abstract class ListItem {}
class HeadingItem implements ListItem {}
class MessageItem implements ListItem {}
```

서로 다른 타입의 클래스를 선언하지만 하나의 추상 클래스를 상속 받는다.

이렇게 하면 실제 사용하는 코드에서 하나의 추상 클래스 타입만 바라보고 위젯 생성이 가능하다.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(
    MyApp(
      items: List<ListItem>.generate(
        1000,
        (i) => i % 6 == 0
            ? HeadingItem('Heading $i')
            : MessageItem('Sender $i', 'Message body $i'),
      ),
    ),
  );
}

class MyApp extends StatelessWidget {
  final List<ListItem> items;

  const MyApp({super.key, required this.items});

  @override
  Widget build(BuildContext context) {
    const title = 'Mixed List';

    return MaterialApp(
      title: title,
      home: Scaffold(
        appBar: AppBar(
          title: const Text(title),
        ),
        body: ListView.builder(
          // Let the ListView know how many items it needs to build.
          itemCount: items.length,
          // Provide a builder function. This is where the magic happens.
          // Convert each item into a widget based on the type of item it is.
          itemBuilder: (context, index) {
            final item = items[index];

            return ListTile(
              title: item.buildTitle(context),
              subtitle: item.buildSubtitle(context),
            );
          },
        ),
      ),
    );
  }
}

/// The base class for the different types of items the list can contain.
abstract class ListItem {
  /// The title line to show in a list item.
  Widget buildTitle(BuildContext context);

  /// The subtitle line, if any, to show in a list item.
  Widget buildSubtitle(BuildContext context);
}

/// A ListItem that contains data to display a heading.
class HeadingItem implements ListItem {
  final String heading;

  HeadingItem(this.heading);

  @override
  Widget buildTitle(BuildContext context) {
    return Text(
      heading,
      style: Theme.of(context).textTheme.headlineSmall,
    );
  }

  @override
  Widget buildSubtitle(BuildContext context) => const SizedBox.shrink();
}

/// A ListItem that contains data to display a message.
class MessageItem implements ListItem {
  final String sender;
  final String body;

  MessageItem(this.sender, this.body);

  @override
  Widget buildTitle(BuildContext context) => Text(sender);

  @override
  Widget buildSubtitle(BuildContext context) => Text(body);
}
```

# Floating app bar

`CustomScrollView` + `SilverAppBar`를 사용하면 되는데  
[예제](https://api.flutter.dev/flutter/material/SliverAppBar-class.html)와 영상을 보면 이해가 더 쉬울듯 하다.

# Work with long lists

그냥 `ListView`는 리스트 개수가 작을때 사용하기 좋다.  
하지만 리스트 개수가 많을 경우 성능상의 이점을 보기 위해서 `ListView.builder` 생성자를 사용하는게 좋다.

`ListView`는 한 번에 모든 리스트들의 위젯을 생성하는데 반해  
`ListView.builder`는 화면에 들어온 리스트들만 생성한다.

# Create a list with spaced items

`Column` 위젯으로 가능

화면에 리스트 수가 적을 경우에는 빈 공간을 균일하게 차지하면서 리스트가 위아래로 펼쳐지고  
리스트의 개수가 많아서 화면의 높이 이상으로 생성된 경우에는 그냥 최소의 스페이스만 남겨놓고 모두 붙어있게 된다.

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  crossAxisAlignment: CrossAxisAlignment.stretch,
  children: List.generate(
    items, (index) => ItemWidget(text: 'Item $index')),
),
```
