<p>本课时将介绍“如何使用 JUnit 5 实现服务的单元测试”。</p>
<p>第 11 课时对“数据库驱动的微服务实现”做了简要的介绍，本课时将介绍如何使用 JUnit 5 进行单元测试。你可能会好奇，实现相关的内容比较多却用一个课时来讲解，而内容相对较少的单元测试部分也同样用一个课时？</p>
<p>这是因为市面上与实现相关的参考资料已经非常多了，而单元测试的介绍则相对较少，甚至被忽略了。单元测试的重要性怎么强调都不过分。没有覆盖率足够高的自动化单元测试，就无法安全的更新代码和进行重构。单元测试是开发人员所依赖的安全网。基于这些原因，本课时将对单元测试进行具体的介绍。</p>
<h4>JUnit 5 介绍</h4>
<p>JUnit 是 Java 单元测试领域中的事实标准，最新版本是 JUnit 5，该版本由 JUnit Platform、JUnit Jupiter 和 JUnit Vintage 组成，这 3 个组件的说明如下表所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/07/AF/CgoCgV6iXLqAfqVwAABV7M2QOBM600.png" alt="图片1.png"><br>
JUnit Jupiter 的编程模型相比于 JUnit 4 有了很大的改进，推荐在新的项目中使用。下面是一些重要的注解：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/14/DF/Ciqah16iXMmAEiJ7AACRoHzs3ew474.png" alt="图片2.png"><br>
下面代码中的 JUnit5Sample 类展示了 @ParameterizedTest、@RepeatedTest 和 @TestFactory 的用法。stringLength方法用来验证字符串的长度，@ValueSource 注解用来提供参数化的测试方法的实际参数。repeatedTest 方法会被重复执行 3 次。dynamicTests 方法返回了一个 DynamicTest 数组作为动态创建的测试。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"JUnit 5 sample"</span>)
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">JUnit5Sample</span> </span>{

  <span class="hljs-meta">@ParameterizedTest</span>
  <span class="hljs-meta">@ValueSource</span>(strings = {<span class="hljs-string">"hello"</span>, <span class="hljs-string">"world"</span>})
  <span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"String length"</span>)
  <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">stringLength</span><span class="hljs-params">(String value)</span> </span>{
    assertThat(value).hasSize(<span class="hljs-number">5</span>);
  }

  <span class="hljs-meta">@RepeatedTest</span>(<span class="hljs-number">3</span>)
  <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">repeatedTest</span><span class="hljs-params">()</span> </span>{
    assertThat(<span class="hljs-keyword">true</span>).isTrue();
  }

  <span class="hljs-meta">@TestFactory</span>
  DynamicTest[] dynamicTests() {
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> DynamicTest[]{
        dynamicTest(<span class="hljs-string">"Dynamic test 1"</span>, () -&gt;
            assertThat(<span class="hljs-number">10</span>).isGreaterThan(<span class="hljs-number">5</span>)),
        dynamicTest(<span class="hljs-string">"Dynamic test 2"</span>, () -&gt;
            assertThat(<span class="hljs-string">"hello"</span>).hasSize(<span class="hljs-number">5</span>))
    };
  }
}
</code></pre>
<p>下面给出了 JUnit5Sample 测试的运行结果。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/07/B0/CgoCgV6iXOuAZf2mAABIdHZQLEg187.png" alt="3.png"></p>
<p>Spring Boot 已经提供了对 JUnit 5 的集成，在项目中可以直接使用。关于 JUnit 5 的更多内容，请参考其他相关资料。</p>
<h4>领域对象测试</h4>
<p>领域对象类中包含了相关的业务逻辑，我们需要添加相应的单元测试，由于领域对象类通常没有其他依赖，这使得测试起来很方便。</p>
<p>下面代码中的 PassengerTest 是领域对象类 Passenger 的单元测试用例。PassengerTest 中的测试方法都很直接，只需要创建 Passenger 对象，调用其中的方法，再进行验证即可。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"Passenger test"</span>)
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PassengerTest</span> </span>{

  <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Faker faker = <span class="hljs-keyword">new</span> Faker(Locale.CHINA);

  <span class="hljs-meta">@Test</span>
  <span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"Add user address"</span>)
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testAddUserAddress</span><span class="hljs-params">()</span> </span>{
    Passenger passenger = createPassenger(<span class="hljs-number">1</span>);
    passenger.addUserAddress(createUserAddress());
    assertThat(passenger.getUserAddresses()).hasSize(<span class="hljs-number">2</span>);
  }

  <span class="hljs-meta">@Test</span>
  <span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"Remove user address"</span>)
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testRemoveUserAddress</span><span class="hljs-params">()</span> </span>{
    Passenger passenger = createPassenger(<span class="hljs-number">3</span>);
    String addressId = passenger.getUserAddresses().get(<span class="hljs-number">0</span>).getId();
    passenger.removeUserAddress(addressId);
    assertThat(passenger.getUserAddresses()).hasSize(<span class="hljs-number">2</span>);
  }

  <span class="hljs-meta">@Test</span>
  <span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"Get user address"</span>)
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testGetUserAddress</span><span class="hljs-params">()</span> </span>{
    Passenger passenger = createPassenger(<span class="hljs-number">2</span>);
    String addressId = passenger.getUserAddresses().get(<span class="hljs-number">0</span>).getId();
    assertThat(passenger.getUserAddress(addressId)).isPresent();
    assertThat(passenger.getUserAddress(<span class="hljs-string">"invalid"</span>)).isEmpty();
  }

  <span class="hljs-function"><span class="hljs-keyword">private</span> Passenger <span class="hljs-title">createPassenger</span><span class="hljs-params">(<span class="hljs-keyword">int</span> numberOfAddresses)</span> </span>{
    Passenger passenger = <span class="hljs-keyword">new</span> Passenger();
    passenger.generateId();
    passenger.setName(faker.name().fullName());
    passenger.setEmail(faker.internet().emailAddress());
    passenger.setMobilePhoneNumber(faker.phoneNumber().phoneNumber());
    <span class="hljs-keyword">int</span> count = Math.max(<span class="hljs-number">0</span>, numberOfAddresses);
    List&lt;UserAddress&gt; addresses = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;(count);
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; count; i++) {
      addresses.add(createUserAddress());
    }
    passenger.setUserAddresses(addresses);
    <span class="hljs-keyword">return</span> passenger;
  }

  <span class="hljs-function"><span class="hljs-keyword">private</span> UserAddress <span class="hljs-title">createUserAddress</span><span class="hljs-params">()</span> </span>{
    UserAddress userAddress = <span class="hljs-keyword">new</span> UserAddress();
    userAddress.generateId();
    userAddress.setName(faker.pokemon().name());
    userAddress.setAddressId(UUID.randomUUID().toString());
    userAddress.setAddressName(faker.address().fullAddress());
    <span class="hljs-keyword">return</span> userAddress;
  }
}
</code></pre>
<h4>数据库测试</h4>
<p>对于数据库驱动的微服务来说，数据库相关的测试是重点，一种常见的做法是在测试时使用专门的数据库来实现，比如 H2 和 HSQLDB，这两个数据库都是纯 Java 语言实现的，支持内存数据库或使用文件存储。从测试的角度来说，这两个数据库可以通过嵌入式的方式在当前 Java 进程中启动，这就降低了测试数据库服务器的管理复杂度。测试中使用的数据本来就是临时性的，只是在测试运行时有用，内存数据库的使用，则进一步省去了管理测试数据库的工作。</p>
<p>Spring Boot 提供了对嵌入式数据库的支持，我们只需要添加 H2 或 HSQLDB 的运行时依赖，Spring Boot 则会在测试时自动配置相应的数据源。</p>
<p>这种做法有一个很大的问题，那就是在运行单元测试时使用的数据库和生产环境的数据库是不一致的。H2 和 HSQLDB 这样的数据库并不适用于生产环境，生产环境中需要使用 PostgreSQL、MySQL 和 SQL Server 这样的数据库。虽然都是使用 JDBC 来访问数据库，我们并不能因此就忽略这些数据库实现之间的区别。这就意味着，通过单元测试的代码，在运行时有可能会由于数据库实现的差异而产生问题。如果发生了这样的情况，则会降低开发人员对单元测试的信任度。</p>
<p>在第 11 课时，我提到了推荐使用数据库迁移工具和手动管理数据库的表模式。如果在单元测试和生产环境中使用不同的数据库实现，则需要维护两套不同的 SQL 脚本，这无疑增加了维护成本。更好的做法是在运行单元测试时，使用与生产环境一样的数据库实现，这看起来很复杂，所幸的是，Docker 可以帮助我们简化很多工作。另外一个附加的好处是，单元测试时 Docker 的使用也与生产环境中的 Kubernetes 中 Docker 的使用保持一致。</p>
<p>在单元测试中使用 Docker 时，我们需要用到 <a href="https://www.testcontainers.org/">Testcontainers</a> 这个第三方库，以及它提供的 Spring Boot 集成。在进行单元测试时，会启动一个 PostgreSQL 的 Docker 容器，并创建一个数据源来指向这个容器中的 PostgreSQL 服务器，其中的具体工作由 Testcontainers 来完成，我们只需要进行配置即可。</p>
<p>下面的代码是乘客管理服务中 PassengerService 的单元测试用例，其中比较重要的是几个注解的使用。</p>
<p>@DataJpaTest 注解由 Spring Boot 提供，其作用是限制 Spring 在扫描 bean 时的范围，只会选择与 Spring Data JPA 相关的 bean。</p>
<p>@AutoConfigureTestDatabase(replace = Replace.NONE) 的作用是禁止 Spring Boot 用嵌入式数据库替换掉当前的数据源。默认情况下，在运行单元测试时，Spring Boot 会配置一个使用嵌入式数据库的数据源，并替换掉应用中声明的数据源。由于我们使用的是 Docker 容器中的数据库，那么需要禁用这个默认行为。Replace.NONE 的作用是要求 Spring Boot 不进行替换，而是继续使用代码中声明的数据源。</p>
<p>@ContextConfiguration 注解声明了使用的配置类 EmbeddedPostgresConfiguration。</p>
<p>@ImportAutoConfiguration 注解导入了由 Testcontainers 提供的 Spring Boot 自动配置类，用来启动 Docker 容器并提供与数据库连接相关的配置属性。</p>
<p>@TestPropertySource 注解添加了额外的配置属性 embedded.postgresql.docker-image 来设置使用的 PostgreSQL 镜像。</p>
<p>在 PassengerServiceTest 类中，通过 @Autowired 注解注入了 PassengerService 类的实例。PassengerServiceTest 类中的测试用例，可使用 PassengerService 类的方法来完成不同的操作，并验证结果。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@DataJpaTest</span>
<span class="hljs-meta">@EnableAutoConfiguration</span>
<span class="hljs-meta">@AutoConfigureTestDatabase</span>(replace = Replace.NONE)
<span class="hljs-meta">@ComponentScan</span>
<span class="hljs-meta">@ContextConfiguration</span>(classes = {
    EmbeddedPostgresConfiguration<span class="hljs-class">.<span class="hljs-keyword">class</span>
})
@<span class="hljs-title">ImportAutoConfiguration</span>(<span class="hljs-title">classes</span> </span>= {
    EmbeddedPostgreSQLDependenciesAutoConfiguration<span class="hljs-class">.<span class="hljs-keyword">class</span>,
    <span class="hljs-title">EmbeddedPostgreSQLBootstrapConfiguration</span>.<span class="hljs-title">class</span>
})
@<span class="hljs-title">TestPropertySource</span>(<span class="hljs-title">properties</span> </span>= {
    <span class="hljs-string">"embedded.postgresql.docker-image=postgres:12-alpine"</span>
})
<span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"Passenger service test"</span>)
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PassengerServiceTest</span> </span>{

  <span class="hljs-meta">@Autowired</span>
  PassengerService passengerService;

  <span class="hljs-meta">@Test</span>
  <span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"Create a new passenger"</span>)
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testCreatePassenger</span><span class="hljs-params">()</span> </span>{
    CreatePassengerRequest request = PassengerUtils.buildCreatePassengerRequest(<span class="hljs-number">1</span>);
    PassengerVO passenger = passengerService.createPassenger(request);
    assertThat(passenger.getId()).isNotNull();
    assertThat(passenger.getUserAddresses()).hasSize(<span class="hljs-number">1</span>);
  }

  <span class="hljs-meta">@Test</span>
  <span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"Add a user address"</span>)
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testAddAddress</span><span class="hljs-params">()</span> </span>{
    CreatePassengerRequest request = PassengerUtils.buildCreatePassengerRequest(<span class="hljs-number">1</span>);
    PassengerVO passenger = passengerService.createPassenger(request);
    passenger = passengerService
        .addAddress(passenger.getId(), PassengerUtils.buildCreateUserAddressRequest());
    assertThat(passenger.getUserAddresses()).hasSize(<span class="hljs-number">2</span>);
  }

  <span class="hljs-meta">@Test</span>
  <span class="hljs-meta">@DisplayName</span>(<span class="hljs-string">"Delete a user address"</span>)
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testDeleteAddress</span><span class="hljs-params">()</span> </span>{
    CreatePassengerRequest request = PassengerUtils.buildCreatePassengerRequest(<span class="hljs-number">3</span>);
    PassengerVO passenger = passengerService.createPassenger(request);
    String addressId = passenger.getUserAddresses().get(<span class="hljs-number">1</span>).getId();
    passenger = passengerService.deleteAddress(passenger.getId(), addressId);
    assertThat(passenger.getUserAddresses()).hasSize(<span class="hljs-number">2</span>);
  }
}
</code></pre>
<p>下面代码中的 EmbeddedPostgresConfiguration 类用来配置运行单元测试时的数据源。以 embedded.postgresql 开头的属性值由 Testcontainers 生成，包含运行的 PostgreSQL 容器的连接信息。通过这些属性，我创建了一个 HikariDataSource 数据源，在运行单元测试时使用。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EmbeddedPostgresConfiguration</span> </span>{

  <span class="hljs-meta">@Autowired</span>
  ConfigurableEnvironment environment;

  <span class="hljs-meta">@Bean</span>(destroyMethod = <span class="hljs-string">"close"</span>)
  <span class="hljs-function"><span class="hljs-keyword">public</span> DataSource <span class="hljs-title">testDataSource</span><span class="hljs-params">()</span> </span>{
    String jdbcUrl = <span class="hljs-string">"jdbc:postgresql://${embedded.postgresql.host}:${embedded.postgresql.port}/${embedded.postgresql.schema}"</span>;
    HikariConfig hikariConfig = <span class="hljs-keyword">new</span> HikariConfig();
    hikariConfig.setDriverClassName(<span class="hljs-string">"org.postgresql.Driver"</span>);
    hikariConfig.setJdbcUrl(environment.resolvePlaceholders(jdbcUrl));
    hikariConfig.setUsername(environment.getProperty(<span class="hljs-string">"embedded.postgresql.user"</span>));
    hikariConfig.setPassword(environment.getProperty(<span class="hljs-string">"embedded.postgresql.password"</span>));
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> HikariDataSource(hikariConfig);
  }
}
</code></pre>
<h4>使用 mock 对象</h4>
<p>一个对象通常有很多依赖的对象，这些被依赖的对象又有各自的依赖对象。在对当前对象进行单元测试时，我们希望仅测试当前对象的行为，比如，对象 A 依赖对象 B、C，而对象 B、C 则分别依赖对象 D、E，如下图所示。在编写对象 A 的单元测试用例时，我们希望可以模拟对象 B、C 的行为，从而测试对象 A 在不同情况下的行为，这就需要用到 mock 对象。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/07/B0/CgoCgV6iXWyAEWGPAAArkcc--wE091.png" alt="4.png"></p>
<p>mock 对象可以模拟一个对象的行为。举例来说，对象 A 的方法 methodA 调用了对象 B 中的方法 methodB，并根据 methodB 的返回值进行不同的操作。在编写对象 A 的 methodA 方法的测试用例时，则需要覆盖不同的代码路径。对象 B 的 mock 可以很好的解决这个问题。在创建了对象 B 的 mock 之后，就可以直接指定 methodB 的返回值了，从而验证 methodA 在不同情况下的行为。</p>
<p>行程派发服务的 TripServiceTest 类用到了 mock 对象。不过 TripServiceTest 类的逻辑比较复杂，因此我选择另外一个更简单的例子来说明 mock 对象的用法，你也可以直接参考示例应用中的 TripServiceTest 类。</p>
<p>下面代码中的 ActionService 类依赖 ValueUpdater 和 EventPublisher 两个对象，其中 ValueUpdater 的 updateValue 方法用来更新值。如果 updateValue 方法的返回值是 true，则 EventPublisher 的 publishEvent 方法会被调用来发布一个 ValueUpdatedEvent 事件。</p>
<pre><code data-language="java" class="lang-java">Service
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ActionService</span> </span>{

  <span class="hljs-meta">@Autowired</span>
  ValueUpdater valueUpdater;

  <span class="hljs-meta">@Autowired</span>
  EventPublisher eventPublisher;

  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">performAction</span><span class="hljs-params">(Integer value)</span> </span>{
    Integer oldValue = valueUpdater.getValue();
    <span class="hljs-keyword">if</span> (valueUpdater.updateValue(value)) {
      ValueUpdatedEvent event = <span class="hljs-keyword">new</span> ValueUpdatedEvent(oldValue, value);
      eventPublisher.publishEvent(event);
      <span class="hljs-keyword">return</span> value * <span class="hljs-number">10</span>;
    }
    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
  }
}
</code></pre>
<p>在对 ActionService 类进行单元测试时，我们需要创建 ValueUpdater 和 EventPublisher 的 mock 对象。在下面的代码中，我使用 @MockBean 注解把 ValueUpdater 和 EventPublisher 都声明为 mock 对象。使用 @Captor 注解声明的 eventCaptor 对象用来捕获 EventPublisher 的 publishEvent 方法被调用时的实际参数值。</p>
<p>在 testValueUpdated 方法中，given(valueUpdater.updateValue(value)).willReturn(true) 的作用是指定 ValueUpdater 的 mock 对象的 updateValue 方法在参数为 value 时，其返回值是 true；接着验证 EventPublisher 的 publishEvent 方法被调用一次，并捕获实际的参数值；最后验证 eventCaptor 中捕获的 ValueUpdatedEvent 参数值的内容。</p>
<p>在 testValueNotUpdated 方法中，ValueUpdater 的 mock 对象的 updateValue 方法其返回值被指定为 false，然后验证 EventPublisher 的 publishEvent 方法没有被调用。</p>
<pre><code data-language="js" class="lang-js">@SpringBootTest
@ContextConfiguration(classes = TestConfiguration.class)
@DisplayName(<span class="hljs-string">"Action service test"</span>)
public <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ActionServiceTest</span> </span>{

  @Autowired
  ActionService actionService;

  @MockBean
  ValueUpdater valueUpdater;

  @MockBean
  EventPublisher eventPublisher;

  @Captor
  ArgumentCaptor&lt;ValueUpdatedEvent&gt; eventCaptor;

  @Test
  @DisplayName(<span class="hljs-string">"Value updated"</span>)
  public <span class="hljs-keyword">void</span> testValueUpdated() {
    int value = <span class="hljs-number">10</span>;
    given(valueUpdater.updateValue(value)).willReturn(<span class="hljs-literal">true</span>);
    assertThat(actionService.performAction(value)).isEqualTo(<span class="hljs-number">100</span>);
    verify(eventPublisher, times(<span class="hljs-number">1</span>)).publishEvent(eventCaptor.capture());
    assertThat(eventCaptor.getValue()).extracting(ValueUpdatedEvent::getCurrentValue)
        .isEqualTo(value);
  }

  @Test
  @DisplayName(<span class="hljs-string">"Value not updated"</span>)
  public <span class="hljs-keyword">void</span> testValueNotUpdated() {
    int value = <span class="hljs-number">10</span>;
    given(valueUpdater.updateValue(value)).willReturn(<span class="hljs-literal">false</span>);
    assertThat(actionService.performAction(value)).isEqualTo(<span class="hljs-number">0</span>);
    verify(eventPublisher, never()).publishEvent(eventCaptor.capture());
  }
}
</code></pre>
<h4>总结</h4>
<p>单元测试在微服务开发中起着重要的作用。本课时对单元测试进行了详细的介绍，包括 JUnit 5 介绍，如何测试领域对象，如何使用 Docker 来使用与生产环境相同的数据库进行测试，以及如何在单元测试中使用 mock 对象。</p>