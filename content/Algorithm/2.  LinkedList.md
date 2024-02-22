
### 배열이란?

컴퓨터의 물리적 메모리는 일정한 크기의 방처럼이해를 할 수 있다.
한 개의 방에 크기에 따라서 여러 변수들을 저장할 수 있게 된다. (이 정도는 알아야 한다...)

그렇다면 배열이란 **물리적으로 붙어있는(연속적인) 메모리의 나열** 이라고 생각하면 쉽다.

```python
array = []
otherarray = list()
```

이렇게 쓰게 되는데 이러한 배열은 RAM(Random Access Memory)로 index를 알 수 있다면 무조건 적으로 접근이 가능하다.

```python
array = ["a","b","c"]
a = array[0]
```

이러한 배열은 장단점이 분명하게 존재한다.

#### 특징
* Create : 선언하기 쉽고 쓰기 간편하다. 다만 배열의 크기가 정해진 경우 O(n)의 시간만큼의 시간이 걸린다.
* Read :만약 위치를 정확하게 명확하게 알수 있다면 원하는 데이터에 O(1) 시간안에 접근이 가능하다.
* Update : 특정 위치에 삽입을 하는 경우 찾는데 O(1) 수정시 O(1)의 시간이 걸리게 된다.
* Delete : 특정 위치에 삭제를 하는 경우 찾는데 O(1) 삭제시 O(n)의 시간이 걸리게 된다.

### LinkedList : 연결리스트

배열은 생성과 읽기가 굉장히 쉬운 자료구조이다. 
하지만 프로그래밍에서 만들고 읽는 것만 하는 부분은 없다. (굉장히 적다) 해서 특정 조건하에 생성은 많이 되질 않고 수정과 삭제가 빈번한 상황에서 특별히 좋은 성능을 가지는 자료구조가 바로 **연결리스트**이다.

#### 특징
 * Create : 특정 위치를 고정해서 생성하는 경우에 O(1) 시간안에 가능하다. 또한 크기가 정해져 있지 않아 배열을 수정하는 시간이 걸리지 않는다.
 * Read : 특정 위치를 찾기 위해서 최악의 경우 모든 Node를 탐색하여 O(n)의 시간이 걸린다.
 * Update : 특정 위치에 삽입을 하는 경우 찾는데 O(n) 수정하는데 O(1) 시간안에 가능하다.
 * Delete : 특정 위치에 삽입을 하는 경우 찾는데 O(n) 삭제하는데 O(1) 시간안에 가능하다.


#### 연결리스트의 개념

연결리스트는 Node를 기본으로 가진다.
```python
class Node:
	def __init__(self,value):
		self.value = value
		self.pointer = Node
```

> Node는 자료를 나타내는 value와 다른 노드를 지칭할 수 있게 해주는 pointer로 구성이 된다.
> value는 Int, String, Double, Class등 여러 타입이 될 수 있다.
> pointer는 다른 노드를 지칭하게 된다.

#### Pointer

Node에 존재하는 pointer의 역할은 다른 Node를 가리키게 하는 것이다.
NodeA를 읽게 된다면 자동적으로 다른 NodeB 혹은 NodeC를 가리키는 하나의 변수를 얻게 되는 것이다.
```python
class Node:
	def __init__(self,value):
		self.value = value
		self.pointer = Node

NodeA = Node(10)
NodeB = Node(11)

NodeA.pointer = NodeB
```
위 코드와 같이 NodeA와 NodeB가 존재한다고 했을 때 NodeA.pointer = NodeB를 했을 때, NodeA의 pointer에는 NodeB의 주소값이 들어가게 되고, 이를 통해 다음 값을 알 수 있게 된다.

### SingleLinkedList

SingleLinkedList는 하나의 노드에서 시작해서 한 방향으로만 추가를 하는 형태를 의미한다.
```python
class Node:
	def __init__(self,value):
		self.value = value
		self.pointer = Node

rootNode = Node(0)
headNode = Node(-1)
headNode = rootNode
for i in range(10):
	addNode = Node(i)
	addNode.pointer = rootNode.pointer
	rootNode.pointer = addNode
```

### DoubleLinkedList
SingleLinkedList는 하나의 노드에서 시작해서 양 방향으로만 추가를 하는 형태를 의미한다.
```python
class Node:
	def __init__(self, value):
	self.value = value
	self.head = None
	self.tail = None

for i in range(1,10):
	addNode = Node(i)
	if rootNode.tail ==null:
		addNode.head = rootNode
		rootNode.tail = addNode
	else:
		addNode.head = rootNode
		addNode.tail = rootNode.tail
		rootNode.tail.head = addNode
		rootNode.tail = addNode
```

# 과제
![[Notes_240111_195936.pdf]]

## Answer

```python
cclass Node:
  def __init__(self, value):
    self.value = value
    self.up = None
    self.down = None
    self.left = None
    self.right = None

  def iter(self):
    iterNode = self
    rightflag = True
    ret_str = ""
    while True:
      ret_str += str(iterNode.value) + " "
      if iterNode.right !=None and rightflag:
        iterNode = iterNode.right
      elif iterNode.left !=None and not rightflag:
        iterNode = iterNode.left
      elif iterNode.right == None and iterNode.down !=None:
        iterNode = iterNode.down
        rightflag = False
      elif iterNode.left == None and iterNode.down !=None:
        iterNode = iterNode.down
        rightflag =True
      elif iterNode.right == None and iterNode.down == None:
        break
    return ret_str

oneNode = Node(1)
leftTop = oneNode
rightiter = oneNode
for i in range(2,26):
  addNode = Node(i)
  if i % 5 == 1:
    addNode.up = leftTop
    leftTop.down = addNode
    leftTop = addNode
    rightiter = addNode
  elif rightiter.value <5:
      rightiter.right = addNode
      addNode.left = rightiter
      rightiter = addNode
  elif rightiter == 5:
    rightiter.right =addNode
    addNode.left = rightiter
    rightiter = addNode
  else:
    rightiter.right = addNode
    addNode.left = rightiter
    addNode.up = rightiter.up.right
    addNode.up.down = addNode
    rightiter = addNode
print(oneNode.iter())```
