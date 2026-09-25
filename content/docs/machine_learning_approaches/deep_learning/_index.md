---
title: Deep learning
weight: 1
date: 2026-09-06
---

## Introduction

Deep learning is machine learning with *deep* neural networks: stacked differentiable layers, trained end-to-end from examples. The task is familiar—predict, generate, or act. The method is to learn a hierarchy of features from raw input (pixels, waveforms, tokens) instead of designing them by hand.

Raw signals are first turned into vectors. Discrete symbols are tokenized and embedded; pixels and waveforms are already numeric and are often patched or framed. Every layer after that is a differentiable map.

The method has two stages. In **training**, a loss scores the network against a signal—labels, the input itself, a learned critic, or a reward; backpropagation and gradient descent update the weights. In **inference**, those weights stay frozen and only the forward pass runs, mapping a new input to a prediction or sample.

Depth helps because later layers reuse and recombine earlier ones, and the same optimization scales with data and compute. Residual connections and normalization are what make very deep stacks trainable. The cost is data and compute; fit on the training distribution does not imply reliability outside it.

The training signal comes in four setups: **supervised** (human labels), **self-supervised** (targets built from the input—next token, masked patch, paired view), **unsupervised** (structure or a data distribution, no task label), and **reinforcement** (a reward or preference). At scale the usual workflow is to pretrain a large model, often self-supervised, then adapt it by fine-tuning, prompting, or alignment.

## Neural network
A neural network is a parameterized function. Each unit takes a weighted sum of its inputs and applies a nonlinearity; stacked and trained, the whole mapping approximates a target function from data. The same stack can later be trained with different losses; what changes first is the wiring. The network *is* the model. Families below differ by **connectivity**: how units link and share weights. That inductive bias is what matches images, sequences, or graphs.

- Multilayer perceptron (MLP)
  - Fully connected layers; each unit sees every activation from the previous layer
  - Universal function approximator in theory, but no bias for space or time
  - Rarely used alone; still the default block for heads and Transformer FFNs
- Convolutional neural network (CNN)
  - Local receptive fields and shared weights extract translation-equivariant spatial features
  - Stacking, striding, and pooling build a hierarchy from edges to objects
  - Long-standing default for vision; still common as backbones and in hybrids
- Recurrent neural network (RNN)
  - A hidden state is updated at each step, so the same weights process a sequence
  - LSTM and GRU add gates for longer dependencies and stabler gradients
  - Largely replaced by Transformers on long sequences; still useful when streaming or latency is tight
- Transformer
  - Self-attention models pairwise token dependencies without recurrence or convolution
  - Training is highly parallel; quality scales with data, parameters, and compute
  - Backbone of modern LLMs and many vision / multimodal models (ViT and descendants)
- Graph neural network (GNN)
  - Message passing lets each node aggregate features from its neighbors
  - Fits graph-structured data: molecules, social graphs, recommenders, meshes
  - Common variants: GCN, GAT, GraphSAGE

## Inference framework
- [Inside NVIDIA GPUs: Anatomy of high performance matmul kernels](https://www.aleksagordic.com/blog/matmul)
- [Qualcomm demo](https://github.com/SnapdragonGameStudios/adreno-gpu-vulkan-code-sample-framework)
- [ggml](https://github.com/ggml-org/ggml)
- [llama.cpp](https://github.com/ggml-org/llama.cpp)
- [ncnn](https://github.com/Tencent/ncnn)

## Resource
- [Hugging face](https://huggingface.co/)
- [Rethink fun deep learning](https://www.rethink.fun/)
- [Practical deep learning for coders](https://course.fast.ai/)