<p>本课时将介绍“如何实现 REST 服务的集成测试”。</p>
<p>在第 12 课时中，介绍了微服务实现中的单元测试。单元测试只对单个对象进行测试，被测试对象所依赖的其他对象，一般使用 mock 对象来代替，单元测试可以确保每个对象的行为满足对它的期望。不过，这些对象集成在一起之后的行为是否正确，则需要由另外的测试来验证。这就是集成测试要解决的问题，与单元测试不同的是，集成测试一般由测试人员编写。本课时将介绍如何进行服务的集成测试。</p>
<p>集成测试的目标是微服务本身，也就是说，把整个微服务看成是一个黑盒子，只通过微服务暴露的接口来进行测试，这个暴露的接口就是微服务的 API。对微服务的集成测试，实际上是对其 API 的测试。由于本专栏的微服务使用的是 REST API，所以着重介绍 REST API 的集成测试。</p>
<p>REST API 的测试本身并不复杂，只需要发送特定的 HTTP 请求到 REST API 服务器，再验证 HTTP 响应的内容即可。在 Java 中，已经有 Apache HttpClient 和 OkHttp 这样的 HTTP 客户端，可以帮助在 Java 代码中发送 HTTP 请求。在测试中，推荐使用 <a href="http://rest-assured.io/">REST-assured</a> 这样的工具来验证 REST API。</p>
<p>我们可以利用 Spring Boot 提供的集成测试支持来测试微服务的 REST API。在进行测试时，Spring Boot 可以在一个随机端口启动 REST API 服务器来作为测试的目标。本课时选择的测试目标是乘客管理服务的 REST API。</p>
<p>下面介绍 3 种不同的测试 REST API 的方式。</p>
<h4>手动发送 HTTP 请求</h4>
<p>我们可以用 Spring Test 提供了的 WebTestClient 来编写 REST API 的验证代码。下面代码中的 PassengerControllerTest 类是乘客管理服务 REST API 的测试用例。</p>
<p>PassengerControllerTest 类上的注解与第 12 课时中出现的 PassengerServiceTest 类上的注解存在很多相似性，一个显著的区别是 @SpringBootTest 注解中 webEnvironment 属性的值 WebEnvironment.RANDOM_PORT。这使得 Spring Boot 可以在一个随机的端口上启动 REST API 服务。PassengerControllerTest 类通过依赖注入的方式声明了 WebTestClient 类和 TestRestTemplate 类对象。</p>
<p>WebTestClient 类提供了一个简单的 API 来发送 HTTP 请求并验证响应。在 testCreatePassenger 方法中，使用了 WebTestClient 类的 post 方法发送一个 POST 请求来创建乘客，并验证 HTTP 响应的状态码和 HTTP 头 Location。</p>
<p>TestRestTemplate 类则用来获取 HTTP 响应的内容。在 testRemoveAddress 方法中，TestRestTemplate 类的 postForLocation 方法用来发送 POST 请求，并获取到 HTTP 头 Location 的内容，该内容是访问新创建的乘客对象的 URL。TestRestTemplate 类的 getForObject 方法访问这个 URL 并获取到表示乘客的 PassengerVO 对象。从 PassengerVO 对象中找到乘客的第一个地址的 ID，再构建出访问该地址的 URL。WebTestClient 类的 delete 方法用来发送 DELETE 请求到该地址对应的 URL，接着使用 WebTestClient 类的 get 方法获取乘客信息并验证地址数量。</p>
<pre><code data-language="java" class="lang-java">@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@EnableAutoConfiguration
@ComponentScan
@Import(EmbeddedPostgresConfiguration.class)
@ImportAutoConfiguration(classes = {
    EmbeddedPostgreSQLDependenciesAutoConfiguration.class,
    EmbeddedPostgreSQLBootstrapConfiguration.class
})
@TestPropertySource(properties = {
    "embedded.postgresql.docker-image=postgres:12-alpine"
})
@DisplayName("Passenger controller test")
public class PassengerControllerTest {

  private final String baseUri = "/api/v1";

  @Autowired
  WebTestClient webClient;

  @Autowired
  TestRestTemplate restTemplate;

  @Test
  @DisplayName("Create a new passenger")
  public void testCreatePassenger() {
    webClient.post()
        .uri(baseUri)
        .bodyValue(PassengerUtils.buildCreatePassengerRequest(1))
        .accept(MediaType.APPLICATION_JSON)
        .exchange()
        .expectStatus().isCreated()
        .expectHeader().exists(HttpHeaders.LOCATION);
  }

  @Test
  @DisplayName("Add a new address")
  public void testAddUserAddress() {
    URI passengerUri = restTemplate
        .postForLocation(baseUri,
            PassengerUtils.buildCreatePassengerRequest(1));
    URI addressesUri = ServletUriComponentsBuilder.fromUri(passengerUri)
        .path("/addresses").build().toUri();
    webClient.post().uri(addressesUri)
        .bodyValue(PassengerUtils.buildCreateUserAddressRequest())
        .exchange()
        .expectStatus().isOk()
        .expectBody(PassengerVO.class)
        .value(hasProperty("userAddresses", hasSize(2)));
  }

  @Test
  @DisplayName("Remove an address")
  public void testRemoveAddress() {
    URI passengerUri = restTemplate
        .postForLocation(baseUri,
            PassengerUtils.buildCreatePassengerRequest(3));
    PassengerVO passenger = restTemplate
        .getForObject(passengerUri, PassengerVO.class);
    String addressId = passenger.getUserAddresses().get(0).getId();
    URI addressUri = ServletUriComponentsBuilder.fromUri(passengerUri)
        .path("/addresses/" + addressId).build().toUri();
    webClient.delete().uri(addressUri)
        .exchange()
        .expectStatus().isNoContent();
    webClient.get().uri(passengerUri)
        .exchange()
        .expectStatus().isOk()
        .expectBody(PassengerVO.class)
        .value(hasProperty("userAddresses", hasSize(2)));
  }

}
</code></pre>
<h4>使用 Swagger 客户端</h4>
<p>因为微服务实现采用 API 优先的策略，在有 OpenAPI 文档的前提下，我们可以使用 Swagger 代码生成工具来产生客户端代码，并在测试中使用。这种方式的好处是客户端代码屏蔽了 API 的一些细节，比如 API 的访问路径。如果 API 的访问路径发生了变化，那么测试代码并不需要修改，只需要使用新版本的客户端即可。在上一节的代码中，我们需要手动构建不同操作对应的 API 路径，这就产生了不必要的耦合。</p>
<p>使用客户端的另外一个好处是，如果 OpenAPI 文档发生变化，则会造成客户端的接口变化。重新生成新的客户端代码之后，已有测试代码会无法通过编译，开发人员可以在第一时间发现问题并更新测试代码。而使用 HTTP 请求的测试代码，则需要在运行测试时才能发现问题。</p>
<p>乘客管理服务 API 的客户端是示例应用的一个模块，只不过代码都是自动生成的。这里我们需要用到 Swagger 代码生成工具的 Maven 插件，下面的代码给出了该插件的使用方式：</p>
<pre><code data-language="html" class="lang-html"><span class="hljs-tag">&lt;<span class="hljs-name">plugin</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>io.swagger.codegen.v3<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>swagger-codegen-maven-plugin<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">executions</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">execution</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">goals</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">goal</span>&gt;</span>generate<span class="hljs-tag">&lt;/<span class="hljs-name">goal</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">goals</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">configuration</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">inputSpec</span>&gt;</span>${project.basedir}/src/main/resources/openapi.yml<span class="hljs-tag">&lt;/<span class="hljs-name">inputSpec</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">language</span>&gt;</span>java<span class="hljs-tag">&lt;/<span class="hljs-name">language</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">apiPackage</span>&gt;</span>io.vividcode.happyride.passengerservice.client.api<span class="hljs-tag">&lt;/<span class="hljs-name">apiPackage</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">generateModels</span>&gt;</span>false<span class="hljs-tag">&lt;/<span class="hljs-name">generateModels</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">generateApiTests</span>&gt;</span>false<span class="hljs-tag">&lt;/<span class="hljs-name">generateApiTests</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">importMappings</span>&gt;</span>
          <span class="hljs-tag">&lt;<span class="hljs-name">importMapping</span>&gt;</span>CreatePassengerRequest=io.vividcode.happyride.passengerservice.api.web.CreatePassengerRequest<span class="hljs-tag">&lt;/<span class="hljs-name">importMapping</span>&gt;</span>
          <span class="hljs-tag">&lt;<span class="hljs-name">importMapping</span>&gt;</span>CreateUserAddressRequest=io.vividcode.happyride.passengerservice.api.web.CreateUserAddressRequest<span class="hljs-tag">&lt;/<span class="hljs-name">importMapping</span>&gt;</span>
          <span class="hljs-tag">&lt;<span class="hljs-name">importMapping</span>&gt;</span>PassengerVO=io.vividcode.happyride.passengerservice.api.web.PassengerVO<span class="hljs-tag">&lt;/<span class="hljs-name">importMapping</span>&gt;</span>
          <span class="hljs-tag">&lt;<span class="hljs-name">importMapping</span>&gt;</span>UserAddressVO=io.vividcode.happyride.passengerservice.api.web.UserAddressVO<span class="hljs-tag">&lt;/<span class="hljs-name">importMapping</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">importMappings</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">configOptions</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">configOptions</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">configuration</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">execution</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">executions</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">plugin</span>&gt;</span>
</code></pre>
<p>下表给出了该插件的配置项说明。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/00/3D/CgqCHl6pNKGAakEHAAC2a0ntw3I633.png" alt="微服务1.png"></p>
<p>Swagger 的代码生成工具可以从 OpenAPI 文档的模式声明中产生对应的 Java 模型类。示例应用已经定义了相关的模型类，因此把配置项 generateModels 的值设置为 false 来禁用模型类的生成，同时使用 importMappings 来声明映射关系。</p>
<p>当需要编写 REST API 测试时，我们只需要依赖这个 API 客户端模块即可。下面代码中的 PassengerControllerClientTest 类是使用 API 客户端的测试类。由于 API 服务器的端口是随机的，构造器中的 @LocalServerPort 注解的作用是获取到实际运行时的端口，该端口用来构建 API 客户端访问的服务器地址。</p>
<p>PassengerApi 类是 API 客户端中自动生成的访问 API 的类。在测试方法中，我使用 PassengerApi 类的方法来执行不同的操作，并验证结果。相对于上一节中 WebTestClient 类的用法，使用 PassengerApi 类的代码更加直观易懂。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@SpringBootTest</span>(webEnvironment = WebEnvironment.RANDOM_PORT)
<span class="hljs-meta">@EnableAutoConfiguration</span>
<span class="hljs-meta">@ComponentScan</span>
<span class="hljs-meta">@Import</span>(EmbeddedPostgresConfiguration<span class="hljs-class">.<span class="hljs-keyword">class</span>)
@<span class="hljs-title">ImportAutoConfiguration</span>(<span class="hljs-title">classes</span> </span>= {
    EmbeddedPostgreSQLDependenciesAutoConfiguration<span class="hljs-class">.<span class="hljs-keyword">class</span>,
    <span class="hljs-title">EmbeddedPostgreSQLBootstrapConfiguration</span>.<span class="hljs-title">class</span>
})
@<span class="hljs-title">TestPropertySource</span>(<span class="hljs-title">properties</span> </span>= {
    <span class="hljs-string">"embedded.postgresql.docker-image=postgres:12-alpine"</span>
})
<span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"Passenger controller test"</span>)
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PassengerControllerClientTest</span> </span>{

  <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> PassengerApi passengerApi;

  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">PassengerControllerClientTest</span><span class="hljs-params">(@LocalServerPort <span class="hljs-keyword">int</span> serverPort)</span> </span>{
    ApiClient apiClient = Configuration.getDefaultApiClient();
    apiClient.setBasePath(<span class="hljs-string">"http://localhost:"</span> + serverPort + <span class="hljs-string">"/api/v1"</span>);
    passengerApi = <span class="hljs-keyword">new</span> PassengerApi(apiClient);
  }

  <span class="hljs-meta">@Test</span>
  <span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"Create a new passenger"</span>)
  <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">testCreatePassenger</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">try</span> {
      ApiResponse&lt;Void&gt; response = passengerApi
          .createPassengerWithHttpInfo(
              PassengerUtils.buildCreatePassengerRequest(<span class="hljs-number">1</span>));
      assertThat(response.getStatusCode()).isEqualTo(<span class="hljs-number">201</span>);
      assertThat(response.getHeaders()).containsKey(<span class="hljs-string">"Location"</span>);
    } <span class="hljs-keyword">catch</span> (ApiException e) {
      fail(e);
    }
  }

  <span class="hljs-meta">@Test</span>
  <span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"Add a new address"</span>)
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testAddUserAddress</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">try</span> {
      String passengerId = createPassenger(<span class="hljs-number">1</span>);
      PassengerVO passenger = passengerApi
        .createAddress(PassengerUtils.buildCreateUserAddressRequest(),
              passengerId);
      assertThat(passenger.getUserAddresses()).hasSize(<span class="hljs-number">2</span>);
    } <span class="hljs-keyword">catch</span> (ApiException e) {
      fail(e);
    }
  }

  <span class="hljs-meta">@Test</span>
  <span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"Remove an address"</span>)
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testRemoveAddress</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">try</span> {
      String passengerId = createPassenger(<span class="hljs-number">3</span>);
      PassengerVO passenger = passengerApi.getPassenger(passengerId);
      String addressId = passenger.getUserAddresses().get(<span class="hljs-number">0</span>).getId();
      passengerApi.deleteAddress(passengerId, addressId);
      passenger = passengerApi.getPassenger(passengerId);
      assertThat(passenger.getUserAddresses()).hasSize(<span class="hljs-number">2</span>);
    } <span class="hljs-keyword">catch</span> (ApiException e) {
      fail(e);
    }
  }

  <span class="hljs-function"><span class="hljs-keyword">private</span> String <span class="hljs-title">createPassenger</span><span class="hljs-params">(<span class="hljs-keyword">int</span> numberOfAddresses)</span> <span class="hljs-keyword">throws</span> ApiException </span>{
    ApiResponse&lt;Void&gt; response = passengerApi
        .createPassengerWithHttpInfo(
         PassengerUtils.buildCreatePassengerRequest(numberOfAddresses));
    assertThat(response.getHeaders()).containsKey(<span class="hljs-string">"Location"</span>);
    String location = response.getHeaders().get(<span class="hljs-string">"Location"</span>).get(<span class="hljs-number">0</span>);
    <span class="hljs-keyword">return</span> StringUtils.substringAfterLast(location, <span class="hljs-string">"/"</span>);
  }
}
</code></pre>
<h4>使用 BDD</h4>
<p>不管是手动发送 HTTP 请求或是使用 Swagger 客户端，相关的测试用例都需要由测试人员来编写。当应用的业务逻辑比较复杂时，测试人员可能需要了解很多的业务知识，才能编写出正确的测试用例。以保险业务为例，一个理赔申请能否被批准，背后有复杂的业务逻辑来确定。这样的测试用例，如果由测试人员来编写，则可能的结果是测试用例所验证的情况，从业务逻辑上来说是错误的，起不到测试的效果。</p>
<p>更好的做法是由业务人员来编写测试用例，这样可以保证应用的实际行为，满足真实业务的期望。但是业务人员并不懂得编写代码。为了解决这个问题， 我们需要让业务人员以他们所能理解的方式来描述对不同行为的期望，这就是<strong>行为驱动开发</strong>（Behaviour Driven Development，BDD）的思想。</p>
<p>BDD 的出发点是提供了一种自然语言的方式来描述应用的行为，对行为的描述由 3 个部分组成，分别是<strong>前置条件</strong>、<strong>动作</strong>和<strong>期望结果</strong>，也就是 Given-When-Then 结构，该结构表达的是当对象处于某个状态中时，如果执行该对象的某个动作，所应该产生的结果是什么。比如，对于一个数据结构中常用的栈对象来说，在栈为空的前提下，如果执行栈的弹出动作，那么应该抛出异常；在栈不为空的前提下，如果执行栈的弹出动作，那么返回值应该是栈顶的元素。这样的行为描述，可以很容易转换成测试用例，来验证对象的实际行为。</p>
<p>BDD 一般使用自然语言来描述行为，业务人员使用自然语言来描述行为，形成 BDD 文档，这个文档是业务知识的具体化。我们只需要把这个文档转换成可执行的测试用例，就可以验证代码实现是否满足业务的需求。这个转换的过程需要工具的支持，本课时介绍的工具是 <a href="https://cucumber.io/">Cucumber</a>。</p>
<p>Cucumber 使用名为 Gherkin 的语言来描述行为，Gherkin 语言使用半结构化的形式。下表给出了 Gherkin 语言中的常用结构。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/00/3D/CgqCHl6pNRKAMlt_AABWRPxiWY4472.png" alt="微服务2.png"></p>
<p>下面的代码给出了乘客管理服务的 BDD 文档示例，其中的第一个场景描述的是添加地址的行为。前置条件是乘客有 1 个地址，动作是添加一个新的地址，期望的结果是乘客有 2 个地址。第二个场景描述的是删除地址的行为，与第一个场景有类似的描述。</p>
<pre><code data-language="sql" class="lang-sql">Feature: Address management
  Manage a passenger's addresses

  Scenario: Add a new address
    Given a passenger <span class="hljs-keyword">with</span> <span class="hljs-number">1</span> addresses
    <span class="hljs-keyword">When</span> the passenger adds a <span class="hljs-keyword">new</span> address
    <span class="hljs-keyword">Then</span> the passenger has <span class="hljs-number">2</span> addresses

  Scenario: <span class="hljs-keyword">Delete</span> an address
    Given a passenger <span class="hljs-keyword">with</span> <span class="hljs-number">3</span> addresses
    <span class="hljs-keyword">When</span> the passenger deletes an address
    <span class="hljs-keyword">Then</span> the passenger has <span class="hljs-number">2</span> addresses
</code></pre>
<p>在有了 BDD 文档之后，下一步是如何把文档变成可执行的测试用例，这就需要用到 Cucumber 了。Cucumber 使用步骤定义把 Gherkin 语言中的结构与实际的代码关联起来。Gherkin 语言中的 Given、When 和 Then 等语句，都有与之对应的步骤定义代码。Cucumber 在运行 BDD 文档的场景时，会找到每个语句对应的步骤定义并执行，步骤定义中包含了进行验证的代码。</p>
<p>下面代码中的 AddressStepdefs 类是上面 BDD 场景的步骤定义。AddressStepdefs 类上的注解与上面 PassengerControllerTest 类的注解是相似的，PassengerClient 类是 Swagger 客户端的一个封装，用来执行不同的操作。</p>
<p>在步骤定义中，passengerWithAddresses 方法对应的是 Given 语句“a passenger with {int} addresses”。语句中的 {int} 用来提取语句中 int 类型的变量，作为参数 numberOfAddresses 的值。在对应的步骤中，调用 PassengerClient 类的 createPassenger 方法来创建一个包含指定数量地址的乘客对象，并把乘客 ID 保存在 passengerId 中。passengerAddsAddress 和 passengerDeletesAddress 方法分别对应两个 When 语句，分别执行添加和删除地址的动作；passengerHasAddresses 方法则与 Then 语句相对应，验证乘客的地址数量。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@SpringBootTest</span>(webEnvironment = WebEnvironment.RANDOM_PORT)
<span class="hljs-meta">@EnableAutoConfiguration</span>
<span class="hljs-meta">@ContextConfiguration</span>(classes = {
    BddTestApplication<span class="hljs-class">.<span class="hljs-keyword">class</span>,
    <span class="hljs-title">PassengerServiceApplication</span>.<span class="hljs-title">class</span>
})
@<span class="hljs-title">ComponentScan</span>
@<span class="hljs-title">Import</span>(<span class="hljs-title">EmbeddedPostgresConfiguration</span>.<span class="hljs-title">class</span>)
@<span class="hljs-title">ImportAutoConfiguration</span>(<span class="hljs-title">classes</span> </span>= {
    EmbeddedPostgreSQLDependenciesAutoConfiguration<span class="hljs-class">.<span class="hljs-keyword">class</span>,
    <span class="hljs-title">EmbeddedPostgreSQLBootstrapConfiguration</span>.<span class="hljs-title">class</span>
})
@<span class="hljs-title">TestPropertySource</span>(<span class="hljs-title">properties</span> </span>= {
    <span class="hljs-string">"embedded.postgresql.docker-image=postgres:12-alpine"</span>
})
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AddressStepdefs</span> </span>{

  <span class="hljs-meta">@Autowired</span>
  PassengerClient passengerClient;

  <span class="hljs-keyword">private</span> String passengerId;

  <span class="hljs-meta">@Given</span>(<span class="hljs-string">"a passenger with {int} addresses"</span>)
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">passengerWithAddresses</span><span class="hljs-params">(<span class="hljs-keyword">int</span> numberOfAddresses)</span> </span>{
    <span class="hljs-keyword">try</span> {
      passengerId = passengerClient.createPassenger(numberOfAddresses);
    } <span class="hljs-keyword">catch</span> (ApiException e) {
      fail(e);
    }
  }

  <span class="hljs-meta">@When</span>(<span class="hljs-string">"the passenger adds a new address"</span>)
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">passengerAddsAddress</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">try</span> {
      passengerClient.addAddress(passengerId);
    } <span class="hljs-keyword">catch</span> (ApiException e) {
      fail(e);
    }
  }

  <span class="hljs-meta">@When</span>(<span class="hljs-string">"the passenger deletes an address"</span>)
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">passengerDeletesAddress</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">try</span> {
      passengerClient.removeAddress(passengerId);
    } <span class="hljs-keyword">catch</span> (ApiException e) {
      fail(e);
    }
  }

  <span class="hljs-meta">@Then</span>(<span class="hljs-string">"the passenger has {int} addresses"</span>)
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">passengerHasAddresses</span><span class="hljs-params">(<span class="hljs-keyword">int</span> numberOfAddresses)</span> </span>{
    <span class="hljs-keyword">try</span> {
      PassengerVO passenger = passengerClient.getPassenger(passengerId);
      assertThat(passenger.getUserAddresses()).hasSize(numberOfAddresses);
    } <span class="hljs-keyword">catch</span> (ApiException e) {
      fail(e);
    }
  }

}
</code></pre>
<p>实际的测试由 Cucumber 来运行，下面代码中的 PassengerIntegrationTest 类是由 Cucumber 运行的测试类。需要注意的是，Cucumber 只支持 JUnit 4，因此在 Spring Boot 应用中需要添加对 junit-vintage-engine 模块的依赖。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@RunWith</span>(Cucumber<span class="hljs-class">.<span class="hljs-keyword">class</span>)
@<span class="hljs-title">CucumberOptions</span>(<span class="hljs-title">strict</span> </span>= <span class="hljs-keyword">true</span>,
    features = <span class="hljs-string">"src/test/resources/features"</span>,
    plugin = {<span class="hljs-string">"pretty"</span>, <span class="hljs-string">"html:target/cucumber"</span>})
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PassengerIntegrationTest</span> </span>{

}
</code></pre>
<h4>数据准备</h4>
<p>集成测试的一个特点是在运行测试之前，对应用的当前状态有一定的要求，而不是从零开始。比如，如果乘客管理服务的 API 支持对地址的分页、查询和过滤，那么在测试这些功能时，则需要被测试的乘客有足够数量的地址来测试不同的情况。这就要求进行一些数据准备工作。下面是 3 种不同的准备数据的做法。</p>
<p>第一种做法是在测试中通过 JUnit 5 的 @BeforeEach 和 @BeforeAll 注解来标注进行数据准备的方法，数据准备通过测试代码中的 API 调用来完成。比如， 可以在每个测试方法执行之前创建一个包含 100 个地址的乘客对象。</p>
<p>第二种做法是通过 SQL 脚本来初始化测试数据库，一个测试用例可以有与之相对应的 SQL 初始化脚本。在运行测试时，SQL 脚本会被执行来添加测试数据。如果测试人员更习惯编写 SQL 脚本，这种方式比第一种更好。Spring Boot 提供了初始化数据库的支持，会自动查找并执行 CLASSPATH 中的 schema.sql 和 data.sql 文件。</p>
<p>第三种做法是使用专门的数据库 Docker 镜像。在之前的测试中，我们使用的是标准的 PostgreSQL 镜像，其中不包含任何数据，不过可以创建一个包含了完整数据的自定义 PostgreSQL 镜像，这样可以测试应用在更新时的行为。比如，新版本的代码中对数据库的模式进行了修改，一个很重要的测试用例就是已有数据的升级。通过使用包含当前版本数据的 PostgreSQL 镜像，可以很容易的测试数据库的升级。Docker 镜像的标签标识了每个镜像包含的数据集的含义，这些都是本地开发和编写测试用例时可以使用的资产。很多数据库的 Docker 镜像都提供了运行初始化脚本的能力。以 PostgreSQL 的 Docker 镜像为例，只需要把 *.sh 或 *.sql 文件添加到 /docker-entrypoint-initdb.d 目录，就可以初始化数据库。</p>
<h4>总结</h4>
<p>微服务的集成测试把微服务当成一个黑盒子，并使用其 REST API 来进行测试。本课时介绍了 3 种不同的 REST API 集成测试方式，包括手动发送 HTTP 请求并验证，使用 Swagger 客户端来发送请求，以及使用 BDD 来编写测试用例。</p>