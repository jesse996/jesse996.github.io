# Rust笔记


不是每个 trait 都可以作为 tarit 对象被使用，这和类型大小是否确认有关。每个 trait 都包含一个隐式的类型参数 Self，代表实现该 tarit 的实际类型。Self 默认有一个隐式的 tarit 限定`?Sized`，形如 `Self:?Sized`。?Sized trait 包含了所有的动态大小类型金额所有的可确定大小的类型。Rust 中大多数类型都是可确定的，就是<T:Sized>。

必须满足下面 2 条规则才可以当作 trait 对象使用：

1. trait 的 Self 不能被限定为 Sized。
2. trait 中所有的方法都必须是对象安全的。

而对象安全的方法必须满足一下三点之一：

1. 方法受 Self：Sized 约束
2. 方法签名同时满足以下三点：
   （1）必须不包含任何泛型参数。
   （2）第一个参数必须为 Self 类型或可以解引用为 Self 的类型
   （3）Self 不能出现在第一个参数以外的地方，包括返回值
3. trait 中不能包含关联常量。

在实践中，只涉及到两条规则。如果一个 trait 中所有的方法有如下属性时，则该 trait 是对象安全的：

-   返回值类型不为 Self
-   方法没有任何泛型类型参数

-   每个文件定义一个模块。lib.rs 定义了一个和自己 crate 同名的模块；一个 mod.rs 定义了一个它所在文件夹名字的模块；其他的每个文件定义了一个同文件名的模块。
-   二进制 crate 的 root 必须是 main.rs，库 crate 的 root 必须是 lib.rs
-   单元测试通常和所测试的代码在同一个文件
-   集成测试，样例，benchmarks 都必须像其他用户一样导入 crate，只能用公开的 API。

## 闭包

1. 如果闭包中没有捕获任何环境变量，则默认自动实现 Fn
2. 如果闭包中捕获了复制语义类型的环境变量，则
    - 如果不需要修改环境变量，无论是否使用了 move，均会自动实现 Fn
    - 如果需要修改环境变量，则自动实现 FnMut
3. 如果闭包中捕获了移动语义类型的环境变量，则
    - 如果不需要修改，且没有使用 move，则自动实现 FnOnce
    - 如果不需要修改，且使用了 move，则自动实现 Fn
    - 如果需要修改，则自动实现 FnMut
4. 使用 move，如果捕获的变量是复制语义类型的，则闭包本身会自动实现 Copy/Clone,否则不会。

每个闭包表达式都是一个独立的类型，这会有一些不便，如不能把不同的闭包保存到一个数组中，但这可以通过把闭包当做 trait 对象来解决。把闭包放到 Box<T>中就可以构建一个闭包的 trait 对象，然后就可以当做类型来使用，

三者关系：
FnOnce-> FnMut -> Fn，即要实现 Fn，必须先实现前面 2 个。

rust 类型分为复制语义和移动语义，复制语义是指分配在栈上，所以复制的时候很简单，直接按位复制，不会出现内存不安全的情况。移动语义指分配在堆上，为了保证内存安全，才有了所有权系统，即一块内存只有有一个变量指向它。

对于复合类型来说，是复制还是移动，取决于其成员的类型，分为 2 种：

-   结构体，枚举体：
    当成员全都是复制语义的时候，复合类型不会自动实现 Copy，要手动实现 Derive(Copy,Clone)，此时复合类型才是复制语义的。如果复合类型中的成员有移动语义的，则无法实现 Copy。
-   元组，数组，Option：类型会自动实现 Copy，如果元素均为复制语义，则元组就是复制，不需要手动再 Derive(Coyp,Clone)，否则元组就是移动语义的。

> 共享可变状态是万恶之源

每个 let 都会创建一个默认的词法作用域，这个作用域就是它的生命周期（lifetime），就是在这个词法作用域中存活，出了就死亡。

**解引用会获得所有权。**

显式生命周期参数是为了解决跨函数借用，编译器无法检查的问题。它只用于编译器的借用检查，来防止垂悬指针。

`'b: 'a`的意思是 b 的存活时间长于 a
结构体实例的生命周期应短于或等于任意一个成员的生命周期。

##### 生命周期省略规则：

-   每个输入上对应一个不同的生命周期参数
-   如果只有一个输入，则输出生命周期等于这个输入的生命周期
-   如果有 self（&self,&mut self），则输出生命周期等于 self 的生命周期

##### trait 对象的生命周期默认以下规则：

-   trait 对象的生命周期默认是'static
-   如果实现 trait 的类型包括&'a x 或&'a mut x，则默认生命周期就是'a
-   如果实现 trait 的类型包含多个类似 T：'a 的从句，则生命周期需要明确指定

##### Cell 和 RefCell 的区别：

-   Cell<T>通过 set/get 来直接操作包裹的值，RefCell<T>通过 borrow/borrow_mut。
-   Cell<T>一般适合复制语义类型，即实现了 Copy，RefCell<T>适合移动语义类型
-   Cell<T>无运行时开销，不会再运行时 panic，RefCell 则有运行时开销，会 panic

#### 写时复制 Cow<T>

Cow<T>是一个枚举体智能指针，包括 2 个可选项：

-   Borrowed：用于包裹引用
-   Woned：用于包裹所有者

cow 提供的功能是：以不可变的方式访问内容，在需要可变借用或所有权的时候再克隆一份数据。cow 要点：

-   Cow<T>实现了 Deref，所以可以直接调用 T 的不可变方法
-   在需要修改 T 时，可以使用 to_mut 方法获取可变借用。该方法会克隆，且仅克隆一次。如果 T 本身有所有权，则调用 to_mut 不会发生克隆
-   在需要修改 T 时，也可以用 into_owned 方法来获取一个拥有所有权的对象。如果 T 是借用类型，则会发生克隆，并创建新的有所有权的对象。如果 T 是所有权对象，则会将所有权转移到新的克隆对象。

# Future Trait

`Future`是 rust 异步的核心，代表一个将来会产生值的一个东东。调用`poll`方法可以让 future 朝着完成进行，如果完成了就返回 Pool::Ready(result)，否则返回`Poll::Pending`并加到事件循环队列中等待再次被`wake`方法调用。

# Waker

Waker 有一个 wake()方法，用来告诉执行器需要执行相关的任务，就会调用相关的 poll 方法。

# Executor

rust 的 future 都是懒执行的，就是说除非用主动推动完成才会完成，否则不会做任何事。一个推动 future 完成的方法是在 async 方法中用`.await`，但最外层的 future 如何完成呢？这就需要一个`Future executor`。
Executor 通过调用`poll`方法执行一系列最外层的 future。典型的，一旦一个 future 开始了，Executor 就会对之调用 poll 方法。当 future 暗示他们准备好了被调用 wake()更进一步，他们就会被放到队列中等待再次被 poll，不断重复直至完成。
先看看 Executor 的定义:

```
struct Executor {
    ready_queue: Receiver<Arc<Task>>
}
```

## 简单的 HashMap 初始化

```rust
vec![('d',5),('c',1)].into_iter().collect::<HashMap<char,i32>>();
```

```rust
fn main() {
    let source = "你好你是谁";
    // first find the byte index, then find the corresponding character index
    // (code point index)
    // this goes through the source twice though :(
    let char_index = source.find("你是").map(|found_byte_index| {
        source
            .char_indices()
            .position(|(byte_index, _)| byte_index == found_byte_index)
            .unwrap()
    });
    dbg!(char_index);
}
```

