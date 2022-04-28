# gRPC

开发环境：macOS(Arm64), .NET6, VS for Mac 2022

## 搭建

---

### Service端

1. 首先打开VS新建一个`gRPC`项目
2. 进入`Program.cs` 会发现给的模板中有一段官方的注释  
    ```csharp
    // Additional configuration is required to successfully run gRPC on macOS.
    // For instructions on how to configure Kestrel and gRPC clients on macOS, visit https://go.microsoft.com/fwlink/?linkid=2099682
    ```  
    大致意思是如果要在macOS上运行`gPRC`的话得需要一些额外配置，所以我们点进官方文档看一眼。  

        Kestrel 不支持 macOS 和更早的 Windows 版本（如 Windows 7）上的带有 TLS 的 HTTP/2。 默认情况下，ASP.NET Core gRPC 模板和示例使用 TLS。 尝试启动 gRPC 服务器时，你将看到以下错误消息：

        无法绑定到 IPv4 环回接口上的 https://localhost:5001 ：“由于缺少 ALPN 支持，macOS 不支持使用 TLS 的 HTTP/2。
        
        若要解决此问题，请将 Kestrel 和 gRPC 客户端配置为使用不带有 TLS 的 HTTP/2。 应仅在开发过程中执行此操作。 如果不使用 TLS，将会在不加密的情况下发送 gRPC 消息。

        Kestrel 必须在 Program.cs 中配置一个不带 TLS 的 HTTP/2 终结点：

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.ConfigureKestrel(options =>
                {
                    // Setup a HTTP/2 endpoint without TLS.
                    options.ListenLocalhost(5001, o => o.Protocols = 
                        HttpProtocols.Http2);
                });
                webBuilder.UseStartup<Startup>();
            });
    ```
    
    可是官方是不是忘记了，在.NET6以后`Startup`已经与`Program.cs`合并到一起了  
    所以正确的配置应该改为如下：
    
    ```csharp
    //在Program.cs中添加一行
    builder.WebHost.ConfigureKestrel(options =>
    {
        // Setup a HTTP/2 endpoint without TLS.
        options.ListenLocalhost(5001, o => o.Protocols =
            HttpProtocols.Http2);
    });
    ```

    保存后，打开终端运行即可。

### Client端

1. 新建一个控制台应用
2. 导入NuGet
   - `Grpc.Net.Client`，其中包含 .NET Core 客户端。
   - `Google.Protobuf` 包含适用于 C# 的 Protobuf 消息。
   - `Grpc.Tools`，其中包含适用于 Protobuf 文件的 C# 工具支持。 运行时不需要工具包，因此依赖项标记为 PrivateAssets="All"。
3. 将`Service`中的`Protos`文件夹复制到`Client`下
4. 卸载`Client`（Upload Project）
5. 右键`Client`->`工具`->`编辑文件`  
   将代码中的部分修改为如下：
   
   ```csharp
    <ItemGroup>
        // 将Service改为Client
        <Protobuf Include="Protos\greet.proto" GrpcServices="Client" />
    </ItemGroup>
   ```
6. Reload Project
7. 使用以下代码更新 `gRPC`客户端 `Program.cs` 文件  
   
   ```csharp
    using Grpc.Net.Client;
    using Service;

    using (var channel = GrpcChannel.ForAddress("http://localhost:5001"))
    {
        var client = new Greeter.GreeterClient(channel);
        var reply = await client.SayHelloAsync(new HelloRequest { Name = "xx" });
        Console.WriteLine("服务端:" + reply.Message);
    }
   ```
8. 终端启动，有可能会跑权限异常，只需要在`系统偏好设置`->`安全性与隐私`->`隐私`->`完全磁盘访问权限`中把终端给勾上即可。

运行结果如下：

![pic](pic/1651162818396.jpg)