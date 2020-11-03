# 单元测试

临时记录，待整理！！！

## 一、入门







## 二、Mockito

### 2.1 Mock



Spring项目中的单元测试，Mock对象的自动装配属性的两种方式：

1. InjectMocks + MockitoAnnotations.initMocks: 目标对象完全是自己创建出来的，而不是Spring注入，类属性需要自己装配；当类属性比较多的时候比较繁杂，需要一一声明Mock或者创建对象。

   ```java
       @InjectMocks
   	PrdNavigationService prdNavigationService = new PrdNavigationServiceImpl();
   
       @Mock
       PrdIssueElementService prdIssueElementService;
   
       @Before
       public void setUp() {
           // 注入Mock属性， 这个方法会自动将Mock注入到@InjectMocks注解的对象属性中
           MockitoAnnotations.initMocks(this);
       }
   ```

2. Autowired + ReflectionTestUtils.setField：这种情况只需要对需要Mock的类属性进行Mock并手动注入即可，但是挨个手动反射注入Mock对象。

   ```java
       @Autowired
       PrdNavigationService prdNavigationService;
   
       @Mock
       PrdIssueElementService prdIssueElementService;
       
   	@Before
       public void setUp() {
           // 注入Mock属性，手动反射注入属性
           ReflectionTestUtils.setField(prdNavigationService, "prdIssueElementService", prdIssueElementService);
       }
   ```

   