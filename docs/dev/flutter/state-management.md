# Flutter 狀態管理

在 Flutter 中，**狀態(State)是指它用於顯示 UI 或管理系統資源的所有物件**，狀態管理是我們組織應用程式以最有效地存取這些物件並在不同的部件之間共享它們的方式。

## 使用 StatefulWidget

要實現可管理的狀態，最簡單的方式就是使用 `StatefulWidget` ：

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(const MyApp());

class MyCounter extends StatefulWidget {
  const MyCounter({super.key});

  @override
  State<MyCounter> createState() => _MyCounterState();
}

class _MyCounterState extends State<MyCounter> {
  int count = 1;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $count'),
        TextButton(
          onPressed: () {
            setState(() {
              count++;
            });
          },
          child: const Text('Increment'),
        )
      ],
    );
  }
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // Root widget
      home: Scaffold(
        appBar: AppBar(
          title: const Text('My Home Page'),
        ),
        body: const MyCounter(),
      ),
    );
  }
}

```

這段程式碼說明了狀態管理的兩個重要概念：

1. 封裝（Encapsulation）：使用 `MyCounter` 的部件**無法存取 `count`變數，也無法存取或更改它**。
2. 物件生命週期（Object lifecycle）： `_MyCounterState` 物件及其 `count` 變數是在第一次建立 `MyCounter` 時建立的，並且一直存在，直到將其從螢幕上刪除為止。

## 在元件中共享狀態

應用程式需要儲存狀態的一些場景包括：

- 更新共享狀態並通知應用程式的其他部分
- 偵聽共享狀態的變更並在變更時重建 UI

以下為最常見的解決方案：

- 使用 **widget 函式建構子**
- 使用 **`InheritedWidget`**
- 使用**回呼**來通知父部件發生改變

### 使用 widget 函式建構子

由於物件是 passed by reference ，部件在其函式建構子中定義需要使用的物件是很常見的。傳遞給部件函式建構子的任何狀態都可以用於建立其 UI：

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(const MyApp());

class MyCounter extends StatelessWidget {
  final int count;
  const MyCounter({super.key, required this.count});

  @override
  Widget build(BuildContext context) {
    return Text('$count');
  }
}

class MyApp extends StatefulWidget {
  const MyApp({super.key});
  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  int count = 0;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // Root widget
      home: Scaffold(
        appBar: AppBar(
          title: const Text('My Home Page'),
        ),
        body: Column(
          children: [
	          // MyCounter 的文字會在 TextButton 點擊時同步更新
            MyCounter(count: count),
            TextButton(
              child: const Text('Increment'),
              onPressed: () {
                setState(() {
                  count++;
                });
              },
            ),
            MyCounter(count: count),
          ],
        ),
      ),
    );
  }
}

```

透過部件函式建構子傳遞應用程式的共享資料可以讓任何閱讀程式碼的人清楚地知道存在共享依賴。**這是一種稱為依賴注入(Dependency injection)的常見設計模式**。

### 使用 InheritedWidget

`InheritedWidget` 解決了一層一層部件向下傳遞資料過於冗長的問題，要使用 `InheritedWidget` 的話，需擴展 `InheritedWidget` 類別並使用 `dependentOnInheritedWidgetOfExactType` 實作靜態方法 `of()`。在建置方法中呼叫 `of()` 的部件會建立由 Flutter 自行管理的依賴項，當此部件使用新資料 rebuild 且 `updateShouldNotify` 傳回 `true` 時，依賴此 `InheritedWidget` 的任何 Widget 都會 rebuild。

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyState extends InheritedWidget {
  const MyState({
    super.key,
    required this.data,
    required super.child,
  });

  final String data;

  static MyState of(BuildContext context) {
    // This method looks for the nearest `MyState` widget ancestor.
    final result = context.dependOnInheritedWidgetOfExactType<MyState>();

    assert(result != null, 'No MyState found in context');

    return result!;
  }

  @override
  // This method should return true if the old widget's data is different
  // from this widget's data. If true, any widgets that depend on this widget
  // by calling `of()` will be re-built.
  bool updateShouldNotify(MyState oldWidget) => data != oldWidget.data;
}

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
	  // 這裡呼叫 of 方法來取用共享狀態
    var data = MyState.of(context).data;
    return Scaffold(
      body: Center(
        child: Text(data),
      ),
    );
  }
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: MyState(
        data: 'Hello, World!',
        child: HomeScreen(),
      ),
    );
  }
}

```

### 使用回呼(Callback)

當值發生變化時，可以透過回呼來通知其他部件，Flutter 提供了 `ValueChanged` 型別，它會定義帶有一個參數的回呼函式：

```dart
typedef ValueChanged<T> = void Function(T value);
```

透過在函式建構子中暴露 `onChanged` ，便可提供一種方法，以便在部件呼叫 `onChanged` 時做出回應。

```dart
class MyCounter extends StatefulWidget {
  const MyCounter({super.key, required this.onChanged});

  final ValueChanged<int> onChanged;

  @override
  State<MyCounter> createState() => _MyCounterState();
}
```

例如，此部件可能會處理 `onPressed` 回呼，並使用 `count` 變數的最新內部狀態呼叫 `onChanged`：

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

typedef ValueChanged<T> = void Function(T value);

class MyCounter extends StatefulWidget {
  final ValueChanged<int> onCounterChange;
  const MyCounter({Key? key, required this.onCounterChange}) : super(key: key);

  @override
  _MyCounterState createState() => _MyCounterState();
}

class _MyCounterState extends State<MyCounter> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
      widget.onCounterChange(_counter);
    });
  }

  void _decrementCounter() {
    setState(() {
      _counter--;
      widget.onCounterChange(_counter);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        Text('Counter: $_counter'),
        ElevatedButton(
          onPressed: _incrementCounter,
          child: const Text('Increment'),
        ),
        ElevatedButton(
          onPressed: _decrementCounter,
          child: const Text('Decrement'),
        ),
      ],
    );
  }
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Counter App'),
        ),
        body: Center(
          child: MyCounter(
            onCounterChange: (value) {
              print('Counter value: $value');
            },
          ),
        ),
      ),
    );
  }
}
```

## Listenables

Flutter 提供了稱為 `Listenable` 的抽象類別，可以更新一個或多個監聽器。使用 `Listenable` 的一些有用方法是：

- 使用 `ChangeNotifier` 並使用 `ListenableBuilder` 訂閱它
- 將 `ValueNotifier` 與 `ValueListenableBuilder` 結合使用

### ChangeNotifier

要使用 `ChangeNotifier` ，建立一個類別並擴產它，並在類別需要 notify 這個 listener 時呼叫 `notifyListeners` 

```dart
class CounterNotifier extends ChangeNotifier {
  int _count = 0;
  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();
  }
}
```

然後將其傳遞給 `ListenableBuilder` ，以確保每當 `ChangeNotifier` 更新其 listener 時都會 rebuild 函式建構子傳回的 subtree

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class CounterNotifier extends ChangeNotifier {
  int _count = 0;
  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();
  }
}

class MyApp extends StatelessWidget {
  MyApp({super.key});

  final CounterNotifier counterNotifier = CounterNotifier();

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Padding(
          padding: const EdgeInsets.all(32.0),
          child: Column(
            children: [
              ListenableBuilder(
                listenable: counterNotifier,
                builder: (context, child) {
                  return Text('counter: ${counterNotifier.count}');
                },
              ),
              TextButton(
                child: const Text('Increment'),
                onPressed: () {
                  counterNotifier.increment();
                },
              ),
            ],
          )),
    );
  }
}
```

### **ValueNotifier**

`ValueNotifier` 是 `ChangeNotifier` 的簡單版本，它儲存單一值。它實作了 `ValueListenable` 和 `Listenable` 介面，因此它與 `ListenableBuilder` 和 `ValueListenableBuilder` 等部件相容。要使用它，請使用初始值建立 `ValueNotifier` 的實例：

```dart
ValueNotifier<int> counterNotifier = ValueNotifier(0);
```

然後使用 `value` 欄位讀取或更新值，並通知任何偵聽器該值已變更。因為 `ValueNotifier` 擴展了 `ChangeNotifier`，所以它也是一個 `Listenable`，並且可以與 `ListenableBuilder` 一起使用。但您也可以使用 `ValueListenableBuilder`，它在 builder 回呼中提供值：

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

ValueNotifier<int> counterNotifier = ValueNotifier(0);

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Padding(
          padding: const EdgeInsets.all(32.0),
          child: Column(
            children: [
              ValueListenableBuilder(
                valueListenable: counterNotifier,
                builder: (context, value, child) {
                  return Text(
                    'Counter: $value',
                    style: const TextStyle(fontSize: 24),
                  );
                },
              ),
              ElevatedButton(
                onPressed: () {
                  counterNotifier.value++;
                },
                child: const Text('Increment'),
              ),
            ],
          )),
    );
  }
}

```

## 使用 MVVM 作為應用程式架構

### 定義 Model

Model 通常是一個類別類，它執行低階任務，例如發出 HTTP 請求、快取資料或管理系統資源（例如外掛程式）。模型通常不需要匯入 Flutter 庫。例如：

```dart
import 'package:http/http.dart';

class CounterData {
  CounterData(this.count);

  final int count;
}

class CounterModel {
  Future<CounterData> loadCountFromServer() async {
    final uri = Uri.parse('https://myfluttercounterapp.net/count');
    final response = await get(uri);

    if (response.statusCode != 200) {
      throw ('Failed to update resource');
    }

    return CounterData(int.parse(response.body));
  }

  Future<CounterData> updateCountOnServer(int newCount) async {
    // ...
  }
}

```

`CounterModel` 的唯一工作是使用 HTTP client 來 fetch 或 update `count` 。這允許在單元測試中使用 Mock 或 Fake 來實現模型，並在應用程式的低階元件和建立完整應用程式所需的高階 UI 元件之間定義清晰的界限。

### 定義 ViewModel

`ViewModel` 會將 View 綁定至 Model。它會保護 model 不直接被 View 存取，並確保資料流由 model 改變開始。資料流被 `ViewModel` 處理，並使用 `notifyListeners` 來通知 View 有異動發生。 `ViewModel` 就像餐廳中的服務生，擔任廚房(kitchen)和消費者(customers)之間的溝通。

```dart
import 'package:flutter/foundation.dart';

class CounterViewModel extends ChangeNotifier {
  final CounterModel model;
  int? count;
  String? errorMessage;
  CounterViewModel(this.model);

  Future<void> init() async {
    try {
      count = (await model.loadCountFromServer()).count;
    } catch (e) {
      errorMessage = 'Could not initialize counter';
    }
    notifyListeners();
  }

  Future<void> increment() async {
    var count = this.count;
    if (count == null) {
      throw('Not initialized');
    }
    try {
      await model.updateCountOnServer(count + 1);
      count++;
    } catch(e) {
      errorMessage = 'Count not update count';
    }
    notifyListeners();
  }
}
```

請注意，當 `ViewModel` 收到來自 Model 的錯誤時，它會儲存一條 `errorMessage`。這可以保護 View 免受未處理的運行時錯誤的影響，這些錯誤可能導致崩潰。相反，View 可以使用 `errorMessage` 欄位來顯示使用者友善的錯誤訊息。

### 定義 View

由於我們的 `ViewModel` 是一個 `ChangeNotifier`，因此當 `ViewModel` 通知其偵聽器時，任何引用它的部件都可以使用 `ListenableBuilder` 來重建其部件樹：

```dart
ListenableBuilder(
  listenable: viewModel,
  builder: (context, child) {
    return Column(
      children: [
        if (viewModel.errorMessage != null)
          Text(
            'Error: ${viewModel.errorMessage}',
            style: Theme.of(context)
                .textTheme
                .labelSmall
                ?.apply(color: Colors.red),
          ),
        Text('Count: ${viewModel.count}'),
        TextButton(
          onPressed: () {
            viewModel.increment();
          },
          child: Text('Increment'),
        ),
      ],
    );
  },
)
```

此模式允許應用程式的業務邏輯與模型層執行的 UI 邏輯和低階操作分開。