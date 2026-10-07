---
title: Basics
weight: 1
date: 2026-10-07
math: true
---

## Deep learning basics
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
p_j &= \frac{\exp\!\left(s_j - \max_k s_k\right)}{\sum_{m=1}^{C} \exp\!\left(s_m - \max_k s_k\right)}, \\
\mathcal{L}_i &= -\log\!\left(p_{y_i}\right), \\
\mathcal{L} &= \frac{1}{N}\sum_{i=1}^{N}\mathcal{L}_i + \frac{\lambda}{2}\left\lVert W \right\rVert_F^2.
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

## Perceiving and understanding the visual world

The central progression is:
**local visual patterns → hierarchical representations → global context → structured predictions → scalable training.**

### Convolutional neural networks

Flattening an image discards its two-dimensional structure. A convolutional layer instead slides a small learnable filter across the input. The same weights are reused at every spatial location, giving fewer parameters and a useful translation-equivariant bias: shifting the input shifts the feature map.

For an input of spatial size $H \times W$, kernel size $K$, padding $P$, and stride $S$, each output dimension is

$$
H' = \left\lfloor \frac{H + 2P - K}{S} \right\rfloor + 1,
\qquad
W' = \left\lfloor \frac{W + 2P - K}{S} \right\rfloor + 1.
$$

Each filter spans the full input-channel depth and produces one output channel. Stacking convolution, nonlinear activation, and downsampling layers grows the **receptive field**: early layers respond to edges and textures, while later layers can represent parts and objects. Padding controls border behavior; stride or pooling reduces spatial resolution. Pooling has no learned parameters, although modern networks often use strided convolution and global average pooling instead.

Compared with hand-designed features such as color histograms and HOG, a CNN learns the feature extractor and classifier jointly from data. This end-to-end approach preserves spatial structure while adapting the representation to the task.

### CNN architectures, normalization, and transfer learning

Architectures evolved by improving optimization and information flow:

| Architecture | Main contribution |
| --- | --- |
| AlexNet | Demonstrated large-scale deep CNN training with ReLU, GPUs, augmentation, and dropout |
| VGG | Used repeated small $3 \times 3$ convolutions to build deeper, uniform networks |
| GoogLeNet | Combined multiple receptive-field sizes in Inception modules and used global average pooling |
| ResNet | Added identity shortcuts so layers learn residual functions, enabling much deeper networks |

A residual block computes

$$
y = F(x; W) + x.
$$

The identity path gives signals and gradients a direct route through the network. It also lets a block fall back toward an identity mapping when extra transformation is unnecessary.

**Batch normalization** normalizes activations using mini-batch statistics during training, then applies learned scale and shift parameters. At inference it uses running statistics. It often permits faster, more stable optimization, but behavior depends on batch size and on switching correctly between training and evaluation modes.

When labeled data are limited, start from a model pretrained on a large dataset. Use it as a fixed feature extractor by replacing and training only the final head, or **fine-tune** some or all layers with a smaller learning rate. The closer the source and target domains are—and the more target data available—the more useful full fine-tuning tends to be.

### Recurrent networks and visual sequences

Images can be connected to sequences: an image may condition a caption, video supplies a temporal sequence, and language provides variable-length supervision. A recurrent neural network updates a hidden state:

$$
h_t = \tanh(W_x x_t + W_h h_{t-1} + b),
\qquad
y_t = W_y h_t + b_y.
$$

Parameters are shared across time, and backpropagation through time applies the chain rule through the unrolled computation. Repeated Jacobian multiplication can make gradients vanish or explode. Gradient clipping limits explosions; LSTMs and GRUs introduce gates and additive state paths that preserve information over longer intervals.

An image-captioning model can encode an image into visual features and decode words autoregressively. During training, the decoder commonly receives ground-truth previous tokens; at inference it consumes its own predictions and stops at an end token. Greedy decoding is cheap, while beam search retains several promising partial sequences. A sequence-to-sequence model generalizes this encoder-decoder pattern to arbitrary input and output sequences.

### Attention and transformers

Recurrence compresses the past into a single evolving state and processes tokens sequentially. Attention instead lets each query retrieve relevant information from a set of keys and values:

$$
\operatorname{Attention}(Q,K,V)
= \operatorname{softmax}\!\left(\frac{QK^{\mathsf T}}{\sqrt{d_k}}\right)V.
$$

The dot products measure query-key compatibility; scaling controls their magnitude; softmax forms normalized weights; and the weighted values provide the output. **Self-attention** draws $Q$, $K$, and $V$ from the same sequence. **Cross-attention** queries one representation using another. Multiple heads learn different relations in parallel.

A transformer block combines multi-head attention, a position-wise MLP, residual connections, and normalization. Positional encodings are necessary because attention alone is permutation-equivariant. Causal masks prevent an autoregressive decoder from seeing future tokens. Unlike an RNN, a transformer trains all sequence positions in parallel, though ordinary self-attention has quadratic cost in sequence length.

A Vision Transformer divides an image into patches, linearly embeds them as tokens, adds positional information, and processes them with transformer blocks. CNNs have stronger locality and translation biases; transformers more directly model long-range interactions and tend to benefit from large-scale pretraining.

### Detection, segmentation, and model interpretation

Classification predicts one label for an image. Richer visual tasks preserve spatial structure:

| Task | Output |
| --- | --- |
| Object detection | Class labels and bounding boxes for object instances |
| Semantic segmentation | A class label for every pixel |
| Instance segmentation | A separate pixel mask for each object instance |
| Panoptic segmentation | A unified labeling of countable objects and amorphous regions |

Object detectors predict both class confidence and box coordinates. **Two-stage detectors** first propose candidate regions and then classify and refine them; **single-stage detectors** make dense predictions directly and are usually simpler and faster. Overlapping predictions are commonly filtered with intersection over union and non-maximum suppression. DETR reframes detection as set prediction: transformer queries predict objects, and bipartite matching assigns predictions to ground truth without hand-designed anchors or non-maximum suppression.

Semantic segmentation networks produce spatial score maps and restore resolution through learned upsampling, interpolation, or encoder-decoder skip connections. Instance segmentation adds object separation, while panoptic segmentation reconciles instance-level “things” with semantic “stuff.”

Interpretation methods reveal what a network has learned, but not all answer the same question. Activation maximization synthesizes inputs that excite a neuron; feature inversion reconstructs inputs compatible with a representation; saliency estimates which pixels most affect a decision. Adversarial examples show that small, deliberately chosen perturbations can change predictions. DeepDream amplifies learned patterns, and neural style transfer optimizes an image to match content features from one image and feature correlations—often represented by Gram matrices—from another.

### Video and multimodal understanding

Video adds time to space. Averaging predictions from independent frames ignores motion; temporal models instead learn how appearance changes. Common approaches include:

- **3D convolutions**, whose kernels span height, width, and time;
- **two-stream networks**, which separately process RGB appearance and motion such as optical flow;
- **factorized spatial-temporal models**, which reduce the cost of full 3D operations;
- **video transformers**, which apply attention across spatial and temporal tokens.

Long videos make uniform dense processing expensive. Sampling clips, using sparse temporal attention, or building hierarchical representations trades detail for coverage. Multimodal supervision—audio, speech, text, and video—can align events that are difficult to recognize from pixels alone. Evaluation must also guard against shortcuts such as scene background, dataset bias, or information leaking between neighboring clips.

### Large-scale distributed training

At scale, training speed depends on hardware utilization, communication, and memory—not only the number of arithmetic operations. In **data parallelism**, workers hold model replicas, process different mini-batches, and aggregate gradients. In **model parallelism**, parameters or layers are split across devices; tensor parallelism divides operations within a layer, while pipeline parallelism assigns consecutive stages to different devices.

The effective global batch size is the per-device batch size times the number of data-parallel workers and gradient-accumulation steps. Larger batches can improve throughput but may require learning-rate adjustment and can change generalization. Mixed precision reduces memory and increases accelerator throughput, with loss scaling used when low-precision gradients would underflow. **Activation checkpointing** saves memory by retaining only selected activations and recomputing the rest during backpropagation, exchanging extra computation for lower memory use. A good configuration balances compute, communication, pipeline idle time, and memory rather than maximizing any one dimension.

## Generative and interactive visual intelligence

The central progression is:
**learn from unlabeled signals → model a data distribution → generate and reconstruct worlds → align vision with language and human goals.**

### Self-supervised representation learning

Supervised learning obtains its target from human labels; self-supervised learning constructs supervision from the data itself. Earlier **pretext tasks** predicted rotations, missing patches, color, temporal order, or one sensory stream from another. Their purpose was not the pretext answer itself, but a representation transferable to downstream tasks.

Contrastive learning pulls together representations of two augmented views of the same example and pushes apart representations of different examples. A common objective for a positive pair $(i,j)$ is

$$
\ell_{i,j}
= -\log
\frac{\exp(\left(\operatorname{sim}(z_i,z_j)/\tau\right))}
{\sum_{k \ne i}\exp(\left(\operatorname{sim}(z_i,z_k)/\tau\right))},
$$

where $\tau$ is a temperature and similarity is often cosine similarity. Augmentations define which changes the representation should ignore, so they are part of the learning problem rather than mere preprocessing. Other approaches avoid explicit negatives through teacher-student targets, stop-gradient operations, clustering, or redundancy reduction. Masked-image modeling reconstructs hidden image content, while methods such as DINO show that self-distillation with vision transformers can produce semantically organized features without class labels.

### Latent-variable and adversarial generative models

A generative model learns a distribution over data rather than only a decision boundary. It may estimate likelihood explicitly, optimize a lower bound, learn through an adversarial game, or define a procedure that transforms noise into samples.

A **variational autoencoder (VAE)** introduces a latent variable $z$ and optimizes the evidence lower bound:

$$
\log p_\theta(x) \geq \mathcal{L}_{\mathrm{ELBO}}(x;\theta,\phi) = \mathbb{E}_{q_\phi(z\mid x)}\left[\log p_\theta(x\mid z)\right] - D_{\mathrm{KL}}\left(q_\phi(z\mid x)\parallel p_\theta(z)\right)
$$

The reconstruction term preserves information about $x$; the KL term regularizes the approximate posterior toward the prior. The reparameterization trick, $z = \mu + \sigma \odot \epsilon$ with $\epsilon \sim \mathcal{N}(0,I)$, allows gradients to pass through sampling. VAEs provide structured latent spaces and likelihood-based training, though pixelwise decoders can yield overly smooth images.

A **generative adversarial network (GAN)** trains a generator against a discriminator:

$$
\min_G \max_D V(D,G) = \mathbb{E}_{x\sim p_{\mathrm{data}}}\left[\log D(x)\right] + \mathbb{E}_{z\sim p_z}\left[\log\left(1-D(G(z))\right)\right]
$$

The generator can produce sharp samples in one forward pass, but the minimax game can be unstable and may collapse to limited modes. Training depends on balanced optimization, suitable objectives, normalization, and architecture choices.

An **autoregressive model** factorizes a joint distribution into conditional probabilities:

$$
p_\theta(x_1,\ldots,x_n)
= p_\theta(x_1)\prod_{i=2}^{n}
  p_\theta\!\left(x_i \mid x_1,\ldots,x_{i-1}\right).
$$

It gives tractable likelihoods and supports flexible conditioning, but generation is sequential unless specialized parallel schemes are used. The chosen ordering—pixels, tokens, or patches—shapes both computation and dependencies.

### Diffusion models

Diffusion models gradually corrupt data with noise in a fixed forward process and learn to reverse it. A common parameterization samples

$$
\alpha_t = 1-\beta_t, \qquad \bar{\alpha}_t = \prod_{s=1}^{t}\alpha_s
$$

$$
q(x_t\mid x_0) = \mathcal{N}\left(\sqrt{\bar{\alpha}_t}x_0,(1-\bar{\alpha}_t)I\right)
$$

$$
x_t = \sqrt{\bar{\alpha}_t}x_0 + \sqrt{1-\bar{\alpha}_t}\epsilon, \qquad \epsilon\sim\mathcal{N}(0,I)
$$

then trains a network $\epsilon_\theta(x_t,t,c)$ to predict the added noise, optionally conditioned on information $c$ such as text. Sampling starts from noise and repeatedly denoises toward an image. The iterative process yields high-quality, diverse samples but is slower than a single generator pass; improved samplers and latent diffusion reduce the cost.

Classifier-free guidance combines conditional and unconditional predictions:

$$
\epsilon_{\mathrm{CFG}}(x_t,t,c) = \epsilon_\theta(x_t,t,\varnothing) + s\left[\epsilon_\theta(x_t,t,c)-\epsilon_\theta(x_t,t,\varnothing)\right], \qquad s\geq 1
$$

At $s=1$, this reduces to the ordinary conditional prediction. Increasing guidance scale $s$ usually improves adherence to the condition at the cost of sample diversity and, at extremes, visual quality. Conditioning, representation choice, noise schedule, and sampling algorithm are therefore distinct design decisions.

### 3D vision and neural scene representations

Images are projections of a three-dimensional world. Recovering geometry is ambiguous from one view, but multiple calibrated views provide constraints through camera models and correspondence. Common shape representations make different tradeoffs:

| Representation | Strength | Limitation |
| --- | --- | --- |
| Depth map | Aligned with one camera view | Represents only visible surfaces |
| Point cloud | Simple and sensor-friendly | Unordered and lacks explicit surfaces |
| Voxel grid | Regular structure, easy 3D convolution | Memory grows cubically with resolution |
| Mesh | Compact explicit surface | Topology and differentiable editing are difficult |
| Implicit function | Continuous resolution and flexible topology | Rendering requires querying many spatial points |

Neural implicit representations learn a function over continuous coordinates. A signed-distance or occupancy field describes geometry; a neural radiance field maps position and viewing direction to density and color. Volume rendering accumulates colors along camera rays using transmittance and density, allowing the scene representation to be trained from posed images by matching rendered pixels. These methods enable novel-view synthesis, but quality depends on view coverage, camera calibration, scene dynamics, and rendering efficiency.

### Vision-language models

Language supplies open-ended semantic supervision beyond a fixed class vocabulary. A dual-encoder model maps images and texts into a shared embedding space and uses a contrastive objective to align matching pairs. After large-scale pretraining, class names or prompts can act as text prototypes for zero-shot recognition. Performance therefore depends on prompt wording, data coverage, and biases in image-text pairs.

Multimodal systems commonly combine:

- a visual encoder that converts images or video into tokens;
- a connector or projection that maps visual features to a language-compatible space;
- a language model that reasons over or generates from the combined context.

Cross-attention or unified token sequences allow richer interaction than global embedding similarity. Training may progress from contrastive alignment to captioning or next-token prediction and then instruction tuning with human or synthetic preference data. Strong language priors make outputs fluent, but fluency is not evidence of visual grounding: a model may hallucinate objects, miss fine spatial relations, or exploit textual shortcuts. Evaluation should test perception, grounding, reasoning, calibration, and robustness separately.

### World models and interactive intelligence

A world model predicts how an environment evolves, often in a learned latent space. Given observations, actions, and possibly language, it models future states or observations:

$$
z_t = E(o_t),
\qquad
\hat{z}_{t+1} = F(z_t,a_t),
\qquad
\hat{o}_{t+1} = D(\hat{z}_{t+1}).
$$

Prediction alone is not enough for interaction. An agent also needs an objective, a policy or planner, and feedback from the environment. A learned model can simulate candidate action sequences, support planning, generate training experience, or provide compact state for reinforcement learning. Errors compound over long rollouts, and pixel accuracy may emphasize irrelevant detail; useful models must preserve controllable objects, physical constraints, uncertainty, and consequences of actions.

Embodied and interactive settings join perception with memory and action. Passive datasets may not reveal causal structure because the learner cannot intervene. Interaction can resolve ambiguity, but it introduces safety, latency, partial observability, and distribution shift: the agent's own actions change which data it sees.

### Human-centered AI

Visual intelligence operates within social systems. Dataset composition, annotation choices, objectives, and deployment context determine whose behavior is represented and which errors matter. Aggregate accuracy can hide large disparities across groups or rare conditions, while benchmark success may fail to predict performance after deployment.

Human-centered evaluation therefore asks more than whether a model is accurate:

- **validity:** does the benchmark measure the capability the application needs?
- **reliability:** does performance persist under shift, ambiguity, and repeated use?
- **fairness:** how are benefits and errors distributed across people and contexts?
- **transparency and recourse:** can affected users understand, contest, or correct outcomes?
- **privacy and provenance:** were data and generated content collected, attributed, and used appropriately?
- **human control:** are uncertainty, failure modes, and escalation paths visible to the operator?

Generative systems also raise questions about consent, representation, authenticity, and misuse. Technical mitigations—filtering, red teaming, provenance signals, calibrated abstention, and monitoring—help only when paired with clearly defined responsibility and deployment policy. The final system is the model plus its data, interface, users, incentives, and feedback loops.

## Resources
- [Stanford CS231n](https://cs231n.stanford.edu)
- [Hugging Face computer vision course](https://huggingface.co/learn/computer-vision-course/unit0/welcome/welcome)
