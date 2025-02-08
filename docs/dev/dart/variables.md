# Dart 中 late, final 與 const 的差異與用法

### **Late variables**

`late` 修飾符有兩種運用場景：

- 宣告一個在宣告後才給予初始值的 non-nullable 變數
- 延遲初始化變數

正常情況下 Dart 控制流程分析可以偵測到 non-nullable 變數在使用之前是否被設定為非 null，但可能會有不符預期的時候。兩種常見的情況是頂層變數和實例變數：Dart 通常無法確定它們是否已 set。

如果你確定變數在使用前已經設定，但 Dart 控制流程分析沒有發現，則可以透過將變數標記為 `late` 來修正錯誤：

```dart
late String description;

void main() {
  description = 'Feijoada!';
  print(description);
}
```

當您將變數標記為 `late` 但在其宣告時對其進行初始化時，**初始化程序將在第一次使用該變數時運行**。這種惰性初始化在以下幾種情況下非常方便：

- 該變數可能不需要，並且初始化它的成本很高。
- 情境是初始化一個實例變數，並且它的初始化程序需要存取 `this`。

例如：

```dart
late String temperature = readThermometer();
```

如果 `temperature` 不曾被使用過， `readThermometer` 就不會被呼叫。

### Final 及 Const

如果在宣告後不會改變變數的值，應使用 `final` 或 `const` 。 `final` 只能設定一次； `const` 是編譯時常數。 （ `const` 為隱式的 `final` 。）

> 注意：[實例變數](https://dart.dev/language/classes#instance-variables) 僅能設定為 `final` 。
> 

`final` 範例如下：

```dart
final name = 'Bob'; // Without a type annotation
final String nickname = 'Bobby';
```

舉例來說， `final` 的 `List` 可以被更改：

```dart
final list = [1, 2, 3];
list.add(4); // ✅ `final` 只限制變數的指向，不限制內容變化
```

`const` 不可再做任何異動，相同值的 `const` 會指向同一個位址，也就是說可以是相等的

```dart
// const list = [1, 2, 3];
// list.add(4); // ❌ 錯誤：const 變數的內容完全不可變

const arr1 = [1, 2, 3];
const arr2 = [1, 2, 3];
print(identical(arr1, arr2)); // ✅ true，兩個 const 變數指向同一個物件
```

在類別中， `const` 只能用來建立靜態的屬性、並且不能用於宣告實例變數：

```dart
class Example {
  final int x; // 可在函式建構子初始化
  static const int y = 10; // 必須是靜態的

  Example(this.x);
}

void main() {
  final obj1 = Example(5);
  final obj2 = Example(5);
  print(identical(obj1, obj2)); // ❌ false，不同的物件

  const obj3 = Example(5); // ❌ 錯誤，const 不能用於類的實例變數
}
```

### **區別**

| 特性 | `final` | `const` |
| --- | --- | --- |
| 可否重新賦值 | ❌ 不能 | ❌ 不能 |
| 何時確定值 | **運行時** | **編譯時** |
| 內容可變性 | ✅（如果是 List，可以修改內容） | ❌（完全不可變） |
| 是否能在類的實例變數使用 | ✅ 可以 | ❌ 不能（但 `static const` 可以） |
| 是否有物件重用 | ❌ 不會重用 | ✅ 相同值的 `const` 變數會指向同一個實例 |

**2025/02/08 synced to open-notes**