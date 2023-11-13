policy gradient theorem은 gradient ascent를 구하기 위한 수학적 수식을 풀어서 설명한다.
$$
\begin{align*}
& J(\theta) = \sum_\tau P(\tau;\theta)R(\tau) \newline
& P(\tau;\theta) = [\prod_{t=0}P(s_{t+1}|s_t,a_t)\pi_\theta(a_t|s_t)]
\end{align*}
$$
여기서 시작을 하면
우리는 아래 와 같은 미분을 할 수 있다.
$$
\nabla_{\theta}J(\theta) = \nabla_\theta \sum_\tau P(\tau;\theta)R(\tau)
$$
이 때 좌변을 미분하기위해 상수들을 제외하면 아래와 같이 쓸 수 있다.


$$
\sum_\tau \nabla_\theta P(\tau;\theta)R(\tau)

$$
이는 다시 (잘 모르지만) 미분식처럼 아래 와같이 쓸 수 있다.

$$
\sum_\tau {\operatorname{P(\tau; \theta)}\over\operatorname{P(\tau; \theta)}} \nabla_\theta P(\tau;\theta)R(\tau)

$$
이는 다시 아래처럼 쓸 수 있다.
$$
\sum_\tau  P(\tau;\theta) {\nabla_\theta \operatorname{P(\tau; \theta)}\over\operatorname{P(\tau; \theta)}} R(\tau)

$$
여기서 [[derivateive log trick]]에 의해서 치환이 가능하다.

다른 페이지 에서 자세하게 설명할 예정이지만 derivateive log trick은 아래 수식과 같다.
$$
\nabla_x log f(x) = {\nabla_x \operatorname{f(x)}\over\operatorname{f(x)}}
$$
어떤 함수 f(x)의 자연로그인지 상용로그인지는 아직잘 모르겠다. ?? 이건 그냥 고등학교때 배웠던 로그 공식이랑 같은거다. logf(x)를 미분하면 f(x)를 미분한 f'(x) / f(x)가 된다는거네. 

이럴 수 있다면 우리가 원하는 cumulate sum의 미분은 아래와 같다.
$$
\nabla_\theta J(\theta) = \sum_\tau P(\tau;\theta)\nabla_\theta logP(\tau;\theta)R(\tau)
$$
이때 sample된 i값을 을 통해 trajectory(경로)를 알수 있다면 아래와 같은 식으로 치환이 가능하다
(사실 이부분에서 왜 p(tau;theta)가 변화하는지 아직 잘 모르겠다. 가중 평균 -> 평균으로 바뀐거 같은데 )
$$
\nabla_\theta J(\theta) = {\operatorname{1}\over\operatorname{m}} \sum_{i=1}^m \nabla_\theta logP(\tau^{(i)};\theta)R(\tau^{(i)})
$$
여기서 우리는  logP(tau|theta)를 간단히 해야할 필요성있다. (아직 저 값을 모르니까)

우리는 아래와 같은 값을 안다.
$$
\nabla_\theta log P(\tau^{(i)};\theta) = \nabla_\theta log[\mu(s_0) \prod_{t=0}^H P(s_{t+1}^{(i)} | s_t^{(i)}, a_t^{(i)}) \pi_\theta(a_t^{(i)}|s_t^{(i)})]
$$

여기서 뮤(state0)는 최초의 상태의 분산이고 이후 수식은 현재 (action, state)에서 만들어진 policy로 선택을 했을 때 next_state가 나올 확률의 총 곱을 의미한다. (Markov Decision Porcess에 의해)

이걸 로그에서의 곱을 합으로 다시 나타내면
$$
\nabla_\theta log P(\tau^{(i)};\theta) = \nabla_\theta log\mu(s_0) + \nabla_\theta \sum_{t=0}^H log P(s_{t+1}^{(i)} | s_t^{(i)}, a_t^{(i)}) + 
\nabla_\theta \sum_{t=0}^H log \pi_\theta(a_t^{(i)}|s_t^{(i)})
$$
여기서 첫항과 두번째 항은 theta에 대해 미분되야 하기 때문에 0이 되고 결국 아래와 같은 식으로 남는다.
$$
\nabla_\theta logP(\tau^{(i)};\theta) = \sum_{t=0}^H \nabla_\theta log\pi_\theta(a_t^{(i)}|s_t^{(i)})
$$
결과적으로 아래와 같은 식이 된다.

$$
\nabla_\theta J(\theta) = {\operatorname{1}\over\operatorname{m}} \sum_{i=0}^m \sum_{t=0}^H \nabla_\theta log \pi_\theta(a_t^{(i)}|s_t^{(i)}) R(\tau^{(i)}) 
$$
제기랄...
