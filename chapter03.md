# Header 만들기

## Color

- Colors.black.shade800

## Row

자식 위젯을 수평으로 배치시킬 때 사용

## Column

자식 위젯을 수직으로 배치시킬 때 사용

## SizedBox

영역을 잡기 위해 사용

## Padding

CSS 패딩 역할

# 개발자 도구

![dev_tools](/images/dev_tools.png);

인스펙터랑 가이드라인 뷰어가 있다. 계층 구조도 볼 수 있고.

# Buttons Section

기억에 남는건 콤마를 꼭 넣어줘야 한다는 것

스트링 따옴표 안의 `$`와 붙여쓰는 문자는 변수로 취급되므로 escaping 해주어야 한다.

## Container

HTML의 `div`의 역할을 한다.

`Container`의 `decoration` arg로 넣은 `BoxDecoration`을 통해 버튼의 rounded corner 스타일 구현 가능

`EdgeInsets.symmetric(verticla: 15, horizontal: 40)`은 패딩을 넣기 쉽게 해 줌

# VSCode Settings

파란 줄이 뜨는 경우가 있는데 그 중 `const` 선언과 관련된 것이 있다.
`const`로 선언된 변수들은 변경할 수 없음. constant로 선언된 값들은 컴파일타임에 결정되고 이는 성능 최적화에 도움이 됨

이런 작업을 수작업으로 할 수 없으므로 자동화 시킬 것
![open_user_setting](/images/open_user_setting.png)
![auto_fix](/images/auto_fix.png)

그리고 트리 지옥에서 벗어날 수 있게 해주는 옵션 체크
![ui_guide](/images/ui_guide.png)

# Code Action

우리의 구세주 노란 전구
프리셋 액션으로 리팩토링 해 준다. 천재다.

# Reusable Widgets

그 전에 Error Lens 확장 기능 설치

노란 전구의 extract widget 기능을 사용하면 재사용 가능한 위젯으로 변경시켜 준다.

# Cards

margin의 개념으로 SizedBox를 자주 쓰는걸 볼 수 있었음

## Icon

`Icons.euro_rounded` 까지 존재하는... 엄청난 프리셋 볼륨

`MainAxisAlignment.spaceBetween` 은 CSS의 `justify-content: space-between` 와 동일한 정렬 효과를 가진다.

`Transfom`도 CSS와 유사. child 포함 구조로 효과의 적용 순서가 정해짐

overflow 처리는 클리핑 처리할 부모 위젯의 속성으로 `clipBehavior: Clip.hardEdge`를 넣어준다.

이렇게 만들어진 카드를 독립적인 위젯 클래스로 만들어서 재사용하는 작업을 마치면 깔끔한 프론트 뷰가 완성된다.

# SingleChildScrollView

스크롤을 만들 수 있게 해 준다.

`MaterialApp` \(  
ㄴ home: `Scaffold` \(  
&emsp;&ensp;ㄴ body: `SingleChildScrollView` \(  
&emsp;&emsp;&emsp;ㄴ child: `Padding` \( ...
