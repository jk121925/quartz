
### 딱보면 제일 어려운...

처음에 프로그래밍을 조금 접하다 보면 알고리즘이라는 것을 알게된다. 이후에 백준이던 다른 어떤 곳이던 문제를 풀다 돌아다니다 보면 Stack Queue와 같은 말을 듣게 된다. 같이 있는 말중에 다른 알고리즘 종류는 tree, graph이런걸 같이 보게되면 stack과 Queue는 상대적으로 어려운 말이라 사실 엄두가 안난다.

하지만 알고보면 제일 쉬운 자료구조가 Stack과 Queue이다. 간단하지만 Stack은 컴퓨터가 작동하는 Stack pointer를 구성하는 자료구조이고, Queue는 운영체제의 page fault 알고리즘 중 하나로 이용되기도 한다.

이제부터 Stack과 Queue의 기본적인 함수와 동작에 대해서 배워보자

### Stack
Stack은 아래와 같은 함수를 가진다.

``` python
def pop()
	'가장 마지막에 추가된 값을 return하고 해당 값을 자료구조에 제거한다.'
	return
def push()
	'자료구조의 맨 앞에 값을 추가한다.'
	return
def isEmpty()
	'자료구조가 비어있는지 리턴한다.'
def top()
	'자료구조의 맨앞에 있는 값을 return 한다. 단 해당 값을 자료구조에서 제거 하지 않는다.'
```

Stack은 FirstInLastOut의 구조를 가진다.

즉 첫번째로 들어간 것은 가장 마지막에 나오게 되는 것이다.

그림으로 보면 아래와 같다.
```
최초에는 아래와 같이 head가 None || null을 가리킨다.
head
 |
None
```

#### "a"의 값을 추가하게 되면 가장 첫번째 값이 추가되고 head가 "a"를 가리키게 된다.
```
head
 |
"a"
 |
None
```
#### "b"의 값을 추가하게되면 이전값인 "a"의 앞에 "b"가 추가된다.
```
head
 |
"b"
 |
"a"
 |
None
```
#### 여기서 top function을 호출하게 되면 "b"를 return 하지만 삭제 하지 않는다.
```
head
 |
"b" <- top
 |
"a"
 |
None
```
#### pop function을 호출하게 되면 "b"를 return하고 자료구조에서 삭제한다.
```
head
 |
 |        "b" <- pop
 |
"a"
 |
None
```

이런 구조를 가지게 되면서 삽입과 삭제(읽기)를 "최악의 경우에도 1번"만에 할 수 있기 때문에 유용한 구조가 될 수 있다.

#### Stack Pointer
![[Pasted image 20240124201613.png]]
출처 https://narakit.tistory.com/144

위에 구조를 보게 되면 main이 프로그램의 진입점이 되면서 var과 var2를 순서대로 선언하고 func2를 호출한다(그림에서 func1 -> func2) 이상황에서 func2로 넘어가야할 때 stack pointer에 func2의 위치(main의 3번째 줄)를 넣어두고 func2가 return하게 되면 해당 값을 main에 func2를 호출하는 시점으로 돌려주는 것이다.

가끔 보게되는 maximum recursion is exceed max stack 은 이 stack pointer가 컴퓨터가 가지는 stack 메모리를 초과했다는 이야기가 된다.

### Queue

Queue는 아래와 같은 함수 구조를 가진다.
```python
def add:
	'값을 맨뒤에 추가한다.'
def popfirst:
	'맨 첫번째 값을 return 후 삭제한다.'
def popLast:
	'맨 마지막 값을 return 후 삭제한다.'
	'dequeue 구조에서 사용'
def first:
	'맨 첫번째 값을 return 한다.'
def last:
	'맨 뒤에 값을 return 한다.'
	'dequeue 구조에서 사용'
def isEmpty:
	'Queue가 비어있는지 확인한다.'
```

사실 Queue의 구조는 각 언어마다 기본적으로 지원하는 방식이 다를 수 도 있다. 하지만 기본적으로 맨 앞의 값과 맨 뒤에 값을 뽑아낼 수 있게 하고, 값은 맨뒤에만 추가할 수 있다.

Queue는 FirstInFirstOut으로 첫번째로 들어간 값을 가장 먼저 확인할 수 있다. 구현방식에 따라 다르게 되지만 추가, 삭제같은 작업은 O(1)의 시간안에 가능하다.

그림으로 보면 아래와 같다.
```
최초에는 아래와 같이 head가 None || null을 가리킨다.
head tail
 |    /
None
```

#### 맨뒤에 "a"의 값이 추가되면 아래와 같은 그림이 된다.
```
head tail
  \    /
    "a"
```

#### 맨뒤에 "b"의 값이 추가되면 아래와 같은 그림이 된다.
```
head         tail
  \          /
    "a" - "b"
```

#### 맨뒤에 "c"의 값이 추가되면 아래와 같은 그림이 된다.
```
head               tail
  \               /
    "a" - "b" - "c"
```

#### pop first를 진행하게 되면 아래와 같은 그림이 된다.
```
pop    head       tail
 |       \        /
"a"      "b" - "c"
```

#### pop last를 진행하게 되면 아래와 같은 그림이 된다.
```
    head tail       pop
      \  /           |
      "b"           "c"
```


### 과제

1. Stack과 Queue를 Array List로 구현 하시오!
2. Stack과 Queue를 Linked List로 구현하시오!
3. 두개의 구현 방식에 따라 Big-O notation을 구하고 비교하시오!