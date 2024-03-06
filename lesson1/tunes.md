---
layout: default
title: Songs of War
nav_order: 5
has_toc: false # on by default
has_children: false
comments: true
usetocbot: true
---
# {{ page.title }}
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}
---

# Tunes of LK
In the last exercise, you have been asked to explore playing tunes from the on-board Buzzer. The Buzzer is connected to GPIO22. To play a tune, you need to generate Pulse Width Modulation Signal.

# Pulses Width Modulation (PWM)
PWM signals are digital signals which alternate between "high" and "low" state. The duration for high state is also known as "on time" and the duration for the low state is called "off time". In order to describe the length of on time relative to off time, we use a metric called duty cycle. Duty cycle is described as percentage. It is percentage of time signal is in on state over a window of interval. For a periodic signal, its period of signal during which it assumes high state. 

![PWM](/global_assets/duty.jpeg)




