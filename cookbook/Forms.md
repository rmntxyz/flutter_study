# Build a form with validation

1. `GlobalKey`로 `Form`만들기
2. 유효성 검사 로직이 있는 `TextFormField`
3. 유효성 검사 와 폼 제출 동작하는 버튼 만들기

`Form`이란 것도 예상대로 위젯으로 존재
그런데 `GlobalKey`는 무엇이냐

> 앱 전체에서 유일한 키

이렇게 얻은 키 값으로 제출할 폼을 식별할 목적으로 쓰이는 듯 하다.

```dart
import 'package:flutter/material.dart';

// Define a custom Form widget.
class MyCustomForm extends StatefulWidget {
  const MyCustomForm({super.key});

  @override
  MyCustomFormState createState() {
    return MyCustomFormState();
  }
}

class MyCustomFormState extends State<MyCustomForm> {
  // Create a global key that uniquely identifies the Form widget
  // and allows validation of the form.
  //
  // Note: This is a GlobalKey<FormState>,
  // not a GlobalKey<MyCustomFormState>.
  final _formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    // Build a Form widget using the _formKey created above.
    return Form(
      key: _formKey,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          TextFormField(
            // The validator receives the text that the user has entered.
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Please enter some text';
              }
              return null;
            },
          ),
          Padding(
            padding: const EdgeInsets.symmetric(vertical: 16),
            child: ElevatedButton(
              onPressed: () {
                // Validate returns true if the form is valid, or false otherwise.
                if (_formKey.currentState!.validate()) {
                  // If the form is valid, display a snackbar. In the real world,
                  // you'd often call a server or save the information in a database.
                  ScaffoldMessenger.of(context).showSnackBar(
                    const SnackBar(content: Text('Processing Data')),
                  );
                }
              },
              child: const Text('Submit'),
            ),
          ),
        ],
      ),
    );
  }
}
```

## `TextFormField`

유저가 입력 가능한 필드를 만들어 준다.  
특징은 `validator`라는 함수가 제공되는데 이 함수는  
유저의 입력에 문제가 있을 때 문자열(error message)을 리턴하게 하고
만약 문제가 없으면 `null`을 리턴 하도록 작성하면 된다.

여기서 만약 `TextFormField`에 입력한 유저의 텍스트 정보를 읽고 싶다면  
`TextEditingController`를 사용해야 한다.  
사용법은 [예제](https://docs.flutter.dev/cookbook/forms/retrieve-input)를 따라가서 확인하자.

선언한 위젯이 없어질때 컨트롤러 `dispose` 해 주는건 국룰

## Submit

서브밋은 그저 버튼 만들고 `onPressed` 함수 구현만 해 주면 된다.

# Create and style a text field

- `TextField`
- `TextFormField`

플러터는 두 가지의 입력 필드를 제공한다.

## `TextField`

가장 보편적으로 사용하는 입력 위젯

`InputDecoration`으로 스타일 및 부가 속성 지정 가능

```dart
TextField(
  decoration: InputDecoration(
    border: OutlineInputBorder(),
    hintText: 'Enter a search term',
  ),
),
```

## `TextFormField`

`TextField`와 거의 동일하지만  
유효성 검사 기능이 있는 것과, 다른 `FormField`위젯과 통합 가능하다는 점이 다르다.

# Focus and text fields

텍스트 필드가 나타날 때 자동으로 인풋이 포커스 되도록 만들 수 있다.

```dart
TextField(
  autofocus: true,
);
```

만약 포커스를 수동으로 지정해 주려면 `FocusNode`를 사용하면 된다.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'Text Field Focus',
      home: MyCustomForm(),
    );
  }
}

class MyCustomForm extends StatefulWidget {
  const MyCustomForm({super.key});

  @override
  State<MyCustomForm> createState() => _MyCustomFormState();
}

class _MyCustomFormState extends State<MyCustomForm> {
  // FocusNode 선언
  late FocusNode myFocusNode;

  @override
  void initState() {
    super.initState();

    // FocusNode 할당
    myFocusNode = FocusNode();
  }

  @override
  void dispose() {
    // !중요!
    myFocusNode.dispose();

    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Text Field Focus'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 첫 번째 필드는 자동 포커스
            const TextField(
              autofocus: true,
            ),
            // 두 번째 필드는 수동 포커스
            TextField(
              focusNode: myFocusNode,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        // 포커싱!
        onPressed: () => myFocusNode.requestFocus(),
        tooltip: 'Focus Second Text Field',
        child: const Icon(Icons.edit),
      ),
    );
  }
}
```

# Handle changes to a text field

`TextField`의 변화에 따른 입력값을 얻는 방법이 두 가지가 있다.

- `TextField`의 `onChanged`함수 구현
- `TextEditingController`사용

두 개 짬뽕한 예제가 있으니 보면서 이해하자.  
특히 `TextEditingController`의 경우에는 폼 제출 예제에서도 사용했었다.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'Retrieve Text Input',
      home: MyCustomForm(),
    );
  }
}

// Define a custom Form widget.
class MyCustomForm extends StatefulWidget {
  const MyCustomForm({super.key});

  @override
  State<MyCustomForm> createState() => _MyCustomFormState();
}

// Define a corresponding State class.
// This class holds data related to the Form.
class _MyCustomFormState extends State<MyCustomForm> {
  // Create a text controller and use it to retrieve the current value
  // of the TextField.
  final myController = TextEditingController();

  @override
  void initState() {
    super.initState();

    // Start listening to changes.
    myController.addListener(_printLatestValue);
  }

  @override
  void dispose() {
    // Clean up the controller when the widget is removed from the widget tree.
    // This also removes the _printLatestValue listener.
    myController.dispose();
    super.dispose();
  }

  void _printLatestValue() {
    final text = myController.text;
    print('Second text field: $text (${text.characters.length})');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Retrieve Text Input'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              onChanged: (text) {
                print('First text field: $text (${text.characters.length})');
              },
            ),
            TextField(
              controller: myController,
            ),
          ],
        ),
      ),
    );
  }
}
```
