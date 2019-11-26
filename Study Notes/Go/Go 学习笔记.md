### Go 学习笔记

参考：[A Tour of Go](tour.golang.org)

#### Basics 

**Function**

- 支持无参数返回，直接返回已命名变量
- 支持多值返回

#### Flowcontrol

**For**

- 循环只有 for, while 可视为缺省的 for
- 死循环可以直接缺省写成 for { }

**If**

- 可以执行简单语句，如 `if v := math.Pow(x, n); v < lim`
- if 定义的变量作用域为整个 if-else 代码块

**Switch**

- 可执行简单语句，作用域为整个 switch 代码块
- 默认 case 之后执行 break, 如需往下执行需要以 fallthrough 结束
- case 无需常量，取值不必为整数
- 无条件 switch 看做 switch true, case 可加条件判断简化长串 if-then-else

**Defer**

- 立即求值，外层函数返回后被调用
- 被压在栈中，调用顺序为 LIFO

#### Moretypes

**Pointer**

- `var p *int` 定义指向 int 的指针 p
- `*p = 21 p` 为指向 21 的指针
- `p := &21` 定义一个指向 21 的指针 p

**Struct**

- 结构体指针允许隐式简介引用，如 (*p).X 可简写为 p.X
- 结构体文法，可通过直接列出字段分配结构体，可仅列出部分字段，如 `v := Vertex{X: 1}`

**Slice**

- `a[low : high]` 定义了一个 a 中从 low 到 high - 1 的切片
- 切片不存储数据，修改切片会同步修改数组中的元素
- `[]bool {true, true, false}` 本质上为创建一个数组再构建一个引用它的切片
- 切片的默认行为为 0 到长度，`a[0:10]、a[:10]、a[0:]、a[:]` 等价
- 切片长度为包含元素的个数，容量为切片头到底层数组尾的长度
- make 可以创建切片，`b := make([]int, 0, 5)` 表示长度为 0 容量为 5
- append 的时候，若底层数组长度不足，会按照 2^n 长度进行扩容

**Range**

- `for i, v := range pows` 中，i 为元素下标，v 为 pow[i] 的副本
- `for i, _ := range pows` 等价于 `for i := range pows`

**Map**

- 顶级类型若只是一个类型名，可省略，`map[string]Vertex{"BL":{1,2}} 等价于 map[string]{"BL":Vertex{1,2}}`
- 双赋值可检查是否存在，`e, ok = m[key]` key 不存在时返回零值

**Function**

- Go 语言可以将函数以值的方式进行传递
- 函数支持闭包：`func adder() func(int) int { ... }`

#### Methods

**Method**

- 可定义特殊接受者，比如 `func (v Vertx) Abs() float64 { ... }`

- 也可为非结构体声明，如

  ```go
  type MyFloat flaot64
  func (f MyFloat) Abs() float64{
      return float64(f)
  }
  ```

- 可为指针接收者声明方法，可直接修改接受者（值接收为修改副本）

- 指针参数的函数必须接收指针

- 以指针或值为接受者的方法被调用时方法接受者可以为值或指针（实质会被解释成 *p 或 &p）

**Interface**

- 接口可看做包含值和具体类型的元组 (value, type)，接口也是值

- 接口类型可保存任何实现了这些方法的值，传递时需遵守 接口/值 定义

  ```go
  type Abser interface{}
  type Vertex struct{}
  func (v *Vertex) Abs() float64{}
  func main(){
      var a Abser
      v := Vertex{3, 4}
      a = &v //不可使用 a = v 因为实现 Abs() 的是 *Vertex 不是 Vertex
      fmt.Print(a.Abs())
  }
  ```

- 底层值为 nil 时接口仍然会被调用，一般需要写判空来处理，nil 接口会调用出错

- 空接口 interface{} 可以保存任意类型值

**Assertion**

- 类型断言 i.(T) 提供了访问接口值底层具体值的方法，如 t, ok := i.(T)

- 类型选择 i.(type) 可以从类型断言中选择执行不同的分支

  ```go
  switch v := i.(type){
      case T: //do something
      case S: //do something
      default: //do something
  }
  ```

**Stringer**

- Stringer 接口的 String() 方法类似于 Java 的 toString()，用法为 `func (p Person) String() string { ... }`

  ```go
  type Stringer interface {
      String() string
  }
  ```

**Error**

- 类似 String(), 可以通过重写 Error 方法来打印错误，用法为 `func (e *MyError) Error() string { ... }`

  ```go
  type error interface{
      Error() string
  }
  ```

- 在 `func (e ErrNegativeSqrt) Error() string { ... }` 内调用 fmt.Sprint(e) 会无限递归

**Reader**

- Read 通过 io.EOF 错误表示遇到数据流结尾

  ```go
  func main(){
      r := strings.NewReader("Hello, Reader")
      b := make([]byte, 8)
      for {
          n, err := r.Read(b)
          fmt.Printf("b[:n] = %q\n", b[:n])
          if err == io.EOF {
              break;
          }
      }
  }
  ```

- 常见的一种模式是 io.Reader 包装另一个 io.Reader 之后通过某种方式修改其数据流

#### Concurrency

**Gorouting**

- `go f (x, y, z)` 中，f x y z 的求值发生在当前 Go 程，f 的执行发生在新的 Go 程

**Channel**

- Channel 为带有类型的管道，可通过 <- 传值

  ```go
  ch := make(chan int) // 创建信道 ch
  ch <- v // v 发送到 ch
  v ：= <- ch // 从 ch 接收值并赋予 v
  ```

- Channel 可设置带缓冲，`ch := make(chan int, 100)`, 缓冲区填满后，向其发送数据阻塞，缓冲区为空时，接收阻塞

- `v, ok := <- ch` 中 ok 判断信道是否被关闭了，一般信道无需关闭，除非告诉接受者不再有值

- `for i := range c` 会不断从信道接收值直到被关闭

- 只有发送者可关闭信道，接受者不可以，通过调用 close (c) 关闭

**Select**

- 阻塞直到某个分支可执行，若满足多个，则随机，语法为：

  ```go
  select{
      case c <- x: // 可以给 c 赋值时执行
      case <- quit: // 可以从 quit 取值时执行
  }
  ```

- 可使用 default 当条件都不满足时执行

**sync.Mutex**

- 提供了互斥锁这种数据结构，通过 Lock 和 Unlock 调用

- 可采用 defer 调用 Unlock 保证互斥锁一定会被解锁

  ```go
  type SafeCounter struct{
      v map[string]int
      mux sync.Mutex
  }
  
  func (c *SafeCounter) Inc(key string){
      c.mux.Lock()
      defer c.mux.Unlock()
      return c.v[key]
  }
  ```

  