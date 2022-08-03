---
layout: post
title: Reservoir Computing
categories: [Robotics]
---

I've found reservoir computing interesting for a long time.  

$$
{\color{green}s_{\color{black}t+1}} = (1 - \alpha) \textcolor{green}{s}_t + \alpha (\textcolor{blue}{W} \textcolor{green}{s}_{t} + {\color{blue}W_{\color{black}\text{in}}} u_t)  , 0 \leq \alpha \leq 1
$$

$$
s_{t+1} = (1 - \alpha) s_t + \alpha (W s_t + W_{\text{in}} u_t)  , 0 \leq \alpha \leq 1
$$

$$
u_{t+1} = \textcolor{blue}{W}_{\text{out}} \textcolor{green}{s}_{t+1}
$$