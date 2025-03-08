# Cubit 是什麼？


`Cubit` 是一個擴充自 `BlocBase` 的類別，並可以被擴展來管理任何類型的 `State`。

![1.png](/img/flutter/bloc/cubit/1.png)

`Cubit` 有可被外部呼叫的函式，用來讓它們觸發狀態變更。 `State` 為 `Cubit` 的輸出，代表應用程式狀態的一部分。 UI 元件可以收到 `State` 通知，並根據目前狀態重繪自身的部分內容。

建立一個 `Cubit` ：

```php
class CounterCubit extends Cubit<int> {
  CounterCubit(int initialState) : super(initialState);
  // 也可以寫成 CounterCubit(super.initialState);
}
```

在建立 `Cubit` 時，我們需要定義 `Cubit` 將管理的狀態類型。創建 `Cubit` 時我們需要做的第二件事是指定初始狀態。我們可以透過使用初始狀態的值呼叫 `super` 來做到這一點。

有上述例子便可建立實例。例如：

```php
final cubitA = CounterCubit(0); // state starts at 0
final cubitB = CounterCubit(10); // state starts at 10
```

### 使用 emit 來更改狀態

`Cubit` 可使用 `emit` 來產出一個新的狀態：

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
}
```

當呼叫 `increment` 時，我們可以透過狀態獲取器存取 `Cubit` 的目前狀態，並透過將目前狀態加 1 來發出一個新狀態。使用範例：

```dart
void main() {
  final cubit = CounterCubit();
  print(cubit.state); // 0
  cubit.increment();
  print(cubit.state); // 1
  cubit.close();
}
```

`close` 會關閉它的內部狀態流。

`Cubit` 會暴露一個 `Stream` ，允許我們接受即時狀態更新：

```dart
Future<void> main() async {
  final cubit = CounterCubit();
  final subscription = cubit.stream.listen(print); // 1
  cubit.increment();
  await Future.delayed(Duration.zero);
  await subscription.cancel();
  await cubit.close();
}
```

在上面的程式碼片段中，我們訂閱了 `CounterCubit` 並在每次狀態變更時呼叫 `print` 。然後我們呼叫將發出新狀態的 `increment` 函式。最後，當我們不想再接收更新並關閉 `Cubit` 時，我們會呼叫 `subscription.cancel()`。

### 觀察一個 Cubit

當 `Cubit` `emit` 了一個新狀態， `Change` 會觸發，我們可以透過重寫(override) `onChange` 來觀察給定 `Cubit` 的所有變化。

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);

  @override
  void onChange(Change<int> change) {
    super.onChange(change);
    print(change);
  }
}
```

例如下方例子

```dart
import 'package:flutter/material.dart';
import 'package:bloc/bloc.dart';

void main() => runApp(const MyApp());

class CounterCubit extends Cubit<int> {
  CounterCubit(super.initialState);
  void increment() => emit(state + 1);

  @override
  void onChange(Change<int> change) {
    super.onChange(change);
    print(change);
  }
}

final cubitA = CounterCubit(0); // state starts at 0
final cubitB = CounterCubit(10); // state starts at 10

void incrementCubitA() {
  cubitA.increment();
}

void closeCubitA() {
  cubitA.close();
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text('Hello World'),
              ElevatedButton(
                onPressed: incrementCubitA,
                child: Text('incrementCubitA'),
              ),
              ElevatedButton(
                onPressed: closeCubitA,
                child: Text('closeCubitA'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

```

每次 `incrementCubitA` 被點擊時，終端機會印出 `Change { currentState: n, nextState: n + 1 }` ；點擊 `closeCubitA` 後如果再次點擊 `incrementCubitA` 就會噴錯。

## **BlocObserver**

使用 bloc 的好處是我們可以在一個地方存取所有變更。在較大的應用程式中，有許多 `Cubit` 來管理應用程式狀態的不同部分是相當常見的。如果我們希望能夠做一些事情來回應所有 `Changes` ，我們可以簡單地建立我們自己的 `BlocObserver`。

```dart
class SimpleBlocObserver extends BlocObserver {
  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);
    print('${bloc.runtimeType} $change');
  }
}
```

例如：

```dart
import 'package:flutter/material.dart';
import 'package:bloc/bloc.dart';

void main() {
  Bloc.observer = CounterBlocObserver();
  runApp(const MyApp());
}

class CounterBlocObserver extends BlocObserver {
  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);
    print('${bloc.runtimeType} $change');
  }
}

class CounterCubit extends Cubit<int> {
  CounterCubit(super.initialState);
  void increment() => emit(state + 1);

  @override
  void onChange(Change<int> change) {
    super.onChange(change);
    print(change);
  }
}

final cubitA = CounterCubit(0);

void incrementCubitA() {
  cubitA.increment();
}

void closeCubitA() {
  cubitA.close();
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text('Hello World'),
              ElevatedButton(
                onPressed: incrementCubitA,
                child: Text('incrementCubitA'),
              ),
              ElevatedButton(
                onPressed: closeCubitA,
                child: Text('closeCubitA'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

```

這個 observer 會觀察所有 `Cubit` 的異動。

### Error Handling

`Cubit` 有 `addError` 方法，可以用來表明發生問題：

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit(super.initialState);
  void increment() => emit(state + 1);

  void triggerError() =>
      addError(Exception('increment error!'), StackTrace.current);

  @override
  void onChange(Change<int> change) {
    super.onChange(change);
    print(change);
  }

  @override
  void onError(Object error, StackTrace stackTrace) {
    print('$error, $stackTrace');
    super.onError(error, stackTrace);
  }
}
```

Bloc Ovserver 可以接收錯誤

```dart
class CounterBlocObserver extends BlocObserver {
  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);
    print('${bloc.runtimeType} $change');
  }

  @override
  void onError(BlocBase bloc, Object error, StackTrace stackTrace) {
    print('${bloc.runtimeType} $error $stackTrace');
    super.onError(bloc, error, stackTrace);
  }
}
```