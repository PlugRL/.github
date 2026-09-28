# PlugRL

Reinforcement learning training and the environments it learns from, split into
two processes and joined by a written protocol: WebSocket and msgpack, with a
feedback return channel.

<img src="https://raw.githubusercontent.com/PlugRL/.github/main/profile/flowchart.svg" width="760"
     alt="Two boxes. On the left, plugrl-env-client, which holds no policy and runs Gymnasium, MuJoCo or LIBERO. On the right, plugrl-server, which holds the policy and the algorithm. Observation goes from client to server, action from server to client, and feedback - reward and termination - from client to server, drawn thicker and green.">


The third arrow is the one that matters. Serving an inference model needs the
first two; learning from what happened needs the third, and the protocol
specifies it rather than leaving it to a convention.

## What runs on it

<a href="https://plugrl.github.io/#what-runs-on-it"><img src="https://plugrl.github.io/media/coverage-grid.jpg" width="100%"
   alt="Sixteen cells, four policy-algorithm pairs on HalfCheetah, Hopper, Walker2d and robomimic square, each with a frame from its trained policy, a training curve and a status. Every pair learns every task. On square each starts from a pretrained policy, and fpo-policy with FPO passes the bar on two of three seeds."></a>

Every combination of the two MLP policies and the two algorithms on four
tasks, and the baseline they are measured against, a Gaussian MLP with PPO.
All sixteen learn. [On the project page](https://plugrl.github.io/#what-runs-on-it)
each cell plays its clip and shows the two commands that trained it. None of
the servers that trained these has MuJoCo, robosuite or gymnasium installed;
the env clients carry them, in two separate environments.

## What the split buys, and what it costs

The training server is 6.5G and wants a GPU. The environment side needs
neither, and need not be Python: it fits on a different class of machine from
the trainer. The boundary between them is cheap.

| | Question | Answer |
|---|---|---|
| [E2](https://github.com/PlugRL/plugrl-server/tree/main/experiments/e2-cross-language) | Does an env client have to be this codebase, or Python? | **No** - an 843-line C++ client with no third-party libraries drove a real training server |
| [E12](https://github.com/PlugRL/plugrl-server/tree/main/experiments/e12-cuda-free-rollout) | Does a rollout machine need CUDA? | **No** - LIBERO's env client goes from 7.8G to **3.4G**, with no nvidia wheels |
| [E13](https://github.com/PlugRL/plugrl-server/tree/main/experiments/e13-gpu-free-rendering) | Or a GPU to render on? | **No, at 1.91x** the wall clock - ten clients rendering on the CPU, 30 of 30 episodes successful |
| [E7](https://github.com/PlugRL/plugrl-server/tree/main/experiments/e7-cross-machine) | What does the boundary cost once packets leave the machine? | **+0.52 ms** on a 184 KiB observation, measured from a VM to its host |
| [E10](https://github.com/PlugRL/plugrl-server/tree/main/experiments/e10-vla-forward-cost) | Is that cheap beside a VLA forward pass? | **Yes** - the split is 1.3-3.6% of a step |

Every experiment directory carries its data and a `FINDINGS.md` that states
what the result does **not** support.

## A real VLA through it

A full-size pi0.5 runs end to end through the boundary on LIBERO. The
unmodified checkpoint scored 99 of 100 on `libero_spatial` and 185 of 200 on
`libero_10`, against openpi's published 98.8 and 92.4, and the server's record
of episodes and steps reconciles exactly with the clients'
([E11](https://github.com/PlugRL/plugrl-server/tree/main/experiments/e11-vla-rl-libero)).
Fine-tuning it with reinforcement learning through PlugRL has not made it
better yet; that record is on [its own page](https://plugrl.github.io/vla/).

## Repositories

| | |
|---|---|
| [plugrl-server](https://github.com/PlugRL/plugrl-server) | Training side: policy, algorithm, checkpoints, and the experiments |
| [plugrl-env-client](https://github.com/PlugRL/plugrl-env-client) | Environment side: steps envs, asks for actions, returns feedback |
| [plugrl-protocol](https://github.com/PlugRL/plugrl-protocol) | The specification, its checkable clauses as tests, and two reference env clients |
| [plugrl.github.io](https://github.com/PlugRL/plugrl.github.io) | Documentation, in English and 中文 |

Start at the [documentation](https://plugrl.github.io) - the quickstart trains
FPO on HalfCheetah with no GPU and nothing to download.

## Who

PlugRL is built by [Chenhao Lu](https://github.com/CTP314),
[Zuo Gou](https://github.com/tactino) and
[Zilin Kang](https://github.com/nothingbutbut).
