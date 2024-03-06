
### 5535
문제는 5525번 때문에 일어났다.

처음에는 엄청 간단한건 줄 알아서 아래처럼 풀었다.
비트마스킹으로 풀어서 I = 1 O=0처럼 생각하고 비교한다는 생각을 했는데 input output을 확인을 안해서 계속 시간 초과가 났다. 사실 시간 초과도 아니고 아얘 되지도 않았다. 

* 왜냐하면 input에서 비교해야할 문자열이의 길이가 1,000,000인데 2^1000000 만큼을 만들려면 엄청나게 큰 메모리가 필요한데 당연히 안되겠지.... 메모리 할당하다 끝나겠다...

```python
import sys

n = int(sys.stdin.readline())
l = int(sys.stdin.readline())
# 1 4 16 64

octN = int((4 ** (n+1) -1)/3)

s = sys.stdin.readline().strip()
checkbit = 0b0

for i in range(octN.bit_length()):
  if s[i] == 'O':
    checkbit = checkbit << 1 | 0b0
  else:
    checkbit = checkbit << 1 | 0b1
ans = 0
if octN ^ checkbit == 0 :
  ans +=1

for j in range(octN.bit_length(),len(s)):
  # print(0b0 if s[j] == 'O' else 0b1)
  checkbit = (checkbit << 1 | (0b0 if s[j] == 'O' else 0b1)) & (2 ** octN.bit_length() -1)
  if checkbit ^ octN == 0:
    ans +=1
print(ans)
```

두번째로 풀었던 방법은 말도 안된다면서 그냥 생각나는대로 풀었다
그냥 string matching

```python
import sys 

n = int(sys.stdin.readline())
l = int(sys.stdin.readline())
s = sys.stdin.readline().strip()
pattern = "IO" * n + "I"
ans = 0
for i in range(l):
  subs = s[i:i+len(pattern)]
  if subs == pattern:
    ans +=1
    i = i+2
print(ans)
```

안되는 줄 알면서도 왜 그랬을까~ 당연히 시간초과가 나지...
![[Pasted image 20240218200515.png|center]]

## KMP

좀 알아보니 KMP라는 알고리즘이 존재한다고 하더라 라고 모른채 넘어가고 싶지만 예전에 한번 짰던 기억이 있다. 물론 지금은 딱 머리에 "문자열 매칭에 쓴다" 그 이상은 남아있지 않다.

얘는 뭐냐? 블로그에서 많이들 하는 얘기는 접미사와 접두사 같은 배열을 하나 만들고 이 배열을 통해서 문자열을 비교하는 시간을 획기적으로 줄인다. 뭐 이런 이야기다.
구성요소는 아래와 같다.

### pi
항상 우선 pi라는 접미사 접두사가 같아지는 길이를 나타내는 배열에 대해서 알아야 한다고 한다. 이를 구하는 코드를 설명해보자

> [] : pi  = 해당 배열의 index는 0 ~ index 까지 접두사와 접미사가 같은 단어의 개수를 나타낸다.
> int : i = 접두사가 확인된 부분까지의 index
> int : j = 문자열 배열중에서 확인된 배열
```python
abaaba라면
pi = [0,0,1,1,2,3] 이된다. 코드는 아래와 같다.

pattern = "ababaaba"
# pi 배열을 선언
pi = [0 for _ in range(len(pattern))]

# 접두사가 확인된 부분까지의 index
i = 0

# 1 ~ pattern끝까지 접두사, 접미사가 같은 부분 문자열을 확인할꺼임
for j in range(1,len(pattern)):
# 여기서 한참 헤멨는데 이부분은 접두사와 접미사가 같은 부분까지 체크하다가 달라지는 경우를 의미한다. 예시에서는 i = 1, j = 3일 떄가 된다.
#이때 중요한 부분은 i = pi[i-1]이 되는 부분인데 개념은 만약에 j까지 접두사와 접미사가 확인중에 달라지는 순간이 오면 patter[j] == pattern[i]가 되는 최초의 접두사의 마지막 부분까지 후퇴하는 거다. 
	while i > 0 and pattern[i] != pattern[j]:
		i = pi[i-1]
# 만약 pattern을 비교했을때 같다면 i와 j를 같이 증가시켜주고 pi[j]=i로 업데이트 시켜준다. j번째가지 확인했을 때 같은 접두사는 i번째까지 있다는 것.
	if pattern[i] == pattern[j]:
		i +=1
		pi[j]=i

# pi배열을 만들었다면 같은 개념으로 패턴매칭을 진행시켜주면된다.
matcing_string = "..."

i = 0
ans =0
for j in range(len(matching_string)):
# 문자가 다른경우는 이전에 같은 접두사까지 뒤로 돌아간다. 특정 부분에서 다르면 그 이전(pi[i-1])까지는 같았다는 이야기가 됨으로 시간을 줄일 수 있게된다.
	while i > and pattern[i] != matching_string[j]:
		i = pi[i-1]
	if pattern[i] ==matching_string[j]:
		i+=1
# 만약에 패턴을 찾았다면 pattern에서 접두사와 접미사가 같은 부분까지 뒤로 갈 수 있게 되는 것이다.
		if i == len(pattern):
			ans +=1
			i = pi[i-1]
print(ans)
```

해보면 코드는 별거 없는데 이해하는데 한참 걸렸고 지금도 어렴풋이 헷갈리지만! 어쨌든 여기까지!