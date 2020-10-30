# 第1章 认识Python篇

## 1.1 python的动态语言与鸭子类型

之前学习python的时候，也知道鸭子类型(ducking typing)这个说法：“当你看到一只鸟走起来像鸭子，游泳起来鸭子，叫起来也像鸭子，那么这只鸟就被称为鸭子类型”。其实字面上很容易理解，但是在Python中是如何体验的，这一点之前一直都不太懂。今天在看书，又再次碰到这个玩意，于是决定把这个鸭子类型给整明白。

按照书上的解释，鸭子类型是**多态**一种形式，在这种形式中，不管对象属于哪个类，也不管声明的具体接口是什么，只要对象实现了相应的方法，函数就可以在对象上执行操作。好吧，这句话如何理解？我们开始吧！

## 1.1.2 简单理解什么是多态?

首先要理解什么是多态，故名思意，多态就是多种状态，不同的对象调用出同一个接口，表现出多种状态。看示例1。

```python 
#示例1
class Animal():
    def who(self):
        print("I am an Animal")
class Duck(Animal):
    def who(self):
        print("I am a duck")

class Dog(Animal):
    def who(self):
        print("I am a dog")

class Cat(Animal):
    def who(self):
        print("I am a cat")
if __name__ == "__main__":
    duck=Duck()
    dog=Dog()
    cat=Cat()
    duck.who()
    dog.who()
    cat.who()
```

在示例1中，我们定义了duck，dog和cat三个对象，每个对象都实现了who()方法。你看，他们的接口名称都是相同的，他们分别调用who()方法，但是有不同的输出，不同的表现。以下是输出结果：

```python
I am a duck
I am a dog
I am a cat
```

好了，多态大概就是这么回事了。更详细的解释留在以后的文章吧!

## 1.1.3 鸭子类型

其实一般不会像上面那样去使用多态，更加优雅的写法应见示例2。

```python
#示例2
class Animal():
    def who(self):
        print("I am an Animal")
class Duck(Animal):
    def who(self):
        print("I am a duck")

class Dog(Animal):
    def who(self):
        print("I am a dog")
        
class Cat(Animal):
    def who(self):
        print("I am a cat")
        
def func(obj):
    obj.who()

if __name__ == "__main__":
    duck=Duck()
    dog=Dog()
    cat=Cat()
    func(duck)
    func(dog)
    func(cat)
```

在示例2中，我们定义一个函数func()，这个函数对参数有一个要求，那就是参数必须有who()这个方法。不管你是什么对象，是duck对象也好，是dog对象也罢，只管对象实现who()方法就可以。这就是鸭子类型，它根本不管你是什么对象，只要你有这个方法，有这个行为，表现得像鸭子，走起来像鸭子，游泳起来鸭子，叫起来也像鸭子，那么尽管你是一只会飞天的猪，也是称为鸭子类型。

## 1.2 动态语言与静态语言

我们现在思考一下，func()函数为什么可以不管传入参数是什么对象，只需要管它的方法？事实上，这跟Python是一门动态语言有关，鸭子类型是编程语言中动态类型语言中的一种设计风格。

根据维基百科，动态编程语言是这样子定义的：

>**动态编程语言**是高级编程语言的一个类别，在计算机科学领域已被广泛应用。它是一类在运行时可以改变其结构的语言：例如新的函数、对象、甚至代码可以被引进，已有的函数可以被删除或是其他结构上的变化。

动态语言是一门在运行时可以改变其结构的语言，这句话如何理解？

我们先看看示例1。

```python
#示例1
class Person(object):
    def __init__(self,name=None,age=None):
        self.name = name
        self.age = age

Jack = Person("Jack",18)
print(Jack.age)
```

在示例1中，我们定义了Person类，然后创建了Jack对象，打印对象的age属性，这没毛病。现实中人除了名字和年龄，还会有其他属性，例如身高和体重。我们尝试打印一下身高属性。

```python
print(Jack.height)
```

毫无疑问，这会报错，因为Person类中没有定义height属性。但是如果在程序运行的时候添加height属性，会发生什么呢？，请看示例2和示例3。

```python 
#示例2
Jack.height = 170
print(Jack.height)
#输出结果：170
```

```python
#示例3
setattr(Jack,'height',170)
print(Jack.height)
#输出结果：170
```

在示例2中，我们给Jack添加了height属性，然后打印，没有报错，可以输出结果。我们打印一下对象的属性。

```python
print(Jack.__dict__)
#输出结果：
# {'name': 'Jack', 'age': 18, 'height': 170}
```

你看，本来对象是没有height属性，但是可以在程序运行过程中给实例动态绑定属性，这就是动态语言的魅力，不过还是有一些坑的，我们再看看示例4。

```python
#示例4
Mia = Person('Mia',18)
print(Mia.__dict__)
#输出结果：
# {'name': 'mia', 'age': 18}
```

奇怪！Mia对象居然没有height属性。为什么？事实上，在示例2中，我们只是给**类示例**动态地绑定了一个属性，而不是给**类**绑定属性，所以重新创建的对象是没有height属性的。如果想要给类添加，也是可以的，见示例5。

```python
#示例5
Person.height = None
Mia = Person("Mia",18)
print(Mia.height)
#输出结果：None
```

搞定了属性的动态绑定，其实动态删除也是同一个道理，请看示例5。

```python
#示例5
Mia = Person("Mia",18)
delattr(Mia，'height')
print(Mia.__dict__)
#输出结果：{'name': 'mia', 'age': 18}
```

搞定了属性的动态绑定和删除，接下来看看方法的绑定和删除，请看示例6。

```python
#示例6
class Person(object):
    def __init__(self,name=None,age=None):
        self.name = name
        self.age = age
        
def speak_name(self):
    print(self.name)

Jack = Person("Jack",18)
Jack.speak_name = speak_name
Jack.speak_name(Jack)
print(Jack.__dict__)
Mia = Person("Mia",18)
print(Mia.__dict__)
```

输出结果：

```python
Jack
{'name': 'Jack', 'age': 18, 'speak': <function speak at 0x000001FE883B2EA0>}
{'name': 'Mia', 'age': 18}
```

在示例6中，对象Jack的属性中已经成功添加了speak函数。但是！有没有感觉示例6中，这个语句

```python
Jack.speak_name(Jack)
```

很别扭。按常理来说，应该

```python 
Jack.speak_name()
```

就行了。如果想要达到这种效果，应该要像下面这样子做。

```python
import types
Jack.speak_name = types.MethodType(speak_name,Jack)
Jack.speak_name(）
#输出结果：Jack
```

其中MethodType用于绑定方法对象。

当然示例6都是给类示例绑定了方法，但是如果要给类绑定方法的话，又应该怎么做？请看示例7。

```python
#示例7
import types
class Person(object):
    def __init__(self,name=None,age=None):
        self.name = name
        self.age = age

def speak_ok(cls):
    print(OK)
 
Person.speak_name = types.MethodType(speak_ok,Person)
Person.speak_ok()
# 输出结果：OK
```

示例1-7给大家解析了维基百科对动态语言的定义，希望可以帮助你对Python的理解。

下面说一下比较容易混淆的概念。

**动态类型语言与动态语言**

其实动态类型语言跟动态语言是不一样的概念。

>动态类型语言指的是在运行期间才去判断数据类型的语言，强调的是数据类型。
>
>动态语言指的是它是在运行时可以改变其结构的语言，强调的是代码结构。

**静态类型语言与静态语言**

> 静态类型语言指的是运行之前（编译期间）会去判断数据类型的语言，强调的也是数据类型。
>
> 静态语言指的是在运行期间不能改变其结构的语言，强调的是代码结构。





* 