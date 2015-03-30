### 类定义

###### 当前类

* 你可以在类定义中放入任何代码
* 类和模块也是对象（不断重复~）
* 不管处在Ruby程序的哪个位置，总是存在一个当前对象**self**；类似地，也总是有一个当前类（模块）存在。
* 在类定义中，当前对象self就是正在定义的类
* 在类定义中，当前类就是self，即正在定义的类
* Ruby解释器总是追踪当前类（或模块）的引用。所有使用**def**定义的方法都成为当前类的实例方法。
* 如果有一个类的引用，则可以用`class_eval()`（或`module_eval()`）方法来打开这个类
* 在顶级作用域定义方法时，当前对象是**main**，所以self不是类，这时当前类的角色由main的类**Object**来充当，所以**这个方法会成为Object类的实例方法**。
* **类实例变量仅仅可以被类本身访问**，不能被类的实例对象或子类访。
* 变量(@@val)可以被子类或类的实例使用。因为类变量并不属于真正的类，它们属于类体系结构。
* 避免使用类变量，尽量使用类实例变量


###### 单件方法（单件类）

* 只针对单个对象生的方法，称为**单件方法**。
```ruby
def obj.new_method
  # codes
end
```
* 类方法（可以在类中调用的方法）的实质就是**类的单件方法**。（类也是对象）
```ruby
# way 1
class MyClass
  def self.my_method; end
end

# way 2
def MyClass.my_other_method; end

# way 3
class MyClass
  class << self # self 是当前类
    def my_method; end
  end
end
```
* 每个eigenclass只有一个实例（之所以称为单件的原因），并且不能被继承

* eigenclass是一个对象的单件方法存活之所。

* **查找方法的过程：如果对象有eigenclass，则从eigenclass开始查找，如果没找到，则沿着祖先链向上来到eigenclass的超类--这个对象所在的类，然后继续查找(开始和第一章讲的一样)**

* eigenclass的超类就是超类的eigenclass；这样，子类就可调用父类的类方法。

* 所有attr_*()方法都定义在Module类中，因此不管self是类还是模块，都可以使用它们。

* **类宏**只是普通的方法。
* **#BasicObject（BasicObject的单件类）的超类是Class。**

###### Ruby对象模型

* 只有一种对象---要么是普通对象，要么是模块
* 只有一种模块---可以是普通模块、类、eigenclass或代理类
* 只有一种方法---它存在于一种模块中---通常是类中
* 每个对象（包括类）都有自己真正的类---要么是普通类，要么是eigenclass
* 除了BasicObject类无超类外，每个类**有且只有一个超类**。这意味着只有一跳向上到BasicObject的祖先链
* 一个对象的eigenlcass的超类就是这个对象的类；一个类的eigenclass的超类就是这个类的eigenclass
* 当调用一个方法时，Ruby先向右迈一步进入接收者真正的类，然后向上进祖先链。这就是Ruby查找方法的全部。

