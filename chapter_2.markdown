### 方法
##### 动态方法

* 动态派发：通过send()方法，想调用的方法可以成为一个参数，这样可以在代码运行期间，直到最后一才决定调用哪个方法。这种技术称为**动态派发**。

```ruby
# 动态派发
# Object#send()
class MyClass
  def my_method(my_arg)
    my_arg * 2
  end
end

obj = MyClass.new
obj.my_method(3) # => 6
```
* 动态方法：运行时定义方法的技术称为**动态方法**

```ruby
# 动态方法
class MyClass
  define_method :my_method do |my_arg|
    my_arg * 3
  end
end

obj = MyClass.new
obj.my_method(2) # => 6
```

```ruby
# 用动态方法重构书中例子

class Computer
  def initialize(computer_id, data_source)
    @id - computer_id
    @data_source = data_source
    
    # 如果给String#grep()方法传递一个块（block）,那么对每个满足正则表达式的元素，这个块都会被执行。
    # 其次，那些匹配括号中正则表达式字符会被存在全局变量$1中。
    # $1 中为字符串
    data_source.methods.grep(/^get_(.)_info$/) { Computer.define_component $1 }
  end
  
  # 此时Computer类是当前隐式的**self**，这意味着在Computer类上调用defin_componen()方法，因此这必然是一个类方法。
  def self.define_component(name)
    define_method(name) {
      info = @data_source.send "get_#{name}_info", @id
      price = @data_source.send "get_#{name}_price", @id
      # 因为$1中为字符串，所以不用转为string（之前define_component :key，所以需要Symbo#to_s()）。
      result = "#{name.capitalize}: #{info} ($#{price})"
      return "* #{result}" if price >= 100
      result
    }
  end
end
```


###### 实用方法

```ruby

String#to_sym() || String#intern() # 字符串转符号
Symbol#to_s || Symbol#id2name() # 符号转字符串

MyClass.instance_methods.delete_if { |method_name| method_name !~ /^test./ } # delete_if

```
