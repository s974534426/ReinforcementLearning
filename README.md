# Deep Reinforcement Learning

## 1. 算法

### Model free

- [x] A2C
- [x] DDPG
- [x] TD3
- [x] SAC
- [x] TRPO(bad)
- [x] PPO(bad)
- [ ] DDPG-HER
- [ ] ACER
- [ ] ACKTR

### Imitation Learning

- [ ] GAIL

## 2. 代码结构

```
TODO
```

## 3. 实验结果

### Logs

记录在对应算法的```logs```文件夹对应环境中，进入目录使用命令```tensorboard --logdir=.```查看。或查看对应算法目录下的```README.md```文件。

### 模型

运行算法下的```test.py```文件测试，模型是在GPU上训练出来的，所以如果在CPU上运行，需要做一定的修改。

## 4. Trick

虽然这些trick可能适用于不同的情况，但大部分代码中都是使用了尽量多的trick来提高性能，在论文中却没有分析这些trick对结果的影响，这样一方面可能导致论文中提出的影响性能的改进并不是由于这部分，而是由于各种trick(比如PPO和TRPO)

### 4.1. 通用trick

### 4.2. On policy trick

### 4.3. Off policy trick

1. Target net，terget net通过提供一个稳定的更新方向，使得算法更加稳定。DQN，DDPG
2. Experience replay，DQN，DDPG
3. Action repeat，在Atari的游戏中，重复执行多次当前动作。DQN，DDPG
4. Actor和critic的权重共享，在Arari游戏中，共享卷积层的权重
5. Soft update，$\theta'=\tau \theta + (1-\tau)\theta', \tau << 1$，使得目标值更加稳定，学习过程接近于监督学习。DDPG，TD3，SAC
6. Deterministic policy加入随机噪声，促进探索。DDPG中加OU噪声，TD3中加高斯噪声
7. Running mean std，online地利用当前采集到的所有状态对state进行归一化，能够让训练更加稳定(但也增大了计算开销，运行速度会变慢)，见```common/utils.py:ZFilter```。所有的算法都可以用(有一定的效果。同时需要注意的是训练时要记录当前的state的平均值，在测试时使用，而不是在测试时重新收集数据，计算平均值)
8. Twin Q network，$y = r + \gamma \min (Q_1(s', \pi(s')), Q_2(s', \pi(s')))$，缓解Q值估计过高的问题。TD3，SAC
9. Delayed update，减缓更新actor，target critic和target actor。TD3
10. GAE，更好的advantage估计,$\widehat{A}_t^{GAE(\gamma, \lambda)} = \sum_{l=0}^{\infty}(\gamma\lambda)^l\delta_{t+l}^V$，用$\lambda$控制方差和偏差。应该是所有涉及advantage的算法都能用的，但GAE需要用到整条轨迹的数据，所以一般在on-policy的TRPO、PPO这种先收集一条轨迹的数据再优化的算法中可以使用，而且应该是TRPO和PPO的标配了。(off-policy也可以先收集一条轨迹的数据，再进行优化，似乎好像说这样效果会更好，但off-policy算法的一般实现中都是与环境交互一次，进行一次更新)
11. 接GAE，在计算GAE后，一般都会对GAE进行归一化
12. Reparameterization trick。SAC
13. 加入熵。一种方法是熵直接加入Q函数中，在最大化Q的同时最大化熵，例如SAC。另一种策略优化时加入熵作为正则化项。
14. 多次更新，对每个batch的数据，更新时多次更新，这样比每次只更新一次效果更好。大部分算法都可以用，像PPO的baselines实现中就这样做了
15. 正交初始化，很多人建议RL中使用正交初始化(为什么呢?)
16. HER，针对sparse reward提出的一种方法，可以应用在任意的off policy算法上

## 5. 超参数调节

初期可以选择一套比较合理的超参数，比如学习率为```3e-4```，甚至更小，batch为```64```等等，超参数如果不是差距过大(比如学习率直接设置为```0.1```)，算法不会完全学不到东西。

如果完全学不到，可能是由于环境过于复杂而选择的算法实现过于基础，或者算法实现错误，这种情况下应该尝试找bug、加入trick，直接换算法，而不是疯狂调参(比如将batch大小从```64```调到```128```或者```256```，这大概率是没有用的。。。)

## 5. TODO

实现的代码比较简单体现了算法本身的思想，但不利于修改。以后最好可以将策略网络、经验池等统一借口作为参数传递给算法；加入n steps、随机种子、状态是否归一化、奖赏是否reshape等参数，利于比较使用不同网络、使用不同的经验池等对最终结果的影响；log中记录Q值、动作的概率等更多的数据；增加对输入数据shape的assert。
