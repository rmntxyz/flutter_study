- infinite list를 위한 패키지
- InheritedWidget을 자식으로 사용
- List를 Selector로 감싸서 사용. 이 때 selector 파라미터에 리슨해야 하는 데이터 지정

```dart
 body: Selector<Catalog, int?>(
        selector: (context, catalog) => catalog.itemCount,
        builder: (context, itemCount, child) => ListView.builder(
          itemCount: itemCount, //null이면 infinite list. 값이 존재할 경우 마지막 항목에서 스크롤 정지
          padding: const EdgeInsets.symmetric(vertical: 18),
          itemBuilder: (context, index) {
            var catalog = Provider.of<Catalog>(context);

            return switch (catalog.getByIndex(index)) {
              Item(isLoading: true) => const LoadingItemTile(),
              var item => ItemTile(item: item)
            };

```
