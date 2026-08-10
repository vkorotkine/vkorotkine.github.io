---
layout: distill
title: discretization of continuous-time noise and process models
description: some bees in my bonnet on continuous-time to discrete-time white noise conversion
tags: 
giscus_comments: true
date: 2025-03-20
featured: true
mermaid:
  enabled: true
  zoomable: true
code_diff: true
map: true
chart:
  chartjs: true
  echarts: true
  vega_lite: true
tikzjax: true
typograms: true

authors:
  # - name: Albert Einstein
  #   url: "https://en.wikipedia.org/wiki/Albert_Einstein"
  #   affiliations:
  #     name: IAS, Princeton
  # - name: Boris Podolsky
  #   url: "https://en.wikipedia.org/wiki/Boris_Podolsky"
  #   affiliations:
  #     name: IAS, Princeton
  # - name: Nathan Rosen
  #   url: "https://en.wikipedia.org/wiki/Nathan_Rosen"
  #   affiliations:
  #     name: IAS, Princeton

bibliography: 2025-03-20-discretization.bib

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
#   - name: Lie Groups
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2
  - name: Random Processes
  - name: From Continuous-Time Covariances to Discrete Time
    subsections:
      - name: Linearize Then Discretize
      - name: Continuous to Discrete Time Noise "In A Vacuum"
  # - name: Code Blocks
  # - name: Interactive Plots
  # - name: Mermaid
  # - name: Diff2Html
  # - name: Leaflet
  # - name: Chartjs, Echarts and Vega-Lite
  # - name: TikZ
  # - name: Typograms
  # - name: Layouts
  # - name: Other Typography?

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }
---
Physical systems are continuous. However, we work with discrete-time models that are amenable
to digital computers. Therefore, we create continuous models of the physical systems that we then discretize. 

This becomes particularly interesting in the case of sensor noise, since there is a randomness that needs to be modeled. Typically we use white noise models. However, while the discrete-time white noise model is fairly straightforward (a Gaussian distribution for every noise sample), the continuous-time model is really not, and deriving the transition from continuous to discrete time for white noise is nuanced. 

A key aspect in this dilemma is that it's hard to decouple the dynamics of the system we are considering from the random process noise properties. For instance, a gyroscope may have some continuous noise characteristics of its own. 
Yet, when we consider discrete-time properties, we need to look at the effect of time, and thus the overall system dynamics. This is only really tractable for linear systems (and thus, linearizations). 

The post is split into three parts. 
The first part on random processes essentially covers terminology. 
We go over why the term power spectral density is used interchangeably with continuous-time covariance. 
The second part covers the "Linearize then Discretize" method, which obtains discretizations by lumping in the effects of system dynamics with effect of random process noise. 
The first and second parts are essentially 
summaries of relevant parts of
Sec. 4.3.2, 4.4, 4.7 of <d-cite key=farrell2008navigation></d-cite>.
The third part is where things get really interesting. We consider 
continuous to discrete time noise "in a vacuum"
and try to decouple the effect of system dynamics from the
random process noise. There is a commonly used formula, which yields good results in practice,
but as far there is no completely satisfying way to derive it from first principles. 
We go through some approaches that have been proposed and discuss them. 
<!-- For instance,
white noise is not physically possible <d-cite key=farrell2008navigation></d-cite>, Sec. 4.4.2.1 as it has infinite power, and is an assumption that works for cases where the noise spectral constant interval is much larger than the bandwidth of interest.  -->





# Random Processes

Noise is modeled using random processes in continuous time. 
A random process $\mathbf{v}(t)$ defines a probability density at every time $t$.

The autocovariance of a random process $\mathbf{v}(t)$ is given by 
\begin{equation}
  \text{cov}(\mathbf{v}(t_1), \mathbf{v}(t_2))=\mathbb{E}[\mathbf{v}(t_1)\mathbf{v}(t_2)^\text{T}] -
  \mathbb{E}[\mathbf{v}(t_1)]\mathbb{E}[\mathbf{v}(t_2)^\text{T}].
\end{equation}

Essentially we take a bunch of samples at different times $t_1, t_2$,
subtract the means at those times, and see how the results vary together. 

In the context of sensor noise, a commonly used assumption is that noise is a _wide sense stationary_ random process,
where the _mean_ and _variance_ of the process are independent of time. 

For a _wide sense stationary_ process, the autocovariance $\text{cov}(\mathbf{v}(t_1), \mathbf{v}(t_2))$
_only_ depends on the time difference $\tau =t_2-t_1$, 
\begin{equation}
\text{cov}(\mathbf{v}(t_1), \mathbf{v}(t_2))=\mathbf{R}(t_2-t_1) = \mathbf{\tau}.
\end{equation}

The Power Spectral Density (PSD) of the process is given by
\begin{equation}
\mathbf{S}(j\omega)=\int_{-\infty}^\infty \mathbf{R}(\tau) \exp (-j\omega \tau) \text{d} \tau.
\end{equation}

The PSD describes the strength of the random process as different frequencies.
For the scalar case, _white_ noise is noise where the PSD is a constant for all frequencies,
$\mathbf{S}(j\omega)=\mathbf{S}_w$. Technically, this kind of noise would have infinite power - but this works in practice.

Using the inverse Fourier transform, white noise implies that the autocovariance is given by
\begin{equation}
\mathbf{R}_{\text{white}}(\tau)= \mathbf{S}_w \delta (\tau) = \mathbf{R}_c \delta (\tau).
\end{equation}
where $\delta(\tau)$ is the Dirac delta. For a long while I was confused by the fact that in
sensor datasheets, the PSD is typically stated, and we read it off and use it as the continuous-time
covariance for sensor noise. At first glance it does not make sense, since PSD is a
frequency domain concept. However, with the assumption of white noise
and a few lines of Fourier transforms, the PSD is shown to have the same value
as the continuous-time noise covariance, denoted $\mathbf{R}_c$. 

# From Continuous-Time Covariances to Discrete Time

## Linearize Then Discretize
Consider a continuous-time system of the form
\begin{equation}
\dot{\mathbf{x}}=\mathbf{A}_c(t)\mathbf{x}(t)+\mathbf{L}_c(t)\mathbf{w}(t), \quad \mathbf{w}(t) \sim \mathcal{N}(\mathbf{0}, \mathbf{Q}_c \delta(\tau))
\end{equation}

with $\mathbf{w}(t)$ being white noise, a Gaussian random process with continuous-time covariance $\mathbf{Q}_c$. 
This system may for instance come from linearization of a nonlinear system.
The goal is to find an equivalent discrete-time system of the form

\begin{equation}
\mathbf{x}_{k+1}=\mathbf{A}_k \mathbf{x}_k + \mathbf{w}_k, \quad \mathbf{w}_k \sim \mathcal{N}(\mathbf{0}, \mathbf{Q}_d),
\end{equation}

which is more tractable to reason about on digital computers. 
The solution for $\mathbf{A}_k$ is fairly straightforward, 

\begin{equation}
\mathbf{A}_k = \exp(\mathbf{A}_c \Delta t),
\end{equation}

where $\Delta t$ is the sampling period, and the matrix exponential is used. 
On the other hand, determining $\mathbf{Q}_d$ requires integration of the continuous-time dynamics to compute the discrete noise $\mathbf{w}_k$, 


<p>\begin{equation}
\mathbf{w}_k
=
\int_{-\infty}^\infty \exp (\mathbf{A} (t_{k+1}-\tau))\mathbf{L}_c (\tau) \mathbf{w}(\tau)\text{d}\tau
\end{equation}
The continuous-time covariance $\mathbf{Q}_d$ is then given by
\begin{align}
\mathbf{Q}_d&=\mathbb{E}[\mathbf{w}_k \mathbf{w}_k^{\text{T}}],
\end{align}
which, after simplification, becomes 
</p>


<p>\begin{equation}
\mathbf{Q}_d = \int_{t_k}^{t_{k+1}}
\exp(\mathbf{A_c}(t_{k+1}-\tau))
\mathbf{L}_c \mathbf{Q}_c \mathbf{L}_c^\text{T} 
\exp(\mathbf{A_c}(t_{k+1}-\tau))^{\text{T}}
\text{d} \tau.
\end{equation}
</p>

<p>
The resulting $\mathbf{Q}_d$ is thus dependent on how
the matrix exponential $\exp(\mathbf{A_c}(t_{k+1}-\tau))$ is computed.
In some cases, it can be computed in closed form, giving an exact solution for $\mathbf{Q}_d$.
It can also be approximated numerically using a Taylor series.
The Taylor series approximation for $\exp(\mathbf{A}\Delta t)$ is given by
</p>

<p>\begin{equation}
\exp(\mathbf{A}\Delta t)=\exp(\mathbf{A}_c\Delta t)=\mathbf{1}+\mathbf{A}_c\Delta t
+\frac{1}{2}(\mathbf{A}_c\Delta t)^2+\frac{1}{3!}(\mathbf{A}_c\Delta t)^3 + \dots
\end{equation}
</p>

Using a zero'th order approximation, $\exp(\mathbf{A}_c\Delta t)\approx \mathbf{1}$, yields
$\mathbf{Q}_d \approx \Delta t \mathbf{L}_c \mathbf{Q}_c \mathbf{L}_c^{\text{T}}$. 
Using the first four terms, a 3rd order Taylor series approximation, yields
<p>\begin{align}
\mathbf{Q}_d&\approx 
\mathbf{Q}_c\Delta t + (\mathbf{A}_c\mathbf{Q}_c + \mathbf{Q}_c \mathbf{A}_c^\text{T}) \frac{\Delta T^2}{2}
+(\mathbf{A}_c^2\mathbf{Q}_c +2\mathbf{A}_c\mathbf{Q}_c\mathbf{A}^\text{T}+\mathbf{Q}(\mathbf{A}_c^\text{T})^2)\frac{\Delta T^3}{6}+ \nonumber \\
&\quad(\mathbf{A}_c^3\mathbf{Q}_c+3\mathbf{A}_c^2\mathbf{Q}_c\mathbf{A}_c^\text{T}
+3\mathbf{A}_c\mathbf{Q}_c(\mathbf{A}_c^\text{T})^2+\mathbf{Q}_c(\mathbf{A}_c^\text{T})^3)\frac{T^4}{24}.
\end{align}
</p>

##  Continuous to Discrete Time Noise "In A Vacuum"

The above discussion is valid for a linear time varying system.
This makes sense, for instance, when we linearize a nonlinear system and want to discretize the result. 
However, in some situations,
we _already_ have an exact discretization of the nonlinear system.
In this situation, we _just_ want the discrete-time analog of the continuous-time noise on the sensor, without looking at system dynamics - since we already have an exact discretization.
For instance, consider the case of angular velocity kinematics with rotations,

\begin{align}
\dot{\mathbf{C}} &= \mathbf{C} \boldsymbol{\omega}^\times \\\\ 
\mathbf{C}_{k+1} &= \mathbf{C} \text{exp}({\Delta t\boldsymbol{\omega}_k}^\times),
\end{align}

where $\mathbf{C}$ is a cosine matrix describing the orientation of our robot,
$\omega$ is an angular velocity measurement, and $\times$ is the cross product operator mapping to the Lie algebra
of the space of rotations the $SO(3)$ group.

The details are irrelevant and involve Lie groups, which is a big topic in and of itself.
I have some <a href="{{ site.baseurl }}/assets/pdf/notes/lie_group_doc.pdf">notes</a> on them
and they are also covered in <d-cite key=barfoot2024state></d-cite> and <d-cite key=sola2021microlietheorystate></d-cite>.
The point is that we have a real-world example where we have 
an exact discretization of the system already, without any assumptions
on Taylor series truncations. 

The key aspect of this is that, in the section above, there are _two_ discretizations happening.
We are discretizing the linear time varying system _as well as_ the continuous random process. 
However, in the current situation, we are _only_ interested in the continuous random process.
If we take the gyroscope noise with its given continuous-time noise characteristics, and we sample it
at a given frequency: What will be the covariance on the resulting discrete-time sampled noise? 
This tends to be quite confusing, as in process noise derivations
such as Sec. 4.7 of <d-cite key=farrell2008navigation></d-cite> and Sec. 8.1.1 of <d-cite key=simon2006optimal></d-cite> the linear system and random noise process discretizations are lumped in together. Sec. 8.1.2 of <d-cite key=simon2006optimal></d-cite> is actually the relevant one for this. 

Formally, given Gaussian white noise 
\begin{equation}
\mathbf{v}(t) \sim \mathcal{N}(\mathbf{0}, \mathbf{Q}_c \delta (t_1-t_2)), 
\end{equation}

what is the "equivalent" discrete-time white noise
\begin{equation}
\mathbf{v}_k \sim \mathcal{N}(\mathbf{0}, \mathbf{Q}_d)?
\end{equation}

The "equivalent" is in quotes for good reason. For a Gaussian random process, _by definition_, 
\begin{equation}
\mathbb{E}[\mathbf{v}(t_k) \mathbf{v}(t_k)^{\text{T}}] = \mathbb{E}[\mathbf{v}_k \mathbf{v}_k^{\text{T}}] = \mathbb{E}[\mathbf{Q}_c \delta (t_1-t_2)] = \mathbb{E}[\mathbf{Q}_c \delta (t_k-t_k)] = \mathbf{Q}_c. 
\end{equation}

So, what is $\mathbf{v}_k$? It does not refer to the actual random process variable $\mathbf{v}(t_k)$.
Rather, it refers to a separate $\mathbf{v}_k$ that behaves in a way that makes sense for us in an estimator.  
The widely used (and seemingly correct) answer is
\begin{equation}
\mathbf{Q}_d = \frac{\mathbf{Q}_c}{\Delta t}.
\end{equation}

However, tracking down exactly where it comes from and how it is derived is nontrivial. 
This is the equation given by 
<a href="https://github.com/ethz-asl/kalibr/wiki/IMU-Noise-Model">the IMU noise model section of Kalibr wiki</a>, which itself cites the appendix
of J. Crassidis' sigma point Kalman filtering paper <d-cite key=crassidis2006sigma></d-cite>. 
The other source for this seem to be the 3D attitude estimation paper by N. Trawny and S. Roulemiotios <d-cite key=trawny2005indirect></d-cite>, and Dan Simon's optimal state estimation book <d-cite key=simon2006optimal></d-cite>, which is itself the cited source in <d-cite key=trawny2005indirect></d-cite>. 

### Forward Euler Discretization of Linear System And Direct Comparison
This derivation I have seen floating around, but I do not have a direct source for it. It's the one that makes the most sense though. 

We take a linear system, discretize it using a forward Euler method,
and then compare to the zero'th order approximation from the Linearize Then Discretize approach.
We are thus able to separate the discretization of the system matrices from discretization of the random process. 

Formally, start with the, for now deterministic, linear system as follows. 
This corresponds to the first step of just discretizing the system matrices, without considering the random process aspect.
Thus, for now, $\mathbf{v}$ is deterministic. 
<p>
\begin{align}
\dot{\mathbf{x}}&=\mathbf{A}_c \mathbf{x}+\mathbf{L}_c \mathbf{v},
\end{align}
</p>
a forward Euler scheme with $\dot{\mathbf{x}}\approx \frac{\mathbf{x}_{k+1}-\mathbf{x}_k}{\Delta t}$ yields
<p>
\begin{align}
\mathbf{x}_{k+1}&=\underbrace{(\mathbf{1}+\Delta t\mathbf{A}_c)}_{\mathbf{A}_d} \mathbf{x}_k+
\underbrace{\Delta t \mathbf{L}_c}_{\mathbf{L}_d} \mathbf{v}. 
\end{align}
</p>

Now let us remember the fact that $\mathbf{v}$ is actually issued from a random process and consider the system
<p>
\begin{align}
\dot{\mathbf{x}}&=\mathbf{A}_c \mathbf{x}+\mathbf{L}_c \mathbf{v}, \quad \mathbf{v} \sim \mathcal{N}(\mathbf{0}, \mathbf{Q}_c \delta (t_1-t_2)),
\end{align}
</p>

such that the equivalent discrete-time system is given by 

<p>
\begin{align}
\mathbf{x}_{k+1}&=\underbrace{(\mathbf{1}+\Delta t\mathbf{A}_c)}_{\mathbf{A}_d} \mathbf{x}_k+
\underbrace{\Delta t \mathbf{L}_c}_{\mathbf{L}_d} \mathbf{v}_k, \quad \mathbf{v}_k\sim \mathcal{N}(\mathbf{0}, \mathbf{R}_d),
\end{align}
</p>
where now $\mathbf{v}_k$ is its own Gaussian random variable, with yet to be determined $\mathbf{R}_d$. 
This is equivalent to 
<p>
\begin{align}
\mathbf{x}_{k+1}&=\mathbf{A}_d \mathbf{x}_k+\mathbf{w}_k, \quad \mathbf{w}_k\sim \mathcal{N}(\mathbf{0}, \mathbf{L}_d\mathbf{R}_d\mathbf{L}_d^T),
\end{align}
</p>

$\mathbf{R}_d$ is used for the covariance as $\mathbf{Q}_d$ has been reserved for the $\mathbf{w}_k$
from the Linearize then Discretize section. 
However, $\mathbf{w}_k$ now corresponds to the $\mathbf{w}_k$ from the Linearize then Discretize section. Using the
zero'th order approximation for $\mathbf{Q}_d$ and comparing to the $\mathbf{L}_d\mathbf{R}_d\mathbf{L}_d^T$
expression we obtained here gives
<p>
\begin{align}
\mathbf{L}_d\mathbf{R}_d\mathbf{L}_d^T &= \Delta t \mathbf{L}_c \mathbf{Q}_c \mathbf{L}_c^{\text{T}} \\
\Delta t^2
\mathbf{L}_c\mathbf{R}_d\mathbf{L}_c^T &= \Delta t \mathbf{L}_c \mathbf{Q}_c \mathbf{L}_c^{\text{T}} \\
\mathbf{R}_d &= \mathbf{Q}_c/\Delta t. 
\end{align}
</p>
We have reverse engineered the covariance on just the sensor noise itself, $\mathbf{R}_d$, by considering
a first-order forward Euler discretization to separate out the system matrix discretization from
the random process discretization. We knew what the result of the overall discretization should be from the zeroth order
approximation in the Linearize and Discretize section. Then we compared the results of the two approaches
and matched the right matrices together. 

This is the approach that made the most sense to me. However, it is very much roundabout. It does not fully answer the question of "If I just have a nonlinear discrete system with continuous covariances specified for my sensor, what is the discrete time covariance?". Rather we have to go through linearizations. 

### Constant Covariance Estimate For A Static System
This is the argument of
Sec. 8.1.2 of <d-cite key=simon2006optimal></d-cite>.
I have a few big issues with the chain of logic presented in there. If anyone has good answers to these, please let me know. 

The argument is essentially as follows. We are trying to isolate the effect of noise, so 
we take a discrete-time system whose state does not change, and whose measurement is directly equal to the state. This is like taking a gyroscope, putting it on a table (such that the true angular velocity does not change), and considering the resulting measurement. 
We then say that if we apply the Kalman filter to this system, the error covariance should not change, since there is nothing time-changing about the system. This implies a specific form for
the discrete-time covariance, which ends up being the same as the continuous-to-discrete-time conversion we seek. 
Formally, we start with the scalar system
<p>
\begin{align}
x_k &= x_{k-1} \\ 
y_k &= x_k + v_k, \quad v_k \sim \mathcal{N}(0, R_d). 
\end{align}
</p>
We then apply a Kalman filter to do a correction at a given timestep. 
There is no uncertainty in the process model, and the correction boils down to
linking subsequent covariances as
<p>
\begin{equation}
P_{k+1}=\frac{P_k R_d}{P_k + R_d}.
\end{equation}
</p>
Right off the bat, my first problem with the argument. This only holds for the scalar case.
But okay. This then implies that the covariance at timestep $k$ is given by
<p>
\begin{equation}
P_{k}=\frac{P_0 R_d}{kP_0+R_d}, 
\end{equation}
</p>
and we want to manipulate this. This is where their argument goes completely off the rails, at least in my understanding. 
We take the following limit, 
<p>
\begin{equation}
\lim_{P_0 \rightarrow \infty}P_{k}=\frac{R}{k}=\frac{R_dT}{k}.
\end{equation}
</p>

And this I completely do not understand. **_Why_** ?! Why do we have to push the initial covariance to infinity? How does this make sense?
If someone understands please let me know. 
The argument then proceeds by setting 
\begin{equation}
R_d=\frac{R_c}{T}
\end{equation}
to make the above expression independent of $T$. 

To be blunt, I do not think the argument presented in this book for the discrete to continuous time conversion
(at least, when purely considering the measurement in Sec. 8.1.2)
makes sense.
It is possible I missed something major. If I did, please let me know by sending me a mail. 
That being said, this has a commonality with the previous approach, in that we specify the continuous-time
to discrete-time covariance conversion based on behaviour we want from the estimator, and not from a pure mathematical consideration
of the random process. 

### Conclusion For Continuous to Discrete Time Noise "In A Vacuum"
I suspect there is no good explanation based on sampling or integrating the white-noise continuous Gaussian process.
One can perhaps do some kind of averaging as
mentioned in 
<a href="https://math.stackexchange.com/questions/3851352/is-there-a-continuous-time-stochastic-process-that-when-sampled-yields-discret">this Stack Overflow post.</a>
However, purely in terms of sensor noise for a robot navigation sensor model, it seems there is no mathematically satisfying way to decouple the system dynamics from the random process.
The best we can do is the forward Euler discretization comparison method. 

<!-- The question is then: 
Given continuous time noise with covariance $\mathbf{Q}_c \delta(\tau)$,
what is the equivalent 
discrete-time white noise in a system with a sample period $T$
and covariance $\mathbf{Q}_d$? What is 
relationship between $\mathbf{Q}_d$ and $\mathbf{Q}_c$? -->
<!-- 
There are multiple steps that happen when going from the continuous linear system to the discrete one
in the above section. Notably, the variable $\mathbf{w}_k$ absorbs TWO effects - one from discretizing
$\mathbf{L}_c(t)$, and the other from discretizing the covariance on $\mathbf{w}(t)$. -->
