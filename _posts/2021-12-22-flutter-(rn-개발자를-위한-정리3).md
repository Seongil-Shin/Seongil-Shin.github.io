---
title: flutter (rn 개발자를 위한 정리 3)
author: 신성일
date: 2021-12-22 21:00:00 +0900
categories: [study, flutter]
tags: []
---

## Flutter 위젯

## Views

### View 컨테이너

- React Native 에서는 View 가 컨테이너이고 Flexbox를 이용한 레이아웃, 스타일, 터치 핸들링, 접근성제어를 지원

- Flutter에서는 [Container](https://api.flutter.dev/flutter/widgets/Container-class.html)나, [Column](https://api.flutter.dev/flutter/widgets/Column-class.html), [Row](https://api.flutter.dev/flutter/widgets/Row-class.html), [Center](https://api.flutter.dev/flutter/widgets/Center-class.html)같은 위젯 라이브러리의 핵심 레이아웃 위젯을 사용할 수 있음.
  - [레이아웃 위젯](https://flutter-ko.dev/docs/development/ui/widgets/layout)

### FlatList, SectionList

- [ListView ](https://api.flutter.dev/flutter/widgets/ListView-class.html) : 목록의 수가 적은 경우에 가장 적합함.
- 무거운 목록이거나 무한 스크롤 목록일때는 ListView.builder 를 사용
  - 자식들을 필요할 때만 빌드하고, 화면에 나타나야할 자식들만 빌드함

```dart
// Flutter
var data = [ ... ];
ListView.builder(
  itemCount: data.length,
  itemBuilder: (context, int index) {
    return Text(
      data[index],
    );
  },
)
```

### Canvas

- ReactNative에서는 직접적으로 지원하는 컴포넌트가 없음
- Flutter에서는 [CustomPaint](https://api.flutter.dev/flutter/widgets/CustomPaint-class.html), [CustomPainter ](https://api.flutter.dev/flutter/rendering/CustomPainter-class.html)클래스를 사용하여 캔버스를 그릴 수 있다.

```dart
// Flutter
class MyCanvasPainter extends CustomPainter {

  @override
  void paint(Canvas canvas, Size size) {
    Paint paint = Paint();
    paint.color = Colors.amber;
    canvas.drawCircle(Offset(100.0, 200.0), 40.0, paint);
    Paint paintRect = Paint();
    paintRect.color = Colors.lightBlue;
    Rect rect = Rect.fromPoints(Offset(150.0, 300.0), Offset(300.0, 400.0));
    canvas.drawRect(rect, paintRect);
  }

  bool shouldRepaint(MyCanvasPainter oldDelegate) => false;
  bool shouldRebuildSemantics(MyCanvasPainter oldDelegate) => false;
}
class _MyCanvasState extends State<MyCanvas> {

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomPaint(
        painter: MyCanvasPainter(),
      ),
    );
  }
}
```

![Canvas on iOS](</assets/img/2021-12-22-flutter-(rn-개발자를-위한-정리3)/canvas-2cc207759f6ab912bf73e1c3298dc2183618ef207ed989f4d83f6c08fd3a3279.png>)

## 레이아웃

### 레이아웃 속성 정의

- React native에서는 style props를 이용해 flexbox 속성을정의함.

  ```react
  // React Native
  <View
    style={{
      flex: 1,
      flexDirection: 'column',
      justifyContent: 'space-between',
      alignItems: 'center'
    }}
  >
  ```

- flutter에서는 컨트롤 위젯 및 스타일 속성을 결합하여 설계된 위젯을 통해 레이아웃을 정의한다.

  ```dart
  // Flutter
  Center(
    child: Column(
      children: <Widget>[
        Container(
          color: Colors.red,
          width: 100.0,
          height: 100.0,
        ),
        Container(
          color: Colors.blue,
          width: 100.0,
          height: 100.0,
        ),
        Container(
          color: Colors.green,
          width: 100.0,
          height: 100.0,
        ),
      ],
    ),
  )
  ```

  - [Padding](https://api.flutter.dev/flutter/widgets/Padding-class.html), [Align](https://api.flutter.dev/flutter/widgets/Align-class.html), [Stack ](https://api.flutter.dev/flutter/widgets/Stack-class.htm)과 같은 위젯을 사용할 수도 있음.
  - 전체는 [레이아웃 위젯](https://flutter-ko.dev/docs/development/ui/widgets/layout) 에서 확인

### 위젯을 겹쳐 쌓아올리는 방법

- React Native에서는 absolute 을 사용
- flutter에서는 Stack을 사용함.

```dart
// Flutter
Stack(
  alignment: const Alignment(0.6, 0.6),
  children: <Widget>[
    CircleAvatar(
      backgroundImage: NetworkImage(
        'https://avatars3.githubusercontent.com/u/14101776?v=4'),
    ),
    Container(
      decoration: BoxDecoration(
          color: Colors.black45,
      ),
      child: Text('Flutter'),
    ),
  ],
)
```

![Stack on iOS](</assets/img/2021-12-22-flutter-(rn-개발자를-위한-정리3)/stack-04b7bf2727e1eb71f5dfea8430ee833f24be1ced1893ae86270795b2ab76c5b9.png>)

## 스타일링

### 컴포넌트 꾸미기

- react native 에서는 인라인 스타일링과 stylesheets.create를 이용한 스타일링 가능

```react
<View style={styles.container}>
  <Text style={{ fontSize: 32, color: 'cyan', fontWeight: '600' }}>
    This is a sample text
  </Text>
</View>

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center'
  }
});
```

- flutter에서는 Text 위젯은 style 속성에 [TextStyle](https://api.flutter.dev/flutter/dart-ui/TextStyle-class.html)클래스를 사용할 수 있음.
  - 여러 곳에서 같은 스타일을 사용하고 싶으면 따로 빼서 적용가능
  - 다른 위젯에서도 비슷한 방식인가?

```dart
var textStyle = TextStyle(fontSize: 32.0, color: Colors.cyan, fontWeight:
   FontWeight.w600);
	...
Center(
  child: Column(
    children: <Widget>[
      Text(
        'Sample text',
        style: textStyle,
      ),
      Padding(
        padding: EdgeInsets.all(20.0),
        child: Icon(Icons.lightbulb_outline,
          size: 48.0, color: Colors.redAccent)
      ),
    ],
  ),
)
```

### Icons와 Colors를 사용하는 방법은?

- flutter에서는 material 라이브러리를 import하면 다양한 [material icon](https://api.flutter.dev/flutter/material/Icons-class.html)과 [컬러](https://api.flutter.dev/flutter/material/Colors-class.html)를 가져옴

```dart
Icon(Icons.lightbulb_outline, color: Colors.redAccent)
```

- Icons 클래스를 사용할떄는 pubspec.yaml 파일에 uses-material-design : true 를 꼭 설정해줘야함

```yaml
name: my_awesome_application
flutter: uses-material-design: true
```

- 쿠퍼티노 아이콘 사용

```yaml
name: my_awesome_application
dependencies:
  cupertino_icons: ^0.1.0
```

- 컴포넌트 전체적인 색과 스타일을 지정하고 싶으면 ThemeData 클래스를 사용
  - MaterialApp 에서 theme 속성에 ThemeData 객체를 설정한다.
  - Colors 클래스는 material 디자인의 [color paletter](https://material.io/design/color/the-color-system.html#color-usage-and-palettes)에 해당하는 색상을 제공

```dart
class SampleApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Sample App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        textSelectionColor: Colors.red
      ),
      home: SampleAppPage(),
    );
  }
}
```

### 스타일 테마 추가

- React Native에서는 컴포넌트 공통 테마는 stylesheets에 정의한 후 컴포넌트에 사용함
- Flutter에서는 [ThemeData](https://api.flutter.dev/flutter/material/ThemeData-class.html) 클래스에 스타일을 정의하고, [MaterialApp](https://api.flutter.dev/flutter/material/MaterialApp-class.html) 위젯의 테마 속성에 전달함으로써 거의 모든 곳에 균일한 스타일을 적용할 수 있음.

```dart
 @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        primaryColor: Colors.cyan,
        brightness: Brightness.dark,
      ),
      home: StylingPage(),
    );
  }
```

- [Theme](https://api.flutter.dev/flutter/material/Theme-class.html)는 MaterialApp 위젯을 사용하지 않아도 적용가능.
  - data 매개변수에서 ThemeData를 가져와 모든 자식 위젯에 ThemeData를 적용함.

```dart
 @override
  Widget build(BuildContext context) {
    return Theme(
      data: ThemeData(
        primaryColor: Colors.cyan,
        brightness: brightness,
      ),
      child: Scaffold(
         backgroundColor: Theme.of(context).primaryColor,
              ...
              ...
      ),
    );
  }
```

## 상태관리

- State 클래스
- [StatefulWidget](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html)
- [상태 관리 방법](https://flutter-ko.dev/docs/development/data-and-backend/state-mgmt)

### StatelessWidget

- 상태 변화가 필요없는 위젯.
- 객체의 자체적인 구성정보나 위젯의 BuildContext 이외의 것에 의존하지 않을때 유용함

- 상태가 없는 위젯의 build 메서드는 보통 3가지 상황에서만 호출됨
  - 위젯이 트리에 추가될 때
  - 위젯의 부모가 설정이 변경됐을때
  - 사용하고 있는 InheritedWidget이 변경될 때

### StatefulWidget

- setState를 호출하여 Flutter 프레임워크에게 상태가 변경됐음을 알려줌.

  - 이후 앱이 build 메서드를 다시 실행하여 변경사항을 반영함

- 아래는 createState() 메서드를 필요로하는 StatefulWidget을 선언하는 예시

  - 위젯의 상태를 관리하는 상태객체 \_MyStatefulWidgetState를 생성함

  ```dart
  class MyStatefulWidget extends StatefulWidget {
    MyStatefulWidget({Key key, this.title}) : super(key: key);
    final String title;

    @override
    _MyStatefulWidgetState createState() => _MyStatefulWidgetState();
  }
  ```

- 아래는 \_MyStatefulWidgetState 라는 상태 클래스.

  - 상태가 바뀌면 setState가 새로운 toggle 값과 함께 호출됨.
  - 이후 프레임워크가 UI위젯을 다시 빌드함.

  ```dart
  class _MyStatefulWidgetState extends State<MyStatefulWidget> {
    bool showtext=true;
    bool toggleState=true;
    Timer t2;

    void toggleBlinkState(){
      setState((){
        toggleState=!toggleState;
      });
      var twenty = const Duration(milliseconds: 1000);
      if(toggleState==false) {
        t2 = Timer.periodic(twenty, (Timer t) {
          toggleShowText();
        });
      } else {
        t2.cancel();
      }
    }

    void toggleShowText(){
      setState((){
        showtext=!showtext;
      });
    }

    @override
    Widget build(BuildContext context) {
      return Scaffold(
        body: Center(
          child: Column(
            children: <Widget>[
              (showtext
                ?(Text('This execution will be done before you can blink.'))
                :(Container())
              ),
              Padding(
                padding: EdgeInsets.only(top: 70.0),
                child: RaisedButton(
                  onPressed: toggleBlinkState,
                  child: (toggleState
                    ?( Text('Blink'))
                    :(Text('Stop Blinking'))
                  )
                )
              )
            ],
          ),
        ),
      );
    }
  }
  ```

### StatefulWidget 과 StatelessWidget의 모범사례

#### 위젯을 설계할때의 고려사항

1. 위젯이 StatefulWidget인지 StatelessWidget인지 결정하라

   - 위젯이 변화하면 Stateful 사용
   - 위젯이 final이거나 immutable이면, Stateless를 사용

2. 어떤 객체가 위젯의 상태를 관리하는지 결정하라

   - Flutter에서 상태를 관리하는 3가지 방법
     - 위젯이 자신의 상태를 관리
     - 부모위젯이 상태를 관리
     - 혼합하여 관리
   - 원칙
     - 해당 상태가 사용자 입력이라면, 상위위젯에서 관리하는 것이 가장 좋음
       - ex : 슬라이더 위치 혹은 체크박스의 선택과 취소
     - 해당 상태가 UI와 연관이 깊으면, 해당 위젯에서 관리
       - ex : 애니메이션
     - 잘모르겠을때는 부모 위젯이 관리하도록 함

3. StatefulWidget의 하위 클래스 및 State

   - 아래 예시는 자신이 직접 상태를 관리하는 것

   ```dart
   class MyStatefulWidget extends StatefulWidget {
     MyStatefulWidget({Key key, this.title}) : super(key: key);
     final String title;

     @override
     _MyStatefulWidgetState createState() => _MyStatefulWidgetState();
   }

   class _MyStatefulWidgetState extends State<MyStatefulWidget> {

     @override
     Widget build(BuildContext context) {
       ...
     }
   }
   ```

4. StatefulWidget을 위젯트리에 추가하기

   ```dart
   class MyStatelessWidget extends StatelessWidget {
     // This widget is the root of your application.

     @override
     Widget build(BuildContext context) {
       return MaterialApp(
         title: 'Flutter Demo',
         theme: ThemeData(
           primarySwatch: Colors.blue,
         ),
         home: MyStatefulWidget(title: 'State Change Demo'),
       );
     }
   }
   ```

   - 위와같이 Stateless 위젯 안에 Stateful 위젯을 추가하는 방식도 통함
