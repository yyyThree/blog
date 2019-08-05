# Spring Cloud Gateway 入门

## 一、名词解释 ##

 - Spring Cloud Gateway：Spring官方推出的网关组件，它旨在为微服务架构提供一种简单有效的统一的API路由管理方式。不仅提供统一的路由方式，并且提供安全、监控/指标、限流等功能。
 - Route（路由）：网关的基本构建块，由一个ID，一个目URI，一组Predicate（断言）和一组Filter（过滤器）定义。
 - Predicate（断言）：可以使用它来匹配来自HTTP请求的任何内容，例如headers 或请求参数，如果断言为真，则路由匹配。
 - Filter（过滤器）：用于过滤/修改请求，以及修改返回内容。

## 二、项目构建 ##
1. **使用gradle进行项目构建**

2. **基础构建demo**
    
    ```
    buildscript {
        ext {
            springBootVersion = '2.0.5.RELEASE'
        }
        repositories {
            mavenCentral()
        }
        dependencies {
            classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
        }
    }
    
    plugins {
        id 'org.springframework.boot' version '2.0.5.RELEASE'
        id 'java'
    }
    
    apply plugin: 'io.spring.dependency-management'
    
    group = 'com.duomai'
    version = '1.0.0'
    sourceCompatibility = '1.8'
    
    bootJar {
        baseName = 'gate_way'
        version =  1.0
    }
    
    repositories {
        mavenCentral()
    }
    
    dependencyManagement {
        imports {
            mavenBom "org.springframework.cloud:spring-cloud-dependencies:Finchley.SR2"
        }
    }
    
    dependencies {
        implementation 'org.springframework.cloud:spring-cloud-starter-gateway'
        implementation 'org.springframework.cloud:spring-cloud-starter-netflix-hystrix'
        implementation('org.springframework.cloud:spring-cloud-starter-contract-stub-runner') {
            exclude group: "org.springframework.boot", module: "spring-boot-starter-web"
        }
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
    }
    
    ```
3. **构建注意项**

    - Spring Boot` 和 `Sping Cloud` 版本兼容问题
      2.1.x版本的`Spring Boot`与2.0.x版本的`Spring Cloud`不兼容，需要将`Spring Boot`与降至2.0.x版本或者将`Spring Cloud`升至`Greenwich.RC2`版本

4. 项目启动

    - 本例中网关启动链接为 `http://192.168.3.102:9000`

## **三、文件分层** ##
```
// 业务代码 src/main
java 
    com
        duomai
            gate_way
                [Prefix]Application.java // 项目启动文件，可以做一些全局设置，如时区设置等
                Route // 路由类，存放网关匹配的各类路由文件
                Filter // 过滤器类，存放自定义过滤器、全局过滤器
                Library // 库类，存放公共类文件/纯定义文件等
                    ApiReturnDefines.java // 接口返回定义
                    ExceptionErrprDefines.java // 异常监听层定义
                Helper // 辅助函数类文件
                OutPut // 接口输出层
                    ApiResult.java // 最终的接口输出格式
                    AiReturn.java // 快速生成ApiResult类，供外部调用
                Exception // 统一的异常处理
                    
// 配置项
resources
    appliaction.properties // 项目配置文件
```
## **四、路由文件编写** ##
1. **为每个需要配置网关的子模块建立单独的文件，例：**
    
    ```
    @Configuration
    public class BalanceCardRoute {
        @Value("${proxy.balanceCard.url}")
        public String balanceCardUrl;
    
        @Bean
        public RouteLocator myRoutes(RouteLocatorBuilder builder) {
            return builder.routes()
                    .route("balanceCard", p -> p
                                    .path("/card/*")
                                    .filters(f -> f.addRequestHeader("Hello", "World"))
                                    .uri(balanceCardUrl)
                            .order(0)
                    )
                    .route("balanceCard2", p -> p
                            .path("/card/get/*")
                            .filters(f -> f.addRequestHeader("Hello", "World"))
                            .uri(balanceCardUrl)
                            .order(1)
                    )
                    .build();
        }
    }
    
    ```
2. **详解**

    - 使用 `@Configuration` 将类注册为配置类
    - 使用 `@Value("${proxy.balanceCard.url}")` 将 `appliaction.properties` 中的配置项注入到类中
    - `route` 表示为一条路由匹配规则，一个方法可以注册多条，执行顺序按照order从小到大开始，匹配成功之后将不再执行后续规则。
    - `balanceCard` 为每条  `route` 的标识
    - `path` 是 `Predicate`（断言）语法的一部分，表示匹配的请求路径`
    - `filters` 表示过滤规则` 
    - `uri` 表示完全匹配之后将请求转发的地址` 
    - `order` 表示此条匹配规则对应的顺序权重

## **五、Predicate（断言）使用** ##
1. **匹配查询路径**

    ```
    return builder.routes()
            .route("balanceCard", p -> p
                            .path("/card/get")
                            // .path("/card/*")
                            .uri(balanceCardUrl)
            )
            .build();
    ```
    
    - 表示匹配路径为 `http://192.168.3.102:9000/card/get` 的请求, 也可以使用 `.path("/card/*")` 来匹配 `http://192.168.3.102:9000/card/*` 的请求。
      <br>
    
2. **匹配指定请求的服务器的域名host**

    ```
    return builder.routes()
            .route("balanceCard", p -> p
                            .host("192.168.3.102:9000")
                            // .host("192.168.3.*")
                            .uri(balanceCardUrl)
            )
            .build();
    ```

    - 表示匹配`Request Heads` 中 `host` 为 `192.168.3.102:9000` 的请求，也可以使用 `.host("192.168.3.*")` 进行批量匹配
      <br>

3. **多条Predicate同时使用**

    ```
    return builder.routes()
            .route("balanceCard", p -> p
                            .path("/card/*")
                            .and()
                            .host("192.168.3.*")
                            .uri(balanceCardUrl)
            )
            .build();
    ```

    - .and()` 表示 与

    - `.or()` 表示 或

    - `.negate()` 表示 非

4. **更多**

  `Predicate` 还有匹配请求时间、校验cookie、校验请求参数、校验ip等一系列功能。
  [参考文档](https://cloud.spring.io/spring-cloud-gateway/spring-cloud-gateway.html#gateway-request-predicates-factories)

## **六、Filter（过滤器）使用TODO** ##
## **七、自定义GatewayFilter** ##
1. **方法一：实现`GatewayFilter`接口**

   1. 在`Filter`文件夹下创建`AuthGatewayFilter.java`，在此例中校验token信息。
   
      ```
       public class AuthGatewayFilter implements GatewayFilter, Ordered {
           private static final String AUTHORIZE_TOKEN = "Authorization";
           private static final String AUTHORIZE_TOKEN_PREFIX = "Bearer ";
       
           private Logger logger = LoggerFactory.getLogger(AuthGatewayFilter.class);
           @Override
           public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
               System.out.print("before1");
               ServerHttpRequest request = exchange.getRequest();
               HttpHeaders headers = request.getHeaders();
               String auth_token = headers.getFirst(AUTHORIZE_TOKEN);
       
               // step1 读取token
               if (auth_token == null || auth_token.isEmpty()) {
                   ApiResult res =  ApiReturn.fail(ApiReturnDefines.AUTH_TOKEN_NOT_FOUND, "没有获取到token");
                   return Common.filterReturn(exchange, res);
               }
       
               if (!auth_token.startsWith(AUTHORIZE_TOKEN_PREFIX)) {
                   ApiResult res =  ApiReturn.fail(ApiReturnDefines.AUTH_TOKEN_NOT_ALLOWED, "token非法，请确保使用的Bearer Token");
                   return Common.filterReturn(exchange, res);
               }
       
               String token = auth_token.replaceFirst(AUTHORIZE_TOKEN_PREFIX, "");
               System.out.print(token);
       
               // step2 解析token
       
               // step3 鉴权
       
               return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                   System.out.print("after1");
               }));
           }
       
           @Override
           public int getOrder() {
               return 0;
           }
       }
      ```
   
      - 校验不符，可直接使用过滤器中断请求并返回。
   
      - 请求访问流程：`filter` 方法(不包含有效 `return`，此例中的 `"before1"`) -> 网关调用 -> `filter` 方法中`return`中的回调函数（此例中打印的`"after1"`）
   
      - 使用 `getOrder` 方法来设定此过滤器的顺序权重，多个过滤器将按照 `order` 值由小到大执行（`"before1"` -> `"before2"` -> 网关调用 -> `"after1"` -> `"after2"`）
   
   2. 应用自定义 `GatewayFilter`
   
      ```
      public RouteLocator myRoutes(RouteLocatorBuilder builder) {
      	System.out.print(balanceCardUrl);
      	return builder.routes()
      		.route("balanceCard", p -> p
      		.path("/card/*")
      		.uri(balanceCardUrl)
      		.filters(new AuthGatewayFilter())
      	)
      	.build();
      }
      ```
   
      - 使用 `.filters(new AuthGatewayFilter())` 将自定义过滤器应用至此条路由
   
      - 可以同时使用多个自定义过滤器： `.filters(new AuthGatewayFilter(), new AuthGatewayFilter2(), new ...)`
   
2. **方法二：继承AbstractGatewayFilterFactory类（TODO）**

## **八、全局过滤器** ##
1. **在`Filter`文件夹下创建`AuthGlobalFilter.java`**
    
    ```
    @Component
    public class AuthGlobalFilter implements GlobalFilter, Ordered {
        private static final String AUTHORIZE_TOKEN = "Authorization";
        private static final String AUTHORIZE_TOKEN_PREFIX = "Bearer ";
    
        private Logger logger = LoggerFactory.getLogger(AuthGatewayFilter.class);
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            System.out.print("before1");
            ServerHttpRequest request = exchange.getRequest();
            HttpHeaders headers = request.getHeaders();
            String auth_token = headers.getFirst(AUTHORIZE_TOKEN);
    
            // step1 读取token
            if (auth_token == null || auth_token.isEmpty()) {
                ApiResult res =  ApiReturn.fail(ApiReturnDefines.AUTH_TOKEN_NOT_FOUND, "没有获取到token");
                return Common.filterReturn(exchange, res);
            }
    
            if (!auth_token.startsWith(AUTHORIZE_TOKEN_PREFIX)) {
                ApiResult res =  ApiReturn.fail(ApiReturnDefines.AUTH_TOKEN_NOT_ALLOWED, "token非法，请确保使用的Bearer Token");
                return Common.filterReturn(exchange, res);
            }
    
            String token = auth_token.replaceFirst(AUTHORIZE_TOKEN_PREFIX, "");
            System.out.print(token);
    
            // step2 解析token
    
            // step3 鉴权
    
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                System.out.print("after1");
            }));
        }
    
        @Override
        public int getOrder() {
            return FilterOrderDefines.GLOBAL_FILTER_AUTH;
        }
    }
    
    ```
    
    与自定义 `GatewayFilter` 接口中方法一的区别仅在于实现的是 `GlobalFilter` 接口
    
2. **应用全局过滤器**
  只需要添加 `@Component` 注解，不需要进行任何额外的配置，实现 `GlobalFilter` 接口之后，过滤器自动会对所有的路由生效。

## **九、单元测试TODO** ##
## **十、容器化部署** ##
1. **创建 `DOCKER` 镜像(同Sping Boot入门篇)**
2. **环境变量配置**
    
    ```
    server.port=${APP_SERVER_PORT:9000}
    
    #log配置
    logging.path=${APP_LOG_PATH:E:/java/demo/gate_way/log}
    logging.level.org.springframework.web=${APP_LOG_SPRING_WEB_LEVEL:ERROR}
    
    #网关代理地址配置
    proxy.balanceCard.url=${APP_PROXY_BALANCE_CARD_URL:http://192.168.3.14:10012}
    ```
3. **启动容器**( `docker/run.sh` )

    - 启动参数

      ```
      -p // 宿主机运行端口
      -d // 宿主机项目地址，缺省为执行run.sh的上一级目录
      -n // 容器名称
      -v // 镜像版本号
      -j // jar包版本，缺省为1.0
      ```

    - 环境变量

      ```
      # 运行配置
      APP_SERVER_PORT // 容器运行jar包的端口
      # 日志配置
      APP_LOG_PATH // 日志输出地址
      APP_LOG_SPRING_WEB_LEVEL // 对应logging.level.org.springframework.web，指org.springframework.web这个包下的日志输出等级，默认为ERROR，开发环境可配置为DEBUG
      # 网关代理地址配置
      APP_PROXY_BALANCE_CARD_URL // 余额储值卡微服务实际请求地址
      ```

    - 启动容器

      ```
      docker run -d \
      -e APP_SERVER_PORT=8080 \
      -e APP_LOG_PATH=/opt/ci123/www/html/java/gate_way/log \
      -e APP_LOG_SPRING_WEB_LEVEL=DEBUG \
      -e APP_PROXY_BALANCE_CARD_URL=http://192.168.3.14:10012 \
      -p $port:80 \
      -v $dir:/opt/ci123/www/html/java/gate_way \
      --restart=always \
      --name $name \
      harbor.oneitfarm.com/duomai/java-gate_way:$version \
      sh /opt/ci123/www/html/java/gate_way/docker/start.sh -j $jarName
      ```