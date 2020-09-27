# Mybatis笔记


接口和 xml 是通过将 namespace 的值设置为接口的全限定名来进行关联的。

接口中的方法名和 xml 中的 id 属性值相对应来进行关联。

可以在 resultMap 中配置 property 属性和 column 属性的映射，或者在 sql 中设置别名这两种方式实现将查询列映射到对象属性的目的

```jsx
select id,user_name userName from sys_user
```

在 mybatis 的配置文件中设置

```jsx
<setting name="mapUnderscoreToCamelCase" value="true" />
```

可以自动将下划线方式命名的数据库列映射到 java 对象的驼峰式命名。

-   数据库的 datetime 类型可以存储 DATE（时间部分默认为 0）和 TIMESTAMP 这两种类型，不能存储 TIME 类型
-   <insert useGeneratedKeys="true" keyProperty="id">...</insert>

useGeneratedKeys 设为 true 后，mybatis 会使用 jdbc 的 getGeneratedKeys 方法取出由数据库内部生成的主键。赋值给 keyProperty 设置的 id 属性。

-   还可以使用 selectKey 返回主键的值
-   接口中方法的参数只有一个，参数类型有两种，一种是基本类型，一种是 java bean
-   接口中方法参数有多个时，可以用 map（不推荐），或者用@Param 注解（推荐）

## 第三章

字段映射：

1.通过 sql 语句的别名

2.使用 mapUnderscoreToCamelCase 配置

和 xml 方式一样

3.使用@ResultMap 注解方式。

-   insert 操作如果不需要返回自增主键，那么 sql 语句要写 id，否则就不写

## 第四章

动态 sql

-   if
-   choose

```jsx
//至少一个when，0个或1个otherwise
<choose>
    <when test=".."></when>
    <when test=".."></when>
    <otherwise></otherwise>
</choose>
```

-   where：where 标签作用:如果该标签包含的元素中有返回值，就插入一个 where，如果 where 后面的字符串是以 and 和 or 开头的，就将 and 或 or 剔除。where 里面不满足的话，就没有 where
-   set：如果该标签包含的元素有返回值，就插入一个 set，如果 set 后面的字符串是以逗号结尾的，就将这个逗号删除
-   trim：where 和 set 标签的功能都可以通过 trim 标签来实现，且在底层就是通过 TrimSqlNode 实现的

where 标签对应的 trim 实现

```jsx
<trim prefix="WHERE" prefixOverrides="AND |OR ">
    ...
</trim>
```

and 和 or 后面的空格不能省略。

set 标签对应 trim 实现

```jsx
<trim prefix="SET" suffixOverrides=",">
    ...
</trim>
```

-   foreach：可以对数组，Map 或者实现了 Iterable 接口的对象进行遍历

```jsx
//实现in集合
select ... from ... where id in
<foreach collection="list" open="(" close=")" separator="," item="id" index="i">
    #{id}
</foreach>
```

collection 属性如何设置呢？

1.只有一个数组参数或者集合参数

-   数组时：默认是 array
-   集合时：默认是 collection，如果是 List，还可以用 list

可以用@Param 参数来指定参数的名字，这时 collection 就设置为指定的名字

2.有多个参数

要用@Param 给每个参数指定名字

3.参数是 Map 类型

将 collection 指定为 Map 中的 key 就行，如果要循环 Map，使用@Param 指定名字

4.参数是一个对象

指定为对象的属性名就行

## 第六章

### 一对一映射

-   使用自动映射处理一对一关系：通过别名
-   使用 resultMap 配置一对一关系
-   使用 resultMap 的 association 标签配置一对一映射

-   association 标签的嵌套查询

```jsx
<resultMap id="userRoleMapSelect" extends="userMap" type="SysUser">
    <association
        property="role"
        column="{id=role_id}"
        select="RoleMapper.selectRoleById"
    />
</resultMap>
```

这样就不是通过一个 sql 获取所有信息。

在 RoleMapper.xml 中

```jsx
<select id="selectRoleById" resultMap="roleMap">
    select * from sys_role where id = #{id}
</select>
```

在 association 中添加 fetchtype 属性，并在设置中设置 aggressiveLazyLoading 为 flase 即可延迟查询 sql

```jsx
<association
    property="role"
    column="{id=role_id}"
    select="tk.mybatis.simple.mapper.RoleMapper.selectRoleById"
    fetchType="lazy"
/>
```

```jsx
<setting name="aggressiveLazyLoading" value="false" />
```

LazyLoadTriggerMethods：

当调用配置中的方法时，加载全部的延迟加载数据。默认值为“equal,clone,hashCode,toString”

### 一对多映射

collection 集合的嵌套结果映射

和 association 类似，集合的嵌套结果映射就是指通过一次 sql 查询将所有的结果查询出来，然后通过配置的结果映射，将结果映射到不同的对象中去

### 鉴别器映射

discriminator 鉴别器，类似 switch 语句,用法：

```jsx
<resultMap id=".." type="..">
    <discriminator column="enable" javaType="int">
        <case value="1" resultMap="..." />
        <case value="0" resultMap="..." />
    </discriminator>
</resultMap>
```

鉴别器有个特殊的地方

```jsx
<resultMap id=".." type="..">
    <discriminator column="enable" javaType="int">
        <case value="1" resultMap="..." />
        <case value="0">
            <id property="id" column="id" />
            <result property="roleName" column="role_name" />
        </case>
    </discriminator>
</resultMap>
```

这种情况下，mybatis 只会对列举出来的配置进行映射，即 id 和 role_name

数据库 BLOB 类型对应的 java 类型通常都是写成 byte[]形式的，因为 byte[]数组不存在默认值的问题，所以不影响一般使用。但是在不指定 javaType 的情况下，MyBatis 默认使用 Byte 类型。由于 byte 类型时基本类型，所以设置 javaType 时要使用带下划线的方式，就是 \_byte[]。

\_byte 对应的时基本类型，byte 对应的时 Byte 类型，在使用 javaType 时一定要注意。

### 使用枚举或其他对象

MyBatis 为 java 和 JDBC 中的基本类型和常用类型提供了 TypeHandler 接口的实现，的处理枚举类型时默认使用 EnumTypeHandler 处理器，这个会将枚举类型转换为字符串类型的字面量。

MyBatis 还提供了另一个 EnumOrdinalTypeHandler 处理器，使用枚举的索引进行处理。

要使用这个处理器，需要在设置中添加

```jsx
<typeHandlers>
    <typeHandler
        javaType="枚举类"
        handler="org.apache.ibatis.type.EnumOrdinalTypeHander"
    />
</typeHandlers>
```

### 自定义类型处理器

自定义类实现 TypeHander 接口

## 对 java8 日期的支持

在 pom.xml 添加依赖：mybatis-typehanders-jsr310

## 第七章.缓存配置

MyBatis 有一级缓存和二级缓存，一级缓存就是本地缓存，默认开启，且不能配置。

### 一级缓存

-   MyBatis 的一级缓存存在于 SqlSession 的生命周期中。
-   在 select 增加 flushCache，会在查询数据前清空当前的一级缓存。这样做会增加查询，要尽量避免。

```jsx
<select id="selectById" flushCache="true">
    ...
</select>
```

任何的 INSERT，UPDATE，DELETE 都会清空一级缓存

### 二级缓存

-   二级缓存存在于 SqlSessionFactory 的生命周期中。
-   在设置里面，chcheEnable 设置为 true（默认就是 true，所以不用设置）

```jsx
<setting name="chcheEnable" value="true" />
```

### xml 方式：

只要在 mapper.xml 中添加<chche/>元素即可。

cache 可以配置的属性有：

-   eviction（收回策略）
    -   LRU
    -   FIFIO
    -   SOFT
    -   WEAK
-   flushInterval（刷新间隔）
-   size（引用数目）
-   readOnly（只读）

### mapper 接口方式

@CacheNamespace,也可以配置

当同时使用 xml 和注解方式时，要同时配置，接口用@CacheNamespaceRef(Mapper.class)

或者 xml 设置<cache-ref namespace="Mapper"/>

当配置为可读写时，MyBatis 使用 SerializedCache 序列化缓存来读写缓存类，每次得到的是一个新实列。被缓存的累要实现 Serializable 接口。

当配置为只读时，使用一个 Map 来保存，所以每次得到的是同一个对象

MyBatis 的二级缓存是和命名空间绑定的，通常每一个 Mapper 映射文件都拥有自己的二级缓存，不同 Mapper 的二级缓存互不影响。

pom.xml 中<dependency>中的<scope>设置为 provided 代表不会将这个 jar 包打包到项目中

