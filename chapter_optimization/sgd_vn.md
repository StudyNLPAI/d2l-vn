<!-- ===================== Bắt đầu dịch Phần 1 ==================== -->
<!-- ========================================= REVISE PHẦN 1 - BẮT ĐẦU =================================== -->

<!--
# Stochastic Gradient Descent
-->

# Hạ Gradient Ngẫu nhiên
:label:`sec_sgd`

<!--
In this section, we are going to introduce the basic principles of stochastic gradient descent.
-->

Trong phần này chúng tôi sẽ giới thiệu các nguyên tắc cơ bản của hạ gradient ngẫu nhiên.

```{.python .input  n=2}
%matplotlib inline
from d2l import mxnet as d2l
import math
from mxnet import np, npx
npx.set_np()
```

<!--
## Stochastic Gradient Updates
-->

## Bước Cập nhật trong Hạ Gradient Ngẫu nhiên

<!--
In deep learning, the objective function is usually the average of the loss functions for each example in the training dataset.
We assume that $f_i(\mathbf{x})$ is the loss function of the training dataset with $n$ examples, an index of $i$, and parameter vector of $\mathbf{x}$, then we have the objective function
-->

Trong học sâu, hàm mục tiêu thường là trung bình của các hàm mất mát cho từng mẫu trong tập huấn luyện.
Giả sử rằng $f_i(\mathbf{x})$ là hàm mất mát của tập huấn luyện có $n$ mẫu, với $i$ là chỉ số mẫu và vector tham số $\mathbf{x}$, thì ta có hàm mục tiêu


$$f(\mathbf{x}) = \frac{1}{n} \sum_{i = 1}^n f_i(\mathbf{x}).$$


<!--
The gradient of the objective function at $\mathbf{x}$ is computed as
-->

Gradient của hàm mục tiêu tại $\mathbf{x}$ được tính như sau


$$\nabla f(\mathbf{x}) = \frac{1}{n} \sum_{i = 1}^n \nabla f_i(\mathbf{x}).$$


<!--
If gradient descent is used, the computing cost for each independent variable iteration is $\mathcal{O}(n)$, which grows linearly with $n$.
Therefore, when the model training dataset is large, the cost of gradient descent for each iteration will be very high.
-->

Nếu hạ gradient được sử dụng, chi phí tính toán cho mỗi vòng lặp độc lập là $\mathcal{O}(n)$, tăng một cách tuyến tính với $n$.
Do đó, với tập huấn luyện lớn, chi phí của hạ gradient cho mỗi vòng lặp sẽ cao.

<!--
Stochastic gradient descent (SGD) reduces computational cost at each iteration.
At each iteration of stochastic gradient descent, we uniformly sample an index $i\in\{1,\ldots, n\}$ for data instances at random, 
and compute the gradient $\nabla f_i(\mathbf{x})$ to update $\mathbf{x}$:
-->

Hạ gradient ngẫu nhiên (_stochastic gradient descent_ - SGD) giúp giảm chi phí tính toán ở mỗi vòng lặp.
Ở mỗi vòng lặp của thuật toán này, ta lấy ngẫu nhiên một mẫu dữ liệu có chỉ số $i\in\{1,\ldots, n\}$, và tính gradient $\nabla f_i(\mathbf{x})$ để cập nhật $\mathbf{x}$:


$$\mathbf{x} \leftarrow \mathbf{x} - \eta \nabla f_i(\mathbf{x}).$$


<!--
Here, $\eta$ is the learning rate.
We can see that the computing cost for each iteration drops from $\mathcal{O}(n)$ of the gradient descent to the constant $\mathcal{O}(1)$.
We should mention that the stochastic gradient $\nabla f_i(\mathbf{x})$ is the unbiased estimate of gradient $\nabla f(\mathbf{x})$.
-->

Ở đây, $\eta$ là tốc độ học.
Ta có thể thấy rằng chi phí tính toán cho mỗi vòng lặp giảm từ $\mathcal{O}(n)$ của hạ gradient xuống còn hằng số $\mathcal{O}(1)$.
Nên nhớ rằng gradient ngẫu nhiên $\nabla f_i(\mathbf{x})$ là một ước lượng không thiên lệch của gradient $\nabla f(\mathbf{x})$.


$$\mathbb{E}_i \nabla f_i(\mathbf{x}) = \frac{1}{n} \sum_{i = 1}^n \nabla f_i(\mathbf{x}) = \nabla f(\mathbf{x}).$$


<!--
This means that, on average, the stochastic gradient is a good estimate of the gradient.
-->

Do đó, về trung bình, gradient ngẫu nhiên là một ước lượng tốt của gradient.

<!--
Now, we will compare it to gradient descent by adding random noise with a mean of 0 to the gradient to simulate a SGD.
-->

Bây giờ, ta sẽ so sánh với hạ gradient bằng cách thêm nhiễu ngẫu nhiên với trung bình bằng 0 vào gradient để mô phỏng một SGD.


```{.python .input  n=3}
def f(x1, x2):
    return x1 ** 2 + 2 * x2 ** 2  # Objective

def gradf(x1, x2):
    return (2 * x1, 4 * x2)  # Gradient

def sgd(x1, x2, s1, s2):  # Simulate noisy gradient
    global lr  # Learning rate scheduler
    (g1, g2) = gradf(x1, x2)  # Compute gradient
    (g1, g2) = (g1 + np.random.normal(0.1), g2 + np.random.normal(0.1))
    eta_t = eta * lr()  # Learning rate at time t
    return (x1 - eta_t * g1, x2 - eta_t * g2, 0, 0)  # Update variables

eta = 0.1
lr = (lambda: 1)  # Constant learning rate
d2l.show_trace_2d(f, d2l.train_2d(sgd, steps=50))
```


<!-- ===================== Kết thúc dịch Phần 1 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 2 ===================== -->

<!--
As we can see, the trajectory of the variables in the SGD is much more noisy than the one we observed in gradient descent in the previous section.
This is due to the stochastic nature of the gradient.
That is, even when we arrive near the minimum, we are still subject to the uncertainty injected by the instantaneous gradient via $\eta \nabla f_i(\mathbf{x})$.
Even after 50 steps the quality is still not so good.
Even worse, it will not improve after additional steps (we encourage the reader to experiment with a larger number of steps to confirm this on his own).
This leaves us with the only alternative---change the learning rate $\eta$.
However, if we pick this too small, we will not make any meaningful progress initially.
On the other hand, if we pick it too large, we will not get a good solution, as seen above.
The only way to resolve these conflicting goals is to reduce the learning rate *dynamically* as optimization progresses.
-->

*dịch đoạn phía trên*

<!--
This is also the reason for adding a learning rate function `lr` into the `sgd` step function.
In the example above any functionality for learning rate scheduling lies dormant as we set the associated `lr` function to be constant, i.e., `lr = (lambda: 1)`.
-->

*dịch đoạn phía trên*

<!--
## Dynamic Learning Rate
-->

## *dịch tiêu đề phía trên*

<!--
Replacing $\eta$ with a time-dependent learning rate $\eta(t)$ adds to the complexity of controlling convergence of an optimization algorithm.
In particular, need to figure out how rapidly $\eta$ should decay.
If it is too quick, we will stop optimizing prematurely.
If we decrease it too slowly, we waste too much time on optimization.
There are a few basic strategies that are used in adjusting $\eta$ over time (we will discuss more advanced strategies in a later chapter):
-->

*dịch đoạn phía trên*


$$
\begin{aligned}
    \eta(t) & = \eta_i \text{ if } t_i \leq t \leq t_{i+1}  && \mathrm{piecewise~constant} \\
    \eta(t) & = \eta_0 \cdot e^{-\lambda t} && \mathrm{exponential} \\
    \eta(t) & = \eta_0 \cdot (\beta t + 1)^{-\alpha} && \mathrm{polynomial}
\end{aligned}
$$
<!-- dịch piecewise~constant, exponential và polynomial -->


<!--
In the first scenario we decrease the learning rate, e.g., whenever progress in optimization has stalled.
This is a common strategy for training deep networks.
Alternatively we could decrease it much more aggressively by an exponential decay.
Unfortunately this leads to premature stopping before the algorithm has converged.
A popular choice is polynomial decay with $\alpha = 0.5$.
In the case of convex optimization there are a number of proofs which show that this rate is well behaved.
Let us see what this looks like in practice.
-->

*dịch đoạn phía trên*


```{.python .input  n=4}
def exponential():
    global ctr
    ctr += 1
    return math.exp(-0.1 * ctr)

ctr = 1
lr = exponential  # Set up learning rate
d2l.show_trace_2d(f, d2l.train_2d(sgd, steps=1000))
```


<!-- ===================== Kết thúc dịch Phần 2 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 3 ===================== -->

<!--
As expected, the variance in the parameters is significantly reduced.
However, this comes at the expense of failing to converge to the optimal solution $\mathbf{x} = (0, 0)$.
Even after 1000 steps are we are still very far away from the optimal solution.
Indeed, the algorithm fails to converge at all.
On the other hand, if we use a polynomial decay where the learning rate decays with the inverse square root of the number of steps convergence is good.
-->

*dịch đoạn phía trên*


```{.python .input  n=5}
def polynomial():
    global ctr
    ctr += 1
    return (1 + 0.1 * ctr)**(-0.5)

ctr = 1
lr = polynomial  # Set up learning rate
d2l.show_trace_2d(f, d2l.train_2d(sgd, steps=50))
```


<!--
There exist many more choices for how to set the learning rate.
For instance, we could start with a small rate, then rapidly ramp up and then decrease it again, albeit more slowly.
We could even alternate between smaller and larger learning rates.
There exists a large variety of such schedules.
For now let us focus on learning rate schedules for which a comprehensive theoretical analysis is possible, i.e., on learning rates in a convex setting.
For general nonconvex problems it is very difficult to obtain meaningful convergence guarantees, since in general minimizing nonlinear nonconvex problems is NP hard.
For a survey see e.g., the excellent [lecture notes](https://www.stat.cmu.edu/~ryantibs/convexopt-F15/lectures/26-nonconvex.pdf) of Tibshirani 2015.
-->

*dịch đoạn phía trên*

<!-- ========================================= REVISE PHẦN 1 - KẾT THÚC ===================================-->

<!-- ========================================= REVISE PHẦN 2 - BẮT ĐẦU ===================================-->

<!--
## Convergence Analysis for Convex Objectives
-->

## *dịch tiêu đề phía trên*

<!--
The following is optional and primarily serves to convey more intuition about the problem.
We limit ourselves to one of the simplest proofs, as described by :cite:`Nesterov.Vial.2000`.
Significantly more advanced proof techniques exist, e.g., whenever the objective function is particularly well behaved.
:cite:`Hazan.Rakhlin.Bartlett.2008` show that for strongly convex functions, i.e., for functions that can be bounded from below by $\mathbf{x}^\top \mathbf{Q} \mathbf{x}$, 
it is possible to minimize them in a small number of steps while decreasing the learning rate like $\eta(t) = \eta_0/(\beta t + 1)$.
Unfortunately this case never really occurs in deep learning and we are left with a much more slowly decreasing rate in practice.
-->

*dịch đoạn phía trên*

<!--
Consider the case where
-->

*dịch đoạn phía trên*


$$\mathbf{w}_{t+1} = \mathbf{w}_{t} - \eta_t \partial_\mathbf{w} l(\mathbf{x}_t, \mathbf{w}).$$


<!--
In particular, assume that $\mathbf{x}_t$ is drawn from some distribution $P(\mathbf{x})$ and that $l(\mathbf{x}, \mathbf{w})$ is a convex function in $\mathbf{w}$ for all $\mathbf{x}$.
Last denote by
-->

*dịch đoạn phía trên*


$$R(\mathbf{w}) = E_{\mathbf{x} \sim P}[l(\mathbf{x}, \mathbf{w})]$$


<!-- ===================== Kết thúc dịch Phần 3 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 4 ===================== -->

<!--
the expected risk and by $R^*$ its minimum with regard to $\mathbf{w}$.
Last let $\mathbf{w}^*$ be the minimizer (we assume that it exists within the domain which $\mathbf{w}$ is defined).
In this case we can track the distance between the current parameter $\mathbf{w}_t$ and the risk minimizer $\mathbf{w}^*$ and see whether it improves over time:
-->

*dịch đoạn phía trên*


$$\begin{aligned}
    \|\mathbf{w}_{t+1} - \mathbf{w}^*\|^2 & = \|\mathbf{w}_{t} - \eta_t \partial_\mathbf{w} l(\mathbf{x}_t, \mathbf{w}) - \mathbf{w}^*\|^2 \\
    & = \|\mathbf{w}_{t} - \mathbf{w}^*\|^2 + \eta_t^2 \|\partial_\mathbf{w} l(\mathbf{x}_t, \mathbf{w})\|^2 - 2 \eta_t
    \left\langle \mathbf{w}_t - \mathbf{w}^*, \partial_\mathbf{w} l(\mathbf{x}_t, \mathbf{w})\right\rangle.
   \end{aligned}
$$



<!--
The gradient $\partial_\mathbf{w} l(\mathbf{x}_t, \mathbf{w})$ can be bounded from above by some Lipschitz constant $L$, hence we have that
-->

*dịch đoạn phía trên*


$$\eta_t^2 \|\partial_\mathbf{w} l(\mathbf{x}_t, \mathbf{w})\|^2 \leq \eta_t^2 L^2.$$


<!--
We are mostly interested in how the distance between $\mathbf{w}_t$ and $\mathbf{w}^*$ changes *in expectation*.
In fact, for any specific sequence of steps the distance might well increase, depending on whichever $\mathbf{x}_t$ we encounter.
Hence we need to bound the inner product. By convexity we have that
-->

*dịch đoạn phía trên*


$$
l(\mathbf{x}_t, \mathbf{w}^*) \geq l(\mathbf{x}_t, \mathbf{w}_t) + \left\langle \mathbf{w}^* - \mathbf{w}_t, \partial_{\mathbf{w}} l(\mathbf{x}_t, \mathbf{w}_t) \right\rangle.
$$


<!--
Using both inequalities and plugging it into the above we obtain a bound on the distance between parameters at time $t+1$ as follows:
-->

*dịch đoạn phía trên*


$$\|\mathbf{w}_{t} - \mathbf{w}^*\|^2 - \|\mathbf{w}_{t+1} - \mathbf{w}^*\|^2 \geq 2 \eta_t (l(\mathbf{x}_t, \mathbf{w}_t) - l(\mathbf{x}_t, \mathbf{w}^*)) - \eta_t^2 L^2.$$


<!--
This means that we make progress as long as the expected difference between current loss and the optimal loss outweighs $\eta_t L^2$.
Since the former is bound to converge to $0$ it follows that the learning rate $\eta_t$ also needs to vanish.
-->

*dịch đoạn phía trên*

<!--
Next we take expectations over this expression. This yields
-->

*dịch đoạn phía trên*


$$E_{\mathbf{w}_t}\left[\|\mathbf{w}_{t} - \mathbf{w}^*\|^2\right] - E_{\mathbf{w}_{t+1}\mid \mathbf{w}_t}\left[\|\mathbf{w}_{t+1} - \mathbf{w}^*\|^2\right] \geq 2 \eta_t [E[R[\mathbf{w}_t]] - R^*] -  \eta_t^2 L^2.$$


<!--
The last step involves summing over the inequalities for $t \in \{t, \ldots, T\}$.
Since the sum telescopes and by dropping the lower term we obtain
-->

*dịch đoạn phía trên*


$$\|\mathbf{w}_{0} - \mathbf{w}^*\|^2 \geq 2 \sum_{t=1}^T \eta_t [E[R[\mathbf{w}_t]] - R^*] - L^2 \sum_{t=1}^T \eta_t^2.$$


<!--
Note that we exploited that $\mathbf{w}_0$ is given and thus the expectation can be dropped. Last define
-->

*dịch đoạn phía trên*


$$\bar{\mathbf{w}} := \frac{\sum_{t=1}^T \eta_t \mathbf{w}_t}{\sum_{t=1}^T \eta_t}.$$


<!--
Then by convexity it follows that
-->

*dịch đoạn phía trên*


$$\sum_t \eta_t E[R[\mathbf{w}_t]] \geq \sum \eta_t \cdot \left[E[\bar{\mathbf{w}}]\right].$$


<!--
Plugging this into the above inequality yields the bound
-->

*dịch đoạn phía trên*

<!-- ===================== Kết thúc dịch Phần 4 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 5 ===================== -->

$$
\left[E[\bar{\mathbf{w}}]\right] - R^* \leq \frac{r^2 + L^2 \sum_{t=1}^T \eta_t^2}{2 \sum_{t=1}^T \eta_t}.
$$


<!--
Here $r^2 := \|\mathbf{w}_0 - \mathbf{w}^*\|^2$ is a bound on the distance between the initial choice of parameters and the final outcome.
In short, the speed of convergence depends on how rapidly the loss function changes via the Lipschitz constant $L$ and how far away from optimality the initial value is $r$.
Note that the bound is in terms of $\bar{\mathbf{w}}$ rather than $\mathbf{w}_T$.
This is the case since $\bar{\mathbf{w}}$ is a smoothed version of the optimization path.
Now let us analyze some choices for $\eta_t$.
-->

*dịch đoạn phía trên*

<!--
* **Known Time Horizon**. 
Whenever $r, L$ and $T$ are known we can pick $\eta = r/L \sqrt{T}$. 
This yields as upper bound $r L (1 + 1/T)/2\sqrt{T} < rL/\sqrt{T}$. 
That is, we converge with rate $\mathcal{O}(1/\sqrt{T})$ to the optimal solution.
* **Unknown Time Horizon**. 
Whenever we want to have a good solution for *any* time $T$ we can pick $\eta = \mathcal{O}(1/\sqrt{T})$. 
This costs us an extra logarithmic factor and it leads to an upper bound of the form $\mathcal{O}(\log T / \sqrt{T})$.
-->

*dịch đoạn phía trên*

<!--
Note that for strongly convex losses 
$l(\mathbf{x}, \mathbf{w}') \geq l(\mathbf{x}, \mathbf{w}) + \langle \mathbf{w}'-\mathbf{w}, \partial_\mathbf{w} l(\mathbf{x}, \mathbf{w}) \rangle + \frac{\lambda}{2} \|\mathbf{w}-\mathbf{w}'\|^2$ 
we can design even more rapidly converging optimization schedules. 
In fact, an exponential decay in $\eta$ leads to a bound of the form $\mathcal{O}(\log T / T)$.
-->

*dịch đoạn phía trên*

<!-- ===================== Kết thúc dịch Phần 5 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 6 ===================== -->

<!-- ========================================= REVISE PHẦN 2 - KẾT THÚC ===================================-->

<!-- ========================================= REVISE PHẦN 3 - BẮT ĐẦU ===================================-->

<!--
## Stochastic Gradients and Finite Samples
-->

## *dịch tiêu đề phía trên*

<!--
So far we have played a bit fast and loose when it comes to talking about stochastic gradient descent.
We posited that we draw instances $x_i$, typically with labels $y_i$ from some distribution $p(x, y)$ and that we use this to update the weights $w$ in some manner.
In particular, for a finite sample size we simply argued that the discrete distribution $p(x, y) = \frac{1}{n} \sum_{i=1}^n \delta_{x_i}(x) \delta_{y_i}(y)$ allows us to perform SGD over it.
-->

*dịch đoạn phía trên*

<!--
However, this is not really what we did.
In the toy examples in the current section we simply added noise to an otherwise non-stochastic gradient, i.e., we pretended to have pairs $(x_i, y_i)$.
It turns out that this is justified here (see the exercises for a detailed discussion).
More troubling is that in all previous discussions we clearly did not do this.
Instead we iterated over all instances exactly once.
To see why this is preferable consider the converse, namely that we are sampling $n$ observations from the discrete distribution with replacement.
The probability of choosing an element $i$ at random is $N^{-1}$. Thus to choose it at least once is
-->

*dịch đoạn phía trên*


$$P(\mathrm{choose~} i) = 1 - P(\mathrm{omit~} i) = 1 - (1-N^{-1})^N \approx 1-e^{-1} \approx 0.63.$$
<!-- cân nhắc dịch -->


<!--
A similar reasoning shows that the probability of picking a sample exactly once is given by ${N \choose 1} N^{-1} (1-N^{-1})^{N-1} = \frac{N-1}{N} (1-N^{-1})^{N} \approx e^{-1} \approx 0.37$.
This leads to an increased variance and decreased data efficiency relative to sampling without replacement.
Hence, in practice we perform the latter (and this is the default choice throughout this book).
Last note that repeated passes through the dataset traverse it in a *different* random order.
-->

*dịch đoạn phía trên*

<!-- ===================== Kết thúc dịch Phần 6 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 7 ===================== -->

<!--
## Summary
-->

## Tóm tắt

<!--
* For convex problems we can prove that for a wide choice of learning rates Stochastic Gradient Descent will converge to the optimal solution.
* For deep learning this is generally not the case. However, the analysis of convex problems gives us useful insight into how to approach optimization, 
namely to reduce the learning rate progressively, albeit not too quickly.
* Problems occur when the learning rate is too small or too large. In practice  a suitable learning rate is often found only after multiple experiments.
* When there are more examples in the training dataset, it costs more to compute each iteration for gradient descent, so SGD is preferred in these cases.
* Optimality guarantees for SGD are in general not available in nonconvex cases since the number of local minima that require checking might well be exponential.
-->

*dịch đoạn phía trên*


<!--
## Exercises
-->

## Bài tập

<!--
1. Experiment with different learning rate schedules for SGD and with different numbers of iterations.
In particular, plot the distance from the optimal solution $(0, 0)$ as a function of the number of iterations.
2. Prove that for the function $f(x_1, x_2) = x_1^2 + 2 x_2^2$ adding normal noise to the gradient is equivalent to minimizing a loss function $l(\mathbf{x}, \mathbf{w}) = (x_1 - w_1)^2 + 2 (x_2 - w_2)^2$ where $x$ is drawn from a normal distribution.
    * Derive mean and variance of the distribution for $\mathbf{x}$.
    * Show that this property holds in general for objective functions $f(\mathbf{x}) = \frac{1}{2} (\mathbf{x} - \mathbf{\mu})^\top Q (\mathbf{x} - \mathbf{\mu})$ for $Q \succeq 0$.
3. Compare convergence of SGD when you sample from $\{(x_1, y_1), \ldots, (x_m, y_m)\}$ with replacement and when you sample without replacement.
4. How would you change the SGD solver if some gradient (or rather some coordinate associated with it) was consistently larger than all other gradients?
5. Assume that $f(x) = x^2 (1 + \sin x)$. How many local minima does $f$ have? Can you change $f$ in such a way that to minimize it one needs to evaluate all local minima?
-->

*dịch đoạn phía trên*

<!-- ===================== Kết thúc dịch Phần 7 ===================== -->
<!-- ========================================= REVISE PHẦN 3 - KẾT THÚC ===================================-->


## Thảo luận
* [Tiếng Anh](https://discuss.mxnet.io/t/2372)
* [Tiếng Việt](https://forum.machinelearningcoban.com/c/d2l)

## Những người thực hiện
Bản dịch trong trang này được thực hiện bởi:
<!--
Tác giả của mỗi Pull Request điền tên mình và tên những người review mà bạn thấy
hữu ích vào từng phần tương ứng. Mỗi dòng một tên, bắt đầu bằng dấu `*`.

Lưu ý:
* Nếu reviewer không cung cấp tên, bạn có thể dùng tên tài khoản GitHub của họ
với dấu `@` ở đầu. Ví dụ: @aivivn.

* Tên đầy đủ của các reviewer có thể được tìm thấy tại https://github.com/aivivn/d2l-vn/blob/master/docs/contributors_info.md
-->

* Đoàn Võ Duy Thanh
<!-- Phần 1 -->
* Nguyễn Duy Du

<!-- Phần 2 -->
* 

<!-- Phần 3 -->
* 

<!-- Phần 4 -->
* 

<!-- Phần 5 -->
* 

<!-- Phần 6 -->
* 

<!-- Phần 7 -->
* 
