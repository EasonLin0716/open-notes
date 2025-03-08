# Widgets

Widgets 是 Flutter app 使用者介面的 building blocks，每個 widget 都是使用者介面中的不可變宣告。

Flutter 開發中常見的實例，例如  `MaterialApp`, `Scaffold`, `AppBar`, `Text`, `Center`, `Builder`, `Column`, `SizedBox`, 和 `ElevatedButton` 都是 widget

也有佈局用的 widget，例如 `Padding`, `Alignment`, `Row`, `Column`, 和`Grid` 

要在 Flutter 建立使用者介面，可以覆寫（override）widget 中的 `build` 方法。每個 widget 都必須包含 `build` 方法並回傳另一個 widget，例如

```dart
class PaddedText extends StatelessWidget {
  const PaddedText({super.key});

  @override
  Widget build(BuildContext context) {
    return const Padding(
      padding: EdgeInsets.all(8.0),
      child: Text('Hello, World!'),
    );
  }
}
```

widget 可以是 stateful 或 stateless 的。

沒有可變狀態的 widget 為 `StatelessWidget` 的子類別。內建的 widget 例如 `Padding` , `Text` , `Icon` 都是 stateless。

**如果 widget 的某個要素需要根據使用者互動或其他因素而改變，則該 widget 是 stateful 的**，也就會是 `StatefulWidget` 的子類別。例如：

```dart
class CounterWidget extends StatefulWidget {
  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Text('$_counter');
  }
}
```

當值改變的時候，widget 必須被重建來更新它使用者介面的一部分，此 widget 便是 `StatefulWidget` 的子類別。**`StatefulWidget` 的子類別沒有 `build` 方法，它們的使用者介面是透過 `State` 物件建構的。** 

## 建立 Stateful Widget

- 有狀態的 Widget 由兩個類別實作： `StatefulWidget` 的子類別和 `State` 的子類別。
- 狀態類別包含 Ｗidget 的可變狀態和 Ｗidget 的 `build()` 方法。
- 當 Widget 的狀態改變時，狀態物件會呼叫 `setState()`，告訴框架重新繪製 Widget。

***2025.03.28 sync to open-notes***