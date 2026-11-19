# PyTorch: Interview Notes and Code Patterns

## 1. Tensors
```python
import torch
x = torch.tensor([[1., 2.], [3., 4.]])
torch.zeros(2, 3); torch.randn(2, 3); torch.arange(10); x.shape; x.dtype; x.device
x.view(1, 4)      # needs contiguous memory;  x.reshape(1, 4) is safer
x.T; x.permute(1, 0); x.unsqueeze(0); x.squeeze(); x.transpose(0, 1)
a @ b             # matrix multiply;  a * b is element-wise (broadcasting applies)
x.to("cuda"); x.cpu(); x.numpy()   # numpy() shares memory on CPU
```
- **Broadcasting:** trailing dimensions are aligned; size-1 dims expand.
- `view` vs `reshape`: `view` never copies and requires compatible strides; `reshape` copies if needed.
- `clone()` copies, `detach()` cuts from the graph, `.item()` gets a Python number.

## 2. Autograd
```python
w = torch.randn(3, requires_grad=True)
loss = ((w * x).sum() - y) ** 2
loss.backward()           # populates w.grad
w.grad.zero_()            # gradients ACCUMULATE; zero them each step
with torch.no_grad(): ... # inference, no graph (saves memory)
torch.inference_mode()    # even faster for inference
```
PyTorch builds a **dynamic computation graph** during the forward pass; `backward()` traverses it. `retain_graph=True` keeps it for multiple backward calls.

## 3. Defining a model
```python
import torch.nn as nn
class MLP(nn.Module):
    def __init__(self, d_in, d_hid, n_cls, p=0.2):
        super().__init__()
        self.net = nn.Sequential(nn.Linear(d_in, d_hid), nn.ReLU(), nn.Dropout(p), nn.Linear(d_hid, n_cls))
    def forward(self, x): return self.net(x)       # logits
model = MLP(768, 256, 4).to(device)
```
- `nn.Module` registers parameters/submodules automatically. `model.parameters()`, `model.state_dict()`.
- `model.train()` enables dropout and BatchNorm updates; `model.eval()` disables them. Forgetting `eval()` is a classic bug.
- Output **logits** for `CrossEntropyLoss` (it applies log-softmax internally); `BCEWithLogitsLoss` for binary.

## 4. Dataset and DataLoader
```python
from torch.utils.data import Dataset, DataLoader
class TextDS(Dataset):
    def __init__(self, X, y): self.X, self.y = X, y
    def __len__(self): return len(self.y)
    def __getitem__(self, i): return self.X[i], self.y[i]
dl = DataLoader(TextDS(X, y), batch_size=32, shuffle=True, num_workers=4, pin_memory=True, drop_last=False)
```
`collate_fn` pads variable-length sequences. Shuffle training data only.

## 5. The training loop (memorise this)
```python
opt = torch.optim.AdamW(model.parameters(), lr=1e-3, weight_decay=1e-2)
sched = torch.optim.lr_scheduler.CosineAnnealingLR(opt, T_max=epochs)
loss_fn = nn.CrossEntropyLoss()
best = 0
for epoch in range(epochs):
    model.train()
    for xb, yb in train_dl:
        xb, yb = xb.to(device), yb.to(device)
        opt.zero_grad(set_to_none=True)
        loss = loss_fn(model(xb), yb)
        loss.backward()
        torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
        opt.step()
    sched.step()

    model.eval()
    correct = total = 0
    with torch.no_grad():
        for xb, yb in val_dl:
            pred = model(xb.to(device)).argmax(1).cpu()
            correct += (pred == yb).sum().item(); total += len(yb)
    acc = correct / total
    if acc > best: best = acc; torch.save(model.state_dict(), "best.pt")
```
Order per step: `zero_grad` -> forward -> loss -> `backward` -> (clip) -> `step`.

## 6. Mixed precision and accumulation
```python
scaler = torch.cuda.amp.GradScaler()
with torch.autocast("cuda", dtype=torch.float16):
    loss = loss_fn(model(xb), yb)
scaler.scale(loss).backward(); scaler.step(opt); scaler.update()
# gradient accumulation: divide loss by N, step every N batches
```
bf16 needs no scaler on supported hardware.

## 7. Saving, loading, device
```python
torch.save(model.state_dict(), "m.pt")
model.load_state_dict(torch.load("m.pt", map_location=device)); model.eval()
device = "cuda" if torch.cuda.is_available() else "cpu"
```
Save the **state_dict**, not the whole pickled model. Checkpoint optimizer and epoch to resume.

## 8. Common layers
`nn.Linear`, `nn.Conv2d`, `nn.MaxPool2d`, `nn.BatchNorm1d/2d`, `nn.LayerNorm`, `nn.Dropout`, `nn.Embedding`, `nn.LSTM(batch_first=True, bidirectional=True)`, `nn.GRU`, `nn.MultiheadAttention`, `nn.TransformerEncoderLayer`.

### Custom attention pooling over hidden states (as in your BiLSTM project)
```python
class AttnPool(nn.Module):
    def __init__(self, d): super().__init__(); self.w = nn.Linear(d, 1)
    def forward(self, h, mask):                 # h: (B, T, d), mask: (B, T) True for real tokens
        s = self.w(h).squeeze(-1).masked_fill(~mask, -1e9)
        a = torch.softmax(s, dim=1).unsqueeze(-1)
        return (a * h).sum(1)
```

## 9. Transfer learning with Hugging Face
```python
from transformers import AutoTokenizer, AutoModel, AutoModelForSequenceClassification
tok = AutoTokenizer.from_pretrained("roberta-base")
enc = AutoModel.from_pretrained("roberta-base")
for p in enc.parameters(): p.requires_grad = False        # frozen backbone -> train a small head
batch = tok(texts, padding=True, truncation=True, max_length=256, return_tensors="pt")
with torch.no_grad(): h = enc(**batch).last_hidden_state   # (B, T, 768)
```
Fine-tune with a small LR (2e-5), warmup, weight decay; use `Trainer` or a manual loop; LoRA via `peft` for large models.

## 10. Embeddings and similarity
```python
from sentence_transformers import SentenceTransformer
m = SentenceTransformer("BAAI/bge-small-en-v1.5")        # share one instance to save RAM
e = m.encode(["a", "b"], normalize_embeddings=True)
sim = e @ e.T                                            # cosine similarity when normalised
```

## 11. Performance and debugging tips
- Use `DataLoader(num_workers>0, pin_memory=True)`; keep data on GPU when possible; avoid `.item()`/`.cpu()` in hot loops.
- `torch.compile(model)` (PyTorch 2) can speed up; profile with `torch.profiler`.
- Check shapes with `print(x.shape)` at each stage; test on one batch; set seeds (`torch.manual_seed`).
- OOM: smaller batch, mixed precision, gradient checkpointing, accumulation, `del` and `torch.cuda.empty_cache()`.
- Detect NaNs: lower LR, clip gradients, check inputs and loss scaling; `torch.autograd.set_detect_anomaly(True)`.

## 12. Serving PyTorch models
Export with TorchScript or ONNX; load the model once at startup (FastAPI lifespan); batch requests; run inference under `torch.inference_mode()`; set `torch.set_num_threads`; add timeouts and a fallback; monitor latency and memory.

## 13. PyTorch vs TensorFlow
PyTorch: dynamic graphs, Pythonic, dominant in research and increasingly production (TorchServe, ONNX). TensorFlow/Keras: strong deployment tooling (TF Serving, TFLite), static graphs via `tf.function`. JAX: functional, XLA, fast research.

## Quick interview Q&A
- **Why call `zero_grad()`?** Gradients accumulate across `backward()` calls.
- **`model.eval()` vs `torch.no_grad()`?** `eval` changes layer behaviour (dropout, BN); `no_grad` stops building the graph. Use both for inference.
- **`CrossEntropyLoss` input?** Raw logits and integer class labels; do not apply softmax first.
- **What is a computation graph?** Record of operations used by autograd to backpropagate.
- **`view` vs `reshape`?** View needs contiguous compatible memory; reshape may copy.
- **How do you freeze layers?** `requires_grad = False` on those parameters (and pass only trainable ones to the optimizer).
- **How do you handle variable-length sequences?** Padding with attention masks (or packed sequences) via `collate_fn`.
- **How do you resume training?** Load model and optimizer state dicts and the epoch.
- **How do you make training reproducible?** Seeds, deterministic ops, fixed data order, logged versions.
- **Data parallelism?** `DistributedDataParallel` (one process per GPU) is preferred over `DataParallel`.
