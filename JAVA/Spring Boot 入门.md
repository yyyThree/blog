# Spring Boot 入门

## 一、开发基础

 - Java基础（两到三小时过一遍）
 <br>
 - [Java开发环境配置][1]（**必须使用JDK1.8**）
 <br>
 - IDE安装（优先使用IntelliJ IDEA）

## 二、名词解释

 - Spring：JAVA开发应用框架
 - Spring Boot：用来简化Spring应用的初始搭建以及开发过程的配置框架
 - Maven：Java项目构建工具，成熟的项目
 - Gradle：更简洁的Java项目构建工具，吸收了旧构建工具的优点。
 - JPA：是Sun官方提出的Java持久化规范，即数据库操作规范。
 - Hibernate：Hibernate是一个ORM框架，是JPA的默认实现方式，一般说JPA都是指Hibernate。
 - Mybatis：Mybatis是一个轻便的ORM框架。
 - Spring data jpa：是Spring基于ORM框架、JPA规范的基础上封装的一套JPA应用框架

##  三、新项目流程

1. **新建gradle项目**

    - File->new->Prroject->Spring Initializr
    
    - 填写Group、Artifact选择Gradle Project项目生成
    
    - 可以直接在生成项目的时候选择对应需要安装的插件，如：web、jpa、mybatis等，也可以在项目初始化完成之后在build.gradle中添加/配置
    
2. **配置build.gradle（位于根目录）**

    ```
    plugins {
        id 'org.springframework.boot' version '2.1.3.RELEASE'
        id 'java'
    }
    
    apply plugin: 'io.spring.dependency-management'
    
    group = 'com.duomai'
    version = '0.0.1-SNAPSHOT'
    sourceCompatibility = '1.8' // JDK最大兼容版本
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        // Spring Boot JPA 组件
        implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
        // Spring Boot Web组件
        implementation 'org.springframework.boot:spring-boot-starter-web'
        // Mybatis插件，注意暂时使用**1.1.1**版本，高版本的运行好像有问题
        implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:1.1.1'
        // Mybatis分页插件
        implementation group: 'com.github.pagehelper', name: 'pagehelper-spring-boot-starter', version: '1.2.10'
        runtimeOnly 'org.springframework.boot:spring-boot-devtools'
        runtimeOnly 'mysql:mysql-connector-java'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
    }
    
    ```
修改了build.gradle后，idea会自动安装/更新依赖包。
参考：[gradle官网][2]、[Spring Boot Web服务搭建][3]、[Spring Boot Mysql使用][4]、[Spring Boot JPA使用][5]

3. **项目基础配置（位于**`src/resources/application.properties`**）**

    ```
    #运行配置
    server.port=9000
    #数据格式配置
    spring.jackson.time-zone=GMT+8 // 设置接口返回时区为东八区
    spring.jackson.date-format=yyyy-MM-dd HH:mm:ss // 自动将接口返回中的日期格式转换为标准格式
    #数据库连接配置
    spring.datasource.url=jdbc:mysql://192.168.0.235:3355/shop_balances?serverTimezone=Asia/Shanghai&tinyInt1isBit=false // serverTimezone选择Mysql东八区，tinyInt1isBit禁止Mysql自动将tinyint(1)类型数据映射为boolean类型
    spring.datasource.username=cishop
    spring.datasource.password=fuyuan1906
    #log配置
    logging.path=E:/java/demo/balance_card/log
    logging.level.com.favorites=DEBUG
    #logging.level.org.springframework.web=INFO
    logging.level.org.hibernate=ERROR
    #mybatis设置
    mybatis.type-aliases-package=com.duomai.balance_card.Model.Mapper
    mybatis.configuration.map-underscore-to-camel-case=true
    logging.level.com.duomai.balance_card.Model.Mapper=DEBUG
    #pagehelper插件设置
    pagehelper.helperDialect=mysql
    pagehelper.reasonable=false
    pagehelper.supportMethodsArguments=true
    pagehelper.params=pageNum=page;pageSize=limit
    #jpa 设置
    spring.jpa.properties.hibernate.hbm2ddl.auto=update
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
    spring.jpa.show-sql= true
    ```

4. **项目启动**

    - IDEA启动，运行[Prefix]Application.java文件即可

    - 命令行启动
        - Maven项目
            在springboot的应用的根目录下运行 `mvn spring-boot:run`

        - Grdale项目
            在springboot的应用的根目录下运行 `gradle bootRun` 或 `gradlew bootRun`
    *（前者是使用本地的gradle版本运行，后者是使用代码仓库中的gradle运行）*

    - 打包构建可执行文件运行

5. **[开发环境热更新设置][6]**
    热更新不是很好用，有一定的延迟时间

## 四、项目文件分层解析

```
// 业务代码 src/main
java 
    com
        duomai
            balance_card
                [Prefix]Application.java // 项目启动文件，可以做一些全局设置，如时区设置、Mapper扫描等
                Config // 配置类，用于注册一些全局配置，如拦截器注册等
                Middleware // 中间件，实现AOP功能
                Controller // 控制器，主要做路由功能
                    xxxController.java
                    BaseErrorController.java // 路由匹配失败时使用的控制器
                Service // 业务代码
                Model // 目录主要用于实体与数据访问层
                    Entity // 数据表实体类
                    Repository // JPA数据仓库
                    Mapper // Mybatis映射文件
                    Provider // Mapper的Sql生成器
                Library // 库类，存放公共类文件/纯定义文件等
                    ApiReturnDefines.java // 接口返回定义
                    ExceptionErrprDefines.java // 异常监听层定义
                Helper // 辅助函数类文件
                OutPut // 接口输出层
                    ApiResult.java // 最终的接口输出格式
                    AiReturn.java // 快速生成ApiResult类，供外部调用
                Exception // 统一的异常处理
                    ControllerHandler // 路由层异常监听
                    SqlHandler // 数据库层异常监听
// 配置项
resources
    appliaction.properties // 项目配置文件
```

## 五、控制器中间件

1. **一般使用Spring过滤器或拦截器实现AOP切面编程**

2. **过滤器和拦截器的对比**
   - 作用域不同
     过滤器依赖于servlet容器，只能在 servlet容器，web环境下使用。
     拦截器依赖于spring容器，可以在spring容器中调用，不管此时Spring处于什么环境。
   - 细粒度的不同
     过滤器的控制比较粗，只能在请求进来时进行处理，对请求和响应进行包装。
     拦截器提供更精细的控制，可以分为controller对请求处理之前、渲染视图之后、请求处理之后三个切面。
   - 中断链执行的难易程度不同
     拦截器可以 preHandle方法内返回 false 进行中断。
     过滤器就比较复杂，需要处理请求和响应对象来引发中断，需要额外的动作，比如将用户重定向到错误页面。
     <br>

3. **拦截器的使用**

   - 编写自定义拦截器（`Middleware/ControllerInterceptor.java`）

     ```
     public class ControllerInterceptor implements HandlerInterceptor {
         private Logger logger = LoggerFactory.getLogger(ControllerInterceptor.class);
         @Override
         public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
             logger.info("preHandle....");
             // token校验等
             String token = request.getHeader("token");
     //      Common.sendJson(response, ApiReturn.fail(1001, "token验证失败"));
     //      return false;
             return true;
         }
     
         @Override
         public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
             logger.info("postHandle...");
         }
     
         @Override
         public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
             // 接口日志记录等
             logger.info("afterCompletion...");
         }
     ```

     ```
      **说明**：
     preHandle：对客户端发过来的请求进行前置处理，如果方法返回true,继续执行后续操作，如果返回false，执行中断请求处理，请求不会发送到Controller。可以在这里校验一些权限信息，如token等,校验失败直接以JSON格式返回请求。
     
     postHandler：在请求进行处理后执行，也就是在Controller方法调用之后处理，前提是preHandle方法返回true。具体来说，postHandler方法会在DispatcherServlet进行视图返回渲染前被调用。
     
     afterCompletion: 该方法在整个请求结束之后执行，前提依然是preHandle方法的返回值为true。
     ```

   - 注册拦截器（`Config/InterceptorConfig.java`）

	 ```
     @Configuration
     public class InterceptorConfig implements WebMvcConfigurer {
         @Override
         // 核心方法
         public void addInterceptors(InterceptorRegistry registry) {
             registry.addInterceptor(ControllerInterceptor())
                     //配置拦截规则
                     .addPathPatterns("/**")
                     .order(1);
     
             // 多个拦截器按上述方法持续注册即可,同时也可以设置order值，从小到大执行。
         }
     
         @Bean
         public HandlerInterceptor ControllerInterceptor() {
             return new ControllerInterceptor();
         }
     }
     ```

## 六、Controller层

1. **定义控制器文件**

        @RestController
        在类文件头部定义，标明为控制器文件，且输出格式为JSON

2. **路由和参数**

    1. 定义路由名称，接收方法

    	```
    	例：@RequestMapping(value = "/get", method = {RequestMethod.GET, RequestMethod.POST})
        
          可选参数：
        
          value：路由名称
        
          method：指定请求的method类型， GET、POST、PUT、DELETE、PATCH等,可多选
        
          consumes：指定处理请求的提交内容类型（Content-Type），例如application/json, text/html
        
          produces： 指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回
        
          params： 指定request中必须包含某些参数值
        
          headers： 指定request中必须包含某些指定的header值
		```
    
    2. 参数接收
    
		```
        例：@RequestParam(value = "fields", required = false, defaultValue = "*") String fields

			value：参数名称

			defaultValue：默认值

			required：是否是必要参数
      	```
    
    3. 自定义错误路由
    
        1. 在Controller层中添加BaseErrorController.java文件，用于监听路由匹配失败的情况
    
		```
		@Controller
		public class BaseErrorController implements ErrorController {
                	@Override
                	public String getErrorPath() {
                    		System.out.print("错误页面");
                    		return "error/error";
                	}

                	@RequestMapping(value = "/error")
                	public void error() throws Exception {
                    		throw new Exception("路由匹配失败");
                	}
		}
          	```
    
        2. 在Exception文件夹中添加ControllerHandler.java，用于捕获路由报错并输出。
        
        	```
		@RestControllerAdvice
            public class ControllerHandler {
                // 缺少必选参数
                @ExceptionHandler({MissingServletRequestParameterException.class})
                @ResponseBody
                public ApiResult requestMissingServletRequest(MissingServletRequestParameterException e){
                    return ApiReturn.fail(ExceptionErrorDefines.RequestMissingServletRequest, e.getMessage());
                }
            }
            	未解决：抛出异常后访问404页面运行环境会报错，但是页面正常  
            	```
        
            参考：*https://www.jianshu.com/p/393f70b55b1b*
    
3. **Service层调用**

   1. Service类成员注入
   
   	- 使用@Autowired修饰符进行依赖注入
	
   		```
        	@Autowired
        	private final CardService cardService;
        	```
    
   	- 用构造函数来做注入类成员（推荐使用）
	
        	```
        	private StoreBalanceCardsRepository cardsRepository;
	        public CardController(StoreBalanceCardsRepository cardsRepository) {
        	    this.cardsRepository = cardsRepository;
        	}
        	**注**：
        	IntelliJ IDEA使用依赖注入会有IDE报错，但不影响实际编译运行，如需去除报错提示，需要在Dao层（Respository/Mapper）类开头添加注解 `@Repository`
        	```
        
   2. 调用
   
   	```
    	cardService.get(id, fields);
	```

## 七、Service层

1. **定义Service文件**

    ```
    @Service
    在类文件头部定义，标明为Service文件
    ```
    
2. **注入Model层操作文件**
   
    ```
    private final StoreBalanceCardsRepository storeBalanceCardsRepository;
    private final StoreBalanceCardsMapper storeBalanceCardsMapper;

        CardService(StoreBalanceCardsRepository storeBalanceCardsRepository, StoreBalanceCardsMapper storeBalanceCardsMapper) {
            this.storeBalanceCardsRepository = storeBalanceCardsRepository;
            this.storeBalanceCardsMapper = storeBalanceCardsMapper;
        }
    ```
    
3. **调用Model层文件**

    
##  八、Model层

1. **Repository**

    - Spring中概念，概念类似于数据仓库，是Spring data jpa的实现。居于业务层和数据层之间，将两者隔离开来，在它的内部封装了数据查询和存储的逻辑。

    - Repository和传统意义上的DAO的区别：
      Repository蕴含着真正的OO概念，即一个数据仓库角色，负责所有对象的持久化管理。DAO则没有摆脱数据的影子，仍然停留在数据操作的层面上。Repository是相对对象而言，DAO则是相对数据库而言，虽然可能是同一个东西，但侧重点完全不同。

2. **Mapper**

    存放Mybatis数据库关系映射方法

3. **Provider**
    为Mapper层提供的SQL生成器，即将SQL的生成与映射解耦。

4. **Entity**

    - 根据表结构自动生成实体类

        ```
        **注意**：
        i、多次自动生成不会覆盖，如需更新需要把旧的实体类文件删除
        
        ii、有需要的话可以在实体类文件右击generate一键生成构造函数、set get方法、@Autowired等
        
        iii、自动生成的实体类中datetime类型的字段会被转为Java的timestamp类型数据，存储的时候也使用timestamp类型即可
        
        iv、生成出来的实体类catalog = "" 报错，是IDE的报错，不影响使用
        ```
        
        参考文档：[https://cdhand.com/article/81][7]

5. **Hibernate 和 Mybatis 的对比**

    - Hibernate优势：

      ```
      i、DAO层开发比MyBatis简单，Mybatis需要维护SQL和结果映射。
      
      ii、对对象的维护和缓存要比MyBatis好，对增删改查的对象的维护要方便。
      
      iii、数据库移植性很好，MyBatis的数据库移植性不好，不同的数据库需要写不同SQL。
      
      iv、有更好的二级缓存机制，可以使用第三方缓存。MyBatis本身提供的缓存机制不佳。
      ```

    - Mybatis优势：

      ```
      i、MyBatis可以进行更为细致的SQL优化，可以减少查询字段。
      
      ii、MyBatis容易掌握，而Hibernate门槛较高。
      ```

    - 选用Mybatis的原因

      ```
      i、Hibernate无法满足动态获取部分字段的需求，即使是使用Hibernate提供的原始SQL也无法实现
      
      ii、Hibernate的JPA查询只适用于一些简单的情况，如遇到复杂的SQL，Repository中的方法名会很长。这时候又将回到Hibernate的自定义SQL查询，即原生SQL。
      
      iii、Hibernate的维护成本比MyBatis高很多，MyBatis的SQL生成完全取决于开发者，所以SQL修改、维护、优化会比较便利。
      ```

## 九、MyBatis使用

1. **引入MyBatis以及pagehelper分页插件**

    ```
    dependencies {
        implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:1.1.1'
        implementation group: 'com.github.pagehelper', name: 'pagehelper-spring-boot-starter', version: '1.2.10'
    }
    ```

2. **application.properties 添加相关配置**

    ```
    #mybatis设置
    mybatis.type-aliases-package=com.duomai.balance_card.Model.Mapper // 项目中Mapper存放的包
    mybatis.configuration.map-underscore-to-camel-case=true // 自动将sql字段下划线转为驼峰，可以保证取出的数据格式就是数据库中存储的格式
    logging.level.com.duomai.balance_card.Model.Mapper=DEBUG // 开启DEBUG模式，用于开发环境，记录执行的SQL。
    #pagehelper插件设置
    pagehelper.helperDialect=mysql // 指定分页插件使用哪种数据库
    pagehelper.reasonable=false // 分页合理化参数，默认值为false。当该参数设置为 true 时，pageNum<=0 时会查询第一页， pageNum>pages（超过总数时），会查询最后一页。默认false 时，直接根据参数进行查询。
    pagehelper.supportMethodsArguments=true // 支持通过 Mapper 接口参数来传递分页参数，默认值false，分页插件会从查询方法的参数值中，自动根据上面 params 配置的字段中取值，查找到合适的值时就会自动分页，设置后无需手动启用分页插件。
    pagehelper.params=pageNum=page;pageSize=limit //  自定义Mapper 接口参数来传递分页参数的参数名
    ```

3. **添加Mapper类扫描的两种方式**

    - 在启动类中添加对mapper包扫描@MapperScan（推荐使用）

      ```
      @SpringBootApplication
      @MapperScan("com.duomai.balance_card.Model.Mapper")
      ```

    - 在具体的Mapper类上面添加注解 `@Mapper`
      <br>

4. **Mapper类SQL用例**

    ```
    public interface StoreBalanceCardsMapper {
        // select用例
        @SelectProvider(type = StoreBalanceCardsSqlBuilder.class, method = "getById")
        List<Map> getById(@Param("id") int id, @Param("fields") String fields, @Param("page") int page, @Param("limit") int limit);
    
        // 列表
        @SelectProvider(type = StoreBalanceCardsSqlBuilder.class, method = "list")
        Page<Map> list(@Param("fields") String fields, @Param("page") int page, @Param("limit") int limit);
        
        // insert用例
        @InsertProvider(type = StoreBalanceCardsSqlBuilder.class, method = "add")
        @Options(useGeneratedKeys=true, keyProperty="storeBalanceCard.id", keyColumn="id")
        int add(@Param("storeBalanceCard") StoreBalanceCards storeBalanceCard);
        
        // update用例
        @UpdateProvider(type = StoreBalanceCardsSqlBuilder.class, method = "updateName")
        @Options(useGeneratedKeys=true, keyProperty="storeBalanceCard.id", keyColumn="id")
        int updateName(@Param("storeBalanceCard") StoreBalanceCards storeBalanceCard, @Param("limit") int limit);
    
        // delete用例
        @DeleteProvider(type = StoreBalanceCardsSqlBuilder.class, method = "deleteById")
        int delete(@Param("id") int id, @Param("limit") int limit);
    }
    ```

    - 通过Provider的方式动态获取SQL

    - 需要分页时在具体的方法后多加page、limit参数，自动实现分页

      ```
      // Service层调用
      List storeBalanceCard = storeBalanceCardsMapper.list(fields, page, limit);
      // Mapper层实现
      Page<Map> list(@Param("fields") String fields, @Param("page") int page, @Param("limit") int limit);
      ```

    - 分页组件仅可用于查询，不可用于更新/删除，更新/删除需要另外实现

5. **Providerle类SQL用例**

    ```
    public class StoreBalanceCardsSqlBuilder {
        private static final String STORE_BALANCE_CARDS = "store_balance_cards";
        public static String getById(int id, String fields) {
            return new SQL(){ { 
                SELECT(fields);
                FROM(STORE_BALANCE_CARDS);
                WHERE("store_balance_cards.id = #{id}");
            } }.toString();
        }
    
        public static String list(String fields) {
            return new SQL(){ {
                SELECT(fields);
                FROM(STORE_BALANCE_CARDS);
            } }.toString();
        }
    
        public static String add(@Param("storeBalanceCard") StoreBalanceCards storeBalanceCard) {
            return new SQL(){ {
                INSERT_INTO(STORE_BALANCE_CARDS);
                VALUES("sys_name", "#{storeBalanceCard.sysName}");
                VALUES("store_id", "#{storeBalanceCard.storeId}");
                VALUES("entity_store_id", "#{storeBalanceCard.entityStoreId}");
                VALUES("name", "#{storeBalanceCard.name}");
                VALUES("type", "#{storeBalanceCard.type}");
                VALUES("item_id", "#{storeBalanceCard.itemId}");
                VALUES("photo", "#{storeBalanceCard.photo}");
                VALUES("state", "#{storeBalanceCard.state}");
                VALUES("upgrade", "#{storeBalanceCard.upgrade}");
                VALUES("recharge_discount", "#{storeBalanceCard.rechargeDiscount}");
                VALUES("updated_at", "#{storeBalanceCard.updatedAt}");
                VALUES("created_at", "#{storeBalanceCard.createdAt}");
            } }.toString();
        }
    
        public static String updateName(@Param("storeBalanceCard") StoreBalanceCards storeBalanceCard, @Param("limit") int limit) {
            return new SQL(){ {
                UPDATE(STORE_BALANCE_CARDS);
                SET("name=#{storeBalanceCard.name}");
                if (storeBalanceCard.getUpdatedAt() != null) {
                    SET("updated_at=#{storeBalanceCard.updatedAt}");
                }
                WHERE("store_id = #{storeBalanceCard.storeId}");
            } }.toString() + " limit " + limit;
        }
    
        public static String deleteById(int id, int limit) {
            return new SQL(){ {
                DELETE_FROM(STORE_BALANCE_CARDS);
                WHERE("store_balance_cards.id = #{id}");
            } }.toString() + " limit " + limit;
        }
    }
    ```

6. [分页组件详解][8]

## 十、接口输出

1. **接口统一输出格式**
   
    ```
    state: // 状态位
    msg: // 接口输出提示信息
    data: { // 总的接口输出数据，可以为空
        data: // 接口数据
        other: // 其他返回数据，如total/page_total等
        ...:
    }
    ```
    
2. **正常接口输出/异常监听输出统一使用**`OutPut/ApiReturn`**方法**

##  十一、异常处理

1. **项目运行异常统一监听**
现有：`ControllrHandler` 路由异常监听、`SqlHandler` 数据库操作异常监听
**需要持续添加**
2. **统一使用接口输出类进行返回，杜绝直接返回报错信息。**

## 十二、单元测试

1. **添加单元测试类**

    `src\test\java\com\duomai\balance_card\BalanceCardApplicationTests.java`

2. **demo**    

    ```
    @RunWith(SpringRunner.class)
    @SpringBootTest
    @AutoConfigureMockMvc
    public class BalanceCardApplicationTests {
    
        @Autowired
        private MockMvc mockMvc;
    
        @Test
        @SuppressWarnings("unchecked")
        public void getCard() throws Exception {
            String card_id = "1";
            String fields = "store_id,item_id,recharge_discount,photo,type,name";
            String res = this.mockMvc.perform(get("/card/get")
                    .param("id", card_id).param("fields", fields)
                    )
                    .andDo(print())
                    .andExpect(status().isOk())
                    .andReturn()
                    .getResponse()
                    .getContentAsString();
           // 接口返回不为空
           assertThat(res).isNotNull();
           // 校验接口返回格式是否完整
           Map<String, Object> api_res = Common.jsonToMap(res);
           assertThat(api_res).isNotNull();
           assertThat(api_res).hasSize(3);
           assertThat(api_res).containsKeys("state", "msg", "data");
    	   // 校验接口返回state是否正确
           int state = (int) api_res.get("state");
           Map<String, Object> api_data = (HashMap<String, Object>) api_res.get("data");
           assertThat(state).isEqualTo(ApiReturnDefines.SUCCESS);
           // 校验data数据是否正确
           assertThat(api_data).containsOnlyKeys("data");
           ArrayList<Map> data = (ArrayList) api_data.get("data");
           String[] fieldsAll = fields.split(",");
           assertThat(data).hasSize(1);
           assertThat(data.get(0)).containsKeys((Object[]) fieldsAll);
        }
    }
    ```

3. **详解**

    - 使用 `@RunWith(SpringRunner.class)`、`@SpringBootTest` 定义测试类
    - 添加注解 `@AutoConfigureMockMvc` ，使用 `Spring MockMvc` 模拟Spring的HTTP请求并将其交给控制器，实际上并没有真正地启动服务器，仅仅是Mock，节省了启动服务器的开销。
    - 使用 `AssertJ` 库类来验证接口返回内容
      在demo中，使用断言验证了接口返回格式、state状态、data数据格式等基本内容。

    **参考文档**：https://spring.io/guides/gs/testing-web/

## 十三、项目打包

1. **不同的构建文件**
   
   - 普通 jar 包 : 会将源码编译后以工具包（即将class打成jar包）的形式对外提供，此时，你的 jar 包不一定要是可执行的，只要能通过编译，可以被别的项目以 import 的方式调用。
   - 可执行 jar 包 : 能通过 java -jar 的命令运行。
   - 普通 war 包 : war 是一个 web 模块，其中包括 WEB-INF，是可以直接运行的 WEB 模块。做好一个 web 应用后，打成包部署到容器中。
- 可执行 war 包 : 普通 war 包 + 内嵌容器 。
  
2. **构建可执行 jar 包**
   - IDE打包
     右侧`Gradle -> Tasks -> build -> build`
   - 命令行打包
     进入项目根目录，执行`gradle build`

3. **运行可执行 jar 包**
    `java -jar build/libs/balance_card-0.0.1.jar`
    运行时可带参数，同`application.properties`中的参数名
    例:
    
    ```
    java -Djava.security.egd=file:/dev/./urandom -jar /opt/ci123/www/html/java/balance_card/build/libs/${jarName}.jar --server.port=80
    ```

## 十四、容器化部署

1. **创建docker镜像**

   - Dockfile 编写

     ```
     FROM java:8
     MAINTAINER duomai
     # 设置镜像源
     COPY sources.list /etc/apt/sources.list
     # 安装扩展
     RUN apt-get update && apt-get install -y \
             wget \
             curl \
             vim \
             git \
             less
     # 配置系统时间
     RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
         && echo 'Asia/Shanghai' > /etc/timezone
     VOLUME /tmp
     # 设置工作目录
     ADD . /opt/ci123/www/html/java
     WORKDIR /opt/ci123/www/html/java
     # 指定输出端口
     EXPOSE 80
     ```
 - 快速构建镜像

        `sh docker/build.sh [version]`

2. **环境变量配置**

   - 配置

     ```
     #数据库连接配置 application.properties
     spring.datasource.url=jdbc:mysql://${APP_DB_HOST:192.168.0.235}:${APP_DB_PORT:3355}/${APP_DB_DATABASE:shop_balances}
     spring.datasource.username=${APP_DB_USER:cishop}
     spring.datasource.password=${APP_DB_PASSWORD:fuyuan1906}
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
     # 数据库配置
     APP_DB_HOST
     APP_DB_PORT
     APP_DB_DATABASE
     APP_DB_USER
     APP_DB_PASSWORD
     # 日志配置
     APP_LOG_PATH // 日志输出地址
     APP_LOG_SPRING_WEB_LEVEL // 对应logging.level.org.springframework.web，指org.springframework.web这个包下的日志输出等级，默认为ERROR，开发环境可配置为DEBUG
     APP_LOG_MYBATIS_LEVEL // 对应logging.level.com.duomai.balance_card.Model.Mapper，指balance_card.Model.Mapper包下的日志输出等级，默认为ERROR，开发环境可配置为DEBUG，开启DEBUG之后可在日志中查看DB操作
     ```

   - 启动容器

     ```
     docker run -d \
     -e APP_SERVER_PORT=8080 \
     -e APP_DB_HOST=192.168.0.235 \
     -e APP_DB_PORT=3355 \
     -e APP_DB_DATABASE=shop_balances \
     -e APP_DB_USER=cishop \
     -e APP_DB_PASSWORD=fuyuan1906 \
     -e APP_LOG_PATH=/opt/ci123/www/html/java/balance_card/log \
     -e APP_LOG_SPRING_WEB_LEVEL=DEBUG \
     -e APP_LOG_MYBATIS_LEVEL=DEBUG \
     -p $port:80 \
     -v $dir:/opt/ci123/www/html/java/balance_card \
     --restart=always \
     --name $name \
     harbor.oneitfarm.com/duomai/java-balance_card:$version \
     sh /opt/ci123/www/html/java/balance_card/docker/start.sh -j $jarName
     ```


[1]: http://www.runoob.com/java/java-environment-setup.html
[2]: https://gradle.org/
[3]: https://spring.io/guides/gs/rest-service/
[4]: https://spring.io/guides/gs/accessing-data-mysql/
[5]: https://spring.io/guides/gs/accessing-data-jpa/
[6]: https://www.cnblogs.com/linkstar/p/8245480.html
[7]: https://cdhand.com/article/81
[8]: https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md
