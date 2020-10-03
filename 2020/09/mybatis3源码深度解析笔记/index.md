# MyBatis3源码深度解析笔记


## Connection

一个 connection 对象表示通过 JDBC 驱动与数据库建立的连接。  
我们可以通过两种方式获取 connection 对象：

1. 通过 JDBC API 中提供的 DriverManager 类获取
2. 通过 DataSource 获取

推荐第 2 种方式。

### DriverManager

所有的 JDBC 驱动都必须实现 Driver 接口，而且实现类必须包含一个静态初始化代码，在静态初始化代码中向 DriverManager 注册自己的一个实例。  
这就是为什么我们使用 JDBC 操作数据库时一般会先加载驱动：

```java
Class.forName("com.mysql.cj.jdbc.Driver")
```

JDBC 4.0 以上的版本利用 SPI（Service Provider Interface）机制使得我们不需要显式的如上去加载驱动类。

SPI 是一种动态替换发现的机制，比如有一个接口，都运行时动态地给它添加实现，只需要添加一个实现，SPI 机制在程序运行时就会发现该实现类。

当服务的提供者提供了一种接口的实现后，需要在 classpath 下的`META-INF/services`目录中创建一个以服务接口命名的文件，这个文件就是实现类。

JDK 中查找服务实现类的工具类是`java.util.ServiceLoader`。在 DriverManager 类中定义了静态初始化代码，会加载所有的驱动类。

### DataSource

使用 DataSource 对象可以提高应用程序的可移植性。我们可以使用 JNDI（Java Naming and Directory Interface）把一个逻辑名称和数据源对象建立映射关系。

JNDI 为应用程序提供了一种通过网络访问远程服务的方式。

`executeQuery()`：执行一个查询语句，并返回一个`ResultSet`。

`executeUpdate()`：执行一个`update、insert`或者`delete`语句，返回更新数量。

`execute()`：可以执行所有操作，当是`select`操作时，返回`true`，可以通过`getResult()`查询结果，其他返回 `false`，通过 `getUpdateCount()`返回影响的行数。

另外，`execute()`可能返回多个结果，可以通过 `getMoreResults()`获取下一个结果。

`PreparedStatement` 对象设置的参数在执行后不能被重置，必须显式的调用 `clearParameters()`方法清除先前设置的值，再为参数重新设置值即可。

`CallableStatement`接口继承自`PreparedStatement`接口，在他的基础上增加了调用存储过程并检索调用结果的功能。通过`Connection`对象的`prepareCall()`方法得到对象。

在`execute()`,`executeUPdate()`中可以接受额外的参数，设置为`Statement.RETURN_GENERATED_KEYS`，然后在调用`Statement`对象的`getGeneratedKeys()`方法获得`ResultSet`对象，就可以获得自增长的键值了。

### ResultSet

ResultSet 对象的类型主要体现在两个方面

1. 游标可操作的方式
2. ResultSet 对象的修改对数据库的影响

后者称为 ResultSet 对象的敏感性。

ResultSet 有 3 种不同的类型

1. TYPE_FORWARD_ONLY  
   这种类型的 ResultSet 不可滚动，游标只能向前移动，即只能使用 next()方法，不能使用 previous()方法
2. TYPE_SCROLL_INSENSITIVE  
   可滚动的，可以相对移动和绝对移动

    当 ResultSet 没有关闭时，ResultSet 的修改对数据库不敏感，也就是说对 ResultSet 的修改不会影响数据库中的记录

3. TYPE_SCROLL_SENSITIVE
   可滚动的，可以相对移动和绝对移动  
   当 ResultSet 没有关闭时，ResultSet 的修改会影响数据库

默认为第一种类型
`rs.getType()`可获得

### ResultSet 并行性

目前 JDBC 中支持两个级别

-   CONCUR_READ_ONLY（默认次类型）
-   CONCUR_UPDATABLE

`rs.getConcurrency()`获得

### ResultSet 可保持性

调用 Connection 对象的 commit()方法能够关闭当前事务中创建的 ResultSet 对象。ResultSet 的 holdability 属性使得 ResultSet 对象是否关闭

-   HOLD_CURSORS_OVER_COMMIT
-   CLOSE_CURSORS_AT_COMMIT

`rs.getHoldability()`获得

默认取决于驱动实现

### ResultSet 属性设置

ResultSet 的`类型、并行性、可保持性`等属性可以在调用 Connection 对象的 `createStatement()、prepareStatement()、prepareCall()`方法时设置

### ResultSet 游标移动

ResultSet 对象中维护了一个游标，指向当前数据行。第一次指向数据的第一行。有以下方法操作游标：

-   next()
-   previous()
-   first()
-   last()
-   beforeFirst()
-   afterFirst()
-   relative(int rows)
-   absolute(int row) :row 从 1 开始

### DatabaseMetaData

获取：

```java
conn.getMetadata();
```

### 事务边界和自动提交

何时开启一个事务是由 JDBC 驱动或数据库隐式决定的。

Connection 对象的`autoCommit`属性决定什么时候结束一个事务。启动自动提交后，会在每个 sql 语句执行完毕后自动提交事务。默认是自动提交开启的。通过`setAutoCommit()`方法设置。禁用后要手动调用`Connection`接口提供的`commit()`方法或`rollback()`方法

### 事务隔离级别

数据并发访问可能会出现以下几种问题：

-   脏读：发生在事务中允许读取未提交的数据。如，A 事务修改了一条数据，但是未提交，此时 A 事务对数据的修改对其他事务是可见的，B 事务中能够读取 A 事务中未提交的修改。一旦 A 事务回滚，B 事务获取的就是不正确的数据。

-   不可重复读：发生在如下场景
    1. A 事务中读取一行数据
    2. B 事务中修改了该行数据
    3. A 事务中再次读取该行数据将得到不同结果
-   幻读：发生在如下场景
    1. A 事务中通过 WHERE 条件读取若干行
    2. B 事务中插入了符合条件的若干条数据
    3. A 事务中通过相同的条件再次读取数据时将会读取到 B 事务中插入的数据

事务隔离级别：

-   TRANSACTION_NONE：  
    表示不支持事务
-   TRANSACTION_READ_UNCOMMITTED：  
    允许读取未提交更改的数据
-   TRANSACTION_READ_COMMITTED：  
    可以防止脏读
-   TRANSACTION_REPEATABLE_READ：  
    可以解决脏读和不可重复读
-   TRANSACTION_SERIALIZABLE：  
    可以解决脏读、不可重复读、幻读，但是并发效率低

默认事务级别由 JDBC 驱动指定。可以通过 `setTransactionIsolation()`方法设置。

### 事务中的保存点

保存点通过在事务中标记一个中间的点来对事务进行更细粒度的控制，一旦设置的保存点，事务就可以回滚到保存点，而不影响保存点之前的操作。

`setSavepoint()`设置保存点，该方法返回一个`Savepoint`对象，该对象可作为`rollback()`方法的参数，用于回滚到保存点。

可以通过 releaseSavepoint()方法手动释放保存点，此保存点后的所有保存点也会被释放。

所有保存点在事务提交或回滚之后会自动释放。

## MyBatis 核心组件

-   Configuration：用于描述 MyBatis 的主配置信息
-   MappedStatement：用于描述 Mapper 中的 SQL 配置信息
-   SqlSession：是 MyBatis 提供的面向用户的 API
-   Executor：是 SQL 执行器
-   StatementHandler：封装了对 JDBC Statement 对象的操作
-   ParameterHandler：为参数占位符设置值。
-   ResultSetHandler：将查询结果转换成 Java 对象
-   TypeHandler：处理 Java 类型与 JDBC 类型之间的映射。

## SqlSession 执行 Mapper 的过程

MyBatis 中 Mapper 的配置分为两部分，分别为 Mapper 接口和 Mapper SQL 配置。MyBatis 通过动态代理的方式创建 Mapper 接口垫代理对象，MapperProxy 类中定义了 Mapper 方法执行时的拦截逻辑，通过 MapperProxyFactory 创建代理实例，MyBatis 启动时，会将 MapperProxyFactory 注册到 Configuration 对象中。另外，MyBatis 通过 MappedStatement 类描述 Mapper SQL 配置信息，框架启动时，会解析 MapperSQL 配置，将所有的 MappedStatement 对象注册到 Configuration 对象中。

通过 Mapper 代理对象调用 Mapper 接口中定义的方法时，会执行 MapperProxy 类中的拦截逻辑，将 Mapper 方法的调用转换为调用 SqlSession 提供的 API 方法。在 SqlSession 的 API 方法中通过 Mapper 的 Id 找到对应的 MappedStatement 对象，获取对应的 SQL 信息，通过 StatementHandler 操作 JDBC 的 Statement 对象完成与数据库的交互，然后通过 ResultSetHandler 处理结果，将结果返回给调用者。

