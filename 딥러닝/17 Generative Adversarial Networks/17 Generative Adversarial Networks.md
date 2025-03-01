생성적 적대 신경망
생성 모델(Generative model): 머신러닝 알고리즘을 사용해 주어진 학습 데이터 집합으로부터 확률 분포를 학습한 후, 해당 분포로부터 새로운 예제를 생성하는 모델

생성 모델은 $p(x\mid w)$ 분포의 관점으로 이해 가능
많은 경우 조건부 생성 모델 $p(x\mid c,w)$에 관심을 가짐
$x$: 데이터 공간의 벡터
$w$: 학습 가능한 매개변수
$c$: 조건 변수 벡터

ex. 동물 이미지 생성 모델에서 $c$는 고양이 또는 개라는 조건

이미지 생성과 같은 실제 응용 분야에서는 확률 분포가 매우 복잡
>딥러닝의 도입이 생성 모델의 성능을 극적으로 향상

![[17.0.1.png]]
판별자 신경망 $d(x,\phi)$가 실제 예제와 생성자 신경망 $g(z,w)$가 생성한 예제와 구별하도록 학습됨
생성자는 실제 이미지와 유사한 이미지를 생성해 **판별자의 오류를 최대화**하려 하고, 판별자는 실제와 합성 예제를 더 잘 구별할 수 있도록 그 **오류를 최소화**하려 함
## 17.1 Adversarial Training
적대적 학습

GAN의 핵심 아이디어: 생성자와 함께 공동으로 학습되는 판별자의 도입
해당 판별자는 생성자의 가중치를 업데이트하기 위한 신호를 제공함

판별자의 목표: 실제 예제와 생성자가 만들어낸 예제를 구별 -> 일반적인 분류 손실 함수를 최소화하는 방향으로 학습

생성자의 목표: 학습 데이터와 동일한 분포에서 예제를 생성해 위 분류 손실 함수를 최대화하는 방향으로 학습

>생성자와 판별자 네트워크가 서로 반대 방향으로 작동하기 때문에 적대적(adversarial)이라는 용어가 사용됨
>원래 비지도 학습 방식으로 풀어야 하는 밀도 모델링 문제(데이터의 분포를 파악하는 문제)를, GAN의 판별자 네트워크 덕분에 지도 학습처럼 접근할 수 있음
### 17.1.1 Loss function
이진 타겟 변수는 다음과 같이 정의됨
$$
\begin{align*}
t = 1, \quad & \text{실제 데이터,} \\
t = 0, \quad & \text{합성 데이터}
\end{align*}
$$

판별자는 로지스틱-시그모이드 활성화함수를 갖는 단일 출력 유닛을 가지고 있으며, 출력은 데이터 벡터 $x$가 실제일 확률을 나타냄
$$
P(t = 1) = d(\mathbf{x}, \boldsymbol{\phi})
$$
판별자의 손실함수는 표준 교차 엔트로피 오차 함수(standard cross-entropy error function)로 다음과 같음
$$
E(\mathbf{w}, \boldsymbol{\phi}) = -\frac{1}{N} \sum_{n=1}^{N} \left\{ t_n \ln d_n + (1 - t_n) \ln (1 - d_n) \right\}
$$
$d_n = d(x_n, \phi)$: 입력 벡터 $x_n$에 대해 판별자가 출력한 값

실제 예제($x_n)$와 합성 예제($g(z_n, w)$의 출력)를 모두 포함해서 학습함
$z_n$은 잠재 공간 분포 $p(z)$로부터 무작위로 샘플링된 값

이때 실제 예제의 타겟값은 1이고($t_n = 1$), 합성 예제의 타겟값은 0이므로($t_n=0)$, 다음과 같이 오류함수를 쓸 수 있음
$$
E_{\text{GAN}}(\mathbf{w}, \boldsymbol{\phi}) =
- \frac{1}{N_{\text{real}}} \sum_{n \in \text{real}} \ln d(\mathbf{x}_n, \boldsymbol{\phi}) 
- \frac{1}{N_{\text{synth}}} \sum_{n \in \text{synth}} \ln (1 - d(\mathbf{g}(\mathbf{z}_n, \mathbf{w}), \boldsymbol{\phi}))
$$
일반적으로 $N_{\text{real}} = N_{\text{synth}}$
역전파를 통한 기울기를 바탕으로 SGD를 통해 학습될 수 있음

$\phi$에 대해선 오차를 최소화하지만, $w$에 대해선 오차를 최대화하려는 적대적 학습이라는 점을 고려해 매개변수는 다음과 같이 업데이트 됨 (판별자는 오차를 작게, 생성자는 오차를 크게 만드는 방향)
$$
\begin{align*}
\Delta \boldsymbol{\phi} &= -\lambda \nabla_{\boldsymbol{\phi}} E_n(\mathbf{w}, \boldsymbol{\phi}) \\
\Delta \mathbf{w} &= \lambda \nabla_{\mathbf{w}} E_n(\mathbf{w}, \boldsymbol{\phi})
\end{align*}
$$
$E_n$: 미니배치에 대해 정의된 오차
실제 학습에선 매개변수 업데이트를 번갈아 수행함 (1 iter에 하나씩 업데이트)
생성자가 완벽한 해법을 찾으면 판별자는 항상 0.5의 출력을 낼 것

GAN이 훈련을 마치면 판별자는 버리고 생성자를 사용
잠재공간으로부터 샘플링하고 훈련된 생성자를 통해 새로운 예제 합성
이론적으로, 생성자와 판별자가 무한한 유연성(복잡도)를 가진다면, 완전히 최적화가 된 GAN은 생성 분포와 데이터 분포가 정확히 일치함

만약 조건부 GAN(conditional GAN)을 사용한다면 조건 변수 $c$에 따라 $p(x\mid c)$ 조건부 분포로부터 샘플을 생성할 수 있음 (ex. 합성 개 이미지 -> 개의 품종을 조건 변수로 사용 가능)
이 경우엔 생성자와 판별자 모두 $c$를 입력으로 받고 라벨이 있는 이미지 쌍으로 학습함

여러 클래스를 개별적으로 GAN 모델을 학습시키는 것과 비교해, 모든 클래스의 공유 표현을 학습할 수 있기 때문에 데이터를 보다 효율적으로 사용함
### 17.1.2 GAN training in practice
실제 GAN을 학습할 땐 표준 오차 함수 최소화와 달리, 학습 도중 목표 함수가 오르기도 하고 내리기도 하므로 진행 상황을 평가할 명확한 지표(metric)가 없음

#### 모드 붕괴(mode collapse)
GAN의 도전과제
생성자의 가중치가 학습 중에 모든 잠재 변수 $z$를 유효한 출력들의 한 부분 집합으로 매핑하도록 적응하는 현상
ex. MNIST 데이터셋에서 훈련된 GAN이 숫자 3만 생성하게 되면, 판별자는 숫자 3이 진짜인지 구별하지 못하지만, 생성자가 전체 범위의 숫자를 생성하지 않다는 점을 인식할 수 없음

#### 빠른 학습 속도를 위한 방법들
![[17.1.1.png]]
$p_{\text{Data}}(x)$: 고정되었지만 알려지지 않은 데이터 분포
$p_\text{G}(x)$: 초기 생성 분포
$d(x)$: 기존 판별자 함수
$\tilde{d}(x)$: 스무스한 판별자 함수

생성 예제들의 판별 값인 $d(g(z,w),\phi)$가 0에 가까운 값을 가지게 되기 때문에, 생성자의 매개변수 $w$에 대한 작은 변화는 판별자의 출력에 거의 영향을 주지 않아 기울기가 작아지고 학습이 느려짐
>판별자 함수를 부드럽게 만들어 이를 해결할 수 있음
>생성자의 훈련을 빠르게 하는 강한 기울기를 제공하는 방식

최소제곱 GAN(least-squares GAN)은 판별자가 (0,1) 범위의 확률 대신 실수(real-valued) 출력을 내도록 변경하고, 교차 엔트로피 오류 함수를 제곱 오차 합(sum-of-squares error) 함수로 대체함으로써 부드럽게 만드는 방법을 사용

인스턴스 노이즈(instance noise) 기법을 적용하여 실제 데이터와 합성 샘플 모두에 가우시안 노이즈를 추가해 판별자 함수를 부드럽게 만들기도 함

아래와 같이 오차 함수의 생성자 항을 다음과 같이 수정하는 방법도 제안됨
$$
- \frac{1}{N_{\text{synth}}} \sum_{n \in \text{synth}} \ln (1 - d(\mathbf{g}(\mathbf{z}_n, \mathbf{w}), \boldsymbol{\phi}))
$$
$$
\frac{1}{N_{\text{synth}}} \sum_{n \in \text{synth}} \ln d(\mathbf{g}(\mathbf{z}_n, \mathbf{w}), \boldsymbol{\phi})
$$
첫번째는 이미지가 가짜일 확률을 최소화, 두번째는 이미지가 진짜일 확률을 최대화
만약 생성 분포 $p_\text{G}(x)$가 실제 데이터 분포 $p_{\text{Data}}(x)$와 크게 다르면 $d(g(z,w)$는 0에 가까워지므로 첫번째의 기울기는 매우 작지만, 두번째는 큰 기울기를 가져 더 빠르게 학습됨

#### 워서슈타인 GAN(Wasserstein GAN)
생성 분포가 데이터 분포로 이동하도록 보장하는 직접적인 방법으로, 두 분포가 데이터 공간에서 얼마나 떨어져 있는지를 반영하도록 오차 함수를 수정함

워서슈타인 거리(Wasserstein distance), 또는 earth mover’s distance를 사용하여 이를 측정할 수 있음

생성 분포를 땅 더미로 생각하고, 이 땅을 옮기면서 데이터 분포를 구성한다고 가정
워서슈타인 거리는 옮긴 땅의 총량에 평균 이동 거리를 곱한 값임

해당 거리를 직접 구현하는 건 어려움
>판별자가 실수 값을 출력하도록 하고 $x$에 대한 판별자 함수 $d(x,\phi)$의 기울기 $\nabla_x d(x,\phi)$를 가중치 클리핑(weight clipping)을 사용하여 제한하는 방식으로 근사

기울기에 대한 패널티를 도입해 개선한 방법인 그래디언트 패널티 워서슈타인 GAN(gradient penalty Wasserstein GAN)의 오차 함수는 다음과 같음
$$
E_{\text{WGAN-GP}}(\mathbf{w}, \boldsymbol{\phi}) =
- \frac{1}{N_{\text{real}}} \sum_{n \in \text{real}} 
\left[ \ln d(\mathbf{x}_n, \boldsymbol{\phi}) 
- \eta \left( \|\nabla_{\mathbf{x}_n} d(\mathbf{x}_n, \boldsymbol{\phi})\|^2 - 1 \right)^2 \right]
+ \frac{1}{N_{\text{synth}}} \sum_{n \in \text{synth}} \ln d(\mathbf{g}(\mathbf{z}_n, \mathbf{w}), \boldsymbol{\phi})
$$
$\eta$: 패널티항의 상대적 중요도를 제어하는 하이퍼파라미터

##### 이게 왜 근사하는 방식인지에 대한 설명 (by. GPT)
워서슈타인 거리는 두 확률 분포 간의 차이를 '땅(질량)'을 옮기는 비용으로 측정하는 개념입니다. 이를 GAN에 적용하려면, 데이터 분포 $p_{\text{data}}(x)$와 생성 분포 $p_G(x)$ 사이의 거리를 최소화하려고 하는데, 이 거리를 정확하게 계산하기는 어렵습니다.

여기서 중요한 이론적 배경은 **Kantorovich-Rubinstein duality**입니다. 이 정리에 따르면, 워서슈타인 거리는 모든 1-Lipschitz 함수 $f$에 대해
$$
W(p_{\text{data}}, p_G) = \sup_{\|f\|_{L \leq 1}} \mathbb{E}_{x \sim p_{\text{data}}} [f(\mathbf{x})] - \mathbb{E}_{x \sim p_G} [f(\mathbf{x})]
$$
와 같이 표현할 수 있습니다. 이 식은 "최대화" 문제로 바뀌게 되며, 여기서 1-Lipschitz 조건은 $f$의 변화율(기울기)이 1을 넘지 않도록 제한하는 조건입니다.

GAN에서는 이 1-Lipschitz 함수 $f$를 판별자 네트워크 $d(x,\phi)$로 근사합니다. 그런데 신경망이 1-Lipschitz 조건을 만족하도록 보장하는 것은 어려우므로, 초기 워서슈타인 GAN에서는 **가중치 클리핑(weight clipping)** 을 사용하여 판별자 네트워크의 가중치가 일정 범위 내에 머물도록 제한합니다. 이렇게 하면 네트워크 전체가 어느 정도 Lipschitz 조건을 따르게 되어, 위의 최대화 문제에 가까운 형태로 최적화할 수 있습니다.

요약하면:
- **워서슈타인 거리**는 두 분포 사이의 '이동 비용'을 의미합니다.
- **Kantorovich-Rubinstein duality**를 통해, 이 거리를 1-Lipschitz 함수를 최적화하는 문제로 변환할 수 있습니다.
- GAN에서는 판별자 네트워크로 이 함수를 근사하고, **가중치 클리핑**을 통해 1-Lipschitz 조건을 간접적으로 강제함으로써 워서슈타인 거리를 근사하게 됩니다.

이러한 방식으로 워서슈타인 GAN은 기존 GAN보다 안정적이고 의미 있는 학습 신호를 생성할 수 있게 됩니다.
## 17.2 Image GANs
GAN이 가장 널리 사용되고 성공적인 응용 분야 중 하나는 이미지 생성
### transpose convolution
![[17.2.1.png]]
판별자는 이미지를 받아 스칼라 확률 값을 출력하므로, 표준 컨볼루션 네트워크를 사용하는 것이 적합
생성자는 낮은 차원의 잠재 공간을 고해상도 이미지로 매핑해야 하므로, 전치 컨볼루션(transpose convolution)을 기반으로 한 네트워크가 사용
### 고품질 이미지를 얻기 위한 방법
![[17.2.2.png]]
고품질 이미지를 얻기 위해 생성자, 판별자를 모두 낮은 해상도에서 시작하여 점진적으로 확장시키는 방법이 제안됨
해당 접근 방식은 학습 속도를 높이고 4×4 크기의 이미지에서 시작하여 1024×1024 크기의 고해상도 이미지를 합성할 수 있게 함
위 사진은 클래스 조건부 이미지 생성을 하는 모델인 BigGAN로 해당 방법을 사용함
### 17.2.1 CycleGAN
해당 모델은 GAN의 변형 중 하나로, 딥러닝 기법이 분류나 밀도 추정과 같은 전통적인 작업을 넘어 다양한 문제를 해결하는 데 어떻게 적용될 수 있는지를 보여줌

![[17.2.3.png]]
사진을 모네 그림으로 변환하거나, 모네 그림을 사진으로 변환하는 문제
CycleGAN의 목표는 사진의 도메인 $X$와 모네 그림의 도메인 $Y$ 사이에 각각의 일대일 대응 관계를 학습하는 것
이를 위해 CycleGAN은 두 개의 조건부 생성자 $g_X$​와 $g_Y$​, 그리고 두 개의 판별자 $d_X$​와 $d_Y$​를 사용

생성자 $g_X(y, w_X)$는 입력으로 그림 $y \in Y$를 받아 해당하는 합성 사진을 생성하고, 판별자 $d_X(x, \phi_X)$는 합성 사진과 실제 사진을 구분

생성자 $g_Y(x, w_Y)$는 사진 $x \in X$를 입력으로 받아 합성 그림 $y$를 생성하고, 판별자 $d_Y(y, \phi_Y)$는 합성 그림과 실제 그림을 구분

따라서 $d_X$​는 $g_X$​가 생성한 합성 사진과 실제 사진의 조합으로 학습되고, $d_Y$​는 $g_Y$​가 생성한 합성 그림과 실제 그림의 조합으로 학습됨

#### 사이클 일관성 오차
만약 이 아키텍처를 표준 GAN 손실 함수를 사용하여 훈련한다면, 생성된 그림이 대응하는 사진과 비슷해지도록 하는 강제력이 전혀 없게 됨
>사이클 일관성 오차(cycle consistency error) 를 손실 함수에 도입
>ex. 사진을 그림으로 변환 후 다시 사진으로 복원했을 때 원래의 사진과 유사하게 만드는 것

![[17.2.4.png]]
$$
E_{\text{cyc}}(\mathbf{w}_X, \mathbf{w}_Y) =
\frac{1}{N_X} \sum_{n \in X} \|\mathbf{g}_X(\mathbf{g}_Y(\mathbf{x}_n)) - \mathbf{x}_n\|_1 \\
+ \frac{1}{N_Y} \sum_{n \in Y} \|\mathbf{g}_Y(\mathbf{g}_X(\mathbf{y}_n)) - \mathbf{y}_n\|_1
$$
위와 같이 정의된 사이클 일관성 오차 항은 일반적인 GAN 손실 함수에 추가되어 다음과 같은 전체 손실 함수가 나타남
$$
E_{\text{GAN}}(\mathbf{w}_X, \boldsymbol{\phi}_X) + E_{\text{GAN}}(\mathbf{w}_Y, \boldsymbol{\phi}_Y) + \eta E_{\text{cyc}}(\mathbf{w}_X, \mathbf{w}_Y)
$$
#### 표현 학습에서의 GAN
잠재 공간이 의미적으로 조직되어 특정 속성을 반영하는 구조를 가지게 됨

![[17.2.5.png]]
위 사진은 잠재 공간 내에서 부드러운 경로를 따라가며 해당하는 이미지들을 생성한 예시
이미지들이 부드럽게 전환되는 모습을 볼 수 있음

더 나아가, 잠재 공간에서 의미 있는 변환(semantically meaningful transformations)에 해당하는 방향들을 식별할 수 있음

얼굴 이미지의 경우 한 방향은 얼굴의 회전 변화에 대응할 수 있고, 다른 방향들은 조명 변화나 미소의 정도 변화 등에 대응할 수 있음

위와 같은 표현은 **분리된(disentangled) 표현**이라고 하며, 특정 속성을 지정하여 새로운 이미지를 생성할 수 있게 함

아래 에시는 성별과 안경 착용 여부와 같은 의미적 속성이 잠재 공간 내의 특정 방향에 대응함을 보여줌
![[17.2.6.png]]