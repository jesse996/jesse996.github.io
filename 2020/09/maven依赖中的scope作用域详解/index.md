# Maven依赖中的scope作用域详解


### compile

默认就是 compile，什么都不配置也就是意味着 compile。compile 表示被依赖项目需要参与当前项目的编译，当然后续的**[测试](http://lib.csdn.net/base/softwaretest)**，运行周期也参与其中，是一个比较强的依赖。打包的时候通常需要包含进去。

### test

scope 为 test 表示依赖项目仅仅参与测试相关的工作，包括测试代码的编译，执行。比较典型的如 junit

### runntime

runntime 表示被依赖项目无需参与项目的编译，不过后期的测试和运行周期需要其参与。与 compile 相比，跳过编译而已，说实话在终端的项目（非开源，企业内部系统）中，和 compile 区别不是很大。比较常见的如 JSR××× 的实现，对应的 API jar 是 compile 的，具体实现是 runtime 的，compile 只需要知道接口就足够了。**[Oracle](http://lib.csdn.net/base/oracle)** jdbc 驱动架包就是一个很好的例子，一般 scope 为 runntime。另外 runntime 的依赖通常和 optional 搭配使用，optional 为 true。我可以用 A 实现，也可以用 B 实现。

### provided

provided 意味着打包的时候可以不用包进去，别的设施(Web **[Container](http://lib.csdn.net/base/docker)**)会提供。事实上该依赖理论上可以参与编译，测试，运行等周期。相当于 compile，但是在打包阶段做了 exclude 的动作。

### system

从参与度来说，也 provided 相同，不过被依赖项不会从 maven 仓库抓，而是从本地文件系统拿，一定需要配合 systemPath 属性使用。

### **import**

它只使用在<dependencyManagement>中，表示从其它的 pom 中导入 dependency 的配置，例如 (B 项目导入 A 项目中的包配置)：

# scope 的依赖传递

A–>B–>C。当前项目为 A，A 依赖于 B，B 依赖于 C。知道 B 在 A 项目中的 scope，那么怎么知道 C 在 A 中的 scope 呢？答案是：  当 C 是 test 或者 provided 时，C 直接被丢弃，A 不依赖 C；  否则 A 依赖 C，C 的 scope 继承于 B 的 scope。

