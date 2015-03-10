# PG - Protobuf Generate

一个从protobuf定义文件自动生产代码的脚本

## protobuf定义文件要求

在这里假设你的 定义文件 a.proto, b.proto 等等都位于 目录A中

*   定义文件中不要加 package, 生成的csharp会自动添加namespace
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

在Mono环境下 用 [protobuf-net][1] 来生成对应的csharp文件，
并且用了预编译 `precompile` 来避免在IOS设备上的问题。

脚本自己也生成了一个 `Protocol` 目录。
里面的 `ProtocolHandler` 类提供了非常方便的方法来 打包/解包 数据。










### Erlang


[1]: https://code.google.com/p/protobuf-net/
