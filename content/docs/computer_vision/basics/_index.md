---
title: Basics
weight: 1
date: 2026-10-07
math: true
---

## Deep Learning Basics
The central learning pipeline is:
**Labeled images → class scores → loss → gradients → parameter updates → evaluation on unseen images.**

### Image classification and the data-driven approach
An image is an array of pixel values; classification assigns it a category. Viewpoint, lighting, occlusion, deformation, and background changes make fixed recognition rules brittle. Instead, a model learns from labeled examples.

CIFAR-10 provides a compact setting: $32 \times 32$ RGB images, ten classes, and 3,072 input values per flattened image. Use **training data** to fit parameters, **validation data** to select hyperparameters, and **test data** for the final evaluation. Fit preprocessing statistics on the training split and apply the same transformation to the other splits.

**k-nearest neighbors (kNN)** stores training examples and predicts by voting among the closest examples under a distance such as $L_1$ or $L_2$. Choose the distance and $k$ through validation. It is a useful baseline, but storing all examples and comparing each query against them makes prediction expensive. Pixel distance also confuses visual similarity with changes in position or background.

### Linear scores, softmax, and regularization
A linear classifier replaces stored examples with learned parameters. Using batches whose examples are rows:

$$
\begin{aligned}
X &\in \mathbb{R}^{N \times D}, &
W &\in \mathbb{R}^{D \times C}, &
b &\in \mathbb{R}^{C}, \\
S &= XW + b, &
S &\in \mathbb{R}^{N \times C}.
\end{aligned}
$$

Here $N$ is batch size, $D$ is input dimension, and $C$ is class count. The course notes also use column-vector notation, $s = Wx + b$; these are equivalent conventions. Algebraically, each class has a weighted sum of inputs. Visually, its weights resemble a template. Geometrically, pairwise class boundaries are hyperplanes.

**Softmax** converts scores to probabilities; cross-entropy penalizes low probability for the correct class:

$$
\begin{aligned}
p_j
&= \frac{\exp(s_j - \max_k s_k)}
        {\sum_k \exp(s_k - \max_\ell s_\ell)}, \\
\ell_i
&= -\log p_{y_i}, \\
L
&= \frac{1}{N}\sum_{i=1}^{N}\ell_i
   + \frac{\lambda}{2}\lVert W \rVert_F^2.
\end{aligned}
$$

Subtracting the maximum helps prevent exponential overflow; log-sum-exp provides a more robust loss computation. $L_2$ regularization discourages large weights. Its strength is selected on validation data. The factor $1/2$ is a convention: this form contributes $\lambda W$ to the gradient.

### Optimization: turning gradients into learning

The gradient gives the local direction of greatest increase in the loss. Gradient descent moves in the opposite direction. Full-batch gradients use every training example; **mini-batch SGD** estimates the gradient from a sampled batch, making updates cheaper:

$$
\theta_{t+1} = \theta_t - \eta \nabla_\theta L(\theta_t).
$$

The learning rate controls step size. Excessive steps can cause instability; tiny steps slow progress. A learning-rate schedule reduces the rate during training. Numerical gradients estimate derivatives by perturbing parameters and are useful for checking analytic gradients, but expensive for training.

Different update rules manage noisy gradients and unequal parameter scales:

| Update rule | Main idea | Practical implication |
| --- | --- | --- |
| SGD | Follow the current mini-batch gradient | Simple baseline; sensitive to learning rate |
| Momentum | Carry a velocity across updates | Reinforces consistent directions and reduces oscillation |
| AdaGrad | Accumulate squared gradients per parameter | Adapts step sizes, but accumulated history can make them very small |
| RMSProp | Use a moving average of squared gradients | Limits the influence of distant gradient history |
| Adam | Combine moving averages of gradients and squared gradients, with bias correction | Adapts steps while retaining directional history |

Track training loss and training/validation accuracy together. A widening accuracy gap suggests overfitting; poor accuracy on both splits warrants checking optimization, preprocessing, and capacity. First verify that the model can fit a tiny dataset, then tune on validation data and retain the best checkpoint.

### Neural networks: learning nonlinear representations

A fully connected network composes affine transformations with nonlinear activations. A two-layer classifier has one hidden layer and two learned weight matrices:

$$
\begin{aligned}
H &= \operatorname{ReLU}(XW_1 + b_1), \\
S &= HW_2 + b_2, \\
\operatorname{ReLU}(z) &= \max(0, z).
\end{aligned}
$$

Without nonlinearities, multiple affine layers collapse into one affine mapping. Nonlinear hidden units let the network learn features and more flexible decision boundaries.

ReLU is simple, but negative inputs receive zero gradient and units can become inactive. Sigmoid and tanh can saturate, producing very small gradients. Depth and width affect capacity and training behavior; larger networks still need appropriate optimization and regularization.

Initialize weights randomly to break symmetry. Their scale matters: inappropriate scaling can shrink or amplify signals through layers. Center inputs using training statistics and choose initialization suited to the activation.

### Backpropagation: the chain rule on a computation graph

The forward pass computes outputs and caches intermediate values. The backward pass traverses operations in reverse, multiplying the upstream gradient by each operation's local derivative. When a value influences several paths, their gradient contributions add.

For a row-batch affine layer $Y = XW + b$, with upstream derivative $G = \partial L / \partial Y$:

$$
\begin{aligned}
\frac{\partial L}{\partial X} &= GW^{\mathsf T}, \\
\frac{\partial L}{\partial W} &= X^{\mathsf T}G, \\
\frac{\partial L}{\partial b} &= \sum_{i=1}^{N} G_{i,:}.
\end{aligned}
$$

For ReLU, pass the upstream gradient where the cached input is positive and zero it elsewhere. For mean softmax cross-entropy, the score gradient is $(P - Y_{\text{one-hot}}) / N$. Add regularization derivatives to the weight gradients.

Backpropagation computes derivatives; the optimizer uses those derivatives to update parameters. Keep these responsibilities separate so layers can be reused. Validate shapes and compare analytic derivatives against centered finite differences on small inputs. ReLU kinks can cause discrepancies when a perturbation crosses zero.

## Resources
- [Stanford CS231n](https://cs231n.stanford.edu)
- [Hugging Face computer vision course](https://huggingface.co/learn/computer-vision-course/unit0/welcome/welcome)
