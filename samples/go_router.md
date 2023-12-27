- 첫 화면 위젯 빌드 시 MaterialApp.router()로 설정.
- nested route은 ShellRoute 이용.
- 데모 앱은 어딘가 잘못되었는지 여기 저기 누르다 보면 동작 정지

```dart

 @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      builder: ...,
      routerConfig: GoRouter(
        refreshListenable: auth, //로그인 여부를 확인하기 위한 파라미터
        debugLogDiagnostics: true,
        initialLocation: '/books/popular',
        //로그인 되어 있지 않으면 리디렉트
        redirect: (context, state) {
          final signedIn = BookstoreAuth.of(context).signedIn;
          if (state.uri.toString() != '/sign-in' && !signedIn) {
            return '/sign-in';
          }
          return null;
        },
        routes: [
          ShellRoute(
            navigatorKey: appShellNavigatorKey, //nested route이 사용할 키
            builder: (context, state, child) {
              return BookstoreScaffold(
                selectedIndex: switch (state.uri.path) {
                  var p when p.startsWith('/books') => 0,
                  var p when p.startsWith('/authors') => 1,
                  var p when p.startsWith('/settings') => 2,
                  _ => 0,
                },
                child: child,
              );
            },
            routes: [
              ShellRoute(
                pageBuilder: ...
                routes: [
                  GoRoute(
                    path: '/books/popular',
                    pageBuilder:...
                    routes: [
                      GoRoute(
                        path: 'book/:bookId',
                        parentNavigatorKey: appShellNavigatorKey, //부모 키 입력

```

네비게이션 관련 방법 세 가지가 모두 나와 있는 부분

```dart
 TextButton(
            child: const Text('View author (Push)'),
              onPressed: () {
                Navigator.of(context).push<void>( //네비게이터 이용
                  MaterialPageRoute<void>(
                    builder: (context) => AuthorDetailsScreen(
                      author: book!.author,
                      onBookTapped: (book) {
                        GoRouter.of(context).go('/books/all/book/${book.id}'); //고라우터 이용
                      },
                    ),
                  ),
                );
              },
            ),
            Link(
              uri: Uri.parse('/authors/author/${book!.author.id}'), //웹뷰를 위한 url_launcher 이용
              builder: (context, followLink) => TextButton(
                onPressed: followLink,
                child: const Text('View author (Link)'),
              ),
            ),

```
