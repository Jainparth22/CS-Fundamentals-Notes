# Deep Learning Fundamentals: Interview Notes

## 1. Neural network basics
- A neuron computes `y = f(w . x + b)`. Layers stack neurons; depth plus non-linearity lets the network learn complex functions (universal approximation).
- **Forward pass:** compute predictions. **Loss:** measure error. **Backward pass (backpropagation):** compute gradients with the chain rule. **Optimizer step:** update weights.
- **Without non-linear activations**, any depth collapses to a single linear map.

## 2. Activation functions
| Function | Formula / property | Notes |
|---|---|---|
| Sigmoid | 1/(1+e^-x), range (0,1) | saturates (vanishing gradients); output layer for binary |
| Tanh | range (-1,1), zero-centred | still saturates |
| ReLU | max(0, x) | default; cheap; "dying ReLU" problem |
| Leaky ReLU / PReLU | small slope for x<0 | avoids dead units |
| GELU / SiLU (Swish) | smooth ReLU-like | transformers (GELU) |
| Softmax | exp(x_i)/sum exp(x_j) | multi-class probabilities |

## 3. Loss functions
| Task | Loss |
|---|---|
| Regression | MSE, MAE, Huber |
| Binary classification | binary cross-entropy (`BCEWithLogitsLoss`) |
| Multi-class | cross-entropy (`CrossEntropyLoss` takes raw logits and class indices) |
| Ranking / embeddings | contrastive, triplet, InfoNCE |
| Segmentation | Dice, IoU |
| Language modelling | token-level cross-entropy (perplexity = exp(loss)) |

## 4. Optimization
- **Gradient descent** variants: batch, SGD, mini-batch.
- **Momentum**, **RMSProp**, **Adam** (adaptive per-parameter step sizes with momentum; default choice), **AdamW** (decoupled weight decay; standard for transformers).
- **Learning rate** is the most important hyperparameter. Schedules: step, cosine, warmup then decay, one-cycle, reduce-on-plateau. LR finder to pick a range.
- **Gradient clipping** stops exploding gradients (RNNs, transformers).
- **Batch size:** larger is faster per epoch but may generalise worse; scale LR with it.

## 5. Initialization and normalization
- **Initialisation:** Xavier/Glorot (tanh/sigmoid), He/Kaiming (ReLU). Bad init causes vanishing/exploding signals.
- **BatchNorm:** normalise per mini-batch across the batch dimension; speeds training; behaves differently in train vs eval (running stats).
- **LayerNorm:** normalise across features per sample; used in transformers and RNNs; independent of batch size.
- Also GroupNorm, RMSNorm.

## 6. Regularisation
Weight decay (L2), dropout (randomly zero units, scaled at train, off at eval), early stopping, data augmentation, label smoothing, batch norm side-effect, smaller model, more data, mixup/cutmix, stochastic depth.

## 7. Vanishing and exploding gradients
Deep networks multiply many small (<1) or large (>1) factors. Fixes: ReLU-family activations, good initialisation, normalisation layers, residual connections, gradient clipping, LSTM/GRU gates.

## 8. Architectures
### CNNs (images)
Convolution (shared local filters), padding, stride, pooling; translation equivariance; receptive field. Classics: LeNet, AlexNet, VGG, **ResNet** (skip connections let gradients flow), Inception, EfficientNet; detection (YOLO, Faster R-CNN), segmentation (U-Net). Transfer learning: freeze a pretrained backbone, replace the head, then fine-tune with a lower LR.

### RNNs, LSTM, GRU (sequences)
RNN passes a hidden state through time; suffers from vanishing gradients. **LSTM** adds a cell state with input, forget and output gates; **GRU** is a simpler gated variant. **BiLSTM** reads both directions. **Attention pooling** over hidden states (as in your MCQ BiLSTM over RoBERTa embeddings) weights important positions.

### Transformers
- **Self-attention:** `Attention(Q,K,V) = softmax(QK^T / sqrt(d_k)) V`. Each token attends to all tokens; parallel over sequence (unlike RNNs).
- **Multi-head attention** learns several relations in parallel. **Positional encodings** (sinusoidal, learned, RoPE) supply order.
- Block: multi-head attention -> add and norm -> feed-forward -> add and norm (residuals everywhere).
- **Encoder-only (BERT, RoBERTa):** bidirectional; classification, NER, embeddings. **Decoder-only (GPT, Llama):** causal mask; text generation. **Encoder-decoder (T5, BART):** seq-to-seq.
- Attention cost is O(n^2) in sequence length; remedies: FlashAttention, sparse/linear attention, sliding windows.

### Autoencoders, VAEs, GANs, diffusion
- **Autoencoder:** compress and reconstruct. **VAE:** latent distribution with KL regulariser (used in your JAR project). **GAN:** generator vs discriminator. **Diffusion:** learn to denoise; state of the art for images.

### Embeddings
Dense vectors that place similar items close; learned for words (word2vec, GloVe), sentences (SBERT, BGE), images, users/items. Compare with cosine similarity or dot product.

## 9. Training practice and debugging
- **Overfit one small batch first**; if it cannot, there is a bug.
- Watch train vs validation loss; check input normalisation, label alignment, learning rate, shuffling, and that the model is in `train()`/`eval()` mode appropriately.
- Common bugs: forgetting `zero_grad`, wrong loss input (softmax before `CrossEntropyLoss`), wrong tensor shapes/broadcasting, data leakage, not detaching tensors, device mismatches.
- Reproducibility: seeds, deterministic flags, logged configs, W&B/TensorBoard.
- Mixed precision (fp16/bf16) speeds training; gradient accumulation simulates big batches.

## 10. Transfer learning and fine-tuning
Pretrain on large data, adapt to a small task. Options: **feature extraction** (frozen backbone), **full fine-tuning**, **parameter-efficient fine-tuning (LoRA, adapters, prompt tuning)**. On tiny datasets, fine-tuning can overfit badly; your MCQ project showed fine-tuned DeBERTa (0.352) losing to zero-shot (0.494) on 2,000 rows. Use lower LR, early stopping, freezing, or avoid fine-tuning altogether.

## 11. Evaluation and common pitfalls
Use held-out data, proper metrics, multiple seeds for variance, ablations. Watch distribution shift between validation and test (your "validation-to-test distribution shift" analysis). Check calibration; confidence is not accuracy.

## 12. Model optimization for serving
Quantisation (int8/fp16), pruning, knowledge distillation, ONNX/TensorRT export, batching, caching, smaller architectures, GPU vs CPU choice.

## Quick interview Q&A
- **What is backpropagation?** Applying the chain rule to compute the gradient of the loss with respect to every weight.
- **Why ReLU over sigmoid?** No saturation for positive inputs, cheaper, mitigates vanishing gradients.
- **BatchNorm vs LayerNorm?** Across batch vs across features; LayerNorm for sequences/transformers.
- **What is dropout and how does it behave at eval?** Randomly zeroes units while training; disabled at eval (outputs scaled consistently).
- **Why do residual connections help?** Provide gradient shortcuts, enabling very deep networks.
- **Explain self-attention.** Each token forms Q, K, V; attention weights = softmax(QK^T/sqrt(d)); output = weighted sum of V.
- **Why divide by sqrt(d_k)?** Keeps dot products from growing large and saturating softmax.
- **LSTM vs transformer?** LSTM sequential, long-range limited; transformer parallel with global attention but O(n^2).
- **What causes overfitting in deep nets and how to fix?** Too much capacity or too little data; regularise, augment, early stop.
- **How do you pick a learning rate?** LR range test, schedules with warmup, watch loss curves.
- **What is transfer learning?** Reusing a pretrained model's knowledge for a new task.
- **Adam vs SGD?** Adam adapts per-parameter steps and converges fast; SGD with momentum sometimes generalises better.
