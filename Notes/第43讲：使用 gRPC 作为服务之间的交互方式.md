<p data-nodeid="19691" class="">在本专栏介绍微服务架构时，我们提到了外部和内部 API 及其区别，示例应用中的微服务采用 API 优先的设计方式，并基于 OpenAPI 规范来创建 REST API。除了 REST API 之外，另外一种常见的开放 API 的格式是 gRPC，本课时将对 gRPC 进行介绍。</p>
<h3 data-nodeid="19692">gRPC 介绍</h3>
<p data-nodeid="19693">在实现 API 的方式中，REST 和 gRPC 经常会被拿来进行比较，这两种方式各有长处和短处。REST 的优势在于简单易用，只需要使用 curl 这样的工具就可以与 API 交互；REST 一般使用 JSON 或 XML 这样的纯文本格式作为表达形式，使得开发和调试变得很容易。</p>
<p data-nodeid="19694">gRPC 本质上是一种远程过程调用，默认使用 Protocol Buffers 作为传输格式，传输协议为 HTTP/2。Protocol Buffers 作为一种二进制格式，可以充分的节省传输带宽，但是相应的开发和调试会变得困难，需要首先对消息进行解码之后，才能得到原始的消息内容。与 REST 相比，gRPC 还可以充分利用 HTTP/2 的多路复用功能来提高性能。gRPC 在云原生中应用广泛，其本身也是 CNCF 中的孵化项目。</p>
<p data-nodeid="19695">与 REST 相比，gRPC 支持 4 种不同的客户端与服务器的交互方式，如下表所示：</p>
<table data-nodeid="19697">
<thead data-nodeid="19698">
<tr data-nodeid="19699">
<th data-org-content="**交互方式**" data-nodeid="19701"><strong data-nodeid="19851">交互方式</strong></th>
<th data-org-content="**说明**" data-nodeid="19702"><strong data-nodeid="19855">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="19705">
<tr data-nodeid="19706">
<td data-org-content="一元 RPC" data-nodeid="19707">一元 RPC</td>
<td data-org-content="客户端发送单个请求，服务器返回单个消息响应" data-nodeid="19708">客户端发送单个请求，服务器返回单个消息响应</td>
</tr>
<tr data-nodeid="19709">
<td data-org-content="服务器流 RPC" data-nodeid="19710">服务器流 RPC</td>
<td data-org-content="客户端发送单个请求，服务器返回一个消息流作为响应" data-nodeid="19711">客户端发送单个请求，服务器返回一个消息流作为响应</td>
</tr>
<tr data-nodeid="19712">
<td data-org-content="客户端流 RPC" data-nodeid="19713">客户端流 RPC</td>
<td data-org-content="客户端发送一个消息流作为请求，服务器返回单个消息作为响应" data-nodeid="19714">客户端发送一个消息流作为请求，服务器返回单个消息作为响应</td>
</tr>
<tr data-nodeid="19715">
<td data-org-content="双向流 RPC" data-nodeid="19716">双向流 RPC</td>
<td data-org-content="客户端和服务器都可以发送消息流" data-nodeid="19717">客户端和服务器都可以发送消息流</td>
</tr>
</tbody>
</table>
<p data-nodeid="19718">在上表的 4 种交互方式中，双向流 RPC 的实现最为复杂，因为需要根据应用的需求来确定客户端和服务器发送消息的顺序。客户端和服务器的消息发送可能是交织在一起的，除了双向流 RPC 之外的其他 3 种交互方式在实现上都相对简单。</p>
<p data-nodeid="19719">对于云原生微服务来说，服务的内部 API 推荐使用 gRPC 来提高性能和减少带宽消耗；而对于服务的外部 API 来说，REST 仍然是目前的主流，也可以同时提供 REST 和 gRPC 两种外部 API。如果外部 API 使用 REST，而内部 API 使用 gRPC，那么我们可以使用 API 网关进行协议翻译。</p>
<p data-nodeid="19720">虽然 gRPC 并不限制消息的内容类型，你可以使用 JSON、XML 或 Thrift 作为消息格式，从支持的成熟度来说，Protocol Buffers 仍然是最佳的选择。下面首先对 Protocol Buffers 进行介绍（以下简称为 Protobuf）。</p>
<h3 data-nodeid="19721">Protocol Buffers</h3>
<p data-nodeid="19722">Protobuf 是一种语言中立、平台中立和可扩展的机制，用来对结构化数据进行序列化。它由 Google 提出，目前是开源的技术。在使用 Protobuf 时，我们通过它提供的语言来描述消息的结构，然后再通过工具生成特定语言上的代码。通过生成的代码来进行消息的序列化，包括写入和读取消息。</p>
<p data-nodeid="19723">Protobuf 的语言规范有两个版本 2 和 3，本课时介绍的是版本 3。Protobuf 中最基本的结构是消息类型，每个消息类型由多个字段组成。对于每个字段，我们需要定义它的名称、类型和编号。</p>
<p data-nodeid="19724">消息的名称使用首字母大写的 CamelCase 格式，而字段名称则使用下划线分隔的小写格式。字段的类型有很多种，比如常见的标量类型，包括 int32、int64、float、double、bool、string 和 bytes 等。除此之外，还可以使用 enum 来定义枚举类型。</p>
<p data-nodeid="19725">下面的代码展示了 Protobuf 中消息的定义，其中第一行的 syntax 声明了使用版本 3，message 用来声明消息类型。在枚举类型中，每个枚举项都需要指定对应的值，并且必须有一个值为 0 的枚举项。</p>
<pre class="lang-java" data-nodeid="19726"><code data-language="java">syntax = <span class="hljs-string">"proto3"</span>; 
message TestMessage { 
  int32 v1 = <span class="hljs-number">1</span>; 
  <span class="hljs-keyword">double</span> v2 = <span class="hljs-number">2</span>; 
  string v3 = <span class="hljs-number">3</span>; 
  bool v4 = <span class="hljs-number">4</span>; 
  <span class="hljs-keyword">enum</span> Color { 
    Red = <span class="hljs-number">0</span>; 
    Green = <span class="hljs-number">1</span>; 
    Blue = <span class="hljs-number">2</span>; 
  } 
  Color color = <span class="hljs-number">5</span>; 
}
</code></pre>
<p data-nodeid="19727">消息中的每个字段都有一个唯一的数字编号，在消息的二进制格式中，该编号会作为字段的标识符。在消息类型被使用之后，该编号不能被修改。编号从 1 ~ 15 的字段只需要一个字节就可以对字段的编号与类型进行编码，而编号为 16 ~ 2047 的字段则需要两个字节，从节省带宽的角度来说，编号 1 ~ 15 应该分配给出现频率较高的字段。</p>
<p data-nodeid="19728">消息类型中定义的字段，如果没有特殊声明，那么在实际的消息中最多出现一次。而对于可能出现多次的字段，则需要使用 repeated 来进行声明。</p>
<p data-nodeid="19729">另外一种特殊的字段类型是 oneof，表示消息中包含的某些字段，在同一时间只会至多设置一个字段的值。当设置其中一个字段的值之后，其他字段的值会被自动清空。在下面代码的 DemoMessage 中，test 的类型是 oneof，这就意味着 test 中定义的 name 和 type 字段不会同时出现。</p>
<pre class="lang-java" data-nodeid="19730"><code data-language="java">message DemoMessage { 
  oneof test { 
    string name = <span class="hljs-number">1</span>; 
    int32 type = <span class="hljs-number">2</span>; 
  } 
  bool enabled = <span class="hljs-number">3</span>; 
  repeated string values = <span class="hljs-number">4</span>; 
}
</code></pre>
<h3 data-nodeid="19731">gRPC 使用</h3>
<p data-nodeid="19732">gRPC 的方法也在 Protobuf 定义文件中声明。gRPC 的一个重要特征是通过代码生成工具来产生服务器和客户端的存根代码，生成的存根代码封装了 gRPC 底层的传输协议的细节。以服务器存根代码来说，开发者所要处理的只是从 Protobuf 文件中生成的 Java 对象，并不需要了解对象序列化的细节；从业务逻辑上来说，也只需要实现对方法调用的处理逻辑即可，并不需要了解传输协议的细节。下面介绍如何为地址管理服务提供 gRPC 协议的 API。</p>
<h4 data-nodeid="19733">Protobuf 文档</h4>
<p data-nodeid="19734">创建 gRPC 服务的第一步是编写 Protobuf 文档，该文档用来描述 gRPC 服务所支持的方法，以及方法的参数和返回值的消息格式。下面的代码是地址管理服务 gRPC 的 Protobuf 文档的部分代码，该文档主要由 3 个部分组成，通过不同的指令来描述。下表给出了 Protobuf 文件中的常用指令。</p>
<table data-nodeid="19736">
<thead data-nodeid="19737">
<tr data-nodeid="19738">
<th data-org-content="**指令**" data-nodeid="19740"><strong data-nodeid="19888">指令</strong></th>
<th data-org-content="**说明**" data-nodeid="19741"><strong data-nodeid="19892">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="19744">
<tr data-nodeid="19745">
<td data-org-content="option" data-nodeid="19746">option</td>
<td data-org-content="与代码生成相关的选项" data-nodeid="19747">与代码生成相关的选项</td>
</tr>
<tr data-nodeid="19748">
<td data-org-content="message" data-nodeid="19749">message</td>
<td data-org-content="不同消息类型的声明" data-nodeid="19750">不同消息类型的声明</td>
</tr>
<tr data-nodeid="19751">
<td data-org-content="service" data-nodeid="19752">service</td>
<td data-org-content="所提供的 gRPC 服务" data-nodeid="19753">所提供的 gRPC 服务</td>
</tr>
<tr data-nodeid="19754">
<td data-org-content="rpc" data-nodeid="19755">rpc</td>
<td data-org-content="服务中包含的可供调用的方法" data-nodeid="19756">服务中包含的可供调用的方法</td>
</tr>
</tbody>
</table>
<pre class="lang-java" data-nodeid="19757"><code data-language="java">syntax = <span class="hljs-string">"proto3"</span>; 
option java_multiple_files = <span class="hljs-keyword">true</span>; 
option java_package = <span class="hljs-string">"io.vividcode.happyride.addressservice.grpc"</span>; 
option java_outer_classname = <span class="hljs-string">"AddressServiceProto"</span>; 
message Area { 
  int32 id = <span class="hljs-number">1</span>; 
  int32 level = <span class="hljs-number">2</span>; 
} 
message Address { 
  string id = <span class="hljs-number">1</span>; 
  repeated Area areas = <span class="hljs-number">6</span>; 
} 
message GetAddressRequest { 
  string address_id = <span class="hljs-number">1</span>; 
  int32 area_level = <span class="hljs-number">2</span>; 
} 
message GetAddressResponse { 
  oneof optional_address { 
    Address address = <span class="hljs-number">1</span>; 
  } 
} 
message GetAreaRequest { 
  int64 area_code = <span class="hljs-number">1</span>; 
  int32 ancestor_level = <span class="hljs-number">2</span>; 
} 
message GetAreaResponse { 
  oneof optional_area { 
    Area area = <span class="hljs-number">1</span>; 
  } 
} 
message AddressSearchRequest { 
  int64 area_code = <span class="hljs-number">1</span>; 
  string query = <span class="hljs-number">2</span>; 
} 
service AddressService { 
  <span class="hljs-function">rpc <span class="hljs-title">GetAddress</span><span class="hljs-params">(GetAddressRequest)</span> <span class="hljs-title">returns</span> <span class="hljs-params">(GetAddressResponse)</span></span>; 
  <span class="hljs-function">rpc <span class="hljs-title">GetArea</span><span class="hljs-params">(GetAreaRequest)</span> <span class="hljs-title">returns</span> <span class="hljs-params">(GetAreaResponse)</span></span>; 
  <span class="hljs-function">rpc <span class="hljs-title">Search</span><span class="hljs-params">(AddressSearchRequest)</span> <span class="hljs-title">returns</span> <span class="hljs-params">(stream Address)</span></span>; 
  <span class="hljs-function">rpc <span class="hljs-title">GetAddresses</span><span class="hljs-params">(stream GetAddressRequest)</span> <span class="hljs-title">returns</span> <span class="hljs-params">(stream Address)</span></span>; 
}
</code></pre>
<p data-nodeid="19758">对于 Java 应用来说，我们可以用下表中给出的选项来对生成的代码进行配置。</p>
<table data-nodeid="19760">
<thead data-nodeid="19761">
<tr data-nodeid="19762">
<th data-org-content="**选项**" data-nodeid="19764"><strong data-nodeid="19905">选项</strong></th>
<th data-org-content="**说明**" data-nodeid="19765"><strong data-nodeid="19909">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="19768">
<tr data-nodeid="19769">
<td data-org-content="java\_multiple\_files" data-nodeid="19770">java_multiple_files</td>
<td data-org-content="当值为 true 时，Protobuf 文件中的每个 message 类型，都会生成各自的 Java 文件；否则，每个 message 类型都会作为单个 Java 类的内部类" data-nodeid="19771">当值为 true 时，Protobuf 文件中的每个 message 类型，都会生成各自的 Java 文件；否则，每个 message 类型都会作为单个 Java 类的内部类</td>
</tr>
<tr data-nodeid="19772">
<td data-org-content="java\_package" data-nodeid="19773">java_package</td>
<td data-org-content="生成的 Java 代码的包名" data-nodeid="19774">生成的 Java 代码的包名</td>
</tr>
<tr data-nodeid="19775">
<td data-org-content="java\_outer\_classname" data-nodeid="19776">java_outer_classname</td>
<td data-org-content="生成的 Java 类的名称。当 java\_multiple\_files 为 false 时，该类作为包含 message 类型的外部类" data-nodeid="19777">生成的 Java 类的名称。当 java_multiple_files 为 false 时，该类作为包含 message 类型的外部类</td>
</tr>
</tbody>
</table>
<p data-nodeid="19778">在 Protobuf 文件中的 message 类型，用来描述 gRPC 服务所提供的方法的参数和返回值的类型。一般来说，每个方法的参数和返回值都有各自独立的 message 类型声明。在方法的声明中，stream 表示流，可以出现在方法的参数或返回值的声明中，对应于不同的交互方式。在上面代码的声明中，GetAddress 和 GetArea 方法使用的是一元 RPC，而 Search 方法使用的是服务器流 RPC，GetAddresses 方法使用的是双向流 RPC。</p>
<h4 data-nodeid="19779">代码生成</h4>
<p data-nodeid="19780">在完成了 protobuf 的声明之后，下一步是使用工具来生成相关的代码存根，生成代码时需要使用 protoc 工具及 gRPC 插件。在 Java 应用中，我们通过 Maven 插件来使用 protoc，Protobuf 文件保存在 src/main/proto 目录中。</p>
<p data-nodeid="19781">下面的代码是相关的 Maven 配置，其中 os-maven-plugin 插件用来检测当前环境的操作系统版本，protobuf-maven-plugin 是运行 protoc 的 Maven 插件。</p>
<pre class="lang-xml" data-nodeid="19782"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">build</span>&gt;</span> 
&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">extensions</span>&gt;</span> 
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">extension</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>kr.motd.maven<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>os-maven-plugin<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.6.2<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span> 
&nbsp; &nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">extension</span>&gt;</span> 
&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">extensions</span>&gt;</span> 
&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">plugins</span>&gt;</span> 
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">plugin</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.xolstice.maven.plugins<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>protobuf-maven-plugin<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>0.6.1<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">configuration</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">protocArtifact</span>&gt;</span>com.google.protobuf:protoc:${protoc.version}:exe:${os.detected.classifier}<span class="hljs-tag">&lt;/<span class="hljs-name">protocArtifact</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">pluginId</span>&gt;</span>grpc-java<span class="hljs-tag">&lt;/<span class="hljs-name">pluginId</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">pluginArtifact</span>&gt;</span>io.grpc:protoc-gen-grpc-java:${grpc.version}:exe:${os.detected.classifier}<span class="hljs-tag">&lt;/<span class="hljs-name">pluginArtifact</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">configuration</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">executions</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">execution</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">goals</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">goal</span>&gt;</span>compile<span class="hljs-tag">&lt;/<span class="hljs-name">goal</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">goal</span>&gt;</span>compile-custom<span class="hljs-tag">&lt;/<span class="hljs-name">goal</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">goals</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">execution</span>&gt;</span> 
&nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">executions</span>&gt;</span> 
&nbsp; &nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">plugin</span>&gt;</span> 
&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">plugins</span>&gt;</span> 
<span class="hljs-tag">&lt;/<span class="hljs-name">build</span>&gt;</span>
</code></pre>
<p data-nodeid="19783">在使用 Maven 构建之后，会在 Maven 项目的 target/generated-sources/protobuf 目录下生成 protobuf 相关的 Java 代码，其中 java 子目录中包含的是消息类型对应的 Java 代码，而 grpc-java 目录下面包含的是 gRPC 服务的代码。下面的代码给出了生成文件的目录结构。</p>
<pre class="lang-java" data-nodeid="19784"><code data-language="java">. 
├── grpc-java 
│&nbsp; &nbsp;└── io 
│&nbsp; &nbsp; &nbsp; &nbsp;└── vividcode 
│&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;└── happyride 
│&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;└── addressservice 
│&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;└── grpc 
│&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;└── AddressServiceGrpc.java 
└── java 
&nbsp; &nbsp; └── io 
&nbsp; &nbsp; &nbsp; &nbsp; └── vividcode 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; └── happyride 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; └── addressservice 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; └── grpc 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── Address.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── AddressOrBuilder.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── AddressSearchRequest.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── AddressSearchRequestOrBuilder.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── AddressServiceProto.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── Area.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── AreaOrBuilder.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── GetAddressRequest.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── GetAddressRequestOrBuilder.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── GetAddressResponse.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── GetAddressResponseOrBuilder.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── GetAreaRequest.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── GetAreaRequestOrBuilder.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ├── GetAreaResponse.java 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; └── GetAreaResponseOrBuilder.java
</code></pre>
<p data-nodeid="19785">生成的 AddressServiceGrpc 类中包含了 gRPC 服务器和客户端的代码，其中 AddressServiceImplBase 是服务端的基本实现类，其中的每个方法都对应 Protobuf 文件中服务 AddressService 中声明的方法。在服务端实现中，只需要继承 AddressServiceImplBase 类，并覆写这些方法即可。</p>
<h4 data-nodeid="19786">服务端实现</h4>
<p data-nodeid="19787">下面的代码给出了具体服务端实现类 AddressGrpcService 的部分代码。在 getAddress 方法中，方法的参数类型 GetAddressRequest 对应于 protobuf 中同名方法的参数的消息类型，而 StreamObserver 对象用来产生作为响应的消息。</p>
<p data-nodeid="19788">StreamObserver 接口中的方法与反应式编程中的 Observer 是相同的，具体的方法如下表所示。</p>
<table data-nodeid="19790">
<thead data-nodeid="19791">
<tr data-nodeid="19792">
<th data-org-content="**方法**" data-nodeid="19794"><strong data-nodeid="19942">方法</strong></th>
<th data-org-content="**说明**" data-nodeid="19795"><strong data-nodeid="19946">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="19798">
<tr data-nodeid="19799">
<td data-org-content="onNext(V value)" data-nodeid="19800">onNext(V value)</td>
<td data-org-content="产生一个消息" data-nodeid="19801">产生一个消息</td>
</tr>
<tr data-nodeid="19802">
<td data-org-content="onError(Throwable t)" data-nodeid="19803">onError(Throwable t)</td>
<td data-org-content="产生一个错误并终止流" data-nodeid="19804">产生一个错误并终止流</td>
</tr>
<tr data-nodeid="19805">
<td data-org-content="onCompleted()" data-nodeid="19806">onCompleted()</td>
<td data-org-content="正常终止流" data-nodeid="19807">正常终止流</td>
</tr>
</tbody>
</table>
<p data-nodeid="19808">Protobuf 中的每个消息都使用构建器模式来创建。下面代码中的 buildAddress 方法用来把 AddressVO 对象转换成 protobuf 中的 Address 类型的消息。而对于产生的消息，可通过 StreamObserver 对象的 onNext 方法来发送，等全部消息发送完成之后，使用 onCompleted 来结束流。在运行时，getAddress 方法最多只调用 onNext 方法一次，而 search 方法则可能多次调用 onNext 方法来产生多条消息。虽然 getAddress 方法使用的是一元 RPC 的交互模式，但是在实现中，仍然以流的形式来表示。</p>
<pre class="lang-java" data-nodeid="19809"><code data-language="java"><span class="hljs-meta">@GRpcService</span> 
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AddressGrpcService</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AddressServiceImplBase</span> </span>{ 
&nbsp; <span class="hljs-meta">@Autowired</span> 
&nbsp; AddressService addressService; 
&nbsp; <span class="hljs-meta">@Autowired</span> 
&nbsp; AreaService areaService; 
&nbsp; <span class="hljs-meta">@Override</span> 
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">getAddress</span><span class="hljs-params">(GetAddressRequest request, 
&nbsp; &nbsp; &nbsp; StreamObserver&lt;GetAddressResponse&gt; responseObserver)</span> </span>{ 
&nbsp; &nbsp; GetAddressResponse.Builder builder = GetAddressResponse.newBuilder(); 
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.addressService 
&nbsp; &nbsp; &nbsp; &nbsp; .getAddress(request.getAddressId(), request.getAreaLevel()) 
&nbsp; &nbsp; &nbsp; &nbsp; .ifPresent(address -&gt; builder.setAddress(<span class="hljs-keyword">this</span>.buildAddress(address))); 
&nbsp; &nbsp; responseObserver.onNext(builder.build()); 
&nbsp; &nbsp; responseObserver.onCompleted(); 
&nbsp; } 
&nbsp; <span class="hljs-meta">@Override</span> 
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">search</span><span class="hljs-params">(AddressSearchRequest request, 
&nbsp; &nbsp; &nbsp; StreamObserver&lt;Address&gt; responseObserver)</span> </span>{ 
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.addressService.search(request.getAreaCode(), request.getQuery()) 
&nbsp; &nbsp; &nbsp; &nbsp; .forEach( 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; address -&gt; responseObserver.onNext(<span class="hljs-keyword">this</span>.buildAddress(address))); 
&nbsp; &nbsp; responseObserver.onCompleted(); 
&nbsp; } 
&nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> Address <span class="hljs-title">buildAddress</span><span class="hljs-params">(AddressVO address)</span> </span>{ 
&nbsp; &nbsp; <span class="hljs-keyword">return</span> Address.newBuilder().setId(address.getId()) 
&nbsp; &nbsp; &nbsp; &nbsp; .setAreaId(address.getAreaId()) 
&nbsp; &nbsp; &nbsp; &nbsp; .setAddressLine(address.getAddressLine()) 
&nbsp; &nbsp; &nbsp; &nbsp; .setLat(address.getLat().toPlainString()) 
&nbsp; &nbsp; &nbsp; &nbsp; .setLng(address.getLng().toPlainString()) 
&nbsp; &nbsp; &nbsp; &nbsp; .addAllAreas(address.getAreas().stream() 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .map(<span class="hljs-keyword">this</span>::buildArea).collect(Collectors.toList())) 
&nbsp; &nbsp; &nbsp; &nbsp; .build(); 
&nbsp; } 
}
</code></pre>
<p data-nodeid="19810">下面的代码是使用双向流 RPC 的 getAddresses 方法，该方法的返回值同样是一个 StreamObserver 对象，表示客户端请求的流。当作为返回值 StreamObserver 对象的 onNext 方法被调用时，说明客户端发送了一个新的消息，对于这个消息的处理方式是往 responseObserver 表示的流中写入作为响应的 Address 类型的消息。</p>
<pre class="lang-java" data-nodeid="19811"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> StreamObserver&lt;GetAddressRequest&gt; <span class="hljs-title">getAddresses</span><span class="hljs-params">( 
&nbsp; &nbsp; StreamObserver&lt;Address&gt; responseObserver)</span> </span>{ 
&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> StreamObserver&lt;GetAddressRequest&gt;() { 
&nbsp; &nbsp; <span class="hljs-meta">@Override</span> 
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onNext</span><span class="hljs-params">(GetAddressRequest request)</span> </span>{ 
&nbsp; &nbsp; &nbsp; AddressGrpcService.<span class="hljs-keyword">this</span>.addressService 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .getAddress(request.getAddressId(), request.getAreaLevel()) 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .ifPresent( 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; address -&gt; responseObserver.onNext( 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; AddressGrpcService.<span class="hljs-keyword">this</span>.buildAddress(address))); 
&nbsp; &nbsp; } 
&nbsp; &nbsp; <span class="hljs-meta">@Override</span> 
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onError</span><span class="hljs-params">(Throwable t)</span> </span>{ 
&nbsp; &nbsp; &nbsp; AddressGrpcService.LOGGER.warn(<span class="hljs-string">"Error"</span>, t); 
&nbsp; &nbsp; } 
&nbsp; &nbsp; <span class="hljs-meta">@Override</span> 
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onCompleted</span><span class="hljs-params">()</span> </span>{ 
&nbsp; &nbsp; &nbsp; responseObserver.onCompleted(); 
&nbsp; &nbsp; } 
&nbsp; }; 
}
</code></pre>
<p data-nodeid="19812">AddressGrpcService 类上的注解 @GRpcService 来自 <a href="https://github.com/LogNet/grpc-spring-boot-starter" data-nodeid="19958">grpc-spring-boot-starter 库</a>，用来在 Spring Boot 中集成 gRPC 服务。该第三方库可以自动启动 gRPC 服务器，免去了烦琐的配置。gRPC 服务器默认在 6565 端口启动，可以通过配置项 grpc.port 来修改。</p>
<h4 data-nodeid="19813">客户端实现</h4>
<p data-nodeid="19814">我们可以用生成的 gRPC 服务的客户端来调用服务。生成的代码中包含了 3 种不同类型的客户端，如下表所示：</p>
<table data-nodeid="19816">
<thead data-nodeid="19817">
<tr data-nodeid="19818">
<th data-org-content="**客户端**" data-nodeid="19820"><strong data-nodeid="19965">客户端</strong></th>
<th data-org-content="**说明**" data-nodeid="19821"><strong data-nodeid="19969">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="19824">
<tr data-nodeid="19825">
<td data-org-content="AddressServiceStub" data-nodeid="19826">AddressServiceStub</td>
<td data-org-content="使用 StreamObserver 的异步调用客户端" data-nodeid="19827">使用 StreamObserver 的异步调用客户端</td>
</tr>
<tr data-nodeid="19828">
<td data-org-content="AddressServiceBlockingStub" data-nodeid="19829">AddressServiceBlockingStub</td>
<td data-org-content="执行同步调用的阻塞客户端" data-nodeid="19830">执行同步调用的阻塞客户端</td>
</tr>
<tr data-nodeid="19831">
<td data-org-content="AddressServiceFutureStub" data-nodeid="19832">AddressServiceFutureStub</td>
<td data-org-content="使用 Guava 中 ListenableFuture 的客户端" data-nodeid="19833">使用 Guava 中 ListenableFuture 的客户端</td>
</tr>
</tbody>
</table>
<p data-nodeid="19834">在这 3 种客户端中，同步调用的客户端使用最简单，在创建客户端时，需要提供 Channel 对象。在下面的代码中，ManagedChannelBuilder 类用来创建连接到本机的 6565 端口的 Channel 对象，而 AddressServiceGrpc 类的 newBlockingStub 方法用来创建阻塞客户端的 AddressServiceBlockingStub 对象，该对象中方法的返回值是实际响应消息类型的对象。</p>
<pre class="lang-java" data-nodeid="19835"><code data-language="java">Channel channel = ManagedChannelBuilder.forAddress(<span class="hljs-string">"localhost"</span>, <span class="hljs-number">6565</span>) 
&nbsp; &nbsp; .usePlaintext().build(); 
AddressServiceBlockingStub blockingStub = AddressServiceGrpc 
&nbsp; &nbsp; .newBlockingStub(channel); 
Iterator&lt;Address&gt; result = blockingStub 
&nbsp; &nbsp; .search(AddressSearchRequest.newBuilder() 
&nbsp; &nbsp; &nbsp; &nbsp; .setAreaCode(<span class="hljs-number">110101001015L</span>) 
&nbsp; &nbsp; &nbsp; &nbsp; .setQuery(<span class="hljs-string">"王府井社区居委会"</span>) 
&nbsp; &nbsp; &nbsp; &nbsp; .build()); 
result.forEachRemaining(System.out::println);
</code></pre>
<p data-nodeid="19836">下面的代码给出了异步非阻塞客户端的使用方式。通过 AddressServiceGrpc 类的 newStub 方法可以创建出作为客户端的 AddressServiceStub 对象。在调用 getAddresses 方法时，返回的 StreamObserver<code data-backticks="1" data-nodeid="19978">&lt;GetAddressRequest&gt;</code> 对象用来发送请求消息，而作为参数的 StreamObserver<code data-backticks="1" data-nodeid="19980">&lt;Address&gt;</code> 对象则用来处理服务器返回的消息。CountDownLatch 对象的作用是等待响应流的结束。</p>
<pre class="lang-java" data-nodeid="19837"><code data-language="java">Channel channel = ManagedChannelBuilder.forAddress(<span class="hljs-string">"localhost"</span>, <span class="hljs-number">6565</span>) 
&nbsp; &nbsp; .usePlaintext().build(); 
AddressServiceStub asyncStub = AddressServiceGrpc.newStub(channel); 
CountDownLatch finishLatch = <span class="hljs-keyword">new</span> CountDownLatch(<span class="hljs-number">1</span>); 
StreamObserver&lt;GetAddressRequest&gt; requestObserver = asyncStub 
&nbsp; &nbsp; .getAddresses(<span class="hljs-keyword">new</span> StreamObserver&lt;Address&gt;() { 
&nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span> 
&nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onNext</span><span class="hljs-params">(Address value)</span> </span>{ 
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(value); 
&nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span> 
&nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onError</span><span class="hljs-params">(Throwable t)</span> </span>{ 
&nbsp; &nbsp; &nbsp; &nbsp; t.printStackTrace(); 
&nbsp; &nbsp; &nbsp; &nbsp; finishLatch.countDown(); 
&nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span> 
&nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onCompleted</span><span class="hljs-params">()</span> </span>{ 
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"Completed"</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; finishLatch.countDown(); 
&nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; }); 
<span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">1</span>; i &lt;= <span class="hljs-number">3</span>; i++) { 
&nbsp; requestObserver.onNext( 
&nbsp; &nbsp; &nbsp; GetAddressRequest.newBuilder() 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .setAddressId(<span class="hljs-string">"962fddbc-54cc-4758-bf01-56e2833c6443"</span>) 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .setAreaLevel(i).build()); 
} 
requestObserver.onCompleted(); 
finishLatch.await(<span class="hljs-number">1</span>, TimeUnit.MINUTES);
</code></pre>
<p data-nodeid="19838">在本地开发和调试中，直接使用 Java 客户端不太方便，推荐使用单独的 gRPC 工具，如 <a href="https://github.com/uw-labs/bloomrpc" data-nodeid="19985">BloomRPC</a>。BloomRPC 可以导入 Protobuf 文件并调用 gRPC 服务，如下图所示。</p>
<p data-nodeid="19839"><img src="https://s0.lgstatic.com/i/image/M00/40/D5/Ciqc1F8zqeKARFX7AAJaspyUGxw904.png" alt="bloomrpc.png" data-nodeid="19989"></p>
<h3 data-nodeid="19840">总结</h3>
<p data-nodeid="19841">相对于 REST API，gRPC 提供了更灵活的交互方式、更好的性能和更少的带宽占用，适用于云原生应用中微服务之间的交互。通过本课时的学习，你可以对 gRPC 和 Protocol Buffers 有基本的了解，以及如何在应用中创建 gRPC 服务，并通过客户端来调用 gRPC 服务。</p>
<p data-nodeid="22469" class="">最后呢，成老师邀请你为本专栏课程进行结课评价，因为你的每一个观点都是我们最关注的点。<a href="https://wj.qq.com/s2/6902680/3fb2/" data-nodeid="22473">点击链接，即可参与课程评价</a>。</p>