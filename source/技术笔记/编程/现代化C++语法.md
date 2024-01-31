# 现代化C++语法

可变模版参数（variadic templates）
typename... Args是C++11引入的

万能引用
F&& f

尾返回类型推导

decltype关键字
编译时类型推导

std::future<T>
std::share_future<T>

std::async

std::make_shared

std::forward()
被称作完美转发。简单来说，std::forward()将会完整保留参数的引用类型进行转发

std::function
[走进C++11（二十三） 函数对象包装器之std::function ](https://mp.weixin.qq.com/s?__biz=MzIwNDgyMTEyMA==&mid=2247483947&idx=1&sn=7e483261bd44f6268ab7602ce4562d41&chksm=973b05c4a04c8cd2ed367b4c8970a3eb5973f3cc3f45b2c01df534468002a72cefe565431fc5)
一种通用、多态的函数封装。std::function的实例可以对任何可以调用的目标实体进行存储、复制、和调用操作，这些目标实体包括普通函数、Lambda表达式、函数指针、以及其它函数对象等。std::function对象是对C++中现有的可调用实体的一种类型安全的包裹（我们知道像函数指针这类可调用实体，是类型不安全的）。
通常std::function是一个函数对象类，它包装其它任意的函数对象，被包装的函数对象具有类型为T1, …,TN的N个参数，并且返回一个可转换到R类型的值
通过std::function对C++中各种可调用实体（普通函数、Lambda表达式、函数指针、以及其它函数对象等）的封装，形成一个新的可调用的std::function对象；


std::bind
两个作用：将可调用对象和其参数绑定成一个仿函数； 只绑定部分参数，减少可调用对象传入的参数

std::packaged_task
[走进C++11（二十九） 将工作打包成任务，丢给执行者 -- std::packaged_task](https://mp.weixin.qq.com/s?__biz=MzIwNDgyMTEyMA==&mid=2247484055&idx=1&sn=3a9fca9f9740ff22e94fb126f2c709a8&chksm=973b0578a04c8c6e612e6c5964270cb346907c599ab1a6d41be844bcaf6849ae249aa15b4f05)

Lambda函数
[走进C++11（二十二） 之lambda（匿名函数）](https://mp.weixin.qq.com/s?__biz=MzIwNDgyMTEyMA==&mid=2247483916&idx=1&sn=79e3c4e23735a92b0c4581aaea3daaf9&chksm=973b05e3a04c8cf5015ac893411f4d3e7996e48bf7f10607c72b7d5982b3e5d14fd9265e115a&cur_album_id=1469697165768835082)


std::condition_variable

.emplace_back
类似.push_back()，减少对象拷贝和构造次数
