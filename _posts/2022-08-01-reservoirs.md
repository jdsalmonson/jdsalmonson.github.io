---
layout: post
title: Reservoir Computing
categories: [Robotics]
---

I've found reservoir computing interesting for a long time.  

$$
\textcolor{blue}{s}_{t+1} = (1 - \alpha) \textcolor{blue}{s}_t + \alpha (\textcolor{blue}{W} \textcolor{blue}{s}_t + \textcolor{blue}{W}_{\text{in}} u_t)
$$