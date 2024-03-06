### 개요

자료구조를 배우는데 왜? Brute-Force라는게 나오냐? 함은 알고리즘이란 **"컴퓨터에게 일을 시키는 방법에 대한 것"** 이라고 생각해보면 가장 간단한 일이 다 해봐 하는 것이다.

![[Pasted image 20240117231227.png]]

"다 해봐"를 좀 간지나게 표현하면 Brute-Force가 되는것이다. 원래 컴퓨터는 전쟁시 적군의 암호를 해독할 요령으로 나온 것으로 알고 있다. 이때는 인간이 1일 걸릴 것을 뭐 4시간만에 할 수 있다면 댕 좋은 상황이었기 때문에 확실한 효력이 있을 것이고 이때 부터 Brute-Force라는 "알고리즘"이 등장한 것이다.

지금부터 brute-Force에 대해서 배워보자!!!

### 브루트 포스란?

한 가지 문제를 통해서 Brute-Force를 이해해 보자

```
배열 [1,2,3,4] 가 존재한다.
여기서 나올수 있는 모든 4가지 숫자를 조합해보자
eg.) 
	1,2,3,4
	1,2,4,3
	1,3,2,4
	...
```

이런 문제가 있다면 머리에 드는 가장 간단한 생각은 이런거지

> for 문으로 하면 되는거 아닌가???

라는 생각이 들고 실제로 짜보려고 하면 이제 문제가 생기기 시작한다.

```python
arr = [1,2,3,4]
ans = ""
for i in range(4):
	for j in range(4):
		for k in range(4):
			for t in range(4):
				if arr[t]
				??????
```

머리에는 😵‍💫띠용😵‍💫과 함께 뇌가 멈추가 시작한다.
![[Pasted image 20240117233843.png|center]]

어찌어찌 for loop로 짜더라도 만약 문제에서 배열이 변하기 시작하면 for loop를 무한정 써야하는데 이걸 계속 쓰고 있으면 뭐하는 건가 라는 생각이 들기 시작한다.

### 탈출! 해! 

이런 상황이면 방법을 다르게 해야한다. **컴퓨터에게 내가 원하는 모든 경우의 수를 탐색하는 방법**에 대해서 알려줘한다. 이때 recursion을 사용한다. 재귀적인 탐색으로 모든 경우의 수를 탐색하는 것이다.

#### 1. 탈출 조건을 정한다.

중요한 것은 탈출하는 방법이다. ㅋㅋㅋㅋ
![[Pasted image 20240117234337.png|center]]
```
배열 [1,2,3,4] 가 존재한다.
여기서 나올수 있는 모든 4가지 숫자를 조합해보자
eg.) 
	1,2,3,4
	1,2,4,3
	1,3,2,4
	...
```

다시 문제로 돌아가 보면 우리가 재귀에서 탈출할 수 있는 조건은 딱하나!!! 4개의 순서를 모두 정했을 경우이다.

```python
arr = [1,2,3,4]
answer = []
def bruteForce(count):
	if count == 4:
		'''
		do Something what you want!!!!
		print(answer)
		'''
		return
```

위에 코드는 bruteForce에 count라는 Argument를 넣어준다.
현재 몇개의 숫자를 선택해서 answer가 가지고 있는지를 나타낸다. 
탈!출! 조건은 answer에 4개의 숫자가 "우리가 의도한 순서대로"들어갔을 때이다.

그럼 다음은? answer에 우리가 의도한 순서대로 들어갈 수 있게 하면 된다.

#### 2. 해!

바로코드!
```python
arr = [1,2,3,4]
visit = [False for i in range(len(arr))]
answer = []
def bruteForce(count):
	if count == 4:
		'''
		do Something what you want!!!!
		print(answer)
		'''
		return
	for i in range(len(arr)):
		if not visit[i]:
			answer.append(arr[i])
			bruteForce(count+1)
			answer.remove(arr[i])
```

우선 visit라는 배열을 만들어서 arr에서 특정 index의 순서를 사용했는지 확인을 한다.
만약 for loop를 돌면서 index의 배열을 answer에 넣지 않았다면 해당 배열을 넣고 recursive function을 호출하게 된다.

#### 3. 그럼?

여기까지 오면 코드는 이해가 될랑 말랑하는데 어떻게 동작하는지 욕이 나오게 된다.
어떻게든 설명해 보면 

```
1번
	  2
	/ 
1   - 3 
	\
	  4
```

1번 호출에서 for loop는 1을 answer에 넣고 2번 호출로 간다.
```
1번   2번
	  2
	/ 
1   - 3 
	\
	  4
```
2번 호출에서 는 아래와 같은 상황이 된다
* count == 1이 되는데 4가 아님으로 return하지 않는다.
* visit[1] = True 이고 visit[2] = False임으로 2를 answer에 넣고 3번 호출한다.
```
1번   2번  3번   4번       5번
		  3  -  4  -  "eject!"
		/
	  2 - 4
	/ 
1   - 3 
	\
	  4
```
3번 호출에서는 2번과 같은 논리로 1과2를 건너뛰고 3을 answer에 넣고 4번 호출을 한다.
4번 호출에서는 answer에 4를 넣고 5번 호출을 한다.
5번 호출에서는 count == 4이기 때문에 eject한다!
```
1번   2번  3번   4번       5번
		  3  -  *4  <-  "eject!"
		/
	  2 - 4
	/ 
1   - 3 
	\
	  4
```
5번에서 return한 뒤에는 4번호출로 가서 answer에 4를 지우게 된다. 
```
1번   2번  3번   4번       5번
		  *3  <-  4  <-  "eject!"
		/
	  2 - 4
	/ 
1   - 3 
	\
	  4
```
이후 코드를 진행하면 더 넣을 번호가 없기 때문에 3번 호출로 가게된다.
```
1번   2번  3번   4번       5번
		  3  <-  4  <-  "eject!"
		/
	  2 -> *4 -> 3  -> "eject!"
	/ 
1   - 3 
	\
	  4
```
3번 호출에서는 3을 제거하고 나머지 forloop를 돌면서 4를 넣게된다.

글은 어렵지만 잘 생각해 보거나 python debug를 통해 이해하면 좋겠다!

### 마치며...

brute-force도 못짜면 문제가 있다 (숙련된  전공자라는 가정하에 "복수전공자는 필수")
중요한건 **탈출!**  **해!** 를 기억합시다


### 과제!

##### 3x3의 정사각형을 1x1 정사각형 2개, 2x1 직사각형 2개, 3x1 직사각형 1개로 채우는 문제

### INIT array
```
0 0 0
0 0 0
0 0 0
```

### Sample answer
```
1 2 3
4 4 3
5 5 5
```
```
4 4 3
2 1 3
5 5 5
```
```
4 4 1
5 5 5
2 3 3

```

#### Data Structure
```
1 - 1x1 rectangle
2 - 1x1 rectangle
3 - 2x1 rectangle
horizon 3 3
vertical 3
         3
4 - 2x1 rectangle
horizon 4 4
vertical 4
         4
5 - 3x1 rectangle
horizon 5 5 5
vertical 5
         5
		 5
```

### HOW MANY RECTANGLE DO YOU FILL?

결과는 갯수만 알려주면 굳!

### Answer
```python
import numpy as np
class Rectangle:
  def __init__(self,value, w,h):
    self.value = value
    self.location = [w,h]
    

def checkBoard(startPoint,endPoint):
  checkValue = board[startPoint[0]:endPoint[0], startPoint[1]:endPoint[1]].ravel()
  TF = lambda checkValue : np.all(checkValue == 0)
  return TF(checkValue)

def fillBoard(startPoint, endPoint,value):
  board[startPoint[0]:endPoint[0], startPoint[1]:endPoint[1]] = value


board = np.zeros((3,3), dtype=int)
ans = 0

rectangles = [Rectangle(1,1,1), Rectangle(2,1,1), Rectangle(3,1,2), Rectangle(4,1,2), Rectangle(5,1,3)]
use = [False for _ in range(5)]

def bruteForce(nowPoint):
  global ans,board
  if nowPoint==5:
    ans+=1
    print("ans-"*5)
    print(board)
    print("-ans"*5)
    return

  for i in range(3):
    for j in range(3):
      if board[i][j] == 0:
        for index,rec in enumerate(rectangles) :
          if  not use[index] and i+ rec.location[0] <4 and j+rec.location[1] <4 and checkBoard([i,j],[i+rec.location[0], j+rec.location[1]]):
            use[index] = True
            fillBoard([i,j],[i+rec.location[0], j+ rec.location[1]],rec.value)
            bruteForce(nowPoint+1)
            fillBoard([i,j],[i+rec.location[0], j+ rec.location[1]],0)
            use[index] = False
          if rec.location[0] != rec.location[1] and not use[index] and i+ rec.location[1] <4 and j+rec.location[0] <4 and checkBoard([i,j],[i+rec.location[1],j+rec.location[0]]):
            use[index] = True
            fillBoard([i,j],[i+rec.location[1],j+rec.location[0]],rec.value)
            bruteForce(nowPoint+1)
            fillBoard([i,j],[i+rec.location[1],j+rec.location[0]],0)
            use[index] = False

bruteForce(0)```
