# 与Rust玩耍

![](https://blog.kanari.moe/wp-content/uploads/2020/02/unsafe_badge-300x300.jpg)
把先前信息低的两篇blog给拆了,这样好点.

<!--more-->

## 一点小插曲: 如何在windows上的rust使用gnu工具链

众所周知windows对编程的体验不如*nix和osx,rust甚至在Windows上的配置都麻烦一点.比如说,为了装个rust,可能需要另外准备vs2019.这两个东西的空间占用差了不止一点半点.

折腾了一天.如果能仔细阅读官方的rustup文档的话,说不定就不用这么头秃了.

主要的选项在于选好host和version.在rustup执行安装时,修改为

```bash
x86_64-pc-windows-gnu
stable-gnu
```

这样安装的版本才会使用gnu工具链.但是如此做的后果,我还不太清楚.

## 以C对撞Rust
自己主要还是C系语言用户,所以实际学习时,总是会相比较着进行.

```rust
let x:i32=10;
```
首先,rust也使用分号.每个变量的声明开头使用let,并且将变量类型跟随名称后面.

随着时间发展,现代语言几乎都抛弃了变量类型在左侧的声明方式.这大概是一个实践上的经验.左侧的类型声明会在复杂状况下造成极大的理解障碍.比如

```cpp
void (*signal(int, void (*fp)(int)))(int);
```

阅读此类东西,总会是让人头疼.你会发现,为了理解它在干嘛,这种声明方式使得阅读顺序必须是顺时针的螺旋式,显得很不自然和明确.一旦调整类型到右侧时,这个问题似乎就不存在了.

所以不但变量的声明类型位于名称左侧,包括函数参数类型和返回值也都是位于右侧.

剩下的..

```rust
//rust中的变量**默认为不可更改**.必须添加`mut`关键字才作为一般意义上的变量考虑.
let mut x=9;
x=6;

let y=9;
y=8; // this is not allowed.

if <expr> {
    //do something
}else if <expr>{
    //do what
}else{
    //yeah?
}

//特殊的死循环
loop{
}

while <expr>{
}

//由于Rust的零代价抽象,迭代器的使用没有性能折扣.
for i in 1..9{
}
//这个是右闭区间
for i in 1..=9{
}

//所有控制流都是表达式,能够返回值;所以我们也没有三元运算符了.
//对于循环,可以使用break.
let flag=if a>0{
    1 //最后一行不写引号,代表返回该值.没记错的话应该是从别的语言那借鉴的.
}else{
    -1
};

fn func(a:i32,b:i32) -> i32{
    a+b
}

```

## 包管理

rust发展太快了,各种文章各说各话.这里主要是一个`crate`(一个包)内的结构.

rust使用`mod`来明确的标注一个模块.关键字默认为私有,使用`pub`来标注一个对外公开的关键字.在2018标准,目录也会作为一个模块路径,文件名作为模块名,文件内关键字同样默认私有.在这种情况,使用`mod`关键字来显式的引入某个模块.对于第三方模块,2018不再需要显式标注引入`extern crate`.(但是仍然需要`use`)

在一个包里,允许有多个crate.如果硬要说,类似于vs studio里的解决方案和项目之间的关系.但是,rust还有一些不那么显然的限制

* 最多有一个lib crate
* 可以有多个binary crate


## 一个最小(?)的HTTP服务器

实现这个服务器的过程中,会遭遇到很多Rust内的问题.假设我们已经对比C++掌握其基本语法了.

```rust
use std::thread;
use std::sync::mpsc;
use std::sync::Arc;
use std::sync::Mutex;
pub struct ThreadPool{
    workers:Vec<Worker>,
    sender: mpsc::Sender<Message>,
}

type Job=Box<dyn FnOnce()+Send+'static>;//?

enum Message{
    NewJob(Job),
    Terminate,
}

impl ThreadPool{
    /// Create thread pool
    /// 
    /// The number of threads in pool
    /// 
    /// # Paincs
    /// 
    /// `new` will panic when size is not great than 0.
    pub fn new(size:usize)->ThreadPool{
        assert!(size>0);

        let (sender,receiver)=mpsc::channel();
        let receiver=Arc::new(Mutex::new(receiver));

        let mut workers=Vec::with_capacity(size);
        for id in 0..size{
            workers.push(Worker::new(id,receiver.clone()));
        }

        ThreadPool{
            workers,
            sender
        }
    }

    pub fn execute<F>(&self,f:F)
        where F:FnOnce()+Send+'static
    {
        let job=Box::new(f);
        self.sender.send(Message::NewJob(job)).unwrap();

    }
}

impl Drop for ThreadPool{
    fn drop(&mut self){
        for _ in &mut self.workers{
            self.sender.send(Message::Terminate).unwrap();
        }
        for worker in &mut self.workers{
            println!("shutting down worker {}",worker.id );

            if let Some(thread)=worker.thread.take(){
                thread.join().unwrap();
            }
        }
    }
}

struct Worker{
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl Worker{
    fn new(id:usize,receiver: Arc<Mutex<mpsc::Receiver<Message>>>)->Worker{
        let thread=thread::spawn(move|| {
            loop{
                let message=receiver.lock().unwrap().recv().unwrap();

                match message{
                    Message::NewJob(job)=>{
                        println!("Worker {} got job.",id );
                        job();
                    },
                    Message::Terminate=>{
                        println!("Worker {} is terminating.",id);
                        break;
                    }
                }
                
            }
        });

        Worker{
            id,
            thread: Some(thread),
        }
    }
}
```
```rust
use std::io::prelude::*;
use std::net::{TcpListener,TcpStream};
use std::fs;
use std::time::Duration;
use std::thread;
use automan::ThreadPool;

fn main() {
    let listener=TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(1);

    for stream in listener.incoming(){
        let stream=stream.unwrap();

        pool.execute(|| {
            handler(stream);
        });
    }
}

fn handler(mut stream: TcpStream){
    let mut buffer = [0; 512];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";
    let sleep = b"GET /sleep HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else if buffer.starts_with(sleep) {
        thread::sleep(Duration::from_secs(5));
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND\r\n\r\n", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();

    let response = format!("{}{}", status_line, contents);

    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

## FnOnce

在C++里,我们能够用Lambda表达式创建闭包,当需要保存一个闭包时,标准库提供了叫`function`的东西.在Rust里,我们也有闭包,而描述一个闭包类型的玩意,就是`Fn***`.

```rust
type Job=Box<dyn FnOnce()+Send+&#039;static>;
```

得益于rust的内存模型,诞生出这个奇葩玩意.反正还是和变量的所有权有关系.`FnOnce`的闭包会获取其捕获变量的所有权.另外`FnMut`的闭包获得其捕获变量的可变借用,`Fn`获得不可变借用.勉强可以对比C++里的`[],[&]`.

至于到底什么闭包会实现哪一个,是由编译器根据实际情况判断的.想要强制将变量挪进闭包时,可以前缀`move`,就像`Worker`的`run`里的代码一样.因为当`run`执行完,receiver将会被丢弃,从而影响到闭包里的引用.

其他的几个`+`号和`Send`,`'static`什么的.`+`就是`+`,要求该类型必须同时实现这三种`trait`.Send是个`trait`(Types that can be transferred across thread boundaries).`'static`等时间周期实际上也是`trait`.

## Box和dyn

```rust
type Job=Box<dyn FnOnce()+Send+&#039;static>;
```

......没完..前面那个`dyn`玩意也是挺奇怪,需要和`impl`关键字对比理解.

Rust的一大特点就是零代价抽象.例如对于泛型,一种实现0抽象的方法就是在编译阶段单例化.C++也是如此处理.

不过有一种情况目测C++无法实现零抽象:在C++中,如果存在父类`A`有虚方法`f`,而子类`B`重写了`f`,又有一个函数`g(A a)`会调用`f`.当把`B`作为参数传给`g`时,C++通过一种叫`虚函数表`的方式实现调用`B`写的`f`.这种行为是发生在运行时的.

对于Rust来讲,虽然没有类与继承,这种操作实际等价于`trait`.上面的情况对等于存在`trait A`,它定义了一个`f`,`B`实现了`A`,`g`同上.Rust**能够在编译阶段**就确定传给`g`的是谁,并且优化掉它.不过这种优化也是有限度的.如果把一堆实现`A`的结构的指针存到数组或者其他内存结构里,那Rust也是无能为力的,只能在运行时动态分发.

再回来看`impl`和`dyn`.它们分别对应了能够优化(静态分发)和动态分发的情景.

另外,采用动态分发的结构也不能在编译时确定大小,所以必须外包`Box`.这一点还是有点特殊的,C++似乎并不会抱怨这个问题.

## Arc和Mutex

```rust
fn new(id:usize,receiver: Arc<Mutex<mpsc::Receiver<Message>>>)->Worker;
```

说真的,看到`Worker`的这个函数签名,当时就像吐.Rust这玩意,动不动就会嵌套上好几层的`<>`.

首先关于`Arc`...先`Rc`.一般情况下,一个变量的所有权是可以在编译阶段就确定下来的.但是总会有不如意的时候,当有多个结构同时享有某个变量的所有权(例如一棵树),就不太好确定其实际的销毁时机,而且Rust也不会给你机会共享所有权.`Rc`是一个引用计数指针,变量所有权由它保管,而它能够提供多份只读借用.

然而`Rc`线程不安全,`Arc`线程安全.不过`Arc`性能弱于`Rc`.

其次是`Mutex`...就是个锁...

至于为啥用锁,先看`mpsc`,Multi-producer, single-consumer FIFO queue communication primitives.

一头雾水的缩写...简而言之,这个东西的设定类似于go里`channel`,同时是多生产者,**单消费者**模式.本身使用没什么绕的地方.绕的地方在于多线程共享和单消费者模式的冲突,所以引入了锁,保证同时只有一个线程能够消费消息.

顺便吐槽一句这代码写的...

```rust
let message=receiver.lock().unwrap().recv().unwrap();
```

其实涉及到`Rc`时,还有另外一个外部可变,内部可变的麻烦事.这里没有提到.

## 还有

`unwrap`,`unwrap`,`unwrap`,`unwrap`,`unwrap`,`unwrap`,`unwrap`,`unwrap`,`unwrap`,`unwrap`,`unwrap`,`unwrap`...

当然它们本来会被替代成更复杂的错误处理.

## 小结

用Rust写的程序,和自己脑袋斗智斗勇几回合,再和编译器斗智斗勇(单方面挨揍)几百回合,最后是真的难以出现bug,毕竟一切都被管的死死的.我还看到Rust里一个很有意思的玩意叫错误驱动,说是先把代码大概模样瞎糊上,然后让编译器check一遍,再一步步改掉编译器的报错~~顺便写代码~~,最后就能在编译器的吐槽下完美实现程序,再加点测试就齐全了...

虽然整个语法看起来复杂啰嗦,不过无法否认这些语句对于工程来讲实际上都是必要的.所以这玩意的气质决定它不太适合作为打比赛时用的语言.翻了翻CF好几场比赛,零星有几个Rust的代码也都没有涉及任何数据结构,本来还想瞻仰一下他们会怎么处理生命周期一类的问题.

如果有机会,拿Rust写课设应该可以玩一玩(大概会被强行用C系语言).至于原本想拿Rust写个本地评测器,还是算了,有空再说吧.
