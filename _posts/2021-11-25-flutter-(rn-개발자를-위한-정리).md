---
title: flutter (rn 개발자를 위한 정리 1)
author: 신성일
date: 2021-11-25 21:00:00 +0900
categories: [study, flutter]
tags: [flutter]
---

## 진입점

```dart
main() {
    // 항상 최상단 앱의 진입점 main() 이 있어야함.
}
```



## 콘솔

```dart
print("hello world")
```



## 변수

- dart 는 타입검사를 하는 언어
- 정적 타입 검사와 런타임 타입검사를 동시에 사용하며, 변수의 값이 변수의 정적타입과 항상 일치하는지 검사함.
- 타입 추론을 하기에 일부 타입표기는 생략가능

```dart
String name = 'dart';	// 명시적 타입 선언
var othername = 'dart';	//타입 추론
```

- 초기화하지 않은 변수는 null값을 가짐.

```dart
var name;		// null
int x;		// null
```

- 좀 더 자세한 [내용](https://dart.dev/guides/language/language-tour#variables)

  

## null, 0 체크

- dart에서는 bool 값 true만 true로 취급한다.
- 따라서 null, 0 값은 그 값으로 대응 되는지 확인해야함.

```dart
var myNull = null;
if (myNull == null) {
    print("is null");
}
var zero = 0;
if(zero == 0) {
    print("is 0")
}
```



## 함수

- 자바스크립트하고는 선언에서 차이가 남.

```dart
fn() {
    return true;
}

bool fn() {
    return true;
}
```

- 한줄 함수일때는 화살표함수 사용가능

```dart
bool fn() => true;
```

- 자세한 [내용](https://dart.dev/guides/language/language-tour#functions)



## 비동기 프로그래밍

- dart에서는 [Future](https://dart.dev/codelabs/async-await) 객체를 사용하여 비동기를 지원함

  ```dart
  import 'dart:convert';
  import 'package:http/http.dart' as http;
  
  class Example {
    Future<String> _getIPAddress() {
      final url = 'https://httpbin.org/ip';
      return http.get(url).then((response) {
        String ip = jsonDecode(response.body)['origin'];
        return ip;
      });
    }
  }
  
  main() {
    final example = new Example();
    example
        ._getIPAddress()
        .then((ip) => print(ip))
        .catchError((error) => print(error));
  }
  ```

- 또 async, await로 비동기를 처리할 수 있음.

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class Example {
  Future<String> _getIPAddress() async {
    final url = 'https://httpbin.org/ip';
    final response = await http.get(url);
    String ip = jsonDecode(response.body)['origin'];
    return ip;
  }
}

main() async {
  final example = new Example();
  try {
    final ip = await example._getIPAddress();
    print(ip);
  } catch (error) {
    print(error);
  }
}
```