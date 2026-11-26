# Day 02 · Spinning Up: key RL concepts and the PPO-Clip objective

**Sources (read in full on 2026-11-26):**
- [Part 1: Key Concepts in RL](https://spinningup.openai.com/en/latest/spinningup/rl_intro.html)
- [Proximal Policy Optimization](https://spinningup.openai.com/en/latest/algorithms/ppo.html)

Notation: $\pi_\theta$ is the policy, $\pi_{\theta_k}$ is the policy that collected the data in the current iteration, $A$ is the advantage, $\epsilon$ is the clip ratio (`clip_ratio`, default 0.2 in the Spinning Up code).

---

## 1. Context from Part 1 that PPO needs

- The agent sees an observation, picks an action from its policy, and gets a reward. The goal is to maximise expected return.
- Value functions: $V^\pi(s)$ is the expected return from state $s$ under $\pi$. $Q^\pi(s,a)$ is the expected return from taking $a$ in $s$ and then following $\pi$.
- Bellman equations say the value of a state is the reward you expect plus the value of where you land next.

## 2. The clipped objective

PPO-Clip updates the policy by maximising

$$
L(s,a,\theta_k,\theta) = \min\Big( r(\theta)\, A^{\pi_{\theta_k}}(s,a),\;\; \text{clip}\big(r(\theta),\, 1-\epsilon,\, 1+\epsilon\big)\, A^{\pi_{\theta_k}}(s,a) \Big)
$$

where the probability ratio is

$$
r(\theta) = \frac{\pi_\theta(a \mid s)}{\pi_{\theta_k}(a \mid s)}
$$

- $r = 1$ means the new policy gives the same probability to $a$ as the old one.
- $r > 1$ means the new policy makes $a$ more likely. $r < 1$ means less likely.
- The clip keeps $r$ inside $[1-\epsilon, 1+\epsilon]$ when the objective is evaluated.

The Spinning Up page also gives a simpler equivalent form, which is what their code implements:

$$
L = \min\big( r\,A,\; g(\epsilon, A) \big), \qquad
g(\epsilon, A) = \begin{cases} (1+\epsilon)\,A & A \ge 0 \\ (1-\epsilon)\,A & A < 0 \end{cases}
$$

**Working through the two cases for one state-action pair:**

- **Advantage positive ($A > 0$).** The objective is $\min(r, 1+\epsilon)\,A$. Raising $r$ raises the objective, but only up to $r = 1+\epsilon$. After that the value is flat at $(1+\epsilon)A$. The new policy gets no extra credit for making a good action much more likely than the old one did.
- **Advantage negative ($A < 0$).** The objective is $\max(r, 1-\epsilon)\,A$. Lowering $r$ raises the objective (because $A$ is negative), but only down to $r = 1-\epsilon$. After that it is flat. There is no extra credit for making a bad action much less likely.

In both cases the term stops rewarding movement once the policy has moved $\epsilon$ away from the old one in the direction the advantage asks for.

## 3. What the advantage means

$$
A^{\pi}(s,a) = Q^{\pi}(s,a) - V^{\pi}(s)
$$

The advantage answers: **how much better is action $a$ in state $s$ than the policy's average behaviour in that state?**

- The sign says whether to make the action more likely ($A > 0$) or less likely ($A < 0$).
- The size says how strongly. An action with $A \approx 0$ is about as good as what the policy already does.

Advantage is useful because the policy only needs to know which actions beat the average. It does not need absolute return values. This is why policy gradient methods use it instead of raw returns.

## 4. Why clipping helps

- **It removes the incentive to move too far.** Without the clip, a large advantage can push the ratio $r$ very high in one update, and the policy can collapse. The clip makes the objective flat past $1 \pm \epsilon$, so the gradient stops pushing once the update is big enough.
- **It acts as a regulariser.** The policy can still move beyond the clip range, because the clip removes the gradient signal rather than forbidding the move. So it discourages large changes without stopping them outright.
- **It is first-order.** TRPO enforces a KL constraint with a second-order method. PPO gets a similar effect with a clipped first-order objective, which is much simpler to implement. The page says it performs at least as well empirically.
- **It is not enough alone.** The Spinning Up code adds early stopping: if the mean approximate KL between the new and old policy exceeds `target_kl` (default 0.01), the gradient steps for that iteration stop.

## 5. Things I am still unsure about

- Why the `min` (rather than a plain clip) is what makes the objective pessimistic. My reading: when $r$ has moved in the direction the advantage favours, the clipped value is taken, so the objective cannot be inflated. When it has moved the other way, the unclipped value is taken, so the penalty still applies. I want to check this on paper.
- How $\hat{A}_t$ is estimated in practice. The page points to Generalized Advantage Estimation (GAE, parameter `lam` = 0.97) and I have not read that yet.

## Next

- Read the GAE part of the [PPO paper (Schulman et al. 2017)](https://arxiv.org/abs/1707.06347) and write how advantages are computed.
- Work through the case analysis above with a numeric example ($\epsilon = 0.2$, $A = +1$ and $A = -1$, several values of $r$), and check the plot against the formula.
