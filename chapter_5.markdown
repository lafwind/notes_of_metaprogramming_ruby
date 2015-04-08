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
    * 


