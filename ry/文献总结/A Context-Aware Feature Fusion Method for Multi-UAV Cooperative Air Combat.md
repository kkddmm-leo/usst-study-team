**A Context-Aware Feature Fusion Method for
Multi-UAV Cooperative Air Combat**
==================================

## **1、创新点**

*   提出了一种基于上下文感知的自适应特征融合框架CAAFF。通过对时间序列态势数据的分层特征融合，CAAFF 框架不仅有效地捕捉了动态节点之间的复杂关系，还增强了信息的上下文嵌入能力，从而实现对敌方持续作战意图的准确解读。

## **2、实现方式**

### **状态空间：**

```math
s_{i}^{j} = [s_{p}^{ij}, s_{v}^{ij}, X_{ij}], \\
s_{p}^{ij} = [x_{i}^{j}, y_{i}^{j}, z_{i}^{j}, D_{i}^{j}] \\
s_{v}^{ij} = [v_{x}^{i}, v_{v}^{i}, v_{z}^{i}, \alpha, \alpha_{b}] \\
S_{i} = [Us_{i}^{m} | D_{i}^{m} \leq D_{obs}^{max}, Us_{i}^{n} | D_{i}^{n} \leq D_{obs}^{max} (i \neq n)]
\\X_{ij} = \omega_1 \alpha_r + \omega_2 \alpha_b + \omega_3 D_r^b + \omega_4 (\| \mathbf{v}_r \| - \| \mathbf{v}_b \|)


```

其中，m表示敌人，n表示队友。Xij表示态势值，我方无人机选择态势值最大的无人机为作战对象。

### **动作空间：**

```math
[n_x, n_z, \mu]
```

分别为：速度方向过载、法向过载、滚转角

## **奖励函数：**

```math
R_{s} = \begin{cases} \gamma_{S} \cdot X_{i,j} & , \exists UAV_{j}, D_{i}^{j} \leq D_{i}^{o} \\ 0 & , \forall UAV_{j}, D_{i}^{j} > D_{i}^{o} \end{cases}
\\R_{p} = \begin{cases} -\gamma_{p} & , pos(i) > Zone_{high} \text{ or } pos(i) < Zone_{low} \\ 0 & , Zone_{high} \geq pos(i) \geq Zone_{low} \end{cases}
\\R_{d} = \begin{cases} \gamma_{d} & , \text{Blue defeat Red} \\ -\gamma_{d} & , \text{Red defeat Blue} \\ 0 & , \text{else} \end{cases}
\\R_{w} = \begin{cases} \gamma_{w} & , Number_{blue} > Number_{red} \\ -\gamma_{w} & , Number_{blue} < Number_{red} \\ 0 & , Number_{blue} = Number_{red} \end{cases}
\\R = \begin{cases} R_{s} + R_{p} + R_{d} & , \text{game is not done} \\ R_{s} + R_{p} + R_{d} + R_{w} & , \text{game is done} \end{cases}
```

### **多层自适应特征融合模块（把“复杂观测”变成更有用的表征）**

*   **底层：Encoder-Decoder**&#x20;

    做降维保信息 每个 UAV 的原始观测 𝑥 𝑖 先过编码器得到低维特征 𝑧 𝑖 ，再用解码器重构 𝑆 𝑖 ，用 MSE 重构误差当作训练损失（可理解为自监督约束，保证降维不丢关键信息）。&#x20;
*   **中层：GACN（图注意力卷积**）

    建模无人机间关系 把所有 UAV 的特征组成节点特征矩阵 𝐻 ( 0 )，用带自环的邻接矩阵做图卷积传播。 再用图注意力计算邻居权重 𝛼 𝑖 𝑗 ​ ，并做多头注意力聚合得到节点新表征。&#x20;

    节点特征会送入 Gumbel-Softmax 得到“中层特征矩阵 𝐻 ′ （用于后续全局注意力）。&#x20;
*   &#x20;**高层：Multi-Head Self-Attention** **得到全局任务向量**

    用 𝐻 ′ 生成 𝑄 , 𝐾 , 𝑉 ，按 scaled dot-product attention 计算每个 head，再 concat 融合得到全局任务向量 𝑔 𝑡 ∗&#x20;

### **Context-Aware 模块：用“时序图”把历史意图融进当前表征**

论文认为只看当前时刻不够，需要把时间序列的关系编码进去，于是引入基于时序图神经网络的上下文模块：

*   **消息生成：**

对边 ( 𝑖 , 𝑗) 在时刻 𝑡 生成消息 𝑚 𝑖 𝑗 ( 𝑡 )

```math
m_{ij}(t) = f_{MSG}(s_{i}^{t-1}, s_{j}^{t-1}, e_{ij}, t_{ij})
```

*   **消息聚合：**

对邻居集合 𝑁 ( 𝑖) 聚合得到 𝑚 𝑖 ( 𝑡 )

```math
m_{i}^{(t)} = f_{AGG}(\{m_{ij}(t)\})
```

*   **状态更新：**

```math
s_{i}^{(t)} = f_{update}(s_{i}^{t-1}, m_{i}^{(t)})
```

*   &#x20;**时间编码**：把时间戳 𝑡 𝑖 通过 TE(⋅) 编码，文中选用 cosine（范围固定在 \[ − 1 , 1 ] 以避免数值不稳）。&#x20;



*   **输出嵌入观测：**

最终输出 ℎ 𝑖 𝐸

```math
h_{i}^{E} = f_{output}(s_{i}^{t-1}, m_{i}^{(t)}, t_{i}')

```

用这个嵌入去“替换传统观测”。

### **把 CAAFF 接到强化学习：CA\_MADDPG\_AFF**

*   actor/critic 不再输入原始观测 𝑠 𝑖，而是输入 CAAFF + context-aware 后的嵌入观测 ℎ 𝑖 𝐸&#x20;

