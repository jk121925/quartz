### Abstract

논문에서는 수용영역에서 local patch에 대한 모델 차별성을 높이기 위해 새로운 구조를 제시했다. 전통적인 CNN모델은 비선형 활성 함수와 함께 사용되는 선형 필터를 이용했다. 여기서 생기는 문제는 여전히 선형 필터를 이용하고 있다는 점이다. 이러한 문제 의식으로 NIN은 수용영역 안에 데이터를 더 복잡하게 추상화 할 수 있는 micro neural network를 제안했다.

### Introduction

일반적인 CNN의 Conv. filter는 GLM(generalized linear model)이다. 이 GLM은 추상화 수준이 낮다고 이야기 하면 GLM을 효과적인 비선형 함수로 대체 한다면 모델의 추상화를 더 잘 할 수 있을 것이라 이야기 한다.

- 추상화란?
    
    내가 생각하는 추상화란 linear하게 딱딱 나눠지는 것이 아니라 인간이 사고하는 것과 비슷한 느낌으로 분류 기준을 좀 더 일반화하는 개념을 의미하는 것
    
![[Network In Network-1.png]]

이에 본 논문은 NIN이라는 모델을 제시하고 GLM을 비선형 함수인 micro network로 대체한다. 이는 역전파로 훈련이 가능하고 Multilayer Perceptron으로 구성되어있다.

(b)에 보이는게 비선형 활수와 함께 여래개의 fully connected layer로 이루어진 multilayer perceptron인데 입력 local patch를 output feature vector와 매칭 시켜주는 모습을 볼 수 있다. 이러한 mlpconv layer를 여러겹으로 쌓은 것이 NIN이다.

여기서는 분류를 위해서 fully connected layer 대신 global average pooling을 사용한다. 이러한 GAP를 통해서 파라미터수를 어느정도 줄여주게 되면 (이뿐만 아니라 GAP자체가 regularizer하는 방법으로 인식됨) overfitting을 방지할 수 있게 된다.

생각해보면 Network 안에 Network가 있다면 당연히 파라미터는 n승으로 늘어날 것 같은데, 당연히 파라미터를 줄여서 regularizer하는 방안을 찾아야 할 것 같았다.

- global average pooling
    
    이는 다른 pooling보다 급격하게 feature의 수를 줄일 수 있다. **이의 목적은 feature를 1차원 벡터로 만들기 위함이다. 이는 같은 채널의 feature들을 모두 평균을 낸다음에 채널 수 만 큼의 원소를 가지는 벡터로 만들어 준다.**
    
    이런 방식으로 하면 (height, width, channel)의 형태를 (channel)의 형태로 간단하게 만든다.
    
    - Why?
        
        GAP는 CNN +FC layer에서 classifier인 fc layer를 없애기 위한 방법으로 도입되었다.
        
        이는 파라미터 수를 급격하게 줄어드는 리소스 측면에 이익을 줄 수 있는 방법이 됩니다.
        
    
    그리고 GAP는 어떤 크기의 feature라도 같은 채널의 값들을 하나의 평균 값으로 대체하기 때문에 벡터로 결과값이 나온다. 
    
    <aside>
    💡 읽어보면 좋은 점
    
    feature map에 fully-connected layer를 사용하게 되면 모든 feature map의 정보가 연결되기 때문에 각 class가 어떤 이유로 선택되었는지 알기가 어렵다. 하지만 global average pooling는 conv층을 통과한 각각의 feature map을 평균한 것 이기 때문에 어느정도 각 feature map의 특성을 가지고 있다고 할 수 있다. 이는 모델의 해석에 도움을 줄 수 있다.
    
    </aside>
    
    [Network In Network(NIN) 정리](https://velog.io/@whgurwns2003/Network-In-NetworkNIN-%EC%A0%95%EB%A6%AC)
    
    ### Convolutional Neural Network
    
    → 이 부분에서는 전통적인 CNN에 대한 단점에 대해서 언급을 한다. 이전의 CNN은 선형적으로 분리가능한 상황에서 효과적이지만 좀 더 효과적인 결과를 위해서 대부분 input에 대해서 비선형적으로 학습 할 수 있어야한다. 따라서 저차원의 결합으로 고차원의 feature를 만드는 것보다 고차원으로 결합하기 전에 저차원에서 추상화를 진행하는 것이 더 중요하다고 이야기 한다.
    
     ****
    
    ### Network In Network
    
    여기부분은 잘 이해가 안되는데
    
    함수 추정기로 RBF와 MLP가 있는데 해당 논문에서는 MLPconv Layers를 채택한 이유를 설명한다.
    
    - MLP(multilayer percetron)
        
        학습과 결과에 시간이 많이 사용되는 복잡한 관계에 대해 사용된다.
        
    - RBF(Radial Basis Function)
        
        MLP에 비해서 학습 및 결과가 안좋아 질 수 있다
        
        딥러닝에서의 RBF 뉴럴네트워크랑 gaussian basis function을 이용하는 것으로 정규분포의 선형 결합으로 데이터의 분포를 근사하는 것을 의미한다.
        
        방사형 구조를 기본으로 하는 네트워크를 이야기한다.
        
        1. 은닉층이 1개다.
        2. 유클리디안 거리를 사용한다.
        3. 역전파 알고리즘을 사용한다.
        4. 안정성 판별이 가능하다.
    - MLP 와 RBF의 차이
        
        두 부분의 차이점은 숨겨진 단위가 네트워크의 이전 계층에서 나오는 결과를 결합하는 방법에 있다.
        
        MLP는 내부 값을 사용하는 반면 RBF는 유클리드 거리를 이용한다.
        
    
    논문에서는 RBF와 MLP중 MLP를 선택한 이유가 2가지 있다.
    
    1. MLP는 역전파로 훈련된 CNN과 양립이 가능하다.
    2. MLP는 feature를 새사용하는 심층 모델 일 수 있다.
    
    ![[Network In Network-2.png]]
    
    위의 수식이 mlpconv layer의 계산 과정인데 relu를 사용한 것이다.
    
    여기서 (2)가 cascaded cross channel parametric pooling과 같다는 것이 중요하다.
    
    - CCCP란?
        
        일반적인 풀링과는 다르게 전체 채널을 pooling시켜 차원을 감소시키는 효과를 가진다. 따라서 일반적인 논문에서 사용하는 1x1 Conv layer와 같은 역할을 할 수 있게 된다.