---
title: python 基础
date: 2020-12-15 07:12:49
tags: [python,引用计数]
---

大家好啊，这是我的第一篇博客

## 一、引用计数：

- ### 概念：

​		当一个对象被创建或者调用时，引用计数 +1，当对象被删除时，引用计数 -1

- ### 查看使用sys模块：

```python
import sys

'''python3默认继承object'''
# 创建Person类  对象被创建，引用计数+1
class Person:
    pass
# 将类附给p1  对象被调用，引用计数+1
p1 = Person()
# 将p1附给p2  对象再次被调用，引用计数+1
p2 = p1
# 删除p2 对象被删除，引用计数-1
del p2
# 打印引用计数
print(sys.getrefcount(p1))  ## 2
```

- ### 嵌套引用/循环引用：

```python
'''
下面的情况形成了嵌套引用，故不能使用上面的方法查看引用计数
'''
class Person:
    pass

class Dog:
    pass

# p为人，d为狗
p = Person()
d = Dog()
# 人的宠物是狗，狗的主人是人
p.pet = d
d.master = p
```



## 二、分代回收

概念：

​		在嵌套引用的前提下，python每隔一段时间就会对原有的数据进行扫描，然后将内存分为三代，年轻代、中年代和老年代，扫描时一旦发现引用计数为0，就将对象回收，老年代中的对象就是存活时间最长的对象

>1、新创建的对象做为0代
>
>2、每执行一个【标记-删除】，存活的对象代数就+1
>
>3、代数越高的对象（存活越持久的对象），进行【标记-删除】的时间间隔就越长


三种情况触发垃圾回收机制：

1. 调用gc.collect()
2. GC达到阀值
3. 程序退出



## 三、解释器

​		python默认解释器为Cpython，具有GIL全局解释器锁

​		pypy3另一种python解释器，对python代码进行动态编译，提高执行效率

```python
num = 0

def change_it(n):
    global num
    # 人为的构造高并发
    for i in range(1000000):
        num += n
        num -= n

    print(num)

threading = [
    # 方法后加括號尾調用
    threading.Thread(target=change_it, args=(8,)),
    threading.Thread(target=change_it, args=(10,))
]

[t.start() for t in threading]
# join 阻塞操作，允许主线程执行完后再执行子线程
[t.join() for t in threading]
# 结果
>python tests.py  >>>0 br/ 0   第二次 >>> 8 br/ 8
>pypy3 tests.py  >>> 0 br/ 0   第二次 >>> 0 br/ 0
'''pypy3更加准确'''

# 查看运行时间
import time

s1 = time.time()
ss = [x**2 for x in range(10000000)]

print("运行时间：",time.time()-s1)

>python tests.py  >>>2.761610746383667
>pypy3 tests.py  >>>6.997313022613525
'''python更快'''
```


## 四、工厂模式

只管接受材料和最终成品，不管是怎么做的

```python
class Dingding:
    def __repr__(self):
        return "钉钉登录"


class Facebook:
    def __repr__(self):
        return "脸书登录"

# 调用实例化对象
# ding = Dingding()
# face = Facebook()
#
# print(ding)

# 工厂类（面向接口开发）
class LoginFactory:
    @staticmethod  # 静态方法，不需要实例化对象
    def test_login(name):
        if name == "ding":
            return Dingding()
        elif name == "face":
            return Facebook()

a1 = LoginFactory.test_login('face')

print(a1)
```

