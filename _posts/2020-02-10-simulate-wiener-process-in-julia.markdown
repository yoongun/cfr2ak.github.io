---
layout: post
title:  "Simulate wiener process in Julia"
date:   2020-02-10
tags: [simulation, physics, julia]
---

{% include mathjax.html %}

You can get the source code from [my Github](https://github.com/cfr2ak/wiener-process-julia)

## Brownian motion

Brownian motion is a sort of random walk process which represents a physical phenomenon of
stochastic movement in a micro-world.

This is the picture I'd took from the microscope (with x40 lense and CMOS sensor).

![Picture of the brownian particle](/assets/brown.jpg)

2 cm in this picture is about 2 $\mu$m in actual distance. And the dots on the screen are 
polystyrene micro-matter. Those are kind of dissolved in a water (not actually) which lets them
on a situation the brownian motion thinks for.

### The math

From the thermodynamics, the mean kinetic energy of the molecule on the equilibrium with
absolute temperature $T$, mass $M$, speed $V_x$ is given by the equation below.

$$\left\langle \frac{1}{2}MV_x^2\right\rangle = \frac{1}{2}\frac{R}{N_A}T$$

Here by the $R$ is the gas constant and the $N_A$ is the Avogadro constant.

No matter how many times a brownian particle gets force from the water
surround it, the mean momentum it gets always become 0. Funny thing in here is 
the fact that the mean value of the momentum not necessarily means that the total displacement
becomes zero. The particle can get the force to move from its original position
with time in its chaotic process!

Let's calculate to show this idea true.

Assuming there is a random force works on the particle $F_{random}$ and some resistance by the 
bonds between water is $-\beta v_x$ with particle's speed $v_x$. Then the equation becomes

$$m\frac{d^2x}{dt^2} = F_{random} - \beta v_x$$

This equation can be transformed as below by multiplying the x on the both sides.

$$m\left[\frac{d}{dt}(xv_x) - (v_x^2)\right] = xF_{random} - \frac{1}{2}\beta \frac{d}{dt}x^2$$

As assumption we've made, we can assume the value $\frac{d}{dt}\langle xv_x\rangle \rightarrow 0$
, $\langle x F_{random}\rangle \rightarrow 0$. Therefore the equation again becomes simpler

$$ \langle mv_x^2\rangle = \frac{1}{2}\beta\frac{d}{dt}\langle x^2 \rangle$$

From the thermodynamic rule we've made over there, $\langle mv_x^2\rangle = RT / N_A$ therefore we can
get the final equation (1 dimensional).

$$ \langle x^2 \rangle = 2Dt $$

the $D$ is called diffusion coefficient and can be calculated by the einstein's equation.

$$D = \frac{RT}{6\pi a \eta N_A}$$

If you calculate yourself of case 2 and 3 dimension by changing our thermodynamic equation, you can get quite
similar equation as in the 1 dimensional.

$$\langle r^2 \rangle = \langle x^2 + y^2 \rangle = 4Dt$$

$$\langle r^2 \rangle = \langle x^2 + y^2 + z^2 \rangle = 6Dt$$

### Wiener process

Wiener process is a brownian motion itself, but with more formal mathematical definitions.
From the [wikipedia](https://en.wikipedia.org/wiki/Wiener_process) you can find the definition of it
as below.

1. $W_0 = 0$
2. W has independent increments
3. W has Gaussian increments: $W_{t+u} - W_t \sim N(0, u)$
4. $W_t$ is continuous in $t$

## Implementation

Let's focus on the definition number 3: $W_{t+u} - W_t \sim N(0, u)$.
All we need is to change the position on each axis with this rule.

First, import the libraries we need to plot and generate a randomness.

{% highlight julia %}
using Plots
using Random, Distributions
{% endhighlight %}

And set some randome seed and declare the list to trace on the simulated particle.
The important part in this snippet is **Normal()**. It creates the random generate with
Gaussian distribution.

{% highlight julia %}
Random.seed!(2020)

d = Normal()
trace_x = Vector{Float64}()
trace_y = Vector{Float64}()
trace_z = Vector{Float64}()

# Initial position
append!(trace_x, 0)
append!(trace_y, 0)
append!(trace_z, 0)
{% endhighlight %}

Iterate over and save it as gif.

{% highlight julia %}
anim = @animate for i=1:1000
    dx, dy, dz = rand(d, 3)
    append!(trace_x, trace_x[i] + dx)
    append!(trace_y, trace_y[i] + dy)
    append!(trace_z, trace_z[i] + dz)

    plot(trace_x, trace_y, trace_z, markersize=2)
end
gif(anim, "./fps15.gif", fps=15)
{% endhighlight %}

## Result

That's all!

![Animation of the simulation](/assets/fps15.gif)