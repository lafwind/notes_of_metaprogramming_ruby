### 第一章 对象模型

##### 重要概念

* 打开类。

  * 打开已存在的类并对其进行动态修改。

* Ruby 的 class 关键字更像是一个作用域而不是类型声明语句。

  * 核心任务是把你带到类的上下文中，让你可以在其中定义方法。

* 猴子打补丁（Monkeypatch）。

  * 为某个类添加的方法的名字与该类原有方法重名。

* 对象

  * 对象仅仅包含它的实例变量（@v）以及一个对自身类的引用。
 
  * 一个对象的实例变量存在于对象本身，而对象的方法存在于对象自身的类。这即是同一个类的对象共享同样的方法，但不共享实例变量的原因。

* 类和模块

  * _类自身也是对象_，它也有自己的类，这个类的名字叫*Class*。这意味着_一个类的方法（**不同于类方法**）就是*Class*的实例方法_。

  * 一个类是一个增强的Module（一组实例方法），增加了_new()_、_allocate()_和_superclass()_这三个方法。

  * 绝大多数适用于类的内容也同样适用于模块。反之亦然。

  * 如果希望代码能在别处被包含（include）或当成_命名空间_，应使用模块；当希望代码被实例化或被继承时，选择使用类。
  
  * Module的超类是Object；Object是一个类，所以它的类是Class（**这对所有类来说都成立！**）
  
  * 所有类最终都继承于Object；Object本身又继承于BasicObject；Class的超类（superclass）是Module；Module的超类是Object

* 常量

  * 常量的作用域不同于变量的作用域。

  * 常量像文件系统一样以树状结构组织，模块（类）像目录，常量像文件。

  * 常量也能修改，但会被Ruby解释器警告！

**总结：对象是一组实例变量外加一个指向类的引用。对象的方法存在于对象的类中，被称为类的实例方法；类是一个对象（Class类的一个实例，也有实例变量（类实例变量）和指向类的引用）外加一组实例方法和对其超类的引用，Class类是Module类的子类，即一个类也是一个模块；和其他对象一样，类作为对象也有自己的方法（比如new()），这些是Class的实例方法；类必须通过引用访问（类的名字）**

* 调用方法

  * 方法查找：按祖先链寻找。先在接收者所在的类中查找，如找不到，沿着超类，超类的超类这条祖先链往上查找。（Class.ancestors()方法）
  
    * Object类是所有类默认的超类；BasicObject类是Ruby类体系结构的根节点
    
    * 向右一步，再向上

  * 执行方法，需要确定方法接收对象（**self**）。
  
    * **每一行代码都会在一个对象中执行--这个对象就是所谓的当前对象**
    
    * **self** 可表示当前对象。因此可以用self关键字来访问它
    
    * 当调用一个方法时，接收者就成为self。所有的实例变量都是self的实例变量
    
    * 未明确指明接收者的方法都在当前self上调用
    
    * 代码调用其他对象的方法时，这个对象成为self
    
    * 实例变量永远被认定为**self**的实例变量
    
    * 没调用任何方法时，self是main对象，这个对象有时被称为顶级上下文（因为这时处在调用堆栈的顶层）
    
```ruby
class MyClass
  def testing_self
    @var = 10 # self的一个实例变量
    my_method() # 跟self.mythod()相同
    self
  end
  
  def my_method
    @var = @var + 1
  end
end

obj = MyClass.new
obj.testing_self # => <MyClass:0x510000 @var = 11>
```

  * 祖先链也包括模块。Module在祖先链中的位置位于包含该module的类的正上方（生成该module的匿名类）。思考一个类中包含多个模块。

  * 私有方法：只能在自身被调用。“私有原则”由以下两条规则决定：

    * 如要方法的接收者不是自己，应明确指明一个接收者

    * 私用方法只能被隐含接收者调用

  * 在类和模块定义中（并且在任何方法定义外），self的角色由这个**类或模块**担任。

  * kernel模块被object类包含，所有它出现在所有对象的祖先链中。如果给Kernel模块增加一个方法，这个**内核方法（Kernel Method）**就对所有对象可用。


##### 实用方法

_注意：Myclass#method表示Myclass的实例方法；Myclass.method表示Myclass的类方法；Myclass::CONSTANT表示类中的一个常量。_

```ruby

# 对象：

obj.instance_variables # 查找对象实例变量
obj.instance_variable_set("@v", 10) # 让对象获得实例变量@v

# 类：

Myclass.class # 查找类的类（略拗口= =）
Myclass.superclass # 查找类的超类
Myclass.instance_methods # 查找类的实例方法
Array.methods # 查找Array中的所有方法
Hash.methods.grep /^re/ # 查找Hash中所有以re开头的方法
MyClass.ancestors # => 祖先链

# 常量：

M.constants # Module#constans()，返回当前范围内的常量，为一个module
Module.contants # 返回当前程序中所有顶级常量，包括类名。
```
