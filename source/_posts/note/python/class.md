---
title: python 面对对象
categories:
  - note
---

python 面对对象

<!--more-->

### 一、定义

- 类与对象

  - 类是模子，确定对象将会拥有的特征（属性）和行为（方法）
  - 对象是类的实例表现
  - 类是对象的类型
  - 对象是特定类型的数据

- 属性与方法
  - 属性：对象具有的各种静态特征
  - 方法：对象具有的各种动态行为

### 二、 三大特性

- 封装：对外部隐藏有关对象工作原理的细节，提供一些可以被外界访问的属性的方法

- 继承：继承是使用现有类的所有功能，并在无需重新编写原来的类的情况下对这些功能进行拓展

- 多态：可对不同类型的对象执行相同的操作，得到的确是不同的结果

```
class Cat(object):
    """
    猫科动物 # __doc__ 文档信息
    """
    # 类的属性
    tag = 'Cat base'

    def __init__(self, name, age, sex=None):
        # 构造函数（默认构造函数可以不传参）
        self.name = name  # 实例化后的属性
        self.__age = age  # 私有属性
        self.sex = sex
        pass

    # 方法
    def showInfo(self):
        """
        显示信息
        """
        rest = '我叫：{0}，年龄：{1}，性别：{2}，tag：{3}'.format(
            self.name, self.__age, self.sex, self.tag)
        return rest

    def eat(self):
        print('要吃东西')

    # 析构函数
    def __del__(self):
        pass


class Beast(object):
    def eat(self):
        print('喜欢吃肉')

    def danger(self):
        print('我是危险猛兽')

    # 私有方法（两个下划线开头，声明该方法为私有方法，只能在类的内部调用）
    def __foo(self):
        print('这是私有方法')


class Tiger(Cat, Beast):
    tag: 'Tiger base'

    def __init__(self, name, age, color='yellow'):
        # 调用父类构造函数（如果子类不重写 __init__，则默认调用父类方法）
        super().__init__(name, age)
        self.color = color

    def eat(self):
        super().eat()
        print('特喜欢吃猪肉')

    # 方法（重载/重写）
    def showInfo(self):
        """
        显示信息
        """
        rest = '我叫：{0}，性别：{1}，tag：{2}, 颜色：{3}'.format(
            self.name, self.sex, self.tag, self.color)
        return rest


if __name__ == '__main__':
    # 实例化
    cat_black = Cat('ss', 2)

    # 报错，不能直接访问到私有变量，解析器对外把 __age 变量改成了 _Cat__age
    # print(cat_black.__age)
    # 不推荐使用 _Cat__age，因为 Python 可能会把 __age 改成不同的变量名
    # print(cat_black._Cat__age) # 2
    print(cat_black.showInfo()) # 我叫：ss，年龄：2，性别：None，tag：Cat base

    # 修改实例属性
    cat_black.name = '黑黑'
    cat_black.__age = 6  # 不报错
    print('cat_black.__age：', cat_black.__age)  # 经过上次设置后直接访问返回 6
    print(cat_black.showInfo())  # 我叫：黑黑，年龄：2，性别：None，tag：Cat base（打印的年龄信息仍是 2，因为外部设置的 __age 跟类内部的 __age 不是一个变量）

    # 判断对象是否为类的实例
    print('判断 cat_black 是否为 Cat 的实例：', isinstance(cat_black, Cat)) # True
    print('判断 cat_black 是否为 Tiger 的实例：', isinstance(cat_black, Tiger)) # False
    print('--------------------------------------\n')

    # 实例可以访问类属性
    cat_white = Cat('小白', 3)
    print(cat_white.showInfo()) # 我叫：小白，年龄：3，性别：None，tag：Cat base
    print('实例可以访问类属性：', cat_black.tag == Cat.tag and cat_white.tag == Cat.tag) # True

    # 类属性会被实例属性覆盖
    cat_white.tag = 'ss'
    print('类属性会被实例属性覆盖：', cat_white.tag != Cat.tag) # True
    print(cat_white.showInfo()) # 我叫：小白，年龄：3，性别：None，tag：ss
    print('--------------------------------------\n')

    # 类的继承
    tiger = Tiger('东北虎', 3)
    print(tiger.showInfo()) # 我叫：东北虎，性别：None，tag：Cat base, 颜色：yellow

    # 子类调用父类方法
    tiger.danger() # 我是危险猛兽

    # 多重继承时如果方法重名，则只继承第一个继承的父类
    # 如果子类未定义方法，则按类声明的圆括号中基类顺序从左至右搜索基类中是否包含方法
    tiger.eat() # 要吃东西\n特喜欢吃猪肉

    # 判断是否为子孙类
    print('判断 Tiger 是否为 Cat 子孙类：', issubclass(Tiger, Cat)) # True
    print('判断 Tiger 是否为 Beast 子孙类：', issubclass(Tiger, Beast)) # True
```

### 三、 类的高级特性

- @property: 将类的方法当做属性来使用

```
class PetCat(object):
    """ 家猫类 """

    def __init__(self, name, age):
        """
        构造方法
        :param name: 猫的名称
        :param age: 猫的年龄
        """
        self.name = name
        # 私有属性，不能给你们操作
        self.__age = age

    @property
    def age(self):
        return self.__age

    @age.setter
    def age(self, value):
        if not isinstance(value, int):
            print('年龄只能是整数')
            return 0
        if value < 0 or value > 100:
            print('年龄只能介于0-100之间')
            return 0
        self.__age = value

    # 描述符
    @property
    def show_info(self):
        """ 显示猫的信息 """
        return '我叫：{0}，今年{1}岁'.format(self.name, self.age)

    def __str__(self):
        """ 显示实例对象的描述信息 """
        return '我的对象: {0}'.format(self.name)


if __name__ == '__main__':
    cat_black = PetCat('小黑', 2)
    rest = cat_black.show_info
    print(rest) # # 我叫：小黑，今年2岁
    # 改变猫的age
    cat_black.age = 'hello' # 年龄只能是整数
    rest = cat_black.show_info
    print(rest) # 我叫：小黑，今年2岁
    # 打印对象的描述信息，可在 __str__ 方法中自定义
    print(cat_black) # 我的对象: 小黑
```

- `__slots__`: 限制实例的属性

```
class PetCat(object):
    """ 家猫类 """

    __slots__ = ('name', 'age')

    def __init__(self, name, age):
        """
        构造方法
        :param name: 猫吃的名称
        :param age: 猫的年龄
        """
        self.name = name
        self.age = age

    @property
    def show_info(self):
        """ 显示猫的信息 """
        return '我叫：{0}，今年{1}岁'.format(self.name, self.age)


class HuaCat(PetCat):
    """ 中华田园猫 """
    __slots__ = ('color', )

def eat():
    print('我喜欢吃鱼')


if __name__ == '__main__':
    # cat_black = PetCat('小黑', 2)
    # rest = cat_black.show_info
    # print(rest)

    # # 给实例添加新的属性（使用 slots 后不允许给实例添加新的属性）
    # cat_black.color = '白色'
    # print(cat_black.color)
    # # 给实例添加新的方法（使用 slots 后不允许给实例添加新的方法）
    # cat_black.eat = eat
    # cat_black.eat()

    # slots 定义的属性仅对当前类实例起作用，对继承的子类是不起作用
    # 除非在子类中也定义 slots，这样，子类实例允许定义的属性就是自身的 slots 加上父类的 slots
    cat_white = HuaCat('小白', 3)
    rest = cat_white.show_info
    print(rest) # 我叫：小白，今年3岁
    cat_white.color = '白色'
    print(cat_white.color) # 白色
    cat_white.name = '旺旺'
    print(cat_white.show_info) # 我叫：旺旺，今年3岁
```

### 四、 类的静态方法和实例方法

```
class Cat(object):
    tag = '猫科动物'

    def __init__(self, name):
        self.name = name

    # 类的静态方法
    @staticmethod
    def breath():
        """ 呼吸 """
        print('猫都需要呼吸空气')

    @classmethod
    def show_info(cls, name):
        """ 显示猫的信息，cls 代表当前类 """
        # print('类的属性：{0}， 实例的属性： {1}'.format(cls.tag, cls.name))
        return cls(name)

    def show_info2(self):
        """ 显示猫的信息 """
        # 设计模式
        print('类的属性：{0}， 实例的属性： {1}'.format(self.tag, self.name))


if __name__ == '__main__':
    # 通过类进行调用静态方法
    Cat.breath() # 猫都需要呼吸空气

    # 通过类的实例进行调用静态方法
    cat = Cat('小黑')
    cat.breath() # 猫都需要呼吸空气

    # 方法内 self 既可以访问实例属性也可以访问类属性
    cat.show_info2() # 类的属性：猫科动物， 实例的属性： 小黑

    # 类调用 classMethod
    cat2 = Cat.show_info('小黄')
    cat2.show_info2() # 类的属性：猫科动物， 实例的属性： 小黄

    # 实例调用 classMethod
    cat3 = cat.show_info('小蓝')
    cat3.show_info2() # 类的属性：猫科动物， 实例的属性： 小蓝
```

### 五、 类的专有方法

- `__init__` : 构造函数，在生成对象时调用
- `__del__ `: 析构函数，释放对象时使用
- `__repr__ `: 打印，转换
- `__setitem__ `: 按照索引赋值
- `__getitem__`: 按照索引获取值
- `__len__`: 获得长度
- `__cmp__`: 比较运算
- `__call__`: 函数调用
- `__add__`: 加运算
- `__sub__`: 减运算
- `__mul__`: 乘运算
- `__truediv__`: 除运算
- `__mod__`: 求余运算
- `__pow__`: 乘方

```
# 运算符重载，对类的专有方法进行重载
class Vector:
   def __init__(self, a, b):
      self.a = a
      self.b = b

   def __str__(self):
      return 'Vector (%d, %d)' % (self.a, self.b)

   def __add__(self,other):
      return Vector(self.a + other.a, self.b + other.b)

v1 = Vector(2,10)
v2 = Vector(5,-2)
print (v1 + v2) # Vector (7, 8)
```
