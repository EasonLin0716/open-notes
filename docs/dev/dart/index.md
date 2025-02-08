# 快速瀏覽 Dart 語法

印出訊息

```dart
void main() {
  print('Hello, World!');
}
```

Dart 中宣告變數的方式（不必明確指派型別）：

```dart
var name = 'Voyager I';
var year = 1977;
var antennaDiameter = 3.7;
var flybyObjects = ['Jupiter', 'Saturn', 'Uranus', 'Neptune'];
var image = {
  'tags': ['saturn'],
  'url': '//path/to/saturn.jpg'
};
```

如果要宣告常數，使用 `final` 或 `const` 。 `final` 只能被 set 一次； `const` 是編譯時常數。實例的變數只能是 `final` 。

```dart
final name = 'Bob'; // Without a type annotation
final String nickname = 'Bobby';

```

對於想要成為編譯時常數的變數使用 `const`。如果 `const` 變數在類別級別，則將其標記為 `static const`。在宣告變數的地方，將值設為編譯時常數，例如數字或字串文字、const 變數或常數算術運算的結果：

```dart
const bar = 1000000; // Unit of pressure (dynes/cm2)
const double atm = 1.01325 * bar; // Standard atmosphere
```

`const` 關鍵字不僅用於聲明常數變數。您也可以使用它來建立常數值，以及聲明建立常數值的建構子。任何變數都可以有一個常數值。

```dart
var foo = const [];
final bar = const [];
const baz = []; // Equivalent to `const []`
```

您可以從 `const` 宣告的初始化表達式中省略 `const`，就像上面的 `baz` 一樣。您可以更改非最終、非常量變數的值，即使它曾經具有 `const` 值：

```dart
foo = [1, 2, 3]; // Was const []
```

流程控制

```dart
if (year >= 2001) {
  print('21st century');
} else if (year >= 1901) {
  print('20th century');
}

for (final object in flybyObjects) {
  print(object);
}

for (int month = 1; month <= 12; month++) {
  print(month);
}

while (year < 2016) {
  year += 1;
}
```

函式宣告

```dart
int fibonacci(int n) {
  if (n == 0 || n == 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

var result = fibonacci(20);
```

如果陳述式只有一行，且想傳入匿名函式，也能使用箭頭語法

```dart
flybyObjects.where((name) => name.contains('turn')).forEach(print);
```

使用其他 lib 的 API

```dart
// Importing core libraries
import 'dart:math';

// Importing libraries from external packages
import 'package:test/test.dart';

// Importing files
import 'path/to/my_other_file.dart';
```