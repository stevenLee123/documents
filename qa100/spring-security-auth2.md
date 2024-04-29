## 基本描述
在构建用户管理平台时，一般常用的框架是spring security，spring security是一个比较重量级的权限校验框架，通过自定义的filter实现了用户登陆以及权限校验的能力

## 概念
**认证**
判定一个用户的身份是否时合法的过程，用户访问资源时要求验证用户的身份信息，身份合法方可继续访问。常见的身份认证方式：用户名密码认证、二维码认证、手机短信登录、指纹认证、面部识别认证
**授权**
用户认证通过后，需要根据用户的权限来控制用户访问资源的过程，拥有资源的访问权限则正常访问，没有权限访问则拒绝访问。授权是为了更细粒度的对隐私数据进行划分，授权发生在用户认证通过之后，授权不同的用户能访问不同的资源

**用户认证信息的保存**
用户认证通过之后，为了避免用户每次操作都进行认证可以将用户的信息保存在session中或是cookie中（一般是token信息会保存在cookie中）

**RBAC模型**
role-base access control
基于角色的控制访问
将用户授予不同的角色，用户登录后根据角色下的分配的资源情况控制用户能访问到的资源信息

## 设计用户权限控制系统的基本组件
1. 用户模型，用于用户登录认证
2. 权限模型，在用户登录认证之后针对不同的用户拥有的不同的权限（一般权限分配在角色下，将角色分配给用户让用户拥有角色下的所有权限）来规定用户能访问的资源
3. 资源数据，一般来说都是各个接口的访问权限
一般来说对用户的身份判断、权限校验都可以在fiter中实现，spring security的核心也是通过一系列的filter来实现用户的登录认证授权功能


登录认证、授权的完整流程：
1. 用户在页面端发起一次会话请求，请求到达认证、权限校验的filter
2. filter首先判断是否需要进行权限校验，不需要则直接放行
3. 需要权限校验的接口，从session信息或token信息中获取用户的登录信息，未登录信息则直接重定向到登录页面，登陆信息有效的则放行
4. 用户进行登录认证后，再次发起请求到达filter，filter继续第3步，获取用户的角色权限信息
5. 根据访问接口需要的角色权限，判断用户是否拥有该角色权限，如果拥有，则放行，没有则拒绝访问

## springboot security
spring security是一个完整的用户登录认证、授权框架，可以根据自身需求整合
### 基本用法
* 使用@EnableWebSecurity开启spring security
* 编写资源接口
* 注入密码解析工具PasswordEncoder、用户详情服务UserDetailsService
* 继承WebSecurityConfigurerAdapter，使用httpSecurity配置校验规则
* 记住我功能，记录当前登录用户的token到cookie中，http.rememberMe().rememberMeParameter(“remeber-me”)
* 注解级别方法支持 ： 在@Configuration支持的注册类上打开注解@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true,jsr250Enabled = true)即可支持方法及的注解支持。prePostEnabled属性 对应@PreAuthorize。securedEnabled 属性支持@Secured注解，支持角色级别的权限控制。jsr250Enabled属性对应@RolesAllowed注解，等价于@Secured。

### 工作原理
spring mvc提供了一个DelegatingFilterProxy的类用来将servlet中的filter放入spring容器中进行管，
在初始化spring security时会在WebSecurityConfiguration 中注入一个名为springSecurityFilterChain过滤器，类型为FilterChainProxy。
FilterChainProxy是一个代理，其中包含了多个SecurityFilterChain的list，SecurityFilterChain中又包含了很多filter，这些filter是spring security的核心，各有各的职责，他们不直接处理认证和授权，会交给具体的认证管理器（AuthentationManager）和决策管理器（AccessDecisionManager）处理

### 一些比较重要的filter
SecurityContextPersistenceFilter filter链拦截入口
UsernamePasswordAuthenticationFilter 处理来自表单的提交认证，表单必须提供用户名密码，内部还有登录成功或失败之后的处理（AuthenticationSuccessHandler 和 AuthenticationFailureHandler）

FilterSecurityInterceptor 保护web资源，使用AccessDecisionManager对当前用户进行授权访问
ExceptionTranslationFilter 捕获来自FilterChain所有的异常，并进行处理，一般处理两类异常，AuthenticationException 和 AccessDeniedException，其它的异常它会继续抛出

### 认证流程
1、用户提交用户名、密码被SecurityFilterChain中的 UsernamePasswordAuthenticationFilter 过滤器获取到，封装为请求Authentication，通常情况下是UsernamePasswordAuthenticationToken这个实现类。

2、 然后过滤器将Authentication提交至认证管理器（AuthenticationManager）进行认证

3、认证成功后， AuthenticationManager 身份管理器返回一个被填充了认证信息的（包括上面提到的权限信息，身份信息，细节信息，但密码通常会被移除） Authentication 实例。

4、SecurityContextHolder 安全上下文容器将第3步填充了信息的 Authentication ，通过SecurityContextHolder.getContext().setAuthentication(…)方法，设置到其中。可以看出AuthenticationManager接口（认证管理器）是认证相关的核心接口，也是发起认证的出发点，它的实现类为ProviderManager。而Spring Security支持多种认证方式，因此ProviderManager维护着一个List 列表，存放多种认证方式，最终实际的认证工作是由AuthenticationProvider完成的。咱们知道web表单的对应的AuthenticationProvider实现类为DaoAuthenticationProvider，它的内部又维护着一个UserDetailsService负责UserDetails的获取。最终AuthenticationProvider将UserDetails填充至Authentication。

调试代码从UsernamePasswordAuthenticationFilter 开始跟踪。

最后的认证流程在AbstractUserDetailsAuthenticationProvider的authenticate方法中。获取用户在retrieveUser方法。密码比较在additionalAuthenticationChecks方法

**AuthenticationProvider接口：认证处理器**
不同的实现支持不同的认证方式
    //认证的方法
   Authentication authenticate(Authentication authentication) throws AuthenticationException;
    //支持哪种认证 
   boolean supports(Class<?> var1); 

**Authentication认证信息**
代表被认证方的抽象主体

**UserDetailsService接口**
获取用户信息的基础接口，loadUserByUsername

**BCryptPasswordEncoder**
密码解析器

### 授权流程
用户认证通过后，对访问资源的权限进行检查的过程。Spring Security可以通过http.authorizeRequests()对web请求进行授权保护。Spring Security使用标准Filter建立了对web请求的拦截，最终实现对资源的授权访问。
1、拦截请求，已认证用户访问受保护的web资源将被SecurityFilterChain中(实现类为DefaultSecurityFilterChain)的 FilterSecurityInterceptor 的子类拦截。
2、获取资源访问策略，FilterSecurityInterceptor会从 SecurityMetadataSource 的子类DefaultFilterInvocationSecurityMetadataSource 获取要访问当前资源所需要的权限Collection 
3、FilterSecurityInterceptor会调用 AccessDecisionManager 进行授权决策，若决策通过，则允许访问资源，否则将禁止访问。
关于AccessDecisionManager接口，最核心的就是其中的decision方法。这个方法就是用来鉴定当前用户是否有访问对应受保护资源的权限。



#### 授权的配置方式：
#### **通过HttpSecurity 配置URL的授权信息**
authenticated() 保护URL，需要用户登录
permitAll() 指定URL无需保护，一般应用与静态资源文件
hasRole(String role) 限制单个角色访问。角色其实相当于一个"ROLE_"+role的资源，角色在处理之后自动带上了ROLE_的前缀，实际上也是被当成了Authority授权许可
hasAuthority(String authority) 限制单个权限访问，
hasAnyRole(String… roles)允许多个角色访问. 
hasAnyAuthority(String… authorities) 允许多个权限访问. 
access(String attribute) 该方法使用 SpEL表达式, 所以可以创建复杂的限制. 
hasIpAddress(String ipaddressExpression) 限制IP地址或子网


### **方法授权：通过注解的方式进行授权**
在启动类上标注@EnableGlobalMethodSecurity(securedEnabled=true) 注解，开启 @Secured注解过滤权限
@EnableGlobalMethodSecurity(jsr250Enabled=true)	开启@RolesAllowed 注解过滤权限
@EnableGlobalMethodSecurity(prePostEnabled=true) 使用表达式实现方法级别的安全性，打开后可以使用以下几个注解：
    @PreAuthorize 在方法调用之前,基于表达式的计算结果来限制对方法的访问。例如@PreAuthorize("hasRole('normal') AND hasRole('admin')")
    可以结合@P标签注解来读取方法中的参数：

```java
@PreAuthorize("#userId == authentication.principal.userId or hasAuthority(‘ADMIN’)")
void changePassword(@P("userId") long userId ){}
```

    @PostAuthorize 允许方法调用,但是如果表达式计算结果为false,将抛出一个安全性异常。此注释支持使用returnObject来表示返回的对象。例如
```java
    @PostAuthorize(" returnObject!=null &&  returnObject.username == authentication.name")
```

    @PostFilter 允许方法调用,但必须按照表达式来过滤方法的结果
```java
@PostFilter("filterObject != authentication.principal.username")
public List<String> getAllUsernamesExceptCurrent() {
    return userRoleRepository.getAllUsernames();
}
```
结果中包含当前访问用户名时，移除

    @PreFilter 允许方法调用,但必须在进入方法之前过滤输入值,对输入参数进行过滤
  ```java
    @PreFilter("filterObject != authentication.principal.username")
public String joinUsernames(List<String> usernames) {
    return usernames.stream().collect(Collectors.joining(";"));
}
  ```
当usernames中的子项与当前登录用户的用户名不同时，则保留；当usernames中的子项与当前登录用户的用户名相同时，则移除。
比如当前使用用户的用户名为zhangsan，此时usernames的值为{"zhangsan", "lisi", "wangwu"}，则经@PreFilter过滤后，实际传入的usernames的值为{"lisi", "wangwu"}


用户登录信息的获取    
可以通过为SecurityContextHolder.getContext().getAuthentication()获取当前登录用户信息

在需要权限管理的方法上使用@Secured(Resource) 方式配合权限，规定了访问访方法的角色列表，在列表中最少指定一种角色,指定多个时，当用户拥有列表中任意一种角色是可以访问方法

在spring security6中@EnableGlobalMethodSecurity注解已经被弃用，使用@EnableMethodSecurity来开启方法授权

#### 使用方法注解时需要注意的问题
默认情况下，在方法中使用安全注解是由Spring AOP代理实现的，这意味着：如果我们在方法1中去调用同类中的使用安全注解的方法2，则方法2上的安全注解将失效。

Spring Security上下文是线程绑定的，这意味着：安全上下文将不会传递给子线程。


## 分布式系统认证方案
分布式认证系统需要实现以下的功能：
* 统一认证授权服务
* 多样的认证场景，支持多种认证方式，如用户名密码认证、短信验证码、二维码、人脸识别等
* 应用接入认证，提供扩展开开放的能力，提供安全的系统对接机制，可开放部分API给第三方使用，内部服务和外部第三方服务均采用统一的介入机制

### 分布式认证方案
**基于session的方式实现**
用户认证通过之后，由服务端统一保存用户的认证信息，服务端可以对会话进行控制
session机制依赖于cookie，客户端需要保存sessionId，而不是所有的客户端都支持cookie，可能会导致多客户端下不能有效使用的问题
**基于token的认证方式**
使用token存储用户的认证信息，不需要服务端再存储认证数据信息，易于维护，可扩展性强，客户端可以把token存储在任意地方，实现web页面和app的统一认证机制
由于token中包含大量的信息，数据量较大，每次请求都需要传递，会占用额外的带宽，token的签名延签操作也会给系统带来额外的负担

### oauth2.0协议
#### 角色：
1、客户端 - 示例中的浏览器、微信客户端
本身不存储资源，需要通过资源拥有者的授权去请求资源服务器的资源。
2、资源拥有者 - 示例中的用户(拥有微信账号)
通常是用户，也可以是应用程序，即该资源的拥有者。
3、授权服务器(也称为认证服务器) - 示例中的微信
用于服务提供者对资源拥有的身份进行认证，对访问资源进行授权，认证成功后会给客户端发放令牌(access_token)，作为客户端访问资源服务器的凭据。
4、资源服务器 - 微信 和 百度等提供资源获取的服务
一些重要概念：
clientDetails(client_id)：客户信息。代表百度 在微信中的唯一索引。 在微信中用appid区分

secret：秘钥。代表百度获取微信信息需要提供的一个加密字段。这跟微信采用的加密算法有关。

scope：授权作用域。代表百度可以获取到的微信的信息范围。例如登录范围的凭证无法获取用户信息范围的信息。

access_token：授权码。百度获取微信用户信息的凭证。微信中叫做接口调用凭证。

grant_type： 授权类型。例如微信目前仅支持基于授权码的 authorization_code 模式。而OAuth2.0还可以有其他的授权方式，例如输入微信的用户名和密码的方式。

userDetails(user_id)：授权用户标识。在示例中代表用户的微信号。 在微信中用openid区分.


### spring security oauth2.0
是auth 2.0的一种实现框架，
oauth2.0 包含两个服务：
授权服务（Authorization server，也叫认证服务）
资源服务（resource server） 
一般在部署时可以在同一个应用中实现这两个服务

授权服务包含对接入端、登入用户的合法性进行验证并颁发token等功能，对令牌的请求断点由spring mvx控制器实现：
默认接口：
认证接口：AuthorizationEndponit, /oauth/authorize
令牌颁发接口： TokenEndpoint ， /oauth/token
Oauth2AuthenticationProcessingFilter用来对请求给出的身份令牌进行解析鉴权

授权服务：
使用@EnableAuthorizationServer 标识启动
通过重写AuthorizationServerConfigurerAdapter的configure方法来实现对基础认证授权功能的配置
public class AuthorizationServerConfigurerAdapter implements AuthorizationServerConfigurer {
   public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {}
   public void configure(ClientDetailsServiceConfigurer clients) throws Exception {}
   public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {}
}
ClientDetailsServiceConfigurer 配置客户端详情服务（ClientDetailsService），客户端详情信息在这里进行初始化，把客户端详情写死在这里或者通过数据库来存储调取详情信息
AuthorizationServerEndpointsConfifigurer 用来配置令牌token的访问点和令牌服务tokenservices
AuthorizationServerSecurityConfifigurer 配置令牌端点的安全约束


客户端详情配置：
ClientDetailsServiceConfigurer
```java
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        //内存配置的方式配置用户信息
        clients.inMemory()//内存方式
                .withClient("c1") //client_id
                .secret(new BCryptPasswordEncoder().encode("secret"))//客户端秘钥
                .resourceIds("order")//客户端拥有的资源列表
                .authorizedGrantTypes("authorization_code",
                        "password", "client_credentials", "implicit", "refresh_token")//该client允许的授权类型
                .scopes("all")//允许的授权范围
                .autoApprove(false)//跳转到授权页面
                .redirectUris("http://www.baidu.com");//回调地址
//                .and() //继续注册其他客户端
//                .withClient()
//                ...
//   加载自定义的客户端管理服务 //   clients.withClientDetails(clientDetailsService);
    }
```


clientId: 用来标识客户的ID。必须。
secret: 客户端安全码，如果有的话。在微信登录中就是必须的。
scope： 用来限制客户端的访问范围，如果是空(默认)的话，那么客户端拥有全部的访问范围。
authrizedGrantTypes：此客户端可以使用的授权类型，默认为空。在微信登录中，只支持authorization_code这一种。
authorities：此客户端可以使用的权限(基于Spring Security authorities)
redirectUris：回调地址。授权服务会往该回调地址推送此客户端相关的信息。

AuthorizationServerEndpointsConfigurer 令牌服务以及令牌服务的各个endponit配置
```java
   @Autowired
	private AuthorizationCodeServices authorizationCodeServices;
	@Autowired
	private AuthenticationManager authenticationManager;
   
   @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
//                .pathMapping("/oauth/confirm_access","/customer/confirm_access")//定制授权同意页面
                .authenticationManager(authenticationManager)//认证管理器
                .userDetailsService(userDetailsService)//密码模式的用户信息管理
                .authorizationCodeServices(authorizationCodeServices)//授权码服务
                .tokenServices(tokenService())//令牌管理服务
                .allowedTokenEndpointRequestMethods(HttpMethod.POST);
    }
    
        //设置授权码模式的授权码如何存取，暂时用内存方式。
    @Bean
    public AuthorizationCodeServices authorizationCodeServices(){
        return new InMemoryAuthorizationCodeServices();
        //JdbcAuthorizationCodeServices
    }
```

AuthorizationServerTokenService 定义一些对令牌进行管理的必要操作，令牌可以被用来夹在用户信息，令牌中包含了相关的权限信息。
通过这个service可以指定令牌的格式和令牌的存储方式
令牌格式使用OAuth2AccessToken的格式
令牌存储方式：
InMemoryTokenStore，在内存中存储token信息
JdbcTokenStore 基于jdbc的实现类，令牌被保存在关系型数据库中，使用这个实现类可以在不同的服务器之间共享令牌信息
JwtTokenStore 使用JSON Web Token，把令牌的信息全部编码整合到令牌本身，服务端可以不用存储令牌相关信息，撤销一个令牌会比较困难，通常用来处理生命周期较短的令牌；另外jwt方式存储可能会比较大，其中包含了用户的凭证信息

AuthorizationServerSecurityConfifigurer 配置令牌端点的安全约束
```java
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security
                .tokenKeyAccess("permitAll()") // oauth/token_key公开
                .checkTokenAccess("permitAll()") // oauth/check_token公开
                .allowFormAuthenticationForClients(); // 表单认证，申请令牌
    }
```
/oauth/check_token
在用户申请到token之后再次访问资源时，资源服务器会携带token去访问授权服务器的/oauth/check_token，对令牌进行解析

#### 资源服务
使用@EnableResourceServer 标识启动

资源服务的配置：
重写ResourceServerConfigurerAdapter多个configure方法，实现对资源服务的配置
```java
	public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
	}
  public void configure(HttpSecurity http) throws Exception {
		http.authorizeRequests().anyRequest().authenticated();
	}
```
**ResourceServerSecurityConfigurer**

tokenServices : ResourceServerTokenServices类的实例，用来实现令牌服务，即如何验证令牌。
tokenStore ： TokenStore类的实例，指定令牌如何访问，与tokenServices配置可选
resourceId ： 这个资源服务的ID，是可选的。但是推荐设置并在授权服务中进行验证。
其他的扩展属性例如tokenExtractor令牌提取器用来提取请求中的令牌。
**HttpSecurity**
这个配置与Spring Security类似
authorizeRequests()方法验证请求。antMatchers方法匹配访问路径。access()方法配置需要的权限。
.sessionManagement()方法配置session管理策略。
其他自定义权限保护规则也通过HttpSecurity来配置。


### jwt令牌
JWT （JSON Web Token）是一个开放的行业标准，定义了一种简单的、子包含的协议格式，用于在通信双方传递json对象，传递的信息经过数据签名可以被验证和信任，JWT可以使用RSA算法的公私钥来签名
由三部分组成：头部（Header）、载荷（Payload）和签名（Signature）。头部包含了关于令牌的元数据，如算法和令牌类型。载荷包含了要传输的信息，如用户ID、权限等。签名是使用私钥对头部和载荷进行加密生成的，用于验证令牌的真实性。这三部分通过点号连接在一起，形成一个JWT令牌。
**令牌结构**
Header.Payload.Signature
header
头部包括令牌的类型以及使用hash算法
{
 "alg": "HS256",
 "typ": "JWT"
} 
将上面的内容使用Base64URL编码，就得到了JWT令牌的第一个部分。
Payload
第二部分是负载，内容也是一个对象，他是存放有效信息的地方，他可以存放JWT提供的现有字段
Signature
签名，此部分用于防止JWT内容被篡改，
使用Base64url将前两部分进行编码，编码后使用点(.)连接组成字符串，最后使用header中声明的签名算法进行签名。

### 在spring security oauth2.0中使用jwt格式令牌
认证授权服务端的配置jwtStore
**首先需要注入jwtTokenStore**
```java
@Configuration
public class TokenConfig {
    private static final String SIGN_KEY="uaa";
    // 使用JWT令牌。
    @Bean
    public TokenStore tokenStore(){
        return new JwtTokenStore(accessTokenConvert());
    }
    @Bean
    public JwtAccessTokenConverter accessTokenConvert(){
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey(SIGN_KEY);
        return converter;
    }
}
```
然后在认证授权配置类中将tokenservice中的tokenstore修改为jwtTokenStore，accessTokenConvert改成JwtAccessTokenConverter
```java
    //使用JWT令牌
    @Autowired
    private JwtAccessTokenConverter accessTokenConverter;
    ...
    public AuthorizationServerTokenServices tokenService() {
        DefaultTokenServices service = new DefaultTokenServices();
        service.setClientDetailsService(clientDetailsService); //客户端详情服务
        service.setSupportRefreshToken(true); //允许令牌自动刷新
        service.setTokenStore(tokenStore); //令牌存储策略-内存
    	//使用JWT令牌
        service.setTokenEnhancer(accessTokenConverter);
        service.setAccessTokenValiditySeconds(7200); // 令牌默认有效期2小时
        service.setRefreshTokenValiditySeconds(259200); // 刷新令牌默认有效期3天
        return service;
    }
```

资源服务端配置使用jwt令牌，取消原来的check_token的配置
配置jwt格式存储令牌
```java
@Configuration
public class TokenConfig {
    private static final String SIGN_KEY="uaa";
    // 使用JWT令牌。
    @Bean
    public TokenStore tokenStore(){
        return new JwtTokenStore(accessTokenConvert());
    }
    @Bean
    public JwtAccessTokenConverter accessTokenConvert(){
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey(SIGN_KEY);
        return converter;
    }
}
```
资源服务器配置中直接使用配置好的jwtTokenStore
```java
    //使用JWT令牌，需要引入与uaa一致的tokenStore，存储策略。
    @Autowired
    private TokenStore tokenStore;
    ...
    //    使用JWT令牌就不再需要远程解析服务了，资源服务可以在本地进行解析。
    //    public ResourceServerTokenServices tokenServices(){
        DefaultTokenServices services = new DefaultTokenServices();
//        RemoteTokenServices services = new RemoteTokenServices();
//        services.setCheckTokenEndpointUrl("http://localhost:53020/uaa/oauth/check_token");
//        services.setClientId("c1");
//        services.setClientSecret("secret");
//        return services;
//    }

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.resourceId(RESOURCE_SALARY) //资源ID
//                .tokenServices(tokenServices()) //使用远程服务验证令牌的服务
                //使用JWT令牌验证，就不需要调用远程服务了，用本地验证方式就可以了。
                .tokenStore(tokenStore)
                .stateless(true); 
    }
```

### httpSecurity配置项
1、openldLogin()  用于基于Openld的脸证
2、headers()  将安全标头添加到响应
3、cors()  配置跨域资源共享( CORS )
4、sessionManagement()  允许配置会话管理
5、portMapper()  允许配置一个PortMapper(HttpSecurity#(getSharedObject(class)))，其他提供SecurityConfigurer的对象使用 PortMapper 从 HTTP 重定向到 HTTPS 或者从 HTTPS 重定向到 HTTP。默认情况下，Spring Security使用一个PortMapperImpl映射 HTTP 端口8080到 HTTPS 端口8443，HTTP 端口80到 HTTPS 端口443
6、jee() 配置基于容器的预认证。 在这种情况下，认证由Servlet容器管理
7、x509() 配置基于x509的认证
8、rememberMe 允许配置“记住我”的验证
9、authorizeRequests() 允许基于使用HttpServletRequest限制访问
10、requestCache() 允许配置请求缓存
11、exceptionHandling() 允许配置错误处理
12、securityContext() 在HttpServletRequests之间的SecurityContextHolder上设置SecurityContext的管理。 当使用WebSecurityConfigurerAdapter时，这将自动应用
13、servletApi() 将HttpServletRequest方法与在其上找到的值集成到SecurityContext中。 当使用WebSecurityConfigurerAdapter时，这将自动应用
14、csrf()  添加 CSRF 支持，使用WebSecurityConfigurerAdapter时，默认启用
15、logout() 添加退出登录支持。当使用WebSecurityConfigurerAdapter时，这将自动应用。默认情况是，访问URL”/ logout”，使HTTP Session无效来清除用户，清除已配置的任何#rememberMe()身份验证，清除SecurityContextHolder，然后重定向到”/login?success”
16、anonymous() 允许配置匿名用户的表示方法。 当与WebSecurityConfigurerAdapter结合使用时，这将自动应用。 默认情况下，匿名用户将使用org.springframework.security.authentication.AnonymousAuthenticationToken表示，并包含角色 “ROLE_ANONYMOUS”
17、formLogin() 指定支持基于表单的身份验证。如果未指定FormLoginConfigurer#loginPage(String)，则将生成默认登录页面
18、oauth2Login() 根据外部OAuth 2.0或OpenID Connect 1.0提供程序配置身份验证
19、requiresChannel() 配置通道安全。为了使该配置有用，必须提供至少一个到所需信道的映射
20、httpBasic() 配置 Http Basic 验证
21、addFilterAt() 在指定的Filter类的位置添加过滤器

保护URL常用的方法有(authorizeRequests())：
1、authenticated()  保护URL，需要用户登录
2、permitAll() 指定URL无需保护，一般应用与静态资源文件
3、hasRole(String role) 限制单个角色访问，角色将被增加 “ROLE_” .所以”ADMIN” 将和 “ROLE_ADMIN”进行比较.
4、hasAuthority(String authority) 限制单个权限访问
5、hasAnyRole(String… roles)允许多个角色访问.
6、hasAnyAuthority(String… authorities) 允许多个权限访问.
7、access(String attribute) 该方法使用 SpEL表达式, 所以可以创建复杂的限制.
8、hasIpAddress(String ipaddressExpression) 限制IP地址或子网



