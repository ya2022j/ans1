## python接口自动化之mock的使用
https://blog.csdn.net/qq_37982823/article/details/122607222

一、Mock是什么？

mock 就是模拟的意思，它的主要功能是使用mock对象替代掉指定的Python对象，以达到模拟对象的行为。在接口数据字段还没开发好，我们可以在写接口自动化的时候，事先使用mock数据。

二、为什么要使用的mock?

在做接口测试时，开发还未完成接口的实现，自动化接口测试代码就没办法完成，这个时候就需要用Mock对象库来模拟接口响应结果，等到开发完成接口功能，再把Mock模拟响应结果的代码删除掉，整个自动化接口测试项目就完成了。

三、Mock可以解决哪些场景问题？

接口的依赖
外部接口调用
测试环境非常复杂
具体来说就是：
1.前后端联调，如果你是一个前端页面开发，现在需要开发一个功能： 下一个订单，支付页面的接口，根据支付结果，支付成功，展示支付成功页，支付失败，展示支付失败页。 要完成此功能，你需要调用后端的接口，根据返回给你的结果，来展示不同的页面。此时后端接口还没开发好，
作为一个前端开发总不能等别人开发好了，你再开发，那你只有加班的命了。 为了同步开发完成任务，此时，你可以根据接口文档的规定，把接口的地址和入参传过去，然后自己mock接口的不同返回界面，来完成前端的开发任务
2.单元测试，单元测试的目的是测试某个小小单元的功能，但现实中开发的函数或方法都是有依赖关系的，比如b函数的参数，需要调用a函数的返回结果,但是我前面已经测试a函数了 这种情况下，就不需要再测一次a函数了，此时就可以用mock模块来模拟调用这部分内容，并给出返回结果。
3.第三方接口依赖，在做接口自动化的时候，有时候需要调用第三方的接口，但是别人公司的接口服务不受你的控制，有可能别人提供的测试环境今天服务给你开着，别人就关掉了，
例如：
我们要测试A模块，然后A模块依赖于B模块的调用。但是，由于B模块的改变，导致了A模块返回结果的改变，从而使A模块的测试用例失败。其实，对于A模块，以及A模块的用例来说，并没有变化，不应该失败才对。
这个时候就是mock发挥作用的时候了。通过mock模拟掉影响A模块的部分（B模块）。至于mock掉的部分（B模块）应该由其它用例来测试。

四、mock的使用

在Python2.x 中 mock是一个单独模块，需要单独安装。
pip install -U mock
在Python3.x中，mock已经被集成到了unittest单元测试框架中，所以，可以直接使用。
from unittest.mock import Mock

五、mock的代码实现
1、mock的基本使用

# -*- coding: utf-8 -*-
# @Time: 2022/1/20 5:50 下午
# @Author: wcystart
# @File: modular.py
# @description:
"""
这里要实现一个Count计算类，add() 方法要实现两数相加。
但，这个功能我还没有完成。这时就可以借助mock对其进行测试。
"""

```python
# -*- coding: utf-8 -*-
# @Time: 2022/1/20 5:51 下午
# @Author: wcystart
# @File: mock_demo01.py
# @description:
from unittest.mock import Mock
import unittest
from modular import Count


class TestCount(unittest.TestCase):
    def test_add(self):
        # 首先，调用被测试类Count()
        count = Count()
        # 通过Mock类模拟被调用的方法add()方法，return_value 定义add()方法的返回值。
        count.add = Mock(return_value=13)
        # 正常的调用add()方法，传两个参数8和5，然后会得到相加的结果13。然后，13的结果是我们在上一步就预先设定好的。
        result = count.add(8, 5)
        self.assertEqual(result, 13)


if __name__ == '__main__':
    unittest.main()


```
class Count():
    def add(self):
        pass





```python

# -*- coding: utf-8 -*-
# @Time: 2022/1/21 11:13 上午
# @Author: wcystart
# @File: function.py
# @description: 定义一个函数，用来计算两个数的和、乘积

def add_and_multiply(x, y):
    addition = x + y
    mul = multiply(x, y)
    return addition, mul


def multiply(x, y):
    return x * y
```
2、mock解决依赖





然后针对：add_and_multiply()函数编写测试用例


```python



import unittest
import function


class MyTestCase(unittest.TestCase):
    def test_add_and_multiple(self):
        x = 3
        y = 5
        addition, mul = function.add_and_multiply(x, y)
        self.assertEqual(8, addition)
        self.assertEqual(15, mul)


if __name__ == '__main__':
    unittest.main()
```

运行结果：
Ran 1 test in 0.004s

OK


add_and_multiply()函数依赖了multiply()函数的返回值。如果这个时候修改multiply()函数的代码。

```python


def multiply(x, y):
    return x * y + 6


再次运行，就会断言失败：
21 != 15

Expected :15
Actual   :21
<Click to see difference>

Traceback (most recent call last):
  File "/Users/wangchuangyan/Desktop/py42/mock/func_test.py", line 16, in test_add_and_multiple
    self.assertEqual(15, mul)
AssertionError: 15 != 21
```

测试用例运行失败了，然而，add_and_multiply()函数以及它的测试用例并没有做任何修改，罪魁祸首是multiply()函数引起的，那么此时我们应该把 multiply()函数mock掉。
```python



import unittest
import function
from unittest.mock import patch


class MyTestCase(unittest.TestCase):
    # 针对add_and_multiply做测试用例
    # def test_add_and_multiply(self):
    #     x = 3
    #     y = 5
    #     addition, multiple = function.add_and_multiply(x, y)
    #     self.assertEqual(8, addition)
    #     self.assertEqual(15, multiple)

    """
    patch()装饰/上下文管理器可以很容易地模拟类或对象在模块测试。
    在测试过程中，您指定的对象将被替换为一个模拟（或其他对象），
    并在测试结束时还原。
    """

    # 模拟function.py文件中multiply()函数
    # 将mock的multiply()函数（对象）重命名为 mock_multiply对象
    @patch("function.multiply")
    def test_add_and_multiple2(self, mock_multiply):
        x = 3
        y = 5
        # 设定mock_multiply对象的返回值为固定的15
        mock_multiply.return_value = 15
        addition, multiple = function.add_and_multiply(x, y)
        # 检查mock_multiply方法的参数是否正确
        mock_multiply.assert_called_once_with(3, 5)

        self.assertEqual(8, addition)
        self.assertEqual(15, multiple)


if __name__ == '__main__':
    unittest.main()
```

再次运行：运行通过
Ran 1 test in 0.004s

OK

Process finished with exit code 0


六、patch装饰器简介

patch装饰器的使用,patch() 作为函数装饰器，为您创建模拟并将其传递到装饰函数

def patch(
target, new=DEFAULT, spec=None, create=False,
spec_set=None, autospec=None, new_callable=None, **kwargs
):
target参数必须是一个str,格式为’package.module.ClassName’， 注意这里的格式一定要写对，如果你的函数或类写在pakege名称为a下，b.py脚本里，有个c的函数（或类），那这个参数就写“a.b.c”
new参数如果没写，默认指定的是MagicMock
spec=True或spec_set=True，这会导致patch传递给被模拟为spec / spec_set的对象 new_callable允许您指定将被调用以创建新对象的不同类或可调用对象。默认情况下MagicMock使用。


