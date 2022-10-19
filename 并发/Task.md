# Task及方法

## FromResult

---

只能提供结果正确的同步 `Task` 对象如果要让返回的 `Task` 对象有一个其他类型的结果（例如以 `NotImplementedException` 结束的 `Task` 对象），就得自行创建使用
`TaskCompletionSource` 的辅助方法

```csharp
    static Task<T> NotImplementedAsync<T>()
    {
        var tcs = new TaskCompletionSource<T>();
        tcs.SetException(new NotImplementedException());
        return tcs.Task;
    }
```

从概念上讲，`Task.FromResult` 只不过是 `TaskCompletionSource` 的一个简化版本，它与上面
的代码非常类似

如果用 `Task.FromResult` 反复调用同一参数，则可考虑用一个实际的 task 变量。例如，可
以一次性建立一个结果为 0 的 `Task<int>` 对象，在以后的调用中就不需要创建额外的实例
了，这样可减少垃圾回收的次数：

```csharp
    private static readonly Task<int> zeroTask = Task.FromResult(0);
    static Task<int> GetValueAsync()
    {
        return zeroTask;
    }
```

## WhenAll

---

框架提供的 Task.WhenAll 方法可以实现这个功能。这个方法的输入为若干个任务，当所有
任务都完成时，返回一个完成的 Task 对象：

```csharp
Task task1 = Task.Delay(TimeSpan.FromSeconds(1));
Task task2 = Task.Delay(TimeSpan.FromSeconds(2));
Task task3 = Task.Delay(TimeSpan.FromSeconds(1));
await Task.WhenAll(task1, task2, task3);
```

如果所有任务的结果类型相同，并且全部成功地完成，则 Task.WhenAll 返回存有每个任务
执行结果的数组：

```csharp
Task task1 = Task.FromResult(3);
Task task2 = Task.FromResult(5);
Task task3 = Task.FromResult(7);
int[] results = await Task.WhenAll(task1, task2, task3);
// "results" 含有 { 3, 5, 7 }
```

如果有一个任务抛出异常，则 Task.WhenAll 会出错，并把这个异常放在返回的 Task 中。
如果多个任务抛出异常，则这些异常都会放在返回的 Task 中。但是，如果这个 Task 在被
await 调用，就只会抛出其中的一个异常。如果要得到每个异常，可以检查 Task.WhenALl
返回的 Task 的 Exception 属性

## WhenAny

---

使用 `Task.WhenAny` 方法。该方法的参数是一批任务，当其中任意一个任务完成时就会返
回。作为返回值的 `Task` 对象，就是那个完成的任务。不要觉得迷惑，这个听起来有点难，
但从代码看很容易实现：

```csharp
 // 返回第一个响应的 URL 的数据长度。
 private static async Task<int> FirstRespondingUrlAsync(string urlA, string urlB)
 {
    var httpClient = new HttpClient();
    // 并发地开始两个下载任务。
    Task<byte[]> downloadTaskA = httpClient.GetByteArrayAsync(urlA);
    Task<byte[]> downloadTaskB = httpClient.GetByteArrayAsync(urlB);
    // 等待任意一个任务完成。
    Task<byte[]> completedTask =
    await Task.WhenAny(downloadTaskA, downloadTaskB);
    // 返回从 URL 得到的数据的长度。
    byte[] data = await completedTask;
    return data.Length;
 }
```

