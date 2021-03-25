## 建议

1.  检查某个对象是否完全实现了某个 interface:

```go
// 用于触发编译期的接口的合理性检查机制
// 如果Handler没有实现http.Handler,会在编译期报错
var _ http.Handler = (*Handler)(nil)
```

如果 `*Handler` 与 `http.Handler` 的接口不匹配, 那么语句 `var _ http.Handler = (*Handler)(nil)` 将无法编译通过.

赋值的右边应该是断言类型的零值。 对于指针类型（如 `*Handler`）、切片和映射，这是 `nil`； 对于结构类型，这是空结构。

```go
type LogHandler struct {
  h   http.Handler
  log *zap.Logger
}
var _ http.Handler = LogHandler{}
func (h LogHandler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

2.   零值 Mutex 是有效的

```go
// Good
var mu sync.Mutex
mu.Lock()

// Bad
mu := new(sync.Mutex)
mu.Lock()
```

如果你使用结构体指针，mutex 可以非指针形式作为结构体的组成字段，或者更好的方式是直接嵌入到结构体中。 如果是私有结构体类型或是要实现 Mutex 接口的类型，我们可以使用嵌入 mutex 的方法：

```go
// 非导出结构体，直接内嵌
type smap struct {
  sync.Mutex // only for unexported types（仅适用于非导出类型）

  data map[string]string
}

func newSMap() *smap {
  return &smap{
    data: make(map[string]string),
  }
}

func (m *smap) Get(k string) string {
  m.Lock()
  defer m.Unlock()

  return m.data[k]
}

// 可导出结构，使用私有变量 
type SMap struct {
  mu sync.Mutex // 对于导出类型，请使用私有锁

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```

3.  slices 和 maps 包含了指向底层数据的指针，因此在需要复制它们时要特别注意。

```go
// --> Bad
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// 你是要修改 d1.trips 吗？
trips[0] = ...

// --> Good
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// 这里我们修改 trips[0]，但不会影响到 d1.trips
trips[0] = ...
```

同样的场景，对于 map：

```go
// --> Bad
type Stats struct {
  mu sync.Mutex

  counters map[string]int
}

// Snapshot 返回当前状态。
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// snapshot 不再受互斥锁保护
// 因此对 snapshot 的任何访问都将受到数据竞争的影响
// 影响 stats.counters
snapshot := stats.Snapshot()

// --> Good
type Stats struct {
  mu sync.Mutex

  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// snapshot 现在是一个拷贝
snapshot := stats.Snapshot()
```

4.  枚举值应该从 1 开始，除非 0 是有意义的：

```go
// Bad
type Operation int

const (
  Add Operation = iota
  Subtract
  Multiply
)
// Add=0, Subtract=1, Multiply=2

// Good
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

再如以下场景中，0 值是默认值，是有意义的，是理想的默认行为：

```go
type LogOutput int

const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)

// LogToStdout=0, LogToFile=1, LogToRemote=2
```

5.  使用 `package time` 处理所有和时间有关的操作

时间前后：

```go
// Bad 
func isActive(now, start, stop int) bool {
  return start <= now && now < stop
}

// Good
func isActive(now, start, stop time.Time) bool {
  return (start.Before(now) || start.Equal(now)) && now.Before(stop)
}
```

使用 `Duration` 表达时间段：

```go
// Bad
func wait(delay int) {
  for {
    // ...
    time.Sleep(time.Duration(delay) * time.Millisecond)
  }
}
wait(10) // 是几秒钟还是几毫秒?

// Good
func wait(delay time.Duration) {
  for {
    // ...
    time.Sleep(delay)
  }
}
wait(10*time.Second)
```

时间加减：对于一个时间加上 24 小时，一般有两个场景：1. 下一个日历日的同一个时间点，这里请使用 `time.AddDate(0 /* years */, 0, /* months */, 1 /* days */)`，2. 仅仅是想保证某个时刻比前一个时刻刚好晚 24 小时，请使用 `time.Add(24 * time.Hour)`

```go
newDay := t.AddDate(0 /* years */, 0, /* months */, 1 /* days */)
maybeNewDay := t.Add(24 * time.Hour)
```

6.  错误处理

返回错误信息时，通常有以下的场景：

-   如果这个错误不需要额外的信息，用 `errors.New("xxx")` 就好， 比如 `redis.Nil` 表示 redis 的 `nil` 错误
-   如果调用者需要检测错误类型，应该使用自定义的错误并且实现 `Error()` 接口
-   如果在调用上下游传递错误，应该使用 `["pkg/errors".Wrap] 的 Wrapped errors`，调用者可以通过 `Cause(err)` 拿到最开始的错误
-   一般情况 `fmt.Errorf()` 就足够了

如果客户端需要检测错误，并且你已经自定义了一个新的错误变量，请使用错误变量：当然你也可以使用 `pkg/errors.Wrap` 和 `pkg/errors.Cause()`。

```go
// ---> Bad
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

func use() {
  if err := foo.Open(); err != nil {
    if err.Error() == "could not open" {
      // handle
    } else {
      panic("unknown error")
    }
  }
}

// -->Good
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if err == foo.ErrCouldNotOpen {
    // handle
  } else {
    panic("unknown error")
  }
}
```

如果想给调用者更多的提示信息，应该使用自定义的错误类，而不是简单的字符串。

在将上下文添加到返回的错误时，请避免使用“failed to”之类的短语以保持上下文简洁，这些短语会陈述明显的内容，并随着错误在堆栈中的渗透而逐渐堆积。

但是，一旦将错误发送到另一个系统，就应该明确消息是错误消息（例如使用`err`标记，或在日志中以”Failed”为前缀）。

7.  类型断言总是使用 `comma ok` 的格式

```go
// Bad
t := i.(string)

// Good
t, ok := i.(string)
if !ok {
  // 优雅地处理错误
}
```

8.  除非在初始化非常重要的全局变量做到“快速失败”的目的，一般场景别用 panic，使用返回错误代替

9.  注意使用 atomic 来执行原子操作，减少数据竞争。(使用 uber 的[包](https://github.com/uber-go/atomic)？)
10.  避免在可导出结构中嵌入另一类型：

相反，只需手动将方法写入具体的列表，该列表将委托给抽象列表。假设已经有一个结构体：

```go
type AbstractList struct {}
// 添加将实体添加到列表中。
func (l *AbstractList) Add(e Entity) {
  // ...
}
// 移除从列表中移除实体。
func (l *AbstractList) Remove(e Entity) {
  // ...
}
```

如果我想使用这个 AbstractList ，可能的场景如下：

```go
// Bad
type ConcreteList struct {
  *AbstractList
}

// Good
type ConcreteList struct {
  list *AbstractList
}
// 添加将实体添加到列表中。
func (l *ConcreteList) Add(e Entity) {
  l.list.Add(e)
}
// 移除从列表中移除实体。
func (l *ConcreteList) Remove(e Entity) {
  l.list.Remove(e)
}
```

11.  避免使用 `init()` 
12.  如果程序包名称与导入路径的最后一个元素不匹配，则必须使用导入别名。

```go
import (
  "net/http"

  client "example.com/client-go"
  trace "example.com/trace/v2"
)
```

13.  文件中的对象放置顺序

-   函数应按粗略的调用顺序排序。

-   同一文件中的函数应按接收者分组。

因此，导出的函数应先出现在文件中，放在`struct`, `const`, `var`定义的后面。

在定义类型之后，但在接收者的其余方法之前，可能会出现一个 `newXYZ()`/`NewXYZ()`

由于函数是按接收者分组的，因此普通工具函数应在文件末尾出现。

14.  字符串 string format

如果你在函数外声明`Printf`-style 函数的格式字符串，请将其设置为`const`常量。

这有助于`go vet`对格式字符串执行静态分析。

```go
// Bad 
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)

// Good
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```



## 性能

1.  make 时，尽可能指定容量



## 一些问题

https://github.com/aceld/golang/





[Uber Go 编码规范](https://github.com/xxjwxc/uber_go_guide_cn#%E5%8A%9F%E8%83%BD%E9%80%89%E9%A1%B9)

![image-20210210171939103](/Users/hujiaming/Library/Application%20Support/typora-user-images/image-20210210171939103.png)