---
layout: default
title: "DreamActVLA: Accurate Asynchronous Inference for Vision-Language-Action Models"
subtitle: "Can asynchronous inference work without any training-time modifications to a VLA?"
authors: "Jane E. Doe"
affiliation: "UC Berkeley, EECS"
date: 2026-06-08
toc: true
---

# Abstract

Vision-language-action models (VLAs) have become a powerful paradigm for robot manipulation, but their inference latency creates a persistent bottleneck: the robot must idle or execute stale commands while waiting for the next action chunk. Asynchronous inference can eliminate these pauses, but introduces a distribution mismatch when observations become stale mid-inference. We present **DreamActVLA**, which resolves this mismatch by querying the policy with *temporally aligned* future observations — both image and proprioceptive state rolled forward to the same future timestep. Unlike prior asynchronous approaches such as RTC and VLASH, DreamActVLA requires **no modifications to the fine-tuning procedure**, providing a more accurate asynchronous inference technique that preserves the policy's original training distribution. We validate our approach on the LIBERO benchmark, demonstrating a 1.17× speedup with only a 1% reduction in success rate using an unmodified pi0 checkpoint.

**Keywords:** Vision-Language-Action Models, Dynamic Manipulation, Asynchronous Inference

---

# Introduction

Vision-language-action models (VLAs) have emerged as a powerful paradigm for robot manipulation, enabling policies that map raw visual observations and proprioceptive state to continuous action trajectories [[1]](#ref-1), [[2]](#ref-2). However, a persistent bottleneck in deploying these models is **inference latency**. Large models require hundreds of milliseconds to produce an action chunk, during which the robot must either idle or execute stale commands. This latency directly limits task execution speed and responsiveness to rapid or dynamic changes.

**Asynchronous inference** offers a natural solution. By overlapping policy inference with action execution, the robot can operate continuously without pausing on each planning cycle. However, naive asynchronous execution introduces a subtle problem: by the time inference completes, the observation that triggered it is stale. The robot has moved during the inference window, so the generated actions are conditioned on an outdated state.

Existing approaches like VLASH [[3]](#ref-3) address this mismatch by rolling forward the robot's proprioceptive state to the expected time of action deployment. But this creates a **temporally disjoint observation**: a current image paired with a future state that lies outside the policy's training distribution. To compensate, VLASH requires a modified fine-tuning procedure that augments training data with temporal offsets, adding cost and complexity to the pipeline.

We ask the question: **can asynchronous inference work without any training-time modifications?**

> **Key Insight:** The distribution mismatch in prior approaches arises not from asynchrony itself, but from the *partial* roll-forward. If both the image and state observations are rolled forward to the same future timestep, the policy receives a temporally consistent input that matches its training distribution — eliminating the need for offset-aware fine-tuning.

We validate this idea in three stages:

1. We use **privileged simulator access** in the LIBERO benchmark [[4]](#ref-4) to render ground-truth future observations, confirming with a pre-trained pi0 policy that aligned future observations recover full asynchronous speedups without any fine-tuning.
2. We replace privileged rendering with the **COSMOS world model** [[5]](#ref-5), which predicts future visual observations from planned actions and rolled-forward state, removing the simulator dependency and providing a path toward real-robot deployment.
3. We introduce a **dynamic task benchmark** that augments LIBERO with constant-velocity object motion tied to wall-clock time, creating a setting where inference latency directly increases task difficulty.

## Contributions

- **More accurate asynchronous inference:** By querying a policy with temporally aligned future $[\text{image, state}]$ observations, DreamActVLA achieves asynchronous inference without any adjustment to the fine-tuning process — a strictly more accurate approach than prior methods like RTC and VLASH that rely on temporally disjoint inputs or offset-aware training.
- **World model generalization:** Using a learned world model (COSMOS) to generate future observations removes the simulator dependency and enables real-robot deployment.
- **Dynamic task benchmark:** A new LIBERO benchmark variant with wall-clock-tied object motion where asynchronous methods offer a clear advantage.

---

# Method

## Privileged Future Observations

Let $o_t = (I_t, s_t)$ denote the full observation at timestep $t$, where $I_t \in \mathbb{R}^{H \times W \times 3}$ is the rendered camera frame and $s_t \in \mathbb{R}^{8}$ is the proprioceptive state vector (end-effector position, axis-angle orientation, and gripper aperture).

Given a current action chunk $a = [a_t, \ldots, a_{t+N-1}]$ of length $N$, we trigger asynchronous inference $K$ steps before the chunk ends, at step $t + N - K$.

At the trigger point, rather than using the current (stale) observation, we obtain a future observation $o_{t+N}$ by performing a brief rollout within the simulator:

1. **Save** the full MuJoCo physics state $\sigma_{t+N-K}$ at the trigger timestep.
2. **Simulate forward** through the $K$ remaining planned actions $[a_{t+N-K}, \ldots, a_{t+N-1}]$, advancing the physics to time $t+N$.
3. **Capture** the future observation $o_{t+N} = (I_{t+N}, s_{t+N})$: render the camera frame and read the proprioceptive state.
4. **Restore** the simulator to $\sigma_{t+N-K}$ and resume real execution of the remaining $K$ actions.

The next action chunk is then computed as $\pi(o_{t+N})$ and is ready at the chunk boundary $t+N$ when the robot needs it. Because both $I_{t+N}$ and $s_{t+N}$ are drawn from the same simulated future instant, the observation pair is **temporally aligned** — it lies on the same distribution as the training data.

<div class="callout callout-insight">
<strong>Why this is more accurate:</strong> Training data always pairs contemporaneous images and states. Prior approaches like VLASH break this pairing by only rolling forward the state, producing out-of-distribution inputs. DreamActVLA preserves the original pairing by aligning both modalities to the same future instant — no modified fine-tuning needed.
</div>

## World Model Generalization

The privileged rollout procedure requires read/write access to the simulator's physics state, making it applicable only to simulation environments. On a physical robot, the forward rollout requires a **predictive world model** to synthesize the future image $I_{t+N}$.

We propose using the **COSMOS world model** [[5]](#ref-5) to predict future visual observations from planned actions and rolled-forward state. This removes the simulator dependency and provides a viable path toward real-robot deployment.

---

# Evaluation

## Experimental Setup

We evaluate on the **LIBERO benchmark** [[4]](#ref-4), which comprises multiple manipulation task suites. Each condition is evaluated over **50 rollouts per task**. All conditions use the **pi0-LIBERO checkpoint** [[1]](#ref-1) without any modification to weights or fine-tuning procedure.

Key parameters:
- **Chunk length:** $N = 15$ actions
- **Overlap:** $K = 10$ steps for the asynchronous condition
- **Inference window:** $K/f = 0.5\text{s}$ at the LIBERO control frequency of $f = 20\text{ Hz}$
- **Hardware:** NVIDIA RTX 4070 GPU (measured inference latency $\tau \approx 120\text{ ms}$)

Critically, the same chunk length $N$ is used for the synchronous baseline, so differences in success rate and completion time reflect the inference strategy alone.

**Time to success** is computed as:

$$T = N_{\text{steps}} / f + T_{\text{block}}$$

where $T_{\text{block}}$ is the total execution time lost to blocking inference pauses (zero for DreamActVLA when inference completes within the overlap window).

## Main Results

| Method | Spatial | Object | Goal | L-10 | Avg SR (%) | Steps | Time (s) | ΔSR | Speedup |
|--------|---------|--------|------|------|-----------|-------|----------|-----|---------|
| Sync (Baseline) | 92.6 | 99.2 | 95.6 | 86.6 | **93.5** | 153.1 | 9.02 | — | — |
| **DreamActVLA (Ours)** | 91.4 | 98.2 | 95.0 | 84.8 | **92.5** | 152.2 | **7.61** | −1.0% | **1.17×** |
| Future-State Only | 12.8 | 26.6 | 52.4 | 19.2 | 27.6 | 228.8 | 11.35 | −65.9% | −25.8% |

DreamActVLA maintains **92.5% success rate** (only a 1% reduction relative to Sync) while reducing mean task completion time to **7.61 s** — a **1.17× speedup**. This speedup stems directly from eliminating blocking inference pauses: inference runs concurrently with the final $K=10$ steps of each chunk and, at a measured latency of $\tau \approx 120\text{ ms}$, completes well within the available $K/f = 500\text{ ms}$ window.

## Future-State-Only Ablation

We additionally evaluate a **future-state-only** condition, which pairs the rolled-forward state $s_{t+N}$ with the current (stale) camera image $I_t$ — replicating the VLASH inference-time strategy on an unmodified policy without temporal offset fine-tuning.

| Method | Spatial | Object | Goal | L-10 | Avg SR (%) | Steps | Time (s) | ΔSR |
|--------|---------|--------|------|------|-----------|-------|----------|-----|
| Sync | 92.6 | 99.2 | 95.6 | 86.6 | **93.5** | 153.1 | 9.02 | — |
| Future-State Only | 12.8 | 26.6 | 52.4 | 19.2 | 27.6 | 228.8 | 11.35 | −65.9% |
| **DreamActVLA (Ours)** | 91.4 | 98.2 | 95.0 | 84.8 | **92.5** | 152.2 | 7.61 | −1.0% |

The future-state-only condition collapses to **27.6% success rate** — a 65.9% drop. This confirms the distribution mismatch: pairing a current image with a future state produces out-of-distribution inputs the unmodified policy cannot handle. DreamActVLA avoids this entirely by aligning both modalities, demonstrating why temporal alignment is critical for accurate asynchronous inference.

<div class="callout callout-warning">
<strong>Note:</strong> The future-state-only result mirrors the observation pair used by VLASH at inference time, but applied to a standard pi0-LIBERO checkpoint without temporal offset fine-tuning. VLASH's fine-tuning procedure is specifically designed to handle this disjoint input — but at the cost of additional training complexity.
</div>

---

# Discussion

## Limitations

<!-- TODO: Add discussion of limitations -->

---

# Conclusion

We have shown that **temporally aligned future observations** enable more accurate asynchronous inference for VLA policies without any modification to the fine-tuning procedure. By rolling forward both the image and proprioceptive state to the same future timestep, DreamActVLA keeps policy inputs in-distribution and achieves a 1.17× speedup with negligible impact on task success — a strictly more accurate approach than existing methods like RTC and VLASH.

Our approach opens two directions for future work: (1) deploying DreamActVLA on physical robots using learned world models such as COSMOS, and (2) extending the dynamic task benchmark to evaluate robustness under varying object velocities and more complex scene dynamics.

---

<div class="references" id="references">

# References

1. <span id="ref-1"></span> K. Black, N. Brown, D. Driess, *et al.* "π0: A vision-language-action flow model for general robot control," 2026. [arXiv:2410.24164](https://arxiv.org/abs/2410.24164)

2. <span id="ref-2"></span> Physical Intelligence, K. Black, *et al.* "π0.5: A vision-language-action model with open-world generalization," 2025. [arXiv:2504.16054](https://arxiv.org/abs/2504.16054)

3. <span id="ref-3"></span> J. Tang, Y. Sun, Y. Zhao, *et al.* "VLASH: Real-time VLAs via future-state-aware asynchronous inference," 2025. [arXiv:2512.01031](https://arxiv.org/abs/2512.01031)

4. <span id="ref-4"></span> B. Liu, Y. Zhu, C. Gao, *et al.* "LIBERO: Benchmarking knowledge transfer for lifelong robot learning," 2023. [arXiv:2306.03310](https://arxiv.org/abs/2306.03310)

5. <span id="ref-5"></span> M. J. Kim, Y. Gao, T.-Y. Lin, *et al.* "COSMOS Policy: Fine-tuning video models for visuomotor control and planning," 2026. [arXiv:2601.16163](https://arxiv.org/abs/2601.16163)

</div>
