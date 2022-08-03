---
layout: post
title: Reservoir Computing
categories: [Robotics]
---

I've found reservoir computing interesting for a long time.  

$$
\textcolor{green}{s}_{t+1} = (1 - \alpha) \textcolor{green}{s}_t + \alpha (\textcolor{blue}{W} \textcolor{green}{s}_{t} + \mathbin{\color{blue}W}_{\text{in}} u_t)  , 0 \leq \alpha \leq 1
$$

$$
s_{t+1} = (1 - \alpha) s_t + \alpha (W s_t + W_{\text{in}} u_t)  , 0 \leq \alpha \leq 1
$$

$$
u_{t+1} = \textcolor{blue}{W}_{\text{out}} \textcolor{green}{s}_{t+1}
$$