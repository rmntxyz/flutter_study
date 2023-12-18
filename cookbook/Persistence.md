# Persist data with SQLite

## Add the dependencies

```sh
$ flutter pub add sqflite path
```

아래와 같은 패키지 세트가 필요하다.

```dart
import 'dart:async';

import 'package:flutter/widgets.dart';
import 'package:path/path.dart';
import 'package:sqflite/sqflite.dart';
```

## Data model

```dart
class Dog {
  final int id;
  final String name;
  final int age;

  const Dog({
    required this.id,
    required this.name,
    required this.age,
  });
}
```

## Open the database

어떤 어플리케이션이든 데이터베이스와의 커넥션이 먼저 이루어 진 후에 R/W 가 가능하다. 서버도 마찬가지.

DB와 의 연결은 비동기로 이루어진다.

```dart
// Avoid errors caused by flutter upgrade.
// Importing 'package:flutter/widgets.dart' is required.
WidgetsFlutterBinding.ensureInitialized();
// Open the database and store the reference.
final database = openDatabase(
  join(await getDatabasesPath(), 'doggie_database.db'),
);
```

## Create the dogs table

```dart
final database = openDatabase(
  join(await getDatabasesPath(), 'doggie_database.db'),
  // db가 생성될때 한 번 실행
  onCreate: (db, version) {
    return db.execute(
      'CREATE TABLE dogs(id INTEGER PRIMARY KEY, name TEXT, age INTEGER)',
    );
  },
  // 버전 업/다운 그레이드를 위한 값
  version: 1,
);
```

## Insert

`toMap()`을 모델 클래스에 구현하여 DB에 넣을 수 있게 한다.

```dart
class Dog {
  final int id;
  final String name;
  final int age;

  const Dog({
    required this.id,
    required this.name,
    required this.age,
  });

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'name': name,
      'age': age,
    };
  }
}
```

`db.insert()`를 호출하여 넣는다.

- 첫 번째 매개변수는 테이블 이름
- 두 번째 매개변수는 모델 데이터(Map)
- 세 번째 매개변수는 데이터 삽입 전략

```dart
Future<void> insertDog(Dog dog) async {
  final db = await database;

  await db.insert(
    'dogs',
    dog.toMap(),
    conflictAlgorithm: ConflictAlgorithm.replace,
  );
}
```

```dart
// Create a Dog and add it to the dogs table
var fido = const Dog(
  id: 0,
  name: 'Fido',
  age: 35,
);

await insertDog(fido);
```

## Retrieve the list

```dart
// A method that retrieves all the dogs from the dogs table.
Future<List<Dog>> dogs() async {
  // Get a reference to the database.
  final db = await database;

  // Query the table for all The Dogs.
  final List<Map<String, dynamic>> maps = await db.query('dogs');

  // Convert the List<Map<String, dynamic> into a List<Dog>.
  return List.generate(maps.length, (i) {
    return Dog(
      id: maps[i]['id'] as int,
      name: maps[i]['name'] as String,
      age: maps[i]['age'] as int,
    );
  });
}
```

```dart
// Now, use the method above to retrieve all the dogs.
print(await dogs()); // Prints a list that include Fido.
```

## Update a Dog

데이터 삽입과 유사하게 모델의 Map 컨버젼이 필요하다.

```dart
Future<void> updateDog(Dog dog) async {
  // Get a reference to the database.
  final db = await database;

  // Update the given Dog.
  await db.update(
    'dogs',
    dog.toMap(),
    // Ensure that the Dog has a matching id.
    where: 'id = ?',
    // Pass the Dog's id as a whereArg to prevent SQL injection.
    whereArgs: [dog.id],
  );
}
content_copy
// Update Fido's age and save it to the database.
fido = Dog(
  id: fido.id,
  name: fido.name,
  age: fido.age + 7,
);
await updateDog(fido);

// Print the updated results.
print(await dogs()); // Prints Fido with age 42.
```

# Delete

```dart
Future<void> deleteDog(int id) async {
  // Get a reference to the database.
  final db = await database;

  // Remove the Dog from the database.
  await db.delete(
    'dogs',
    // Use a `where` clause to delete a specific dog.
    where: 'id = ?',
    // Pass the Dog's id as a whereArg to prevent SQL injection.
    whereArgs: [id],
  );
}
```

# Read and write files

파일을 앱에 넣거나 읽고 써야 하는 경우가 발생할 수 있다.

데이터 유지를 위한 저장된 파일, 오프라인 앱, 인터넷에서 다운로드된 파일 등등.

## Find the correct local path

`path_provider` 패키지는 플랫폼(iOS, Android) 상관없이 파일 시스템에 접근 할 수 있는 API를 제공한다.

두 가지 타입의 파일 경로가 있습니다.

- 임시 디렉토리
  - 시스템이 언제든 삭제할 수 있는 파일 디렉토리
- 문서 디렉토리

  - 앱을 위한 파일들을 넣을 수 있는 디렉토리

  ```dart
  Future<String> get _localPath async {
    final directory = await getApplicationDocumentsDirectory();

    return directory.path;
  }
  ```

## Create a reference to the file location

문서 디렉토리를 접근하여 파일을 가져오는 방법

비동기로 진행되므로 async-await

```dart
Future<File> get _localFile async {
  final path = await _localPath;
  return File('$path/counter.txt');
}
```

## Write data to the file

위에 작성된 파일 가져오기 함수를 사용하여 그 안에 카운터 숫자 값을 넣는 예제

`File.writeAsString()` 메소드를 사용한다.

```dart
Future<File> writeCounter(int counter) async {
  final file = await _localFile;

  // Write the file
  return file.writeAsString('$counter');
}
```

## Read data from the file

`File.readAsString()` 메소드를 사용한다.

```dart
Future<int> readCounter() async {
  try {
    final file = await _localFile;

    // Read the file
    final contents = await file.readAsString();

    return int.parse(contents);
  } catch (e) {
    // If encountering an error, return 0
    return 0;
  }
}
```

# Store key-value data on disk

키-값 컬렉션을 다루는 대표적인 예시는 앱 설정과 관련된 것들이다.
Web의 LocalStorage와 유사하다고 볼 수 있다.
간단한 키-값 컬렉션을 디스크에 저장하려고 할 때 사용할 수 있는 `shared_preferences` 플러그인을 살펴보자.

## Add the dependency

```sh
$ flutter pub add shared_preferences
```

## Save data

```dart
final prefs = await SharedPreferences.getInstance();

await prefs.setInt('counter', counter);
```

## Read data

특이사항은 read할 때 `await`가 필요 없다는 점.

```dart
final prefs = await SharedPreferences.getInstance();

final counter = prefs.getInt('counter') ?? 0;
```

## Remove data

```dart
final prefs = await SharedPreferences.getInstance();

await prefs.remove('counter');
```

## Supported types

몇 가지 제약이 있다.

- 오직 primitive types만 지원
  - int
  - double
  - bool
  - String
  - List<String>
- 대량의 데이터 저장에는 알맞지 않음
- 앱을 다시 시작할 때 데이터가 유지된다는 보장이 없음 (???)
