# Zhixing Train

## Open SFT and RL Infrastructure

`zhixing-train` is the training orchestration layer of Zhixing Stack. It consumes versioned dataset releases and launches supervised fine-tuning, GRPO, RLVR, and related optimization workflows.

它对应原来的 `slime-infra`。Slime 是当前的重要 backend，但不是 Zhixing Stack 的公共协议或总品牌。

## 中文定位

```text
DatasetRelease
TrainingRecord
Base Model
Training Config
        ↓
   Zhixing Train
        ↓
CheckpointManifest
Train Metrics
Model Artifact
```

负责：

- SFT 配置和启动
- GRPO/RLVR 配置和启动
- Slime adapter
- rollout provider 对接
- checkpoint 管理
- metrics 和 run manifest
- 模型产物注册
- 与 Runtime、Bench 的兼容性检查

## 示例入口

```bash
zhixing-train sft \
  --dataset artifacts/datasets/coder-sft.yaml \
  --model <base-model> \
  --backend slime \
  --config configs/sft.yaml
```

```bash
zhixing-train rl \
  --policy artifacts/checkpoints/sft-step-1000 \
  --env-registry artifacts/envs/registry.jsonl \
  --backend slime \
  --config configs/grpo.yaml
```

## Backend 设计

```text
Zhixing Train
├── backends/slime
├── backends/verl
├── backends/trl
└── backends/custom
```

上层只依赖 `Zhixing Train` 的训练和产物协议，不直接依赖 Slime 的内部目录或 launcher。

## 边界

训练层不负责：

- 生成任务和环境
- 实现 verifier
- 直接修改 runtime event schema
- 将 hidden oracle 写入训练数据
- 维护 benchmark score 定义

数据由 [Zhixing Data](https://github.com/Zeng-Weijun/zhixing-data) 生产，执行和 rollout 由 [Zhixing Runtime](https://github.com/Zeng-Weijun/zhixing-runtime) 提供，最终评测由 [Zhixing Bench](https://github.com/Zeng-Weijun/zhixing-bench) 负责。

## English

`zhixing-train` provides backend-independent orchestration for SFT, GRPO, RLVR, and future optimization methods.

The public interface is defined in terms of dataset releases, training runs, checkpoints, and metrics. Slime is implemented as an adapter and can be replaced without changing the rest of the Zhixing Stack.

The trainer consumes data; it does not own task generation, environment verification, or benchmark scoring.

## Status

Public alpha scaffold. Training launchers and backend adapters will be added incrementally.
