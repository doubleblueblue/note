学习网站：https://refactoringguru.cn/design-patterns/factory-method
# 设计模式基础概念
大多数设计模式的概念和面向对象的概念是一致的，但有些有些许不同或增补：如下
1. 混入类(mixin Class):指一个可以继承了多个虚接口的虚类
2. 类继承和接口继承。如果按照我的个人理解，类继承会同时继承声明和实现，接口继承则是只需要继承声明即可。书上原话的描述非常精准-类继承根据一个对象的实现定义了另一个对象的实现。它是代码和表示的共享机制。接口继承描述了一个对象什么时候能被用来替代另一个对象。
3. 对接口编程而不是对实现编程：这个具体到操作规范，想一下前公司的代码,总会有一个基类负责实现，一个基类负责接口。负责实现的基类一般都是原本的控件，负责接口的则是另外的自定义纯虚类。这个设计非常符合CPP的原生设计，CPP原生设计中分开了声明和定义，但是类的继承其实并没有分开（声明的继承和实现的继承），面向接口编程本质上就是分开声明的继承和实现的继承。回想一下绝大多数的编程场景，我们并非真的希望从父类去继承一个实现，大多数时候只是约束所有的子类都应该有这个声明，因此这个时候其实另起一个纯虚类是更加合适的。
# 设计模式的分类
## 创建型设计模式
### <font color="green">工厂模式(Factory Method)</font> 
工厂也有其分类：</br>
1. 简单工厂模式，提供默认的实现，创建具体的对象
2. 工厂方法模式，不提供实现，创建继承于baseFactory的工厂，由工厂进行具体实现，uml如下：
<div align='center'><img style="background:CornflowerBlue;color:CornflowerBlue;" src="./pic/umlFactory.svg"></div>

### <font color="green">抽象工厂模式(Abstract Factory)</font> 
抽象工厂则是工厂方法模式的一个优化方案，在工厂方法模式下，如果要进行进一步的分类，可能会导致工厂的派生工厂极其之多，抽象工厂则是取其中有共性的一部分，抽象出来一个抽象类。uml如下：
<div align='center'><img style="background:CornflowerBlue;color:CornflowerBlue;" src="./pic/umlAbstractFactory.svg"></div>

### <font color="green">建造者模式(builder pattern)</font>
建造者模式是一个优化构造函数的模式，还在苦于构造函数过于庞大，赋值操作过于冗余，总是要给一些不是自己的参数赋值的苦恼吗？考虑一下建造者模式吧。uml图如下：
<div align='center'><img style="background:CornflowerBlue;color:CornflowerBlue;" src="./pic/umlBuilder.svg"></div>

### <font color="green">原型模式(Prototype)</font>
//TODO
### <font color="green">单例模式(Singleton)</font>
看图，不多BB：
<div align='center'><img style="background:CornflowerBlue;color:CornflowerBlue;" src="./pic/umlSingleton.svg"></div>

## 结构型设计模式
### 适配器模式(Adapter)



### 桥接模式(bridge)s
### 组合模式(Composite)
### 装饰器模式(Decorator)
这个装饰器模式，如果是按照我的理解来命名，我觉得它更像是一个装饰链（在线性思维中），当然套娃的比喻也很形象。跟策略模式或者说回调相比，这个更为自由，最大的特点就是可以多层套，也有策略的自由。
uml图如下：



### 外观模式(Facade)
### 享元模式(Flyweight)
### 代理模式(Proxy)
## 行为型设计模式
### 解释器模式(interpreter)
### 模板模式(Template Method)
### 责任链模式(Chain of Responsibility)
### 命令模式(Command)
### 迭代器模式(Iterator)
迭代器模式在我看来也是策略模式的一种变种，它的主要目的是解决不同方式遍历整个聚合对象。这种不同，就是通过策略模式来实现的。应该说，迭代器模式就是一个重点关注迭代算法步骤的策略。
### 中介者模式(Mediator)
### 备忘录模式(Memento)
### <font color="green">观察者模式(Observer)</font>
### 状态模式(State)
### <font color="green">策略模式(Strategy)</font>
策略模式的目的是，让一个行为可以方便地替换多种方案，比如函数A有可能使用多种计算方法（变化的），那么我们去额外写一个计算接口类，派生出实际的计算方法。在函数A中声明一个对象，然后在A的计算方法中调用对应的派生出的实际的计算方法。好处是之后需要添加或更改方法的时候只需要派生出新的子类，然后在A中修改调用即可。</br>
类图如下：
<div align='center'><img style="background:CornflowerBlue;color:CornflowerBlue;" src="./pic/umlStrategy.svg"></div>

### 访问者模式(Vistor)
访问者模式的优点：
1. 开闭原则：可以在不同类的对象上执行新的行为,且无需对这些类做修改。（无非是增加个调用，两行代码，不会是加了逻辑的200行）。
2. 单一职责原则：可以将同一行为的不同版本放在同一个类中。
3. 访问者对象可以在交互时收集一些对象的有用信息。
访问者模式的缺点：
1. 每次在元素层次结构中添加和移除一个类时，都要更新所有的访问者。
2. 访问者在和对象交互时，是会受到权限的限制的，并不能有protected或者private的权限。