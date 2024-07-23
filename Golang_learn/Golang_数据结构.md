---
title: _Golang 数据结构
---
# 稀疏数组（SparseArray）
## 基本介绍
当一个数组中大部分元素为0，或者为同一个值的数组时，可以使用稀疏数组来保存该数组。

稀疏数组的处理方式是：
1. 记录数组一共有几行几列，由多少个不同的值；
2. 把具有不同的值的元素的行列及值记录在一个小规模的数组中，从而缩小程序的规模。

例如：
一个数组中
0行3列的值是55，
0行6列的值是81，
3行3列的值是14；

稀疏数组是这样记录的：
| row | col | value |
| ----- | ----- | ----- |
| 0 | 3 | 55 |
| 0 | 6 | 81 |
| 3 | 3 | 14 |
## 应用实例
1）使用稀疏数组，来保留类似前面的二维数组（棋盘、地图等）；
2）把稀疏数组存盘，并且可以从新恢复原来的二维数组。

```golang
/*
	稀疏数组
*/

//值节点
type ValueNode struct {
	row   int
	col   int
	value interface{}
}

//标准的稀疏数组应有 记录元素的二维数组的规模（行、列、默认值）
type SparseArray struct {
	Row int
	Col int
	Val int
	Arr []ValueNode
}

func (sa *SparseArray) GetName() string {
	return "稀疏数组"
}

//test
func (sa *SparseArray) Test() {
	// 1.先创建一个原始数组
	var array [11][11]int
	array[1][2] = 1 //黑子
	array[2][3] = 2 //白子

	// 2.转成稀疏数组
	sparseArr := SparseArray{
		Row: len(array),
		Col: len(array[0]),
		Val: 0,
		Arr: make([]ValueNode, 0),
	}
	for row, a := range array {
		for col, v := range a {
			if v != 0 {
				sparseArr.Arr = append(sparseArr.Arr, ValueNode{
					row:   row,
					col:   col,
					value: v,
				})
			}
		}
	}
	fmt.Println(sparseArr)
}
```

# 队列（Queue）
## 基本介绍
队列是一个有序列表，可以用**数组**或是**链表**来实现。
遵循**先进先出**的原则。

## 数组模拟队列
应有最大容量maxSize。
因为队列的输出输入分别从前后端来处理，所以需要两个变量front及rear分别记录队列前后端的下标，front会随着数据输出而改变，而rear会随着数据输入而改变。
==只能用一次==

1）入队时将尾指针rear往后移：rear+1
2）当front == rear时，表示队列为空
3）当rear == maxSize-1时，表示队列已满
```golang
/*
	队列
*/

type Queue struct {
	//头部，默认为-1
	front int
	//尾部，默认为-1
	rear int
	//最大容量
	maxSize int
	arr     []int
}

// 在尾部添加一个新的元素
func (q *Queue) Push(num int) (err error) {
	if q.rear >= q.maxSize-1 {
		return errors.New("error: Queue full")
	}
	q.rear++
	q.arr[q.rear] = num
	return
}

// 移除第一个元素
func (q *Queue) Pop() (value int, err error) {
	if q.Empty() {
		err = errors.New("error: Queue empty")
	} else {
		q.front++
		value = q.arr[q.front]
		q.arr[q.front] = 0
	}
	return
}

// 访问最后一个元素
func (q *Queue) Back() (value int, err error) {
	if q.Empty() {
		err = errors.New("error: Queue empty")
	} else {
		value = q.arr[q.rear]
	}
	return
}

// 访问第一个元素
func (q *Queue) Front() (value int, err error) {
	if q.Empty() {
		err = errors.New("error: Queue empty")
	} else {
		value = q.arr[q.front+1]
	}
	return
}

// 获取当前元素的个数
func (q *Queue) Size() int {
	return len(q.arr)
}

// 判断队列是否为空
func (q *Queue) Empty() bool {
	if q.front == q.rear {
		return true
	} else {
		return false
	}
}

// 获取队列所有元素
func (q *Queue) GetArr() []int {
	array := make([]int, 0)

	array = append(array, q.arr...)

	return array
}

func NewQueue(maxSize int) *Queue {
	q := &Queue{
		front:   -1,
		rear:    -1,
		maxSize: maxSize,
		arr:     make([]int, maxSize),
	}
	return q
}
```

## 数组模拟循环队列
来源：https://blog.csdn.net/weixin_42117918/article/details/81839751
通过==取模==的方式来实现环形队列

1）rear的下一个为front时表示队列已满，即**队列容量需要空出一个作为约定**：
(rear+1) % maxSize == front % maxSize
2）当front == rear时，表示队列为空
3）初始化时，front = 0，rear = 0
4）获取元素的个数，(rear - front + maxSize) % maxSize
```golang
/*
	循环队列
*/

type CircularQueue struct {
	//头部，默认为0
	front int
	//尾部，默认为0
	rear int
	//最大容量
	maxSize int
	arr     []int
}

// 在尾部添加一个新的元素
func (cq *CircularQueue) Push(num int) (err error) {
	if cq.IsFull() {
		err = errors.New("error: CircularQueue full")
	} else {
		cq.arr[cq.rear] = num
		cq.rear = (cq.rear + 1) % cq.maxSize

		fmt.Println("rear: ", cq.rear)
	}
	return
}

// 移除第一个元素
func (cq *CircularQueue) Pop() (value int, err error) {
	if cq.Empty() {
		err = errors.New("error: CircularQueue empty")
	} else {
		value = cq.arr[cq.front]
		cq.arr[cq.front] = 0
		cq.front = (cq.front + 1) % cq.maxSize

		fmt.Println("front: ", cq.front)
	}
	return
}

// 访问最后一个元素
func (cq *CircularQueue) Back() (value int, err error) {
	if cq.Empty() {
		err = errors.New("error: CircularQueue empty")
	} else {
		value = cq.arr[cq.rear]
	}
	return
}

// 访问第一个元素
func (cq *CircularQueue) Front() (value int, err error) {
	if cq.Empty() {
		err = errors.New("error: CircularQueue empty")
	} else {
		value = cq.arr[cq.front]
	}
	return
}

// 获取当前元素的个数
func (cq *CircularQueue) Size() int {
	return (cq.rear - cq.front + cq.maxSize) % cq.maxSize
}

// 判断队列是否为空
func (cq *CircularQueue) Empty() bool {
	return cq.front == cq.rear
}

// 判断队列是否已满
func (cq *CircularQueue) IsFull() bool {
	return cq.front%cq.maxSize == (cq.rear+1)%cq.maxSize
}

// 获取队列所有元素
func (cq *CircularQueue) GetArr() []int {
	array := make([]int, 0)
	array = append(array, cq.arr...)
	return array
}

func NewCircularQueue(maxSize int) *CircularQueue {
	cq := &CircularQueue{
		front:   0,
		rear:    0,
		maxSize: maxSize,
		arr:     make([]int, maxSize),
	}
	return cq
}
```


# 链表（List）
## 单链表
设置一个头节点，头节点的作用主要是标识链表头，本身不存放数据。

## 双向链表

## 环形链表