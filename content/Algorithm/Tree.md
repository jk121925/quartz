중구 난방으로 생각을 하긴 하지만 지금까지 배운 부분들을 생각해보면 선형 자료구조는 전부 배웠다고 생각한다. LinkedList, Stack, Queue 정도를 이해할 수 있다면 선형 자료구조를 모두 배웠다고 할 수 있다. 선형자료구조란 하나의 데이터 뒤에 하나의 데이터가 오는 상황이다
```
a - b - c - d ...
```
하지만 세상에 많은 문제는 선형자료구조로 전부 풀리지 않는다. 예를 들면 인스타그램이나 페이스북과 같은 sns는 한사람이 여러 사람을 알고 있게 되고 이러한 관계들이 무수히 많이 중첨되어 생기는 거미줄 같은 구조를 가지고 있다. 

![[Pasted image 20240131231452.png|center|500]]

이런 현실의 구조를 다뤄야 하는 상황에서는 분명 다른 자료구조가 필요할 것이다.
이번에는 비선형 자료구조에서 가장 기본이 되는 TREE에 대해서 알아봅시당!

### Tree

사실 이 영어만 보면 모르는 사람을 없을 것이다... 하지만 생각보다 이러한 구조를 만드는 것은 어렵다. 조금 자세히 살펴보자

트리는 아래의 용어를 사용해서 표현된다.
![[Pasted image 20240131232310.png|center|300]]
```
1. 노드(Node)
	트리를 구성하는 기본요소가 된다. 노드는 키 혹은 어떠한 값과 하위 노드에 대한 포인터를 가지고 있다.
2. 간선(Edge)
	노드와 노드를 연결해 주는 연결선을 의미한다.
3. 부모노드(Parent Node)
	자식을 가지는 노드
4. 자식노드(Child Node)
	부모 노드의 하위 노드
5. 형제노드(Sibling Node)
	같은 부모를 가지는 노드
6. 단말노드(terminal Node) || 리프 노드(leaf Node)
	자식이 없는 마지막 노드
7. 가지노드(branch Node)
	자식이 하나 이상있는 노드
8. 깊이(depth) == Level
	루트 노드에서 어떤 노드까지의 간선 수
9. Degree
	노드의 자식수 
```

이제 트리를 좀 구분해 보자면 트리는 아래와 같은 특징을 가진다.
* 하나의 루트노드와 0개 이상의 하위 트리로 구성되어있다.
* 자세히 보면 트리 내부에 트리가 있는 구조로도 이해할 수 있다. -> 재귀적이다.
* cycle한 연결을 가지고 있지 않는다. (순환되지 않는다.)
* 자식은 단 하나의 부모노드를 가진다.

트리를 코드로 짜보면 아래와 같이 짤 수 있다
단. degree가 2인 트리
```python
class Node:
	def __init__(self, num):
		self.num = num
		self.left = None
		self.right = None

rootNode = Node(0)
Node1 = Node(1)
Node2 = Node(2)
Node3 = Node(3)
Node4 = Node(4)
Node5 = Node(5)
Node6 = Node(6)
rootNode.left = Node1
rootNode.right = Node2

Node1.left = Node3
Node1.right = Node4
Node2.left = Node5
Node2.right = Node6
```
위의 코드로 짜게 되면 아래와 같은 구조를 가진다.
```
         1
       /   \
     2      3
    /  \   /  \
   4   5  6   7
```

중요한건 결국 LinkedList를 제대로 구현할지 모르면 트리구조는 제대로 구현할 수 없다는 점이다.
물론 다른 방법으로 구현할 수도 있다.

```python
treeIndex = [-1,0,0,1,1,2,2] <- 각 Node의 부모를 나타낸다.
treeValue = [0,1,2,3,4,5,6]
```

**당연하게도 구조를 나타내는 방법은 여러가지 일 수 있다. 중요한 건 상황에 맞는 선택이다.**

### 순회
문제는 저렇게 구조를 만들어도 내가 잘 만들었는지 확인할 수 있는 방법이 아직 없다는 것이다.
일일이 따라다니면서 출력을 할 수는 없지 않은가? 

![[Pasted image 20240131233449.png|center]]


CRUD의 관점에서 읽는 방법을 트리구조에서는 순회라고 부른다. 결국 다 돌아다니면 너는 누구니 하는 느낌으로 이해할 수 있다.

![[Pasted image 20240131233649.png|center]]

크게 3가지 순회법이 있다. 
#### 전위 순회

이친구는 root를 먼저 방문하는 방법이다.
root를 출력한 후 왼쪽을 방문하고 오른쪽을 방문한다.
```python
        1
      /   \
     2     3
    /  \  /  \
   4   5 6    7

아래와 같은 구조에서 전위순회를 하게 되면 아래와 같이 출력이 된다.
1 -> 2 -> 4 -> 5 -> 3 -> 6 -> 7

rootNode = Node(0) # 아래 노드들이 딸려있다고 생각한다.

def preorder(node):
	print(node.value)
	preorder(node.left)
	preorder(node.right)
```

#### 중위 순회

왼쪽자식을 방문하고 루트노드를 방문한뒤 오른쪽자식을 방문한다.
```python
        1
      /   \
     2     3
    /  \  /  \
   4   5 6    7

아래와 같은 구조에서 전위순회를 하게 되면 아래와 같이 출력이 된다.
4 -> 2 -> 5 -> 1 -> 6 -> 3 -> 7

rootNode = Node(0) # 아래 노드들이 딸려있다고 생각한다.

def inorder(node):
	inorder(node.left)
	print(node.value)
	inorder(node.right)
```

#### 후위 순회

왼쪽 자식을 방문하고 오른쪽 자식을 방문한뒤 루트 노드를 탐색한다.
```python
        1
      /   \
     2     3
    /  \  /  \
   4   5 6    7

아래와 같은 구조에서 전위순회를 하게 되면 아래와 같이 출력이 된다.
4 -> 5 -> 2 -> 6 -> 7 -> 3 ->1

rootNode = Node(0) # 아래 노드들이 딸려있다고 생각한다.

def postorder(node):
	postorder(node.left)
	postorder(node.right)
	print(node.value)
```

30분안에 많은 것을 할 수 없지만 우선 요기까지!!!

### 과제

1. 재귀로 짜진 순회 코드를 loop로 바꿔서 짜오세요!

2. 구현
	우리는 검색엔진을 구축하고 있습니다. 
	검색창에 어떤 단어를 치면 해당 단어까지 같고 그 뒤가 다른 모든 단어를 보여주고 싶습니다
	예를 들어
		DB에 as, ase, asc, asde, ascff, asced라는 단어들이 있다면
		1. a를 검색창에 치면 as, ase, asc, asde, ascff, asced가 후보가 됩니다.
		2. as를 검색창에 치면 as, ase, asc, asde, ascff, asced가 후보가 됩니다.
		3. asc를 검색창에 치면 asc, ascff, asced가 후보가 됩니다.
	과연 최적의 자료구조는 어떻게 될까요? -> 코드를 짜고 해당 코드의 시간복잡도와 공간 복잡도를 알려주세요!

3. https://github.com/jk121925/eject 해당 폴더에 repository를 **fork**해서 [이름]assignment4.py로 pull request를 보내주세요~!