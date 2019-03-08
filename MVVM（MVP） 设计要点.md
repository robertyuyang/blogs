1.View层：
ViewController是View层，但依然鼓励多写View的子类

2.ViewModel层:

2.1 View层的h和m里尽量不能包含model层的头文件（包括前置声明）

——View层永远不依赖model层，即使是一个简单的TableViewCell。

2.2 ViewModel头文件里尽量不要包含model层的头文件（包括前置引用），如果要包含，尽量只在init里把model做入参，其他方法尽量少处理model对象，尤其不要把model对象当输出参数，除非“万不得已”

——不能让ViewModel层轻易把它封装的model吐出来，否则失去了封装的意义。万不得已：ViewModel在与其他ViewModel通信时需要交互复杂对象。

2.3 ViewModel和上下层（View，Model）可以不严格按MVVM框架中用数据双向绑定去连接，但切记不可反向依赖。比如只能View依赖ViewModel，不能反过来，如果需要ViewModel往View传数据，可以用数据绑定、delegate、block、观察者，NSNotification。（实际上已经接近MVP）

2.4 尽量让大ViewModel生出小ViewModel。


3.Model层：
尽量使用Service层分离原来的model层。
广义的model层主要解决这么几个问题：
- 数据获取
- 数据加工
- 数据持久
- 数据展现

model尽量瘦，只搭建模型和encode方式。
- 数据获取放到Service层，让Service去做从网络或本地获取数据的工作。

- 垮model和复杂的数据加工要放到逻辑层（ViewModel，Presenter，Controller）或者Service层。如果数据处理的要素都在同一个model，简单加工可放到model里。
- model层只规定encode，decode方式，具体的持久时机，持久位置，持久逻辑，尽量让Service层或者逻辑层去做。
- model把数据展现工作交给ViewModel。ViewModel有两个重要的职责：1.数据加工（业务逻辑）2.隔离：将view不关心的model的部分隔离掉，从而做到抽象。

4.增加类设计层次，多继承，多抽象，多用@protocol。

5.尽量不使用单例

6.写任何一个#import和@class前多思考该不该这么做

7.是否用Pod私有库，url路由，各模块开发者酌情。