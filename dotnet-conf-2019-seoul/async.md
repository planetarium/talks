---
marp: true
html: true
---

# .NET에서 비동기 프로그래밍 배우기
문성원 at Planetarium
@longfin / swen@planetariumhq.com

---

![image](https://user-images.githubusercontent.com/128436/66150659-9d291c00-e650-11e9-930c-6ebcc0021d1c.png)

---

![image](https://user-images.githubusercontent.com/128436/66151206-c8f8d180-e651-11e9-975f-02e6f14908b3.png)

---

# GitHub 밖에선 이렇습니다. 😅

- PHP로 프로그래밍을 처음 배워서
- Java로 웹 개발을 이어서 하다
- 스타트업에서 Python/TypeScript로 이것저것 만지다가
- 지금은 C#으로 블록체인을 만들고 있습니다. 

---

![image](https://user-images.githubusercontent.com/128436/66222903-2e5dc880-e70d-11e9-9a52-e3233276e5f0.png)

---

![image](https://user-images.githubusercontent.com/128436/66223007-66650b80-e70d-11e9-92ea-0533b72557a7.png)

---

# 그래서 오늘 이야기 할 것들

- 배경
- 배운 것들 & 해 본 것들
- 유용한 라이브러리들
- Q&A

---

# 배경

---

![width: 800px](https://user-images.githubusercontent.com/128436/65844635-0e777f00-e372-11e9-862b-37e4a9e23875.png)

---

![width: 800px](https://user-images.githubusercontent.com/128436/65844718-4c74a300-e372-11e9-92b3-a30ef9f05bf7.png)

---

Unity에서 쓸 수 있는 **블록체인 코어** 라이브러리를 만듭니다.

---

# 블록체인의 구성 요소

- 저장소 (Storage)
- 합의 방식 (Consensus)
- 네트워크 (Network)

---

# 네트워크 없이 만들었던 PoC

- 저장소: 파일 기반
- 합의 방식: Hashcash PoW(Proof of Work)
- 네트워크: 드롭박스 공유 기능을 통해 파일을 통쨰로 공유

---

![image](https://user-images.githubusercontent.com/128436/66223967-bd6be000-e70f-11e9-8e73-011df28a7736.png)

---

> 블록체인은 이제 대강 알 것 같은데, 슬슬 네트워크 만들어야 하지 않나요?

---

# P2P 블록체인 네트워크?

- 서버가 없다 ≠ Listen이 없다.
- 의외로 TCP 서버와 크게 다를 것도 없긴 한데, 
  - 웹서버도 직접 짜본 적이 없더라고요. 😅
- 피어에 연결하고 있는 중에도 다른 피어로부터 연결을 처리해야 합니다.

---

![image](https://user-images.githubusercontent.com/128436/65845434-3a483400-e375-11e9-8d9a-3e6f0d1915a6.png)

---

# C#/.NET에서 지원하는 비동기 프로그래밍 패턴들

- Event-based Asynchronous Pattern (EAP)
- Asynchronous Programming Pattern (APM)
- **Task-based Asynchronous Pattern (TAP)**
  - MS에서 권장하는 방법이라니 이걸로 해보기로

---

# 첫 인상

```csharp
var httpResult = await client.GetAsync("https://google.com");
```

- 편해보인다. & 있어보인다.
- 다른 언어(e.g. Python, TypeScript)랑 크게 다르지 않네?
  - 사실 이 쪽이 원조라고 합니다. 👵🏼
- 근데 저러면 매번 스레드가 생겨서 기다리는 건가?
  - 다행히 그렇지는 않았습니다. 😅

---

# 그러던 와중에…

> h**g: ZeroMQ 란걸 쓰면 소켓 프로그래밍을 안전하게 할 수 있다던데요.

---

# ZeroMQ

- https://zeromq.org/
- POSIX 소켓과 굉장히 비슷한 인터페이스를 제공하지만 사실은 준 프레임워크
- 다양한 편의 기능 제공
  - 다양한 비동기 프로그래밍 패턴 제공
  - 길이 제한이 없는 멀티 파트 메시지
  - Listen / Connect 의 순서 제약이 없음
  - 자동 재연결 처리

---

# 사족 👀

- 알아보니 nanomsg라는 게 있었고…
- 또 다시 알아보니 nng라는 게 있었고…
- 이걸 C#/.NET 어떻게 쓰나 보니 nng.NET 이라는 게 있었는데…
- 기껏 다 만들었더니 Windows 에서만 겨우 겨우 돌아갔습니다.

→ 이식성이 좋은 라이브러리를 찾아봅시다.

---

# NetMQ

- https://github.com/zeromq/netmq
- C#/.NET에서 바로 쓸 수 있는 구현체
- 바인딩이 아닌 순수 C#으로 포팅 되었고
- 나름대로 C# 스타일로 작성 되어 있습니다.

---

![width:800px](https://user-images.githubusercontent.com/128436/65833418-2cf65f80-e30b-11e9-8abd-a47a9dfe5b68.png)

---

# 배운 것들 & 해 본 것들

---

# `Swarm<T>`

- 네트워크 파사드 (Facade)
- 서버 / 클라이언트 역할을 동시에 수행...
  - 한다곤 하지만 TCP 서버에 가깝습니다.
- 메시지 송수신 이외에 주기적으로 수행하는 특수한 작업들도 있습니다.
  - 트랜잭션 전파
  - 라우팅 테이블 갱신

→ 오늘 자세히 다루진 않습니다. 😅

---

# `Task.WhenAll()`은 이럴 때 많이 쓰시죠?

```csharp
var tasks = new Task[] 
{
    GetAsync("https://github.com/planetarium/libplanet"),
    GetAsync("https://github.com/planetarium/libplanet-explorer"),
    GetAsync("https://github.com/planetarium/libplanet-explorer-frontend")
};

await Task.WhenAll(tasks);
```

---

# 이런 일에도 쓸 수 있습니다.

```csharp
var tasks = new Task[]
{
    ReceiveMessage(),
    BroadcastTxs(),
    RefreshTable(),
};

await Task.WhenAll(tasks);
```

---

# 예외 처리를 해볼까요. 💥

```csharp
var tasks = new Task[]
{
    ReceiveMessage(),
    BroadcastTxs(),
    RefreshTable(),
};

try 
{
    await Task.WhenAll(tasks);
}
catch (RefreshTableException)
{
    RebuildTable();
}
```

하지만 `RefreshTableException`은 발생하지 않습니다.

---

# 📄를 봅시다.

> Creates a task that will complete when **all** of the `Task` objects in an enumerable collection have completed.

https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenall?view=netstandard-2.0

---

# `Task.WhenAny()`를 써보죠.

```csharp
var tasks = new Task[]
{
    ReceiveMessage(),
    BroadcastTxs(),
    RefreshTable(),
};

try 
{
    await Task.WhenAny(tasks);
}
catch (RefreshTableException)
{
    RebuildTable();
}
```

하지만 여전히 `RefreshTableException`은 발생하지 않습니다.

---

# 다시 📄를 봅시다.

> Creates a task that will complete when any of the supplied tasks have completed.

https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenany?view=netstandard-2.0

별 문제 없는 것 같은데요?

---

# 다시 📄를 (끝까지) 봅시다.

> **Returns**
> `Task<Task>`
> A task that represents **the completion of one of the supplied tasks**. The return task's Result is the task that completed.

https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenany?view=netstandard-2.0

완료된 `Task` 그대로 반환하기 때문에 아무런 예외가 발생하지 않습니다. 

---

# 그럼 어떻게 하죠?

```csharp
var tasks = new Task[]
{
    ReceiveMessage(),
    BroadcastTxs(),
    RefreshTable(),
};

try 
{
    await await Task.WhenAny(tasks);
}
catch (RefreshTableException)
{
    RebuildTable();
}
```

---

# Dispose() 에선 await 못하나요?

```csharp
class Swarm<T> : IDisposable 
{
    public void Dispose() 
    {
        await SayGoodByeAsync(); // CS4033
    }
}
```

---

# `.Wait()` 라는게 있다던데…

```csharp
class Swarm<T> : IDisposable 
{
    public void Dispose() 
    {
        SayGoodByeAsync().Wait();
    }
}
```

에러는 없어졌네요. 그치만…

---

# `.Wait()` 라는게 있다던데…

```csharp
class Swarm<T> : IDisposable 
{
    public void Dispose() 
    {
        SayGoodByeAsync().Wait();
    }
}
```

`Swarm<T>.Dispose()` 가 `SynchronizationContext`를 갖는 스레드에서 실행하면 데드락이 발생합니다.

---

# IAsyncDisposable

- https://docs.microsoft.com/ko-kr/dotnet/api/system.iasyncdisposable?view=netcore-3.0
- `public System.Threading.Tasks.ValueTask DisposeAsync();`
- 하지만 .netstandard 2.0 (& Unity)에서는 쓸 수가 없네요. 😰

---

# 적당히 타협하기

```csharp
class Swarm<T>
{
    public Task StopAsync() 
    {
        await SayGoodByeAsync();
    }
}
```

- Libplanet(라이브러리)에서는 `Dispose()` 대신 `async StopAsync()`만을 제공하고
- 게임에서는 일정 시간 동안만 이를 기다리고 종료하게끔 작성합니다.

---

# Unity에서 잘 돌아갈까요?

- Unity에서 사용되는 C#/.NET 런타임은 Mono
  - 정규 버전이 아닌 Bleeding Edge
- NetMQ는 .NET Framework와 .NET Core를 모두 지원하긴 하니… 괜찮지 않을까요?

---

![image](https://user-images.githubusercontent.com/128436/66052168-1c442480-e56b-11e9-86a1-8c422f55f2d1.png)

---

![width:800px](https://user-images.githubusercontent.com/128436/66052343-6decaf00-e56b-11e9-86fd-1c096e54e0dd.png)

https://github.com/zeromq/netmq/issues/631

---

# 범인은 AsyncIO 🕵️‍♂️

- https://github.com/somdoron/AsyncIO
- NetMQ에서 비동기 소켓이 필요할땐 .NET 소켓 대신 이걸 사용하는데…
- 실행하는 플랫폼에 따라 IOCP / .NET Completion Port를 구분합니다.
- Unity + Windows에선 IOCP가 제대로 동작하지 않습니다. 😥


---
# May the `Force` be with you

```
        static Swarm()
        {
            if (!(Type.GetType("Mono.Runtime") is null))
            {
                ForceDotNet.Force();
            }
        }
```

https://github.com/planetarium/libplanet/blob/master/Libplanet/Net/Swarm.cs#L76-L82

---

# 동기 메소드를 비동기로 만들기

- NetMQ(<4.0.0.239-pre)의 API는 모두 *동기* 메소드
- 아무리 겉의 인터페이스를 TAP로 작성해도 소켓은 블록될 수 밖에 없는 구조

---

```csharp
public async SendMessage(Peer peer)
{
    Dealer socket = CreateSocket(peer);
    socket.Connect(); // Blocking!
    socket.Send("hello"); // and, Blocking!
    socket.Receive();
}
```

---

```csharp
    internal static class NetMQSocketExtension
    {
        public static async Task SendFrameAsync(
            this IOutgoingSocket socket,
            byte[] data,
            TimeSpan? timeout = null,
            TimeSpan? delay = null,
            CancellationToken cancellationToken = default(CancellationToken))
        {
            TimeSpan delayNotNull = delay ?? TimeSpan.FromMilliseconds(100);
            TimeSpan elapsed = TimeSpan.Zero;

            while (!socket.TrySendFrame(data))
            {
                await Task.Delay(delayNotNull, cancellationToken);

                elapsed += delayNotNull;
                if (elapsed > timeout)
                {
                    throw new TimeoutException(
                        "The operation exceeded the specified time: " +
                        $"{timeout}."
                    );
                }
            }
        }
    }
```

---

# 근본적인 문제가 남아 있습니다.

- ZeroMQ / NetMQ의 모든 소켓은 **스레드-안전(Thread-safe)하지 않습니다.**
  - TMI) 요건 버그가 아니라 설계 철학에 해당하는 부분이라고 하네요.
- 실행 문맥이 보존되는 환경이라면 괜찮지만, 그렇지 않은 환경에서는 오작동의 원인이 될 수 있습니다.

---

![image](https://user-images.githubusercontent.com/128436/66147690-50dadd80-e64a-11e9-9a36-33a39cff3d65.png)

---

![image](https://user-images.githubusercontent.com/128436/66147798-85e73000-e64a-11e9-88b0-03c46e3ced94.png)

---

# 아이디어는 이렇습니다.

- `async`/`await` 를 사용하는 환경(`Task`)를 구분하는 실행 단위(`NetMQRuntime`)을 만들고
- `NetMQPoller`로 단일 스레드를 붙여 둔 다음
- `SynchronizationContext.SetSynchronizationContext()`를 통해 실행 단위 안에서 동기화 문맥을 고정합니다.

---

![image](https://user-images.githubusercontent.com/128436/66148161-40773280-e64b-11e9-9511-e9a0350932be.png)

https://github.com/zeromq/netmq/pull/802

---

```csharp
async Task ServerAsync()
{
    using (var server = new RouterSocket("inproc://async"))
    {
        for (int i = 0; i < 1000; i++)
        {
            var (routingKey, more) = await server.ReceiveRoutingKeyAsync();
            var (message, _) = await server.ReceiveFrameStringAsync();

            // TODO: process message

            await Task.Delay(100);
            server.SendMoreFrame(routingKey);
            server.SendFrame("Welcome");
        }
    }
}
```

---

```csharp
async Task ClientAsync()
{
    using (var client = new DealerSocket("inproc://async"))
    {
        for (int i = 0; i < 1000; i++)
        {
            client.SendFrame("Hello");
            var (message, more) = await client.ReceiveFrameStringAsync();

            // TODO: process reply

            await Task.Delay(100);
        }
    }
}

static void Main(string[] args)
{
    using (var runtime = new NetMQRuntime())
    {
        runtime.Run(ServerAsync(), ClientAsync());
    }
}
```

---

# 그런데 빠진게 있더라고요.

- 추가된 비동기 메소드들이 취소 토큰(`CancellationToken`)을 받지 않았습니다. 😨
  - 덕택에 타임 아웃도 구현할 수 없었죠.
- 저희가 애용하던 멀티 파트 메세지 지원도 없었습니다. 😰
- 덤으로 테스트도 깨지고 있었습니다. 😭

---

# 제멋대로 오픈 소스 철학

- 상용 코드의 퀄리티 ≥ 오픈 소스 코드의 퀄리티
- **혼자 구현한 코드의 퀄리티 ＜ 같이 구현한 코드의 퀄리티**

---

# 그런고로 직접 고쳐보기로 했습니다.

```csharp
        public static async Task<NetMQMessage> ReceiveMultipartMessageAsync(
            [NotNull] this NetMQSocket socket, 
            int expectedFrameCount = 4,
            CancellationToken cancellationToken = default(CancellationToken))
        {
            var message = new NetMQMessage(expectedFrameCount);

            while (true)
            {
                (byte[] bytes, bool more) = await socket.ReceiveFrameBytesAsync(cancellationToken);
                message.Append(bytes);

                if (!more)
                {
                    break;
                }
            }

            return message;
        }
```

---

```csharp
        public static Task<(byte[], bool)> ReceiveFrameBytesAsync(
            [NotNull] this NetMQSocket socket
        )
        {
            if (NetMQRuntime.Current == null)
                throw new InvalidOperationException("NetMQRuntime must be created before calling async functions");
            socket.AttachToRuntime();
            var msg = new Msg();
            msg.InitEmpty();
            if (socket.TryReceive(ref msg, TimeSpan.Zero))
            {
                var data = msg.CloneData();
                bool more = msg.HasMore;
                msg.Close();
                return Task.FromResult((data, more));
            }

            TaskCompletionSource<(byte[], bool)> source = new TaskCompletionSource<(byte[], bool)>();
            
            void Listener(object sender, NetMQSocketEventArgs args)
            {
                if (socket.TryReceive(ref msg, TimeSpan.Zero))
                {
                    var data = msg.CloneData();
                    bool more = msg.HasMore;
                    msg.Close();
                    socket.ReceiveReady -=  Listener;
                    source.SetResult((data, more));
                }
            }
            socket.ReceiveReady += Listener;
            return source.Task;
        }
```

---

```csharp
        public static Task<(byte[], bool)> ReceiveFrameBytesAsync(
            [NotNull] this NetMQSocket socket, 
            CancellationToken cancellationToken = default(CancellationToken)
        )
        {
            // ...

            TaskCompletionSource<(byte[], bool)> source = new TaskCompletionSource<(byte[], bool)>();
            cancellationToken.Register(() => source.SetCanceled());
            
            // ...
            return source.Task;
        }
```

---

# 다행히 업스트림에도 머지되었습니다. 😉

![width:720px](https://user-images.githubusercontent.com/128436/66149658-841f6b80-e64e-11e9-8d09-023a17835474.png)

---

# 유용한 라이브러리들

---

# AsyncEx

- https://github.com/StephenCleary/AsyncEx
- .NET에서 제공하는 기능/API의 비동기 API 
  - `lock` → `AsyncLock`
  - `BlockingCollection<T>` → `AsyncCollection<T>`
  - `AutoResetEvent` → `AsyncAutoResetEvent`
- 여러가지 유틸리티
  - `AsyncContext`
  - `AsyncLazy`
  - `CancellationTokenHelpers`

---

# `AsyncLock`

```csharp
var _mutex = new object();
lock (_mutex)
{
    await RunAsync(); // CS1996 Cannot await in the body of a lock statement
}
```

```csharp
var _mutex = new AsyncLock();
using (await _mutex.LockAsync()) 
{
    await RunAsync();
}
```

---

# `AsyncCollection<T>`

```csharp
var requests = new AsyncCollection<Message>();

// Add
await requests.AddAsync(new Message("Hello"));

// receive
while (true)
{
    var request = await requests.TakeAsync(cancellationToken);
}
```

---

# `AsyncAutoResetEvent`

```csharp
class Swarm
{
    public BlockChain BlockChain { get; }
    public AsyncAutoResetEvent BlockReceived { get; }

    private void ReceiveBlock() 
    {
        // ...
        BlockReceived.Set();
    }
}

// Unittest
[Fact]
public async Task BlockReceiving() 
{
    var swarm = new Swarm();
    await swarm.RunAsync();

    // ...

    await swarm.BlockReceived;
    Assert.Contains(swarm.BlockChain, block);
}
```

---

# AsyncEnumerable

- https://github.com/Dasync/AsyncEnumerable
- C# 8.0 에 추가되는 Async Streams등을 미리 써 볼 수 있는 라이브러리

---

# Streams

```csharp
// Producer
public IEnumerable<byte[]> GetStreams()
{
    foreach (byte[] bytes in GetData()) 
    {
        yield return bytes;
    }
}

// Consumer
foreach (byte[] chunk in GetStreams())
{
    Console.WriteLine($"Received data size = {chunk.Count}");
}
```

---

# Async Streams

```csharp
// Producer
public async IAsyncEnumerable<byte[]> GetStreamsAsync()
{
    foreach (byte[] bytes in await GetDataAsync()) 
    {
        yield return bytes;
    }
}

// Consumer
await foreach (var chunk in GetStreamsAsync())
{
    Console.WriteLine($"Received data size = {chunk.Count}");
}
```

---

# Async Streams (`AsyncEnumerable`)

```csharp
using Dasync.Collections;

// Producer
public IAsyncEnumerable<byte[]> GetStreamsAsync() 
{
    return new AsyncEnumerable(async yield =>
    {
        foreach (byte[] bytes in await GetDataAsync())
        {
            await yield.ReturnAsync(bytes);
        }
    });
}

// Consumer
await asyncStreams.ForEachAsync(chunk => 
{
    Console.WriteLine($"Received data count = {chunk.Count}");
});

```

---

# `ParallelForeachAsync()`

```csharp
await uris.ParallelForEachAsync(
    async uri =>
    {
        var str = await httpClient.GetStringAsync(
            uri, 
            cancellationToken
        );
        result.Add(str);
    },
    maxDegreeOfParallelism: 5,
    cancellationToken: cancellationToken
);
```

---

# 함께 하실 분을 찾고 있습니다.

<img src="https://user-images.githubusercontent.com/128436/66122529-e3fc1f00-e61a-11e9-9559-4dcbc45a72e4.png" style="width:400px; display: block; margin: 0 auto 0 auto;" />

http://bit.ly/plnt-hire-2019

---

# Q&A
