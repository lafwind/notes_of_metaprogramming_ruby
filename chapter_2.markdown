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
obj.my_method(3)        # => 6

# send
obj.send(:my_method, 3) # => 6
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
    @id = computer_id
    @data_source = data_source
    
    # 如果给String#grep()方法传递一个块（block）,那么对每个满足正则表达式的元素，这个块都会被执行。
    # 其次，那些匹配括号中正则表达式字符会被存在全局变量$1中。
    # $1 中为字符串
    data_source.methods.grep(/^get_(.)_info$/) { Computer.define_component $1 }
  end
  
  # 此时Computer类是当前隐式的**self**，这意味着在Computer类上调用define_componen()方法，因此这必然是一个类方法。
  def self.define_component(name)
    define_method(name) {
      info = @data_source.send "get_#{name}_info", @id
      price = @data_source.send "get_#{name}_price", @id
      # 因为$1中为字符串，所以不用转为string（之前define_component :key，所以需要Symbol#to_s()）。
      result = "#{name.capitalize}: #{info} ($#{price})"
      return "* #{result}" if price >= 100
      result
    }
  end
end

# 使用
my_computer = Computer.new(36, DS.new)
my_computer.cpu # => * Cpu: 2.16Ghz ($220)
```

##### Object#method_missing()方法

* method_missing()是Kernel的一个实例方法，所有对象都继承自Kerne模块，所以任何对象都有一个method_missing()方法。

```ruby
# method_missing()是个私有方法，不过可以通过send()来调用
nick.send :method_missing, :my_method # => NoMethodError: undefined method...
```

* 每一个来到method_missing()办公桌上的消息都带着被调用方法的名字，以及所有调用时传递的参数和块。

```ruby
class Lawyer
  def method_missing(method, *args)
    puts "You called: #{method}(#{args.join(', ')})"
    puts "(You also passed it a block)" if block_given?
  end
end

bob = Lawyer.new
bob.talk_simple('a', 'b') do
  # a block
end

# => You called: talk_simple(a, b)
     (You also passed it a block)
```

* 覆写method_missing()方法使你可以调用实际上并不存在的方法。

* 幽灵方法（Ghost Methods）：被method_missing()方法处理的消息，从调用者角度看，和普通方法没什么区别，但实际上接收者并没有相应的方法。这被称为幽灵方法。

```ruby
class Table
  def method_missing(id, *args, &block)
    return as($1.to_sym, *args, &block) if id.to_s =~ /^to_(.*)/ # 调用as(:csv)
    return rows_with($1.to_sym => args[0]) if id.to_s =~ /^rows_with_(.*)/ # 调用rows_with(:country)
    super # 调用Kernel#method_missing()，抛出一个NoMethodError错误（本来的功用，这是super关键字所做的事情）
  end
  
  #...
end
```

* 一个捕获幽灵方法调用并把它们转发另一个对象的对象（有时也会在转发前包装一些自己的逻辑），称为**动态代理**。

* 白板类。在代理类中删除绝大多数继承来的方法。它所拥有的方法比Object类还要少。

* 可通过两种途径来删除一个方法：

  * Module#undef_method()方法，它删除所有的（包括继承来的）方法；
  
  * Module#remove_method()方法，它只会删除接收者自己的方法，而保留继承来的方法。

* 幽灵方法的一些缺点：

  * 命名冲突（白板类解决）
  
  * 比使用普通方法慢，因为在调用幽灵方法时，方法查找的路径一般要长一些。(可以在第一次调用幽灵方法，为它创建一个动态方法，这样以后的调用就可以直接调用这个动态方法)
  
  * 调用未定义的方法（不是我们所希望的去掉用method_missing()的方法）导致调用method_missing()。（尽可能清晰哪些方法是真的需要调用method_missing()，对于不属于其中的，应回到Kennel#method_missing()（super关键字）。）
  
  * 其他神秘bugs（使用迭代的方式开发，先用普通方法来实现功能，然后自信代码没问题后，把这些方法重构到method_missing()。）
  
* 一些Object类中的方法是被Ruby内部使用的，如果对它们重定义或删除Ruby可会诡异的崩掉。为了避免这种事情的发生，Ruby在这些方前面用下划线打头（*/_/_send/_/_*，*/_/_id/_/_*），并会你折腾它们时发出警告。

```ruby
# 用method_missting()来重构书中例子

class Computer
  # 白板，解决命名冲突
  instance_methods.each do |m|
    undef_method m unless m.to_s =~ /^__|method_missing|respond_to?/
  end
  
  def initialize(computer_id, data_source)
    @id = computer_id
    @data_source = data_source
  end
  
  def method_missing(name, *args)
    super if !@data_source.respond_to?("get_#{name}_info")
    info = @data_source.send("get_#{name}_info", args[0])
    price = @data_source.send("get_#{name}_price", args[0])
    result = "#{name.to_s.capitalize}: #{info} ($#{price})"
    return "* #{result}" if price >= 100
    result
  end
  
  # 解决对象无法响应幽灵方法的问题
  def respond_to?(method)
    @data_source.respond_to?("get_#{method}_info") || super
  end
end

# 使用
my_computer = Computer.new(36, DS.new)
my_computer.cpu # => * Cpu: 2.16Ghz ($220)

```

###### 实用方法

```ruby

String#to_sym() || String#intern() # 字符串转符号
Symbol#to_s || Symbol#id2name() # 符号转字符串

MyClass.instance_methods.delete_if { |method_name| method_name !~ /^test./ } # delete_if

puts “(You also passed it a block)” if block_given? # block_given?

# Module#const_missing()，Object#method_missing()

```
