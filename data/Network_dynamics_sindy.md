# Network-SINDy 网络动力学交接文档

## 1. 方法说明

本轮使用的是 Network-SINDy / STLSQ，而不是 PSE-main。它适合当前要的标准网络动力学形式：

```text
dx_i/dt = f(x_i) + sum_j W_ji*g(x_i,x_j)
```

其中：

```text
W_ji = A_sub[j,i] / sum_k A_sub[k,i]
```

自环被去掉，所以自身动力学只进入 `f(x_i)`，邻居影响只进入 `g(x_i,x_j)`。

本轮目录：

```text
/home/yaoyi/斑马鱼/pse_network_dynamics_runs/network_sindy_20260514
```

核心文件：

```text
network_sindy_runner.py
results/summary.csv
results/NETWORK_SINDY_REPORT.md
results/size_*/<network>/best_expr_pairwise_required.txt
results/size_*/<network>/selected_models.csv
results/size_*/<network>/candidates.csv
results/size_*/<network>/pareto.csv
```

## 2. 实验设置

数据沿用之前筛出的活动性较强子图：

- `200` 节点子图
- `500` 节点子图

三种网络结构：

- `gaussian_te`
- `joint_granger`
- `dbn_refine`

验证方式：

- 按时间顺序切分，而不是随机打乱。
- 前 `70%` 时间点训练。
- 后 `30%` 时间点验证。
- 每个网络最多使用 `350000` 个训练样本和 `150000` 个验证样本。

候选库：

```text
f(x_i): 1, x_i, x_i^2, x_i^3, sin(x_i), cos(x_i)-1, tanh(x_i), exp(x_i)-1, log1p_abs(x_i)
```

```text
g(x_i,x_j): x_j-x_i, x_i*(x_j-x_i), (x_j-x_i)^2, (x_j-x_i)^3,
            sin(x_j-x_i), cos(x_j-x_i)-1, exp(x_j-x_i)-1
```

选择规则：

- 使用 STLSQ 稀疏回归。
- 默认选择含有非零 `g` 的模型，保证交付结果是网络动力学表达式。
- `summary.csv` 里同时记录了不强制 `g` 时的 `unconstrained_val_mse`，用于判断网络项是否带来明显收益。

## 3. 全部结果

| subset_size | network | val_mse | complexity | 结论 |
| --- | --- | ---: | ---: | --- |
| 200 | `dbn_refine` | 0.045203 | 11 | 误差最低之一，但 `g` 比较复杂 |
| 200 | `gaussian_te` | 0.045255 | 8 | 简洁、可解释 |
| 200 | `joint_granger` | 0.045711 | 11 | 可用，但略复杂 |
| 500 | `dbn_refine` | 0.045132 | 9 | 500 节点中误差最低，`g` 很简洁 |
| 500 | `gaussian_te` | 0.045468 | 10 | 可解释性较好 |
| 500 | `joint_granger` | 0.045275 | 13 | 误差接近最低，但 `g` 更复杂 |

整体看，Network-SINDy 结果的验证误差约在 `0.0451-0.0457`，比之前的 pairwise 稀疏版约 `0.0525-0.0529` 更低。这说明按时间切分后，这个方法在当前候选库上拟合更稳定。

## 4. 建议主汇报的 500 节点表达式

### 4.1 `dbn_refine / 500`

这是 500 节点里验证误差最低的结果。

```text
f(x_i) =
-39.7916*x_i
+ 2.78975*x_i^2
+ 10.2974*x_i^3
+ 54.6088*sin(x_i)
- 5.61983*(cos(x_i)-1)
- 4.62837*tanh(x_i)
- 10.2309*(exp(x_i)-1)
```

```text
g(x_i,x_j) =
1.62217*(x_j - x_i)
- 1.4575*sin(x_j-x_i)
```

完整形式：

```text
dx_i/dt =
f(x_i) + sum_j W_ji * [1.62217*(x_j - x_i) - 1.4575*sin(x_j-x_i)]
```

解释：

- `g` 非常简洁，只包含线性差分和正弦差分。
- 可以解释为“扩散型耦合 + 非线性相位差修正”。
- 这是目前最适合作为最终网络动力学表达式的候选之一。

### 4.2 `gaussian_te / 500`

```text
f(x_i) =
-25.9702*x_i
+ 2.8245*x_i^2
+ 5.20885*x_i^3
+ 38.6938*sin(x_i)
- 8.89855*tanh(x_i)
- 4.22833*(exp(x_i)-1)
```

```text
g(x_i,x_j) =
-2.93493*(x_j - x_i)
+ 2.3904*x_i*(x_j - x_i)
+ 2.71359*sin(x_j-x_i)
- 4.31637*(cos(x_j-x_i)-1)
```

解释：

- `g` 中有 `x_i*(x_j-x_i)`，说明邻居影响受目标节点自身状态调制。
- 可以作为 `dbn_refine / 500` 的对照：它不是纯差分耦合，而是状态依赖耦合。

### 4.3 `joint_granger / 500`

```text
f(x_i) =
-52.7921*x_i
+ 1.94332*x_i^3
+ 64.5175*sin(x_i)
+ 11.1226*(cos(x_i)-1)
- 24.111*tanh(x_i)
+ 11.9027*(exp(x_i)-1)
```

```text
g(x_i,x_j) =
-19.2682*(x_j - x_i)
+ 2.56372*x_i*(x_j - x_i)
- 3.68798*(x_j - x_i)^2
+ 2.78877*(x_j - x_i)^3
+ 20.102*sin(x_j-x_i)
- 13.9212*(cos(x_j-x_i)-1)
- 1.1073*(exp(x_j-x_i)-1)
```

解释：

- 误差接近 `dbn_refine / 500`，但 `g` 明显复杂。
- 可以解释为强非线性相互作用核，不建议作为最简洁主公式。

## 5. 重要风险

### 5.1 不强制 `g` 时，最稀疏模型倾向自项

`summary.csv` 中 `unconstrained_has_g=False` 对所有网络都成立。这说明如果只看最稀疏、最低验证误差附近的模型，算法会倾向选择只有 `f(x_i)` 的自项动力学。

因此本轮交付的网络动力学结果是“强制保留网络项”的结果。这个选择符合任务要求，但要在汇报时说明。

### 5.2 `f(x_i)` 仍然有较强非线性抵消

很多 `f` 表达式里同时出现：

```text
x_i
x_i^2
x_i^3
sin(x_i)
tanh(x_i)
exp(x_i)-1
```

这些函数在钙活动数据的取值范围内可能高度相关，因此系数不能过度解释为真实生物物理常数。更稳妥的解释是：`f` 捕捉节点自身恢复/非线性响应，`g` 捕捉网络耦合形态。

### 5.3 `g` 的结论比系数更可靠

三种 500 节点网络都给出了基于 `x_j-x_i` 的相互作用核：

- `dbn_refine`: 线性差分 + 正弦差分
- `gaussian_te`: 差分 + 状态依赖差分 + 正弦/余弦差分
- `joint_granger`: 更高阶差分 + 三角函数 + 指数差分

因此可以较稳妥地说：当前数据支持“节点间活动差驱动的非线性耦合”，但不应把每个系数当作精确机制参数。

## 6. 推荐汇报口径

如果只交一个最终表达式，建议交：

```text
Network-SINDy, 500-node dbn_refine:

dx_i/dt = f(x_i) + sum_j W_ji*g(x_i,x_j)

f(x_i) =
-39.7916*x_i
+ 2.78975*x_i^2
+ 10.2974*x_i^3
+ 54.6088*sin(x_i)
- 5.61983*(cos(x_i)-1)
- 4.62837*tanh(x_i)
- 10.2309*(exp(x_i)-1)

g(x_i,x_j) =
1.62217*(x_j - x_i)
- 1.4575*sin(x_j-x_i)
```

理由：

- 500 节点主结果，比 200 节点更稳。
- 验证误差最低：`0.045132`。
- `g` 很简洁，容易解释为差分耦合加非线性修正。
- 形式严格满足 `f(x_i)+sum_j W_ji*g(x_i,x_j)`。

如果导师质疑网络项是否真的必要，需要说明：自项模型在数值上也能解释不少方差，当前结果是按照“必须给出网络动力学表达式”的目标强制选择非零 `g`。

