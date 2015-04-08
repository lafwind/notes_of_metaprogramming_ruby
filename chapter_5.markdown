### 第五章 编写代码的代码

###### Kernel#eval

* Kernel#eval()方法会直接执行字符串中的代码，并返回执行结

```ruby
array = [10, 20]
element = 30

# eval(statements, @binding, file, line)
eval("array << element") # => [10, 20, 30]
```

* Binding对象（是一个用对象表示的完整作用域）是一个比更“纯净”的闭包，因为它们只包作用域而不包含代码

* eval()是\*eval()家族的一员：跟instance_eval()和class_eval()方法不同，它执行的是一个代码字符串，而不是块

* eval()需要一个代码字符串作为参数，而instance_eval()和class_eval()除了块，也可以接受代码字符串作为参数

* 代码字符串中的代码与块中的代码没有太大的区别。代码字符串可以像块那样访问局部变量。

```ruby
array = ['a', 'b', 'c']
x = 'd'
array.instance_eval "self[1] = x"
array # => ['a', 'd', 'c']
```

* 只要能用块，尽量用块，代码字符串有些缺点。

* 代码字符串的缺点：
  * 不能利用编辑器的功能特性，比如语法高亮和自动完成
  * 难以阅读和修改
  * 运行前不会进行语法检查，容易导致程在运行时出乎意料的失败
  * 安全
    * 代码注入(污染对象，安全级别)
    ```ruby
    def explore_array(method)
      code = "['a', 'b', 'c'].#{method}"
      puts "Evaluating: #{code}"
      eval code
    end
    
    loop do
      p explore_array(gets())
    end
    
    # 代码注入
    object_id; Dir.glob("*")
    => ['a', 'b', 'c'].object_id; Dir.glob("*")
    ```
    
###### 钩子方法（Hook Methods）

* 钩子方法，像钩子一样挂在一个特定事件上
```ruby
class String
  def self.inherited(subclass)
    puts "#{self} was inherited by #{subclass}"
  end
end

class MyString < String; end
=> String was inherited by MyString
```
* 钩子方法例子：Class#inherited()、**Module#included()**、Module#method_add()、Module#method_removed、Module#method_undefined()

* 类扩展混入---类扩展和钩子方法的结合，步骤（得到一个混入，用来给包含者添加类方法（通常是类宏））：

  * 定义一个模块，比如叫MyMixin
  
  * 在MyMixin中定义一个内部模块（通常叫做ClassMethods），并给它定义一些方法。这些方法最终会成为类方。
  
  * 覆写MyMixin#included()方法来用ClassMethods扩展包含者(使用extend()方法，该方法会把ClassMethods的方法混入到包含者的eigenclass中，而eigenclass的实例方法即为该类的类方法)
 
```ruby
# 完整版
module MyMixin
  def self.included(base)
    base.extend(ClassMethods)
  end
  
  # 包含者的实例方法
  def y
    "y()"
  end
  
  # 包含者的类方法
  module ClassMethods
    def x
      "x()"
    end
  end
end

# 如果不需要为包含者定义实例方法，
# 则根本不用定义内部模块，
# 只要把所有方法都定义在混入本身(self)

module MyMixin
  def self.included(base)
    base.extend(self) # MyMixin
  end
  
  #包含者的类方法
  def x
    "x()"
  end
end
```

###### 完整代码

```ruby
module CheckedAttributes
  def self.included(base)
    base.extend ClassMethods
  end
  
  module ClassMethods
    def attr_checked(attribute, &validation)
      define_method "#{attribute}=" do |value|
        raise 'Invalid attribute' unless validation.call(value)
        instance_variable_set("@#{attribute}", value)
      end
      
      define_method attribute do
        instance_variable_get "@#{attribute}"
      end
    end
  end
end
```


