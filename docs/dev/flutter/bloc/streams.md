# 快速理解 Streams

Stream 為非同步資料的隊列。就像一個水管中流動的水，水管本身就是 Stream、水就是非同步資料。以下是一個範例：

```dart
Stream<int> countStream(int max) async* {
    for (int i = 0; i < max; i++) {
        yield i;
    }
}

Future<int> sumStream(Stream<int> stream) async {
    int sum = 0;
    await for (int value in stream) {
        sum += value;
    }
    return sum;
}

void main() async {
    /// Initialize a stream of integers 0-9
    Stream<int> stream = countStream(10);
    /// Compute the sum of the stream of integers
    int sum = await sumStream(stream);
    /// Print the sum
    print(sum); // 45
}
```

透過將函數標記為 `async*`，我們可以使用 `yield` 傳回資料流。在上面的範例中，我們傳回 `max` 參數的整數流。每次我們在 `async*` 函數中 `yield` 時，我們都會透過 `Stream` 推送該資料。

`sumStream` 等待 `Stream` 中的每個值並傳回 `Stream` 中所有整數的總和。