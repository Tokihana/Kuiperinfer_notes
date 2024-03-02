



# Google Test

Google Test是用于编写C++测试的框架，支持多种类型的测试，而不是只有单元测试（unit test）。

编写测试的要求：

- 测试应该是独立且结果可重复的
- 测试应该是有组织的（organized），并且能够反映被测试代码的结构
- 测试应该是轻量（portable）且可重用的
- 如果测试未通过，应尽可能返回更多信息
- 测试框架应该能够帮助测试者更多关注被测试的代码，而不是过多关注如何编写测试
- 测试应该能够快速运行

Test定义：使用特定的输入，运行特定的程序，并验证运行的结果。



## 基本概念

使用GoogleTest的第一步是编写`assertion`，`assertion`用来验证某个条件是否为真，其结果可以是success，nonfatal failure或者是fatal failure。

如果测试报了fatal failure，则会中断。

Test使用`assertion`来检验代码是否如期运行，如果测试出现问题，则测试失败。

Test suite（测试组）包含一个或多个测试，组织test suite应档能够反映被测试代码的结构。如果测试组中有测试需要共享对象或者写成，应当放进`test fixture`类中。

一个测试程序（test program）可包含多个测试组。

整体来说是个层级结构，从`assertion`开始写，组成test，test组成suite，suite组成测试程序。



# 编写测试

## 头文件

需要声明以下头文件

```c++
#include <gtest/gtest.h>
#include <glog/logging.h>
```



## Assertion

Assertion是一种宏。包括下面两种：

- `ASSERT_*`：如果fail，则产生fatal failures，并中断当前函数。

- `EXPECT_*`：如果fail，产生nonfatal failures，不中断。

`EXPECT_*`能够在一个test中报多个failure，通常来说更适合；当然，如果某个错误发生后，程序没必要再继续运行下去，则用`ASSERT_*`更好。

可以通过`<<`在`assertion`后面添加额外的信息。

例子：

```c++
ASSERT_EQ(x.size(), y.size()) << "Vectors x and y are of unequal length";
for(int i = 0; i < x.size(); ++i){
    EXPECT_EQ(x[i], y[i]) << "Vectors x and y differ at index " << i;
}
```

所有可被流式输出的内容都可以被放在assertion信息里。

除了`EQ`之外，还有很多种类型的assertion，可参考https://google.github.io/googletest/reference/assertions.html。

可能比较常用的：THAT，TRUE，FALSE，EQ，NE，LT，LE，GT，GE

在test1.cpp和axby.cpp中，用到了EQ和LE。



## TEST

创建test的方法：

1. 使用`TEST()`宏来定义和命名test函数；该函数是通常的C++函数，且不返回值
2. 在函数中写assertion
3. test的结果取决于assertion，如果一个test内任意assertion报fail，则test报fail

`TEST()`的参数按general到specific排列，第一个参数是suite名，第二个参数是test名，名称中不能有下划线`_`。

例子：

```c++
int f(int n); // 函数声明
TEST(fTest, ZeroInput){
    EXPECT_EQ(f(0), 1);
} // TEST 1
TEST(fTest, PositiveInput){
    EXPECT_EQ(f(1), 1);
    EXPECT_EQ(f(8), 512);
} // TEST 2
```

上例中，fTest是suite名，ZeroInput和PositiveInput为Test名。

逻辑上相关的test应当放在同一个suite中，即`TEST()`的第一个参数应该相同。



## Test Fixture

如果两个或更多的test会使用同一个数据，则可以用test fixture来重用对象。

1. 从`testing::Test`派生一个类，该类以`protected:`起始
2. 在这个派生类中，声明要使用的对象
3. 编写默认的constructor或者SetUp()函数来配置每个test中的对象。
4. 编写destructor或者TearDown()来释放在SetUp()中alloc的内存。
5. 定义test中要用的subroutine。

用`TEST_F()`来代替`TEST()`，并将派生类的类名称作为第一个传参名称。派生类要在`TEST_F()`之前定义。

例子。以一个队列类`Queue`的测试为例，首先给出该类的接口：

```c++
template <typename E>
class Queue{
    public:
    	Queue();
    	void Enqueue(const E& element);
    	E* Dequeue();
    	size_t size() const;
    	...
};
```

定义fixture class，习惯上命名为`*Test`

```c++
class QueueTest : public testing::Test{
    protected:
    	void SetUp() override{
            // q0_ remains empty
            q1_.Enqueue(1);
            q2_.Enqueue(2);
            q2_.Enqueue(3);
        }
    	Queue<int> q0_;
    	Queue<int> q1_;
    	Queue<int> q2_;
};
```

编写test

```c++
TEST_F(QueueTest, IsEmptyInitially){
    EXPECT_EQ(q0_.size(), 0);
}
TEST_F(QueueTest, DqueueWorks){
    int * n = q0_.Dequeue();
    EXPECT_EQ(n, nullptr);
    
    n = q1_.Dequeue();
    ASSERT_NE(n, nullptr);
    EXPECT_EQ(*n, 1);
    EXPECT_EQ(q1_.size(), 0);
    delete ;
    
    n = q2_.Dequeue();
    ASSERT_NE(n, nullptr);
    EXPECT_EQ(*n, 2);
    EXPECT_EQ(q1_.size(), 1);
    delete ;
}
```



## Invoking the Tests

在定义好test后，可以通过调用RUN_ALL_TESTS()来运行所有的test，如果所有的test都通过，则返回0，否则返回1。

该函数会测试所有可以关联的test，包括不同test suite甚至不同文件中的test



## 编写main()函数

> 通常情况下是不用写main()的，用gtest_main()就行。

如果需要在test之前做一些其他框架内没法处理的操作，则可以自己写个main()，并将返回值设为`RUN_ALL_TESTS()`.

例子：

```c++
int main(int argc, char **argv){
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

`testing::InitGoogleTest()`用于传递控制台参数，使得用户可以控制测试程序的行为，这部分可参考：https://google.github.io/googletest/advanced.html。



# 参考

- 用户手册：https://google.github.io/googletest/
- 入门：https://google.github.io/googletest/primer.html
- GitHub：https://github.com/google/googletest/tree/main