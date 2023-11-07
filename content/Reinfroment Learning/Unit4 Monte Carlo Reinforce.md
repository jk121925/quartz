## 이제는 뭘 할 것인가?

이전 장까지 해서 value-based function을 학습하는 방법에 대해서 배웠다. 정리를 하자면 아래와 같다.

1. value-based function은 state와 action에 따른 reward를 계산하는 방법이다.
2. 우리의 목적은 특정 state에 대해서 가장 높은 reward를 가지는 action을 찾는 것인데 이러한 action을 return하는 것을 policy라고 한다.
3. 이런 policy는 random하거나 greedy한 방식을 통해서 찾아질 수 있으며 Q-table을 통해서 학습시킬 수 있다.
4. 이때 Q-table은 Environment에 따라서 굉장히 많아 질 수 있고 이는 유한 시간내에 답을 찾기 어려울 수 있으므로 우리는 Deep Q-Learning Network를 통해 학습을 진행한다.

이번 장에서는 Monte Carlo라고 불리는 policy-based  gradient algorithm에 대해서 학습한다.

## value-based function이면 되는 줄 알았는데... Policy-based...

우선 누누이 이야기 했다 시피 Reinforcement learning은 어떤 state에 대해서 최대의 reward를 가져다주는 action을 찾는 가장 적합한 policy를 학습하는 것이다.

여기서 지금까지 배운 value-based function은 아래와 같은 특징을 갖는다.
1. optimal value function은 optimal policy와 같다 (Q-table을 잘 만들면 optimal policy와 같다.)
2. 우리의 목표는 예측값과 target value의 차이를 최소한으로 줄이는 것이다.
3. 여기에서 우리는 policy를 가지고 있고 이는 value function에 의해 항상 선택된다.

### 하지만 policy-based method는 직접적으로 policy를 학습한다.
-이게뭔말이고?-

하니. 근본적인 아이디어는 policy를 parameterize하겠다는 것이다. 뭐 예를 들어 policy를 neural network로 정의하고 output으로 나오는 action을 확률로 정의 하겠다는 것이다.

$$
\begin{align*}
Stochastic\ Policy\newline
\pi_\theta(s) = P[A|s;\theta]
\end{align*}
$$
해당 수식의 뜻은 현상태에 대한 policy를 적용하는 것은 어떤 상황에 대한 action을 확률로 나타낸 값이 된다.

### 여기서 우리의 목표는 parameterized된 policy의 gradient ascent를 최소화하는 것이다.

이렇게 하기 위해서 우리는 parameter theta를 조절해 가면서 policy를 정리한다.
![[Pasted image 20231107225755.png]]

내가 이해한대로 설명하자면 
1. 현재 state에는 여러 parameter가 존재한다. 막대의 위치, 속력, 카트의 위치 속력
2. 이떄 이러한 것들을 parameterize하고 input layer로 정리한뒤 neural network를 거쳐 하나의 vector로 정리하고 이를 left와 right로 output하는 flow를 정리한다.
3. 적절한 theta를 찾게 된다면 쓸만한 Policy를 찾게 되는 것이다.

### 그렇다면 policy-based와 policy-gradient method의 차이는 뭔디?

일딴 이런 관계가 성립된다
$$ policy-gradient\ method \in policy-based $$
~~**이런 policy-based method는 각업데이트 최신의 정보를 업데이트하는데 사용한다?** 뭔말인교? 이런 공통점이 있다.~~

여기의 2가지 다른점은 결국 어떻게 parameter theta를 최적화시키느냐가 다른 부분이다.

* policy-based method는 직접적으로 최적의 policy를 찾는데 parameter theta를 hill climbing, simulated annealing 혹은 evolution strategies와 같은 목적 함수를 local에서 maximizing하며 간접적으로 찾게 된다. 
* 반면 policy-gradient method는 parameter theta를 경사상승법 (gradient ascent)으로 찾게 되고 optimal function J(theta)를 찾게 되는 것이다.

## 그래도 꽤 좋은 hugging face... 
## 그러면 policy-gradient method는 뭐가 좋고 뭐가 나쁜데?

### Advantages
1.  action value들을 저장하지 않고 policy를 추정할 수 있다. 
2. policy-gradient methods는 value function이 할 수 없는 **stochastic policy**를 배울 수 있다.
	1. 이에 대한 결과로 우리는 exploration/exploitation trade-off를 우려하지 않아도 된다. 왜냐하면 policy-gradient의 결과는 확률적으로 분산된 값을 통해 state에 대한 action을 평가할 수 있고 항상 같은 state에 대해서 같은 결과를 선택하지 않을 수 있기 때문이다.
	2. perceptual aliasing문제를 제거할 수 있는데 이런 문제는 같은 상황에 대해서 다른 action을 취할때를 이야기한다.


예를 하나 들어보면 아래와 같다. 청소기가 hamster를 죽이지 않고 먼지만 빨아들일 수 있는 방법에 대한 것이다. 
![[Pasted image 20231107231840.png]]
이때 빨간색 칸에서는 특정 policy만을 학습하는 상황에서 한방향으로 학습이 되어있다면 항상 같은 곳을 머무를 것이다. (quasi-deterministic := greedy epsilon strategy)
![[Pasted image 20231107232006.png]]
위와 같은 상황에서는 수많은 episode를 돌려도 의미가 있지 않다.

![[Pasted image 20231107232227.png]]
하지만 optimal stochastic policy는 빨간색 칸에서 확률적으로 움직이기 때문에 결과적으로 짧은 순간만 해당 칸에서 갇혀있을 수 있다.

### Policy-gradient methods는 고차원의 action-spaces 및 연속적인 action-spaces를 가진 공간에서 좀더 좋다.

Deep-Q-learning의 경우에 state에대해서 action과 다음 state에 대해서 학습을 하는데 이는 Q-table에 저장이 되어 학습을 진행한다.

문제는 이러한 action이 무수히 많으면 어떻게 되냐는 것이다. 자율주행 자동차와 같은 문제의 경우 너무나 많은 action을 가지고 있기 때문에 학습에 어려움이 있을 수 있지만 policy-gradient method는 확률로 계산되어 나오기 때문에 이런 문제에 대해서 훨씬 효과 적이다.

### Policy-gradient methods have better convergence properties

결국 어떤 action을 결정하는데 있어서 value-based로 학습을 하게 되면 현재 state와 action에 따른 reward에 따라서 각 state에 대한 action의 value가 최적의 값과 멀어질 가능성이 있는데 이러한 부분이 policy-gradient에서는 조금더 둔감하다고 볼수 있다.

### Disadventage
1. policy-gradient method는 global optimum보다 local maximum으로 수렴(converges)한다.
2. 훨씬 오래 학습한다.
3. 너무 많은 변수를 가질 수 있는데 이는 actor-critic을 통해서 해결해 볼수 있다.

그럼이제 배워보자 ~~수식 더럽게 많더만...~~

[[Unit4 Policy-gradient methods]]