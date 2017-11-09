# 含义
* 简单工程模式又叫静态方法模式（因为工厂类定义了一个静态方法）
* 现实生活，工厂是负责生产产品的；同样在设计模式中，简单工厂模式我们可以理解为负责生产对象的一个类，称为“工厂类”。

# 解决的问题
将“类”实例化的操作与“使用对象的操作分开”，让使用者不用知道具体参数就可以实例化出所需要的产品“类”，从而避免了在客户端代码中显示指定，实现了解耦。

# 模式原理
## 模式组成
|组成|关系|作用|
|:---:|:---:|:---:|
|抽象产品|具体产品的父类|描述产品的公共接口|
|具体产品|抽象产品的子类；工厂类创建的目标类|描述生成的具体产品|
|工厂|被外界调用|根据传入不同参数从而创建不同具体产品类的实例|

## 总结
* 创建型模式对类的实例化过程进行了抽象，能够将对象的创建与对象的使用过程分离
* 简单工厂模式又称为静态工厂方法模式，它属于类创建型模式。在简单工厂模式中，可以根据不同参数返回不同的类的实例。简单工厂模式专门定义了一个类负责创建其他类的实例，被创建的实例通常都具有共同的父类。
* 简单工厂模式的要点在于：当你需要什么，只需要传入一个正确的参数，就可以获取你所需要的对象，而无需知道其创建细节。
