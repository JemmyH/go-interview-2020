## 一、简介

[默克尔树](https://en.wikipedia.org/wiki/Merkle_tree)是一种典型的二叉树结构，由**一个根节点**、**一组中间节点** 和 **一组叶节点** 组成。默克尔树最早由 `Merkle Ralf `在 1980 年提出，曾广泛用于 **文件系统** 和 **P2P** 系统中，比如 `Git`、区块链、`IPFS` 等大名鼎鼎的项目或技术。

他又被称为 **哈希树**，即存储哈希值的树。树的叶子结点是 **数据块**(文件或者对象)的哈希值，而非叶子结点保存的是其子节点连接起来后的哈希值。简单来说，它有以下特点：

-   最下面的叶节点包含存储数据或其哈希值。
-   非叶子节点（包括中间节点和根节点）都是它的两个孩子节点内容的哈希值。
-   如果是奇数个叶子结点，那么其父节点保存的哈希值就是它本身或者复制一份自己凑成对再进行哈希的结果(具体实现取决于实际情况)

当然，默克尔树可以推广到多叉树的情形，此时非叶子节点的内容为它所有的孩子节点的内容的哈希值。

## 二、原理与用途

最开始，我们有一组已经准备好的数据块(比如文件)，他们根据某个标准有序(比如根据文件名字典排序)，而每一个文件都有唯一的哈希值与之对应。这是最底层的情况，当我们向上走的时候，每两个当前层的节点(左右孩子结点)的哈希值可以重新组合，形成一个新的节点(父节点)，这个新的结点中不存储数据，其哈希值为左右孩子结点组合后再次使用预设的哈希函数求哈希值。如此以往，直到生成树根，这个树根我们称为  `Merkel Root`。有一个特殊情况需要注意，有可能某一层的节点数是奇数，这样就会剩下最后一个结点，再没有结点与其组队生成父节点，这种情况下有两种解决方案：一种是复制一份自己；另一种是不复制，让其父节点只有它一个子节点，而且是左孩子结点。

目前，默克尔树的典型应用场景包括如下几种。

### 快速比较大量数据

对每组数据排序后构建默克尔树结构。当两个默克尔树根相同时，则意味着所代表的两组数据必然相同。否则，必然不同。

由于 Hash 计算的过程可以十分快速，预处理可以在短时间内完成。利用默克尔树结构能带来巨大的比较性能优势。

### 快速定位修改

![默克尔树](https://pic.downk.cc/item/5f9104531cd1bbb86ba7e0ac.png)

假如我们基于文件 `D0~D3` 构建如上的默克尔树，如果 `D2` 被修改，那么会影响到结点 `N2`、`N2` 和 `Root`。此时我们可根据发生变化的节点，沿着  `Root -> N5 -> N2`， 通过 `O(logN)` 的时间复杂度快速定位到哪个结点发生了变化。

### 零知识证明

它指的是证明者能够在不向验证者提供任何有用的信息的情况下(没有泄露信息)，使验证者相信某个论断是正确的。有一个很简单的例子：A要向B证明自己拥有某个房间的钥匙，假设该房间只能用钥匙打开锁，而其他任何方法都打不开。这时有2个方法：

1.  A把钥匙出示给B，B用这把钥匙打开该房间的锁，从而证明A拥有该房间的正确的钥匙。

2.  B确定该房间内有某一物体，A用自己拥有的钥匙打开该房间的门，然后把物体拿出来出示给B，从而证明自己确实拥有该房间的钥匙。

后面的第二种方法属于零知识证明。它的好处在于，在整个证明的过程中，B始终不能看到钥匙的样子，从而避免了钥匙的泄露。

在默克尔树中，我们仍旧以上图为例，如何向他人证明我拥有 `D0` 这个数据，而不用暴露更多系统的信息呢？模仿上面的例子，验证者随机提供数据 `D1`、`D2` 和 `D3`，证明者构造如图的默克尔树，并公布 `N1` 、`N5` 和 `Root`。验证者自行计算 `Root` 值，看是否一致，从而检验 `D0 ` 是否存在，因为如果存在，`N0` 一定相同，那么 `N4(N0-N1)` 也一定相同、`Root(N4-N5)`也一定相同。整个过程中验证着没有得到任何除了 `D0` 外的敏感信息(其他的 `D`)。

## 三、Golang 实现

首先，我们定义需要的结构：

```go
// Node 表示默克尔树中的 叶结点、非叶结点 或者 Root
type Node struct {
	Tree   *MerkleTree // 所在的 Merkle Tree
	Parent *Node       // 父节点
	Left   *Node       // 左孩子
	Right  *Node       // 右孩子
	leaf   bool        // 是否叶子结点
	Hash   []byte      // 如果是叶子结点，则为叶子结点数据的哈希值；如果是非叶子结点，则为左右孩子哈希值组合后的哈希值
	C      Content     // 叶子结点存储的数据块
}

// Content 代表一个数据块
type Content interface {
	CalculateHash() ([]byte, error)
	Equals(other Content) (bool, error)
}

// MerkleTree 默克尔树
type MerkleTree struct {
	Root         *Node            // Merkle Root 树根
	merkleRoot   []byte           // 树根的哈希值
	Leafs        []*Node          // 所有的叶子结点
	hashStrategy func() hash.Hash // 计算哈希的方法
}

```

需要注意的是，`hashStrategy` 是一个函数，其返回 `type hash.Hash interface`，目前最常见的实现是 `sha256.New` 等，这里为了说明清楚原理，我们自己实现一个，计算 `hash`  时，只是简单将其转化为 `[]byte` 即可：

```go
type myHash struct {
	hash      []byte
	data      []byte
	blockSize int
}

func newMyHash() hash.Hash {
	h := &myHash{
		data:      make([]byte, 0),
		blockSize: 64,
	}
	return h
}

// Write 将 p 中的数据更新进 m 
func (m *myHash) Write(p []byte) (n int, err error) {
	nn := 0
	if len(m.data) == 0 {
		m.data = p
		nn = len(p)
	} else {
		m.data = append(m.data, 38)
		m.data = append(m.data, p...)
		nn = len(m.data) + 1 + len(p)
	}
	return nn, nil
}

// Sum 后面追加
func (m *myHash) Sum(b []byte) []byte {
	m.data = append(m.data, b...)
	return m.data
}

func (m *myHash) Reset() {
	m.data = make([]byte, 0)
	m.blockSize = 64
}

func (m *myHash) Size() int {
	return len(m.data)
}

func (m *myHash) BlockSize() int {
	return m.blockSize
}

func newMyHashFunc() hash.Hash {
	return newMyHash()
}
```

另外，对于 `type Content interface`，我们也简单实现一个：

```go
type myContent string

func newMyContent(s string) myContent {
	return myContent(s)
}

func (c myContent) CalculateHash() ([]byte, error) {
	//hash := md5.New()
	//hash.Write(c.ToBytes())
	//return hash.Sum(nil), nil
	return []byte(c), nil
}

func (c myContent) Equals(other merkletree.Content) (bool, error) {
	return reflect.DeepEqual(c, other), nil
}
```

### 创建

接下来我们提供一个构造方法：

```go
//NewTree creates a new Merkle Tree using the content cs.
func NewTree(cs []Content) (*MerkleTree, error) {
	var defaultHashStrategy = sha256.New  // 默认使用 sha256.New 进行哈希
	t := &MerkleTree{
		hashStrategy: defaultHashStrategy,
	}
	root, leafs, err := buildWithContent(cs, t)  // 逐层构建结点
	if err != nil {
		return nil, err
	}
	t.Root = root
	t.Leafs = leafs
	t.merkleRoot = root.Hash
	return t, nil
}

// NewTreeWithHashStrategy 效果同 NewTree，不过使用自定义的哈希函数
func NewTreeWithHashStrategy(cs []Content, hashStrategy func() hash.Hash) (*MerkleTree, error) {
	t := &MerkleTree{
		hashStrategy: hashStrategy,
	}
	root, leafs, err := buildWithContent(cs, t)
	if err != nil {
		return nil, err
	}
	t.Root = root
	t.Leafs = leafs
	t.merkleRoot = root.Hash
	return t, nil
}
```

接着我们来看 `buildWithContent` 做了什么：

```go
// buildWithContent 主要将 Content 转变成 Node，为下一步的逐层构建做好准备
func buildWithContent(cs []Content, t *MerkleTree) (*Node, []*Node, error) {
	if len(cs) == 0 {
		return nil, nil, errors.New("error: cannot construct tree with no content")
	}
	var leaves []*Node
    // 将当前的所有 Content 转化成 Node，放在数组 leaves 中
	for _, c := range cs {
		hash, err := c.CalculateHash()
		if err != nil {
			return nil, nil, err
		}

		leaves = append(leaves, &Node{
			Hash: hash,
			C:    c,
			leaf: true,
			Tree: t,
		})
	}
	root, err := buildIntermediate(leaves, t)  // 逐层构建默克尔树，最后返回树根
	if err != nil {
		return nil, nil, err
	}

	return root, leaves, nil
}
```

再看 `buildIntermediate` 如何逐层构建：

```go
func buildIntermediate(nl []*Node, t *MerkleTree) (*Node, error) {
	var nodes []*Node
	// 如果是单数，不复制自己以凑成对，而是使自己的父节点只有一个左孩子结点(自己)，没有右孩子结点
	for i := 0; i < len(nl); i += 2 {
		h := t.hashStrategy()
		left, right := i, i+1
		var chash []byte
		if right == len(nl) {
			// 单数个，父节点计算哈希时只计算左孩子的
			chash = nl[left].Hash
		} else {
            // 双数个，父节点从左右子孩子的哈希计算得到自己的哈希
			chash = append(nl[left].Hash, nl[right].Hash...)
		}
		if _, err := h.Write(chash); err != nil {
			return nil, err
		}
        // 生成父节点
		node := &Node{
			Left: nl[left],
			Hash: h.Sum(nil),
			Tree: t,
		}
		if right < len(nl) {
			node.Right = nl[right]
		}
		nodes = append(nodes, node)
		if right < len(nl) {
			node.Right.Parent = node
		}
		nl[left].Parent = node
        
        // 如果只有两个，说明当前构造的 node 就是根节点，结束递归
		if len(nl) == 2 {
			return node, nil
		}
	}
    // 递归调用
	return buildIntermediate(nodes, t)
}
```

### 打印

为了方便调试，我们先实现反序列化默克尔树——逐层遍历二叉树。逐层遍历二叉树是数据结构课程中的基础操作，需要用到一个队列，我们先实现一个简单的队列：

```go

type queue struct {
	data []*Node
}

func newQueue() queue {
	q := queue{data: make([]*Node, 0)}
	return q
}

// 入队
func (q *queue) enqueue(c *Node) {
	q.data = append(q.data, c)
}

// 出队
func (q *queue) dequeue() *Node {
	if len(q.data) == 0 {
		return nil
	}
	data := q.data[0]
	q.data = q.data[1:]
	return data
}

// 是否为空
func (q *queue) isEmpty() bool {
	return len(q.data) == 0
}

// 队列中元素个数
func (q *queue) len() int {
	return len(q.data)
}

```

借助队列实现默克尔树的打印：

```go
// Print 打印默克尔树
func (m *MerkleTree) Print() {
	if len(m.Leafs) == 0 {
		fmt.Println("empty tree")
		return
	}
	q := newQueue()
	q.enqueue(m.Root)
	for !q.isEmpty() {
		size := q.len()
		for i := 0; i < size; i++ {
			tmp := q.dequeue()
			if tmp == nil {
				break
			}
			if !tmp.leaf {
				fmt.Printf("hash(%s)   ", tmp.Hash)
			} else {
				fmt.Printf("hash(%s)    ", tmp.Hash)
			}

			if tmp.Left != nil {
				q.enqueue(tmp.Left)
			}
			if tmp.Right != nil {
				q.enqueue(tmp.Right)
			}
		}
		fmt.Print("\n")
	}
}
```

### 查找

先看实现：

```go
// 查找 content 对应的从上到下的路径，index 表示是否为左孩子
func (m *MerkleTree) GetMerklePath(content Content) ([][]byte, []int64, error) {
	for _, current := range m.Leafs {
		ok, err := current.C.Equals(content)
		if err != nil {
			return nil, nil, err
		}

		if ok {
			currentParent := current.Parent
			var merklePath [][]byte
			var index []int64
			for currentParent != nil {
				// 当前节点是父节点的右孩子
				if bytes.Equal(currentParent.Right.Hash, current.Hash) {
					merklePath = append(merklePath, currentParent.Right.Hash)
					index = append(index, 1)
				} else {
					merklePath = append(merklePath, currentParent.Left.Hash)
					index = append(index, 0)
				}
				current = currentParent
				currentParent = currentParent.Parent
			}
			// 添加 root
			if len(merklePath) > 0 {
				if bytes.Equal(m.Root.Left.Hash, merklePath[0]) {
					index = append(index, 0)
				} else {
					index = append(index, 1)
				}
				merklePath = append(merklePath, m.Root.Hash)
			}

			return merklePath, index, nil
		}
	}
	return nil, nil, nil
}
```

### 验证(证明)

首先验证一棵默克尔树是否是有效的：

```go
func (m *MerkleTree) VerifyTree() (bool, error) {
	calculatedMerkleRoot, err := m.Root.verifyNode()
	if err != nil {
		return false, err
	}

    // 重新根据各个结点构建一棵默克尔树，并得到其 root，看是否与已存在的相同
	if bytes.Compare(m.merkleRoot, calculatedMerkleRoot) == 0 {
		return true, nil
	}
	return false, nil
}

func (n *Node) verifyNode() ([]byte, error) {
	if n.leaf {
		return n.C.CalculateHash()
	}
	var (
		rightBytes []byte
		leftBytes  []byte
		err        error
	)
    // 递归处理
	if n.Right != nil {
		rightBytes, err = n.Right.verifyNode()
		if err != nil {
			return nil, err
		}
	}
	if n.Left != nil {
		leftBytes, err = n.Left.verifyNode()
		if err != nil {
			return nil, err
		}
	}

	h := n.Tree.hashStrategy()
	if _, err := h.Write(append(leftBytes, rightBytes...)); err != nil {
		return nil, err
	}

	return h.Sum(nil), nil
}
```

再次验证某个 `Content` 是否属于这棵树(零知识证明)：

```go
func (m *MerkleTree) VerifyContent(content Content) (bool, error) {
	for _, l := range m.Leafs {
		ok, err := l.C.Equals(content)
		if err != nil {
			return false, err
		}
		// 存在于已知的节点中
		if ok {
            // 逐层计算 hash，并比较
			currentParent := l.Parent
			for currentParent != nil {
				h := m.hashStrategy()
				var allBytes []byte

				leftBytes, err := currentParent.Left.calculateNodeHash()
				if err != nil {
					return false, err
				}
				allBytes = leftBytes
				if currentParent.Right != nil {
					rightBytes, err := currentParent.Right.calculateNodeHash()
					if err != nil {
						return false, err
					}
					allBytes = append(allBytes, rightBytes...)
				}

				if _, err := h.Write(allBytes); err != nil {
					return false, err
				}
				if bytes.Compare(h.Sum(nil), currentParent.Hash) != 0 {
					return false, nil
				}
				currentParent = currentParent.Parent
			}
			return true, nil
		}
	}
	return false, nil
}

// calculateNodeHash 计算当前 node 的哈希(左右孩子哈希值组合后，再求哈希)
func (n *Node) calculateNodeHash() ([]byte, error) {
	if n.leaf {
		return n.C.CalculateHash()
	}

	h := n.Tree.hashStrategy()
	var allBytes []byte
	allBytes = n.Left.Hash
	if n.Right != nil {
		allBytes = append(allBytes, n.Right.Hash...)
	}
	if _, err := h.Write(allBytes); err != nil {
		return nil, err
	}

	return h.Sum(nil), nil
}
```

### 重建

```go
// RebuildTree 根据保存的文件块(leaves)重新构建默克尔树
func (m *MerkleTree) RebuildTree() error {
	var cs []Content
	for _, c := range m.Leafs {
		cs = append(cs, c.C)
	}
	root, leafs, err := buildWithContent(cs, m)
	if err != nil {
		return err
	}
	m.Root = root
	m.Leafs = leafs
	m.merkleRoot = root.Hash
	return nil
}
```

也可以根据提供的 `[]Content` 重新构建：

```go
// RebuildTreeWith 根据提供的 content 完全重建一棵树
func (m *MerkleTree) RebuildTreeWith(cs []Content) error {
	root, leafs, err := buildWithContent(cs, m)
	if err != nil {
		return err
	}
	m.Root = root
	m.Leafs = leafs
	m.merkleRoot = root.Hash
	return nil
}
```

## 四、参考文档

- [Merkle 树结构](https://github.com/yeasy/blockchain_guide/blob/master/05_crypto/merkle_trie.md)
- [Merkle Tree（默克尔树）算法解析](https://blog.csdn.net/wo541075754/article/details/54632929)
- [cbergoon/merkletree - go语言实现的merkle树](https://github.com/cbergoon/merkletree)