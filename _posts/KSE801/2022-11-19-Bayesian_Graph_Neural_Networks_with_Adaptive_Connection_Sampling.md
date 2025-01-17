---
title:  "[ICML 2020] Bayesian Graph Neural Networks with Adaptive Connection Sampling"
permalink: Bayesian_Graph_Neural_Networks_with_Adaptive_Connection_Sampling.html
tags: [reviews]
---

### [Review] Bayesian Graph Neural Network with Adaptive Connection Sampling



#### 0. 본 논문을 온전히 이해하기 위해 알면 좋을 사전 지식들

- Variational Inference

- Random process - Indian buffet process

- 미분 불가능한 값들을 Neural Network에 적용하기 위해 필요한 조치  

경고 - 본 논문은 수식이 매우 많이 나옵니다. 각 수식을 이해하기에 앞서, 각 파트별로 정리한 내용을 먼저 참고하신 후 수식을 보실 것을 권장합니다. 





#### 1. 문제 정의

- Graph Neural Networks(이하 GNN)은 그래프에서 좋은 representation learning 측면에서 좋은 성과를 많이 내왔다.

- 하지만 GNN은 아래 2가지 한계점이 있다.
  
  1. 과적합 및 Over-smoothing 으로 인해 깊은 레이어층을 쌓았을 때 성능이 악화된다.
  
  2. 그래프 구조의 복잡성으로 인해 불확실성을 측정할 수 있는 Bayesian neural Network(이하 BNN) 적용에 어려움이 있다. 

#### 

#### 2. 연구 동기

- 과적합을 해결하는 기존의 다양한 방법이 존재한다. 하지만 Over-smoothing 문제까지 함께 해결하는 방법이 없다. 따라서 둘을 함께 해결하는 방법을 찾을 필요가 있다.
  
  - Deep neural network에서 과적합을 해결하기 위해 Dropout을 많이 사용한다. 하지만 Dropout은 GNN에서  Laplacian smoothing을 활용하는데, 이는 층을 깊게 했을 때 주변 노드로의 표현값과 유사해지는 Over-smoothing 문제를 일으킨다. 
  
  - 최근(Rong et al., 2019) 연구에서 Dropout와 DropEdge를 함께 사용했을 때 과적합과 Over smoothing 문제를 일부 해결하였다. 하지만 그 외에 Dropout 기반 방식에 대한 연구는 따로 없다.
  
  - 또한 기존의 방법에서는 Dropout rate를 하나의 상수 값처럼 다뤘었다. 본 연구에서는 Dropout rate를 데이터셋에 맞춰 최적화할 것이다.  
  
  <br>

- GNN의 정보를 포괄적으로 활용하며 일반화할 수 있는 BNN 적용 방법이 아직 없다.
  
  - Bayesian은 1) 환경의 노이즈로 측정 향상을 통해 개선 가능한 aleatoric uncertainty (= $P(D|H)$ : 우도)과 모델의 적합도로 데이터 확보를 통해 개선 가능한 Epistemic uncertainty( = $P(H|D)$ : 사후 확률)을 분리하여 고려할 수 있다는 강점이 있다.
  
  - 최근들어 Bayesian을 Neural Network에 적용하는데 성공했다. 이때 해결해야하는 문제는 2가지였다.  
  
  - 첫째, Stochastic한 변수들에 대해 미분을 어떻게 적용할 것인가. Neural Network은 Gradient descent을 통해 모델을 학습한다. 따라서 미분이 불가능하다면 Neural Network에 적용할 수 없다. 하지만 1) score-function gradient estimator, 2) continuous relaxation of the drop masks 방법 등을 통해 Stocastic한 변수들도 미분할 수 있게 되었다. 
  
  - 두번째로, Bayesian에서 사후 확률(Posterior) 계산에 필요한 Evidence form($\int P(D|H) P(H) dH))$을 어떻게 구할 것인가. 적분식을 풀기 위해선 어마어마한 계산량을 필요로 한다. 따라서  실제로 사용하기엔 제약이 있었다. 하지만 Sampling 기반의 MCMC 방법을 통해 Evidence form에 대한 근사값을 계산해내어, 최근 Convolution neural network(이하 CNN)에 성공적으로 적용한 연구가 발표되었다(Gal & Ghahramani, 2015).   
  
  - 이후 GNN에서도 관측된 그래프를 임의의 랜덤 그래프 모델에서 파라미터를 바꾼 형태로 고려하여 BNN을 적용한 연구가 있다(Zhang et al. 2019). 하지만 이 방식은 막대한 계산양 뿐만 아니라 어떤 랜덤 그래프 모델을 설정하는 가가 성능에 큰 영향을 미쳐, 다른 문제에 동일하게 적용할 수 없다. 또한 불확실성을 학습함에 있어서 그래프의 노드를 무시하고, 그래프의 위상만 고려했다는 한계가 있다.
  
  <br>

- 따라서 이 연구는 <mark>1) 과적합 및 Over-smooth를 방지하고자 Graph DropoutConnect을 제안</mark>하며, <mark>2) BNN 활용 간 Graph 전반의 정보를 사용하도록  할 것</mark>이다. 마지막으로 <mark>3) Dropout rate를 데이터에 맞춰 적응시켜 모델 성능을 향상시킬 것이다.</mark>
  
  <br>

#### 3. 방법

- 이 연구에서 제시하는 방법은 크게 3가지다. 
1. **Dropout 방식을 일반화한 Graph DropoutConnect(이하 GDC)를 제안한다.**
   
   > ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/1.png)
   > 
   > $H^{(l)}$ : $l_{th}$ Hidden layer of GNN
   > 
   > $\sigma(.)$ : Activation function
   > 
   > $f_l$ : the number of features of $l_{th}$ layer
   > 
   > $\mathfrak{N}$(A) : normalizing operator i.e. $\mathfrak{N}(A) = I_N + D^{-\frac{1}{2}} AD^{-\frac{1}{2}}$
   > 
   > $\bigodot$ : Hadamard product
   > 
   > $Z^{(l)}$ : Dropout 형태를 결정하는 Matrix.  Random binary matrix
   > 
   > - $z_{i,j}$ 값이 {0, 1} 중에서 선택되는 것은 베르누이 분포를 따른다고 가정한다.
   > 
   > $W^{(l)}$ : Feature translate $l$ layer to $l+1$ layer
   
   <br>
   
   - Dropout Connect의 일반화 증명
     
     - $Z_{i,j}^{(l)}$ 의 값을 어떻게 설정하냐에 따라서 Dropout, Drop edge, Node sampling의 결과가 나온다.
     
     > Ex 1 - Drop out) $Z^{(l)} \in$ $\{0, 1\}^{n \times f_l}$
     > 
     > ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/3.png)
     
     > Ex 2 - Drop edge) $Z^{(l)} \in $ $\{0,1\}^{n \times n}$
     > 
     > ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/4.png)
     
     > Ex 3 - Node Sampling) $diag(z^{(l)}) \in$ $\{0,1\}^{n \times n}$
     > 
     > ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/5.png)
     
     → GDC 수식을 통해 3가지 Dropout 방법 모두를 표현할 수 있다. 따라서 <mark>GDC를 최적화하는 것은 3가지 Dropout의 조합을 최적화하는 것과 동일하다. </mark>
   
   <br>
   
   - 추가로 GDC는 그래프 노드 예측을 위해 Neighborhood aggregation을 할 때, 각 채널별로 독립적으로 시행한다. 
     
     - 채널간 연결을 유지한다면 채널 간의 Over smoothing 현상이 일어날 수 있다.
     
     - 이를 방지하고자 채널별로 GDC를 독립적으로 수행할 필요가 있다.

<br>

**<3-1.정리>**

=> Dropout과 Dropedge의 조합이 과적합 및 Oversmoothing 문제를 해결하였음을 이전 연구를 통해 확인했다. 따라서 Dropout과 Dropedge, Node sampling 의 조합을 최적화한 GDC는 더욱 효과적으로 해결할 수 있다. 또한 각 채널 간의 Over smoothing을 방지하기 위해 채널 간 GDC를 독립적으로 수행한다. 

<br>

2. **GDC sampling이 Message passing과 동일하게 볼 수 있다는 점을 통해 Graph의 노드 정보까지 활용하는 BNN 모델을 만든다**
   
   - GDC에서는 각 GNN의 Layer마다 Masking을 통해  학습에 활용할 Adjacency Node를 Sampling한다. 이는 GNN 모델에서의 Message passing로 해석될 수 있다. 
     
     - GDC 식을  $l+1$ 층의 Hidden value v ($h_v^{(l+1)}$)로 범위를 좁혀보자  
     
     > ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/6.png)
     > 
     > > $c_v$ : Constant derived from the degree of node v
     > > 
     > > $z_{vu}^{(l)}\in {0,1}^{1 \times f_l}$ : $l$ layer 층의 node u 의 값 중 어떤 것을 $l+1$ layer 층에 있는  node v 로 보낼지 결정하는 Mask row vector. 
     
     > 이 식을 풀어내면 아래와 같다. 
     > 
     > ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/7.png)$(s.t. W_{vu}^{(l)} := z_{uv}^{(l)}W^{(l)}
     > )$
     > 
     > > $c_v$ 는 계산을 간편하게 해주기 위해서 생략. 
     
     - Message passing에 대한 식은 아래와 같다. 
     
     > $h_v^{l+1} = \sigma(W_l AGG(\{h_u^{(l)} | u \in N(v)\}))$
     
     - 즉, 위의 식은<u> 노드 v에 대해서 Neighborhood에 있는 노드들을 통해 Message passing 하는 것으로 해석</u>할 수 있다. 
       - Message passsing은 각 노드들의 정보를 반영한다. 
       - <mark>즉, GDC 방법 또한 Node 정보를 반영한다. </mark>
   
   <br>
   
   - 더 나아가 우리는 GDC를 통해 구한 $h_v^{l+1}$ 값이 evidence form $p_\theta(w)$의 근사값$q_\theta(\theta)$임을 유추할 수 있다 
     
     - 우린 Gal(2017)연구를 통해 "GNN 모델의 Message passing은 BNN의 evidence form $p_\theta(w)$을 근사하는 것과 동일함" 을 알고 있다.
     
     - 정리하면, GDC를 통한 Random masking을 통한 $h_v^{l+1}$ 값은 GNN 모델의 Message passing 의 한 형태로 볼 수 있다. 그리고 Message passing 결과는 GAL 연구를 통해 BNN의 Evidence form을 근사한 결과와 동일함을 안다. 
     
     - <u>즉, GDC을 통해 구한 값은 Evidence form의 근사값$q_\theta(w)$가 된다.</u>
       
       > Tip1. 실제 분포인 $p_\theta(w)$을 구하긴 어렵다. 따라서 이와 유사하지만 우리가 구하기 쉬운 분포를 $q_\theta(w)$로 설정한다.    
       > 
       > Tip2. 모델을 보다 쉽게 표현하기 위해 활용하는 변수들을 Variational paramter라고 부르며, 이를 활용한 예측을 Variational inference라 부른다.  $q_\theta(w)$ 의 경우 Variational distribution이라고 부른다. 
   
   <br>
   
   - 이때, 근사값인 $q_\theta(w)$가 본래 분포인 $p(w)$ 에서 많이 벗어나지 않도록 Kullback-Leibler(이하 KL) divergence를 추후 Variational inference 간 규제항으로 삼는다. 
     
     > KL divergence : $KL(q_{\theta_l}(W_e^{(l)}) || p(W_e^{(l)}))$ 
     > 
     > let $q_{\theta_l}(W_e^{l}) = \pi_l \delta(W_e^{(l)} -0) + (1- \pi_l)\delta(W_e^{(l)} - M^{(l)})$
     > 
     > > $\delta$ : dirac delta function 
     
     - 이때, 사전 분포를 discrete quantisd Gaussian 분포라고 가정한다. 이는 KL term에 대한 Analytically한 평가를 가능하게 한다.(Gal et al, 2017).
     
     - 각 층별 분포를 independent하다고 가정했기 때문에 Factorize를 통해 $q_\theta(W)$ 와 $p(W)$의 계산을 쉽게 할 수 있다. 
       
       > $KL(q_{\theta})(w || p(w)$ =$\sum_{l=1}^L\sum_{e=1}^{|\varepsilon|} KL(q_{\theta_l}(W_e^{(l)}) || p(W_e^{(l)}))$
       > 
       > ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/8.png)
       > 
       > $M^{(l)}$ : network weight parameters
       > 
       > $\mathcal{H}(\pi)$ : Entropy of success rate $\pi$
     
     <br>
   
   - 마지막으로 Loss 함수를 최소로 만드는 값 $\theta = \{M, \pi\}$ 을 미분을 통해 구한다
     
     > ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/9.png)
     > 
     > > $Y_0$ : the collection of the available labels for the observed nodes 
     
     - $M$ 에 대한 최적화는 Monte Carlo sampling을 통해서 계산할 수 있다.
     
     - 하지만 $\pi$는 Binary 값으로 미분이 불가능하여, 추가 조치없인 Gradient descent 방식을 적용할 수 없다. 
       
       - 여기에선 최근에 개발된 Augment-REINFORCE-Merge(ARM) 방법(Yin & zhou, 2019; Uin et al., 2020)을 채택한다.
   
   <br>
   
   **<3-2. 정리>** 
   
   => GDC 방식을 통해 $l+1$ layer의 값을 계산하는 것은 GNN의 Message passing 방법으로 해석되며, 이는 GDC가 Graph의 노드 정보를 반영하고 있음을 의미한다. 다음으로 BNN의 모델을 구성하기 위해 필요한 값들을 하나씩 구한다. 먼저 Bayesian의 Posterior을 구하기위해 필요한 evidence form의 근사값 $q_\theta(w)$은  Gal(2017)을 통해 Message passing 의 결과, 즉 GDC의 결과임을 알 수 있다. 이후 모델 학습을 위해 Loss 함수를 정의하며, 이때 Binary 값을 미분가능하게 만들기 위해 Augment-Reinforce-Merge 방법을 적용한다.
   
   <br>

3. **Droprate($\pi_l$)을 Hierarchical beta-Bernoulli 로 모델링하여 주어진 데이터셋에 최적화한다**
   
   - Beta-Bernoulli process의 Marginal representation이 Indian Buffet Process 로 보일 수 있다(Ghahramain & Grifiths, 2006). 
     
     > Tip1. Neural network에서 적절한 Layer의 개수를 최적화하기 위해 Random process를 활용한다. 
     > 
     > Tip2. Indian Buffet process는 Random process에서 distribution을 유추하기 위해 적용하는 Sampling base Scheme이다. 
   
   - 이를 활용하여, binary random mask를 아래와 같이 제안하겠다.
     
     > ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/10.png)
     > 
     > $a_e^{(l)}$ : element of the adjacency matrix A corresponding to edge e 
     > 
     > $\hat a_e^{(l)}$ : element of the matrix $\hat A^{(l)} = A \bigodot Z^{(l)}$ 
   
   <br>
   
   - Hierarchical beta-Bernoulli GDC 계산을 위한 방법으로 Gibbs sampling이 있지만, 큰 데이터 셋에 적용하기엔 계산양이 많다. 따라서 계산양이 적은 Variational inference 방식을 제안한다. 
     
     - Variational distribution을 $q(Z^{(l)}, \pi_l) = q(Z^{(l)}|\pi_l) q(\pi_l)$ 으로 정의한다.
       
       > $q(\pi_l) : q(\pi_l; a_l, b_l) = a_lb_l\pi_l^{a_l-1}(1-\pi_l^{a_l})^{b_l-1} s.t. a_l, b_l > 0$
       > 
       > $l$ 층의 beta 분포를 대체하기 위해 Kumaraswamy distribution로 가정
       
       - 앞서 dicrete quantised Gaussian 분포로 가정했기 때문에 각 egde가 독립이다. 따라서 $q(Z^{(l)}|\pi_l) = \prod_{e=1}^{|\varepsilon|}q(z_e^{(l)}|\pi_l)$로 표현할 수 있다. 
     
     - 정의한 $\pi_l$에 대한 Bernoulli distribution을 KL-term 및 Loss함수에 적용한다. 
       
       > ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/11.png)
       
       > KL-term의 첫번째 항은 p,q 간에 동일한 분포를 가지고 있어 0이 된다. 따라서 두번쨰 항만 계산하면 된다.  
       > 
       > ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/12.png)
       > 
       > > $\gamma $ : Euler-Mascheroni constant
       > > 
       > > $\Psi(.)$ : Digamma function 
     
     - 이때 Random mask의 값이 Discrete하기 때문에 KL-term을 바로 미분할 수 없다. 
       
       - 가능한 방법 중 하나로 concrete distribution relaxation (Jang et al., 2016; Gal et al., 2017) 이 있으나, 이 방법은 큰 분산을 가져 모델의 정확도를 떨어트릴 수 있다. 
       
       - 따라서 이 연구에서는 Boluki 연구(Boluki et al.)와 동일하게  ARM estimator을 적용함으로써 문제를 해결하겠다. 

<br>

**<3-3. 정리>**

=> Droprate를 최적화해주기 위해 Neural network로 모델링한다. Neural Network의 Layer 개수 k개를 최적화하기 위해 Random Process를 적용한다. 이때 각 노드별 Masking 유무는 0,1 이므로 Beta-Binomial Process 형태를 띈다. Beta-Binomial process을 통해 Beta 분포를 최적화하고자 Indian buffet process을 통해 distribution을 모델링한다. 이후 Loss 함수을 통해 최적화해주며, 이 과정에서 미분 불가능한 값에 대해  ARM-estimator을 적용해준다. 



<br>



#### 4. 실험

- 실험 환경
  
  - Dataset : Cora, Citeseer, Cora-ML dataset
  
  - Condition 
    
    - Learning rate : 0.005 
    
    - Layer : 2층 이상. paramener 학습을 위해 50 warm-up training. 
    
    - Dimension of output feature : 128 
    
    - L2-regularization foctor : $5 \times 10^{-3}$, $10^{-2}$ and $10^{-3}$ for each dataset 
    
    - Temperature of concrete distribution : 0.67
    
    <br>

- **실험 1 : Drouout 방식에 따른 노드 분류 정확도 비교**
  
  - Baseline : GCN with Dropout(DO), GCN with DropEdge(DE), GCN with DO and DE 
  
  ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/15.png)
  
  > BBDE : Beta-Bernoulli DropEdge. 
  > 
  > - DropEdge 기반 모델에 최적화된 Dropout rate를 활용 
  > 
  > BBGDC : Beta-Bernoulli Graph Dropout Connect 
  > 
  > - GDC 기반 모델에 최적화된 Dropout rate를 활용
  
  -> BBGDC 방법이 Dropout 방법 중 SOTA 성능을 달성한다. 대부분의 경우 Dropout과 DropEgde을 개별적으로 적용했을 때보다 같이 사용했을 때 성능이 향상됨을 알 수 있다. 또한 모델링을 통해 Dropout rate을 최적화했을 때 항상 성능이 항상된다. BBGDC을 제외한 다른 방법들은 항상 레이어 층을 늘렸을 때 성능이 떨어지나, BBGDC는 Citeseer의 경우를 제외하고 성능이 향상되었다. 
  
  <br>

- **실험 2 : Dropout rate 최적화 방식에 따른 분류 정확도 비교**
  
  - Baseline : Bernoulli DropEdge with ARM(BDE-ARM), Beta-bernoulli DropEdge with ARM(BBDE-ARM)
  
  ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/16.png)
  
  -> 최적화한 Dropout rate를 Dropout 방식, Dropedge 방식, GDC 방식 모두에 적용해봤을 때 GDC가 최고의 성능을 달성했다. 
  
  <br>

- **실험 3 : Dropout 방식에 따른 불확실성 측정 결과** 
  
  > Base linde : GCN-DO 
  > 
  > Metric : Patch Accuracy vs Patch Uncertainty(PAvPU)(Mukhoti & Gal, 2018) 
  
  ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/17.png)
  
  -> PAvPU의 값이 높을수록 예측이 잘 맞다. 즉, GCN-BBGDC가 모든 Threshold에서 높은 성과를 내고 있음을 확인할 수 있다. 
  
  <br>

- **실험 4 : Dropout 방식에 따른 과적합 및 Over-fitting 방지 효과 확인**
  
  > Baseline : GCN-DO
  > 
  > Metric : Total variation(TV). TV 은 smoothness를 측정한다(Chen et al., 2015). TV 수치가 낮아질수록 over-smoothing 을 의미한다. 
  
  ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/18.png)
  
  -> GCN-DO는 학습 횟수(epoch)가 증가함에 따라 첫번째 hidden 층에선 TV가 조금씩 증가하고, 둘째 층에선 TV가 줄어든다. 반면 GCN-BBGDC는 에포크가 증가함에 따라 첫째, 둘째 hidden 층에서 TV가 증가한다. 이러한 차이는 hidden 레이어 층의 개수를 늘렸을 때, GCD-DO는 성능이 악화되지만 GCN-BBGDC는 그렇지 않은 이유를 설명해준다. GDC의 구조 뿐만 아니라 Droprate의 Adaptive learning이 BBGDC 방법의 강건성을 제공한다.  
  
  <br>

- **실험 5 : GDC 방법 간 Block 적용에 따른 영향 확인**
  
  - GDC는 모든 input, output 특징 간에 개별적인 Mask를 고려하기 때문에 많은 메모리 용량을 필요로 한다. 하나의 해결 방법으로 특징들을 Block 화한다면 필요로 하는 메모리 양을 줄일 수 있다. Block에 따른 성능에 미치는 영향을 확인한다.
  
  ![](../../images/Bayesian_Graph_Neural_Network_with_Adaptive_Connection_Sampling/19.png)
  
  -> Block의 수가 늘어날 수록 정확도가 향상된다. 즉, Block의 개수를 정하는 것은 모델의 성능과 메모리 사용량을 Trade-off하는 요인임을 확인할 수 있다. 
  
  <br>

#### 5. 결론

- 본 연구는 GNN에 BNN을 적용한다. 기존에도 GNN에 BNN을 적용하려는 시도가 있었으나, 계산양이 많고 Graph의 구조적 정보만을 반영하여 한계가 많았다. 반면 본 연구의 모델은 GDC 방법을 통해 그래프의 노드 정보까지 반영한다는 점, Sampling 기반으로 evidence form을 계산함으로써 계산양을 줄여냈다는 점에서 차별점을 가진다. 

- 본 연구는 기존의 Dropout 방법을 일반화한 GDC 방법을 제안한다. 기존 연구에서 Dropout과 Dropedge 모델을 함께 사용했을 때 과적합 및 Oversmoothing 문제를 해소할 수 있었다. 본 연구에서는 GDC를 최적화함으로써 보다 효율적으로 과적합 및 Oversmoothing을 막을 수 있음을 보인다. 

- 본 연구는 Dropout rate을 모델링함으로써 주어진 데이터 셋에 대해 최적화한다.  최적화된 Dropout rate는 모델의 성능을 향상시키며, GDC 모델에 적용했을 때 성능이 가장 좋았다. 

<br>

---

##### Posting author information

- 강현구 (Hyeongu Kang)
  
  - GSDS at KAIST 
  
  - Machine learning method, Active learning, Semi-supervised learning
  
  - [GitHub](https://github.com/gusrn0505)
  
  

<br>

### 6. 참고 논문 및 추가 자료

- Reference 
  
  - Boluki, S., Ardywibowo, R., Dadaneh, S. Z., Zhou, M., and Qian, X. Learnable Bernoulli dropout for Bayesian deep learning. In Proceedings of the Twenty Third International Conference on Artificial Intelligence and Statistics,
    pp. 3905–3916, 2020.
  
  - Chen, S., Sandryhaila, A., Moura, J. M., and Kovacevic, J.Signal recovery on graphs: Variation minimization. IEEE
    Transactions on Signal Processing, 63(17):4609–4624, 2015.
    
    Gal, Y. and Ghahramani, Z. Bayesian convolutional neural networks with bernoulli approximate variational inferrence. arXiv preprint arXiv:1506.02158, 2015.
  
  - Gal, Y., Hron, J., and Kendall, A. Concrete dropout. In Advances in neural information processing systems, pp. 3581–3590, 2017
    
    Ghahramani, Z. and Griffiths, T. L. Infinite latent feature models and the Indian buffet process. In Advances in neural information processing systems, pp. 475–482, 2006.
  
  - Hajiramezanali, E., Dadaneh, S. Z., Karbalayghareh, A., Zhou, M., and Qian, X. Bayesian multi-domain learning for cancer subtype discovery from next-generation sequencing count data. In Advances in Neural Information
    Processing Systems, pp. 9115–9124, 2018.
  
  - Jang, E., Gu, S., and Poole, B. Categorical reparameterization with gumbel-softmax. arXiv preprint arXiv:1611.01144, 2016.
  
  - Mukhoti, J. and Gal, Y. Evaluating bayesian deep learning methods for semantic segmentation. arXiv preprint arXiv:1811.12709, 2018.
  
  - Rong, Y., Huang, W., Xu, T., and Huang, J. DropEdge: Towards the very deep graph convolutional networks for node classification, 2019.
  
  - Kumaraswamy, P. A generalized probability density function for double-bounded random processes. Journal of Uin et al., 2020)
  
  - Yin, M. and Zhou, M. ARM: Augment-REINFORCEmerge gradient for stochastic binary networks. In International Conference on Learning Representations, 2019.
  
  - Zhang, Y., Pal, S., Coates, M., and Ustebay, D. Bayesian graph convolutional neural networks for semi-supervised classification. In Proceedings of the AAAI Conference on Artificial Intelligence, volume 33, pp. 5829–5836, 2019.
  
  - Zhou, M., Chen, H., Ren, L., Sapiro, G., Carin, L., and Paisley, J. W. Non-parametric Bayesian dictionary learning for sparse image representations. In Advances in neural information processing systems, pp. 2295–2303, 2009

- Github : https://github.com/armanihm/GDC
