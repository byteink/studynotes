- 如果mock的对象永远不删除，最后的验证将不会发生。

- 需要在mock对象的函数被调用前设置expectations。

- Expectations语法
```cpp
EXPECT_CALL(mock_object, method(matchers))
.Times(cardinality)
.WillOnce(action)
.WillRepeatedly(action);
```
第一个参数是mock的对象，第二个参数是方法和它的参数。
- Matcher
```cpp
EXPECT_CALL(turtle, Forward(100));
using ::testing::_;
EXPECT_CALL(turtle, Forward(_));     // 任意参数都是ok
using ::testing::Ge;...
EXPECT_CALL(turtle, Forward(Ge(100))); // 大于100
```
- Cardinalities
`Times(0)`：应该一次也不被调用
Times可以被忽略
   - 如果EXPECT_ALL中没有 WillOnce() 也没有WillRepeatly()，则默认是 Times(1)
   - 如果有 n 次 WillOnce()，但是没有 WillRepeatedly，当n >=1时，则意味着Times(n)
   - 如果有 n 次 WillOnce()和一次 WillRepeatedly，当 n >= 0时，意味着Times(AtLeast(n))

- Actions
如果函数的返回值类型是内置类型或者指针，默认行为是：void则没有返回；bool返回false，其他返回0.
C++11及以后如果返回类型有默认构造函数，则返回值使用默认构造函数

   可以使用一系列的`WillOnce`和一个可选的`WillRepeatedly`来指定返回值。
```cpp
using ::testing::Return;...
EXPECT_CALL(turtle, GetX())
.WillOnce(Return(100))
.WillOnce(Return(200))
.WillOnce(Return(300));
```
如果指定了Times(n)，没有WillRepeatedly，WillOnce的个数小于n，则剩余的执行默认行为。

  EXPECT_CALL语句只会计算一次action 从句：
```cpp
int n = 100;
EXPECT_CALL(turtle, GetX())
.Times(4)
.WillRepeatedly(Return(n++));     // return 100 always```

- Using Multiple Expectations
当一个mock方法被调用时，将会根据定义逆序搜索，如果找到参数匹配的就停止。

- Ordered vs Unordered Calls
默认匹配是无序的，强制有序
```cpp
using ::testing::InSequence;...
TEST(FooTest, DrawsLineSegment) {
...
{
InSequence dummy;

EXPECT_CALL(turtle, PenDown());
EXPECT_CALL(turtle, Forward(100));
EXPECT_CALL(turtle, PenUp());
}
Foo();
}
```
