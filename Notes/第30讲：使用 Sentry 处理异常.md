<p data-nodeid="9094" class="">异常是 Java 应用中处理错误的标准方式。在捕获异常时，通常的做法是以日志的方式记录下来,可以使用第 29 课时介绍的日志聚合技术栈来处理异常。</p>
<p data-nodeid="9095">但是异常中包含了很多与代码相关的信息，尤其是异常的堆栈信息，对错误调试很有帮助。如果只是把这些异常消息当成普通的日志消息，则没办法将其充分利用，更好的做法应该是对异常进行特殊的处理，也就是本课时会主要讲解的内容。</p>
<h3 data-nodeid="9096">Java 中的异常</h3>
<p data-nodeid="9097">在 Java 开发中，总是免不了与异常打交道，异常表示的是错误的情况。</p>
<h4 data-nodeid="9098">1.异常的类型</h4>
<p data-nodeid="10388">Java 中的异常可以分成 3 类，分别是检查异常（Checked Exception）、非检查异常（Unchecked Exception）和错误（Error）。这三种异常都是 Throwable 的子类型，类层次结构如下图所示。</p>
<p data-nodeid="10389" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/27/71/CgqCHl714nqAF02gAABfHmw5Su8570.png" alt="002.png" data-nodeid="10393"></p>


<p data-nodeid="9101">只有 Throwable 类或者其子类的实例，才能被 Java 虚拟机以错误来抛出，或是作为 throw 语句的对象。同样的，只有 Throwable 类及其子类才能作为 catch 子句的类型。</p>
<p data-nodeid="9102">Error 类表示的是严重的系统错误，应用程序不应该试图去捕获和处理这样的错误。这样的错误通常表示异常的情况，应该由虚拟机来处理，只能终止程序的执行。常见的错误包括表示内存不足的 OutOfMemoryError、表示类链接错误的 LinkageError 及其子类、表示 IO 错误的 IOError等。Error 类型是保留给虚拟机使用的，应用程序不应该创建自定义的 Error 类的子类。</p>
<p data-nodeid="9103">与 Error 相对应的 Exception 类，表示的是程序可以捕获和处理的异常情况，异常可分成检查异常和非检查异常两类。这两者的区别在于，检查异常的使用由编译器来检查，而非检查异常则不需要。Throwable 类及其子类，如果不是 Error 或 RuntimeException 及它们的子类型，则被视为检查异常，否则是非检查异常。当检查异常出现在方法声明的 throws 子句中时，该方法的调用者必须对声明的异常进行处理，要么使用 catch 子句来捕获并处理该异常，要么把异常往上传递。非检查异常则没有这样的限制，可以自由地抛出和捕获。</p>
<h4 data-nodeid="9104">2.异常对象</h4>
<p data-nodeid="9105">Throwable 对象在创建时有两个基本的参数，其中一个是 String 类型的 message，表示概要性的描述；另外一个是 Throwable 类型的 cause，表示导致当前 Throwable 对象产生的原因。由于每个 Throwable 对象都可以有自己的原因。多个 Throwable 对象可以通过这种关系串联起来，形成异常链。</p>
<p data-nodeid="9106">在出现异常时，异常对象是获取错误信息的最重要的渠道。因此，异常类中应该包含足够多的信息来描述错误出现时的情况。这一点与日志记录是相似的，其目的都是为了方便开发人员查找错误的根源。异常类除了必须继承自 Throwable 类之外，与其他的 Java 类并没有区别。异常类也可以添加不同的属性和方法。</p>
<p data-nodeid="9107">下面代码中的 OrderNotFoundException 类表示找不到指定标识符对应的订单，其构造参数orderId 表示订单的标识符。当 OrderNotFoundException 异常被抛出时，可以利用异常消息中包含的订单标识符快速查找问题。</p>
<pre class="lang-java" data-nodeid="9108"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OrderNotFoundException</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Exception</span> </span>{
&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String orderId;
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">OrderNotFoundException</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String orderId)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">super</span>(String.format(<span class="hljs-string">"Order %s not found"</span>, orderId));
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.orderId = orderId;
&nbsp; }
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getOrderId</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.orderId;
&nbsp; }
}
</code></pre>
<h4 data-nodeid="9109">3.异常翻译</h4>
<p data-nodeid="9110">当一个方法中使用 throws 子句声明了它所能抛出的异常之后，这些异常就成了这个方法的公开 API 的一部分。从抽象的层次来说，一个方法抛出的异常的抽象层次，应该与该方法的抽象层次互相匹配。举例来说，服务层的方法在实现中需要调用数据访问层的代码，数据访问层的代码可能抛出相应的异常，服务层的方法需要捕获这些异常，并翻译成服务层所对应的异常。这就需要用到上面提到的异常链。</p>
<p data-nodeid="9111">在下面的代码中，MyDataAccess 类的 getData 方法会抛出 DataAccessException 异常。在 MyService 类的 service 方法中，DataAccessException 异常被捕获，并翻译成服务层的MyServiceException 异常。DataAccessException 异常对象则作为 MyServiceException 异常的原因。</p>
<pre class="lang-java" data-nodeid="9112"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyService</span> </span>{
&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp; MyDataAccess dataAccess;
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">service</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> MyServiceException </span>{
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.dataAccess.getData();
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (<span class="hljs-keyword">final</span> DataAccessException e) {
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> MyServiceException(<span class="hljs-string">"Failed to get data"</span>, e);
&nbsp; &nbsp; }
&nbsp; }
}
</code></pre>
<p data-nodeid="9113">通过异常翻译，既可以保证异常的抽象层次，又可以保留错误产生的追踪信息。</p>
<h4 data-nodeid="9114">4.检查异常与非检查异常</h4>
<p data-nodeid="9115">在设计 Java 的 API 时，一个常见的讨论是检查异常和非检查异常应该在什么时候使用。关于这一点，社区中有很多不同的观点，不同的开发团队也可能选择不同的策略。检查异常的特点在于它们的使用由编译器来强制保证。如果不使用 try-catch 来捕获异常，或是重新抛出异常，代码无法通过编译。</p>
<p data-nodeid="9116">检查异常的这种特点，如果应用得当，会是代码调用者的一大助力，可以让调用者清楚地了解到可能出现的错误情况，并加以处理。而如果使用不当，则会让调用者感觉到很困扰。设计不好的检查异常可能会产生不好的使用模式。</p>
<p data-nodeid="9117">一种常见的情况是，调用者除了忽略异常之外，没有别的处理方法。一个典型的例子是 Java 标准库中的 java.net.URLEncoder 类的 encode 方法。这个方法的一个参数是进行编码的字符集名称，而这个方法会抛出检查异常 UnsupportedEncodingException，也是因为可能找不到指定的字符集。实际上，绝大多数情况下都会使用 UTF-8 作为字符集，而 UTF-8 属于必须支持的字符集，因此这个 UnsupportedEncodingException 不可能出现。</p>
<p data-nodeid="9118">对于这样的情况，一种做法是对原始方法进行封装，去掉检查异常，如下面的代码所示。另外一种做法是把原来的方法拆分成两个方法，其中一个方法用来判断是否可以进行编码，另外一个方法进行编码，但是不抛出检查异常。</p>
<pre class="lang-java" data-nodeid="9119"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> String <span class="hljs-title">encode</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String input)</span> </span>{
&nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; <span class="hljs-keyword">return</span> URLEncoder.encode(input, <span class="hljs-string">"UTF-8"</span>);
&nbsp; } <span class="hljs-keyword">catch</span> (<span class="hljs-keyword">final</span> UnsupportedEncodingException ignored) {
&nbsp; &nbsp; <span class="hljs-comment">// 不会出现的情况</span>
&nbsp; }
&nbsp; <span class="hljs-keyword">return</span> input;
}
</code></pre>
<p data-nodeid="9120">使用检查异常的目的是希望调用者可以从错误中恢复。比如，当从文件中读取系统的配置时，如果出现 IOException，则可以使用默认配置。非检查异常用来表示程序运行中的错误。由于非检查异常并不强制进行处理，在使用时会方便一些。比如，Integer.parseInt 会抛出非检查异常NumberFormatException。如果输入的字符串来自内部，则可以忽略对该异常的处理；如果来自外部的用户输入，则需要处理该异常。由于非检查异常的这种灵活性，一般的观点是认为应该优先使用非检查异常。在 Kotlin 中，所有的异常都是非检查的。检查异常的另外一个劣势在于不能与 Java 流 API 一同使用。</p>
<h4 data-nodeid="9121">5.异常处理的原则</h4>
<p data-nodeid="9122">首先是不要忽略异常。使用 try-catch 捕获异常之后，不做任何处理是一种危险的做法，会造成意想不到的问题。如果确定异常不可能产生，应该在 catch 子句中添加注释来说明忽略该异常的理由，如上面代码中对 UnsupportedEncodingException 异常的处理。另外一种更加安全的做法是添加日志记录，至少可以保留异常的相关信息。</p>
<p data-nodeid="9123">另外一个原则是避免多长的异常链。当异常链过长时，在输出堆栈信息或记录日志时，会占用过多的空间。在很多时候，异常链的根本原因就已经足够了。只需要通过 Throwable 的 getCause方法来遍历异常链，并找到作为根的 Throwable 对象即可。更简单的做法是使用 Guava 中Throwables 类的 getRootCause 方法。</p>
<h3 data-nodeid="9124">使用 Sentry</h3>
<p data-nodeid="9125"><a href="https://github.com/getsentry/sentry" data-nodeid="9243">Sentry</a> 是开源的记录应用错误的服务。对于应用来说，既可以使用 Sentry.io 提供的在线服务，也可以在自己的服务器上部署。Sentry 提供了<a href="https://hub.docker.com/r/getsentry/sentry/" data-nodeid="9247">容器镜像</a>，在 Kubernetes 上运行部署也很简单。</p>
<h4 data-nodeid="9126">1.配置</h4>
<p data-nodeid="9127">如果使用 Sentry.io 提供的在线服务，在使用之前，首先需要注册账号和创建新的项目。在项目的配置界面中，可以找到客户端秘钥（DSN）。这个秘钥是 Sentry SDK 发送数据到服务器所必需的。复制该 DSN 的值，并以系统属性 sentry.dsn 或环境变量 SENTRY_DSN 传递给应用。</p>
<p data-nodeid="9128">除了 DSN 之外，Sentry 还提供了很多配置项。这些配置项可以添加在 CLASSPATH 上的sentry.properties 文件中，也可以通过 Java 系统属性或环境变量来传递。所有的系统属性都以 sentry. 开头，而环境变量都以 SENTRY_ 开头。下表给出了常用的配置项。当需要提供配置项 environment 的值时，可以使用系统属性 sentry.environment 或环境变量SENTRY_ENVIRONMENT。</p>
<table data-nodeid="9130">
<thead data-nodeid="9131">
<tr data-nodeid="9132">
<th data-org-content="配置项" data-nodeid="9134">配置项</th>
<th data-org-content="说明" data-nodeid="9135">说明</th>
</tr>
</thead>
<tbody data-nodeid="9138">
<tr data-nodeid="9139">
<td data-org-content="release" data-nodeid="9140">release</td>
<td data-org-content="应用的版本号" data-nodeid="9141">应用的版本号</td>
</tr>
<tr data-nodeid="9142">
<td data-org-content="environment" data-nodeid="9143">environment</td>
<td data-org-content="当前的运行环境，如开发、测试、交付准备或生产环境" data-nodeid="9144">当前的运行环境，如开发、测试、交付准备或生产环境</td>
</tr>
<tr data-nodeid="9145">
<td data-org-content="servername" data-nodeid="9146">servername</td>
<td data-org-content="服务器主机名" data-nodeid="9147">服务器主机名</td>
</tr>
<tr data-nodeid="9148">
<td data-org-content="tags" data-nodeid="9149">tags</td>
<td data-org-content="附加的标签名值对，以 tag1:value1,tag2:value2 的形式传递" data-nodeid="9150">附加的标签名值对，以 tag1:value1,tag2:value2 的形式传递</td>
</tr>
<tr data-nodeid="9151">
<td data-org-content="extra" data-nodeid="9152">extra</td>
<td data-org-content="附加的数据，以 key1:value1,key2:value2 的形式传递" data-nodeid="9153">附加的数据，以 key1:value1,key2:value2 的形式传递</td>
</tr>
<tr data-nodeid="9154">
<td data-org-content="stacktrace.app.packages" data-nodeid="9155">stacktrace.app.packages</td>
<td data-org-content="异常堆栈信息中，属于应用代码的包的名称" data-nodeid="9156">异常堆栈信息中，属于应用代码的包的名称</td>
</tr>
<tr data-nodeid="9157">
<td data-org-content="stacktrace.hidecommon" data-nodeid="9158">stacktrace.hidecommon</td>
<td data-org-content="异常堆栈信息中，隐藏异常链中非应用代码的帧" data-nodeid="9159">异常堆栈信息中，隐藏异常链中非应用代码的帧</td>
</tr>
<tr data-nodeid="9160">
<td data-org-content="uncaught.handler.enabled" data-nodeid="9161">uncaught.handler.enabled</td>
<td data-org-content="处理并发送未捕获的异常" data-nodeid="9162">处理并发送未捕获的异常</td>
</tr>
</tbody>
</table>
<p data-nodeid="9163">Sentry 的大部分配置项都与运行环境有关，因此不适合放在 sentry.properties 文件中，而是在运行时由底层平台来提供。</p>
<p data-nodeid="9164">在使用 Sentry 之前，需要在 Java 应用的 main 方法中调用 Sentry.init 方法来进行初始化。</p>
<h4 data-nodeid="9165">2.集成日志实现</h4>
<p data-nodeid="9166">Sentry 提供了对日志实现框架的集成。在记录日志时，通常都会记录下相关的异常对象。通过与 Sentry 的集成，日志事件中包含的异常会被自动发送到 Sentry。以 Log4j 2 来说，只需要配置使用 Sentry 的日志输出源即可，如下面的代码所示。</p>
<pre class="lang-xml" data-nodeid="9167"><code data-language="xml"><span class="hljs-meta">&lt;?xml version="1.0" encoding="UTF-8"?&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">configuration</span> <span class="hljs-attr">status</span>=<span class="hljs-string">"warn"</span>&gt;</span>
&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">appenders</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">Console</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"Console"</span> <span class="hljs-attr">target</span>=<span class="hljs-string">"SYSTEM_OUT"</span>&gt;</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">PatternLayout</span> <span class="hljs-attr">pattern</span>=<span class="hljs-string">"%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"</span>/&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">Console</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">Sentry</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"Sentry"</span>/&gt;</span>
&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">appenders</span>&gt;</span>
&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">loggers</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">root</span> <span class="hljs-attr">level</span>=<span class="hljs-string">"INFO"</span>&gt;</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">appender-ref</span> <span class="hljs-attr">ref</span>=<span class="hljs-string">"Console"</span>/&gt;</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">appender-ref</span> <span class="hljs-attr">ref</span>=<span class="hljs-string">"Sentry"</span> <span class="hljs-attr">level</span>=<span class="hljs-string">"WARN"</span>/&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">root</span>&gt;</span>
&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">loggers</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">configuration</span>&gt;</span>
</code></pre>
<h4 data-nodeid="9168">3.手动记录事件</h4>
<p data-nodeid="9169">除了与日志实现集成之外，还可以通过 Sentry 的 Java 客户端来手动记录事件。使用Sentry.capture 方法可以记录不同类型的事件，包括 String、Throwable 和 Sentry 的 Event对象。Event 是 Sentry 提供的事件 POJO 类，包含了事件所具备的属性，可以从 EventBuilder 构建器中创建。</p>
<p data-nodeid="9170">在下面的代码中，通过 EventBuilder 创建出新的 Event 对象，并由 Sentry.capture 方法来记录。</p>
<pre class="lang-java" data-nodeid="9171"><code data-language="java">Sentry.capture(<span class="hljs-keyword">new</span> EventBuilder()
&nbsp; &nbsp; .withMessage(<span class="hljs-string">"Test message"</span>)
    .withLevel(Level.WARNING)
&nbsp; &nbsp; .withTag(<span class="hljs-string">"tag1"</span>, <span class="hljs-string">"value1"</span>)
&nbsp; &nbsp; .withExtra(<span class="hljs-string">"key1"</span>, <span class="hljs-string">"value1"</span>)
);
</code></pre>
<p data-nodeid="9172">除了直接使用 Event 对象之外，事件中包含的数据，还可以来自当前的上下文。默认情况下，Sentry 使用 ThreadLocal 来保存上下文对象，与当前的线程关联起来，这一点与日志实现中的MDC 是相同的。在发送事件到 Sentry 时，事件会自动包含当前上下文中的内容。</p>
<p data-nodeid="9173">在下面的代码中，通过 Sentry.getContext 方法可以获取到当前的上下文对象，并对其中的数据进行修改。</p>
<pre class="lang-java" data-nodeid="9174"><code data-language="java">Sentry.getContext().setUser(
&nbsp; &nbsp; <span class="hljs-keyword">new</span> UserBuilder().setUsername(<span class="hljs-string">"test"</span>).setEmail(<span class="hljs-string">"test@test.com"</span>).build()
);
Sentry.getContext().addTag(<span class="hljs-string">"tag1"</span>, <span class="hljs-string">"value1"</span>);
Sentry.getContext().addExtra(<span class="hljs-string">"key1"</span>, <span class="hljs-string">"value1"</span>);
Sentry.capture(<span class="hljs-string">"A new message"</span>);
</code></pre>
<p data-nodeid="9175">Sentry 还支持收集一种名为面包屑（Breadcrumb）的数据，表示一些相关的事件。面包屑中可以包含下表中的属性。</p>
<table data-nodeid="9177">
<thead data-nodeid="9178">
<tr data-nodeid="9179">
<th data-org-content="属性" data-nodeid="9181">属性</th>
<th data-org-content="说明" data-nodeid="9182">说明</th>
</tr>
</thead>
<tbody data-nodeid="9185">
<tr data-nodeid="9186">
<td data-org-content="message" data-nodeid="9187">message</td>
<td data-org-content="事件的消息" data-nodeid="9188">事件的消息</td>
</tr>
<tr data-nodeid="9189">
<td data-org-content="data" data-nodeid="9190">data</td>
<td data-org-content="以哈希表表示的元数据" data-nodeid="9191">以哈希表表示的元数据</td>
</tr>
<tr data-nodeid="9192">
<td data-org-content="category" data-nodeid="9193">category</td>
<td data-org-content="事件的类别" data-nodeid="9194">事件的类别</td>
</tr>
<tr data-nodeid="9195">
<td data-org-content="level" data-nodeid="9196">level</td>
<td data-org-content="事件的严重性级别" data-nodeid="9197">事件的严重性级别</td>
</tr>
<tr data-nodeid="9198">
<td data-org-content="type" data-nodeid="9199">type</td>
<td data-org-content="事件的类型" data-nodeid="9200">事件的类型</td>
</tr>
</tbody>
</table>
<p data-nodeid="9201">在下面的代码中，通过上下文对象的 recordBreadcrumb 方法可以记录 Breadcrumb 对象。</p>
<pre class="lang-java" data-nodeid="9202"><code data-language="java">Sentry.getContext().recordBreadcrumb(
&nbsp; &nbsp; <span class="hljs-keyword">new</span> BreadcrumbBuilder()
&nbsp; &nbsp; &nbsp; &nbsp; .setMessage(<span class="hljs-string">"Event 123"</span>)
&nbsp; &nbsp; &nbsp; &nbsp; .setData(ImmutableMap.of(<span class="hljs-string">"key1"</span>, <span class="hljs-string">"value1"</span>))
&nbsp; &nbsp; &nbsp; &nbsp; .setCategory(<span class="hljs-string">"test"</span>)
&nbsp; &nbsp; &nbsp; &nbsp; .setLevel(Breadcrumb.Level.INFO)
&nbsp; &nbsp; &nbsp; &nbsp; .setType(Type.DEFAULT)
&nbsp; &nbsp; &nbsp; &nbsp; .build());
Sentry.capture(<span class="hljs-string">"A message with breadcrumb"</span>);
</code></pre>
<h4 data-nodeid="9203">4.用户界面</h4>
<p data-nodeid="9204">捕获的事件可以通过 Sentry 的界面来查看，下图给出了 Sentry 中查看问题列表的界面。</p>
<p data-nodeid="9205"><img src="https://s0.lgstatic.com/i/image/M00/26/CE/CgqCHl7y-reAPzbkAADpk0eyBgI056.png" alt="sentry-list.png" data-nodeid="9303"></p>
<p data-nodeid="9206">对于每个问题，可以查看详细信息，如下图所示。</p>
<p data-nodeid="9207"><img src="https://s0.lgstatic.com/i/image/M00/26/CE/CgqCHl7y-r-AdEmEAAHmAQ4V61I139.png" alt="sentry-details.png" data-nodeid="9307"></p>
<p data-nodeid="9208">Sentry 界面所提供的功能很强大，可以帮助开发人员快速获取相关信息。</p>
<h3 data-nodeid="9209">总结</h3>
<p data-nodeid="9210">通过记录 Java 应用运行中产生的异常，可以方便开发人员查找问题的根源。通过本课时的学习，你可以掌握 Java 中使用异常的基本知识和相关实践细节，包括检查异常和非检查异常的使用和异常处理的原则等，还可以了解到如何使用 Sentry 来记录异常和发布相关的事件。</p>