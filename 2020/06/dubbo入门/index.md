# Dubbo入门


Dubbo 的功能除了基本的 RPC 职能外，核心功能便是监控及服务注册。

## provider 服务

首先创建一个 maven 工程：springboot-provider，再添加一个 moven 模块：sample-api，再里面创建一个接口：IHelloService

```java
public interface IHelloService {
    String sayHello(String name);
}
```

然后`mvn install` **springboot-provide**这个顶层项目，这会在本地的 maven 仓库安装这父子两个库。
再添加一个 springboot 模块：**sample-provider**，来实现`IHelloService`接口

```java
@DubboService
public class HelloServiceImpl implements IHelloService {
    @Value("${dubbo.application.name}")
    private String serviceName;

    @Override
    public String sayHello(String s) {
        return String.format("[%s]: Hello,%s", serviceName, s);
    }
}
```

在 pom.xml 添加依赖

```xml
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.7</version>
        </dependency>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>sample-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
```

在 application.properties 添加 dubbo 的配置

```
dubbo.application.name=jesse-dubbo
dubbo.protocol.port=20880
dubbo.protocol.name=dubbo
dubbo.registry.address=N/A
```

启动类：

```java
@DubboComponentScan
@SpringBootApplication
public class SampleProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(SampleProviderApplication.class, args);
    }

}
```

provider 服务就启动了~

---

## consumer 服务

新建一个 springboot 项目：`springboot-consumer`，在 pom.xml 添加依赖，同上。同样也要在 application.properties 添加 dubbo 的配置，不过最少只要配置 name 就行

```
dubbo.application.name=springboot-consumer
```

启动类：

```java
@SpringBootApplication
public class SpringbootConsumerApplication {

    @DubboReference(url = "dubbo://127.0.0.1:20880")
    private IHelloService helloService;

    public static void main(String[] args) {
        SpringApplication.run(SpringbootConsumerApplication.class, args);
    }

    @Bean
    public ApplicationRunner runner(){
        return args -> System.out.println(helloService.sayHello("Mic"));
    }
}
```

完~

