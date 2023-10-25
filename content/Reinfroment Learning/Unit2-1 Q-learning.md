# Q-learning

Q-Learning이라는 놈은 **off-policy value-based method that use TD approach to train its action-value function**이다.

중요한 점은 
1. off-policy 
2. value-based : 최적의 policy를 찾기위해서 각 state의 action pair의 value를 계산한다.
3. TD approach : episode의 전체 rewoard를 계산할 수 없기 때문에 다음 state의 reward를 할인해서 value를 계산한다.
![[스크린샷 2023-10-25 오후 9.22.43.png]]

### Q-learning은 Q value를 최대화하는 Q-Function을 찾는 것이다!

여기서 Q-Function은 Q-table로 표현된다.
>[!example]
> ![[스크린샷 2023-10-25 오후 9.24.24.png]]
> 위 그림은 예제인데 쥐가 가장 큰 reward를 받기 위한 방법을 이야기한다.
> ![[스크린샷 2023-10-25 오후 9.26.10.png]]
> 최초에 Q-talbe은 0으로 초기화 되고 action과 state pair의 2차원 table을 가진다.
> 이때 쥐가 선택할 수 있는 action은 오른쪽으로 가거나 아래쪽으로 가는 방향이다.
> 이러한 관계를 통해 state-action pair를 업데이트 해나가는 것이 Q-learning의 중점이다.

### 결과적으로 이러한 방법을 통해 Q-function을 찾게 되고 이는 Q-table로 표현이 되는 것이다.

아래는 Q-learning의 presudo Code이다.
# 줄바꿈이 안된다 제기랄 몰라 내일해 ㅋㅋㅋㅋ

$$ 
Algorithm : Q-Learning\ \newline \ Input : policy \pi 
$$

$$
`
g(x)=Ax^4
$$
![[스크린샷 2023-10-25 오후 9.44.52.png]]

![[스크린샷 2023-10-25 오후 9.45.16.png]]


