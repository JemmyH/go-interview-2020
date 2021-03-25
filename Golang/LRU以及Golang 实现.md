https://github.com/hashicorp/golang-lru

## 一、LRU简介

`LRU` 全称为 `Least Recented Used`，**最近最少使用**，意思是 **“淘汰掉最不经常使用的”**，常用于页面置换算法。在计算机组成原理中，一方面，有著名的 **时间局部性**原理：**如果一个数据被访问了，那么将来被再次访问的概率更大**。另一方面，硬盘容量最大，但访问速度很慢；内存速度很快，但容量有限；为了进一步提升数据访问的性能，在 CPU 内部还有 `一级缓存`、`二级缓存`等。在这其中，速度越快，单位成本就越高，容量就越小；新的内容不断被载入，旧的内容肯定要被淘汰。这就是 `LRU` 最常见的使用场景。

下面我们通过一个 demo 来解释一下 LRU 的工作原理：

假设我们的缓存容量只有 3，我们按照 7 0 1 2 0 3 0 4 的次序去访问缓存。假设在最上面的是 **最近** 访问的，那么 LRU 的工作流程大致是这样的：



![](https://geektutu.com/post/geecache-day1/lru.jpg)

这张图很好地表示了 LRU 算法最核心的 2 个数据结构

-   蓝色的是字典(map)，存储键和值的映射关系。这样根据某个键(key)查找对应的值(value)的复杂是`O(1)`，在字典中插入一条记录的复杂度也是`O(1)`。
-   红色的是双向链表(double linked list)实现的队列。将所有的值放到双向链表中，这样，当访问到某个值时，将其移动到队尾的复杂度是`O(1)`，在队尾新增一条记录以及删除一条记录的复杂度均为`O(1)`。

## 二、Golang 实现

首先，我们需要一个 **双向链表(`doublelist`)**，在  `go` 的标准库中已经有一个现成的实现，我对其加了注释和几个打印方法以方便调试：

```go
package doublelist
/*
  @File    :   doublelist.go
  @Time    :   2020/11/27 10:37:53
  @Author  :   Jemmy(hujm20151021@gmail.com)
  @Desc    :   package doublelist 是一个双向链表的实现
*/

import "fmt"

// Element 表示一个双向链表的结点
type Element struct {
	next, prev *Element    // 后继节点和前驱结点
	list       *List       // 当前节点属于哪个链表
	Value      interface{} // 结点的值
}

// Next 返回结点 e 的后继节点
func (e *Element) Next() *Element {
	if p := e.next; e.list != nil && p != &e.list.root {
		return p
	}
	return nil
}

// Prev 返回 e 的前驱结点
func (e *Element) Prev() *Element {
	if p := e.prev; e.list != nil && &e.list.root != p {
		return p
	}
	return nil
}

// -------------------------------------------------------------------------------------

// List 表示一个双向链表
type List struct {
	root Element // 哨兵结点(头结点)，本身不存储值。在本次实现中，链表中头结点和尾结点共同指向此 root，你可以把它当成一个环
	len  int     // 链表中元素的个数(不包含头结点)
}

// New 返回一个初始化后的双向链表
func New() *List {
	return new(List).Init()
}

// Init 初始化一个双向链表(清空所有元素)
func (l *List) Init() *List {
	l.root.next = &l.root
	l.root.prev = &l.root
	l.len = 0
	return l
}

// Len 返回当前双向链表中元素个数
func (l *List) Len() int {
	return l.len
}

// Front 返回双链表中第一个元素
func (l *List) Front() *Element {
	if l.len == 0 {
		return nil
	}
	return l.root.next
}

// Back 返回双链表的最后一个元素
func (l *List) Back() *Element {
	if l.len == 0 {
		return nil
	}
	return l.root.prev
}

// Remove 将 e 从 l 中移除(前提是 e 属于 l)
func (l *List) Remove(e *Element) interface{} {
	if e == nil || l == nil {
		return nil
	}
	if e.list == l {
		l.remove(e)
	}
	return e.Value
}

// PushFront 将以 v 作为 Value 的 Element 插入到 l 的头部(作为第一个元素)
func (l *List) PushFront(v interface{}) *Element {
	l.checkAndInit()
	return l.insertValue(v, &l.root)
}

// PushBack 将以 v 作为 Value 的 Element 插入到 l 的尾部(作为最后一个元素)
func (l *List) PushBack(v interface{}) *Element {
	l.checkAndInit()
	return l.insertValue(v, l.root.prev)
}

// InsertBefore 将以 v 作为 Value 的结点插入到 at 前面
func (l *List) InsertBefore(v interface{}, at *Element) *Element {
	if at.list == nil || at.list != l {
		return nil
	}
	return l.insertValue(v, at.prev)
}

// // InsertBefore 将以 v 作为 Value 的结点插入到 at 后面
func (l *List) InsertAfter(v interface{}, at *Element) *Element {
	if at.list == nil || at.list != l {
		return nil
	}
	return l.insertValue(v, at)
}

// MoveToFront 将 e 移动到 l 的头部(作为第一个结点)
func (l *List) MoveToFront(e *Element) {
	if e.list != l || l.root.next == e {
		return
	}
	l.move(e, &l.root)
}

// MoveToBack 将 e 移动到 l 的尾部(作为最后一个结点)
func (l *List) MoveToBack(e *Element) {
	if e.list != l || l.root.prev == e {
		return
	}
	l.move(e, l.root.prev)
}

// AppendToFront 将 list 整个插入到 l 的第一个元素前面
func (l *List) AppendToFront(list *List) {
	l.checkAndInit()
	for i, e := list.Len(), list.Back(); i > 0; i, e = i-1, e.Prev() {
		l.insertValue(e.Value, &l.root)
	}
}

// AppendToBack 将 list 整个插入到 l 的最后一个元素后面
func (l *List) AppendToBack(list *List) {
	l.checkAndInit()
	for i, e := list.Len(), list.Front(); i > 0; i, e = i-1, e.Next() {
		l.insertValue(e.Value, l.root.prev)
	}
}

// ListElementsAsc 从前到后返回双链表所有元素的 Value
func (l *List) ListElementsAsc() (res []interface{}) {
	res = make([]interface{}, 0, l.len)
	if l.len == 0 {
		return
	}
	for i, e := l.len, l.root.next; i > 0; i, e = i-1, e.next {
		res = append(res, e.Value)
	}
	return
}

// ListElementsDesc 从后到前返回双链表所有元素的 Value
func (l *List) ListElementsDesc() (res []interface{}) {
	res = make([]interface{}, 0, l.len)
	if l.len == 0 {
		return
	}
	for i, e := l.len, l.root.prev; i > 0; i, e = i-1, e.prev {
		res = append(res, e.Value)
	}
	return
}

// PrintAllAsc 从前到后打印双链表所有元素
func (l *List) PrintAllAsc() {
	if l.len == 0 {
		return
	}
	for i, e := l.len, l.root.next; i > 0; i, e = i-1, e.next {
		fmt.Printf("%v\t", e.Value)
	}
	fmt.Println()
}

// PrintAllDesc 从后到前打印双链表的所有元素
func (l *List) PrintAllDesc() {
	if l.len == 0 {
		return
	}
	for i, e := l.len, l.root.prev; i > 0; i, e = i-1, e.prev {
		fmt.Printf("%v\t", e.Value)
	}
	fmt.Println()
}

// ***************************** internal method ******************

// checkAndInit 检查 root 的next 是否为 nil，如果是 nil，后续的操作将引起 crash
func (l *List) checkAndInit() {
	if l.root.next == nil {
		l.Init()
	}
}

// remove 将 e 从双链表中删除，同时更新链表个数，成功后返回 e
func (l *List) remove(e *Element) *Element {
	e.prev.next = e.next
	e.next.prev = e.prev
	l.len--

	// 防止内存泄漏
	e.next = nil
	e.prev = nil
	e.list = nil

	return e
}

// insert 将 e 作为 at 的后继结点插入到双链表中，同时更新链表个数，成功后返回 e
func (l *List) insert(e, at *Element) *Element {
	n := at.next
	at.next = e
	e.next = n
	e.prev = at
	n.prev = e
	e.list = l
	l.len++
	return e
}

// insertValue 是 insert 方法的便捷表示
func (l *List) insertValue(v interface{}, at *Element) *Element {
	return l.insert(&Element{Value: v}, at)
}

// move 将 e 移动为 at 的后继，成功后返回 e
func (l *List) move(e, at *Element) *Element {
	if e == at {
		return e
	}
	// remove e
	e.prev.next = e.next
	e.next.prev = e.prev

	// append e to at
	n := at.next
	at.next = e
	e.next = n
	e.prev = at
	n.prev = e

	return e
}
```

其次，因为 `LRU` 是一个功能性描述，实现方法有很多种，最好定义一个 `interface{}` ：

```go
package simplelru

/*
* @CreateTime: 2020/11/27 20:40
* @Author: hujiaming
* @Description:
名词解释：
	- eviction 淘汰
*/

// LRUCache 表示一个 LRU 应该包含的功能
type LRUCache interface {
	// Add 向 cache 中增加一个元素，如果发生淘汰会返回 true
	Add(key, value interface{}) bool

	// Get 返回 key 在 cache 中的 value(会更新 key 的使用频率)
	Get(key interface{}) (value interface{}, ok bool)

	// Contains 检查 key 是否存在于缓存之中(这个操作不会更新 key 的使用频率)
	Contains(key interface{}) (ok bool)

	// Peek 返回 key 对应的 value(不更新 key 的更新频率)
	Peek(key interface{}) (value interface{}, ok bool)

	// Remove 将 key 从 cache 中删除
	Remove(key interface{}) bool

	// RemoveOldest 删除使用频率最小的元素，如果删除成功，会返回这个 key value true，否则返回 nil,nil,false
	RemoveOldest() (interface{}, interface{}, bool)

	// GetOldest 获取使用频率最小的元素
	GetOldest() (interface{}, interface{}, bool)

	// Keys 按照使用频率从小到大的顺序，返回所有的元素
	Keys() []interface{}

	// Len 返回 cache 中的元素个数
	Len() int

	// Purge 清空缓存
	Purge()

	// Resize 更新缓存大小，返回被淘汰的元素个数
	Resize(int) int
}
```

之后，我们定义一个结构体来实现它。

在上述对 `type LRUCache interface` 的实现中，需要关注以下几点：

1.  我们需要一个 map，用于快速定位某个值在双端链表中的位置，所以 `key` 就是元素的值， `value` 就是对应的 `Element`。
2.  对于淘汰规则：**我们规定，越靠近双端链表的头部位置，越最近、最多使用**。所以新增的元素，是通过**头插入法**插入到双向链表的；当一个元素被访问时，意味着其被访问频率增加，此时需要将其移到双向链表头部；当我们插入一个元素时，如果双向链表的容量不够，需要淘汰 **最远、访问频率最小** 的元素，即双向链表的尾部元素，再将新插入的元素放到双向链表首部。
3.  在某些情况下，我们需要对被淘汰的元素进行 `callback` 处理，可以通过回调函数实现。 

```go
package simplelru

import (
	"errors"

	"hujm.net/lru_demo/doublelist"
)

/*
* @CreateTime: 2020/11/27 18:43
* @Author: hujiaming
* @Description: LRU 内部实现

- 新增时(没有元素存在)
	- 当 LRU 容量够时，直接将新增元素放在双向链表头部
	- 当 LRU 容量不够时，先移除链表尾部的元素，并将新增元素插入头部
- 新增时(已经存在)
	- 将该元素移动到链表头部
- 访问时
	- 有一个元素被访问，则需要将这个元素移动到双端链表头部

*/

type PopCallback func(key interface{}, value interface{})

// LRU 是一个非并发安全的、指定容量的 lru 实现(并发安全在更高层控制)
type LRU struct {
	size        int                                 // 容量
	elements    *doublelist.List                    // 存放元素的双向链表 double_list
	items       map[interface{}]*doublelist.Element // 存放某个元素的值在双向链表中的位置的 map
	popCallback PopCallback                         // 当某个元素弹出时的回调函数
}

// NewLRU 根据 size 和 onPop 返回一个 LRU 对象
func NewLRU(size int, onPop PopCallback) (*LRU, error) {
	if size <= 0 {
		return nil, errors.New("invalid size")
	}
	return &LRU{
		size:        size,
		elements:    doublelist.New(),
		items:       make(map[interface{}]*doublelist.Element),
		popCallback: onPop,
	}, nil
}

// Add 向 cache 中增加一个元素，如果发生淘汰会返回 true
func (l *LRU) Add(key, value interface{}) (overflowed bool) {
	// if exists
	if ent, ok := l.items[key]; ok {
		l.elements.MoveToFront(ent)      // 将此元素移到双向链表的头部
		ent.Value.(*entry).value = value // 将对应的 key 的 value 更新为传入的 value
		return false                     // 没有元素被弹出
	}

	// 添加一个新的元素到 LRU 中
	ent := &entry{key: key, value: value}
	inserted := l.elements.PushFront(ent) // 将新元素插入到链表头部
	l.items[key] = inserted               // 更新 map

	// 容量是否溢出
	hasOverflow := l.elements.Len() > l.size
	if hasOverflow {
		// 如果插入后超过容量，需要移除被访问频率最小的那个：队尾的元素(最后一个元素)
        l.removeOldest()
	}
	return hasOverflow
}

// Get 返回 key 在 cache 中的 value(会更新 key 的使用频率)
func (l *LRU) Get(key interface{}) (value interface{}, ok bool) {
	if ent, ok := l.items[key]; ok {
		l.elements.MoveToFront(ent) // 更新频率
		if ent.Value.(*entry) == nil {
			return nil, false
		}
		return ent.Value.(*entry), true
	}
	return nil, false
}

// Contains 检查 key 是否存在于缓存之中(这个操作不会更新 key 的使用频率)
func (l *LRU) Contains(key interface{}) (ok bool) {
	_, ok = l.items[key]
	return ok
}

// Peek 返回 key 对应的 value(不更新 key 的更新频率)
func (l *LRU) Peek(key interface{}) (value interface{}, ok bool) {
	if ent, ok := l.items[key]; ok {
		return ent.Value.(*entry).value, true
	}
	return nil, false
}

// Remove 将 key 从 cache 中删除
func (l *LRU) Remove(key interface{}) bool {
	if ent, ok := l.items[key]; ok {
		l.elements.Remove(ent)
		return true
	}
	return false
}

// RemoveOldest 删除使用频率最小的元素，如果删除成功，会返回这个 key value true，否则返回 nil,nil,false
func (l *LRU) RemoveOldest() (interface{}, interface{}, bool) {
	ent := l.elements.Back()
	if ent != nil {
		l.removeElement(ent)
		kv := ent.Value.(*entry)
		return kv.key, kv.value, true
	}
	return nil, nil, false
}

// // GetOldest 获取使用频率最小的元素
func (l *LRU) GetOldest() (interface{}, interface{}, bool) {
	ent := l.elements.Back()
	if ent != nil {
		kv := ent.Value.(*entry)
		return kv.key, kv.value, true
	}
	return nil, nil, false
}

// Keys 按照使用频率从小到大的顺序，返回所有的元素
func (l *LRU) Keys() []interface{} {
	keys := make([]interface{}, len(l.items))
	cnt := 0
	for ent := l.elements.Back(); ent != nil; ent = ent.Prev() {
		keys[cnt] = ent.Value.(*entry).key
		cnt++
	}
	return keys
}

// Len 返回 cache 中的元素个数
func (l *LRU) Len() int {
	return l.elements.Len()
}

// Purge 清空整个 LRU
func (l *LRU) Purge() {
	// clean map
	for k, v := range l.items {
		if l.popCallback != nil {
			l.popCallback(k, v)
		}
		delete(l.items, k)
	}
	// clean double list
	l.elements.Init()
}

// Resize 更新缓存大小，返回被淘汰的元素个数
func (l *LRU) Resize(size int) int {
	diff := l.Len() - size
	l.size = size
	if diff < 0 {
		return 0
	}
	for i := 0; i < diff; i++ {
		l.removeOldest()
	}
	return diff
}

// ******************* internal methods ********************

// removeOldest 淘汰掉访问频率最小的元素
func (l *LRU) removeOldest() {
	entry := l.elements.Back()
	if entry != nil {
		l.removeElement(entry)
	}
}

// removeElement 将 e 从 LRU 中移除
func (l *LRU) removeElement(e *doublelist.Element) {
	l.elements.Remove(e)

}

// entry 表示双端链表中的一个元素
type entry struct {
	key   interface{}
	value interface{}
}
```

在上面的 `LRU struct` 中，有一个双向链表和一个 map，对于元素的操作，并不是线程安全的，为了线程安全，我们可以在 `LRU struct` 的基础上再封装一层：

```go
package lru

import (
	"sync"

	"hujm.net/lru_demo/simplelru"
)

/*
* @CreateTime: 2020/11/27 21:13
* @Author: hujiaming(hujm20151021@gmail.com)
* @Description: LRU 实现
 */

// Cache 表示一个 LRU
type Cache struct {
	lru  simplelru.LRUCache
	lock sync.RWMutex
}

// New 返回一个容量为 size 的 LRU 实例
func New(size int) (*Cache, error) {
	return NewWithPopCallback(size, nil)
}

// NewWithPopCallback 返回一个容量为 size、带有处理淘汰元素函数的 LRU 实例
func NewWithPopCallback(size int, onPop func(key interface{}, value interface{})) (*Cache, error) {
	lru, err := simplelru.NewLRU(size, onPop)
	if err != nil {
		return nil, err
	}
	return &Cache{lru: lru}, nil
}

// Add 向 cache 中增加一个元素，如果发生淘汰会返回 true
func (c *Cache) Add(key, value interface{}) bool {
	c.lock.Lock()
	popped := c.lru.Add(key, value)
	c.lock.Unlock()
	return popped
}

// Get 返回 key 在 cache 中的 value(会更新 key 的使用频率)
func (c *Cache) Get(key interface{}) (value interface{}, ok bool) {
	c.lock.Lock()
	value, ok = c.lru.Get(key)
	c.lock.Unlock()
	return value, ok
}

// Contains 检查 key 是否存在于缓存之中(这个操作不会更新 key 的使用频率)
func (c *Cache) Contains(key interface{}) (ok bool) {
	c.lock.RLock()
	ok = c.lru.Contains(key)
	c.lock.RUnlock()
	return ok
}

// Peek 返回 key 对应的 value(不更新 key 的更新频率)
func (c *Cache) Peek(key interface{}) (value interface{}, ok bool) {
	c.lock.RLock()
	value, ok = c.lru.Peek(key)
	c.lock.RUnlock()
	return value, ok
}

// Remove 将 key 从 cache 中删除
func (c *Cache) Remove(key interface{}) bool {
	c.lock.Lock()
	ok := c.lru.Remove(key)
	c.lock.Unlock()
	return ok
}

// RemoveOldest 删除使用频率最小的元素，如果删除成功，会返回这个 key value true，否则返回 nil,nil,false
func (c *Cache) RemoveOldest() (interface{}, interface{}, bool) {
	c.lock.Lock()
	key, value, ok := c.lru.RemoveOldest()
	c.lock.Unlock()
	return key, value, ok
}

// GetOldest 获取使用频率最小的元素
func (c *Cache) GetOldest() (interface{}, interface{}, bool) {
	c.lock.Lock()
	key, value, ok := c.lru.GetOldest()
	c.lock.Unlock()
	return key, value, ok
}

// Keys 按照使用频率从小到大的顺序，返回所有的元素
func (c *Cache) Keys() []interface{} {
	c.lock.RLock()
	keys := c.lru.Keys()
	c.lock.RUnlock()
	return keys
}

// Len 返回 cache 中的元素个数
func (c *Cache) Len() int {
	c.lock.RLock()
	length := c.lru.Len()
	c.lock.RUnlock()
	return length
}

// Purge 清空缓存
func (c *Cache) Purge() {
	c.lock.Lock()
	c.lru.Purge()
	c.lock.Unlock()
}

// Resize 更新缓存大小，返回被淘汰的元素个数
func (c *Cache) Resize(size int) int {
	c.lock.Lock()
	popped := c.lru.Resize(size)
	c.lock.Unlock()
	return popped
}

// ContainsOrAnd 先在不更新频率的条件下检查 key 是否存在于 cache 中
// 如果不存在，将这个 key-value 加入 cache，添加过程中如果有元素被淘汰，则 popped 为 true
func (c *Cache) ContainsOrAnd(key, value interface{}) (ok, popped bool) {
	c.lock.Lock()
	defer c.lock.Unlock()

	if c.lru.Contains(key) {
		return true, false
	}
	popped = c.lru.Add(key, value)
	return false, popped
}

// PeekOrAnd 先在不更新频率的条件下获取 key 对应的 value，如果找到，返回对应值，同时 ok 为 true，popped 为 false
// 如果没找到， 将 key-value 加入缓存，previous 为 nil，ok 为 false，如果加入的过程中有元素被淘汰，popped 为 true
func (c *Cache) PeekOrAnd(key, value interface{}) (previous interface{}, ok, popped bool) {
	c.lock.Lock()
	defer c.lock.Unlock()

	previous, ok = c.lru.Peek(key)
	if ok {
		return previous, true, false
	}

	popped = c.lru.Add(key, value)
	return nil, false, popped
}

```

**参考资料**：

-   [https://github.com/hashicorp/golang-lru](https://github.com/hashicorp/golang-lru)
-   [go 标准库中双向链表](https://github.com/golang/go/blob/master/src/container/list/list.go)
-   [https://geektutu.com/post/geecache-day2.html](https://geektutu.com/post/geecache-day2.html)

