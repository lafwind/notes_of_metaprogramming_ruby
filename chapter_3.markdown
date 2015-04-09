### 第三章 代码块

##### 块（block）

* 块可以定义在大括号中（单行），也可以放在do ... end关键字中（多行）。

* **只有在调用一个方法时才可以定义一个块**，块会被直接传递给这个方法，然后该方法可以用**yield**这个关键字回调块

* 块可以有自己的参数，块中最后一行代码执行结果将会被作为返回值（和方法一样）

```ruby
def a_method(a, b)
  a + yield(a, b)
end
a_method(1, 2) { |x, y| (x + y) * 3 } # =>10
```

* Ruby可以通过Kernel#block_given?()方法来询问当前的方法调用是否包含块

``` ruby
def a_method
  return yield if block_given?
  'no block'
end

a_method                      # => "no block"
a_method {"here's a block!"}  # => "here's a block!"
```

* 如果在block_given?()返回false时使用了yield，会得到一个运行时错误。

* 绑定：即执行环境（局部变量、实力变量、self等）

* 块的要点在于它们是完整的，可以立即执行。它们既包含代码，也包含一组绑定。

* 当创建块时，会获取到局部绑定，把块连同它自己的绑定传给一个方法。

* 一个块可以获取局部绑定，并一直带着它们。

* 可以在块的内部定义额外的绑定，但这些绑定会在块结束时消失

* 块会覆盖具有相同名字的局变量（在ruby1.9中已被修正）

###### 作用域（scope）

* 如果一个对象调用**相同对象**中的另一个方法，那么实例变量在调用过程中始终存在于作用域中。

* 局部变量总是会掉出作用域（这就是它们被称为“局部”的原因）。

* 程序会在三个地方关闭前一个作用域，同时打开一个新的作用域（**作用域门**）：

  * 类定义（**class**关键字为标志）
  
  * 模块定义（**module**关键字为标志）
  
  * 方法（**def**关键字为标志）
  
* **class/module**和**def**之间还有点微妙区别：在类和模块定义中的代码**会被立即执行**；相反，方法定义中的代码**只有在方法被调用时被执行**

* 扁平作用域：闭包中的基本概念，用方法来代替作用域门，就可以让一个作用域看到另一个作用域中的变量，将两个作用域挤压在一起，共享各自的变量。

* 用`Class.new()`代替`class`，用`Module.new()`代替`module`，用`Module#define_method()`代替`def`，这就形成一个扁平作用域

```ruby
my_var = "Success"

MyClass = Class.new do
  puts "#{my_var} in the class definition!"
  
  define_method :my_method do
    puts "#{my_var} in the method!"
  end
end

MyClass.new.my_method
# => Success in the class definition!
     Success in the method!
```

* 在扁平作用域中定义多个方，则这些方法可以用一个作用域门进行保护，并共享绑定，这种技术称之为**共享作用域**

```ruby
def define_methods
  shared = 0
  
  Kernel.send :define_method, :counter do
    shared
  end
  
  Kernel.send :define_method, :inc do |x|
    shared += x
  end
end

defind_methods

counter # => 0
inc(4)
counter # => 4
```

###### instance_eval()

* 把传递给instance_eval()方法的块称为一个**上下文探针**，因为它就像一个深入到对象的代码片段，对其进行操作

```ruby
class MyClass
  def initialize
    @v = 1
  end
end

obj = MyClass.new
obj.instance_eval do
  # 运行时，该块的接收者会成为self，因此可以访问接收者的私有方法和实例变量
  self      # => <MyClass:0x123456 @v=1>
  @v        # => 1
end

# instance_eval()方法可以在不碰其他绑定的情况下修改self对象
v = 2
obj.instance_eval { @v = v }
obj.instance_eval { @v } # => 2

# instance_exec()功能和instance_eval()相似，但可传入参数
class C
  def initialize
    @x, @y = 1, 2
  end
end

C.new.instance_exec(3) { |arg| (@x + @y) * arg } # => 9
```

* instance_eval()会修改self，也会修改接收者的eigenclass（class_eval()会修改self和当前类），instance_eval()的标准含义是**我想修改self**

```ruby
s1, s2 = "abc", "def"

s1.instance_eval do
  def swoosh!; reverse; end
end

s1.swoosh!                # => "cba"
s2.respond_to?(:swoosh!)  # => false
```

* 洁净室：一个仅是为了在其中执行块的对象

```ruby
class CleanRoom
  def complex_calculation
    # ...
  end
  
  def do_something
    # ...
  end
end

clean_room = CleanRoom.new
clean_room.instance_eval do
  if complex_calculation > 10
    do_something
  end
end
```

###### 可调用对象

* Ruby中绝大多数东西都是对象，但是**块不是对象**。

* 一个Proc就是一个转换成对象的块，可以把块传给`Proc.new`方法来创建一个Proc。之后用`Proc#call`方法来执行这由块转换而来的对象（这种技术成为**延迟执行**）：

```ruby
inc = Proc.new { |x| x + 1 }
# more
inc.call(2) # => 3

# lambda
dec = lambda { |x| x - 1 }
dec.class # Proc
dec.call(2) # => 1
```

* 在方法的参数列表中添加**最后一个以&开头的参数**，就能把块附加到一个绑定上了（该参数上）。

* &操作符的意义是，这是一个Proc对象，我想把它当做一个块来使用；简单的去掉&，就能再次得到一个Proc对象。（以&符号开头的为块，去掉该符（&）就变对象）

* 通过`Proc.new()`方法、`Method#to_proc()`方法和**&**操作符可将一种可调用对象转换为另一种可调用对象：

```ruby
# 块转Proc
def my_method(&the_proc)
  the_proc
end

p = my_method { |name| "Hello, #{name}" } 
puts p.class            # => Proc
puts p.call("Lafwind")  # => Hello, Lafwind

# Proc转块

def my_method(greeting)
  puts "#{greeting}, #{yield}!"
end

my_proc = Proc.new { "Lafwind" }
my_method("Hello", &my_proc)

# => Hello, Lafwind!
```

* 使用lambda()创建的Proc称为lambda，而使用其他方式创建的则简单称为proc

* 可调用对象的区别：

  * 块：虽然不是真正的对象，但是是可调用的；在定义它们的作用域中执行；**`return`语句从定义可调用对象的原始上下文中返回**；对于传入参数数目处理宽松。
  
  * proc: Proc类的对象，跟块一样，也在定义自身的作用域中执行；**`return`语句从定义可调用对象的原始上下文中返回**；对于传入参数数目处理宽松。
  
  * lambda：也是Proc类对象，**和普通proc对象有些微区别**；它和块、proc一样都是闭包，因此也在定义自身的作用域中执行；**`return`语句从可调用对象中返回**；对于传入参数数目处理严格（与方法相比，某些极端情况下略为宽松）。
  
  * 方法：**绑定于对象，在对象所在的作用域中执行**；可以和某个作用域解绑定，然后再绑定到另一个对象的作用域上；对于传入参数数目处理最严格。

##### 其他

* 全局变量（$var）在系统的任何部分都可以修改它们，几乎无法追踪谁把它们改成了什么，所以，**如非必要，尽可能少使用全局变量**

* 顶级实例变（在任何方法，类，模块外的@var），它是顶级对象main的实例变量，只要在main对象扮演self时，就可以访问顶级实例变量，但其他对象成为self时，顶级实例变量就退出作用域。由于不想全局变量那么有全局性，一般认为顶级实例变量比全局变量更安全。

* 类变量（@@var），属于类体系结构。避免使用类变量，尽量使用类实例变量
