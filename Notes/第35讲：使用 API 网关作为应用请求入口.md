<p data-nodeid="1393">在第 31 课时介绍 API 组合时，对 API 网关进行了基本的介绍，API 网关是外部用户访问应用数据的唯一入口。API 网关既充当了开放数据桥梁的作用，同时也是内部数据的屏障，其所提供的功能很多，包括请求路由、边界功能、API 组合和协议翻译等。本课时将着重介绍 Istio 中的虚拟服务以及 API 网关的请求路由功能。</p>


<p data-nodeid="3">在介绍 API 网关之前，首先介绍一下 Istio 中与服务相关的基本概念。</p>
<h3 data-nodeid="4">服务发现</h3>
<p data-nodeid="5">在介绍 Istio 的功能时，服务是经常会被提到的概念，这个词在不同的语境中有不同的含义。对 Istio 来说，服务表示的是应用行为的一个集合，每个服务在服务注册表中都有自己的名字。一个服务由一个或多个网络终端（Endpoint）组成，而每个终端有对应的一个或多个工作负载（Workload）。</p>
<p data-nodeid="6">以地址管理服务为例，它表示的是与地址管理相关的行为的集合，其名称是 address-service。该服务可能由多个部分组成，每个部分有自己的网络终端，即便该服务只有一个部分，在运行时也可能有该服务的不同版本，而服务的每个版本都有各自的终端。</p>
<p data-nodeid="7">如果以 Kubernetes 中的概念来做一下类比，那网络终端就是 Kubernetes 中的服务，而工作负载则与 Pod 相对应。如果一个服务存在多个版本，那么每个版本都有各自对应的 Kubernetes 部署，而每个部署有各自的 Pod 作为工作负载。</p>
<p data-nodeid="8">Istio 在内部维护了一个服务注册表，包含服务及其对应的网络终端。Istio 自身并不支持服务发现，而是依赖底层平台的支持，如 Kubernetes 和 Consul。除了通过服务发现机制查找到的服务之外，还可以手动添加服务到注册表中。这种手动添加方式适用于将第三方服务或遗留系统集成到服务网格中。</p>
<h3 data-nodeid="9">虚拟服务</h3>
<p data-nodeid="10">为了更加灵活的控制服务之间的流量，Istio 引入了虚拟服务的概念，虚拟服务实际上是一系列路由规则的集合。每个虚拟服务由主机名和路由两个部分组成，主机名指的是接收请求的目标，而路由规则指的是对于接收到的请求的处理方式。</p>
<p data-nodeid="11">虚拟服务由 Istio 中的自定义资源 VirtualService 来表示，路由规则根据使用的协议有不同的配置。虚拟服务支持的协议通过不同的属性名称来确定，如下表所示：</p>
<table data-nodeid="13">
<thead data-nodeid="14">
<tr data-nodeid="15">
<th data-org-content="**属性名称**" data-nodeid="17"><strong data-nodeid="262">属性名称</strong></th>
<th data-org-content="**支持的协议**" data-nodeid="18"><strong data-nodeid="266">支持的协议</strong></th>
</tr>
</thead>
<tbody data-nodeid="21">
<tr data-nodeid="22">
<td data-org-content="http" data-nodeid="23">http</td>
<td data-org-content="HTTP 1.1、HTTP 2 和 gRPC" data-nodeid="24">HTTP 1.1、HTTP 2 和 gRPC</td>
</tr>
<tr data-nodeid="25">
<td data-org-content="tls" data-nodeid="26">tls</td>
<td data-org-content="TLS、HTTPS" data-nodeid="27">TLS、HTTPS</td>
</tr>
<tr data-nodeid="28">
<td data-org-content="tcp" data-nodeid="29">tcp</td>
<td data-org-content="TCP" data-nodeid="30">TCP</td>
</tr>
</tbody>
</table>
<p data-nodeid="31">这 3 种协议的路由规则都支持 match 和 route 两个属性，match 属性表示规则所匹配的条件，route 属性则表示请求要发送的实际目的地和相应的处理方式。</p>
<h4 data-nodeid="32">路由的目的地</h4>
<p data-nodeid="33">每个路由可以有多个目的地，每个目的地的定义包含下表所示的属性：</p>
<table data-nodeid="35">
<thead data-nodeid="36">
<tr data-nodeid="37">
<th data-org-content="**属性**" data-nodeid="39"><strong data-nodeid="279">属性</strong></th>
<th data-org-content="**说明**" data-nodeid="40"><strong data-nodeid="283">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="43">
<tr data-nodeid="44">
<td data-org-content="destination" data-nodeid="45">destination</td>
<td data-org-content="转发请求的目的地" data-nodeid="46">转发请求的目的地</td>
</tr>
<tr data-nodeid="47">
<td data-org-content="weight" data-nodeid="48">weight</td>
<td data-org-content="发送到该目的地的请求的百分比" data-nodeid="49">发送到该目的地的请求的百分比</td>
</tr>
<tr data-nodeid="50">
<td data-org-content="headers" data-nodeid="51">headers</td>
<td data-org-content="对 HTTP 头所做的修改" data-nodeid="52">对 HTTP 头所做的修改</td>
</tr>
</tbody>
</table>
<p data-nodeid="53">下面是一个虚拟服务的示例，该服务的主机名是 address-service.happyride.svc.cluster.local，对应的目的地的主机名是相同的。我们通过 headers 属性修改 HTTP 响应的头来添加自定义头 x-hello。</p>
<pre class="lang-yaml" data-nodeid="54"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">networking.istio.io/v1beta1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">VirtualService</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">address-service</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">hosts:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-string">address-service.happyride.svc.cluster.local</span>
  <span class="hljs-attr">http:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">"address-service-http"</span>
      <span class="hljs-attr">route:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">destination:</span>
            <span class="hljs-attr">host:</span> <span class="hljs-string">address-service.happyride.svc.cluster.local</span>
          <span class="hljs-attr">headers:</span>
            <span class="hljs-attr">response:</span>
              <span class="hljs-attr">add:</span>
                <span class="hljs-attr">x-hello:</span> <span class="hljs-string">world</span>
</code></pre>
<p data-nodeid="55">当创建了该虚拟服务之后，在 Kubernetes 集群内部，可以通过下面的命令来访问服务的 API。</p>
<pre class="lang-java" data-nodeid="2782"><code data-language="java">curl -i&nbsp; http:<span class="hljs-comment">//address-service/actuator/health/liveness</span>
</code></pre>


<p data-nodeid="57">下面是上述命令的输出结果，从中可以看到添加的自定义 HTTP 头。</p>
<pre class="lang-java" data-nodeid="3707"><code data-language="java">HTTP/<span class="hljs-number">1.1</span> <span class="hljs-number">200</span> OK
content-type: application/vnd.spring-boot.actuator.v3+json
date: Thu, <span class="hljs-number">09</span> Jul <span class="hljs-number">2020</span> <span class="hljs-number">22</span>:<span class="hljs-number">50</span>:<span class="hljs-number">02</span> GMT
x-envoy-upstream-service-time: <span class="hljs-number">3</span>
server: envoy
x-hello: world
transfer-encoding: chunked
{<span class="hljs-string">"status"</span>:<span class="hljs-string">"UP"</span>}
</code></pre>

<p data-nodeid="59">在指定服务的主机名时，可以只使用服务名称，而不是完整的 DNS 名称。当只使用服务名称时，Istio 会将服务名称解析为虚拟服务资源所在名称空间中的服务。仅使用服务名称的做法虽然简单，但是可能会错误地指向不正确的名称空间，因此推荐使用完整的 DNS 名称，可以避免潜在的配置问题。</p>
<p data-nodeid="60">destination 属性表示的目的地可以包含如下属性：</p>
<table data-nodeid="62">
<thead data-nodeid="63">
<tr data-nodeid="64">
<th data-org-content="**属性**" data-nodeid="66"><strong data-nodeid="298">属性</strong></th>
<th data-org-content="**说明**" data-nodeid="67"><strong data-nodeid="302">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="70">
<tr data-nodeid="71">
<td data-org-content="host" data-nodeid="72">host</td>
<td data-org-content="服务注册表中的服务名称" data-nodeid="73">服务注册表中的服务名称</td>
</tr>
<tr data-nodeid="74">
<td data-org-content="subset" data-nodeid="75">subset</td>
<td data-org-content="目的地服务中的子集名称" data-nodeid="76">目的地服务中的子集名称</td>
</tr>
<tr data-nodeid="77">
<td data-org-content="port" data-nodeid="78">port</td>
<td data-org-content="端口" data-nodeid="79">端口</td>
</tr>
</tbody>
</table>
<h4 data-nodeid="80">路由的匹配条件</h4>
<p data-nodeid="81">路由规则的 match 属性表示的是匹配条件的列表，只需要列表中任意的一个条件满足，那么该路由就可以匹配成功。在单个匹配条件中，也可以添加多个断言来检查请求的不同方面的属性。同一个匹配条件中的不同断言，需要全部满足，该条件才能满足。</p>
<p data-nodeid="82">下表给出了匹配条件中的断言可以检查的请求中的属性。</p>
<table data-nodeid="84">
<thead data-nodeid="85">
<tr data-nodeid="86">
<th data-org-content="**属性**" data-nodeid="88"><strong data-nodeid="315">属性</strong></th>
<th data-org-content="**匹配目标**" data-nodeid="89"><strong data-nodeid="319">匹配目标</strong></th>
</tr>
</thead>
<tbody data-nodeid="92">
<tr data-nodeid="93">
<td data-org-content="uri" data-nodeid="94">uri</td>
<td data-org-content="URI 中的路径" data-nodeid="95">URI 中的路径</td>
</tr>
<tr data-nodeid="96">
<td data-org-content="scheme" data-nodeid="97">scheme</td>
<td data-org-content="URI 的模式" data-nodeid="98">URI 的模式</td>
</tr>
<tr data-nodeid="99">
<td data-org-content="method" data-nodeid="100">method</td>
<td data-org-content="HTTP 方法" data-nodeid="101">HTTP 方法</td>
</tr>
<tr data-nodeid="102">
<td data-org-content="authority" data-nodeid="103">authority</td>
<td data-org-content="URI 中的 authority" data-nodeid="104">URI 中的 authority</td>
</tr>
<tr data-nodeid="105">
<td data-org-content="headers" data-nodeid="106">headers</td>
<td data-org-content="HTTP 头" data-nodeid="107">HTTP 头</td>
</tr>
<tr data-nodeid="108">
<td data-org-content="port" data-nodeid="109">port</td>
<td data-org-content="端口" data-nodeid="110">端口</td>
</tr>
<tr data-nodeid="111">
<td data-org-content="queryParams" data-nodeid="112">queryParams</td>
<td data-org-content="查询参数" data-nodeid="113">查询参数</td>
</tr>
<tr data-nodeid="114">
<td data-org-content="withoutHeaders" data-nodeid="115">withoutHeaders</td>
<td data-org-content="不包含的 HTTP 头" data-nodeid="116">不包含的 HTTP 头</td>
</tr>
</tbody>
</table>
<p data-nodeid="117">在上表中，uri、scheme、method 和 authority 都是单个字符串类型的值，需要指定匹配的条件； headers、queryParams 和 withoutHeaders 则是一个哈希表，这个表中的键是 HTTP 头或查询参数的名称，而值则是相应的字符串匹配方式。</p>
<p data-nodeid="118">在匹配字符串时，支持下表中给出的 3 种模式。</p>
<table data-nodeid="120">
<thead data-nodeid="121">
<tr data-nodeid="122">
<th data-org-content="**模式**" data-nodeid="124"><strong data-nodeid="341">模式</strong></th>
<th data-org-content="**说明**" data-nodeid="125"><strong data-nodeid="345">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="128">
<tr data-nodeid="129">
<td data-org-content="exact" data-nodeid="130">exact</td>
<td data-org-content="完全匹配" data-nodeid="131">完全匹配</td>
</tr>
<tr data-nodeid="132">
<td data-org-content="prefix" data-nodeid="133">prefix</td>
<td data-org-content="匹配特定的前缀" data-nodeid="134">匹配特定的前缀</td>
</tr>
<tr data-nodeid="135">
<td data-org-content="regex" data-nodeid="136">regex</td>
<td data-org-content="使用正则表达式进行匹配" data-nodeid="137">使用正则表达式进行匹配</td>
</tr>
</tbody>
</table>
<p data-nodeid="138">下面的代码展示了路由匹配时不同条件及其断言的使用。第一个匹配条件要求 URI 前缀是 /test，并且存在值为 hello 的 HTTP 头 x-myheader；第二个匹配条件要求 URI 为 /demo，并且查询参数 from 的前缀为 myserver。</p>
<pre class="lang-yaml" data-nodeid="139"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">networking.istio.io/v1beta1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">VirtualService</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">virtual-service</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">hosts:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-string">virtual-service</span>
  <span class="hljs-attr">http:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">"virtual-service-http"</span>
      <span class="hljs-attr">match:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">uri:</span>
            <span class="hljs-attr">prefix:</span> <span class="hljs-string">"/test"</span>
          <span class="hljs-attr">headers:</span>
            <span class="hljs-attr">x-myheader:</span>
              <span class="hljs-attr">exact:</span> <span class="hljs-string">"hello"</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">uri:</span>
            <span class="hljs-attr">exact:</span> <span class="hljs-string">"/demo"</span>
          <span class="hljs-attr">queryParams:</span>
            <span class="hljs-attr">from:</span>
              <span class="hljs-attr">prefix:</span> <span class="hljs-string">"myserver"</span>        
      <span class="hljs-attr">route:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">destination:</span>
            <span class="hljs-attr">host:</span> <span class="hljs-string">test-service</span>
</code></pre>
<h3 data-nodeid="140">目的地规则</h3>
<p data-nodeid="141">对于每个目的地，可以使用目的地规则来进行配置。目的地规则通过自定义资源 DestinationRule 来配置，DestinationRule 资源可以使用下表中的属性：</p>
<table data-nodeid="143">
<thead data-nodeid="144">
<tr data-nodeid="145">
<th data-org-content="**属性**" data-nodeid="147"><strong data-nodeid="358">属性</strong></th>
<th data-org-content="**说明**" data-nodeid="148"><strong data-nodeid="362">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="151">
<tr data-nodeid="152">
<td data-org-content="host" data-nodeid="153">host</td>
<td data-org-content="服务注册表中的服务名称" data-nodeid="154">服务注册表中的服务名称</td>
</tr>
<tr data-nodeid="155">
<td data-org-content="trafficPolicy" data-nodeid="156">trafficPolicy</td>
<td data-org-content="流量控制策略" data-nodeid="157">流量控制策略</td>
</tr>
<tr data-nodeid="158">
<td data-org-content="subsets" data-nodeid="159">subsets</td>
<td data-org-content="服务的子集" data-nodeid="160">服务的子集</td>
</tr>
</tbody>
</table>
<h4 data-nodeid="161">流量控制策略</h4>
<p data-nodeid="162">一个目的地可以对应多个实际的主机。流量控制策略指的是请求在不同的主机之间进行转发时的策略，最常见的是使用 loadBalancer 属性来配置负载均衡器。一共有 3 种方式来配置负载均衡器。</p>
<p data-nodeid="163">第一种是使用 simple 属性来设置简单的负载均衡策略，可以支持的策略如下表所示：</p>
<table data-nodeid="165">
<thead data-nodeid="166">
<tr data-nodeid="167">
<th data-org-content="**属性值**" data-nodeid="169"><strong data-nodeid="375">属性值</strong></th>
<th data-org-content="**说明**" data-nodeid="170"><strong data-nodeid="379">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="173">
<tr data-nodeid="174">
<td data-org-content="ROUND_ROBIN" data-nodeid="175">ROUND_ROBIN</td>
<td data-org-content="默认的轮换策略（Round Robin）" data-nodeid="176">默认的轮换策略（Round Robin）</td>
</tr>
<tr data-nodeid="177">
<td data-org-content="LEAST_CONN" data-nodeid="178">LEAST_CONN</td>
<td data-org-content="随机选择 2 个主机，选中其中活动请求数较少的" data-nodeid="179">随机选择 2 个主机，选中其中活动请求数较少的</td>
</tr>
<tr data-nodeid="180">
<td data-org-content="RANDOM" data-nodeid="181">RANDOM</td>
<td data-org-content="随机选择一个主机" data-nodeid="182">随机选择一个主机</td>
</tr>
<tr data-nodeid="183">
<td data-org-content="PASSTHROUGH" data-nodeid="184">PASSTHROUGH</td>
<td data-org-content="由调用者指定转发的地址，实际上不进行负载均衡" data-nodeid="185">由调用者指定转发的地址，实际上不进行负载均衡</td>
</tr>
</tbody>
</table>
<p data-nodeid="186">第二种是使用 consistentHash 属性设置基于哈希算法的负载均衡，这种方式可以实现会话亲和（Session Affinity）的负载均衡。哈希算法的输入可以是 HTTP 请求中的头、Cookie、查询参数的值，或是请求的 IP 地址。一个典型的场景是使用保存会话 ID 的 Cookie 来进行负载均衡。</p>
<p data-nodeid="187">第三种是使用 localityLbSetting 属性设置的基于地理位置的负载均衡。如果服务的集群横跨多个地理位置，那么可以通过这种策略来设置流量的分配。</p>
<h4 data-nodeid="188">服务的子集</h4>
<p data-nodeid="189">一个服务的终端可以被划分成多个子集。典型的场景是同一个服务的不同版本同时运行，每个版本的终端组成一个子集，每个子集可以使用下表中的属性来配置：</p>
<table data-nodeid="191">
<thead data-nodeid="192">
<tr data-nodeid="193">
<th data-org-content="**属性**" data-nodeid="195"><strong data-nodeid="399">属性</strong></th>
<th data-org-content="**说明**" data-nodeid="196"><strong data-nodeid="403">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="199">
<tr data-nodeid="200">
<td data-org-content="name" data-nodeid="201">name</td>
<td data-org-content="子集的名称" data-nodeid="202">子集的名称</td>
</tr>
<tr data-nodeid="203">
<td data-org-content="labels" data-nodeid="204">labels</td>
<td data-org-content="选择子集中终端的标签" data-nodeid="205">选择子集中终端的标签</td>
</tr>
<tr data-nodeid="206">
<td data-org-content="trafficPolicy" data-nodeid="207">trafficPolicy</td>
<td data-org-content="子集特有的流量控制策略，覆盖目的地规则的相关设置" data-nodeid="208">子集特有的流量控制策略，覆盖目的地规则的相关设置</td>
</tr>
</tbody>
</table>
<p data-nodeid="209">在下面代码的目的地规则中，定义了服务的两个子集，分别是 v1 和 v2，对应于服务的两个版本。每个子集的选择器通过标签来选择对应的主机。</p>
<pre class="lang-yaml" data-nodeid="210"><code data-language="yaml"><span class="hljs-string">apiVersion:</span>&nbsp;<span class="hljs-string">networking.istio.io/v1beta1</span>
<span class="hljs-string">kind:</span>&nbsp;<span class="hljs-string">DestinationRule</span>
<span class="hljs-attr">metadata:</span>
&nbsp;&nbsp;<span class="hljs-string">name:</span>&nbsp;<span class="hljs-string">address-service</span>
<span class="hljs-attr">spec:</span>
&nbsp;&nbsp;<span class="hljs-string">host:</span>&nbsp;<span class="hljs-string">address-service.happyride.svc.cluster.local</span>
&nbsp;&nbsp;<span class="hljs-attr">trafficPolicy:</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">loadBalancer:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">simple:</span>&nbsp;<span class="hljs-string">LEAST_CONN</span>
&nbsp;&nbsp;<span class="hljs-attr">subsets:</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">-</span>&nbsp;<span class="hljs-string">name:</span>&nbsp;<span class="hljs-string">v1</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">labels:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">app.vividcode.io/service-version:</span>&nbsp;<span class="hljs-string">"1"</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">-</span>&nbsp;<span class="hljs-string">name:</span>&nbsp;<span class="hljs-string">v2</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">labels:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">app.vividcode.io/service-version:</span>&nbsp;<span class="hljs-string">"2"</span>
</code></pre>
<p data-nodeid="211">在添加了目的地规则之后，可以配置虚拟服务在不同的子集之间分配流量。在下面的代码中，对服务的请求会被平均分配到两个服务的版本中。</p>
<pre class="lang-yaml" data-nodeid="212"><code data-language="yaml"><span class="hljs-string">apiVersion:</span>&nbsp;<span class="hljs-string">networking.istio.io/v1beta1</span>
<span class="hljs-string">kind:</span>&nbsp;<span class="hljs-string">VirtualService</span>
<span class="hljs-attr">metadata:</span>
&nbsp;&nbsp;<span class="hljs-string">name:</span>&nbsp;<span class="hljs-string">address-service-versioned</span>
<span class="hljs-attr">spec:</span>
&nbsp;&nbsp;<span class="hljs-attr">hosts:</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">-</span>&nbsp;<span class="hljs-string">address-service.happyride.svc.cluster.local</span>
&nbsp;&nbsp;<span class="hljs-attr">http:</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">-</span>&nbsp;<span class="hljs-string">name:</span>&nbsp;<span class="hljs-string">"address-service-http"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">route:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">-</span>&nbsp;<span class="hljs-attr">destination:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">host:</span>&nbsp;<span class="hljs-string">address-service.happyride.svc.cluster.local</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">subset:</span>&nbsp;<span class="hljs-string">v1</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">weight:</span>&nbsp;<span class="hljs-number">50</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">-</span>&nbsp;<span class="hljs-attr">destination:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">host:</span>&nbsp;<span class="hljs-string">address-service.happyride.svc.cluster.local</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">subset:</span>&nbsp;<span class="hljs-string">v2</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">weight:</span>&nbsp;<span class="hljs-number">50</span>
</code></pre>
<h3 data-nodeid="213">网关</h3>
<p data-nodeid="214">上面对虚拟服务的使用，都是用来定义服务网格内部的流量控制策略，对于外部的访问请求，则需要通过网关来控制。Istio 中有两种类型的网关，分别是<strong data-nodeid="422">入口网关</strong>和<strong data-nodeid="423">出口网关</strong>。</p>
<h4 data-nodeid="215">入口网关</h4>
<p data-nodeid="216">入口网关用来控制进入到服务网格中的请求，它本身也是 Istio 中的组件，在默认的安装中，该组件已经启用。入口网关实际上也是一个 Istio 的服务代理，只不过处理的是外部请求。</p>
<p data-nodeid="217">网关使用 Istio 的自定义资源 Gateway 来描述。下面的代码是网关的示例：</p>
<pre class="lang-yaml" data-nodeid="218"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">networking.istio.io/v1beta1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Gateway</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">app-gateway</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">selector:</span>
    <span class="hljs-attr">app:</span> <span class="hljs-string">istio-ingressgateway</span>
  <span class="hljs-attr">servers:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">port:</span>
        <span class="hljs-attr">number:</span> <span class="hljs-number">80</span>
        <span class="hljs-attr">name:</span> <span class="hljs-string">http</span>
        <span class="hljs-attr">protocol:</span> <span class="hljs-string">HTTP</span>
      <span class="hljs-attr">hosts:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-string">"happyride.com"</span>
</code></pre>
<p data-nodeid="5554">其中， selector 用来选择提供网关服务的入口网关组件，这里使用的是 Istio 默认的入口网关组件；servers 则定义了网关可供外部请求访问的服务。每个服务中的 port 属性声明了服务的端口，包括端口的名称、端口号和协议；hosts 属性则声明了所暴露的主机名。</p>


<p data-nodeid="221">主机名可以包含名称空间作为前缀，也可以使用通配符，比如 happyride/* 表示 happyride 名称空间下的全部主机名。除了实际的名称空间之外，还可以使用“ * ”和“ . ”，分别表示任意名称空间和当前名称空间，在省略名称空间时，默认为“ * ”。上面代码中的主机名是 happyride.com，表示任意名称空间中的 happyride.com 主机。</p>
<p data-nodeid="222">虚拟服务需要与网关绑定，并且虚拟服务的主机名必须与网关暴露的主机名相匹配。下面代码中的虚拟服务用来暴露地址管理服务和乘客管理服务的 API，其中的 gateways 属性用来指定该虚拟服务所绑定的网关名称，并且 hosts 属性声明的主机名与对应网关的主机声明是相同的。虚拟服务定义了两个 HTTP 路由，分别对应于两个服务，每个路由与一个 URI 前缀相对应。而rewrite 的作用则是在请求发送到目标服务之前，改写 URI 来去掉前缀。</p>
<pre class="lang-yaml" data-nodeid="223"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">networking.istio.io/v1beta1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">VirtualService</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">app-service</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">hosts:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-string">happyride.com</span>
  <span class="hljs-attr">gateways:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-string">app-gateway</span>  
  <span class="hljs-attr">http:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">"address-service-external"</span>
      <span class="hljs-attr">match:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">uri:</span>
            <span class="hljs-attr">prefix:</span> <span class="hljs-string">"/address"</span>
      <span class="hljs-attr">rewrite:</span>
        <span class="hljs-attr">uri:</span> <span class="hljs-string">"/"</span>         
      <span class="hljs-attr">route:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">destination:</span>
            <span class="hljs-attr">host:</span> <span class="hljs-string">address-service.happyride.svc.cluster.local</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">"passenger-service-external"</span>
      <span class="hljs-attr">match:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">uri:</span>
            <span class="hljs-attr">prefix:</span> <span class="hljs-string">"/passenger"</span>
      <span class="hljs-attr">rewrite:</span>
        <span class="hljs-attr">uri:</span> <span class="hljs-string">"/"</span>         
      <span class="hljs-attr">route:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">destination:</span>
            <span class="hljs-attr">host:</span> <span class="hljs-string">passenger-service.happyride.svc.cluster.local</span>        
</code></pre>
<p data-nodeid="224">Istio 的入口网关组件会在 Kubernetes 上暴露一个类型为 LoadBalancer 的服务，这个服务会作为外部访问请求的入口。</p>
<p data-nodeid="225">下面的命令通过入口网关来访问应用的地址管理服务。这里需要设置 Host 头的值来匹配入口网关中的声明。</p>
<pre class="lang-java" data-nodeid="6017"><code data-language="java">curl -i -H <span class="hljs-string">"Host: happyride.com"</span> http:<span class="hljs-comment">//192.168.64.7:30001/address/actuator/health/liveness</span>
</code></pre>

<h4 data-nodeid="227">出口网关</h4>
<p data-nodeid="228">除了入口网关，还可以通过出口网关来控制应用往外发送的请求。但一般情况下，对于发送到外部的请求并不会限制。Istio 提供了两种模式来配置对外部请求的访问策略，并通过配置项 outboundTrafficPolicy.mode 来指定。默认的模式是 ALLOW_ANY，也就是允许发送请求给外部的未知服务；另外一个模式是 REGISTRY_ONLY，表示只允许发送给服务注册表中的服务。</p>
<p data-nodeid="229">默认的 ALLOW_ANY 模式虽然使用方便，但是存在一定的安全隐患，建议的做法是切换到 REGISTRY_ONLY 模式。</p>
<p data-nodeid="230">在 istio-system 名称空间中的配置表 istio 中包含了 Istio 的基本配置，其中的 mesh 键对应的值是一个 YAML 文件，表示全局的配置。可以通过下面的命令来检查当前的模式。如果找不到 mode 属性，则说明当前使用的默认模式。</p>
<pre class="lang-dart" data-nodeid="18016"><code data-language="dart">kubectl <span class="hljs-keyword">get</span> cm istio -n istio-system -o yaml | grep mode
</code></pre>





<p data-nodeid="232">下面的命令用来更新已有 Istio 的配置来切换到 REGISTRY_ONLY 模式。</p>
<pre class="lang-dart" data-nodeid="14324"><code data-language="dart">istioctl install --<span class="hljs-keyword">set</span> meshConfig.outboundTrafficPolicy.mode=REGISTRY_ONLY
</code></pre>








<p data-nodeid="234">在完成了修改之后，所有访问外部服务的请求都会返回 502 错误。为了正常访问，需要对外部服务进行注册，注册可通过创建服务条目资源来完成。下面的代码创建了第三方支付服务所对应的 ServiceEntry 资源，其中，location 属性的值 MESH_EXTERNAL 声明了该服务位于服务网格之外，ports 属性定义了允许访问的端口，resolution 属性表示服务代理如何解析服务的 IP 地址，值 DNS 则表示使用 DNS 解析。</p>
<pre class="lang-yaml" data-nodeid="235"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">networking.istio.io/v1beta1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">ServiceEntry</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">payment-service</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">hosts:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-string">www.alipay.com</span>
  <span class="hljs-attr">location:</span> <span class="hljs-string">MESH_EXTERNAL</span>
  <span class="hljs-attr">ports:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">number:</span> <span class="hljs-number">80</span>
      <span class="hljs-attr">name:</span> <span class="hljs-string">http</span>
      <span class="hljs-attr">protocol:</span> <span class="hljs-string">HTTP</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">number:</span> <span class="hljs-number">443</span>
      <span class="hljs-attr">name:</span> <span class="hljs-string">https</span>
      <span class="hljs-attr">protocol:</span> <span class="hljs-string">HTTPS</span>  
  <span class="hljs-attr">resolution:</span> <span class="hljs-string">DNS</span>
</code></pre>
<p data-nodeid="236">通过出口网关，可以让所有的对外部服务的请求都通过统一的网关来管理。Istio 的 egressGateway 组件本质上也是一个 Istio 代理。使用下面的命令来安装出口网关组件。</p>
<pre class="lang-java" data-nodeid="18939"><code data-language="java">istioctl install --set values.gateways.istio-egressgateway.enabled=<span class="hljs-keyword">true</span>
</code></pre>

<p data-nodeid="238">出口网关的创建与入口网关相似，也是创建 Gateway 类型的资源，只不过 selector 属性中选择的是出口网关组件。下面的代码创建了应用所使用的出口网关：</p>
<pre class="lang-yaml" data-nodeid="239"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">networking.istio.io/v1beta1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Gateway</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">app-external</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">selector:</span>
    <span class="hljs-attr">app:</span> <span class="hljs-string">istio-egressgateway</span>
  <span class="hljs-attr">servers:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">port:</span>
        <span class="hljs-attr">number:</span> <span class="hljs-number">80</span>
        <span class="hljs-attr">name:</span> <span class="hljs-string">http</span>
        <span class="hljs-attr">protocol:</span> <span class="hljs-string">HTTP</span>
      <span class="hljs-attr">hosts:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-string">"*"</span>
</code></pre>
<p data-nodeid="240">为了使用出口网关，需要创建外部服务对应的虚拟服务资源来与网关进行绑定。下面的代码用来创建相应的 VirtualService 资源，其中，gateways 属性中包含了两个网关名称——除了应用本身的网关之外，还有一个名为 mesh 的网关。网关 mesh 表示服务网格本身。</p>
<p data-nodeid="241">当虚拟服务的 gateways 属性中包含 mesh 时，则表明该虚拟服务的规则对于服务网格内部的请求也是适用的。也就是说，当服务网格内部的代码访问支付服务时，根据虚拟服务的规则，该请求会被路由给 Istio 的出口网关；如果请求已经达到了出口网关，那么就会直接发送给外部服务。</p>
<pre class="lang-yaml" data-nodeid="242"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">networking.istio.io/v1beta1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">VirtualService</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">payment-serivce</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">hosts:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-string">www.alipay.com</span>
  <span class="hljs-attr">exportTo:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-string">"*"</span>
  <span class="hljs-attr">gateways:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-string">mesh</span>
    <span class="hljs-bullet">-</span> <span class="hljs-string">app-external</span>
  <span class="hljs-attr">http:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">match:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">port:</span> <span class="hljs-number">80</span>
          <span class="hljs-attr">gateways:</span>
            <span class="hljs-bullet">-</span> <span class="hljs-string">mesh</span>
      <span class="hljs-attr">route:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">destination:</span>
            <span class="hljs-attr">host:</span> <span class="hljs-string">istio-egressgateway.istio-system.svc.cluster.local</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">match:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">port:</span> <span class="hljs-number">80</span>
          <span class="hljs-attr">gateways:</span>
            <span class="hljs-bullet">-</span> <span class="hljs-string">app-external</span>
      <span class="hljs-attr">route:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">destination:</span>
            <span class="hljs-attr">host:</span> <span class="hljs-string">www.alipay.com</span>
</code></pre>
<p data-nodeid="243">由于支付服务使用的是 HTTPS 协议，所以还需要对支付服务添加目的地规则。在下面的代码中，第一个虚拟服务资源用来把发送到支付服务的 80 端口请求，改写成发送到 443 端口；第二个目的地规则资源则对支付服务启用 TLS 支持。</p>
<pre class="lang-yaml" data-nodeid="244"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">networking.istio.io/v1alpha3</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">VirtualService</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">rewrite-port-for-alipay</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">hosts:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">www.alipay.com</span>
  <span class="hljs-attr">http:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-attr">match:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">port:</span> <span class="hljs-number">80</span>
    <span class="hljs-attr">route:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">destination:</span>
        <span class="hljs-attr">host:</span> <span class="hljs-string">www.alipay.com</span>
        <span class="hljs-attr">port:</span>
          <span class="hljs-attr">number:</span> <span class="hljs-number">443</span>
<span class="hljs-meta">---</span>
<span class="hljs-attr">apiVersion:</span> <span class="hljs-string">networking.istio.io/v1beta1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">DestinationRule</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">payment-service</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">host:</span> <span class="hljs-string">www.alipay.com</span>
  <span class="hljs-attr">trafficPolicy:</span>
    <span class="hljs-attr">loadBalancer:</span>
      <span class="hljs-attr">simple:</span> <span class="hljs-string">ROUND_ROBIN</span>
    <span class="hljs-attr">portLevelSettings:</span>
      <span class="hljs-bullet">-</span> <span class="hljs-attr">port:</span>
          <span class="hljs-attr">number:</span> <span class="hljs-number">443</span>
        <span class="hljs-attr">tls:</span>
          <span class="hljs-attr">mode:</span> <span class="hljs-string">SIMPLE</span>
</code></pre>
<h3 data-nodeid="245">总结</h3>
<p data-nodeid="19862">通过 Istio 的服务网格提供的流量控制功能，可以实现复杂的服务管理的场景。通过本课时的学习，你可以了解 Istio 中虚拟服务和目的地规则等资源的使用方式，以及如何使用入口网关和出口网关来分别控制进入服务网格的请求，以及从服务网格发出的请求。</p>