# .NET 6 vs .NET 8 vs .NET 10: Deprecated/Downgraded Features, Alternatives, and Sample Code

This guide compares **deprecated, obsoleted, or downgraded APIs and features** across .NET 6, .NET 8, and .NET 10, with before/after sample code and best practices.

| Feature/API/Method                | .NET 6 Status                | .NET 8 Status                  | .NET 10 Status                | Sample: Old Usage (.NET 6)                                 | Sample: Best Practice (.NET 8/10)                             | Best Practice Advice                                               |
|-----------------------------------|------------------------------|--------------------------------|-------------------------------|------------------------------------------------------------|---------------------------------------------------------------|-------------------------------------------------------------------|
| `BinaryFormatter`                 | Obsolete (warnings)          | Throws Exception               | Removed, new JSON clipboard APIs | See below                                              | See below                                                  | Use `System.Text.Json`, new APIs for clipboard, protobuf          |
| `WebRequest` / `WebClient`        | Obsolete (warnings)          | Throws Exception               | Removed                      | See below                                               | See below                                                   | Use `HttpClient` for all network calls                            |
| Sync I/O in ASP.NET Core          | Discouraged                  | Throws Exception by default    | Not available                 | See below                                               | See below                                                   | Use async APIs; never block on I/O                                |
| `Assembly.LoadWithPartialName`    | Obsolete (still works)       | Throws Exception               | Removed                      | See below                                               | See below                                                   | Use `Assembly.Load` with strong name                              |
| .NET Remoting APIs                | Not Supported                | Not Available                  | Not Available                 | See below                                               | See below                                                   | Use gRPC, REST, SignalR for IPC                                   |
| `Thread.Suspend/Resume/Abort`     | Obsolete                     | Throws Exception               | Removed                      | See below                                               | See below                                                   | Use `CancellationToken` and cooperative cancellation              |
| `AppDomain.CreateDomain`          | Not Supported                | Not Available                  | Not Available                 | See below                                               | See below                                                   | Use processes or AssemblyLoadContext instead                      |
| Clipboard Serialization           | N/A                          | N/A                            | New JSON clipboard APIs       | N/A                                                     | See below                                                   | Use JSON APIs for clipboard serialization [[5]](https://www.bytehide.com/blog/whats-new-in-dotnet-10)  |

---

## 1. `BinaryFormatter` Serialization

**Old (.NET 6, obsolete):**
```csharp
using System.Runtime.Serialization.Formatters.Binary;
using System.IO;

var obj = new MyClass { Name = "Hello" };
var formatter = new BinaryFormatter();
using var fs = File.OpenWrite("data.bin");
formatter.Serialize(fs, obj);
```

**.NET 8+ Best Practice:**
```csharp
using System.Text.Json;
using System.IO;

var obj = new MyClass { Name = "Hello" };
string json = JsonSerializer.Serialize(obj);
File.WriteAllText("data.json", json);
```

**.NET 10 Clipboard Example (new API):**
```csharp
// Hypothetical: Clipboard.SetDataJson, Clipboard.GetDataJson
Clipboard.SetDataJson("myObject", obj);
var restored = Clipboard.GetDataJson<MyClass>("myObject");
```

---

## 2. `WebRequest` / `WebClient`

**Old (.NET 6, obsolete):**
```csharp
using System.Net;

var request = WebRequest.Create("https://example.com");
using var response = request.GetResponse();
using var stream = response.GetResponseStream();
using var reader = new StreamReader(stream);
string content = reader.ReadToEnd();
```

**.NET 8/10 Best Practice:**
```csharp
using System.Net.Http;

using var client = new HttpClient();
string content = await client.GetStringAsync("https://example.com");
```

---

## 3. Synchronous I/O in ASP.NET Core

**Old (.NET 6, discouraged):**
```csharp
public IActionResult Post()
{
    byte[] buffer = new byte[1024];
    int read = Request.Body.Read(buffer, 0, buffer.Length); // synchronous
    // ... do something
    return Ok();
}
```

**.NET 8/10 Best Practice:**
```csharp
public async Task<IActionResult> Post()
{
    byte[] buffer = new byte[1024];
    int read = await Request.Body.ReadAsync(buffer, 0, buffer.Length); // async
    // ... do something
    return Ok();
}
```

---

## 4. `Assembly.LoadWithPartialName`

**Old (.NET 6, obsolete):**
```csharp
var assembly = Assembly.LoadWithPartialName("System.Data");
```

**.NET 8/10 Best Practice:**
```csharp
var assembly = Assembly.Load("System.Data, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089");
```

---

## 5. .NET Remoting

**Old (not available in .NET Core/6/8/10, shown for reference):**
```csharp
// Not supported in .NET 6/8/10
RemotingConfiguration.RegisterWellKnownServiceType(...);
```

**.NET 8/10 Best Practice:**
```csharp
// Use gRPC, REST API, or SignalR instead
// Example: gRPC service contract
public class MyGreeterService : Greeter.GreeterBase
{
    public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
    {
        return Task.FromResult(new HelloReply { Message = "Hello " + request.Name });
    }
}
```

---

## 6. `Thread.Suspend/Resume/Abort`

**Old (.NET 6, obsolete):**
```csharp
Thread t = new Thread(SomeWork);
t.Start();
t.Suspend(); // obsolete
t.Resume();  // obsolete
t.Abort();   // obsolete
```

**.NET 8/10 Best Practice:**
```csharp
CancellationTokenSource cts = new CancellationTokenSource();
Thread t = new Thread(() => SomeWork(cts.Token));
t.Start();
// Request cancellation (cooperatively)
cts.Cancel();
```
```csharp
void SomeWork(CancellationToken token)
{
    while (!token.IsCancellationRequested)
    {
        // do work
    }
}
```

---

## 7. `AppDomain.CreateDomain`

**Old (.NET Framework, not in .NET Core/6/8/10):**
```csharp
AppDomain domain = AppDomain.CreateDomain("NewDomain");
```

**.NET 8/10 Best Practice:**
```csharp
// Use AssemblyLoadContext for isolation
using System.Runtime.Loader;

var context = new AssemblyLoadContext("IsolatedContext", isCollectible: true);
// Load assemblies into this context as needed
```
Or, use a new process for full isolation.

---

## 8. Clipboard Serialization (New in .NET 10)

**.NET 10 Best Practice:**
```csharp
// Example: Serialize clipboard data using new JSON APIs
Clipboard.SetDataJson("myObject", obj);
var restored = Clipboard.GetDataJson<MyClass>("myObject");
```

---

## References

- [What's new in .NET 10 (Microsoft)](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/overview)
- [Breaking Changes in .NET 10: Migration Guide](https://www.mobilize.net/blog/dotnet8-to-dotnet10-migration-guide)
- [What’s New in .NET 10: Features, Comparison & Migration](https://srutatech.com/2025/04/05/whats-new-in-net-10-features-comparison-migration/)
- [What’s New in .NET 10: Everything You Need to Know](https://www.bytehide.com/blog/whats-new-in-dotnet-10)
