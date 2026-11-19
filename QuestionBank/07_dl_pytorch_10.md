# Deep Learning and PyTorch: 10 Most-Asked Questions

**Q1. What is a tensor and how does it differ from a NumPy array?**
A multi-dimensional array like NumPy's, but PyTorch tensors can live on a GPU and track operations for automatic differentiation.

**Q2. What is autograd and what does `requires_grad` do?**
Autograd builds a dynamic computation graph during the forward pass and computes gradients with `backward()`. Tensors with `requires_grad=True` are tracked; gradients accumulate in `.grad`.

**Q3. Walk through a typical PyTorch training loop.**
For each batch: move data to the device, `optimizer.zero_grad()`, forward pass, compute loss, `loss.backward()`, (clip gradients), `optimizer.step()`. After each epoch run validation in `eval()` mode under `no_grad`, step the scheduler, save the best checkpoint.

**Q4. Why call `zero_grad()`, `backward()` and `step()`?**
`zero_grad` clears accumulated gradients; `backward` computes new gradients; `step` updates parameters using them.

**Q5. `model.train()` vs `model.eval()`, and `no_grad` vs `inference_mode`?**
`train()`/`eval()` switch layer behaviour (dropout, batch norm). `no_grad` stops building the graph; `inference_mode` is stricter and faster for pure inference. Use both when evaluating.

**Q6. What are `Dataset` and `DataLoader`?**
`Dataset` defines `__len__` and `__getitem__`; `DataLoader` batches, shuffles and loads in parallel workers, with a `collate_fn` to pad variable-length samples.

**Q7. How do you save and load a model?**
Save `model.state_dict()` (plus optimizer and epoch for resuming) with `torch.save`; load with `model.load_state_dict(torch.load(path, map_location=device))` and call `model.eval()`.

**Q8. What causes vanishing and exploding gradients and how do you handle them?**
Repeated multiplication of small or large factors through depth. Use ReLU-family activations, good initialisation, normalisation, residual connections, LSTM/GRU gates, and gradient clipping.

**Q9. How do you fine-tune a pretrained model and reduce GPU memory?**
Load pretrained weights, replace the head, optionally freeze the backbone, use a small learning rate with warmup. For memory: smaller batch, mixed precision (AMP), gradient accumulation, gradient checkpointing, parameter-efficient fine-tuning (LoRA).

**Q10. How would you debug a model that is not learning, and what is `DistributedDataParallel`?**
Overfit a tiny batch first; check data and labels, loss function inputs (logits), learning rate, zero_grad placement, normalisation, train/eval mode. `DistributedDataParallel` runs one process per GPU with synchronised gradients and is preferred over `DataParallel`; `torch.compile` can speed up models.
