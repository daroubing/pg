# PG - Protobuf Generate

一个从protobuf定义文件自动生产代码的脚本

## 约定

*   目前默认消息ID为 Int16, 也就是2Bytes


## protobuf定义文件要求

在这里假设你的 定义文件 a.proto, b.proto 等等都位于 目录A中

*   定义文件中不要加 package, 生成的csharp会自动添加namespace
*   确保message, enum 的名字全局唯一
*   message 的名字 建议是首字母大写的驼峰命名法。比如 `message PlayerAction {...}`
*   目录A中需要有 `pg` 用的配置文件 `pg.conf`， 格式参考 此repo中的 pg.conf
*   目录A中需要有 `pg` 用的给消息ID定义文件 `define.ini`.

     格式为：
     
     ```
     [define]
     MessageOne = 1
     MessageTwo = 2
     ```
     
     表示每个消息所对应的消息ID
     
*   生成的文件位于 out/ 目录下的 各自语言的目录中。

     将其拷贝到你项目中即可。


### Csharp with Mono

生成的文件:

```
out/csharp
    |----ProtocolDefine.dll
    |----ProtocolSerializer.dll
    |----protobuf-net.dll
    |----Protocol
            |----Handler.cs
            |
            |----Implement
                    |----MessageOne.cs
                    |----MessageTwo.cs
                        ...
```


*   在Mono环境下 用 [protobuf-net][1] 来生成对应的csharp文件，并且将其编译为一个`ProtocolDefine.dll`文件。 

    ```
    namespace:  Protocol.Define
    class name: 就是定义中的各种 message 的名字
    ```
    
*   用预编译 `precompile` 来避免在IOS设备上的AOT问题。产生的文件为 `ProtocolSerializer.dll`

    ```
    namespace:  Protocol
    class name: Serializer
    ```
    
    **NOTE: 你并不会直接使用这个类，见下面的 `ProtocolHandler` 类提供的 `Pack/Process` 方法**
    
    
*   脚本自己也生成了一个 `Protocol` 目录。里面的 `Protocol.ProtocolHandler` 类提供了非常方便的方法来 打包/解包 数据。

*   `Implement`目录中的各种 `MessageName.cs` 文件， 其中的 `Process`方法 就是你要写代码的地方，该如何处理数据

#### 使用方法

比如有个 proto 文件定义如下:
```
message MyMessage {
    required int32 id = 1;
    required string name = 2;
}
```

然后在这个proto文件的目录中 创建 `pg.conf` 和 `define.ini`

`pg.conf` 可参考此 repo 中的 自己修改

`define.ini` 内容如下： 表示 MyMessage 这个消息的ID是10

```
[define]
MyMessage = 10
```

最后使用 命令 `pg /PATH/*.proto` 生成文件，会得到上述文件，将其拷贝至自己的项目中即可。


在自己的项目中使用

**序列化**

```csharp

// 首先实例化一个消息
var msg = new Protocol.Define.MyMessage();

// 填充数据
msg.id = 1;
msg.name = "Alex";

// 把消息ID和消息一起序列化. 
var data = Protocol.ProtocolHandler.PackWithId(msg);

// 完毕。 buffer 类型是 byte[]。 你可以将其通过socket发送出去

```

**反序列化**

```csharp
Protocol.ProtocolHandler.Process(data);

// data 是待处理数据， 类型是 byte[].

```

你要自己去实现生成的 `Protocol/Implement/MyMessage.cs` 文件中的 `Process` 方法

```csharp
namespace Protocol.Implement
{
    public static class MyMessage
    {
        public static void Process(Protocol.Define.MyMessage msg)
        {
            // Logic here
            Console.WriteLine("id = {0}, name = {1}", msg.id, msg.name)
        }
    }
}

```


就是这么简单！！！

socket数据收发可以 看 我的这个项目 [YLib][2]






### Erlang

生成的文件

```
out/erlang
    |---- gpb.hrl
    |---- protocol.hrl
    |---- protocol.erl
    |---- protocol_handler.erl
    |---- protocol_implement.erl
```

将其拷贝到你的项目中

#### 使用方法

首先还是和上面一样，用 `pg` 生成代码

如果你的`pg.conf` 中 没有设置 `msg_prefix` 和 `msg_suffix`， 那么 `MyMessage` 回生成一个 `#'MyMessage'{id, name}` 的 record

在代码中需要 `-include("protocol.hrl")`，


**序列化**

```erlang
Msg = #'MyMessage'{id = 1, name = <<"Tom">>},
Data = protocol_handler:pack_with_id(Msg),

%% 完毕，直接用gen_tcp:send() 将Data发送出去
```

**反序列化**
```erlang
NewState = protocol_handler:process(Data, State),
%% 完毕， 因为pg生成的代码是嵌入基于OTP设计的项目中使用的。所以这里有 State 这个参数
%% 具体处理逻辑需要到 protocol_implement.erl 中去实现。
```



[1]: https://code.google.com/p/protobuf-net/
[2]: https://github.com/yueyoum/YLib
