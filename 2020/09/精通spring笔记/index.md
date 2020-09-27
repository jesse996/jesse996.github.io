# 精通Spring笔记


## 反应式编程

### 反应式特点

-   即时响应式：系统会即时响应用户请求
-   回弹性：将故障限制在本地范围内
-   弹性：可以灵活添加资源和释放资源
-   消息驱动：消息（或事件）驱动

传统方式：轮询
反应式方式：订阅事件和事件链

### 反应式流

Maven 依赖如下：

```xml
<groupId>org.reactivestreams</groupId>
<artifactId>reactive-stream</artifactId>

<artifactId>reactive-stream-tck</artifactId>
```

一些重要接口：

```java
public interface Subscriber<T>{
    public void onSubScribe(Subscription s);
    public void onNext(T t);
    public void onError(THrowable t);
    public void onComplete();
}

public interface Publisher<T>{
    public void  subscribe(Subscriber<? super T> s);
}

public interface Subscription{
    public void request(long n);
    public void cancle();

}
```

### Reactor

由 Spring Pivotal 团队开发，建立在反应式流基础上。
依赖如下：

```xml
<artifactId>reactor-core</artifactId>

<artifactId>reactor-test</artifactId>
```

-   Flux:表示 0~n 个元素的反应式流
-   Mono：表示 0 或 1 个元素的反应式流

### Spring Web Reactive

Spring Web Reactive 基于于 Spring MVC 相同的基本编程模型。  
 、| Spring MVC | Spring Web Reactive|  
 ----|----|---
用途|传统 Web 应用程序|反应式 Web 应用程序|
编程模型|具有@RequestMapping 的@Controller|与 Spring MVC 相同
基本 API|Servlet API|反应式 HTTP
运行位置|Servlet 容器|Severlet 容器（>3.1）、Netty、Undertow

### 反应式数据库

-   ReactiveMongo 旨在实现响应式能力并避免阻塞性操作。
-   r2dbc

### 异常处理

-   受检异常：服务方法引发此异常时，所有消费方方法应处理或引发异常
-   未受检异常：服务方法引发此异常时，不需要消费方方法应处理或引发异常

RuntimeException 及其所有子类均为未受检异常，其他异常为受检异常

#### Spring 的异常处理

它将大多数异常作为未受检异常

### 用 Java config 代替 xml 简化配置

#### 在 CompontScan 中使用 basePackageClasses 属性

```java
@ComponentScan(basePackageClasses = ApplicationController.calss)
public calss SomeApllication{}
```

每个指定类的包都会接受扫描。这会确保即使包被重命名或转移，都可以按预期执行扫描

### 强制性依赖项首选构造函数注入而不是 setter 注入

### 管理依赖项版本

spring boot 最简单的方法是默认的将 spring-boot-start-parent 作为父级 POM。

有时必须将自定义的企业 POM 作为父级 POM，那么用一下方法管理版本：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>@{spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

如果未使用 spring boot，可以使用 spring BOM 管理所有基本 Spring 依赖项

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-framework-bom</artifactId>
            <version>@{org.springframework-version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

