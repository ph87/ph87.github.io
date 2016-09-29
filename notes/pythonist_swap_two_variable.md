在知乎上看到有人在讨论一道题目，据说是高德地图的面试题：

> 如何在不引用第三个变量的限制条件下，交换两个变更的值？

这样的题目，几乎所有的 Pythonista 都会用下面的方法来实现：

~~~python
a, b = b, a
~~~

然后马上有人反驳了，他认为该语句右边的表达式，实际上是元组(b, a)的语法糖，这样的话，就产生了第三个变量，正确的写法应该像 c 语言里那样，使用加法或异或的方式来实现：

~~~python
a = a + b
b = a - b
a = a - b
~~~

听起来好像很有道理的样子，然而事实上并不是这样，可以做一下实验：

~~~python
# foo.py
hoge = 1
piyo = 2
hoge, piyo = piyo, hoge
~~~

执行命令 `python -m dis foo.py` ，结果是什么呢？如下：
```
  1           0 LOAD_CONST               0 (1)
              3 STORE_NAME               0 (hoge)

  2           6 LOAD_CONST               1 (2)
              9 STORE_NAME               1 (piyo)

  3          12 LOAD_NAME                1 (piyo)
             15 LOAD_NAME                0 (hoge)
             18 ROT_TWO
             19 STORE_NAME               0 (hoge)
             22 STORE_NAME               1 (piyo)
             25 LOAD_CONST               2 (None)
             28 RETURN_VALUE
```

可见并没有出现理论上应该出现的那个元组，为什么会这样呢？

究其原因，其实是Python的某些虚拟机，如CPython，在处理类似的「互换」操作时，提供了专门的优化指令「ROT_TWO」，该指令能够直接操作 stack 顶部的两个值，而并不会先计算表达式再赋值。


